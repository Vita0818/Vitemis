# CURRENT_STATE

最近一次自查日期：2026-06-25

## 当前真实状态总览

- Flotis 是 macOS 浮动面板语音输入 + prompt 命令注入工具，v0.4（git tag）。
- 单 target app，22 个 Swift 源文件，无第三方依赖，无 sandbox/entitlements/signing。
- 三条转写路径（Apple Speech 设备端 / OpenAI Realtime WebSocket 流式 / OpenAI HTTP multipart）代码就绪。
- 命令系统（8 默认 + 自定义）+ 三 provider 配置 + Keychain 凭据隔离已实现。
- **无 README、无测试 target、无 CHANGELOG**。所有"意图"性结论从代码 + `project.yml` + `run.sh` + commit tag 推断。
- 真机端到端验证状态 `UNKNOWN`（本轮未运行构建）。

## 已有能力

| 能力 | 入口 / 关键类型 | 测试覆盖 | 手动验证 | 真机验证 |
|---|---|---|---|---|
| 浮动面板 UI | `FloatingPanelController` / `FloatingPanelView` / `FloatingPanelLayout` | 无 | UNKNOWN | UNKNOWN |
| 全局热键 | `HotkeyManager`（Carbon） | 无 | UNKNOWN | UNKNOWN |
| 面板切换 | ⌘⌥⇧0 | 无 | UNKNOWN | UNKNOWN |
| 语音切换 | ⌘⌥⇧R | 无 | UNKNOWN | UNKNOWN |
| Apple Speech 转写 | `AppleSpeechTranscriber` | 无 | UNKNOWN | UNKNOWN |
| OpenAI Realtime 转写 | `OpenAIRealtimeTranscriber` + `StreamingAudioCapture` | 无 | UNKNOWN | UNKNOWN |
| OpenAI HTTP 转写 | `OpenAIHTTPTranscriber` + `AudioRecorder` | 无 | UNKNOWN | UNKNOWN |
| 剪贴板注入 | `ClipboardPasteInjector` | 无 | UNKNOWN | UNKNOWN |
| 命令 CRUD | `CommandStore` | 无 | UNKNOWN | UNKNOWN |
| 命令快捷键 | `HotkeyManager` + `KeyboardShortcutDescriptor` | 无 | UNKNOWN | UNKNOWN |
| Provider CRUD | `SpeechProviderStore` | 无 | UNKNOWN | UNKNOWN |
| Keychain key 管理 | `KeychainSecretStore` | 无 | UNKNOWN | UNKNOWN |
| 设置 UI | `SettingsView`（Commands/Speech/Providers 三 tab） | 无 | UNKNOWN | UNKNOWN |
| AX 权限检查 | `AccessibilityPermission` | 无 | UNKNOWN | UNKNOWN |
| 目标 app 跟踪 | `ClipboardPasteInjector`（NSWorkspace 观察者） | 无 | UNKNOWN | UNKNOWN |

## 未完成 / 进行中

- **无测试**：整个项目无测试 target。回归全靠手动。
- **无 README/CHANGELOG**：项目身份与变更史无文档化。
- **`VoiceInputMode` 疑似 vestigial**：`AppState.voiceMode` 被 published 但 `VoiceInputController` 按 `SpeechProviderKind` 分支，未见读 `voiceMode`。
- **状态不持久化**：`isPanelVisible`/`selectedSpeechLocale`/`voiceMode` 每次启动重置（无 UserDefaults 写入）。`UNKNOWN` — 是否有意。
- **无分发签名/notarization 配置**：`project.pbxproj` Debug/Release 均无 `CODE_SIGN_*`/`DEVELOPMENT_TEAM`。
- **Keychain 无 service/access group/access control**：item 仅以 account 键，无跨 app 共享、无生物识别保护。`UNKNOWN` — 是否计划加固。

## 风险

- **零测试 + 零文档**：任何改动回归风险高，且无文档锚点判断"是否破坏既有行为"。
- **未沙箱化**：Carbon 热键 + `CGEvent` 注入设计的必要条件，但意味着无 hardened runtime 保护；分发前需补签名/notarization。
- **Keychain 弱隔离**：无 service/access group，item 仅 account 键，理论上其他 app 可查询（macOS keychain 默认行为）。
- **`VoiceInputMode` 混淆**：存在两套"模式"概念（`VoiceInputMode` 枚举 vs `SpeechProviderKind` 分派），易误判哪个是真实控制流。
- **第三方 Realtime 兼容性未验证**：默认 hardcode OpenAI API 形状。
- **`run.sh` 擦 DerivedData + reset TCC**：每次运行破坏既有构建缓存与权限，适合冷启动开发，不适合增量迭代。

## 工作区状态

本轮自查（2026-06-25）为只读分析，未产生仓库改动。`.DS_Store` 存在（macOS 常态）。无其他未跟踪业务文件。

## 文档与源码冲突

无现有 `docs/` 文档，故无文档-源码冲突。本目录文档为首次创建，均据源码推断。

值得注意的源码内"自相矛盾"点（非文档冲突，但写入文档供后续参考）：

| 位置 | 内容 | 说明 |
|---|---|---|
| `AppState.voiceMode` vs `VoiceInputController` | `voiceMode` published 但未被 controller 读 | controller 按 `providerStore.activeProvider.kind` 分派；`VoiceInputMode` 疑似 vestigial |
| 文件名 vs 主类型名 | 多个 typealias 使二者不一致（`OpenAICompatibleTranscriber.swift`→`OpenAIHTTPTranscriber`；`TranscriptionProviderConfig.swift`→`SpeechProviderConfig` 等） | 文档已标注实际类型名 |
