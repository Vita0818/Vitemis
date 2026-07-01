# CURRENT_STATE

最近自查日期：2026-06-25

## 当前真实状态总览

- Forgis 是本地代码迁移助手（Python CLI/Agent + 受控文件工具运行时）。代码头状态：**v7.2**（`ForgisMac` SwiftPM 可执行 + SwiftUI 三栏壳，mock/fixture 数据）on **v7.1**（本地 MVP：init/status/run --unit/resume）on **v6.0**（Qwen Visual Evidence Mode）。
- `RELEASE_NOTES.md` 仅冻结 v5.0 表面，与源码版本漂移。
- 默认 backend DeepSeek（shim over `openai_compatible_client.py`，非流式 Chat Completions）。
- 真实运行 gate：`dry_run=false` + `run_agent=true` + `confirm_real_run=true` 三者同时满足。

> 状态以源码为准。`RELEASE_NOTES.md` v5.0 与源码 v7.2 冲突时，以源码为准。

## 已有能力

| 能力 | 入口 / 关键类型 | 测试覆盖 | 手动验证 | 真机验证 |
|---|---|---|---|---|
| GH Actions 迁移工作流 | `.github/workflows/migrate.yml` | `validate-forgis.yml` smoke | UNKNOWN | UNKNOWN |
| 本地 CLI v7.1 | `agent/cli.py`（help/doctor/smoke/init/status/run --unit/resume） | `tests/test_v7_*.py` | UNKNOWN | UNKNOWN |
| 配置解析 | `agent/forgis_config.py`（`ResolvedConfig`） | `tests/test_forgis_config.py`（134 tests） | UNKNOWN | UNKNOWN |
| 默认 tool loop | `agent/forge.py`/`tool_loop.py`/`file_tools.py` | 部分 | UNKNOWN | UNKNOWN |
| 受控文件工具 | `agent/file_tools.py`（`FileToolSandbox`） | 部分 | UNKNOWN | UNKNOWN |
| 命令执行 allowlist | `agent/command_runner.py`/`build_runner.py` | 部分 | UNKNOWN | UNKNOWN |
| 安全校验 | `agent/guardrails.py`/`validate_target_output.py` | 部分 | UNKNOWN | UNKNOWN |
| 迁移单元/计划 | `agent/migration_units.py`/`migration_plan_store.py` | 部分 | UNKNOWN | UNKNOWN |
| 分阶段翻译 | `agent/staged_translation.py`/`source_inventory.py` | 部分 | UNKNOWN | UNKNOWN |
| 报告生成 | `agent/run_report.py`（`forgis.run_report.v6.0`） | fixtures | UNKNOWN | UNKNOWN |
| Qwen 视觉模式 v6.0 | `agent/qwen_vision.py`/`visual_evidence.py` | mock-first | UNKNOWN | UNKNOWN（需 `QWEN_API_KEY`） |
| ForgisMac v7.2 壳 | `Apps/ForgisMac/Sources/ForgisMacApp.swift` | 无 | UNKNOWN | mock/fixture 仅 |

## 未完成 / 进行中

- **不实现的**：流式、Responses API、本地 server/gateway、council、多 Agent、自动截图、artifact 上传、Keychain、多 provider vision、任意 shell。
- **schema 漂移**：`RELEASE_NOTES.md` 说 `forgis.run_report.v5.0`，源码实为 `v6.0`（含 visual 字段）。
- **`rules/` 目录空**：`rules/profiles/` 与 `rules/stacks/` 无加载逻辑。
- **`reports/` 目录空**。
- **README 历史版本段**可能误导（889 行/47KB，多版本段）。

## 风险

- **单一大测试文件**：`tests/test_forgis_config.py` ~6000 行/134 tests，维护风险。
- **legacy `validation_commands` 字符串**：仍经 `bash -lc` 运行（有 warning）；argv mapping 推荐。
- **视觉模式未来风险**：Phase 8+ 截图采集、Phase 9+ artifact 上传、Phase 10+ 多 provider 尚未实现。
- **README/RELEASE_NOTES 漂移**：与源码版本不一致。

## 工作区状态

本轮自查（2026-06-25）为只读分析，未产生仓库改动。

## 文档与源码冲突

| 冲突位置 | 冲突内容 | 采用源码为准的原因 | 建议 |
|---|---|---|---|
| `RELEASE_NOTES.md` vs `CURRENT_STATE.md`/`ARCHITECTURE.md`/`DO_NOT_BREAK.md` | RELEASE_NOTES 冻结 `forgis.run_report.v5.0`；源码与三份 docs 记录 `v6.0`（含 visual 字段） | 源码 `agent/run_report.py` 实际产出 v6.0 schema | 更新 RELEASE_NOTES 或标注其过期 |
| `README.md` 历史版本段 vs 当前源码 | README 含多版本段，可能误导版本判断 | 源码 v7.2 是当前真实状态 | README 标注历史段或重写 |
| 自查日期不一致 | CURRENT_STATE/PROJECT_MAP=06-25；ARCHITECTURE/DO_NOT_BREAK/TESTING=06-23；QWEN_VISUAL_MODE=05-29 | — | 统一自查日期 |
