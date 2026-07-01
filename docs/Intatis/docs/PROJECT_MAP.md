# PROJECT_MAP

最近自查日期：2026-06-25

本文描述当前仓库结构。判断依据来自 `Package.swift`、`project.yml`、`Makefile`、源码、测试文件和脚本。

## 目录结构总览

```text
Intatis/
├── .build/            SwiftPM 构建产物（gitignored）
├── .git/              Git 仓库（remote: github.com/Vita0818/Intatis，v0.10）
├── .gitattributes     LF 规范化
├── .gitignore         忽略 .build、Intatis.xcodeproj、*.env
├── .swiftpm/          SwiftPM 缓存
├── AGENTS.md          英文原则导向型入口（本目录 docs/ 为标准化中心版）
├── Apps/              app target
│   ├── IntatisMac/    全量 macOS app（链接全部 11 个 product）+ entitlements
│   ├── IntatisiOS/    Chat-only iOS app（7-product 子集）
│   └── intatis-cli/   CLI
├── ARCHITECTURE.md    Intatis 架构设计（draft-0，2026-06-11，中文）
├── Makefile           build/test/release/install/app 便利 target
├── NOTICE.md          Clean-room 声明 + 依赖策略
├── Package.swift      SwiftPM manifest（11 lib + CLI + 10 test target）
├── Packages/          11 个库模块（各 Sources/ + Tests/，SharedUI 无 Tests）
├── README.md          Intatis readme
├── docs/              Cowork 设计文档与状态（7 个 COWORK_*）
└── project.yml        XcodeGen spec（生成 Intatis.xcodeproj）
```

## Target / 模块

### 库 target（11）— `Packages/<Name>/Sources`

| Target | 类型 | 依赖 | 职责 |
|---|---|---|---|
| `IntatisCore` | lib | — | 类型化 ID、错误、工作区路径约束、平台能力信封、会话类型 |
| `IntatisProtocol` | lib | Core | 结构化事件/线协议词汇（Event/Envelope/JSONRPC/Command/CoworkEvents/MultimodalEvents/TurnStats/JSONValue） |
| `IntatisProviders` | lib | Core, Protocol | OpenAI 兼容模型访问（ProviderRegistry/Capability/Endpoints/ChatProvider/OpenAIWireProvider/OpenAIToolCalling/SSE/HTTPDataClient/ImageGeneration/Transcription/VideoGeneration/ToolCalling） |
| `IntatisArtifacts` | lib | Core, Protocol | 文件-backed blob 存储 + JSON 索引 |
| `IntatisConversation` | lib | Core, Protocol, Providers, Artifacts | 事件溯源会话 + UI 投影（EventLog JSONL / ChatLoop / Projection / CodeProjection） |
| `IntatisTools` | lib | Core, Protocol | 哑工具执行器（ToolProtocol/FileTools/PatchTool/PathConfinement/ShellGit） |
| `IntatisPermission` | lib | Core, Protocol, Providers | 3 层权限门（PermissionTypes/PermissionEngine/DeterministicPolicyGate/ModelPermissionReviewer/SecretScanner） |
| `IntatisAgentKernel` | lib | Core, Protocol, Providers, Tools, Permission, Conversation, Artifacts | 单 agent 工具循环（Agent/AgentLoop/ContextBuilder/PermissionResponder） |
| `IntatisCowork` | lib | Core, Protocol, Providers, Tools, Permission, Conversation, AgentKernel | 多 agent 编排（Orchestrator/MessageBus/Mediator/AgentRegistry/CoordinatorTools/AskAgentTool） |
| `IntatisMultimodal` | lib | Core, Protocol, Providers, Artifacts, Conversation | 图像/视频/转写 → artifacts |
| `IntatisSharedUI` | lib | Core, Protocol, Providers, Conversation, Artifacts | 跨平台 SwiftUI。**无 Tests** |

### App target

| Target | 类型 | 平台 | Bundle ID | 链接 |
|---|---|---|---|---|
| `IntatisMac` | application | macOS | `com.intatis.app` | 全部 11 个 product |
| `IntatisiOS` | application | iOS | `com.intatis.ios` | 7 个 chat 子集 product（无 Tools/Permission/AgentKernel/Cowork） |
| `intatis-cli` | executable | CLI（macOS/Linux） | — | Core, Protocol, Providers, Conversation, Tools, Permission, AgentKernel, Cowork |

### 测试 target（10）— `Packages/<Mod>/Tests`

`IntatisCoreTests`、`IntatisProtocolTests`（+V02/V03/V04）、`IntatisProvidersTests`（+Multimodal/ToolCalling）、`IntatisArtifactsTests`、`IntatisConversationTests`（+Code）、`IntatisToolsTests`、`IntatisPermissionTests`（+Reviewer）、`IntatisAgentKernelTests`、`IntatisCoworkTests`、`IntatisMultimodalTests`。`IntatisSharedUI` 无测试。`swift test` 无头。

## 关键文件

- 入口：`Apps/IntatisMac/Sources/IntatisMacApp.swift`、`Apps/IntatisiOS/Sources/IntatisiOSApp.swift`、`Apps/intatis-cli/Sources/IntatisCLI.swift`
- Cowork 链路：`Packages/IntatisCowork/Sources/Orchestrator.swift`、`MessageBus.swift`、`Mediator.swift`、`CoordinatorTools.swift`、`AskAgentTool.swift`
- Agent 内核：`Packages/IntatisAgentKernel/Sources/AgentLoop.swift`、`Agent.swift`、`ContextBuilder.swift`、`PermissionResponder.swift`
- 权限门：`Packages/IntatisPermission/Sources/PermissionEngine.swift`、`DeterministicPolicyGate.swift`、`ModelPermissionReviewer.swift`、`SecretScanner.swift`
- 事件日志：`Packages/IntatisConversation/Sources/EventLog.swift`
- Provider：`Packages/IntatisProviders/Sources/OpenAIWireProvider.swift`、`ProviderRegistry.swift`
- Cowork 设计文档：`docs/COWORK_AGENT_ARCHITECTURE.md`、`COWORK_TASK_CONTEXT_MODEL.md`、`COWORK_AGENT_INVOCATION_MODEL.md`、`COWORK_CURRENT_FINDINGS.md`、`COWORK_MIGRATION_PLAN.md`、`COWORK_V0_10_SMOKE.md`、`COWORK_V0_10_STATUS.md`

## 生成物 / 产物

- SwiftPM 构建产物：`.build/`（含 release 可执行）
- Xcode 工程产物：`Intatis.xcodeproj`（xcodegen 生成，gitignored）
- App bundle：`IntatisMac.app` / `IntatisiOS.app`（Xcode 构建产物）

## 脚本与工具

| 脚本 | 用途 | 调用方式 |
|---|---|---|
| `Makefile` | build/test/release/install/app/clean 便利 target | `make build` / `make test` / `make app` 等 |
| `project.yml` | XcodeGen 工程规格 | `xcodegen generate`（`make app` 内调用） |

## 不确定项

- Intatis 仓与 Councis 仓共享同一 `ARCHITECTURE.md` 与 `Packages/` 结构。二者关系（Councis 是 Intatis 的 CLI 原型分支？独立产品？）`UNKNOWN` — 需用户确认。
- `Apps/intatis-cli/Sources/Interactive.swift` REPL 是否接入 `main()` `UNKNOWN`（Councis 仓调研显示疑似死代码，Intatis 仓需独立确认）。
