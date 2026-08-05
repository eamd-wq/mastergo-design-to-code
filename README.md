# MasterGo Design to Code

基于 MasterGo Magic MCP 的通用设计转代码 Skill，通过设计稿 DSL 辅助 AI 完成高保真界面实现。

## 使用前准备

1. 安装并配置 [MasterGo Magic MCP](https://github.com/mastergo-design/mastergo-magic-mcp)，将认证凭据保存在安全的环境变量或 MCP 配置中。
2. 准备 MasterGo 设计链接，或对应的文件 ID、图层 ID。
3. 提前准备设计稿使用的 Icon，并向 AI 说明每个 Icon 的名称、资源地址、用途、交互状态和显示要求。不要让 AI 根据截图猜测 Icon。
4. 需要给AI一张设计稿图片让其参考验证。（目前使用GPT5.5 5.6，用最高的思考等级）

如果无法获取完整 DSL，Skill 会说明原因并停止开发，不会仅依据截图生成实现。

## 使用方式

将本仓库作为 Skill 安装到支持 Skills 的 AI 编程环境中，然后在任务中调用：

```text
使用 $mastergo-design-to-code 根据以下 MasterGo 设计稿完成界面开发：

- 设计链接：<MasterGo 链接>
- 实现要求：<功能与交互要求>
- Icon：<资源地址、名称、用途和状态说明>
```

AI 会先读取 DSL、分析当前代码库和公共组件，再开始实现与验证。
