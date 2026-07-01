# PROJECT_MAP.template.md

> 本模板定义 `docs/PROJECT_MAP.md` 应记录的内容。填充后删除本说明块。

# PROJECT_MAP

最近自查日期：{{YYYY-MM-DD}}

本文描述当前仓库结构。判断依据来自 {{BUILD_MANIFEST_TYPES}}（如 Xcode project / Package.swift / project.yml / Makefile）、源码、测试文件和脚本。

## 目录结构总览

用树形或列表展示顶层目录与关键子目录，标注每个目录的职责：

```text
{{PROJECT_ROOT}}/
├── {{DIR}}/        # 职责说明
├── {{DIR}}/        # 职责说明
└── ...
```

## Target / 模块

逐项列出工程的 target、module 或产品：

| Target / 模块 | 类型 | 平台 | 入口 | 职责 |
|---|---|---|---|---|
| {{TARGET}} | app/lib/test | {{PLATFORM}} | {{ENTRY}} | {{RESPONSIBILITY}} |

## 关键文件

列出需要重点理解的源码文件，按链路或职责分组：

- 入口：{{ENTRY_FILES}}
- 核心链路：{{KEY_FILES}}
- 配置：{{CONFIG_FILES}}
- 测试：{{TEST_FILES}}

## 生成物 / 产物

列出构建、脚本或工作流会产生的文件或目录：

- 构建产物：{{BUILD_OUTPUTS}}
- 脚本生成物：{{SCRIPT_OUTPUTS}}
- 报告：{{REPORT_OUTPUTS}}

## 脚本与工具

列出 `Scripts/` 或等价目录下的脚本及用途：

| 脚本 | 用途 | 调用方式 |
|---|---|---|
| {{SCRIPT}} | {{PURPOSE}} | {{USAGE}} |

## 不确定项

无法从源码确认的目录或文件，标注 `UNKNOWN`，不要编造。
