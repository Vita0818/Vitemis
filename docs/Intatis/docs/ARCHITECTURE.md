# ARCHITECTURE

最近自查日期：2026-06-25

## 总体架构

Intatis 是 clean-room 本地 AI 工作区，三个产品面：Chat（普通多模态对话）/ Code（单 agent 本地工作区）/ Cowork（多 agent 本地工作区协作）。macOS 全量；iOS chat 子集。

```text
                    ┌─────────────────────────────────────┐
                    │      Intatis* 内核模块（共享）        │
                    │  Core / Protocol / Providers         │
                    │  Conversation / Artifacts / Multimodal│
                    │  Tools / Permission / AgentKernel     │
                    │  Cowork / SharedUI                   │
                    └──────────────┬──────────────────────┘
                                   │
            ┌──────────────────────┼──────────────────────┐
            │                      │                      │
   ┌────────▼─────────┐  ┌────────▼─────────┐  ┌─────────▼──────────┐
   │  IntatisMac       │  │  IntatisiOS       │  │  intatis-cli        │
   │  Chat/Code/Cowork │  │  Chat 子集        │  │  CLI                │
   │  (全量内核)        │  │  (无 workspace)   │  │                    │
   └───────────────────┘  └───────────────────┘  └────────────────────┘
```

## 主要链路

### Chat 链路（无工具，iOS/macOS chat 子集）

```text
ChatLoop.send() -> buildHistory() from EventLog
  -> append userMessage -> provider.stream
  -> message_delta / message_completed -> turnStats
（无工具、无权限、无工作区）
```

### Cowork 链路（多 agent 编排，macOS 全量）

```text
CoworkViewModel -> Orchestrator (actor)
  -> AgentRegistry / MessageBus(log:, mediator:) / PermissionEngine
  -> attach(agent) -> log agent_attached
  -> send(text, to:@mention) -> 路由到目标 agent
  -> 为每次 run 构建 AgentLoop + BusMessenger + OrchestratorManager
       coordinator(canCoordinate=true) -> ToolRegistry.standard + [AskAgent/Spawn/List/Remove]
       worker -> ToolRegistry.standard 仅
  -> AgentLoop.send() 循环（maxIterations 默认 50）
       ContextBuilder.initialMessages (system + priorHistory 投影 + user)
       -> provider.stream -> message_delta
       -> toolCalls -> runTool -> PermissionEngine.decide
            askUser -> PermissionResponder -> UI 卡片 (CheckedContinuation)
       -> ToolObservation 回填 -> 重复直至无 tool call
  -> 每次状态变更 append 到 EventLog
MessageBus.deliver -> Mediator.mediate
  -> SecretScanner.containsSecret -> block
  -> content.count > maxChars(4000) -> block "send a summary"
  -> ForwardingReviewer(可选)
  -> .forward (log agent_to_agent_message + permission_review allow)
```

关键不变量：
- 两级层级，无递归 spawn；worker 默认无 coordinator 工具。
- `@main` agent 不可被 remove。
- MessageBus 是唯一投递路径；Mediator 默认转发摘要不转发原始字节。
- 任何 model tool_call 到执行都必须过 PermissionEngine，无旁路。

### 多模态链路

```text
MultimodalService.generateImage/transcribe/generateVideo(轮询 job)
  -> provider 调用 -> ArtifactStore 写入
  -> log: artifact_added / artifact_progress
```

## 数据模型

| 类型 | 职责 | 持久化方式 | 关键字段约束 |
|---|---|---|---|
| `Envelope` | 事件信封 | JSONL 一行一个 | `seq` 单调递增；`v:1`；18 种 `type` |
| `Event` | 事件 payload 联合 | 随 Envelope | 追加演进；replay 时不可解码行跳过 |
| `Agent` | agent 值类型 | 内存 | `canCoordinate: Bool`；默认 profile `.reviewed` |
| `Capability` | provider 能力枚举 | 配置 | chat/tool_calling/vision/realtime/audio/image/video/embedding |
| `PlatformProfile` | 平台能力信封 | launch-time | `.iOS`（最受限）/`.macAppStore`/`.macDeveloperID`；`current` 默认 `.iOS` |
| `PermissionProfile` | 每 agent 模式 | agent | manual/reviewed/autopilot/read_only/locked；硬 DENY 优先 |
| `Artifact` / `ArtifactRef` | blob + 索引 | blobs/<id>.<ext> + index.json | ISO-8601；pretty JSON |

## 同步 / 通信机制

- **进程内**：v0.1 内核全进程内运行。`Orchestrator`/`EventLog`/`MessageBus` 均为 `actor`。
- **JSON-RPC 2.0 词汇**已定义（`JSONRPC.swift`：Command→request、Envelope→event notification），但**尚未挂传输**。未来 `intatis agent --stdio` / `intatis daemon` 是规划中管道。`UNKNOWN` — 当前无 out-of-process 传输实现。
- **Provider 线协议**：OpenAI 兼容 HTTP/SSE（`/chat/completions` streaming）。`WireFormat.openai` 是唯一 shipped 格式。

## 安全机制

### 凭据存储
- `KeychainStore`（GUI）：generic-password item，service `com.intatis.app`/`com.intatis.ios`，account `default-openai`。配置只存 `KeychainRef` 引用，不存秘密；`KeychainSecretResolver` 懒加载。

### 工作区边界
- `PathConfinement.resolve`：拒 `..` 遍历与越界绝对路径。Tools 执行与权限门均使用。

### 权限 3 层门
1. `DeterministicPolicyGate`（纯函数、模型无关、runs first、`deny` 终局）：locked→deny；敏感路径→deny；路径越界→deny；read_only 下网络→deny；readOnly side-effect→allow；destructive→ask；exec→evaluateShell；write→evaluateWrite。
2. `ModelPermissionReviewer`（模型评审）：只见 gate `pass`；只能收窄为 deny/ask，**不能**覆盖硬 deny。
3. `PermissionEngine`（组合）：`pass` 且无 reviewer → `askUser`。

### 秘密拦截
- `SecretScanner`：敏感路径/basename/扩展名、秘密内容标记（`-----BEGIN`、`PRIVATE KEY`、`AKIA`、`sk-`、`ssh-rsa`、`xoxb-`、`ghp_`、`AIza`…）、受保护配置路径。
- `Mediator`：agent 间转发时拦截秘密 + 超长原始转储（>4000 字符要求发摘要）。

### Sandbox / Entitlements
- `IntatisMac.AppStore.entitlements`：`app-sandbox=true`、`network.client=true`、`files.user-selected.read-write`、`files.bookmarks.app-scope`。**无 `run_shell`**。
- `IntatisMac.DeveloperID.entitlements`：非 sandbox、Hardened Runtime。

## 模式开关 / 内核切换

无 Rokurics 式新旧内核开关。Cowork 架构原则见 `docs/COWORK_PRINCIPLES.md`——当前实现与原则的差距见该文档"当前已知 Cowork 问题"。

## 与文档/源码的关系

- 仓内根 `ARCHITECTURE.md`（draft-0, 2026-06-11，中文）描述 Intatis 内核/Cowork 设计。本目录 `docs/ARCHITECTURE.md` 据实际源码重写并与之对照。
- Cowork 设计细节见 `docs/COWORK_AGENT_ARCHITECTURE.md` 等 7 个 COWORK_* 文档；原则提炼见 `docs/COWORK_PRINCIPLES.md`。
