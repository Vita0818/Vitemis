# ARCHITECTURE

最近自查日期：2026-06-25

## 总体架构

Flotis 是 macOS 浮动面板语音输入工具：用户通过全局热键或面板按钮触发语音录入，语音经转写后通过模拟 ⌘V 粘贴到当前最前台的非 Flotis app。同时支持用户自定义 prompt 命令（文本片段）经快捷键一键粘贴。

```text
┌─────────────────────────────────────────────────────────┐
│  全局热键 (HotkeyManager, Carbon)                        │
│  togglePanel / toggleVoice / command快捷键              │
└──────────┬───────────────────────────┬──────────────────┘
           │                           │
   ┌───────▼───────┐          ┌────────▼─────────┐
   │ 浮动面板 UI    │          │ 语音输入控制器     │
   │ FloatingPanel  │          │ VoiceInputCtrl    │
   │ (NSPanel)      │          │ (状态机)          │
   └───────┬───────┘          └────────┬─────────┘
           │ 命令按钮点击                 │ toggleRecording
           ▼                            ▼
   ┌─────────────────┐     ┌──────────────────────────┐
   │ ClipboardPaste  │     │ 按 provider.kind 分支     │
   │ Injector        │     │ ┌────────┐ ┌──────────┐  │
   │ (模拟⌘V)        │◀────│ │Apple   │ │Realtime  │  │
   └─────────────────┘     │ │Speech  │ │WebSocket  │  │
                           │ └────────┘ └──────────┘  │
                           │ ┌──────────────────────┐ │
                           │ │HTTP (multipart .m4a) │ │
                           │ └──────────────────────┘ │
                           └──────────────────────────┘
                                      │ 转写文本
                                      ▼
                           ┌──────────────────────────┐
                           │ ClipboardPasteInjector    │
                           │ → 目标 app ⌘V             │
                           └──────────────────────────┘
```

## 主要链路

### 语音输入主链路

```text
HotkeyManager (toggleVoice 快捷键 ⌘⌥⇧R)
  -> AppDelegate.onToggleVoice -> VoiceInputController.toggleRecording()
  -> 按 VoiceInputState 分派：
     .idle/.failed -> start()
     .recording/.streaming -> stopAndInject()
     其他 -> busy 忽略
```

`start()` 读 `providerStore.activeProvider`，按 `provider.kind` 分三条路径：

#### 路径 A：Apple Speech（`.appleSpeechLive`，设备端，免费）
```text
state .requestingPermission
  -> AppleSpeechTranscriber(localeIdentifier:)
     (SFSpeechRecognizer + 自有 AVAudioEngine tap
      -> SFSpeechAudioBufferRecognitionRequest(shouldReportPartialResults:true)
      -> requestAuthorization + mic access)
  -> transcriber.start()
  -> state .recording
  -> partialTranscriptHandler/finalTranscriptHandler 更新 appState.transcriptPreview
```

#### 路径 B：OpenAI Realtime（`.openAIRealtimeTranscription`，WebSocket 流式）
```text
校验 API key (Keychain) -> 失败则 fail("请先配置…API Key")
state .connecting
  -> OpenAIRealtimeTranscriber(config:apiKey:)
     (URLSessionWebSocketTask -> realtimeURL+realtimePath+?model=
      headers: Authorization Bearer, OpenAI-Beta: realtime=v1
      send: session.update {input_audio_format, input_audio_transcription, turn_detection}
      send: input_audio_buffer.append {audio: base64})
  -> StreamingAudioCapture() (AVAudioEngine tap -> PCM16 Int16 @ sampleRate/channels)
  -> capture.audioChunkHandler -> appendRealtimeAudio -> transcriber.appendAudio
  -> transcriber.start() -> capture.start(sampleRate:24000, channels:1)
  -> state .streaming
  -> receive loop: transcription.delta/.completed -> 更新预览
```

#### 路径 C：OpenAI HTTP（`.openAIHTTPTranscription`，multipart 文件）
```text
校验 API key -> 失败则 fail
state .requestingPermission
  -> AudioRecorder() (AVAudioRecorder -> temp <UUID>.m4a, AAC 16000Hz mono)
  -> recorder.startRecording()
  -> state .recording
stop:
  -> state .transcribing
  -> recorder.stopRecording() -> temp .m4a URL
  -> OpenAIHTTPTranscriber(apiKey:).transcribeFile(fileURL, config:)
     (multipart/form-data POST baseURL+endpointPath
      fields: model, language?, prompt?, temperature?, response_format=json
      file part: Content-Type audio/m4a
      parse JSON: text / transcript / data.text)
  -> temp 文件 defer 删除
  -> injectFinalTranscript
```

### 注入链路（`ClipboardPasteInjector.inject`）

```text
1. main-thread hop; AccessibilityPermission.check() -> 失败 completion(false) 立即返回
2. 首次在 burst 中快照当前剪贴板（深拷贝所有 NSPasteboardItem types）
3. enqueue PasteOperation (monotonic generation + 目标 app)
4. processNextOperationIfNeeded:
   clearContents(); setString(text, .string)
   激活目标 app（必要时）
   等 0.1s -> waitForModifierKeysToRelease (轮询 CGEventSource.flagsState ⌘⌥⇧⌃ 最多 0.8s)
            (重要：触发热键的修饰键可能仍按住)
   simulateCmdV() (CGEvent keyDown/keyUp virtualKey 0x09 'v' + .maskCommand -> .cghidEventTap)
   等 0.5s -> finish(success:)
5. 队列排空后 0.25s 恢复剪贴板（仅当 generation 未变/无更新操作）
```

