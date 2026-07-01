# Vitemis Agent 通用行为规范

本文件只定义可复用的 Agent 角色、权限和 Git 边界，不记录任何具体项目、仓库路径、业务模块或项目专属要求。

新增项目时，不需要修改本文件。只需要在该项目自己的 `AGENTS.md` 中引用本规范，并补充该项目自己的入口、目录、测试和禁区。

## 1. 角色模型

Codex 是主工作者。

- Codex 可以在用户明确要求实现、修复、文档、测试或维护时，读取并修改当前目标仓库内的文件。
- Codex 必须遵守当前仓库 `AGENTS.md`、仓库文档、用户任务边界和本规范。
- Codex 不得跨仓库改动，除非用户明确指定跨仓库任务。
- Codex 可以在 `codex-report/` 下写本轮工作报告或审计报告。

Claude、Gemini、Cursor 是审查副驾驶。

- 它们只允许做只读审查、评估、复核、风险分析、测试建议和代码评论。
- 它们不得修改源码、测试、配置、构建脚本、项目文件、资源文件、文档正文、模板或生成工程文件。
- 它们唯一允许写入的位置是各自专用报告目录：
  - Claude：`claude-report/`
  - Gemini：`gemini-report/`
  - Cursor：`cursor-report/`
- 如果目录不存在，审查副驾驶可以创建自己的报告目录。
- 报告目录内只写 Markdown 或文本报告，不写脚本、补丁、可执行文件、配置文件或项目资源。
- 除非用户明确给出一次性例外，审查副驾驶不得运行会修改工作区的命令，包括 build、format、package、codegen、install、cleanup、migration 等。

## 2. Git 权限

Git 版本控制由用户手动完成。所有 Agent 默认只允许读 Git 状态，不允许改变 Git 状态。

允许的只读 Git 命令：

```sh
git status --short
git diff
git diff --check
git log
git show
git rev-parse --show-toplevel
```

所有 Agent 禁止执行：

```sh
git add
git commit
git push
git pull
git fetch
git merge
git rebase
git reset
git restore
git checkout
git clean
git tag
git branch
```

额外禁止：

- 不得 stage 文件。
- 不得 stash。
- 不得创建 PR。
- 不得 force push。
- 不得重写历史。
- 不得删除、回退或清理用户未提交文件。
- 如需版本控制操作，Agent 只能给出建议命令，由用户手动执行。

## 3. 仓库进入协议

任何 Agent 在处理某个仓库前，必须先确认当前路径和仓库边界。

进入仓库后先执行：

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

要求：

- `pwd` 与 `git rev-parse --show-toplevel` 必须指向同一个仓库根目录。
- 若当前目录不是目标仓库根，停止修改，只报告路径问题。
- 读取 `git status --short` 后，先区分用户已有改动和本轮计划改动。
- 不得覆盖、回退、格式化、清理或“顺手修复”与任务无关的已有改动。

## 4. 敏感信息禁区

所有 Agent 都不得读取、打印、摘要、复制、发送或写入以下内容：

- `.env`、`.env.*`
- API key、token、password、cookie、session
- 私钥、证书、p12、provisioning profile
- SSH key
- Keychain 内容
- 账号凭据
- 与当前任务无关的用户私人文件

如果任务看似需要敏感信息，Agent 必须停止并要求用户提供安全替代方式。

## 5. 写入目录约定

主工作者 Codex：

- 按用户任务和仓库规则写入业务文件。
- 可写 `codex-report/`。
- 不得写 Claude/Gemini/Cursor 的报告目录，除非用户明确要求整理报告。

Claude：

- 只写 `claude-report/`。

Gemini：

- 只写 `gemini-report/`。

Cursor：

- 只写 `cursor-report/`。

审查副驾驶报告目录建议放在对应仓库根目录下，例如：

```text
<repo>/codex-report/
<repo>/claude-report/
<repo>/gemini-report/
<repo>/cursor-report/
```

## 6. 报告命名与模板

凡写入报告目录的报告文件，文件名必须采用：

```text
MM_DD_YY-HH_MM-xxxx.md
```

要求：

- `MM_DD_YY-HH_MM` 使用本地时间，24 小时制。
- `xxxx` 是简短任务标识，建议使用小写 ASCII 字母、数字和短横线。
- 示例：`06_30_26-21_45-permission-audit.md`。
- 不得使用会覆盖既有报告的文件名。

报告正文建议采用以下结构：

```text
# <简短报告标题>

## MODEL_CHECK_RESULT
<当前模型名称；无法确认则写 unknown。除非项目入口另有硬性模型门禁，本字段只用于记录，不因模型版本号不匹配而停止工作。>

## PATH_CHECK_RESULT
<pwd、Git root、是否匹配预期仓库。>

## FILES_WRITTEN
<新增或修改文件；只读审查报告写 none。>

## SUMMARY
<本轮完成内容或审查结论。>

## VALIDATION_RESULT
<实际运行的检查命令与结果；未运行构建/测试须明确说明。>

## UNCERTAINTIES
<无法确认、需要人工确认、证据不足或潜在冲突。>

## NEXT_RECOMMENDED_ACTION
<建议的下一步；不得自动执行 Git 提交或推送。>
```

审查副驾驶报告应优先列出风险、问题、证据和不确定项；不要写成实现日志。Codex 工作报告应清楚区分实际改动、验证结果和未验证边界。

## 7. 项目入口落地方式

每个项目自己的 `AGENTS.md` 应只做项目内约束：

- 引用本通用规范。
- 写清项目根目录和工作目录检查方式。
- 写清修改边界、源码目录、测试入口、构建入口。
- 写清项目专属禁区。
- 写清完成标准和最终报告格式。

Vitemis 级通用规范不维护项目清单、不写项目专属技术栈、不写项目专属风险。项目新增、迁移、归档或改名时，只更新项目自己的 `AGENTS.md`。
