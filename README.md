# Claude Skills

Custom Claude Code skills for use across projects. Each skill is a `SKILL.md` file that instructs Claude how to perform a specific task.

## Available Skills

### `plan` ⭐ _primary entry point for complex tasks_

Orchestrates the full planning pipeline. Gathers requirements interactively in the main context (so the user can answer questions), then offloads codebase analysis to an `analyse-task` subagent and plan writing to a `write-plan` subagent. Each subagent runs with a clean context, keeping the pipeline fast and cheap.

**Use when:** building or planning anything non-trivial.

**Trigger phrases:** "plan this", "plan this feature", "let's plan", "I need to build", "help me plan", "create this", "build this", "I want to create", "I want to build", "implement this"

**Depends on:** `analyse-task`, `write-plan`

---

### `analyse-task`

Explores the codebase to understand existing context, then asks clarifying questions one at a time — purpose, constraints, success criteria, edge cases, out of scope — until zero ambiguities remain. Produces a concise analysis document as a handoff to `write-plan`.

**Use when:** requirements are unclear or incomplete and need to be resolved before planning or implementation.

**Trigger phrases:** "analyse this task", "analyse this", "clarify requirements", "before we plan"

---

### `write-plan`

Writes a comprehensive, bite-sized implementation plan from a completed task analysis. Tasks are 2-5 minute steps with exact file paths, real code, and exact commands (no placeholders). Self-reviews for coverage after writing, then outputs the full plan to the conversation.

**Use when:** requirements are fully understood and a structured implementation plan is needed.

**Trigger phrases:** "write the plan", "create the implementation plan", "now plan it", "write a plan"

**Depends on:** `analyse-task`

---

### `perform-mr-review`

Fetches a GitLab branch or resolves an MR URL to its source branch, reads the diff and key changed files, then dispatches a structured code review via the `superpowers:code-reviewer` subagent. Returns findings grouped as critical blockers, important fixes, minor suggestions, and what looks good.

**Use when:** the user asks to review an MR or provides a branch name or GitLab MR URL.

**Trigger phrases:** "review this MR", "review branch", "pull this branch for review", `/performMRreview`

---

### `pipeline-plantuml`

Generates PlantUML pipeline diagrams (flow, job dependency graph, sequence), saves `.puml` source files, renders them to PNG via the PlantUML public server, and returns Markdown-embeddable image syntax.

**Use when:** a pipeline diagram is needed, either directly by the user or by another skill.

**Trigger phrases:** "generate a pipeline diagram", "draw a pipeline flow", "create a job dependency graph", "visualise this pipeline"

---

### `gitlab-cicd-docs`

Reads a `.gitlab-ci.yml` file and produces a structured Markdown documentation file with a pipeline overview, stages & jobs breakdown, and embedded PlantUML diagrams (pipeline flow per trigger, job dependency graph, sequence diagram).

**Use when:** the user wants to document a GitLab pipeline, understand a `.gitlab-ci.yml` file, or generate diagrams from a CI/CD config.

**Trigger phrases:** "document this pipeline", "create docs for gitlab-ci.yml", "explain this pipeline", "generate CI/CD documentation"

**Depends on:** `pipeline-plantuml`

---

## How to Use a Skill

Skills are loaded into Claude Code via the MCP skills server or by referencing the skill directory in your project's Claude configuration. Once loaded, Claude will automatically invoke the appropriate skill based on trigger phrases, or you can invoke one explicitly:

```
/gitlab-cicd-docs
/pipeline-plantuml
/perform-mr-review
```

If a skill depends on another (noted in the skill entry above), both must be available in your configuration.

## Contributing

### Adding a Skill

1. Create a directory at the root named after the skill (use lowercase kebab-case, e.g. `my-skill`).
2. Add a `SKILL.md` using this structure:

   ```markdown
   ---
   name: <skill-name>
   description: >
     <When to invoke — include use cases and trigger phrases.>
   ---

   # Skill Title

   Brief description of what the skill does.

   ## Workflow

   Step-by-step instructions Claude will follow.

   ## Quality Rules

   - Constraints and output standards
   ```

3. Update this README with the skill's purpose, trigger phrases, and any dependencies.

### Guidelines

- **Name clearly:** the skill name should reflect what it does, not how it works.
- **Trigger phrases:** include natural language phrases a user might say so Claude knows when to invoke the skill automatically.
- **Dependencies:** if your skill relies on another skill, call that out explicitly in the `SKILL.md` body and in this README.
- **Output paths:** use `<output_dir>/` as a placeholder for any files the skill produces — never hardcode paths.
- **Scope:** one skill, one responsibility. If a skill is growing complex, consider splitting it.
