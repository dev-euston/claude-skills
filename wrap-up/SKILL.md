---
name: wrap-up
description: Use when ending a session or when explicitly asked to wrap up, update docs, or save memory. Reviews session work and updates README.md, CLAUDE.md, and auto-memory files as needed.
---

# Wrap Up

Review the work done in this session and update documentation and memory as needed. For the current repo:

1. **README.md (all levels)** — If new features, architecture changes, setup steps, or usage patterns were introduced, update any affected README.md files — including the root README and any nested package/app READMEs (e.g. `apps/server/README.md`, `apps/ui-web/README.md`). Only edit if there is genuinely new or changed information.

2. **CLAUDE.md** — If new conventions, commands, patterns, gotchas, or workflow guidance emerged during the session, add them to CLAUDE.md. Only edit if there is something concretely useful for a future Claude session to know.

3. **Auto-memory** — Update memory files in `~/.claude/projects/` for the current project if there is new user feedback, preferences, project context, or non-obvious decisions worth preserving across sessions. Follow the memory file format with frontmatter (name, description, type) and update MEMORY.md index.

Do NOT make cosmetic edits. Do NOT add content that is already captured or derivable from the code. Read existing files before editing any of them. If nothing needs updating, do nothing.
