# Vitemis 最低可继续开发状态清理记录

时间：2026-10-03 14:11 AWST（Australia/Perth）

## MODEL_CHECK_RESULT

Codex / GPT-6；运行时未提供可核验的更精确模型版本。

## PATH_CHECK_RESULT

- 工作区根：`/Users/vita/Vitemis`，pwd 与 Git root 一致。
- 已按各自真实 Git root 记录并复核 18 个源码仓库；所有这些源码仓库的 `.git` 标记仍在。
- 范围扩展到用户明确要求的 Mac 上 Xcode / iOS 调试缓存、模拟器设备与系统镜像，以及 Vitemis 可重建产物。
- 其他项目的 DerivedData、Xcode 本体、用户配置、证书与实际手机数据不在删除范围。

## FILES_WRITTEN

- 新增本记录：`codex-report/10_03_26-14_11-minimal-restart-cleanup.md`。
- 删除 103 个已审核的文件/目录目标；删除错误 0。
- 使用 `xcrun simctl delete` 删除准确列出的 23 台已关闭模拟器；使用 `xcrun simctl runtime delete` 删除唯一 iOS 27.0 / 24A5390f 系统镜像。
- 用户另行明确选择“删除这 6 份旧归档”后，删除对应 Kikaria / KikariaMac `.xcarchive`。
- 未编辑业务源码、测试、工程配置、构建脚本或依赖锁文件。未 stage、commit、push 或执行 Git 历史变更。

## SUMMARY

- 本轮清理期间磁盘可用容量：373.555 GB → 425.984 GB，增加 **52.429 GB（48.828 GiB）**。
- 上一轮单独测得增加 59.802 GB；两轮分别测得的清理增量合计约 112.230 GB。两轮之间其他系统活动会影响当前磁盘净余额。
- 本轮文件目标按分配块合计 42.457 GiB，另外移除模拟器数据及约 8.047 GB 的系统镜像。分配块合计与磁盘实际增量不是同一指标，不将差额未经验证地归因于某一种原因。

### 主要清理项

| 类型 | 清理情况 |
| --- | --- |
| 模拟器 | 23 台设备删除；当前设备数 0、系统镜像数 0 |
| 真机调试缓存 | Mac 上三份 iOS DeviceSupport 符号缓存清空，约 13.30 GiB |
| Vitemis Xcode DerivedData 与共享编译缓存 | 20.431 GiB |
| 仓库内残留构建产物、依赖 checkout 缓存、旧产品包及备份 | 8.019 GiB |
| Xcode 旧归档 | 6 份 Kikaria / KikariaMac 归档，约 203.875 MiB |
| 文档下载缓存 | 约 0.510 GiB，可由 Xcode 再下载 |

### 最低保留内容

- 全部项目源码、资源、测试、构建脚本、工程配置、锁文件、Git 历史、项目说明与下一目标记录。
- 现有第三方源码仓库与固定版本的必要依赖资产。
- `Intatis/.intatis/runtime-kit/`：本地定制 Runtime Kit，保留以支持准确版本的再次集成。
- `Volans/Egakium/.deps/cef` 和 `config/cef.cmake`：保留当前 pinned CEF 及接线配置。
- Chromium 源码与 `Chromium/checkout/src/out/EgakiumNative/args.gn`：保留源码及手写构建参数，编译输出已清理。
- 当前 Xcode 27.0（27A5228h）及其工具链仍安装在原位置。

### 具有源代码价值的两个例外

| 保留目录 | 原因 | 大小 |
| --- | --- | ---: |
| `Outposts/Kikaria-Apple/.build` | 含状态无法确认的 SwiftMath checkout，不能证明可无损删除 | 135.19 MiB |
| `Volans/Egakium/Vendor/SwiftStreamingMarkdown/.build` | swift-markdown / swift-cmark checkout 存在本地差异，按源码保留 | 11.26 MiB |

共约 146.45 MiB。没有为了让缓存目录数量归零而丢弃可能有用的源码状态。

## VALIDATION_RESULT

