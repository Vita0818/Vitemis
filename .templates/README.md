# Vitemis 项目文档模板

本目录提供可复用的项目文档模板，供 Vitemis 下各新项目套用。模板提炼自最成熟项目 Rokurics 的操作协议结构，并参考 Forgis / Lybris / Kikaria 的对齐版本。

## 为什么要模板

Vitemis 下有多个项目，成熟度不一。统一"AI Agent 进入仓库时检查什么、读什么、报告按什么格式"的协议，可以：

- 让任何 Agent 在任何项目里有可预测的入口与边界。
- 避免每个项目从零重写常驻上下文。
- 让项目文档有可对照的基线，便于审计与对齐。
- 让所有新项目默认继承 dependency-first / no-fallback 规则，禁止在已有外部能力时重复实现或增加替代适配层。

## 目录结构

```text
.templates/
├── README.md                        # 本文件
├── AGENTS.template.md               # AGENTS.md 8 段骨架 + 占位符
├── CLAUDE.template.md               # Claude 根入口，只读审查副驾驶
├── GEMINI.template.md               # Gemini 根入口，只读审查副驾驶
└── docs/
    ├── CURRENT_STATE.template.md
    ├── PROJECT_MAP.template.md
    ├── ARCHITECTURE.template.md
    ├── DO_NOT_BREAK.template.md
    └── TESTING.template.md
```

## 套用步骤

1. 复制 `AGENTS.template.md` 到目标项目根，改名为 `AGENTS.md`。
2. 复制 `CLAUDE.template.md`、`GEMINI.template.md` 到目标项目根，分别改名为 `CLAUDE.md`、`GEMINI.md`。
3. 复制 `docs/` 下 5 份模板到目标项目的 `docs/`，去掉 `.template` 后缀。
4. 逐文件填充：把 `{{占位符}}` 替换为项目实际内容；删除"本模板说明"引导块。
5. 凡无法从源码确认的内容，标注 `UNKNOWN` 或 `需要后续确认`，不要编造。
6. 文档与源码冲突时，以源码为准，并在 `CURRENT_STATE.md` 的"文档与源码冲突"段记录。

## AGENTS.md 标准结构（8 段）

1. **必读顺序**：改任何代码/配置/脚本/测试前必读的 docs 列表；冲突时以源码为准。
2. **工作目录检查**：`pwd` / `git rev-parse --show-toplevel` / `git status --short`，必须同根。
3. **修改边界**：文档任务只能改 `AGENTS.md` + `docs/`；源码目录白名单。
4. **禁止事项**：破坏性 Git / 未经用户明文要求的 add、commit、push / 跨子仓库提交 / 引入依赖 / 写密钥 / 绕安全机制。
5. **下一目标**：`docs/NEXT_TARGET.md` 用于临时记录下一目标；目标完成或不再有效后删除。
5. **项目理解要求**：入口文件 + 关键类型/链路清单。
6. **文档索引**：每个 docs 文件一句话说明。
7. **完成标准**：读过什么 / 只改范围内文件 / 跑相称检查 / 未跑构建须声明。
8. **最终报告格式**：8 字段（MODEL_CHECK_RESULT / PATH_CHECK_RESULT / FILES_WRITTEN / PROJECT_AUDIT_SUMMARY / DOCS_CONTENT_SUMMARY / VALIDATION_RESULT / UNCERTAINTIES / NEXT_RECOMMENDED_ACTION）。
9. **外部依赖优先与禁止兜底**：继承 `/Users/vita/Vitemis/docs/DEPENDENCY_POLICY.md`；直连官方 API，依赖不可接入时明确阻断，不新增替代 adapter/shim/backend/fallback。

## docs/ 最小集（5 份）

| 文档 | 作用 |
|---|---|
| `CURRENT_STATE.md` | 当前真实状态、已有能力、风险、工作区改动、文档与源码冲突 |
| `PROJECT_MAP.md` | 目录、target、入口、关键文件、生成物、脚本地图 |
| `ARCHITECTURE.md` | 总体架构、主要链路、数据模型、同步/安全机制、模式开关 |
| `DO_NOT_BREAK.md` | 工程禁区、数据格式、协议、路径、回归要求、不可降级项 |
| `TESTING.md` | 环境、构建、测试、lint/format、手动验证矩阵、验证边界声明 |

> 项目可在最小集之上按需增加专属文档（如 Rokurics 的 `SYNC_STATE_AUDIT.md`）。在 AGENTS.md 的"必读顺序"和"文档索引"中追加引用即可。

## 占位符清单

| 占位符 | 含义 |
|---|---|
| `{{PROJECT_NAME}}` | 项目名称 |
| `{{PROJECT_TYPE_DESCRIPTION}}` | 项目类型描述（如"Swift/Xcode 多 target 工程"） |
| `{{GIT_ROOT}}` | 仓库根目录绝对路径 |
| `{{EXTRA_REQUIRED_DOCS_LIST}}` | 额外必读文档列表（无则留空） |
| `{{SOURCE_TREE_NO_TOUCH_LIST}}` | 未经允许不得修改的源码目录列表 |
| `{{PROJECT_SPECIFIC_FORBIDDEN_RULES}}` | 项目专属禁止规则（无则留空） |
| `{{ENTRY_FILES_LIST}}` | 入口文件清单 |
| `{{KEY_LINKS_AND_CHAINS}}` | 关键链路清单 |
| `{{EXTRA_DOCS_INDEX}}` | 额外文档索引项（无则留空） |
| `{{YYYY-MM-DD}}` | 自查日期 |

各 `docs/*.template.md` 内还有针对性占位符，见各文件说明。

## 裁剪指南

- **新项目 / 草稿项目**：5 份 docs 都建，内容可精简但结构齐全，不确定项标 `UNKNOWN`。
- **成熟项目**：5 份 docs 内容应详尽；可按需追加项目专属文档。
- **特例项目（如调度器、模板站）**：8 段 AGENTS.md 结构保留，docs 内容按项目性质裁剪，但至少保留 5 份最小集。
