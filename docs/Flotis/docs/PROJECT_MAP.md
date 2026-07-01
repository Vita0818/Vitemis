# PROJECT_MAP

最近自查日期：2026-06-25

本文描述当前仓库结构。判断依据来自 `project.yml`、`run.sh`、`Flotis.xcodeproj/project.pbxproj`、源码和 git 历史。

## 目录结构总览

```text
Flotis/
├── .git/              Git 仓库（main 分支，tag v0.1–v0.4）
├── Flotis/            22 个 Swift 源文件
├── Flotis.xcodeproj/  xcodegen 生成（project.pbxproj, xcworkspace, 空 xcshareddata）
├── project.yml        xcodegen 规格（单 macOS app target）
└── run.sh             清理 + xcodegen + xcodebuild + open 辅助脚本
```

## Target / 模块

| Target | 类型 | 平台 | 入口 | 职责 |
|---|---|---|---|---|
| `Flotis` | application | macOS 13+ | `Flotis/FlotisApp.swift` `@main struct FlotisApp` | 浮动面板语音输入 + prompt 命令注入工具，`LSUIElement=YES`（无 Dock 图标） |

- Bundle ID：`com.flotis.Flotis`（`project.yml` `bundleIdPrefix: com.flotis`）。
- Swift 5.0；`GENERATE_INFOPLIST_FILE=YES`；`INFOPLIST_KEY_LSUIElement=YES`。
- Info.plist 用法描述：`NSMicrophoneUsageDescription`、`NSSpeechRecognitionUsageDescription`（中文）。
- **无第三方依赖**；无 entitlements / 无 sandbox / 无 code signing 配置（Debug/Release 均无）。

## 关键文件

`Flotis/` 下 22 个源文件：

| 文件 | 主类型（注意 typealias） | 职责 |
|---|---|---|
| `FlotisApp.swift` | `FlotisApp` / `AppDelegate` | `@main` 入口；装配 AppState/CommandStore/ProviderStore/VoiceController/PanelController/HotkeyManager；`Settings` 场景 |
| `AppState.swift` | `AppState` | 中央 `ObservableObject`：published UI/voice 状态 + 辅助功能检查 |
| `VoiceInputMode.swift` | `VoiceInputMode` / `VoiceInputState` | mode 枚举（疑似 vestigial）+ 状态机枚举（中文 displayText） |
| `VoiceInputController.swift` | `VoiceInputController` | `@MainActor` 状态机：捕获→转写→注入，按 provider.kind 分支 |
| `SpeechTranscribing.swift` | `StreamingSpeechTranscribing` / `FileSpeechTranscribing`（`typealias SpeechTranscribing`） | 转写协议 |
| `AppleSpeechTranscriber.swift` | `AppleSpeechTranscriber` | 设备端 `SFSpeechRecognizer` + `AVAudioEngine` 流式 |
| `OpenAIRealtimeTranscriber.swift` | `OpenAIRealtimeTranscriber` | WebSocket OpenAI Realtime（PCM16 base64 流） |
| `OpenAICompatibleTranscriber.swift` | `OpenAIHTTPTranscriber`（`typealias OpenAICompatibleTranscriber`） | multipart 文件上传 `/v1/audio/transcriptions` |
| `TranscriptionProviderConfig.swift` | `SpeechProviderConfig` / `SpeechProviderKind`（`typealias TranscriptionProviderConfig`） | provider 模型 + 3 个默认 provider（固定 UUID） |
| `TranscriptionProviderStore.swift` | `SpeechProviderStore`（`typealias TranscriptionProviderStore`） | singleton：CRUD/persist provider（UserDefaults）+ active 选择 + key 读写 |
| `AudioRecorder.swift` | `AudioRecorder` | 文件式 `AVAudioRecorder` → temp `.m4a`（HTTP 路径用） |
| `StreamingAudioCapture.swift` | `StreamingAudioCapture` | `AVAudioEngine` tap → PCM16 chunk（Realtime 路径用） |
| `ClipboardPasteInjector.swift` | `ClipboardPasteInjector` | singleton：快照剪贴板→置文本→激活目标 app→等修饰键→模拟 ⌘V→恢复 |
| `HotkeyManager.swift` | `HotkeyManager` | singleton：Carbon 全局热键注册/分派 |
| `PromptCommand.swift` | `PromptCommand` / `KeyboardShortcutDescriptor` / `ShortcutModifiers` / `KeyCodeDisplay` | 命令模型 + 快捷键描述符 + 键码映射 |
| `CommandStore.swift` | `CommandStore` | singleton：CRUD/persist 命令（JSON 文件）+ 快捷键校验 + 默认 |
| `FloatingPanelController.swift` | `FloatingPanelController` | `NSWindowController` over floating `NSPanel` + 尺寸/定位 |
| `FloatingPanelView.swift` | `FloatingPanelView` / `FloatingPanelLayout` | 主面板 UI（命令网格/状态/语音按钮/provider picker/预览）+ 自适应布局 |
| `VoiceSettingsView.swift` | `SettingsView` + 多个编辑子视图 | 设置窗口（Commands/Speech/Providers 三 tab）+ `ShortcutRecorderView` |
| `UIStrings.swift` | — | 集中中文 UI 字符串常量 |
| `AccessibilityPermission.swift` | `AccessibilityPermission` | AX 信任检查 + 打开系统设置 |
| `KeychainSecretStore.swift` | `KeychainSecretStore` | singleton：generic-password Keychain CRUD（API key） |

## 生成物 / 产物

- 构建产物：`Flotis.app`（在 `~/Library/Developer/Xcode/DerivedData/Flotis-*/Build/Products/Debug/`）
- 用户数据：
  - `~/Library/Application Support/Flotis/commands.json`（命令，atomic 写）
  - UserDefaults 键 `flotis.speechProviders.v1`（provider 配置）
  - Keychain generic-password（account = `apiKeyReference`）
  - temp `<UUID>.m4a`（HTTP 转写临时音频，用后删除）

## 脚本与工具

| 脚本 | 用途 | 调用方式 |
|---|---|---|
| `run.sh` | kill 旧实例 → 擦 DerivedData → `tccutil reset Accessibility com.flotis.Flotis` → `xcodegen` → `xcodebuild build` → 定位 `.app` → `open` | `./run.sh` |
| `project.yml` | xcodegen 工程规格 | `xcodegen generate`（`run.sh` 内调用） |

## 不确定项

- `VoiceInputMode` 枚举疑似 vestigial：`AppState.voiceMode` 被 published 但 `VoiceInputController` 实际按 `SpeechProviderKind` 分支，未见读 `voiceMode`。`UNKNOWN` — 是否计划用于未来 UI。
- `AppState.isPanelVisible` / `selectedSpeechLocale` / `voiceMode` 不持久化（无 UserDefaults 写入），每次启动重置。`UNKNOWN` — 是否有意为之。
- 无 README / CHANGELOG / 测试 target。所有"意图"性结论均从代码与 commit tag（v0.1–v0.4）推断。
