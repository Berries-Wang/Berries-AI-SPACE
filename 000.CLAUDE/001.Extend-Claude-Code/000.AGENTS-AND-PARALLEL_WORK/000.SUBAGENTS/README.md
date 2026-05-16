# subagents
Subagents 是处理特定类型任务的专门 AI 助手。当一个辅助任务会用搜索结果、日志或文件内容充斥您的主对话，而您不会再次引用这些内容时，请使用一个 subagent：该 subagent 在自己的上下文中完成这项工作，仅返回摘要。当您不断生成相同类型的工作者并使用相同的指令时，定义一个自定义 subagent。

每个 subagent 在自己的 context window 中运行，具有自定义系统提示、特定的工具访问权限和独立的权限。**当 Claude 遇到与 subagent 描述相匹配的任务时，它会委托给该 subagent，该 subagent 独立工作并返回结果。** <sup>自动调用吗?</sup>


## 摘要
|说明|参考|备注|
|-|-|-|
|- Subagents 在单个会话中工作。要在并行运行许多独立会话并从一个地方监控它们，请参阅 background agents。对于相互通信的会话，请参阅 agent teams。|-|-|
|-|-|-|
|-|-|-|


## Subagents 特征
|特征|描述|备注|
|-|-|-|
|- 保留上下文|- 通过将探索和实现保持在主对话之外|-|
|-|-|-|
|- 强制执行约束|- 通过限制 subagent 可以使用的工具|-|
|-|-|-|
|- 跨项目重用配置|- 使用用户级 subagents|-|
|-|-|-|
|- 专门化行为|- 为特定领域使用专注的系统提示|-|
|-|-|-|
|-控制成本|- 通过将任务路由到更快、更便宜的模型（如 Haiku）|-|


Claude 使用每个 subagent 的描述来决定何时委托任务。创建 subagent 时，请编写清晰的描述，以便 Claude 知道何时使用它。

Claude Code 包括几个内置 subagents，如 Explore、Plan 和 general-purpose。您也可以创建自定义 subagents 来处理特定任务。

## 自定义subagent
Subagents 是带有 YAML frontmatter 的 Markdown 文件。根据范围将它们存储在不同的位置。当多个 subagents 共享相同的名称时，更高优先级的位置获胜。

|Location|Scope|Priority|如何创建|
|-|-|-|-|
|托管设置|组织范围|1（最高）|通过 managed settings 部署|
|--agents CLI 标志|当前会话|2|启动 Claude Code 时传递 JSON|
|.claude/agents/|当前项目|3|交互式或手动|
|~/.claude/agents/|所有您的项目|4|交互式或手动|
|Plugin 的 agents/ 目录|启用 plugin 的位置|5（最低）|与 plugins 一起安装|

##### CLI 定义的 subagents
```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  },
  "debugger": {
    "description": "Debugging specialist for errors and test failures.",
    "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes."
  }
}'
```

---
---

### 编写 subagent 文件
Subagent 文件使用 YAML frontmatter 进行配置，然后是 Markdown 中的系统提示：
> Subagents 在会话启动时加载。如果您直接在磁盘上添加或编辑 subagent 文件，请重启您的会话以加载它。通过 /agents 界面创建的 Subagents 无需重启即可立即生效。

