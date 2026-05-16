# 使用 skills 扩展 Claude
Skills 扩展了 Claude 能做的事情。创建一个 SKILL.md 文件，其中包含说明，Claude 会将其添加到其工具包中。Claude 在相关时<sup>?</sup>使用 skills，或者你可以使用 /skill-name 直接调用一个。

## 目标
|目标|备注|说明|
|-|-|-|
|- Skill 是什么？内部原理是怎样的?|-|-|
|- 如何自定义一个SKILL|-|-|
|-|-|-|


## 捆绑Skills
> 捆绑技能（Bundled Skills）其实就是官方为你预先打包好、开箱即用的一组“专业指令集”。

Claude Code 包括一组捆绑 skills，在每个会话中都可用，包括 /simplify、/batch、/debug、/loop 和 /claude-api。与大多数内置命令不同，内置命令直接执行固定逻辑，捆绑 skills 是基于提示的：它们为 Claude 提供详细的说明，让它使用其工具来编排工作。你调用捆绑 skills 的方式与调用任何其他 skill 相同，输入 / 后跟 skill 名称。

## 配置Skills
> 完整的请参考: [000.CLAUDE/001.Extend-Claude-Code/002.SKILL/001.SKILL-Template](../../../000.CLAUDE/001.Extend-Claude-Code/002.SKILL/001.SKILL-Template)

每个 skill 都需要一个 SKILL.md 文件，包含两部分：YAML frontmatter（在 --- 标记之间）告诉 Claude 何时使用该 skill，以及包含 Claude 在调用该 skill 时遵循的说明的 markdown 内容。目录名称变成你输入的命令，description 帮助 Claude 决定何时自动加载该 skill。
将此保存到 ~/.claude/skills/summarize-changes/SKILL.md：

```markdown
---
name: summarize-changes
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
---

## Instructions

1. First, run the shell command `git diff HEAD` to retrieve the current uncommitted changes.
2. If the diff is empty, simply state that there are no uncommitted changes.
3. Otherwise, summarize the changes in two or three bullet points.
4. Finally, list any risks you notice, such as missing error handling, hardcoded values, or tests that need updating. 
```
