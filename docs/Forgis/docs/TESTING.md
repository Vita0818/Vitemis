# TESTING

最近自查日期：2026-06-25

## 环境

- Python 3.11（`actions/setup-python@v5`）
- 依赖：仅 `PyYAML>=6.0.2`（`pip install -r requirements.txt`）
- bash（shell 脚本）；`git`+`gh`（PR）
- GH Actions secrets：`FORGIS_TARGET_TOKEN`/`FORGIS_SOURCE_TOKEN`/model secret env
- 测试用 mock HTTP（无真实 API）；视觉测试无需真实 Qwen key

## 构建

```sh
python3 -m py_compile agent/*.py
bash -n agent/create_pr.sh
bash -n agent/build_target.sh
```

## 测试

```sh
# CI 跑的窄集
python -m unittest tests/test_forgis_config.py tests/test_openai_compatible_client.py tests/test_v7_cli_config.py tests/test_v7_local_cli.py tests/test_v7_local_smoke.py tests/test_v7_local_init_status.py tests/test_v7_local_migration_flow.py tests/test_v7_validation_commands.py

# 全量
python3 -m unittest
```

- 集成：`validate-forgis.yml` controller smoke（创建 `tmp/source`/`tmp/target` + 最小 config/task，跑 `forge.py`）。本地 CLI dry-run smoke：`doctor`/`smoke --workdir`/`init`/`status`/`run --unit`/`resume`。
- UI 测试：N/A（无前端 UI；v6.0 仅受控视觉工具）。

## Lint / Format

无 ruff/black/mypy/prettier/eslint 配置。仅 `git diff --check` + `bash -n`。

## 手动验证矩阵

| 场景 | 步骤 | 预期 | 状态 |
|---|---|---|---|
| 配置解析 | 跑 config 测试 | 134 tests 过 | 已记录通过 |
| 视觉配置 | visual_validation 字段校验 | 拒绝 secret/key/path | UNKNOWN |
| Qwen 适配器 | mock-first 测试 | 无 key 返回 `QWEN_PERMISSION_GATED` | UNKNOWN |
| dry-run | `doctor`/`smoke --workdir` | 不真实运行 | UNKNOWN |
| 本地 v7.1 流程 | `init`/`status`/`run --unit`/`resume` | 单元执行 + 报告 | UNKNOWN |
| validation_commands | argv mapping vs legacy 字符串 | argv 推荐 | UNKNOWN |
| 工具沙箱 | 拒绝危险路径 | 拒绝 | UNKNOWN |
| 分阶段翻译 | staged_translation 流程 | overview→per_file→stabilization | UNKNOWN |
| 迁移计划 | persistence/resume | 写 v5.0，读旧版 | UNKNOWN |
| 报告 | `forgis.run_report.v6.0` | 含 visual block | UNKNOWN |
| GH 工作流 | migrate.yml | 失败分支 PR | UNKNOWN |

## 验证边界声明

- 文档任务：至少 `git diff --check` + `git status --short`；**未运行构建/测试**须声明。
- 代码任务：按改动风险跑窄测试或全量；`validation_commands`/visual/Qwen/tool/report/schema/security 改动须对应测试覆盖。

## 常见问题

- 缺/空 `FORGIS_CONFIG.yml`；未知字段；非法路径。
- `dry_run=false` 但无 `confirm_real_run=true`。
- 缺 `model_env` secret。
- shell 字符串 build/test 命令（应改 YAML 数组）。
- 无有意义目标输出。
- guardrail 违规。