```txt
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

Frontmatter 定义了 subagent 的元数据和配置。正文成为指导 subagent 行为的系统提示。**Subagents 仅接收此系统提示（加上基本环境详细信息，如工作目录），而不是完整的 Claude Code 系统提示。**

#### frontmatter 字段
以下字段可以在 YAML frontmatter 中使用。只有 name 和 description 是必需的。

|Field|Required|Description|
|-|-|-|
|name|Yes|使用小写字母和连字符的唯一标识符。Hooks 将此值作为 agent_type 接收。文件名不必匹配|
|description|Yes|Claude 何时应该委托给此 subagent|
|tools|No|Tools subagent 可以使用。如果省略，继承所有工具。要将 Skills 预加载到上下文中，请使用 skills 字段而不是在此处列出 Skill|
|disallowedTools|No|要拒绝的工具，从继承或指定的列表中删除|
|model|No|Model 使用：sonnet、opus、haiku、完整模型 ID（例如，claude-opus-4-7）或 inherit。默认为 inherit|
|permissionMode|No|Permission mode：default、acceptEdits、auto、dontAsk、bypassPermissions 或 plan。对于 plugin subagents 被忽略|
|maxTurns|No|subagent 停止前的最大代理轮数|
|skills|No|Skills 在启动时加载到 subagent 的上下文中。注入完整的技能内容，而不仅仅是描述。Subagents 仍然可以通过 Skill 工具调用未列出的项目、用户和 plugin 技能|
|mcpServers|No|MCP servers 对此 subagent 可用。每个条目要么是引用已配置服务器的服务器名称（例如，"slack"），要么是内联定义，其中服务器名称为键，完整的 MCP server config 为值。对于 plugin subagents 被忽略|
|hooks|No|Lifecycle hooks 限定于此 subagent。对于 plugin subagents 被忽略|
|memory|No|Persistent memory scope：user、project 或 local。启用跨会话学习|
|background|No|设置为 true 以始终将此 subagent 作为 background task 运行。默认：false|
|effort|No|此 subagent 活跃时的努力级别。覆盖会话努力级别。默认：从会话继承。选项：low、medium、high、xhigh、max；可用级别取决于模型|
|isolation|No|设置为 worktree 以在临时 git worktree 中运行 subagent，为其提供存储库的隔离副本。如果 subagent 不进行任何更改，worktree 会自动清理|
|color|No|Subagent 在任务列表和转录中的显示颜色。接受 red、blue、green、yellow、purple、orange、pink 或 cyan|
|initialPrompt|No|当此代理作为主会话代理运行时（通过 --agent 或 agent 设置），自动提交为第一个用户轮次。Commands 和 skills 被处理。前置于任何用户提供的提示|

#### 控制subagents能力
|名称|参考|
|-|-|
|- 将技能预加载到 subagents|- [将技能预加载到 subagents](./001.控制subagent能力/002.将技能预加载到%20subagents.md)|
|-|-|
|- 启用持久内存|- [启用持久内存](./001.控制subagent能力/003.启用持久内存.md)|
|-|-|
|- 使用 hooks 的条件规则|- [使用 hooks 的条件规则](./001.控制subagent能力/001.使用%20hooks%20的条件规则.md)|


---
---

## 使用 subagents
|名称|说明|参考|
|-|-|-|
|- 自动委托|- Claude 根据您请求中的任务描述、subagent 配置中的 description 字段和当前上下文自动委托任务。要鼓励主动委托，在您的 subagent 的 description 字段中包含”use proactively”之类的短语。|- 主动地、静默地将任务分配给一个专门的 Subagent（子代理）去独立执行|
|-|-|-|
|- 显式调用 subagents|- 当自动委托不够时，您可以自己请求 subagent|- 自然语言：在提示中命名 subagent；Claude 决定是否委托(<br/>+ Use the test-runner subagent to fix failing tests) <br/>- @-mention：保证 subagent 为一个任务运行(<br/>+ @-mention subagent。 输入 @ 并从类型提前中选择 subagent，就像您 @-mention 文件一样。这确保特定 subagent 运行，而不是将选择留给 Claude：<br/>+ +@"code-reviewer (agent)" look at the auth changes)<br/>- 会话范围：整个会话使用该 subagent 的系统提示、工具限制和模型，通过 --agent 标志或 agent 设置(将整个会话作为 subagent 运行。 传递 `--agent <name> `以启动一个会话，其中主线程本身采用该 subagent 的系统提示、工具限制和模型：<br/>+claude --agent code-reviewer)|
|-|-|-|
|- 链接 subagents|- 对于多步骤工作流，要求 Claude 按顺序使用 subagents。每个 subagent 完成其任务并将结果返回给 Claude，然后将相关上下文传递给下一个 subagent。|- Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them|
|-|-|-|
|- 运行并行研究|- 对于独立的调查，生成多个 subagents 以同时工作： <br/> 每个 subagent 独立探索其区域，然后 Claude 综合这些发现。当研究路径彼此不依赖时，这效果最好。<sup>当 subagents 完成时，它们的结果返回到您的主对话。运行许多 subagents，每个都返回详细结果，可能会消耗大量上下文。</sup> <br/>- 对于需要持续并行性或超过您的 context window 的任务，[agent teams](../../../../000.CLAUDE/001.Extend-Claude-Code/000.AGENTS-AND-PARALLEL_WORK/001.AGENT-TEAM/README.md) 为每个工作者提供自己的独立上下文 。|- Research the authentication, database, and API modules in parallel using separate subagents|


---
---

## 示例subagent
### 代码审查者
```txt
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

---
---

## 参考资料
- [https://code.claude.com/docs/zh-CN/sub-agents](https://code.claude.com/docs/zh-CN/sub-agents)