---
name: perform-mr-review
description: Use when the user asks to review an MR, pull a branch for review, or invokes /performMRreview. Accepts a GitLab branch name (e.g. TNR-16503) or a GitLab MR URL.
---

# Perform MR Review

Fetch a branch, analyse the diff, and dispatch a structured code review via the `superpowers:code-reviewer` subagent.

## Input

The argument (`$ARGUMENTS`) is either:
- A **branch name** — e.g. `TNR-16503`
- A **GitLab MR URL** — e.g. `https://sgts.gitlab-dedicated.com/.../merge_requests/42`

## Steps

### 1. Resolve the branch name

**If a branch name was given:** use it directly.

**If a GitLab MR URL was given:** extract the MR IID from the URL (the number after `merge_requests/`), then fetch the source branch via the GitLab API:

```bash
# URL-encode the project path (replace / with %2F)
PROJECT="wog%2Fced%2Fship-tnr%2Fcustoms-tnr%2Fcos%2Fcos-webapp"
MR_IID=<number from URL>
curl -s "https://sgts.gitlab-dedicated.com/api/v4/projects/${PROJECT}/merge_requests/${MR_IID}" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['source_branch'])"
```

If the API call fails (no token, network error), ask the user: "I couldn't resolve the branch automatically — what is the source branch name?"

### 2. Pull latest main and fetch the branch

```bash
git fetch origin main
git fetch origin <BRANCH>
git log --oneline origin/main...origin/<BRANCH>
git diff origin/main...origin/<BRANCH> --stat | grep -v "\.png"
```

### 3. Get SHAs

```bash
BASE_SHA=$(git rev-parse origin/main)
HEAD_SHA=$(git rev-parse origin/<BRANCH>)
```

### 4. Read key source files

From the diff stat, identify the most important **non-binary** changed files (components, logic, config — skip PNG snapshots). Read them to understand the implementation before dispatching the reviewer.

### 5. Build a description of what was implemented

From the branch name, commit messages, and file changes, summarise:
- What new components / features / fixes were added
- What patterns/requirements they should follow (check `CLAUDE.md` for project conventions)
- Any notable dependency or config changes

### 6. Dispatch the `superpowers:code-reviewer` subagent

Use the Agent tool with `subagent_type: superpowers:code-reviewer` and fill in:

```
WHAT_WAS_IMPLEMENTED: <1-2 sentence summary>
PLAN_OR_REQUIREMENTS: <bullet list of what the branch should do, from ticket name + commits>
BASE_SHA: <origin/main SHA>
HEAD_SHA: <branch HEAD SHA>
DESCRIPTION: <brief summary for the reviewer>

Ask the reviewer to focus on:
- Correctness and completeness (e.g. are all new files exported?)
- Consistency with project patterns (CLAUDE.md)
- Accessibility and TypeScript correctness
- Test coverage
- Dependency concerns
```

### 7. Present findings

When the reviewer returns, present a structured summary:
- 🔴 **Critical** — blockers
- 🟡 **Important** — should fix before merge
- 🔵 **Minor** — nice-to-have
- ✅ **What looks good**

## Notes

- Ignore PNG visual snapshot changes in the diff stat — they are auto-generated baselines. Mention in the review if they all changed (likely a Node/font version bump) and ask the reviewer to verify visually.
- The main branch is `main`.
- Project is at `git@sgts.gitlab-dedicated.com:wog/ced/ship-tnr/customs-tnr/cos/cos-webapp.git`
