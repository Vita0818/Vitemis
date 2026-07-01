# TESTING

最近自查日期：2026-06-25

## 环境

- 操作系统 / 平台：macOS 13+（deploymentTarget "13.0"，`project.yml`）
- 工具链版本：Swift 5.0（`project.yml` `SWIFT_VERSION`）；Xcode（xcodegen 生成工程）
- 依赖管理：无第三方依赖；构建工具链为 `xcodegen` + `xcodebuild`
- 凭据 / 配置：
  - API key：Keychain generic-password（account = `apiKeyReference`，如 `flotis.speechprovider.openai.realtime.apikey`）
  - Provider 配置：UserDefaults 键 `flotis.speechProviders.v1`
  - 命令：`~/Library/Application Support/Flotis/commands.json`
  - 系统权限：辅助功能（必需，paste 用）、麦克风、语音识别（Apple provider 用）

## 构建

```sh
./run.sh                                # kill旧实例 + 擦DerivedData + tccutil reset + xcodegen + xcodebuild build + open
xcodegen generate                       # 仅生成 Flotis.xcodeproj
xcodebuild -project Flotis.xcodeproj -scheme Flotis build
```

- 配置：Debug / Release（XCBuildConfiguration 均有）
- 产物：`Flotis.app`，位于 `~/Library/Developer/Xcode/DerivedData/Flotis-*/Build/Products/Debug/`

## 测试

**无测试 target。** `project.yml` 无 test target；无 `*Test*`/`*Spec*` Swift 文件；`xcshareddata` 仅自动生成 scheme。

所有验证依赖手动矩阵。

## Lint / Format

仓内无 lint/format 配置。`UNKNOWN` — 是否有 SwiftFormat/SwiftLint 需后续确认。建议至少 `xcodebuild build` 通过。

## 手动验证矩阵

| 场景 | 步骤 | 预期 | 状态 |
|---|---|---|---|
| 构建启动 | `./run.sh` | xcodegen + build 成功，Flotis.app 启动，浮动面板显示 | UNKNOWN（需真机） |
| AX 权限 | 首次启动 → 系统设置授权辅助功能 | 面板显示黄色提示 → 授权后消失 | UNKNOWN |
| 命令注入 | 焦点另一 app → ⌘⌥⇧1 | 命令 1 内容粘贴到目标 app | UNKNOWN |
| 命令按钮 | 面板点击命令按钮 | 命令内容粘贴到前台 app | UNKNOWN |
| 面板切换 | ⌘⌥⇧0 | 面板显示/隐藏切换 | UNKNOWN |
| Apple 语音 | 选 Apple provider → ⌘⌥⇧R 说话 → 再按停止 | 转写文本粘贴到目标 app | UNKNOWN |
| Realtime 语音 | 配 OpenAI Realtime key → 选该 provider → 说话 → 停止 | 流式预览 + 最终文本粘贴 | UNKNOWN（需 OpenAI key） |
| HTTP 语音 | 配 OpenAI HTTP key → 选该 provider → 说话 → 停止 | 录音→转写→文本粘贴 | UNKNOWN（需 OpenAI key） |
| provider 切换 | 面板底部 Picker 切换 | activeProvider 变更，下次语音用新 provider | UNKNOWN |
| 命令编辑 | Settings → Commands → 增删改/重排/改快捷键 | commands.json 更新，热键重注册 | UNKNOWN |
| provider 编辑 | Settings → Transcription Providers → 增删/设当前/填 key | UserDefaults + Keychain 更新 | UNKNOWN |
| 快捷键冲突 | 设置与 togglePanel/toggleVoice 相同快捷键 | 被拒绝 | UNKNOWN |
| 删除最后一个 provider | Settings 删到只剩一个再删 | 拒绝 | UNKNOWN |
| 剪贴板恢复 | 注入后检查剪贴板 | 恢复注入前内容（无新操作时） | UNKNOWN |

## 验证边界声明

- 文档任务：至少运行 `git diff --check` 与 `git status --short`；**未运行构建/测试**（无测试 target），须在最终报告中声明"未运行构建/测试"。
- 代码任务：因无测试 target，必须手动跑上表相关场景；未运行时须声明原因。
- 本目录文档为只读分析产出，未运行 `./run.sh` 或 `xcodebuild`。

## 常见问题

- **模拟器不可用**：Flotis 依赖 Carbon 全局热键、`CGEvent`、`NSPanel`、辅助功能权限，必须在真机 macOS 验证；模拟器无法验证注入链路。
- **AX 权限重置**：`run.sh` 每次 `tccutil reset Accessibility com.flotis.Flotis`，故每次 `./run.sh` 后须重新授权。
- **修饰键按住**：触发热键后立即注入会因修饰键未释放而失败；`ClipboardPasteInjector.waitForModifierKeysToRelease` 最多等 0.8s。手动测试快速连按热键时注意。
- **第三方 Realtime 兼容性**：默认 Realtime provider 假设 OpenAI API 形状（`gpt-4o-mini-transcribe`、`wss://api.openai.com/v1/realtime`、`OpenAI-Beta: realtime=v1`、pcm16 24kHz）。与第三方"OpenAI 兼容"realtime 端点的互操作 `UNKNOWN`。
