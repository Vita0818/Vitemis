# ARCHITECTURE

最近自查日期：2026-06-25

## 总体架构

Forgis 是本地代码迁移助手 + 受控文件工具运行时。三件事：读 `FORGIS_CONFIG.yml` + 任务文件；当开关允许时调 OpenAI 兼容文本模型；给模型受控文件交互工具。**不含**平台迁移智能（那在目标仓任务文件 + 可选 `skills/*.md`）。

```text
输入(target_repo) -> migrate.yml -> resolve_config.py -> forge.py 校验
  -> tool_loop.py (默认循环) -> file_tools.py 沙箱 -> 模型调用
  -> guardrails.py / validate_target_output.py 校验
  -> run_report.py / migration_plan_store.py / create_pr.sh
```

## 主要链路

### 默认 tool loop
```text
checkout -> resolve_config -> forge.py 校验 -> tool_loop
  -> file_tools 沙箱（虚拟路径，拒绝绝对/..//.git/secret/symlink/source 写入）
  -> 模型调用（非流式 Chat Completions）
  -> guardrails/validation/report/PR
```

### 分阶段翻译（staged_translation）
```text
overview -> per_file -> stabilization + 微阶段 gate
```

### GH Actions 真实运行
```text
real-run gate: dry_run=false + run_agent=true + confirm_real_run=true
  -> 失败分支 PR（${target_branch}-run-${GITHUB_RUN_ID}-${GITHUB_RUN_ATTEMPT}，无强推）
```

### Qwen Visual Evidence Mode
```text
reference-guided migration: 用户放源 App 截图到目标仓
  -> Qwen 只读截图输出视觉指导
  -> DeepSeek/主 Agent 才能改代码
```

## 数据模型

| 类型 | 职责 | 持久化 | 关键约束 |
|---|---|---|---|
| `ResolvedConfig` | 解析后配置 | 内存 | 真实运行 gate 三字段 |
| `VisualValidationConfig` | 视觉配置 | `FORGIS_CONFIG.yml` | 禁 secret/key/path |
| `StagedTranslationConfig` | 分阶段配置 | `FORGIS_CONFIG.yml` | — |
| `FileToolSandbox` | 文件工具沙箱 | 内存 | 虚拟路径，拒绝危险路径 |
| `ToolLoopResult` | 循环结果 | 内存 | — |
| `RuntimeController` | 运行时控制 | 内存 | 视觉状态/gate |
| `MigrationUnit`/`MigrationPlan` | 迁移单元/计划 | `FORGIS_MIGRATION_PLAN.json` | 写 v5.0，读 v4.8/v3.9/v3.8/v3.7 |
| `SourceUnit` | 源清单 | 内存 | — |
| `VisualEvidencePaths`/`VisualEvidenceSummary` | 视觉证据 | `visual-evidence/<run_id>/...` | — |
| `QwenVisionResult` | Qwen 结果 | 内存 | 无 key/headers/base64/bytes |
| `RunReportWriteResult` | 报告 | `FORGIS_RUN_REPORT.json` | schema `forgis.run_report.v6.0` |

## 安全机制

- **三重真实运行 gate**：`dry_run=false` + `run_agent=true` + `confirm_real_run=true`
- **source 只读**；**target_subdir 写入边界**；**config/task snapshot-hash 只读**
- `guardrails.py check-secret-leaks` 扫描 target_subdir
- `command_runner.py` allowlist + cwd-in-target_subdir；`run_build`/`run_tests` profile-restricted
- 文件工具虚拟路径沙箱：拒绝 absolute/`..`/`.git`/secret-like/symlink/source 写入/target-root 写入/workflow 写入
- `model_env` 仅 env 名映射，不存真实 secret
- Qwen 显式 key gate + mock-first 测试；无 key 返回 `QWEN_PERMISSION_GATED`
- `visual_validation` 拒绝 secret/key/path/未知字段
- 报告/PR body 脱敏 + 截断（30000 标准，3000 短）

## 模式开关 / 内核切换

无新旧内核开关。`dry_run`/`run_agent`/`confirm_real_run` 三开关控制真实运行；Qwen 视觉模式经 `visual_validation.enabled`(auto/true/false) 控制。

## 与文档/源码的关系

- 仓内 7 份 docs 均为描述性参考文档（非操作协议型）；操作协议在 `AGENTS.md`。
- `RELEASE_NOTES.md` 冻结 v5.0，与源码 v7.2 漂移（run report schema v5.0 vs v6.0）。以源码为准。
