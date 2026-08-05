# MasterGo MCP 工作流

## 服务发现

- 官方仓库：`https://github.com/mastergo-design/mastergo-magic-mcp`
- 先检查当前环境是否已经提供 MasterGo MCP 工具、配置或对应安装包；已存在则直接使用，不重复安装。
- 只有找不到可用服务时，才按照官方仓库最新说明安装并配置。
- Token、Cookie、密钥等认证信息必须从用户提供的安全输入、环境变量或 MCP 配置读取。不要将认证信息写进 Skill、代码、配置示例、日志或最终回答。

## 支持的输入

- MasterGo 短链接或完整设计链接。
- `fileId` 与 `layerId`。
- URL 中的 `source_layer_id`，存在时优先作为查询根图层。
- 用户提供的截图只能在 DSL 成功获取后作为视觉校验补充，不能替代 DSL 或单独作为开发依据。

## DSL 前置门禁

在修改代码前完成以下检查：

1. 确认可用认证凭据已通过安全输入、运行环境或 MCP 配置提供。
2. 成功读取目标图层完整且可解析的 DSL，并确认所需 section 没有缺失。
3. 如果认证凭据不可用，或认证、权限、链接、MCP 服务、网络等问题导致 DSL 获取失败、返回空数据或无法完整读取，先执行必要诊断和有限重试。
4. 重试后仍无法通过门禁时，向用户说明具体原因、失败环节以及需要补充或修复的内容，并立即中断任务。此时不得修改实现代码，也不得依据截图、预览图、节点名称或经验猜测设计数值和交互。

只有成功通过以上门禁，才能进入设计分析和编码阶段。

## 标准读取顺序

1. 调用 `mcp__mastergo_magic_mcp__mcp__getDesignSections`，先不传 `sectionIndex`，获取：
   - 根图层尺寸、名称、类型和背景。
   - `totalSections`、`totalNodes`。
   - 每个 section 的 id、名称、节点数和页面绝对 bbox。
   - 被拆分容器的布局方向、间距和内边距。
2. 按 3～5 个 section 一批调用同一工具，读取从 `0` 到 `totalSections - 1` 的全部 DSL。不能因为节点结构相似、名称为空或只需要局部元素而跳过 section。
3. 全部 section 完成后并行调用：
   - `mcp__mastergo_magic_mcp__mcp__getDesignSvgs`，获取 PATH 节点对应的完整 `svgHtml`。
   - `mcp__mastergo_magic_mcp__mcp__getDesignTexts`，解析以 `T{sectionIndex}|{nodeId}` 表示的长文本。
4. 如果 DSL 中存在组件文档链接，再调用 `mcp__mastergo_magic_mcp__mcp__getComponentLink` 获取组件说明。
5. `mcp__mastergo_magic_mcp__mcp__getDsl` 仅在 `getDesignSections` 不可用或明确返回错误时作为 fallback；完成 section 工作流后不要再调用它重复取数。
6. `mcp__mastergo_magic_mcp__mcp__getMeta` 可用于获取页面级元信息，但不能代替 section DSL。

## 数据使用规则

- 通过节点的 `fillStyleId`、`strokeStyleId`、字体 style 和 styles map 获取真实颜色与排版参数。
- 使用 `layoutStyle` 的宽高和相对坐标理解节点内部布局，使用 section bbox 理解页面级叠加关系。
- PATH 节点必须使用 MCP 返回的 SVG 数据，不得伪造 path。
- 长文本必须使用文本接口返回的原文，不得自行改写、翻译或补全。
- 设计图中的图片 URL 可能是临时地址；需要落地资源时及时下载，并遵循当前项目的资源管理规则。
- 保存工具结果时只保存设计数据，不保存凭据；输出日志时避免打印认证信息和不必要的大段 DSL。

## 失败处理

- 请求超时：保持当前批次结果，缩小批量后重试缺失 section，不要从头重复全部请求；仍无法取得完整 DSL 时触发前置门禁并停止开发。
- 设计链接无权限：报告不可访问的链接或图层、权限错误及所需的有效凭据、可访问链接或完整 DSL 数据，然后停止开发。
- MCP 未配置：先检查本地或全局安装与 MCP 配置；完成必要配置后仍不可用，或缺少有效认证凭据时，说明原因并停止开发。
- DSL 返回为空、损坏或不完整：记录失败接口和缺失范围，有限重试后仍失败则停止开发，不使用截图补齐。
- DSL 与截图不一致：确认是否读取了正确页面、图层和最新版本；无法确认可靠设计来源时，向用户说明冲突并停止开发，不自行选择或推断。
