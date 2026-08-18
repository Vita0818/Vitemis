# ARCHITECTURE.template.md

> 本模板定义 `docs/ARCHITECTURE.md` 应记录的内容。填充后删除本说明块。

# ARCHITECTURE

最近自查日期：{{YYYY-MM-DD}}

## 总体架构

用一段话和一张分层/数据流图说明项目的整体架构：

```text
{{ARCHITECTURE_DIAGRAM_OR_LAYERS}}
```

## 主要链路

逐条说明项目的核心数据流或业务链路。每条链路记录：

- 链路名称
- 起点（用户操作 / 系统事件 / 外部输入）
- 途经的关键类型与模块
- 终点（持久化 / UI 更新 / 外部调用）
- 关键不变量或约束

链路示例：

```text
{{LINK_NAME}}: {{ENTRY}} -> {{TYPE_A}} -> {{TYPE_B}} -> {{TYPE_C}} -> {{SINK}}
```

## 数据模型

列出核心数据模型 / 领域对象：

| 类型 | 职责 | 持久化方式 | 关键字段约束 |
|---|---|---|---|
| {{TYPE}} | {{RESPONSIBILITY}} | {{PERSISTENCE}} | {{CONSTRAINTS}} |

## 同步 / 通信机制（如适用）

如果项目涉及多端、多进程、多组件同步或通信，记录：

- 通信方式（本地 HTTPS / IPC / 文件 / 消息总线 / 网络）
- 协议与报文格式
- 状态机或会话模型
- 失败、重试、冲突处理策略

## 安全机制

列出项目中的安全边界与机制：

- 认证 / 配对 / 身份
- 加密 / 签名 / 校验（TLS、HMAC、nonce、指纹、pinning）
- 凭据存储（Keychain / 环境变量 / 配置）
- 权限边界（文件访问、能力租约、沙箱）
- 不可绕过的安全检查点

## 模式开关 / 内核切换（如适用）

如果项目存在新旧内核、模式开关或渐进式迁移，记录：

- 模式枚举与切换序列
- 各模式下的行为边界
- 默认/release 模式
- 回切路径与回退保证

## 与文档/源码的关系

本节描述架构结论的来源：哪些来自源码、哪些来自设计文档、哪些是推断。冲突时以源码为准。

## 外部依赖决策与禁止兜底

- 记录每项外部能力对应的 exact dependency、官方 API/扩展点、版本或固定 identity，以及许可证、provenance、安全、平台和分发约束。
- 有外部依赖提供同等能力时必须直接调用官方能力；架构中不得并存第一方重写、替代 adapter/shim/compatibility layer、parallel backend、preview backend、shadow implementation 或运行时 fallback。
- 本地边界只允许官方 API 必需的最薄生命周期、类型、权限、配置和 bundle 接线，不得复制依赖核心逻辑。
- 依赖无法接入时必须把能力标为 blocked 并明确失败；不得用 legacy、另一 provider/backend、mock、cache 或简化实现冒充完成。
- 安全 fail-closed 和明确要求的数据迁移必须与产品能力分开记录，不能演化成备用实现。
