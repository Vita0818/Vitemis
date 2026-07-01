# Vitemis 通用 Agent 文档层

本目录用于存放 Vitemis 工作区级的通用 Agent 规范、文档模板和落地说明。

本层只放可复用规则，不维护具体项目清单、项目状态、技术栈、业务风险或仓库路径。新增、迁移、归档项目时，不应修改本层通用规范；只更新对应项目自己的 `AGENTS.md` 和项目内 `docs/`。

## 通用规范

- `VITEMIS_AGENT_POLICY.md`：Agent 角色、权限、报告目录、报告命名、报告模板、Git 禁区、敏感信息禁区。

## 项目入口原则

每个项目自己的 `AGENTS.md` 负责项目内事实：

1. 必读顺序。
2. 工作目录检查。
3. 修改边界。
4. 禁止事项。
5. 项目理解要求。
6. 文档索引。
7. 完成标准。
8. 最终报告格式。

项目 `AGENTS.md` 可以引用本层通用规范，但必须把项目专属内容留在项目内部。

## 通用角色模型

- Codex：主工作者，可按用户任务和项目边界修改文件。
- Claude：审查副驾驶，只读审查，只写 `claude-report/`。
- Gemini：审查副驾驶，只读审查，只写 `gemini-report/`。
- Cursor：审查副驾驶，只读审查，只写 `cursor-report/`。

Git 版本控制由用户手动完成。Agent 不提交、不推送、不 stage、不清理、不重写历史。

报告文件统一写入对应报告目录，命名为 `MM_DD_YY-HH_MM-xxxx.md`，正文先写 `MODEL_CHECK_RESULT`，再写内容摘要、验证结果、`UNCERTAINTIES` 和下一步建议。

## 新项目接入方式

新项目不需要改本目录。

建议流程：

1. 在新项目根目录创建 `AGENTS.md`。
2. 在新项目内部创建项目自己的 `docs/`。
3. 在项目 `AGENTS.md` 中引用 `VITEMIS_AGENT_POLICY.md`。
4. 写清该项目自己的入口、目录、构建、测试、禁区和报告格式。
5. 若项目有特殊 Agent 模式或权限模型，只写在该项目内部。

## 模板

可复用模板位于 `/Users/vita/Vitemis/.templates/`。模板可用于创建项目级 `AGENTS.md` 和项目内 `docs/` 最小集。
