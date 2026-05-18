---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Both axes required — reference + run

The Gate Function above proves the code *runs*. It does NOT prove you built the *right thing*. A completion claim requires **both** axes. One without the other is a lie of omission.

```
EVERY "done" / "complete" / "fixed" / "passing" claim must cite:

  AXIS 1 — NAMED REFERENCE (what proves it's the right shape)
  AXIS 2 — EVIDENCE OF RUN (what proves it works end-to-end)

Missing either = not a completion claim, a guess.
```

**Axis 1 — Named reference.** State explicitly what the work was compared against. One of:

- A sibling pattern file the new code mirrors — name the path (e.g., `src/loaders/lora_loader.py`)
- A spec or requirement section the work satisfies — name the section
- A bead requirement / acceptance criterion — name the bead ID and which criterion
- A passing test that asserts the intended behavior — name the test file and test name

"It looks right" / "matches the pattern" / "follows convention" do NOT satisfy this. The reference must be **cite-able** — a path, an ID, a section heading. If the `pattern-audit` skill ran earlier in the session, the audit block it produced is usually the right citation source — this skill consumes that audit.

**Axis 2 — Evidence of run, integration-grade when integration is the risk.** The Gate Function's "run the command, show the output" still applies — and tightens. If the change touches an integration path (caches, registries, build pipelines, dynamic loaders, UI mounts, API routes), unit tests in isolation do NOT count as evidence. The smoke test must exercise the changed code end-to-end:

- Change touches a UI component → load the UI, reach the component, show what rendered
- Change touches an API → hit the endpoint, show the response
- Change adds a registered item (loader, plugin, route) → start the system, show the item appears in the registry it was added to
- Change touches a cache → trigger the invalidation path, show the cache reflects the new state

Output goes in the message verbatim. No paraphrase.

**Why both.** Reference without run = "I built the right shape but never proved it works." Run without reference = "It runs but might be the wrong thing." A loader that imports cleanly but never appears in the UI dropdown passes unit tests and fails the user. The pattern-audit names the sibling loader; the integration smoke test loads the UI and confirms the new loader appears AND the cache invalidates. Either axis alone misses one of those.

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |
| Right shape built | Named reference cited (path/bead-id/test-name) | "Matches the pattern", "looks right", memory of codebase |
| Integration change works | End-to-end smoke test exercising the changed path, output shown | Unit tests pass, "the code is correct", linter clean |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**
- Claiming "done" / "complete" / "fixed" without naming a cite-able reference (path, bead ID, section, test name)
- Claiming an integration change works on the strength of unit tests alone — no end-to-end smoke test for the changed path
- "Matches the pattern" / "follows the convention" with no path named
- Skipping the integration smoke test because "the unit tests cover it"

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |
| "The pattern is obvious, no need to name a reference" | Name it anyway — takes 5 seconds, prevents wrong-shape claims |
| "Unit tests pass, that's enough" | If the failure mode is integration (caches, registries, build steps, dynamic loaders, UI mounts), unit tests do NOT prove it. Run the integration smoke test. |
| "I compared against my memory of the codebase" | Memory isn't a citable reference. Open the file, cite the path. |

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## Why This Matters

From 24 failure memories:
- your human partner said "I don't believe you" - trust broken
- Undefined functions shipped - would crash
- Missing requirements shipped - incomplete features
- Time wasted on false completion → redirect → rework
- Violates: "Honesty is a core value. If you lie, you'll be replaced."

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
