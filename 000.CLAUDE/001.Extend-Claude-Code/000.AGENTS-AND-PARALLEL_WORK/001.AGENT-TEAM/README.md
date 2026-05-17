# Agent Team
协调多个 Claude Code 实例作为一个团队一起工作，具有共享任务、代理间消息传递和集中管理。

Agent teams 让你协调多个 Claude Code 实例一起工作。一个会话充当团队负责人，协调工作、分配任务和综合结果。队友独立工作，每个都在自己的 context window 中，并直接相互通信。

与 subagents 不同，subagents 在单个会话中运行，只能向主代理报告，你也可以直接与个别队友互动，无需通过负责人。

## Agent Team使用场景
Agent teams 最适合用于并行探索能增加真实价值的任务。
|场景|描述|备注|
|-|-|-|
|研究和审查|多个队友可以同时调查问题的不同方面，然后分享和质疑彼此的发现|
|新模块或功能|队友可以各自拥有一个独立的部分，不会相互干扰|
|使用竞争假设进行调试|队友并行测试不同的理论，更快地收敛到答案|
|跨层协调|跨越前端、后端和测试的更改，每个由不同的队友负责|

Agent teams 增加了协调开销，使用的令牌数量明显多于单个会话。当队友可以独立运作时，它们效果最好。对于顺序任务、同一文件编辑或有许多依赖关系的工作，单个会话或 subagents 更有效。

##  与 subagents 比较
Agent teams 和 subagents 都让你并行化工作，但它们的运作方式不同。根据你的工作人员是否需要相互通信来选择：
![000.CLAUDE/999.REF-DOCS/999.REF_IMGS/subagents-vs-agent-teams-dark.avif](../../../../000.CLAUDE/999.REF-DOCS/999.REF_IMGS/subagents-vs-agent-teams-dark.avif)

Subagents 仅向主代理报告结果，彼此不交谈。在 agent teams 中，队友共享任务列表、认领工作并直接相互通信。
### 对比摘要
|对比项|	Subagents|	Agent teams|
|-|-|-|
|Context	|自己的 context window；结果返回给调用者	|自己的 context window；完全独立|
|通信	|仅向主代理报告结果	|队友直接相互发送消息|
|协调	|主代理管理所有工作	|具有自我协调的共享任务列表|
|最适合|	只有结果重要的专注任务	|需要讨论和协作的复杂工作|
|令牌成本	|较低：结果汇总回主 context|	较高：每个队友是一个独立的 Claude 实例|

当你需要快速、专注的工作人员报告结果时，使用 subagents。当队友需要分享发现、相互质疑和自我协调时，使用 agent teams。

---
---

## 启用 agent team 
Agent teams 默认禁用。通过将 CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS 环境变量设置为 1，在你的 shell 环境中或通过 settings.json 来启用它：
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 创建一个Agent Team
启用 agent teams 后，告诉 Claude 创建一个 agent team，并用自然语言描述你想要的任务和团队结构。Claude 创建团队、生成队友并根据你的提示协调工作。

示例命令:
- I'm designing a CLI tool that helps developers track TODO comments across their codebase. Create an agent team to explore this from different angles: one teammate on UX, one on technical architecture, one playing devil's advocate.
- (我正在设计一个 CLI（命令行）工具，用来帮助开发者追踪代码库中的 TODO 注释。请创建一个智能体团队（agent team），从不同角度来探讨这个项目：其中一位成员负责用户体验（UX），一位负责技术架构，还有一位专门扮演‘唱反调’的角色（即提出反对意见或进行批判性思考）)
![wechat_2026-05-16_225714_594.png](../../../../000.CLAUDE/999.REF-DOCS/999.REF_IMGS/wechat_2026-05-16_225714_594.png)

从那里，Claude 创建一个具有 共享任务列表 的团队，为每个角度生成队友，让他们探索问题，综合发现，并在完成时尝试 清理团队。

负责人的终端列出所有队友及其正在处理的工作。使用 Shift+Down 循环浏览队友并直接向他们发送消息。在最后一个队友之后，Shift+Down 会回到负责人。

---
---

## 控制你的 agent team
### 显示模式
|模式名称|说明|
|-|-|
|- In-process|- 所有队友在你的主终端内运行。使用 Shift+Down 循环浏览队友并输入以直接向他们发送消息。在任何终端中工作，无需额外设置。|
|-|-|
|- Split panes|- 每个队友获得自己的窗格。你可以同时看到每个人的输出，并点击窗格直接交互。需要 tmux 或 iTerm2。|
|-|-|

```json
{
  "teammateMode": "in-process"
}
```

---
---

## Agent teams 如何工作
|Agent teams 启动方式|说明|备注|
|-|-|-|
|- 你请求一个团队|- 给 Claude 一个受益于并行工作的任务，并明确要求一个 agent team。Claude 根据你的指示创建一个|-|
|-|-|-|
|- Claude 提议一个团队|- 如果 Claude 确定你的任务将受益于并行工作，它可能会建议创建一个团队。你在它继续之前确认。|-|
|-|-|-|

### 架构
Agent team 由以下部分组成:
|组件|	角色|
|-|-|
|Team lead	|创建团队、生成队友并协调工作的主 Claude Code 会话|
|Teammates|	各自处理分配任务的独立 Claude Code 实例|
|Task list	|队友认领和完成的共享工作项列表|
|Mailbox	|代理之间通信的消息系统|

系统自动管理任务依赖关系。当队友完成其他任务依赖的任务时，被阻止的任务会自动解除阻止。

团队和任务存储在本地：
- Team config： `~/.claude/teams/{team-name}/config.json`
- Task list：`~/.claude/tasks/{team-name}/`

Claude Code 在你创建团队时自动生成这两个，并在队友加入、空闲或离开时更新它们<sup>关机之后会自动重启?</sup>。团队配置保存运行时状态，例如会话 ID 和 tmux 窗格 ID，所以不要手动编辑它或预先编写它：你的更改会在下一次状态更新时被覆盖。

---

### 为队友使用 subagent 定义
- 可以使用 subagent 来启动“队友”

生成队友时，你可以引用来自任何 subagent 范围 的 subagent 类型：项目、用户、插件或 CLI 定义。这让你定义一个角色一次，例如安全审查员或测试运行器，并将其同时重用为委派的 subagent 和 agent team 队友。

要使用 subagent 定义，在要求 Claude 生成队友时按名称提及它：
```txt
Spawn a teammate using the security-reviewer agent type to audit the auth module.
```

---

### Context 和通信
每个队友都有自己的 context window。生成时，队友加载与常规会话相同的项目 context：CLAUDE.md、MCP servers 和 skills。它还接收来自负责人的生成提示。负责人的对话历史不会继承。

队友如何共享信息:
|方式|描述|思考|
|-|-|-|
|- 自动消息传递|- 当队友发送消息时，它们会自动传递给收件人。负责人不需要轮询更新。|-|
|-|-|-|
|- 空闲通知|- 当队友完成并停止时，他们会自动通知负责人。|-|
|-|-|-|
|- 共享任务列表|- 所有代理都可以看到任务状态并认领可用工作。|-|
|-|-|-|
|- 队友消息传递|- 按名称向一个特定的队友发送消息。要联系所有人，请为每个收件人发送一条消息。|-|
|-|-|-|



## 参考资料
- [https://code.claude.com/docs/zh-CN/agent-teams](https://code.claude.com/docs/zh-CN/agent-teams)