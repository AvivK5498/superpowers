---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to — hybrid bead/file model:**

The bead created during brainstorming/grilling is the plan's home.

- **Small/medium work** → the plan lives entirely in the **bead**: a lean task list (the dependency DAG) plus the File Structure and interfaces, recorded in the bead's `description`/`notes`. No full code blocks — the executor curates per-task code detail at dispatch time.
- **Large work** (heuristic: more than ~5 tasks, OR tasks with heavy per-task code detail that won't sit cleanly in a bead field) → the lean plan still lives in the bead, PLUS a separate plan file at `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md` carrying the heavy code detail. The bead references the file.
- (User preferences for plan location override the file default.)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

### Dependency DAG

Tasks are not an implicitly sequential list — the plan must produce an explicit **dependency graph**. For each task, record what it depends on (`Depends on: Task N` / `Depends on: none`). Tasks that share no dependency chain are independent and can run in parallel; the plan must make that parallelizable structure visible, not bury it in list order. The executor uses the DAG to dispatch independent tasks concurrently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header** (in the bead's `description`, or at the top of the large-work plan file):

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

The format below is the **full** large-work format (the plan file). For a **bead-resident lean plan**, keep the `Files:`, `Depends on:`, and the step list, but drop the code blocks — name the test and the behavior in prose; the executor curates the actual code at dispatch time.

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Depends on:** Task M (or `none` — independent, parallelizable)

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

## TDD and the External-Resource Carve-Out

The TDD steps above (write failing test → run it fail → implement → run it pass) are the **default** for every task.

Exception: when a task's test would require an **external resource** — GPU hardware, a paid or rate-limited API, real credentials that cost money, human visual/subjective confirmation, or infra that can't be faked without testing the fake — the plan must **FLAG that test for the user to decide on** before it's built. Mark the step clearly, e.g. `- [ ] **Step 1: Write the failing test** ⚠️ NEEDS USER DECISION — test requires <resource>`.

If the user declines the real-resource test, the plan specifies a **mocked test** instead: mock the boundary (the GPU/API/renderer), not the logic under test. The task still follows TDD against the mock.

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- In a large-work plan file: complete code in every step — if a step changes code, show the code. In a bead-resident lean plan: name the behavior and let the executor curate the code at dispatch time
- Exact commands with expected output
- Every task records its dependencies (the DAG)
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan, hand off to execution:

**"Plan complete — lean plan in bead `<id>`** (large work: plus `docs/superpowers/plans/<filename>.md`)**. Executing via subagent-driven-development."**

- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review
- The executor reads the dependency DAG and dispatches independent tasks in parallel