- 所有 103 个审核目标均确认不存在；三个删除批次均无错误。
- `simctl list devices --json`：设备数 0；`simctl runtime list --json`：镜像数 0。
- Simulator Devices 目录仅剩约 24 KiB 的管理元数据；Xcode Archives 为 0；DerivedData 约 95.38 MiB，主要属于范围外项目。
- 审核了 226 个构建缓存中的依赖 checkout；其中 218 个无本地差异且 HEAD 可在已抓取的上游 remote 历史中找到。最终删除了 216 个落在已批准生成物范围内的 checkout；不确定的目录保留。
- 1,581,692 个保留文件/链接的路径、mode、大小、mtime、inode 元信息摘要清理前后相同。该校验不是全文内容哈希。
- 18 个源码仓库保留；新增 18,896 条已跟踪生成物删除记录，均位于批准的缓存/产物路径。所有原有业务源码状态仍保留。
- 较最早 Git 状态快照另外观察到 4 个 `.DS_Store` 变为 M；它们在删除计划形成时已被保护、未被本轮脚本删除/写入，删除前后的文件元信息校验也一致。未回退这些元数据变化。
- 根目录 `git diff --check` 通过；未运行构建/测试，因为源码与配置未变，运行会重新生成刚移除的缓存。
- 自动审批最初拒绝了包含 Archives 的整批全局删除。随后将可重建缓存单独执行；6 份归档在用户明确批准后单独删除。没有绕过审批。

## UNCERTAINTIES

- 未声称已经通过一次全新环境重建；本次验收是源码/配置保留与清理范围核对。
- 下次开发需要重新解析或下载清掉的依赖缓存；iOS 模拟器测试需要重新下载兼容的系统镜像并创建设备。
- 再连接真机调试时，Xcode 可能重新生成 DeviceSupport / 符号缓存。
- 删除记录尚未提交；Git 历史中原有生成物仍存在，本次没有改写历史或清理 Git 对象。

## RESTART_GUIDE

1. 进入准备继续的项目根目录，先读该项目 `AGENTS.md`、`docs/CURRENT_STATE.md`、`docs/TESTING.md`，以及仍存在的 `docs/NEXT_TARGET.md`。所有源码保持原来的目录关系。
2. 使用保留的 `Package.swift` / `Package.resolved` 或 Xcode 工程解析依赖；按项目原有文档生成工程并构建。首次构建会重新创建 `.build`、DerivedData、Rust target 或网页依赖等缓存。
3. Intatis 下游项目继续引用同一份 `/Users/vita/Vitemis/Intatis`。使用留下的 exact Runtime Kit；Egakium 继续使用留下的 pinned CEF。
4. 需要 iOS 测试时，在 Xcode 下载兼容的 iOS Simulator runtime 并创建新模拟器。真机调试由 Xcode 按设备版本重新准备支持文件。
5. 旧 `.app`、安装备份与归档按用户要求移除；需要产物时从留下的工程重新构建。

## DOCUMENTATION_AND_NEXT_ACTION

源码、接口和构建配置均未改变，无需改写业务架构或用户教程；本次环境收缩、保留边界及恢复方法统一记录在本报告。当前清理任务完成，后续开发由用户选择项目与目标后再开始。

## EXACT_CLEANUP_SCOPE

