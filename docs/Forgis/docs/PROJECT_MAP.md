# PROJECT_MAP

最近自查日期：2026-06-25

本文描述当前仓库结构。判断依据来自 `Package.swift`、`.github/workflows/`、源码、测试和 `requirements.txt`。

## 目录结构总览

```text
Forgis/
├── .github/workflows/   migrate.yml（主工作流）+ validate-forgis.yml（smoke）
├── Apps/ForgisMac/      v7.2 SwiftUI 壳（mock/fixture，6 个 .swift）
├── agent/               主体 Python CLI/Agent 代码（*.py + *.sh）
├── docs/                7 份项目文档（5 标准 + QWEN_VISUAL_MODE + DS_GUIDE）
├── examples/            local_migration_fixture/
├── prompts/             prompt 模板
├── reports/             空（报告输出目录）
├── rules/               profiles/ + stacks/（空，无加载逻辑）
├── skills/              6 个迁移 skill（migration_general/swiftui_to_compose/...）
├── tests/               test_forgis_config.py(~6000行) + test_v7_*.py 等
├── tmp/                 临时
├── AGENTS.md            入口协议
├── README.md            889行/47KB（含历史版本段）
├── RELEASE_NOTES.md     冻结 v5.0
└── requirements.txt     仅 PyYAML>=6.0.2
```

## Target / 模块

| Target | 类型 | 入口 | 职责 |
|---|---|---|---|
| Forgis CLI/Agent | Python | `agent/cli.py`（本地）/ `.github/workflows/migrate.yml`（CI） | 本地代码迁移助手 + 受控文件工具 |
| ForgisMac | SwiftPM 可执行 | `Apps/ForgisMac/Sources/ForgisMacApp.swift` `@main` | v7.2 SwiftUI 三栏壳（mock/fixture） |

## 关键文件

- 入口：`.github/workflows/migrate.yml`、`agent/cli.py`、`agent/forge.py`、`agent/resolve_config.py`、`agent/tool_loop.py`、`Apps/ForgisMac/Sources/ForgisMacApp.swift`
- 配置解析：`agent/forgis_config.py`（`ResolvedConfig`/`VisualValidationConfig`/`StagedTranslationConfig`）
- 模型客户端：`agent/openai_compatible_client.py`、`agent/deepseek_agent.py`（默认 `deepseek-v4-pro`）
- 工具沙箱：`agent/file_tools.py`（`FileToolSandbox`）、`agent/command_runner.py`、`agent/build_runner.py`
- 安全校验：`agent/guardrails.py`、`agent/validate_target_output.py`、`agent/model_env.py`
- 迁移计划：`agent/migration_units.py`、`agent/migration_plan_store.py`、`agent/migration_state.py`、`agent/plan_audit.py`、`agent/staged_translation.py`、`agent/source_inventory.py`
- 报告：`agent/run_report.py`、`agent/pr_body.py`
- 视觉模式：`agent/qwen_vision.py`、`agent/visual_evidence.py`、`agent/runtime_controller.py`
- Shell：`agent/create_pr.sh`、`agent/build_target.sh`
- 测试：`tests/test_forgis_config.py`（~6000 行/134 tests）、`tests/test_openai_compatible_client.py`、`tests/test_v7_*.py`

## 配置文件

- `requirements.txt`：仅 `PyYAML>=6.0.2`
- 目标仓内：`FORGIS_CONFIG.yml`（或 CLI `--config`）、`FORGIS_TASK.md`
- `model_env`：仅 env 名映射，不存真实 secret
- `validation_commands`：argv mapping 推荐；legacy 字符串经 `bash -lc`（有 warning）
- `visual_validation` block

## 生成物 / 产物

- `FORGIS_RUN_REPORT.json`（schema `forgis.run_report.v6.0`，含 visual block）
- `FORGIS_MIGRATION_PLAN.json`（写 `v5.0`，读 `v4.8`/`v3.9`/`v3.8`/`v3.7`）
- `FORGIS_LOG.md`（run log，默认 `{target_subdir}/FORGIS_LOG.md`）
- PR body（30000 chars 标准，3000 chars 短）
- 视觉证据：`visual-evidence/<run_id>/<target_repo_slug>/{reference,actual,qwen}`
- `reports/`（空目录）

## 脚本与工具

| 脚本 | 用途 | 调用方式 |
|---|---|---|
| `.github/workflows/migrate.yml` | 主迁移工作流（输入 `target_repo`） | GH Actions 手动触发 |
| `.github/workflows/validate-forgis.yml` | controller smoke test | GH Actions |
| `agent/create_pr.sh` | 创建 PR | 工作流内调用 |
| `agent/build_target.sh` | 构建目标 | 工作流内调用 |

## 不确定项

- `rules/profiles/` 与 `rules/stacks/` 空（无加载逻辑）。
- README 历史版本段可能误导版本判断。
