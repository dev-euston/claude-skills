---
name: plan
description: Orchestrator for complex feature planning. Gathers requirements interactively, then offloads codebase analysis and plan writing to subagents to keep the main context small. This is the primary entry point for any non-trivial task. Trigger phrases: "plan this", "plan this feature", "let's plan", "I need to build", "help me plan", "create this", "build this", "I want to create", "I want to build", "implement this".
---

# Plan (Orchestrator)

Orchestrate the full planning pipeline for complex tasks. Handle all user interaction in this context, then delegate the heavy lifting to focused subagents.

**Announce at start:** "I'm using the plan skill to gather requirements, then I'll offload analysis and planning to subagents."

**Depends on:** `analyse-task`, `write-plan`

## Why an orchestrator

Each subagent starts with a clean context. Keeping analysis and planning in separate subagents means neither accumulates the full conversation history. This makes both faster and cheaper. The orchestrator's job is to gather everything a subagent needs upfront, so it can run to completion without asking questions.

---

## Phase 1: Requirements gathering (inline — do not offload)

This phase happens in the orchestrator's context because subagents cannot interactively ask the user questions mid-run.

### 1a. Explore project context

Do a quick codebase scan to understand what already exists. This informs the questions you ask — don't ask what the code already answers.

### 1b. Scope check

If the task spans multiple independent subsystems, flag it immediately. Agree on scope before asking any other questions.

### 1c. Clarifying questions

Ask questions **one at a time** until every ambiguity is resolved.

Focus on:
- **Purpose:** what problem does this solve and for whom?
- **Constraints:** performance, compatibility, security, dependencies
- **Success criteria:** how will the engineer know it's done?
- **Edge cases:** what inputs or states could cause this to fail?
- **Out of scope:** what explicitly does NOT need to be built?

Prefer multiple-choice questions. Do not proceed to Phase 2 until zero open questions remain.

---

## Phase 2: Analysis (subagent)

Once requirements are fully gathered, dispatch the `analyse-task` subagent.

### Subagent prompt

Construct the prompt as a self-contained brief. Include everything the subagent needs — it has no access to this conversation:

```
You are running the `analyse-task` skill.

## Task
[Original task description]

## Gathered requirements
[Full Q&A from Phase 1, written as resolved statements — not raw dialogue]
- Goal: ...
- Constraints: ...
- Success criteria: ...
- Out of scope: ...
- Edge cases: ...

## Instructions
Requirements have already been gathered. Skip the clarifying questions phase.
Go straight to codebase exploration and produce the analysis document.
Save it to: <output_dir>/plans/YYYY-MM-DD-<feature-name>-analysis.md
```

### On completion

The subagent returns the full analysis document. Read it before proceeding — do not pass it blindly to the next subagent. If the analysis reveals a scope problem or something that changes the requirements, resolve it with the user before continuing.

---

## Phase 3: Planning (subagent)

Dispatch the `write-plan` subagent with the analysis document as its sole input.

### Subagent prompt

```
You are running the `write-plan` skill.

## Analysis document
[Paste the full contents of the analysis document here]

## Instructions
The analysis document above contains everything you need.
Read the codebase only when the analysis references something you need to see
and the benefit of reading it outweighs the cost.
Save the plan to: <output_dir>/plans/YYYY-MM-DD-<feature-name>.md
Output the full plan when done.
```

### On completion

The subagent returns the full plan. Output it to the conversation so the user can read it immediately.

---

## Phase 4: Handoff

After presenting the plan, confirm with the user:

> "Plan saved to `<path>`. Ready to implement, or do you want to adjust anything first?"
