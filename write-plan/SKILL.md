---
name: write-plan
description: Write a comprehensive, bite-sized implementation plan from a completed task analysis document. Use after analyse-task has produced its analysis. Trigger phrases: "write the plan", "create the implementation plan", "now plan it", "write a plan".
---

# Write Plan

Write a comprehensive implementation plan from the task analysis document. This skill runs as a subagent — the analysis document is your primary input. Read the codebase only when the analysis document references something you need to see and the benefit outweighs the cost of the read.

**Depends on:** `analyse-task` — the analysis document must exist before running this skill.

**Announce at start:** "I'm using the write-plan skill to create the implementation plan."

**Save plan to:** `<output_dir>/plans/YYYY-MM-DD-<feature-name>.md`

## Guiding principles

Assume the engineer executing this plan has zero context for the codebase and questionable taste. Give them everything they need: exact files, exact code, exact commands. DRY. YAGNI. TDD. Frequent commits.

The plan is the executor's sole context. Write it so an agent with only this document can implement the feature correctly, without reading the codebase for anything already knowable from the plan. Embed code, signatures, and file content directly when doing so saves the executor a read that would cost more than the space it takes. Skip embedding large files; reference them by path instead.

## File structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for.

- Each file has one clear responsibility.
- Prefer smaller, focused files over large ones.
- Files that change together should live together. Split by responsibility, not layer.
- In existing codebases, follow the patterns identified in the analysis document.

Lock in decomposition decisions here. Each task should produce self-contained, independently testable changes.

## Bite-sized task granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" — step
- "Run it to make sure it fails" — step
- "Implement the minimal code to make the test pass" — step
- "Run the tests and make sure they pass" — step
- "Commit" — step

## Plan document header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py` (lines 23-45 shown below)
- Test: `tests/exact/path/to/test.py`

<!-- For files being modified, inline the relevant existing code the executor
     needs to understand before making changes. Skip this for large files
     where only a line or two changes and the diff is self-explanatory. -->

**Existing code in `exact/path/to/existing.py` (lines 23-45):**
```py
# paste relevant excerpt
```

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the executor may read tasks out of order)
- Steps that describe what to do without showing how
- References to types, functions, or methods not defined anywhere in the plan

## Self-review

After writing the complete plan, review the analysis document with fresh eyes.

**1. Requirements coverage:** Can you point to a task for every requirement? List any gaps and add missing tasks.

**2. Placeholder scan:** Search for the red flags above. Fix every one.

**3. Type consistency:** Do types, method signatures, and property names in later tasks match what was defined in earlier tasks? A name mismatch is a bug.

Fix issues inline. No need to re-review after fixing.

## Output

After saving the plan file, output the full plan contents to the conversation.
