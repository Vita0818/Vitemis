# TESTING

最近自查日期：2026-06-25

## 环境

- 操作系统 / 平台：macOS 13+（库与 CLI）；iOS 16+（IntatisiOS）；CLI 理论支持 Linux（`#if canImport(SwiftUI)` 守卫）
- 工具链版本：Swift 5.9
- 依赖管理：SwiftPM（`Package.swift`）+ XcodeGen（`project.yml` → `Intatis.xcodeproj`）。v0.1 **零第三方依赖**。
- 凭据 / 配置：Keychain（service `com.intatis.app`/`com.intatis.ios`，account `default-openai`）+ UserDefaults（`intatis.baseURL`、`intatis.model`）

## 构建

### SwiftPM（库 + CLI）

```sh
swift build                 # Debug
make build                  # 同上
swift build -c release      # Release
make release                # 同上
make install                # 符号链接到 BINDIR
```

### XcodeGen（.app bundle）

```sh
make app                    # xcodegen generate && open Intatis.xcodeproj
```

## 测试

```sh
swift test                  # 全部无头 XCTest
make test                   # 同上
swift test --filter IntatisCoworkTests
swift test --filter IntatisPermissionTests/ReviewerTests
```

- 测试 target（10）：`Packages/<Mod>/Tests/`。`IntatisSharedUI` 无测试。
- `swift test` 无头：无测试 target 依赖 UI/app target。

## Lint / Format

仓内无显式 lint/format 配置。`UNKNOWN` — 是否有 SwiftFormat/SwiftLint 需后续确认。建议至少 `swift build` 通过。

## 手动验证矩阵

| 场景 | 步骤 | 预期 | 状态 |
|---|---|---|---|
| IntatisMac chat | `make app` → Xcode 运行 IntatisMac → chat 发消息 | 流式回复 | UNKNOWN（需真机 + key） |
| IntatisMac cowork | Xcode 运行 IntatisMac → cowork → @mention | agent 间路由 + 权限卡片 | UNKNOWN |
| IntatisMac multimodal | Xcode 运行 → 图像/转写 | artifact 写入 + 事件 | UNKNOWN |
| IntatisiOS chat | Xcode 运行 IntatisiOS → chat | 流式回复，无工具/shell | UNKNOWN |
| iOS 子集边界 | 检查 IntatisiOS 链接的 product | 不含 Tools/Permission/AgentKernel/Cowork | 已确认（project.yml） |
| 权限门硬 deny | worker 尝试 spawn_agent | 被拒 | 已有测试覆盖 |
| Cowork 循环 | A→B→A 委派 | 被拒 | UNKNOWN — 见 COWORK_PRINCIPLES §8 |

## 验证边界声明

- 文档任务：至少运行 `git diff --check` 与 `git status --short`；**未运行构建/测试**，须声明。
- 代码任务：按改动风险运行相称的 `make test` / `make build` / `make app`；改 Cowork/AgentKernel 必须加测试（见 `docs/COWORK_PRINCIPLES.md` §8）。

## 常见问题

- **Linux 构建**：`IntatisSharedUI` 用 `#if canImport(SwiftUI)` 守卫，包应能在 Linux 无头构建。
- **Cowork 原则 vs 实现**：当前实现与 `docs/COWORK_PRINCIPLES.md` 原则有已知差距（见该文档 §6"当前已知 Cowork 问题"）。改动前先核对差距清单。