目标 app 跟踪：`NSWorkspace.didActivateApplicationNotification` 观察者记录最后前台非 Flotis app；`targetApplicationForInjection()` 用当前前台非 Flotis app 或回退到记忆的。

### 命令链路

```text
触发：
  (a) FloatingPanelView 命令按钮点击 -> ClipboardPasteInjector.inject(command.content)
  (b) 全局热键 ⌘⌥⇧1..8 -> AppDelegate.onCommandHotkeyPressed(id)
      -> commandStore 查找 -> 若 enabled -> injectCommand(command) -> 同上注入
```

## 数据模型

| 类型 | 职责 | 持久化方式 | 关键字段约束 |
|---|---|---|---|
| `PromptCommand` | prompt 命令 | `~/Library/Application Support/Flotis/commands.json`（数组，pretty+sorted+atomic） | `id: UUID`（8 默认固定 `1111…`–`8888…`）、`isEnabled`、`sortIndex`、`shortcut: KeyboardShortcutDescriptor?` |
| `KeyboardShortcutDescriptor` | 快捷键 | 随命令 | `keyCode: UInt32` + `modifiers: ShortcutModifiers` |
| `SpeechProviderConfig` | provider 配置 | UserDefaults（在 `SpeechProviderStoreSnapshot` 内） | `id: UUID`（3 默认固定 `AAAA…`/`BBBB…`/`CCCC…`）、`kind`、`apiKeyReference: String?`（Keychain account，非秘密本身） |
| `SpeechProviderKind` | provider 类型枚举 | — | `appleSpeechLive` / `openAIRealtimeTranscription` / `openAIHTTPTranscription` |
| `VoiceInputState` | 状态机 | 内存（不持久化） | idle/requestingPermission/connecting/recording/streaming/stopping/transcribing/injecting/failed(String) |
| `VoiceInputMode` | mode 枚举（疑似 vestigial） | 内存（不持久化） | appleSpeech/externalProvider；`AppState.voiceMode` 发布但未被 controller 读 |

## 同步 / 通信机制

无多端同步。所有状态进程内、单实例。`HotkeyManager` 用 Carbon 全局事件（`GetApplicationEventTarget`），回调分派到主队列。

## 安全机制

### 辅助功能权限（Accessibility）
- `AccessibilityPermission.check()`：`AXIsProcessTrustedWithOptions([kAXTrustedCheckOptionPrompt: false])`（非提示式）。
- 启动时检查；`FloatingPanelView` 每 1s 轮询；`ClipboardPasteInjector.inject` 每次注入前检查，缺失则 `completion(false)`。
- **必需**：`CGEvent` 模拟 ⌘V 依赖 AX 信任。无信任时不注入。
- `run.sh` 每次运行 `tccutil reset Accessibility com.flotis.Flotis` 并提醒用户授权。

### 麦克风 / 语音识别
- Info.plist 用法描述（中文）声明。
- 运行时懒请求：`AVCaptureDevice.requestAccess(for:.audio)`（`AudioRecorder`/`StreamingAudioCapture`/`AppleSpeechTranscriber` 各自实现）；`SFSpeechRecognizer.requestAuthorization()`（仅 `AppleSpeechTranscriber.start()`）。

### Keychain 凭据
- `KeychainSecretStore`：generic-password item（`SecItemAdd/Update/CopyMatching/Delete`）。
- **无 `kSecAttrService`、无 access group、无 access control（生物识别/硬件保护）flag**：item 仅以 `kSecAttrAccount = apiKeyReference` 为键，存于用户默认 keychain。
- 配置只存 `apiKeyReference` 字符串，不存秘密本身；UI 提示"API Key 只保存到 Keychain，不写入配置文件"。
- `deleteProvider` 同步删除关联 Keychain item。
- 默认 provider 的 `apiKeyReference`：`flotis.speechprovider.openai.realtime.apikey` / `...openai.http.apikey`；自定义 provider：`flotis.speechprovider.<UUID>.apikey`。

### 沙箱 / 签名
- **无 entitlements、无 sandbox、无 code signing 配置**（Debug/Release 的 `XCBuildConfiguration` 均无 `CODE_SIGN_*`/`DEVELOPMENT_TEAM`/`*.entitlements`）。
- 应用未沙箱化——这是 Carbon 全局热键 + `CGEvent` 注入设计的必要条件。
- `UNKNOWN` — 无 notarization/分发签名配置；分发加固状态未确认。

## 模式开关 / 内核切换

无新旧内核开关。provider 切换即"模式"切换：UI `Picker` 写 `providerStore.setActiveProvider(id:)`，`VoiceInputController.start()` 按 `activeProvider.kind` 分派。`VoiceInputMode` 枚举疑似 vestigial（published 但未读）。

## 与文档/源码的关系

- 无 README，所有架构结论从源码 + `project.yml` + `run.sh` + commit tag 推断。
- 多个文件用 typealias 使文件名与主类型名不一致（见 PROJECT_MAP 表），文档已标注实际主类型名。
