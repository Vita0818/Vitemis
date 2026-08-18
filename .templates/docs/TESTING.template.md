# TESTING.template.md

> 本模板定义 `docs/TESTING.md` 应记录的内容。填充后删除本说明块。

# TESTING

最近自查日期：{{YYYY-MM-DD}}

## 环境

列出构建与测试所需的环境：

- 操作系统 / 平台：{{OS_PLATFORM}}
- 工具链版本：{{TOOLCHAIN_VERSION}}（Xcode / Swift / Python / Node 等）
- 依赖管理：{{DEPENDENCY_MANAGER}}
- 凭据 / 配置：{{REQUIRED_CREDENTIALS_OR_CONFIG}}

## 构建

列出构建命令：

```sh
{{BUILD_COMMAND}}
```

- 配置：Debug / Release
- 目标：{{TARGETS}}
- 产物位置：{{BUILD_OUTPUT}}

## 测试

列出测试命令：

```sh
{{TEST_COMMAND}}
```

- 单元测试：{{UNIT_TEST_TARGETS}}
- UI 测试：{{UI_TEST_TARGETS}}
- 集成测试：{{INTEGRATION_TEST_TARGETS}}
- 只跑特定测试：{{ONLY_TESTING_EXAMPLE}}

## Lint / Format

列出静态检查命令：

```sh
{{LINT_COMMAND}}
{{FORMAT_COMMAND}}
```

## 手动验证矩阵

列出无法用自动化测试覆盖、需要人工验证的场景：

| 场景 | 步骤 | 预期 | 状态 |
|---|---|---|---|
| {{SCENARIO}} | {{STEPS}} | {{EXPECTED}} | {{STATUS}} |

## 验证边界声明

明确哪些验证本轮实际运行过、哪些未运行：

- 文档任务：至少运行 `git diff --check` 与 `git status --short`；通常**未运行构建/测试**，须在最终报告中声明。
- 代码任务：按改动风险运行相称的 build / test / lint；未运行时须声明原因。

## 外部依赖与禁止兜底验证

- 验证 exact 外部依赖可用时只调用其官方 API/扩展点，不调用第一方重复实现。
- 验证依赖缺失、版本不兼容、构建/签名/许可证/平台/安全条件不成立时产生明确、可诊断失败并停止该能力。
- 验证失败路径不会切换到 legacy、另一 provider/backend、adapter/shim、cache、mock、简化实现或不完整路径。
- 测试 double 只能存在于测试 target，不得进入 production selection 或 runtime fallback。
- Review 必须检查新增 wrapper/adapter/facade 是否仅为官方 API 必需的最薄接线；任何核心能力复制都必须拒绝。

## 常见问题

- 真机/模拟器/特定平台缺失时的降级验证方式。
- 测试基础设施失败（如模拟器启动超时）时的处理。