| 路径 | 类型 | 原分配块（MiB） |
| --- | --- | ---: |
| `/Users/vita/Vitemis/Outposts/.gradle` | platform-build-cache | 0.00 |
| `/Users/vita/Vitemis/Forgis/.build` | build-cache | 93.87 |
| `/Users/vita/Vitemis/Packagis/.build` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Intatis/.build` | build-cache | 345.95 |
| `/Users/vita/Vitemis/Lybris/.build` | build-cache | 175.81 |
| `/Users/vita/Vitemis/Councis/.build` | build-cache | 174.18 |
| `/Users/vita/Vitemis/Intatis/dist` | old-product-bundle | 570.32 |
| `/Users/vita/Vitemis/Volans/Egakium/.build` | build-cache | 173.65 |
| `/Users/vita/Vitemis/Forgis/tests/__pycache__` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Flotis/FloatingCapsult/.build` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Outposts/Intatis-Android/.gradle` | platform-build-cache | 1.22 |
| `/Users/vita/Vitemis/Virgo/Mopelium/.build` | build-cache | 171.94 |
| `/Users/vita/Vitemis/Vela/Kikaria/.build` | build-cache | 526.55 |
| `/Users/vita/Vitemis/Outposts/Kikaria-Android/.gradle` | platform-build-cache | 1.32 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Apple/.build` | build-cache | 6.56 |
| `/Users/vita/Vitemis/Vela/Rokurics/.build` | build-cache | 6.56 |
| `/Users/vita/Vitemis/Volans/Egakium/build` | xcode-and-cef-build | 2773.24 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Android/.gradle` | platform-build-cache | 13.01 |
| `/Users/vita/Vitemis/Virgo/Kuzio/DerivedData` | build-cache | 175.60 |
| `/Users/vita/Vitemis/Forgis/agent/__pycache__` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Virgo/Kuzio/.build` | build-cache | 175.55 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/.build` | build-cache | 0.12 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/dist` | old-product-bundle | 570.32 |
| `/Users/vita/Library/Developer/Xcode/iOS DeviceSupport` | developer-cache | 13617.86 |
| `/Users/vita/Library/Developer/Xcode/DocumentationCache` | developer-cache | 522.16 |
| `/Users/vita/Library/Developer/Xcode/DocumentationIndex` | developer-cache | 0.00 |
| `/Users/vita/Library/Developer/CoreSimulator/Caches` | developer-cache | 0.00 |
| `/Users/vita/Library/Developer/CoreSimulator/Temp` | developer-cache | 0.00 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Windows/Rokurics.Tests/bin` | platform-build-cache | 92.81 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Windows/Rokurics.Tests/obj` | platform-build-cache | 1.27 |
| `/Users/vita/Vitemis/Councis/Vendor/SwiftStreamingMarkdown/.build` | build-cache | 67.73 |
| `/Users/vita/Vitemis/Volans/Egakium/Chromium/cipd-cache` | cipd-download-cache | 0.00 |
| `/Users/vita/Vitemis/Vela/Rokurics/RokuricsVisualDiagnostics/DerivedData` | build-cache | 219.47 |
| `/Users/vita/Vitemis/Outposts/Intatis-Android/shared/build` | gradle-build | 2.11 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Android/app/build` | gradle-build | 316.43 |
| `/Users/vita/Vitemis/Intatis/Experiments/WebRendererParity/dist` | web-build | 0.00 |
| `/Users/vita/Vitemis/Outposts/Intatis-Android/app/build` | gradle-build | 90.09 |
| `/Users/vita/Vitemis/Dashis/Packages/DashisCodexBarCollector/.build` | build-cache | 618.13 |
| `/Users/vita/Vitemis/Intatis/Experiments/WebRendererParity/node_modules` | node-dependencies | 0.05 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Apple/RokuricsVisualDiagnostics/DerivedData` | build-cache | 0.02 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Windows/Rokurics/obj` | platform-build-cache | 126.95 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Windows/Rokurics/bin` | platform-build-cache | 286.85 |
| `/Users/vita/Vitemis/Outposts/.mcp-tools/qwen-vision/__pycache__` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Intatis/Vendor/SwiftStreamingMarkdown/.build` | build-cache | 67.42 |
| `/Users/vita/Vitemis/Outposts/Flotis-Apple/FloatingCapsult/.build` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Outposts/Kikaria-Android/app/build` | gradle-build | 142.77 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/CompilationCache.noindex` | xcode-derived-data | 0.03 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Councis-akslpacfnbgtcxavivrvkmzfdwhg` | xcode-derived-data | 1162.95 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Dashis-cckvunhznnuqxcajshxmhxrgxjlk` | xcode-derived-data | 718.63 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Egakium-dfelnxgucsctrbcuyjqxfzdwibwp` | xcode-derived-data | 4022.65 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Flotis-dvzbmzoinaxfibfrlsxtpwkkajni` | xcode-derived-data | 158.65 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Forgis-gbizuonqtlacavfzbubgwcfjlnxd` | xcode-derived-data | 175.65 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Intatis-eihwkjottzheczfznbegmrerxdoj` | xcode-derived-data | 3170.97 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Intatis-gdqrzudjxpqvoydvbrvdthmtnnkc` | xcode-derived-data | 5064.64 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Kikaria-gngajovbzvpmgjasdcngemmohdch` | xcode-derived-data | 262.77 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Kuzio-crqiyinmvyhqkvfuqjpdbiujjrzp` | xcode-derived-data | 118.63 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/ModuleCache.noindex` | xcode-derived-data | 980.94 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Mopelium-ayhedmbkyopsticgczhiqcbyofjp` | xcode-derived-data | 3306.00 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Rokurics-cgwdzudifyyzembcxrbnwajskfjl` | xcode-derived-data | 925.71 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/SDKExplicitPrecompiledModules` | xcode-derived-data | 601.41 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/SDKStatCaches.noindex` | xcode-derived-data | 32.19 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/SymbolCache.noindex` | xcode-derived-data | 42.49 |
| `/Users/vita/Library/Developer/Xcode/DerivedData/Translatis-ejhpnaqohhakrhfoqcjxsmcppwbf` | xcode-derived-data | 176.93 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/Vendor/SwiftStreamingMarkdown/.build` | build-cache | 67.42 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/Experiments/WebRendererParity/node_modules` | node-dependencies | 0.05 |
| `/Users/vita/Vitemis/Virgo/Kuzio/Packages/KuzioLibraryAPI/.build` | build-cache | 0.00 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/Experiments/WebRendererParity/dist` | web-build | 0.00 |
| `/Users/vita/Vitemis/Virgo/Mopelium/Vendor/SwiftStreamingMarkdown/.build` | build-cache | 67.84 |
| `/Users/vita/Vitemis/Volans/Egakium/Experiments/WebRendererParity/dist` | web-build | 0.00 |
| `/Users/vita/Library/Developer/Xcode/Archives/2026-05-21/Kikaria 2026-5-21, 15.06.xcarchive` | old-project-archive | 34.54 |
| `/Users/vita/Library/Developer/Xcode/Archives/2026-05-10/Kikaria 2026-5-10, 17.05.xcarchive` | old-project-archive | 34.87 |
| `/Users/vita/Library/Developer/Xcode/Archives/2026-06-15/Kikaria 2026-6-15, 18.53.xcarchive` | old-project-archive | 34.54 |
| `/Users/vita/Library/Developer/Xcode/Archives/2026-05-15/Kikaria 2026-5-15, 14.11.xcarchive` | old-project-archive | 34.18 |
| `/Users/vita/Library/Developer/Xcode/Archives/2026-05-15/KikariaMac 2026-5-15, 14.23.xcarchive` | old-project-archive | 44.43 |
| `/Users/vita/Library/Developer/Xcode/Archives/2026-05-07/Kikaria 2026-5-7, 17.36.xcarchive` | old-project-archive | 21.30 |
| `/Users/vita/Vitemis/Intatis/Packages/IntatisTools/Runtime/rbook-helper/target` | rust-build | 0.00 |
| `/Users/vita/Vitemis/Vela/Rokurics/Intatis-Snapshot/Experiments/WebRendererParity/node_modules` | node-dependencies | 0.05 |
| `/Users/vita/Vitemis/Vela/Rokurics/Intatis-Snapshot/Vendor/SwiftStreamingMarkdown/.build` | build-cache | 67.42 |
| `/Users/vita/Vitemis/Vela/Rokurics/Intatis-Snapshot/Experiments/WebRendererParity/dist` | web-build | 0.00 |
| `/Users/vita/Vitemis/Virgo/Mopelium/Packages/MopeliumTools/Runtime/rbook-helper/target` | rust-build | 0.00 |
| `/Users/vita/Vitemis/Volans/Egakium/Chromium/checkout/src/out/EgakiumNative/obj` | chromium-build | 18.10 |
| `/Users/vita/Vitemis/Volans/Egakium/Chromium/checkout/src/out/EgakiumNative/gen` | chromium-build | 2.07 |
| `/Users/vita/Vitemis/Volans/Egakium/Chromium/checkout/src/out/EgakiumNative/package.json` | chromium-build | 0.00 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/Packages/IntatisTools/Runtime/rbook-helper/target` | rust-build | 0.00 |
| `/Users/vita/Vitemis/Vela/Rokurics/Intatis-Snapshot/Packages/IntatisTools/Runtime/rbook-helper/target` | rust-build | 0.00 |
| `/Users/vita/Vitemis/Outposts/Dashis-Apple/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Outposts/Flotis-Windows/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Apple/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Outposts/docs/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Outposts/Rokurics-Android/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Outposts/Mopelium-Apple/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Outposts/Intatis-Apple/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Forgis/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Councis/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Vela/Rokurics/RokuricsShared/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Vela/Rokurics/TimeTrace/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Vela/Rokurics/docs/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Vela/Rokurics/AppData/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Vela/Rokurics/Rokurics/.DS_Store` | finder-metadata | 0.02 |
| `/Users/vita/Vitemis/Vela/Rokurics/Rokurics/Assets.xcassets/.DS_Store` | finder-metadata | 0.01 |
| `/Users/vita/Vitemis/Vela/Rokurics/Intatis-Snapshot/codex-report/.DS_Store` | finder-metadata | 0.02 |
| `/Users/vita/Vitemis/Vela/Rokurics/Intatis-Snapshot/ThirdPartyStandards/.DS_Store` | finder-metadata | 0.01 |

### 核对期间保留的 Finder 元数据变更

- `Dashis/.DS_Store`：保留当前内容和 Git 状态。
- `Outposts/Rokurics-Windows/.DS_Store`：保留当前内容和 Git 状态。
- `Vela/Rokurics/.DS_Store`：保留当前内容和 Git 状态。
- `Vela/Rokurics/Intatis-Snapshot/.DS_Store`：保留当前内容和 Git 状态。
