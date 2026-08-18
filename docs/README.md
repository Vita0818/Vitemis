# Vitemis 通用 Agent 文档层

本目录用于存放 Vitemis 工作区级的通用 Agent 规范、文档模板和落地说明。

本层只放可复用规则，不维护具体项目清单、项目状态、技术栈、业务风险或仓库路径。新增、迁移、归档项目时，不应修改本层通用规范；只更新对应项目自己的 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md` 和项目内 `docs/`。

## 自动读取入口

- `/Users/vita/Vitemis/AGENTS.md`：Codex 可自动发现的 Vitemis Agent 通用规范，包含 Agent 角色、权限、报告目录、报告命名、报告模板、Git 边界和敏感信息禁区。
- 项目根 `CLAUDE.md`：Claude 可自动发现的只读审查入口。
- 项目根 `GEMINI.md`：Gemini CLI 可自动发现的只读审查入口。
- `VITEMIS_AGENT_POLICY.md`：兼容旧引用的入口文件，只负责指向 `/Users/vita/Vitemis/AGENTS.md`。
- `DEPENDENCY_POLICY.md`：所有第一方项目强制继承的 dependency-first / no-fallback 合同；外部依赖已有同等能力时必须直连官方 API，不得自行重写或增加替代适配层。

## 项目入口原则

每个项目自己的根入口文件负责项目内事实：

1. 必读顺序。
2. 工作目录检查。
3. 修改边界。
4. 禁止事项。
5. 项目理解要求。
6. 文档索引。
7. 完成标准。
8. 最终报告格式。
9. 已完成的持久性改动必须及时回写到相关项目文档；若无需更新文档，最终报告说明原因。
10. `docs/NEXT_TARGET.md` 作为临时下一目标记录；目标完成或不再有效后删除。
11. `docs/DEPENDENCY_POLICY.md` 的外部依赖优先、禁止功能兜底和禁止重复适配层规则必须写入项目 `AGENTS.md`、`ARCHITECTURE.md`、`DO_NOT_BREAK.md` 与 `TESTING.md`，或由项目提供同等严格的专属合同。

项目 `AGENTS.md` 可以引用 `/Users/vita/Vitemis/AGENTS.md`，但必须把项目专属内容留在项目内部。项目根 `CLAUDE.md` 和 `GEMINI.md` 必须明确只读审查边界，并分别限制写入 `claude-report/` 与 `gemini-report/`。

## 通用角色模型

- Codex：主工作者，可按用户任务和项目边界修改文件。
- Claude：审查副驾驶，只读审查，只写 `claude-report/`。
- Gemini：审查副驾驶，只读审查，只写 `gemini-report/`。
- Cursor：审查副驾驶，只读审查，只写 `cursor-report/`。

Git 版本控制默认由用户手动完成。编辑、整理文档、修复代码、运行验证或准备工作都不等于提交请求。Codex 只有在用户当前任务明文要求具体 Git 操作时，才可以执行对应的非破坏性 Git mutation；若用户要求提交，提交范围只限当前 Git root，不得递归进入、暂存、提交或推送子仓库、submodule、nested Git repo 或依赖 checkout。审查副驾驶始终保持只读。所有 Agent 都不得清理用户改动或重写历史。

报告文件统一写入对应报告目录，命名为 `MM_DD_YY-HH_MM-xxxx.md`，正文先写 `MODEL_CHECK_RESULT`，再写内容摘要、验证结果、`UNCERTAINTIES` 和下一步建议。

所有项目入口都应要求 Agent 在完成实现、修复、验证或文档维护后，及时把已完成的持久性改动回写到相关项目文档；若无需更新文档，最终报告必须说明原因。

所有项目 `docs/` 都应使用固定文件名 `NEXT_TARGET.md` 记录下一目标。该文件只保留一个 active target，目标完成或不再有效后删除。

## 新项目接入方式

新项目不需要改本目录。

建议流程：

1. 在新项目根目录创建 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md`。
2. 在新项目内部创建项目自己的 `docs/`，并准备临时目标记录文件 `docs/NEXT_TARGET.md`。
3. 在项目 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md` 中引用 `/Users/vita/Vitemis/AGENTS.md`。
4. 写清该项目自己的入口、目录、构建、测试、禁区和报告格式。
5. 若项目有特殊 Agent 模式或权限模型，只写在该项目内部。

## 模板

可复用模板位于 `/Users/vita/Vitemis/.templates/`。模板可用于创建项目级 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md` 和项目内 `docs/` 最小集。
