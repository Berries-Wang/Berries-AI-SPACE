# 使用 worktrees 运行并行会话
在单独的 git worktrees 中隔离并行 Claude Code 会话，以便更改不会相互冲突。涵盖 --worktree 标志、子代理隔离、.worktreeinclude、清理和非 git VCS hooks。

git worktree 是一个单独的工作目录，具有自己的文件和分支，但与主检出共享相同的存储库历史和远程。在自己的 worktree 中运行每个 Claude Code 会话意味着一个会话中的编辑永远不会触及另一个会话中的文件<sup>还是会有冲突的，只有在不冲突的时候才能使用</sup>，因此您可以让 Claude 在一个终端中构建功能，同时在第二个终端中修复错误。


Worktrees 是运行 Claude 并行的几种方式之一。它们隔离文件编辑，而子代理和代理团队协调工作本身。参考:[000.CLAUDE/001.Extend-Claude-Code/000.AGENTS-AND-PARALLEL_WORK/README.md](../../../../000.CLAUDE/001.Extend-Claude-Code/000.AGENTS-AND-PARALLEL_WORK/README.md)
- Worktrees 负责保护你的代码文件不被改乱，而子代理和代理团队负责让 AI 聪明地干活
  + Worktrees 的核心作用是物理层面的环境隔离
  + 子代理与代理团队：协调工作的“大脑分工”
- 如果想让多个 Claude 同时安全地修改代码而不打架，你需要用 Worktrees；如果想让 Claude 更高效地思考、拆解任务或多人配合，需要用到 子代理或代理团队。



## 在 worktree 中启动 Claude
传递 --worktree 或 -w 来创建隔离的 worktree 并在其中启动 Claude。默认情况下，worktree 在您的存储库根目录下的 `.claude/worktrees/<value>/` 下创建，在名为 `worktree-<value>` 的新分支上：

```bash
claude --worktree feature-auth
```

在首次在目录中使用 --worktree 之前，请通过在该目录中运行一次 claude 来接受工作区信任对话框。如果尚未接受信任，--worktree 将以错误退出并提示您首先在目录中运行 claude，包括与 -p 结合使用时。
- 将 .claude/worktrees/ 添加到您的 .gitignore，以便 worktree 内容不会在您的主检出中显示为未跟踪的文件。

### 选择基础分支
Worktrees 从您的存储库的默认分支 origin/HEAD 分支，因此它们从与远程匹配的干净树开始。如果未配置远程或获取失败，worktree 会回退到您当前的本地 HEAD。要始终从本地 HEAD 分支，请在设置中将 worktree.baseRef 设置为 "head"。将 baseRef 设置为 "head" 会使新 worktrees 携带您未推送的提交和功能分支状态，这在隔离需要在进行中的工作上操作的子代理时很有用。该设置仅接受 "fresh" 或 "head"，不接受任意 git refs：
```json
{
  "worktree": {
    "baseRef": "head"
  }
}
```

要从特定的拉取请求分支，请传递以 # 为前缀的 PR 编号或完整的 GitHub 拉取请求 URL。Claude Code 从 origin 获取 `pull/<number>/head` 并在 `.claude/worktrees/pr-<number>` 创建 worktree：
```bash
claude --worktree "#1234"
```

要完全控制 worktrees 的创建方式，请配置 [WorktreeCreate hook](../../../../000.CLAUDE/001.Extend-Claude-Code/003.Claude-Hooks/000.WorktreeCreate.md)<sup>Claude Code</sup>，它完全替代默认的 git worktree 逻辑。



---
---


## 参考资料
- [https://code.claude.com/docs/zh-CN/worktrees](https://code.claude.com/docs/zh-CN/worktrees)