---
name: adversarial-review
type: agent-guide
description: Review another agent's work refute-by-default — the best-in-class form. ALWAYS apply when reviewing a branch, a PR, a revision after a prior review, an audit you are deepening, or a bug you are root-causing, even if it looks correct. Run the project's validation YOURSELF first — never sign off on the worker's pasted output. As a panel (the strong form): a review lead runs three or more INDEPENDENT reviewers, each one distinct lens, who draft before they compare. You draft facts + a human-attention list; you NEVER issue the verdict (Pass/Fail) — a human or an independent reviewer owns it. Skip for original authoring (a spec, research, audit, bug report — nothing exists yet to refute) and for writing the fix itself (that is a new task).
---

# Skill: adversarial-review

## Purpose

Reviews fail when the reviewer accepts the worker's framing. Set the worker's claims aside, read the
code with fresh eyes, and **run the validators yourself**. The same hostility-to-plausible-explanations
applies whether you are reviewing a branch, deepening an audit, or hunting a bug. You produce *findings
+ evidence*; the verdict is not yours.

**Before you start, copy [`references/task-template.md`](./references/task-template.md) into your task
file** — it is the session frame; fill it in as you go.

## Solo, or a panel of three lenses

- **Solo** — one reviewer, all the rules below, the six questions answered explicitly.
- **Panel (the strong form, Corpus [ADR-0099](https://github.com/jcosta33/corpus/blob/main/docs/adrs/0099-review-orchestration-and-role-routing.md)):**
  a **review lead** runs **three or more independent reviewers**, each taking **one distinct lens** —
  *requirement correctness · verification/evidence/repro · maintainability/design risk* (add security,
  migration-safety, performance, data-integrity, concurrency as the change warrants). The lead
  reconciles, dedupes, and writes the one packet. Diverse lenses beat redundant reviewers because
  **agreement is not a correctness signal** — models err in correlated ways, and the correlation grows
  with capability ([\[48\]](../../docs/sources.md#48)).

### Two panel rules that make it work

- **Draft before you compare.** Each lens drafts its findings *before* seeing the others — independence
  first, comparison second. Revealing early collapses the panel into one correlated opinion ([\[48\]](../../docs/sources.md#48)).
- **Vote, don't debate; refute first.** Resolve a disagreement by each reviewer **independently
  re-checking** and then **voting**, not by argument — debate propagates the shared error. On a contested
  finding the skeptic writes the **refutation first** (why the claim is unproven); the finding survives
  only if the refutation fails against fresh evidence.

## Project context (the AGENTS.md contract)

Resolve the project's commands from the consuming repo's `AGENTS.md > Commands` table — `cmdTest` /
`cmdLint` / `cmdBuild` / `cmdTypecheck`, plus any project validation row. If a slot is missing or
undefined, **ask** — do not guess a command.

## Core procedure

### 1. Bind to the requirements first

Read the spec / acceptance criteria / task scope **before** the diff. A review unbound from requirements
both over-corrects on style and misses spec violations. Every finding later maps to a requirement id or
is explicitly out-of-scope.

### 2. Run validation yourself

Run install / validate / test in **your own worktree**, the branch checked out. The worker's pasted
output is evidence they ran it *once*, not that it passes *now*. Re-running is the lever — substantive
participation, not a sign-off, is what review buys ([\[49\]](../../docs/sources.md#49)).

### 3. The six adversarial questions (answer each explicitly)

1. **What was the intent?** State it in your words. If you can't, you haven't read enough.
2. **Does the code do it?** Point at the lines that address each in-scope requirement.
3. **What didn't change that should have?** Renamed types, callers, tests, docs — the bug is often in
   *unchanged* code.
4. **What edge cases are unhandled?** Empty / max / concurrent / partial-state / unicode / time-zone.
5. **What production failure modes are possible?** Network, races, resource exhaustion, retry storms.
6. **What was claimed but not verified?** Comb the worker's notes for "should never", "harmless", "by
   happy accident" — confessions of unverified assumptions.

### 4. Cross-module caller search

For every changed public surface, grep for callers and read the calling code, not just the changed
module: `git grep -n '<changed-symbol>' src/ tests/`. Paste the output or summarise the call-site count.

### 5. Every finding carries evidence and a false-positive risk

A finding is: **severity** (BLOCKER / MAJOR / MINOR) · **file:line** · **the specific issue** ·
**the evidence** (the diff line, the re-run output, the caller) · **an FP-risk** (high / medium / low) ·
**a fix sketch**. The FP-risk is not optional: a reviewer who floods low-confidence flags gets ignored,
so a finding that surfaces noise is worse than one that stays quiet — keep effective false positives low
([\[51\]](../../docs/sources.md#51)). Vague concerns are not findings; sharpen to file:line or drop.

### 6. Mistrust confident language; don't trust the diff to be the work

"Should never happen", "harmless", "unlikely to fire" are assumptions to investigate, not assurances. A
diff that touches 3 files when the spec called for 8 is *evidence* something was missed.

## Output — economical

Lead with the finding and its evidence; file:line throughout; no soft language ("maybe consider possibly
…"), no praise. Justify a finding to make the reader's check **cheap**, not to convince — long persuasive
prose raises trust without raising scrutiny ([\[52\]](../../docs/sources.md#52)). Pairs with the
[`concise-output`](../concise-output/SKILL.md) skill (Corpus ADR-0109).

## The verdict is not yours

You draft **facts + a human-attention list** — never Pass / Fail / approve / merge, never `status: pass`,
never close a task. A human or an independent reviewer owns the result (Corpus ADR-0077 D8). And **never
review your own implementation** — an author scoring their own change cannot be trusted to disagree with
it (ADR-0056).

## Pairs with

- [`persona-skeptic`](../persona-skeptic/SKILL.md) — the refute-by-default *stance* (the attitude); this
  skill is the *procedure* (the steps + the panel). Load both for a hard review.
- `corpus-reviewer` (the corpus-agents catalog) — the runnable, fresh-context projection of this style.

## What does not belong

- Style preference as a finding (unless it violates a project convention); soft-language findings;
  findings without a fix sketch; approving on the worker's claims without re-running validation;
  inheriting the worker's framing; demoting a finding to dodge blocking the worker.

## Bundled resources

- [`references/task-template.md`](./references/task-template.md) — the fillable review session frame:
  diff overview, the findings table (severity · file:line · evidence · FP-risk · fix), suggested
  decision, and a self-review hard gate. Copy it in, substitute the project commands, fill it as you go.
