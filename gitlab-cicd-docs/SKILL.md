---
name: gitlab-cicd-docs
description: >
  Generate CI/CD pipeline documentation from a .gitlab-ci.yml file. Produces a structured Markdown document with pipeline overview, stages & jobs breakdown, and PlantUML diagrams (pipeline flow, job dependency graph, sequence diagram). Use this skill whenever the user wants to document a GitLab pipeline, understand a .gitlab-ci.yml file, or generate diagrams from a CI/CD config. Triggers on phrases like "document this pipeline", "create docs for gitlab-ci.yml", "explain this pipeline", "generate CI/CD documentation".
---

# GitLab CI/CD Documentation Skill

Reads a `.gitlab-ci.yml` file and produces a concise Markdown documentation file with embedded PlantUML diagrams.

## Workflow

### Step 1 — Locate and read the file

1. `reference/{project}/.gitlab-ci.yml` — canonical location in this repo
2. The path the user specifies
3. Current working directory

For large files (>500 lines), read in chunks focusing on: `stages:`, `workflow:`, `default:`, and all non-anchor job definitions (keys not prefixed with `.`).

### Step 2 — Parse the pipeline

| Element | How to find it |
|---|---|
| **Stages** | Top-level `stages:` list — order defines execution order |
| **Jobs** | Top-level keys that are not reserved words (`stages`, `variables`, `include`, `workflow`, `default`) and not anchor-prefixed (`.`) |
| **Job stage** | `job.stage:` (defaults to `test` if omitted) |
| **Triggers** | Infer distinct trigger types from `workflow:` rules and job `rules:` (e.g. feature branch, MR, main, release — but use whatever this pipeline actually has) |
| **Dependencies** | `job.needs:` (DAG), `job.dependencies:` (artifacts) |
| **Environments** | `job.environment:` (name, url) |
| **Artifacts** | `job.artifacts:` paths — omit if trivial |

Do NOT record docker image names.

### Step 3 — Generate diagrams

**Read the `pipeline-plantuml` skill first** — it handles encoding and URL generation.

1. **Pipeline flow diagrams** — one per distinct trigger type, named `flow-<trigger>.puml` (e.g. `flow-feature-branch.puml`, `flow-main.puml`). Derive trigger names and job sets from `rules:`. Skip a trigger if its job set is identical to another. Use the `struct` pattern from the `pipeline-plantuml` skill.
2. **Job dependency graph** (`job-dependency-graph.puml`) — driven by `needs:`; fall back to stage order if none.
3. **Sequence diagram** (`pipeline-sequence.puml`) — Developer → GitLab CI → stages in order → Environment (if a deploy stage exists).

Save all `.puml` files to `<output_dir>/diagrams/`.

### Step 4 — Write the Markdown document

Save to `<output_dir>/pipeline-docs.md` using this structure:

---

## Document structure

```markdown
# CI/CD Pipeline Documentation — {project}
> Generated from `.gitlab-ci.yml` · <date>

## Pipeline Overview

<2-3 sentences: what this pipeline builds/deploys and any key patterns worth knowing upfront>

---

<One section per trigger type, with its flow diagram:>

### Pipeline Flow — <Trigger Name>
![Pipeline Flow: <Trigger Name>](<url>)

<Repeat for each trigger>

### Trigger → Deploy Sequence
![Pipeline Sequence](<url>)

---

## Stages & Jobs Breakdown

### Stage: `<stage_name>`

> <one-line purpose>

| Job | Description | Artifacts |
|-----|-------------|-----------|
| `job-name` | One sentence | `path/` or — |

<Add a sub-section only for jobs with DAG needs, environment deploys, manual triggers, or non-obvious behaviour:>

#### `<job-name>`
- **Needs:** `upstream-job`
- **Key steps:** what it actually does — skip echo/cd boilerplate

<Skip stages with no jobs.>

---

### Job Dependency Graph
![Job Dependency Graph](<url>)

---

## Notes & Observations

<Only if genuinely noteworthy: unused stages, manual gates, required CI/CD variables, scaffolded-but-inactive paths>
```

---

## Quality rules

- Use exact job and stage names from the YAML
- Do not repeat trigger conditions in prose — the flow diagrams show that
- Do not include docker image names anywhere
- No `> 📎 Source:` captions under images
- `—` for absent table fields
- Job sub-sections are optional — only when the table row isn't enough
- Notes section is optional — only include it if it adds real value

## Output files

```
<output_dir>/
├── pipeline-docs.md
└── diagrams/
    ├── flow-<trigger>.puml     ← one per distinct trigger
    ├── job-dependency-graph.puml
    └── pipeline-sequence.puml
```
