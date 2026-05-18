---
name: pattern-audit
description: Use BEFORE writing code for any feature that has existing analogs in the codebase — "add another X like the existing ones", "create a new <thing> like <sibling>", "extend the <Y> pattern", "implement another worker/endpoint/loader/component/route matching the others", "add a new dropdown/loader/handler/block". Locates 2-3 sibling implementations, names the one to mirror, and surfaces non-obvious shared touchpoints (caches, registries, build steps, fixtures) the new code must also wire into. Skip only for truly greenfield work, pure questions, or cleanup with no siblings.
---

# Pattern Audit

## Overview

New code should look like the code that already exists for the same role. When an agent invents a divergent shape for something the codebase has already established — a slightly different loader signature, a hand-rolled dropdown when siblings share a component, a fresh cache when one already exists — the result is two rounds of correction and a quiet bug where the new code missed the shared touchpoint the siblings rely on.

**Core principle:** Mirror before you write. The cheapest moment to discover the pattern is *before* the first line of code, not after the review.

**Announce at start:** `"I'm using the pattern-audit skill to identify the reference pattern."`

## When to Use

Use when the requested work has obvious siblings already in the codebase:

- "Add another X like the existing ones"
- "Create a new <loader/endpoint/worker/block/component> — sibling to <Y>"
- "Extend the <Z> pattern with..."
- "Implement <thing> matching how the others do it"
- Behavior modifications where existing examples constrain the shape

## When NOT to Use

- Truly greenfield work — first of its kind in the codebase
- Pure questions, explanations, or research
- Pure cleanup (formatting, deleting dead code)
- Refactoring an isolated module with no siblings to mirror

## The Procedure

You MUST complete all six steps before writing code.

### 1. Identify the target role

State in one line what the new code *is* — its role, not its behavior. Examples: "a LoRA loader node", "a dropdown for the toolbar", "a POST endpoint on the workers API", "a BlockFlow block".

### 2. Find sibling implementations

Use Grep and Glob to locate 2-3 existing analogs. Search by role-shaped terms (class suffixes, directory names, registration calls, decorator names) — not just by literal name. If the first search returns zero, search harder before concluding none exist.

### 3. Audit each candidate

For each sibling, capture:

- **Path + key entry point** (file and function/class)
- **Pattern it follows** — structure, naming, dependencies, conventions
- **Non-obvious idioms** — shared caches, registries, module-load side effects, decorator-based registration, test fixtures it depends on

Read the candidate files. Skimming filenames is not auditing.

### 4. Name the reference

Pick the *one* sibling the new code will mirror. State why — most recent, most idiomatic for the layer, closest shape to the new requirement. A single named reference beats an averaged "common pattern".

### 5. Surface non-obvious touchpoints

This step is where the bugs live. List every shared resource the chosen reference touches that the new code must also touch:

- Caches (shared in-memory, on-disk, model)
- Registries (decorator-based, module import side effects, manifest files)
- Build steps, codegen, schema regeneration
- Test fixtures, snapshots, factories
- Shared components (dropdowns, error wrappers, base classes)

The canonical failure mode this skill exists to prevent: a new sibling that *looks* right but bypasses the shared cache/registry the other siblings all wire into. Do not skip this step.

### 6. Record the audit

For work tracked in a bead, write the audit to the bead's `design` field. For trivial untracked work, inline it in the conversation under a `## Pattern audit` heading. Use this fixed template:

```
## Pattern audit
Role: <new code's role>
Candidates:
  - <path>: <pattern summary>
  - <path>: <pattern summary>
  - <path>: <pattern summary>
Mirroring: <chosen path> — <why>
Non-obvious touchpoints: <caches / registries / build steps / test fixtures>
```

## The Hard Rule

```
NO CODE UNTIL THE AUDIT IS RECORDED
```

If a good-faith search finds no analog, state it explicitly: *"no analog found in codebase — proceeding greenfield with <stated approach>"*. For non-trivial work, get explicit acknowledgment from your human partner before proceeding.

## Red Flags — STOP

If you catch yourself thinking:

- "I'll just write it and see what fits" — write the audit first
- "It's obvious, no audit needed" — 30 seconds of grep proves or disproves that
- "Audit produced 0 candidates" without an explicit search trail — search harder before declaring greenfield
- "I'll skip the touchpoints step" — caches and registries are exactly where the bugs come from; this step is the reason the skill exists
- "I'll audit after the first draft" — the first draft sets the shape; auditing after is rework

## Integration With Other Skills

- **`brainstorming`** captures *intent*; pattern-audit captures *shape*. If brainstorming is already loaded for this work, embed the audit inside its design step rather than running standalone.
- **`verification-before-completion`** requires citing the reference pattern this audit named — the audit produces the artifact that verification later checks against.
- **`writing-plans`** — when the plan covers multiple new siblings, run the audit once per role and record each block in the plan.
