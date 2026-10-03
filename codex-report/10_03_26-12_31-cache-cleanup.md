# Vitemis 缓存与构建产物清理记录

时间：2026-10-03 12:31 AWST（Australia/Perth）

## MODEL_CHECK_RESULT

Codex / GPT-6；运行时未提供可核验的更精确模型版本。

## PATH_CHECK_RESULT

`pwd` 与 `git rev-parse --show-toplevel` 在 Vitemis 根目录均为 `/Users/vita/Vitemis`。实际执行删除前，15 个涉及删除的真实 Git 根均重新核验一致。清理后比较了 18 个仓库的状态。

Outposts 下若干 Apple 目录和 Rokurics 的 Intatis-Snapshot 是父仓库下的复制目录，其中部分 AGENTS.md 仍指向原始项目路径。本次以实际 Git 边界为准，从 Outposts / Vela/Rokurics 的真实根执行缓存维护；没有把这些复制目录当成独立仓库。用户本次明确要求清理整个 Vitemis 的缓存和中间产物，覆盖旧文档中针对常规开发任务的禁止顺带清理规则。

## FILES_WRITTEN

- 新增本记录：`codex-report/10_03_26-12_31-cache-cleanup.md`。
- 删除经过 Git 与文件系统预检的生成物；未修改源码、测试、项目配置、依赖锁文件或现有说明文档。

## SUMMARY

- 成功删除 41,359 个精确文件/目录目标，涵盖 569,875 个文件与符号链接；删除错误 0。
- 删除目标按文件分配块合计：65.659 GiB（70.501 GB）。
- 即时磁盘可用容量：316.211 GB → 376.013 GB，增加 59.802 GB（55.695 GiB）。
- 文件分配块之和与磁盘可用容量增量不是同一计量；APFS 共享块、快照与其他系统活动可能影响差异，未将差值归因于某一个未经验证的原因。

### 各项目删除占用

| 项目 | 删除目标分配块（GiB） |
| --- | ---: |
| `Volans/Egakium` | 29.088 |
| `Outposts` | 10.424 |
| `Intatis` | 8.742 |
| `Virgo/Mopelium` | 4.499 |
| `Councis` | 4.419 |
| `Vela/Rokurics` | 3.887 |
| `Virgo/Kuzio` | 2.603 |
| `Vela/Kikaria` | 1.046 |
| `Lybris` | 0.596 |
| `Packagis` | 0.202 |
| `Flotis` | 0.148 |
| `Forgis` | 0.003 |
| `.` | 0.000 |

### 保留内容

- 全部源码、未提交改动、项目资源、测试、构建脚本、锁文件、Git 元数据与嵌套仓库。
- SwiftPM/Xcode 的依赖源码 checkout、repository 存储、SourcePackages 和必要依赖 artifacts。
- `Intatis/.intatis/runtime-kit/`、`Volans/Egakium/.deps/cef`、Chromium 第三方源码与工具链。
- 发布目录 `dist/`（实验网页的可重建 dist 除外）、`egakium-install-backups`、构建参数 `args.gn`、诊断日志、测试结果、报告及截图。
- 敏感路径和文件未读取内容；保护规则排除了凭据、会话、密钥、证书等。

## VALIDATION_RESULT

- 删除前：按准确路径核对 HEAD 跟踪情况、暂存状态与 ignored/untracked 状态；删除范围不包含已跟踪文件。
- 删除前：再次检查候选子树无 Git 仓库、受保护文件或最近 10 分钟发生变化的文件；符号链接仅删除链接本身，不跟随。
- 删除后：41,359 个目标全部确认不存在。
- 1,651,783 个保留文件/链接的路径、mode、大小、mtime 和 inode 元信息摘要在清理前后完全一致；这是文件元信息校验，不是全文内容哈希。
- 18 个仓库的 `git status --short` 在写本报告前与清理前完全一致，包含原有修改、删除与未跟踪状态。
- Vitemis 根 `git diff --check` 退出码 0。
- 未运行构建/测试：本次没有业务源码、测试或配置改动；重新构建会再次生成刚清理的缓存。
- 不需要更新架构、接口或用户教程：没有产品行为或构建配置的持久变更，清理事实记录在本报告。

## UNCERTAINTIES

- 仍保留约 1.897 GiB 的已跟踪候选缓存/元数据；清理它们将产生仓库文件删除记录，不属于本次保持现有 Git 文件状态的范围。
- 未验证重新构建成功；下次构建需要重新生成编译产物，部分实验的 Node 依赖需要重新安装。
- 132.67 GiB 的最初盘点是排除 Git 元数据及受保护路径后的文件分配块合计，不是整个 Vitemis 的完整物理磁盘占用。

## NEXT_RECOMMENDED_ACTION

需要继续收缩为更严格的源码归档时，可单独审查已跟踪生成物、发布包和安装备份；本次保留它们，不自动扩展到源码快照、Git 历史或必要 Runtime Kit。

## CLEANED_SCOPES

以下列出经过审核的清理根及实际删除量。每个根内的受保护内容按上述规则保留。

