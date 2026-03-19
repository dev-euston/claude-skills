---
name: pipeline-plantuml
description: Generate PlantUML diagrams for pipelines — pipeline flows, job dependency graphs, and sequence diagrams. Save .puml source files, render to PNG via PlantUML's public server, and embed the image in Markdown. Use this skill whenever a pipeline diagram is needed, or whenever another skill needs to produce a diagram as part of a larger document. Always use this skill instead of generating diagram syntax ad-hoc.
---

# Pipeline PlantUML Skill

Generates PlantUML pipeline diagrams, saves source files, renders via the public server, and returns Markdown-embeddable image syntax.

## Workflow

### 1. Write the .puml source

Save to `<output_dir>/diagrams/<diagram_name>.puml`. Always wrap in `@startuml` / `@enduml` and include a `title`.

### 2. Render via PlantUML public server

Run this Python snippet via Bash to encode and get the URL:

```python
import zlib, base64, string

def plantuml_encode(text):
    compressed = zlib.compress(text.encode('utf-8'))[2:-4]
    std = base64.b64encode(compressed).decode('ascii')
    table = string.ascii_uppercase + string.ascii_lowercase + string.digits + '+/'
    puml  = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz-_'
    return ''.join(puml[table.index(c)] if c in table else c for c in std)

with open('<path_to_puml_file>') as f:
    content = f.read()
print(f"https://www.plantuml.com/plantuml/png/{plantuml_encode(content)}")
```

### 3. Embed in Markdown

```markdown
![<Diagram Title>](<rendered_url>)
```

---

## Diagram patterns

### Pipeline flow (trigger-based)

Use `struct` blocks for stage groupings and `abstract` nodes for each trigger. Add `top to bottom direction` so chains render side by side. Always use `as alias` on structs — never use hyphenated names directly in arrows. Use `--` inside structs to divide job groups.

Infer which jobs and stages appear per trigger from the pipeline's `rules:` — do not assume a fixed set of triggers.

```plantuml
@startuml
top to bottom direction
title Pipeline Flow

abstract "Trigger A" as A
abstract "Trigger B" as B

struct "stage-1" as a_s1 {
  job-x
  job-y
}
struct "stage-2" as a_s2 {
  job-z
}
A --> a_s1
a_s1 --> a_s2

struct "stage-1" as b_s1 {
  job-x
}
struct "stage-2" as b_s2 {
  job-z
  job-w
}
B --> b_s1
b_s1 --> b_s2
@enduml
```

### Job dependency graph

Use `(job)` nodes with `-->` arrows from `needs:` relationships. Label arrows with the stage name.

```plantuml
@startuml
title Job Dependency Graph

(job-a) --> (job-b) : stage-name
(job-a) --> (job-c) : stage-name
(job-b) --> (job-d) : next-stage
@enduml
```

### Sequence diagram (trigger → deploy)

Show the actor, then each stage as a participant in execution order.

```plantuml
@startuml
title Pipeline Sequence: Trigger to Deploy

actor Developer
participant "GitLab CI" as CI
participant "Stage A" as A
participant "Stage B" as B
participant "Environment" as Env

Developer -> CI : git push / MR
CI -> A : trigger
A -> B : on success
B -> Env : deploy
Env --> Developer : complete
@enduml
```

---

## Quality rules

- Give every diagram a descriptive `title`
- Match node labels exactly to job/stage names from the source
- Use `-->` (dashed) for conditional flows, `->` (solid) for always-runs
- No `> 📎 Source:` captions in Markdown — just the image
