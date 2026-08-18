# macOS 应用图标完整流程

## 目录

1. [目标与职责](#1-目标与职责)
2. [资料优先级与硬性边界](#2-资料优先级与硬性边界)
3. [工作目录与版本策略](#3-工作目录与版本策略)
4. [阶段一：环境预检](#4-阶段一环境预检)
5. [阶段二：最小需求简报](#5-阶段二最小需求简报)
6. [阶段三：原创概念与选择](#6-阶段三原创概念与选择)
7. [阶段四：建立可复现规格](#7-阶段四建立可复现规格)
8. [阶段五：通过 Illustrator 接口构建](#8-阶段五通过-illustrator-接口构建)
9. [阶段六：导出和检查分层素材](#9-阶段六导出和检查分层素材)
10. [阶段七：视觉迭代](#10-阶段七视觉迭代)
11. [阶段八：Icon Composer 组装](#11-阶段八icon-composer-组装)
12. [阶段九：Xcode 接入](#12-阶段九xcode-接入)
13. [阶段十：成品验收](#13-阶段十成品验收)
14. [交付结构](#14-交付结构)
15. [常见故障与处理](#15-常见故障与处理)
16. [完成定义](#16-完成定义)
17. [权威资料](#17-权威资料)

## 1. 目标与职责

把“用户说应用是什么”转化为一套可追溯、可重建、可在 Apple 平台正确渲染的应用图标资产。默认由 Codex 完成概念提案、Illustrator 矢量构建、分层导出、预览检查、Icon Composer 组装建议和 Xcode 接入验证；用户只承担两类高价值决策：

1. 确认图标要表达的核心含义。
2. 从少量方向中选择或评价视觉结果。

不要要求没有设计经验的用户学习 Illustrator、理解贝塞尔曲线、手动整理图层或自行排查 SVG。

本流程区分三类资产：

- **App icon**：应用身份和品牌入口，本流程的主要对象。
- **Interface icon**：工具栏、菜单、按钮等应用内功能图标，适合使用 SF Symbols。
- **Marketing artwork**：官网、商店宣传和文档中的扁平导出，可从 Icon Composer 或源文件派生，但不能替代可编辑源。

## 2. 资料优先级与硬性边界

### 2.1 资料优先级

遇到冲突时按以下顺序处理：

1. 当前 Apple Human Interface Guidelines、Icon Composer/Xcode 文档和 SF Symbols 使用条款。
2. 当前 Adobe Illustrator 官方脚本文档和本机 Illustrator 脚本字典。
3. 当前目标仓库的 `AGENTS.md`、构建配置、已有 `.icon` 和项目文档。
4. 本流程中的默认值。

Apple 和 Adobe 工具会更新。涉及许可、画布、外观模式、导出兼容性或构建规则时，先核验当前官方文档，不把本文档的版本性细节当作永久事实。

### 2.2 SF Symbols 边界

Apple 明确禁止在 app icon、logo 或其他商标用途里使用 SF Symbols，或使用与其容易混淆的图像。因此：

- 不把 SF Symbol 导入 Illustrator 作为应用图标图层。
- 不描摹、扩展轮廓、扭曲、旋转、拼接或轻微修改 SF Symbol 后用于应用图标。
- 不以“只是灵感”为理由保留明显相同的主轮廓。
- 可在同一项目中使用 SF Symbols 设计应用内界面图标，但将该工作作为独立任务和独立资产处理。
- 若概念与某个常见 SF Symbol 接近，重新寻找更具体的产品隐喻、不同负空间或不同物体关系，直到整体轮廓具有原创识别度。

### 2.3 原创与品牌边界

- 不描摹竞品应用图标、第三方 logo、版权插画或未经授权的角色形象。
- 参考竞品只用于检查品类惯例和避免撞形，不用于复刻构图。
- 若用户提供品牌资产，先确认其有权使用，再将其视为输入，不擅自重绘成近似商标。
- 对可能产生混淆的结果明确提示，不假定技术可导出就代表许可安全。

### 2.4 文件与自动化边界

- 每个版本创建新的 Illustrator 文档和版本目录。
- 不修改用户当前打开的 Illustrator 文档。
- 不覆盖旧版本，除非用户明确指定目标并确认覆盖。
- 使用绝对路径；执行前解析并确认输入、输出和目标应用。
- 发送 Apple Events 前请求系统授权；不绕过 macOS Automation 权限。
- Icon Composer 没有已确认的公共脚本接口时，只使用最少量 GUI 操作，并在操作前说明。
- 不把“脚本执行成功”视为“图标视觉通过”。

## 3. 工作目录与版本策略

优先在目标应用仓库内使用明确的设计资产目录；若仓库已有约定，沿用约定。没有约定时使用：

```text
design/app-icon/
├── brief.md
├── icon-spec.json
├── source/
│   ├── app-icon-v001.ai
│   └── build-icon-v001.jsx
├── layers/
│   ├── 01-background.svg
│   ├── 02-supporting-shape.svg
│   └── 03-primary-symbol.svg
├── previews/
│   ├── app-icon-v001-1024.png
│   └── app-icon-v001-contact-sheet.png
├── AppName.icon/
└── review.md
```

规则：

- 版本使用 `v001`、`v002` 递增，不用 `final-final-2`。
- `source/` 是可编辑真源；`layers/` 和 `previews/` 是派生物。
- `.icon` 是 Icon Composer/Xcode 真源，不把单张 PNG 当作其替代品。
- 图层文件名以两位数字开头，并按后到前排列。
- 写清哪个版本进入产品构建；保留被否决方向的预览，不必保留所有临时脚本输出。

## 4. 阶段一：环境预检

在设计前做只读检查：

1. 确认目标仓库根、工作区规则和现有未提交改动。
2. 搜索已有 `.icon`、`.icns`、`Assets.xcassets`、`ASSETCATALOG_COMPILER_APPICON_NAME` 和图标相关项目文档。
3. 确认 Adobe Illustrator 的安装路径和版本。
4. 确认 `osascript` 可用，并检查 Illustrator 的 AppleScript 字典是否提供 `do javascript`。
5. 确认 Icon Composer 和目标 Xcode 版本可用。
6. 若只做概念探索，不启动 Illustrator；先完成黑白方向。

不要在预检阶段：

- 打开或修改用户现有 Illustrator 文档；
- 更改 Xcode 构建设置；
- 安装软件、升级订阅或接受付费项目；
- 删除旧图标资产；
- 发送 Apple Events。

预检输出应说明：可用能力、缺失能力、需要用户批准的动作，以及本轮是否只做到设计素材而不接入工程。

## 5. 阶段二：最小需求简报

### 5.1 优先从仓库推断

先读取应用名称、README、产品描述、主界面和已有品牌颜色。能够可靠推断的内容不重复询问用户。

### 5.2 必需信息

将简报压缩为：

- 应用名称。
- 一句话功能。
- 主要用户。
- 最希望被记住的一个含义。
- 三个气质词，例如“安静、精确、轻盈”。
- 明确不要出现的物体、颜色或风格。
- 仅 macOS，还是需要 iPhone、iPad、Apple Watch 共用。

如果缺失信息会导致完全不同的主隐喻，只问一个问题，例如：“它首先应该让人想到转写，还是想到隐私？”不要一次发出长问卷。

### 5.3 写成设计判据

把主观描述转换为可验证句子：

- “轻盈”转成：大面积留白、少于四个深度组、没有厚重外框。
- “专业”转成：稳定轴线、受控色数、低装饰噪声。
- “快速”转成：明确方向性或节奏，但避免通用闪电图形。
- “隐私”转成：通过遮蔽、局部封闭或边界关系表达，不直接复制系统锁形 glyph。

## 6. 阶段三：原创概念与选择

### 6.1 先做单一主隐喻

每个方向只保留一个可在一秒内说清的主隐喻。允许一个辅助关系，不堆叠功能清单。

优先寻找：

- 产品中的独有动作；
- 输入与输出之间的关系；
- 用户获得的结果；
- 应用名称中可转化的物理意象；
- 通过负空间形成的第二含义。

避免默认使用齿轮、火箭、闪电、魔杖、对话框、锁、云朵等高频图形；只有产品语义特别强且轮廓足够原创时才采用。

### 6.2 提出三个黑白方向

每个方向包含：

1. 一句主隐喻。
2. 黑白轮廓描述。
3. 预计的 2–4 个深度组。
4. 32 px 的主要风险。
5. 与常见竞品或系统 glyph 的区分点。

不要在这一阶段讨论玻璃高光、阴影或渐变。先验证轮廓和负空间。

### 6.3 选择门槛

把三个方向在 32 px 和 16 px 的纯黑轮廓中比较。推荐满足以下条件的方向：

- 32 px 一眼可辨，16 px 仍保留主要质量块。
- 不依赖细线、微小孔洞或文字。
- 单色时仍有独特身份。
- 不容易被误认成 SF Symbol、竞品图标或通用模板。
- 能合理拆成不超过四个深度组。

让用户只做一次高价值选择。用户不确定时，直接推荐风险最低且扩展性最高的方向。

## 7. 阶段四：建立可复现规格

选择方向后写入 `icon-spec.json`。规格至少包含：

```json
{
  "name": "AppName",
  "version": "v001",
  "canvas": { "width": 1024, "height": 1024, "colorMode": "RGB" },
  "concept": {
    "metaphor": "一句主隐喻",
    "personality": ["calm", "precise", "light"],
    "avoid": ["text", "thin strokes", "SF Symbols", "premasked corners"]
  },
  "safeArea": { "visualInset": 96 },
  "palette": [
    { "name": "primary", "hex": "#4D6BFF" },
    { "name": "secondary", "hex": "#D7E0FF" }
  ],
  "layers": [
    { "order": 1, "name": "background", "role": "support" },
    { "order": 2, "name": "primary-form", "role": "identity" },
    { "order": 3, "name": "accent", "role": "focus" }
  ],
  "exports": {
    "layerFormat": "svg",
    "previewSizes": [1024, 512, 256, 128, 64, 32, 16]
  }
}
```

`visualInset` 是工作起点而非 Apple 永久规则。使用当前 Apple Design Resources 模板核对网格和视觉安全区。

规格中使用数值表达：

- 中心、边界、半径、角度、线宽；
- 布尔运算关系；
- 填充、描边和透明度；
- 图层顺序和对象名称；
- 对称与非对称的意图；
- 哪些属性留到 Icon Composer 设置。

## 8. 阶段五：通过 Illustrator 接口构建

### 8.1 为什么使用接口

Adobe 官方支持 JavaScript/ExtendScript 和 AppleScript，并允许通过 API 自动生成、选择、编辑和组织图稿。优先使用脚本接口，以获得可重复的几何、稳定图层名和一致导出；不要用鼠标录制来代替确定性构建。

### 8.2 脚本生成规则

为当前版本生成独立 JSX，并满足：

- 创建新文档，不依赖活动文档。
- 使用 1024 x 1024 RGB 画板。
- 明确设置画板和坐标转换，不把屏幕向下的 Y 轴与 Illustrator 坐标混用。
- 建立编号图层，例如 `01-background`、`02-primary-form`、`03-accent`。
- 给重要对象稳定命名，便于后续精确修改。
- 颜色以明确 RGB 值写入。
- 对复合图形使用路径和布尔结果；最终导出前展开需要展开的外观。
- 保存 `.ai` 源文件，再导出派生物。
- 默认拒绝覆盖已存在文件。
- 捕获错误并返回阶段、对象名和错误信息。

不在通用流程里硬编码某一个图标的 JSX。每个图标从 `icon-spec.json` 生成自己的构建脚本；当多次出现稳定重复逻辑后，再把经过验证的部分提取为共享脚本。

### 8.3 执行方式

在 macOS 上使用 Illustrator AppleScript 的 `do javascript`，传入 JSX 的绝对路径：

```sh
osascript -e 'tell application id "com.adobe.illustrator" to do javascript (POSIX file "/absolute/path/build-icon-v001.jsx")'
```

执行前：

1. 确认目标脚本路径和输出目录。
2. 告知用户将向 Illustrator 发送自动化指令。
3. 请求环境所需的升级权限。
4. 若出现 macOS Automation 提示，让用户确认，不尝试绕过。

执行后：

1. 检查命令返回和 Illustrator 错误。
2. 检查 `.ai`、SVG 和 PNG 是否真实生成且时间戳属于本轮。
3. 不因为文件存在就判定视觉正确。
4. 若 Illustrator 停留在对话框，不继续盲目发送脚本；先识别并报告对话框。

Illustrator 是可脚本控制的桌面进程，不是真正无界面的渲染服务。脚本路线是接口调用，不等于完全 headless。

## 9. 阶段六：导出和检查分层素材

### 9.1 分层原则

Apple 建议按 Z 轴从后到前设计，并在 Icon Composer 内最多整理为四个组。每组应表达一个深度层，而不是把每个小形状都变成组。

典型结构：

```text
01-background       环境色或大面积支撑
02-secondary-form   次要物体或关系
03-primary-form     核心身份轮廓
04-accent           小而关键的焦点（仅在必要时）
```

### 9.2 导出规则

- 每层导出完整 1024 x 1024 画板，而不是导出该对象的紧边界。
- 所有 SVG 使用相同坐标原点、宽高和 `viewBox="0 0 1024 1024"`。
- 使用透明画布，不包含最终平台遮罩。
- 优先 SVG；只对不受支持的渐变、栅格内容或无法可靠表达的效果使用透明 PNG。
- 文字转轮廓；不依赖外部字体。
- 不保留链接图片、外部 CSS、外部滤镜或网络资源。
- 不在导出层里烘焙 blur、shadow、specular、refraction、translucency 和平台背景渐变。
- 文件名编号按后到前递增，以便 Icon Composer 按字母顺序导入。

导出脚本应临时隐藏其他图层、导出完整画板，并在每次导出后恢复原始可见性和选择状态。

### 9.3 结构预检

使用可用的 XML/SVG 工具检查：

- 文件可解析；
- `viewBox` 一致；
- 图层非空；
- 没有文字节点；
- 没有外部引用；
- 不含意外位图；
- 路径没有超出画布到破坏裁切的程度；
- 文件名和顺序与规格一致。

结构预检只能发现技术问题，不能取代视觉检查。

## 10. 阶段七：视觉迭代

### 10.1 必须实际渲染

每次有意义的改动都导出：

- 1024 px 单图，用于观察边缘、曲线和颜色；
- 512、256、128、64、32、16 px 尺寸；
- 浅色、深色和中性背景接触表；
- 必要时加入 Dock、Finder 或 App Switcher 的上下文模拟。

使用图像查看能力检查实际像素。不要只读取 SVG 文本或 Illustrator 对象属性。

### 10.2 审查顺序

按以下顺序检查，避免先纠结细节：

1. **身份**：一秒内是否知道它代表什么或至少记得它？
2. **轮廓**：32 px 是否仍可辨？
3. **视觉重心**：是否看起来居中，而不只是数学居中？
4. **层级**：主形是否明显压过辅助形？
5. **负空间**：缩小后孔洞是否闭合、间隙是否粘连？
6. **颜色**：明暗和色盲条件下是否仍有结构？
7. **原创性**：是否像某个 SF Symbol、竞品或图标模板？
8. **一致性**：曲率、角度、端点和重复元素是否统一？

### 10.3 反馈方式

向用户提供：

- 当前推荐版本；
- 一句为什么；
- 最重要的一个弱点；
- 一个聚焦选择，例如“更安静，还是更有速度感？”

避免要求用户逐项填写颜色、线宽、圆角和坐标。用户说“感觉太硬”时，由 Codex 把它转换为可执行几何调整。

## 11. 阶段八：Icon Composer 组装

### 11.1 导入

将编号 SVG/PNG 导入 Icon Composer。Apple 文档支持一次导入多个文件或文件夹，并会按名称组织；因此保持编号和完整画布。

确认：

- 图层落在正确位置，没有因紧边界导出而漂移；
- 后到前顺序正确；
- 组数不超过四个；
- 主形、支撑形和焦点能独立调节；
- 没有导入圆角矩形或圆形平台遮罩。

### 11.2 材质和外观

在 Icon Composer 内添加或调整：

- 背景色或简单背景渐变；
- blur 和 shadow；
- specular highlight；
- refraction；
- opacity、translucency 和其他 Liquid Glass 属性；
- 平台和外观差异。

使用安装版本实际提供的所有外观模式。至少检查 Default、Dark、Mono；如果版本还提供 Clear、Tinted 或其他模式，也逐一检查。不要根据旧版名称假定当前工具只支持某一组模式。

效果服务于轮廓，不用效果掩盖薄弱概念。关闭材质后，源图仍应具备清晰的主次关系。

### 11.3 操作方式

目前没有已确认的公开 Icon Composer 脚本接口。需要实际组装时：

1. 优先让用户打开目标 `.icon` 或由 Codex 通过受控 GUI 打开。
2. 使用最少量 Computer Use 完成导入、排序和属性调整。
3. 在每个关键更改后截图或导出预览。
4. 遇到意外系统提示、文件替换、Xcode 工程变更或权限请求时停止确认。

## 12. 阶段九：Xcode 接入

### 12.1 添加资源

将 `AppName.icon` 作为主应用 target resource 加入 Xcode。若使用 XcodeGen 或其他工程生成器，应修改声明式项目配置，而不是只改生成后的工程文件。

设置或核对：

```text
ASSETCATALOG_COMPILER_APPICON_NAME = AppName
```

名称通常与 `.icon` 文件基名一致。不要同时保留一个冲突的 asset catalog app icon 配置，除非当前 Apple 文档和部署策略明确要求兼容方案。

Apple 说明：添加 Icon Composer 文件后，它会替代此前代表 app icon 的 asset catalog；Xcode 可从 `.icon` 为旧系统生成相似版本。如果产品必须在旧系统保留原图标，应继续使用 asset catalogs，并先验证当前 Xcode 的具体兼容路径。

### 12.2 构建验证

从目标仓库声明的正常构建入口运行。验证：

- Xcode/`actool` 识别 `.icon` 为资源；
- 构建产物包含 `AppName.icns`；
- 构建产物包含 `Assets.car`；
- 编译后的 Info.plist 含正确的 `CFBundleIconFile` 和 `CFBundleIconName`；
- 安装副本与本轮构建产物的图标文件一致；
- 主 App 的图标接入没有误伤 helper、extension、input method 或其他 target 的独立图标。

若仓库使用签名、安装或 Launch Services 注册，按项目规则执行；“本机 ad-hoc 可启动”不等于 Developer ID/notarized 分发通过。

### 12.3 环境故障识别

Vitemis 的真实项目曾出现：沙箱内首次构建因 Icon Composer 无法访问系统导出服务而失败，在正常 Xcode 环境用同一源码重跑后成功。遇到类似错误时：

1. 保存完整构建错误。
2. 区分源素材错误、`actool` 错误和系统服务权限错误。
3. 经用户批准后，在正常 Xcode 环境重跑同一构建。
4. 不把系统服务故障误判成图标设计或 SVG 失败。

## 13. 阶段十：成品验收

### 13.1 尺寸矩阵

至少检查：

| 尺寸 | 主要目的 |
|---|---|
| 1024 | 曲线、边缘、渐变和导出瑕疵 |
| 512 | 大图标整体关系 |
| 256 | Finder 与常见展示尺寸 |
| 128 | 主次层级 |
| 64 | 细节是否开始粘连 |
| 32 | 主轮廓和负空间门槛 |
| 16 | 最小质量块和对比底线 |

### 13.2 外观与上下文矩阵

检查：

- Icon Composer 当前版本的全部 appearance modes；
- 浅色、深色、中灰和接近主色的背景；
- Dock、Finder、Spotlight、App Switcher、About 窗口和 Xcode 预览；
- 实际安装后的应用图标，而非只看设计工具预览；
- 支持旧系统时，由 Xcode 生成的兼容图标。

### 13.3 通过标准

- 32 px 保留清楚的主轮廓。
- 16 px 不出现破碎噪点或完全糊成一团。
- 关键结构不依赖文字、细线或微小孔洞。
- 不同外观模式下主形仍有足够对比。
- 视觉重心稳定。
- 与常见竞品并排时有明显区分。
- 没有 SF Symbols 或其容易混淆的近似形。
- 每层对齐、非空、命名明确、没有外部资源。
- Icon Composer 效果没有遮掩主概念。
- Xcode 生成并实际使用预期 `.icns` 和 `Assets.car`。

## 14. 交付结构

完整交付至少包括：

- `brief.md`：产品含义和选择记录。
- `icon-spec.json`：可复现规格。
- `.ai`：可编辑 Illustrator 真源。
- `.jsx`：当前版本构建脚本，若使用脚本生成。
- `layers/`：编号的完整画布 SVG/PNG。
- `previews/`：大图、尺寸接触表和关键外观预览。
- `AppName.icon/`：Icon Composer 真源。
- `review.md`：通过项、未验证项和已知限制。
- Xcode/project 配置改动及对应验证记录（若本轮包含集成）。

如果用户只要求概念或分层素材，明确说明哪些后续交付物尚未产生，不把阶段性交付称为完整成品。

## 15. 常见故障与处理

### Illustrator 不响应 AppleScript

- 确认应用 ID、安装版本和应用是否已启动。
- 检查 macOS Automation 权限。
- 使用一次性最小 JSX 创建空文档验证通路。
- 不通过模拟点击掩盖接口故障。

### JSX 成功但没有输出

- 检查所有输出路径是否为绝对路径。
- 检查目录是否存在、是否可写以及文件是否因拒绝覆盖而跳过。
- 把错误写入明确日志或返回值。
- 检查 Illustrator 是否停在保存、字体或警告对话框。

### SVG 导入后错位

- 检查是否导出了对象紧边界而不是完整画板。
- 比较所有 `viewBox`、宽高和坐标原点。
- 检查 Illustrator 脚本的 Y 轴转换。
- 检查隐藏图层状态是否在导出之间泄漏。

### Icon Composer 预览与设备不同

- 在实际 Xcode 构建和真实目标系统上复核。
- 逐层关闭 Screen/混合模式、透明度和复杂效果以定位差异。
- 将不稳定效果简化为更基础的形状或透明 PNG。
- 保存工具版本、系统版本和复现步骤；不要只凭工具预览宣告通过。

### Xcode 没有生成 `.icns`

- 确认 `.icon` 进入正确 target 的 Resources。
- 确认 `ASSETCATALOG_COMPILER_APPICON_NAME` 与基名一致。
- 查看 `actool` 命令是否同时接收 `.icon` 和资产目录。
- 检查是否存在冲突的 app icon asset 配置。
- 区分沙箱系统导出服务失败与资源配置错误。

### 32 px 不可辨

- 先删除小装饰，不先加描边。
- 放大主要负空间或合并相近质量块。
- 提高主次面积差和明度差。
- 必要时回到概念阶段，不用材质效果补救。

## 16. 完成定义

只有满足下列条件，才称为“图标完成”：

- 用户确认产品含义和方向。
- 原创性/SF Symbols 边界已检查。
- 规格、Illustrator 源和分层导出可重建。
- 大图和 16–1024 px 预览已实际查看。
- Icon Composer 中的目标平台和全部可用外观已检查。
- 若包含工程接入，Xcode 构建产物和 Info.plist 图标键已核验。
- 交付物和未验证项已清楚记录。

以下情况不能称为完成：

- 只提供文字概念；
- 只在 Illustrator 里看过 1024 px；
- 只验证 SVG/XML 结构；
- 只在 Icon Composer 预览、没有导出或构建；
- 使用了 SF Symbol 或近似形；
- 没有保存可编辑源；
- 没有说明未完成的平台或外观验证。

## 17. 权威资料

- [Apple：SF Symbols Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/sf-symbols) — app icon、logo 和商标用途限制；应用内 symbol 使用方式。
- [Apple：App icons Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/app-icons) — 当前应用图标设计原则。
- [Apple：Creating your app icon using Icon Composer](https://developer.apple.com/documentation/Xcode/creating-your-app-icon-using-icon-composer) — 1024 画布、分层、SVG/PNG、效果、最多四组和 Xcode 行为。
- [Apple：Icon Composer](https://developer.apple.com/icon-composer/) — 当前功能、平台和外观模式。
- [Apple：Create icons with Icon Composer（WWDC25）](https://developer.apple.com/videos/play/wwdc2025/361/) — Illustrator 分层导出和材质工作流演示。
- [Apple Design Resources](https://developer.apple.com/design/resources/) — 最新网格与图标模板。
- [Adobe：Install and run scripts in Illustrator](https://helpx.adobe.com/illustrator/desktop/automate-visualize-data/automate-actions/install-and-run-scripts.html) — JavaScript、ExtendScript 和 AppleScript 支持。
- [Adobe Illustrator Developer](https://developer.adobe.com/illustrator/) — Illustrator API、脚本与自动化能力。
