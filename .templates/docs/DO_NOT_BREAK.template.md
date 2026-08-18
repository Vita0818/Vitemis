# DO_NOT_BREAK.template.md

> 本模板定义 `docs/DO_NOT_BREAK.md` 应记录的内容。填充后删除本说明块。

# DO_NOT_BREAK

本文列出不可破坏的工程禁区、数据格式、协议、路径和回归要求。修改前必须确认不违反下列任一条目。

## 工程禁区

- 不执行破坏性 Git 操作：`git reset --hard`、`git clean -fd`、`git checkout .`、强制 push、删除未提交文件。
- 未经用户明文要求具体 Git 操作，不 add、不 commit、不 push、不创建 PR；编辑、整理、修复、验证或准备工作都不等于提交请求。
- 若用户要求提交，只提交当前 Git root 中与本任务相关的文件；不得递归进入、暂存、提交或推送子仓库、submodule、nested Git repo 或依赖 checkout。
- 不引入新依赖，不改构建脚本，不改测试源码，除非任务明确要求。
- 不绕过安全机制（{{SECURITY_MECHANISMS}}）。

## 外部依赖与禁止兜底禁区

- 当外部依赖已经提供同等能力时，必须直接使用其官方 API/扩展点；不得自行重写，也不得新增替代 adapter、shim、compatibility layer、wrapper、proxy、facade、parallel backend、preview backend、shadow implementation 或临时 fallback。
- exact 依赖不可用或不兼容时必须明确失败并停止该能力；不得静默切换 legacy、另一 provider/backend、缓存、mock、简化实现或不完整路径。
- 本地代码只能是官方 API 必需的最薄生命周期、类型、权限、配置和 bundle 接线，不能复制或重新解释依赖核心行为。
- 现有 fallback 或重复实现不得扩张。安全 fail-closed 与明确要求的旧数据解码/迁移必须保持最窄范围，不能成为备用产品实现。

## 数据格式禁区

列出不可随意改动的数据格式：

- 持久化文件格式：{{FORMAT}}（路径、schema、版本）
- 配置文件格式：{{CONFIG_FORMAT}}
- 跨端/跨进程报文格式：{{WIRE_FORMAT}}
- 资源/资源包格式：{{ASSET_FORMAT}}

> 这些格式承载真实用户数据或跨端契约，改名、重排字段、改单位、改编码都可能造成数据丢失或对端不识别。

## 协议禁区

列出不可随意改动的协议：

- 通信协议：{{PROTOCOL}}（route、method、header、body schema、安全校验）
- 同步协议：{{SYNC_PROTOCOL}}（状态机、冲突规则、proof 规则）
- 迁移协议：{{MIGRATION_PROTOCOL}}（模式序列、回切、回退）

## 路径禁区

列出不可随意改动的路径约束：

- 文件根目录：{{FILE_ROOTS}}
- 安全范围书签 / App Group / 容器路径
- 资源打包路径
- 构建产物路径

## 回归要求

列出修改时必须保持的回归行为：

- {{REGRESSION_REQUIREMENT_1}}
- {{REGRESSION_REQUIREMENT_2}}

## 不可降级项

列出不能因重构、简化或"清理"而退化的行为：

- {{NON_REGRESSIVE_ITEM_1}}
- {{NON_REGRESSIVE_ITEM_2}}

## 验证要求

修改后必须运行哪些验证才能视为安全：

- {{VERIFICATION_COMMAND_1}}
- {{VERIFICATION_COMMAND_2}}
