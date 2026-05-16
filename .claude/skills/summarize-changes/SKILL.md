---
name: summarize-changes
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
---

## Instructions

1. First, run the shell command `git diff HEAD` to retrieve the current uncommitted changes.
2. If the diff is empty, simply state that there are no uncommitted changes.
3. Otherwise, summarize the changes in two or three bullet points.
4. Finally, list any risks you notice, such as missing error handling, hardcoded values, or tests that need updating. 