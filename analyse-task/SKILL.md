---
name: analyse-task
description: Analyse a task by exploring the codebase and asking clarifying questions one at a time until all ambiguities are resolved. Produces a self-contained analysis document that gives the write-plan subagent everything it needs without re-reading the codebase. Trigger phrases: "analyse this task", "analyse this", "before we plan", "clarify requirements".
---

# Analyse Task

Resolve every open question about a task and produce a self-contained analysis document. This document is the sole input to the `write-plan` subagent — write it assuming that agent has no codebase access and no conversation history.

**Announce at start:** "I'm using the analyse-task skill to explore the codebase and resolve any open questions."

<HARD-GATE>
Do NOT produce the analysis document until all clarifying questions have been answered. If any ambiguity remains, ask another question first.
</HARD-GATE>

## Invocation modes

**Standalone:** full flow below — explore, check scope, ask questions, produce document.

**Via `plan` orchestrator:** requirements are pre-gathered and passed in the prompt. Skip Steps 2–3 and go straight to Step 1 (codebase exploration) then Step 4 (document). The prompt will say "Requirements have already been gathered."

## Step 1: Explore project context

Read the codebase to understand what already exists: relevant files, patterns, dependencies, test setup, and conventions. Check recent commits if git history is available.

**Do not ask the user about anything the codebase already makes clear.**

## Step 2: Scope check

If the task covers multiple independent subsystems, flag this immediately and suggest splitting — one plan per subsystem. Don't refine details until scope is agreed.

## Step 3: Clarifying questions

*(Skip this step if requirements were pre-gathered by the orchestrator.)*

Ask questions **one at a time** until zero open questions remain.

Focus on:
- **Purpose:** what problem does this solve and for whom?
- **Constraints:** performance, compatibility, security, dependencies
- **Success criteria:** how will the engineer know it's done?
- **Edge cases:** what inputs or states could cause this to fail?
- **Out of scope:** what explicitly does NOT need to be built?

Prefer multiple-choice questions when possible.

## Step 4: Write the analysis document

Produce a Markdown document that gives a planner everything needed to write the implementation plan. The goal is to minimise what the next agent needs to look up — embed context directly when it saves more than it costs (e.g. a small but critical function is worth inlining; a 400-line file is not). Use judgement.

**Save to:** `<output_dir>/plans/YYYY-MM-DD-<feature-name>-analysis.md`

### Document structure

````markdown
# Task Analysis: [Feature Name]

## Goal
[One sentence describing what this builds and why]

## Requirements
- [Bullet list of confirmed requirements]

## Out of scope
- [Bullet list of what will NOT be built]

## Success criteria
- [How the engineer knows it's done — observable, testable]

## Constraints
- [Performance, security, compatibility, dependency restrictions]

## Edge cases to handle
- [Specific inputs or states that need explicit handling]

## Codebase context

### Relevant files
| File | Purpose | Notes |
|------|---------|-------|
| `exact/path/to/file.ts` | What it does | What the planner needs to know |

### Key code to inline
<!-- Inline small but critical snippets where a planner needs to see the exact
     shape of the code to make design decisions — interfaces, function signatures,
     type definitions, test helpers. Skip large files; reference them by path instead. -->

**`src/path/to/types.ts` — existing types the new code must conform to:**
```ts
// paste the relevant excerpt here
```

**`tests/path/to/helper.ts` — test utilities already available:**
```ts
// paste relevant excerpt
```

### Patterns and conventions
- [How this codebase names things, structures files, handles errors, writes tests]
- [Example: "tests use `describe/it` with a shared `setup()` fixture in `tests/helpers.ts`"]

### Dependencies
- [Package name + version for anything the implementation will use]

### How to run tests
```bash
# exact command to run the relevant test suite
```

## Open questions
None.
````

## Step 5: Output

After saving the document, output its full contents to the conversation as the handoff to `write-plan`.
