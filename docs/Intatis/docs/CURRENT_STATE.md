# CURRENT_STATE

最近一次自查日期：2026-06-25

## 当前真实状态总览

- Intatis 是 clean-room 本地 AI 工作区，v0.10（git）。三个产品面：Chat / Code / Cowork。
- 11 个 Intatis* 内核模块代码就绪，10 个测试 target 有覆盖。macOS 全量、iOS chat 子集。
- Cowork 架构原则已定义（见 `docs/COWORK_PRINCIPLES.md` 与仓内 7 个 COWORK_* 文档），但当前实现与原则有已知差距。
- 真机 + 真实 key 的端到端验证状态 `UNKNOWN`。

## 已有能力

| 能力 | 入口 / 关键类型 | 测试覆盖 | 手动验证 | 真机验证 |
|---|---|---|---|---|
| IntatisMac chat | `IntatisMacApp` / `ChatViewModel` / `ChatLoop` | `IntatisConversationTests` | UNKNOWN | UNKNOWN |
| IntatisMac cowork | `CoworkViewModel` / `Orchestrator` / `AgentLoop` | `IntatisCoworkTests` / `IntatisAgentKernelTests` | UNKNOWN | UNKNOWN |
| IntatisMac multimodal | `MultimodalService` | `IntatisMultimodalTests` | UNKNOWN | UNKNOWN |
| IntatisiOS chat | `IntatisiOSApp` / `IOSAppEnvironment` | 无（SharedUI 无测试） | UNKNOWN | UNKNOWN |
| 权限 3 层门 | `PermissionEngine` / `DeterministicPolicyGate` / `ModelPermissionReviewer` | `IntatisPermissionTests` / `ReviewerTests` | UNKNOWN | UNKNOWN |
| Provider OpenAI 兼容 | `OpenAIWireProvider` / `SSE.swift` | `IntatisProvidersTests` | UNKNOWN | UNKNOWN |
| 事件日志 | `EventLog`（JSONL append-only） | `IntatisConversationTests` | UNKNOWN | UNKNOWN |
| Artifact 存储 | `ArtifactStore` | `IntatisArtifactsTests` | UNKNOWN | UNKNOWN |

## 未完成 / 进行中

- **Cowork 原则差距**（见 `docs/COWORK_PRINCIPLES.md` §6）：first-level child 仍可能获 coordinator 工具；`ask_agent` 允许 self-call；`ask_agent` 创建嵌套 AgentLoop；priorHistory 用全局投影；`spawn_agent` 被当只读；MessageBus payload 过薄；无 task contract / capability lease。按实现顺序（§7）推进中。
- **JSON-RPC out-of-process 传输未挂**：词汇已定义，`intatis agent --stdio` / `intatis daemon` 是规划中管道，当前内核全进程内运行。
- **SwiftGit2/libgit2 集成**：规划用于 sandbox 内 in-process git，许可证待审查。
- **Interactive.swift REPL 接入状态** `UNKNOWN`：需确认 `main()` 是否触达。

## 风险

- **Cowork 实现与原则漂移**：原则文档定义了 TaskContract/CapabilityLease/ContextProjector 等抽象，但当前实现尚未完全落地，易误判完成度。
- **真机验证缺口**：大量能力有测试但真机端到端状态 UNKNOWN。
- **Intatis 与 Councis 仓关系**：二仓共享 ARCHITECTURE.md 与 Packages 结构，关系未明示（Councis 是 Intatis CLI 原型分支？独立产品？）。`UNKNOWN`。
- **clean-room 覆盖**：`NOTICE.md` 以 Intatis 名义声明，Councis 复用 Intatis 模块时是否覆盖未明示。

## 工作区状态

本轮自查（2026-06-25）为只读分析，未产生仓库改动。

## 文档与源码冲突

| 冲突位置 | 冲突内容 | 采用源码为准的原因 | 建议 |
|---|---|---|---|
| 仓内 `AGENTS.md`（英文原则导向）vs 本目录 `AGENTS.md`（8 段操作协议型） | 仓内 AGENTS 是原则导向型，缺工作目录检查/完成标准/报告格式等操作协议段 | 标准化要求 8 段操作协议 | 以本目录版为标准化草稿；可选择性同步回仓内 |
| `docs/COWORK_*` 设计文档 vs 当前实现 | 设计文档描述 TaskContract/CapabilityLease/ContextProjector 等抽象，部分尚未实现 | 源码是实际实现 | 差距已记录在 `docs/COWORK_PRINCIPLES.md` §6 |
