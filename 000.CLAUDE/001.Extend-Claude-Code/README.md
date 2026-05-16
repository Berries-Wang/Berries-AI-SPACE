# Extend Claude Code(扩展 Claude Code)
Claude Code 结合了一个能够推理代码的模型和内置工具，用于文件操作、搜索、执行和网络访问。内置工具涵盖了大多数编码任务。本指南涵盖扩展层：您添加的功能，用于自定义 Claude 的知识、将其连接到外部服务以及自动化工作流。

## 扩展插入代理循环<sup>阅读:[000.CLAUDE/000.Claude工作原理.md](../../000.CLAUDE/000.Claude工作原理.md)</sup>的不同部分：
> 这些扩展都是插入到核心代理循环之上的层。它们利用循环的能力来增强 Claude 的功能，同时保持核心循环的完整性和效率。

|扩展|描述|参考|注意|
|-|-|-|-|
|- CLAUDE.md |- 添加 Claude 每个会话都能看到的持久上下文|- [000.CLAUDE/001.Extend-Claude-Code/CLAUDE.md](../../000.CLAUDE/001.Extend-Claude-Code/CLAUDE.md)|-|
|-|-|-|-|
|-Skills |- 添加可重用的知识和可调用的工作流|-|-|
|-|-|-|-|
|-MCP |- 将 Claude 连接到外部服务和工具|-|-|
|-|-|-|-|
|-Subagents|- 在隔离的上下文中运行自己的循环，返回摘要|-|-|
|-|-|-|-|
|-Agent teams|- 协调多个独立会话，具有共享任务和点对点消息传递|-|-|
|-|-|-|-|
|-Hooks|- 在生命周期事件上触发，可以运行脚本、HTTP 请求、提示或 subagent|-|-|
|-|-|-|-|
|- Plugins 和 marketplaces|-  打包和分发这些功能|-|-|


### 将功能与您的目标相匹配
|功能	|作用	|何时使用	|示例|
|-|-|-|-|
| CLAUDE.md	|每次对话加载的持久上下文|	项目约定、“始终执行 X” 规则	|”使用 pnpm，而不是 npm。提交前运行测试。“|
| Skill	Claude |可以使用的说明、知识和工作流	|可重用内容、参考文档、可重复的任务	|/deploy 运行您的部署清单；包含端点模式的 API 文档 skill|
| Subagent|	返回摘要结果的隔离执行上下文|	上下文隔离、并行任务、专门的工作者|	读取许多文件但仅返回关键发现的研究任务|
| Agent teams	|协调多个独立的 Claude Code 会话|	并行研究、新功能开发、使用竞争假设进行调试|	生成审查者同时检查安全性、性能和测试|
| MCP	|连接到外部服务|	外部数据或操作	|查询您的数据库、发布到 Slack、控制浏览器|
| Hook	|由事件触发的脚本、HTTP 请求、提示或 subagent|	必须在每个匹配事件上运行的自动化	|每次文件编辑后运行 ESLint|

Plugins 是打包层。Plugin 将 skills、hooks、subagents 和 MCP servers 捆绑到单个可安装单元中。Plugin skills 是命名空间的（如 /my-plugin:review），因此多个 plugins 可以共存。当您想在多个存储库中重用相同的设置或通过 marketplace 分发给他人时，使用 plugins。