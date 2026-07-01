# DO_NOT_BREAK

本文列出不可破坏的工程禁区、数据格式、协议、路径和回归要求。修改前必须确认不违反下列任一条目。

## 工程禁区

- 不执行破坏性 Git 操作：`git reset --hard`、`git clean -fd`、`git checkout .`、强制 push、删除未提交文件。
- 未经用户明确要求，不 commit、不 push、不创建 PR。
- 不引入新依赖，不改构建脚本，不改测试源码，除非任务明确要求。
- 不绕过辅助功能权限检查、Keychain 凭据隔离或 Carbon 全局热键注册边界。

## 数据格式禁区

- **命令 JSON**：`~/Library/Application Support/Flotis/commands.json`。`[PromptCommand]` 数组，pretty-printed + sorted keys + atomic 写。8 个默认命令固定 UUID（`1111…`–`8888…`）。首启文件缺失写默认；解码失败回退默认并设 `lastError`。
- **Provider 配置**：UserDefaults 键 `flotis.speechProviders.v1`。`SpeechProviderStoreSnapshot { providers:[SpeechProviderConfig], activeProviderID:UUID }` JSON。3 个默认 provider 固定 UUID（`AAAA…`/`BBBB…`/`CCCC…`）。`load()` 解码失败回退默认。
- **Keychain item**：generic-password，`kSecAttrAccount = apiKeyReference`。**不存 service/access group/access control**。配置只存 `apiKeyReference` 引用，不存秘密。
- **临时音频**：`FileManager.default.temporaryDirectory/<UUID>.m4a`，AAC 16000Hz mono，HTTP 路径用，转写后删除。

> 这些格式承载真实用户配置或凭据契约，改名、重排字段、改 account 键都可能造成配置丢失或 key 找不到。

## 协议禁区

- **OpenAI Realtime WebSocket 协议**：
  - client→server：`session.update`（`input_audio_format`/`input_audio_transcription{model,language?,prompt?}`/`turn_detection`=`{type:"server_vad"}`或null）、`input_audio_buffer.append`（`{type, audio: base64}`）、`input_audio_buffer.commit`
  - server→client 事件：`conversation.item.input_audio_transcription.delta/.completed`、`transcript.delta/.completed`、`transcription.*`、`input_audio_buffer.speech_started/.speech_stopped`、`error`
  - delta 文本从 `delta`/`text`/`transcript` 键（含嵌套 `item`/`transcription`）提取
  - header：`Authorization: Bearer`、`OpenAI-Beta: realtime=v1`
- **OpenAI HTTP 转写协议**：multipart/form-data POST `baseURL+endpointPath`（默认 `https://api.openai.com/v1/audio/transcriptions`）。fields：`model`、`language?`、`prompt?`、`temperature?`、`response_format=json`。file part `Content-Type: audio/m4a`。响应 JSON 解析 `text`（fallback `transcript`/`data.text`）。
- **Carbon 热键协议**：固定 ID `togglePanel=100`/`toggleVoice=200`/commands 起始 `1000`。签名 `0x464C5448`（"FLTH"）。事件类 `kEventClassKeyboard`/`kEventHotKeyPressed`。

## 路径禁区

- **命令文件**：`~/Library/Application Support/Flotis/commands.json`（atomic 写）。
- **临时音频**：`FileManager.default.temporaryDirectory/`（HTTP 路径，用后删除）。
- **Keychain**：用户默认 keychain（无 access group 隔离）。
- **目标 app 注入**：`CGEvent` 发到 `.cghidEventTap`（系统级，非特定 app）。

## 回归要求

- `ClipboardPasteInjector.inject` 在 `AccessibilityPermission.check()` 失败时必须 `completion(false)` 返回，**不得**绕过注入。
- `waitForModifierKeysToRelease` 必须在模拟 ⌘V 前运行（触发热键的修饰键可能仍按住，否则 ⌘V 变成其他组合）。
- `simulateCmdV` 用 virtualKey `0x09` + `.maskCommand`，不得改键码。
- `deleteProvider` 必须同步删除关联 Keychain item（不得遗留孤儿 key）。
- `setShortcut(_:for:)` 校验：无修饰键、等于 togglePanel/toggleVoice、已被其他命令占用 → 拒绝。不得放行。
- `CommandStore` 删除后 `enabledCommands` 仍按 `sortIndex` then `title` 排序。
- `SpeechProviderStore` 删除最后一个 provider 必须拒绝。
- 默认 provider 的固定 UUID 不得改动（配置引用依赖）。
- `LSUIElement=YES` 不得改为 NO（无 Dock 图标是核心交互）。

## 不可降级项

- `AppleSpeechTranscriber.start()` 必须请求 `SFSpeechRecognizer.requestAuthorization()` 与麦克风权限；不得跳过。
- `AppleSpeechTranscriber.stop()` 末尾 sleep 500ms 等 final transcript（不得移除，否则丢末段）。
- `OpenAIRealtimeTranscriber.stop()` 末尾 sleep 700ms 等 final transcript（同上）。
- `OpenAIHTTPTranscriber` temp `.m4a` 必须 `defer` 删除（不得泄漏）。
- `FloatingPanelController` 的 `NSPanel` style 必须含 `.nonactivatingPanel`（不抢焦点）；`level=.floating`；`hidesOnDeactivate=false`；`collectionBehavior` 含 `.canJoinAllSpaces`/`.fullScreenAuxiliary`/`.stationary`。
- `run.sh` 的 `tccutil reset Accessibility com.flotis.Flotis` 不得移除（开发期确保权限提示）。
- API key 不得写入 UserDefaults/配置文件，只存 Keychain 引用 `apiKeyReference`。

## 验证要求

修改后必须运行哪些验证才能视为安全：

- `./run.sh`（xcodegen + xcodebuild build + open）——主构建验证
- `xcodebuild -project Flotis.xcodeproj -scheme Flotis build`（无 run.sh 的清理）
- 文档任务：至少 `git diff --check` + `git status --short`
- **无测试 target**：无自动化测试可跑；代码改动须手动验证语音输入/命令注入/AX 权限流程
- 未运行构建时，最终报告必须声明"未运行构建/测试"