| 清理根 | 删除量（MiB） | 精确目标数 |
| --- | ---: | ---: |
| `.DS_Store` | 0.02 | 1 |
| `Councis/.DS_Store` | 0.01 | 1 |
| `Councis/.build` | 4026.45 | 3348 |
| `Councis/Vendor/SwiftStreamingMarkdown/.build` | 499.03 | 6 |
| `Flotis/FloatingCapsult/.build` | 151.87 | 6 |
| `Forgis/.DS_Store` | 0.01 | 1 |
| `Forgis/agent/__pycache__` | 1.89 | 66 |
| `Forgis/rules/.DS_Store` | 0.01 | 1 |
| `Forgis/tests/.DS_Store` | 0.01 | 1 |
| `Forgis/tests/__pycache__` | 1.08 | 22 |
| `Forgis/tmp/.DS_Store` | 0.01 | 1 |
| `Intatis/.DS_Store` | 0.02 | 1 |
| `Intatis/.build` | 7091.07 | 6040 |
| `Intatis/Experiments/WebRendererParity/dist` | 10.51 | 2 |
| `Intatis/Experiments/WebRendererParity/node_modules` | 140.77 | 346 |
| `Intatis/Packages/IntatisTools/Runtime/rbook-helper/target` | 490.27 | 5 |
| `Intatis/ThirdPartyStandards/.DS_Store` | 0.01 | 1 |
| `Intatis/Vendor/SwiftStreamingMarkdown/.build` | 1219.13 | 13 |
| `Intatis/codex-report/.DS_Store` | 0.02 | 1 |
| `Lybris/.build` | 610.15 | 1631 |
| `Outposts/.mcp-tools/qwen-vision/__pycache__` | 0.01 | 1 |
| `Outposts/Flotis-Apple/.DS_Store` | 0.01 | 1 |
| `Outposts/Flotis-Apple/FloatingCapsult/.build` | 151.87 | 6 |
| `Outposts/Intatis-Apple/.DS_Store` | 0.02 | 1 |
| `Outposts/Intatis-Apple/.build` | 66.15 | 954 |
| `Outposts/Intatis-Apple/.swiftpm/.DS_Store` | 0.01 | 1 |
| `Outposts/Intatis-Apple/Experiments/WebRendererParity/dist` | 10.51 | 2 |
| `Outposts/Intatis-Apple/Experiments/WebRendererParity/node_modules` | 140.77 | 346 |
| `Outposts/Intatis-Apple/Packages/IntatisTools/Runtime/rbook-helper/target` | 490.27 | 5 |
| `Outposts/Intatis-Apple/ThirdPartyStandards/.DS_Store` | 0.01 | 1 |
| `Outposts/Intatis-Apple/Vendor/SwiftStreamingMarkdown/.build` | 1395.25 | 13 |
| `Outposts/Intatis-Apple/codex-report/.DS_Store` | 0.02 | 1 |
| `Outposts/Kikaria-Apple/.DS_Store` | 0.01 | 1 |
| `Outposts/Kikaria-Apple/.build` | 6064.23 | 55 |
| `Outposts/Kikaria-Apple/Kikaria/.DS_Store` | 0.01 | 1 |
| `Outposts/Kikaria-Apple/Kikaria/Assets.xcassets/.DS_Store` | 0.01 | 1 |
| `Outposts/Kikaria-Apple/Kikaria/Assets.xcassets/AppIcon.appiconset/.DS_Store` | 0.01 | 1 |
| `Outposts/Kikaria-Apple/Presets/.DS_Store` | 0.01 | 1 |
| `Outposts/Rokurics-Apple/.DS_Store` | 0.02 | 1 |
| `Outposts/Rokurics-Apple/.build` | 2135.95 | 46 |
| `Outposts/Rokurics-Apple/AppData/.DS_Store` | 0.01 | 1 |
| `Outposts/Rokurics-Apple/Rokurics/.DS_Store` | 0.02 | 1 |
| `Outposts/Rokurics-Apple/Rokurics/Assets.xcassets/.DS_Store` | 0.01 | 1 |
| `Outposts/Rokurics-Apple/RokuricsShared/.DS_Store` | 0.01 | 1 |
| `Outposts/Rokurics-Apple/RokuricsVisualDiagnostics/DerivedData` | 219.45 | 6 |
| `Outposts/Rokurics-Apple/TimeTrace/.DS_Store` | 0.01 | 1 |
| `Outposts/Rokurics-Apple/docs/.DS_Store` | 0.01 | 1 |
| `Packagis/.build` | 206.80 | 9 |
| `Vela/.DS_Store` | 0.01 | 1 |
| `Vela/Kikaria/.DS_Store` | 0.01 | 1 |
| `Vela/Kikaria/.build` | 1071.07 | 21 |
| `Vela/Kikaria/Kikaria/.DS_Store` | 0.01 | 1 |
| `Vela/Kikaria/Kikaria/Assets.xcassets/.DS_Store` | 0.01 | 1 |
| `Vela/Kikaria/Kikaria/Assets.xcassets/AppIcon.appiconset/.DS_Store` | 0.01 | 1 |
| `Vela/Kikaria/Presets/.DS_Store` | 0.01 | 1 |
| `Vela/Rokurics/.build` | 2135.95 | 46 |
| `Vela/Rokurics/Intatis-Snapshot/Experiments/WebRendererParity/dist` | 10.51 | 2 |
| `Vela/Rokurics/Intatis-Snapshot/Experiments/WebRendererParity/node_modules` | 140.77 | 346 |
| `Vela/Rokurics/Intatis-Snapshot/Packages/IntatisTools/Runtime/rbook-helper/target` | 490.27 | 5 |
| `Vela/Rokurics/Intatis-Snapshot/Vendor/SwiftStreamingMarkdown/.build` | 1203.28 | 13 |
| `Virgo/.DS_Store` | 0.01 | 1 |
| `Virgo/Kuzio/.build` | 930.18 | 2221 |
| `Virgo/Kuzio/DerivedData` | 1735.74 | 2165 |
| `Virgo/Kuzio/Packages/KuzioLibraryAPI/.build` | 0.01 | 2 |
| `Virgo/Mopelium/.DS_Store` | 0.01 | 1 |
| `Virgo/Mopelium/.build` | 4086.39 | 3337 |
| `Virgo/Mopelium/Packages/MopeliumTools/Runtime/rbook-helper/target` | 199.73 | 4 |
| `Virgo/Mopelium/Vendor/SwiftStreamingMarkdown/.build` | 320.89 | 5 |
| `Volans/.DS_Store` | 0.01 | 1 |
| `Volans/Egakium/.DS_Store` | 0.01 | 1 |
| `Volans/Egakium/.build` | 2734.72 | 3072 |
| `Volans/Egakium/Chromium/checkout/src/out/EgakiumNative` | 12510.82 | 11313 |
| `Volans/Egakium/Chromium/cipd-cache` | 2541.65 | 2 |
| `Volans/Egakium/Experiments/WebRendererParity/dist` | 10.51 | 2 |
| `Volans/Egakium/ThirdPartyStandards/.DS_Store` | 0.01 | 1 |
| `Volans/Egakium/build` | 11988.25 | 5835 |
| `Volans/Egakium/codex-report/.DS_Store` | 0.02 | 1 |
| `docs/.DS_Store` | 0.01 | 1 |
| `skills/.DS_Store` | 0.01 | 1 |

