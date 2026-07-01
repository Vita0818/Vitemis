# {{PROJECT_NAME}} 项目常驻上下文

本文是 AI Agent 每轮进入本仓库时的入口文件。执行任何代码修改、配置修改、构建脚本修改或测试源码修改之前，必须先按顺序阅读并核对下列文档：

1. `docs/CURRENT_STATE.md`
2. `docs/PROJECT_MAP.md`
3. `docs/ARCHITECTURE.md`
4. `docs/DO_NOT_BREAK.md`
5. `docs/TESTING.md`
{{EXTRA_REQUIRED_DOCS_LIST}}

如果文档与源码、工程配置、测试或脚本冲突，必须以当前源码和配置为准，并在最终报告中明确指出冲突位置和采用源码为准的原因。

## 工作目录检查

每轮开始先在项目根目录执行：

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

要求：

- `pwd` 与 `git rev-parse --show-toplevel` 必须指向同一个仓库根目录：`{{GIT_ROOT}}`。
- 如果当前目录不是 Git root，停止修改，只报告路径问题。
- 读取 `git status --short` 后，先区分用户已有改动与本轮计划改动；不得覆盖、回退或清理用户已有改动。

## 修改边界

本仓库是 {{PROJECT_TYPE_DESCRIPTION}}。

未来常规任务可以按用户要求修改业务源码；但在只要求项目自查或文档更新的任务中，只允许修改：

- `AGENTS.md`
- `docs/` 下的项目说明文档

除非用户明确要求，不要修改：

{{SOURCE_TREE_NO_TOUCH_LIST}}

## 禁止事项

- 不执行破坏性 Git 操作：`git reset --hard`、`git clean -fd`、`git checkout .`、强制 push、删除用户未提交文件。
- 未经用户明确要求，不 commit、不 push、不创建 PR。
- 不引入新依赖，不改构建脚本，不改测试源码，除非任务明确要求。
- 不把密钥、token、证书私钥、shared secret、账号密码、完整指纹、完整 API 响应、完整转写文本或个人隐私路径写入文档。
{{PROJECT_SPECIFIC_FORBIDDEN_RULES}}

## 项目理解要求

修改前至少确认：

- target 和入口：{{ENTRY_FILES_LIST}}。
{{KEY_LINKS_AND_CHAINS}}

不确定的模块必须标注 `UNKNOWN` 或 `需要后续确认`，不要编造。

## 文档索引

- `docs/PROJECT_MAP.md`：目录、target、入口、关键文件和生成物地图。
- `docs/ARCHITECTURE.md`：总体架构、主要链路、数据模型和安全机制。
- `docs/CURRENT_STATE.md`：当前真实状态、已有能力、风险、工作区改动。
- `docs/TESTING.md`：环境、构建、测试、lint/format 与手动验证方式。
- `docs/DO_NOT_BREAK.md`：工程禁区、数据格式、协议、路径和回归要求。
{{EXTRA_DOCS_INDEX}}

## 完成标准

完成任务前至少做到：

- 说明本轮实际阅读/检查过哪些源码、配置或测试。
- 只修改任务范围内文件。
- 保留用户已有改动。
- 运行与任务相称的检查；文档任务至少运行 `git diff --check` 与 `git status --short`。
- 如未运行构建或测试，最终报告必须明确写"未运行构建/测试"。

## 最终报告格式

最终报告建议包含：

1. `MODEL_CHECK_RESULT`：当前模型名称；无法确认时写无法确认。
2. `PATH_CHECK_RESULT`：`pwd`、Git root、是否匹配预期。
3. `FILES_WRITTEN`：新增/修改文件。
4. `PROJECT_AUDIT_SUMMARY`：识别到的项目结构、主要模块和关键链路。
5. `DOCS_CONTENT_SUMMARY`：各文档内容摘要。
6. `VALIDATION_RESULT`：实际运行命令与结果。
7. `UNCERTAINTIES`：无法确认、需要人工确认的点。
8. `NEXT_RECOMMENDED_ACTION`：下一步建议；不要自动继续改业务源码。
