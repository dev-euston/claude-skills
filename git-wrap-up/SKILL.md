---
name: git-wrap-up
description: Update docs and memory based on git commits since the last wrap-up. Use instead of wrap-up when you want to sync documentation to actual committed changes rather than the current session.
---

# Git Wrap Up

Update documentation and memory to reflect what has actually been committed since the last wrap-up. This is a git-history-driven version of wrap-up — it reads the commit log, not the conversation.

## Step 1 — Find the baseline

Run `git log --oneline` to get the commit list. Look for the most recent commit whose message contains a wrap-up marker (e.g. `[wrap-up]`, `docs:`, `chore: wrap-up`, or similar). If none found, use the last 20 commits as the window. Report what baseline commit you're using.

## Step 2 — Understand the changes

For each commit in the window:
- Read the commit message and infer intent
- Run `git show --stat <sha>` to see which files changed
- For non-obvious changes, read the diff unfiltered: `rtk proxy git show <sha> -- <file>` on key files (use `rtk proxy` to bypass RTK output filtering and get the full diff)

Group commits into logical themes (e.g. "new routes added", "schema changes", "frontend pages", "refactors").

## Step 3 — Update README.md files (if warranted)

If the commits introduced new features, changed architecture, added setup steps, or altered usage patterns — update the affected README.md files (root and any nested package READMEs). Read existing content before editing. Only change what is genuinely new or incorrect.

## Step 4 — Update CLAUDE.md (if warranted)

If the commits reveal new conventions, non-obvious patterns, commands, gotchas, or workflow guidance that a future Claude session should know — add them to CLAUDE.md. Do not duplicate what's already there or in the code itself.

## Step 5 — Update auto-memory (if warranted)

Update memory files in `~/.claude/projects/<slug>/memory/` if the commits reveal:
- New project context or milestone state (project memory)
- Non-obvious architectural decisions (project memory)
- Any user feedback captured in commit messages or PR descriptions (feedback memory)

Follow the memory frontmatter format (`name`, `description`, `type`). Update `MEMORY.md` index. Do not duplicate existing entries — update them instead.

## Step 6 — Leave a marker (optional)

If the session itself included a commit, note in your response what baseline future runs should use (the HEAD sha at time of wrap-up). You do not need to commit a marker file unless the user asks.

---

Do NOT make cosmetic edits. Do NOT add content already captured or derivable from the code. If nothing needs updating, say so clearly.