## TRACKED_SCOPES_PRESERVED

| 已跟踪候选范围 | 保留候选量（MiB） |
| --- | ---: |
| `Dashis/.DS_Store` | 0.01 |
| `Dashis/Packages/DashisCodexBarCollector/.build` | 554.73 |
| `Flotis/.DS_Store` | 0.01 |
| `Forgis/.build` | 93.87 |
| `Outposts/.DS_Store` | 0.02 |
| `Outposts/.gradle` | 0.00 |
| `Outposts/Dashis-Apple/.DS_Store` | 0.01 |
| `Outposts/Flotis-Windows/.DS_Store` | 0.01 |
| `Outposts/Intatis-Android/.gradle` | 1.22 |
| `Outposts/Intatis-Android/app/build` | 90.04 |
| `Outposts/Intatis-Android/shared/build` | 2.08 |
| `Outposts/Kikaria-Android/.gradle` | 1.32 |
| `Outposts/Kikaria-Android/app/build` | 142.71 |
| `Outposts/Mopelium-Apple/.DS_Store` | 0.01 |
| `Outposts/Rokurics-Android/.DS_Store` | 0.01 |
| `Outposts/Rokurics-Android/.gradle` | 12.79 |
| `Outposts/Rokurics-Android/app/build` | 316.33 |
| `Outposts/Rokurics-Windows/.DS_Store` | 0.01 |
| `Outposts/Rokurics-Windows/Rokurics.Tests/bin` | 92.81 |
| `Outposts/Rokurics-Windows/Rokurics.Tests/obj` | 1.27 |
| `Outposts/Rokurics-Windows/Rokurics/bin` | 286.85 |
| `Outposts/Rokurics-Windows/Rokurics/obj` | 126.95 |
| `Outposts/docs/.DS_Store` | 0.01 |
| `Vela/Rokurics/.DS_Store` | 0.02 |
| `Vela/Rokurics/AppData/.DS_Store` | 0.01 |
| `Vela/Rokurics/Intatis-Snapshot/.DS_Store` | 0.02 |
| `Vela/Rokurics/Intatis-Snapshot/ThirdPartyStandards/.DS_Store` | 0.01 |
| `Vela/Rokurics/Intatis-Snapshot/codex-report/.DS_Store` | 0.02 |
| `Vela/Rokurics/Rokurics/.DS_Store` | 0.02 |
| `Vela/Rokurics/Rokurics/Assets.xcassets/.DS_Store` | 0.01 |
| `Vela/Rokurics/RokuricsShared/.DS_Store` | 0.01 |
| `Vela/Rokurics/RokuricsVisualDiagnostics/DerivedData` | 219.45 |
| `Vela/Rokurics/TimeTrace/.DS_Store` | 0.01 |
| `Vela/Rokurics/docs/.DS_Store` | 0.01 |
