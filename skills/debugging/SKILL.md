---
name: debugging
type: agent-guide
description: Find a defect's root cause from observed runtime evidence, not a guess — reproduce it reliably, isolate it to the smallest failing surface, instrument and read the real values, then verify the fix kills the repro and you know why. ALWAYS apply when chasing a bug, crash, wrong output, regression, or "works on my machine" discrepancy where the cause is not yet pinned to a line. Do not propose a fix on a guessed cause, edit code to test a hunch, or remove the reproduction before you understand why it fired. Skip when the cause is already located and proven (just write the fix) and when stabilizing a flaky, non-deterministic test (that is its own discipline).
---

# Skill: debugging

## Purpose

The default failure mode in debugging is the confident story. You read a stack trace, form
"it must be the null check in the parser", and start editing — before any execution evidence says
the parser is even on the failing path. Coding agents are especially prone to this: the first
plausible cause *reads* like a conclusion, and the edit *feels* like progress. It usually isn't.
You fix a symptom, the bug moves, and now you have two.

This skill installs a **gate**: you do not commit to a root-cause conclusion, and you do not
propose a fix, until **observed execution data localizes the cause to a line and a state**. A guess
is a hypothesis to test against the running program, never a conclusion to act on. The discipline is
a 4-phase systematic loop — reproduce, isolate, identify, verify — grounded in live runtime evidence.
It works in any repo and any runtime; nothing enforces it at edit time, so you hold yourself to it.

If the cause is already proven to a line, you are past this skill — write the fix. If the failure
is intermittent (green sometimes, red sometimes), that is non-determinism, a different discipline —
load `fix-flaky-test`.

## The gate

> No fix is proposed on a guessed cause. A conclusion of the form "the bug is X at file:line"
> requires runtime evidence — a value you *read* from the running program — that puts X on the
> failing path with the failing state. Until then, every "it must be X" is a hypothesis, and the
> only move is to test it against execution.

Why this is the whole game: the cheapest-feeling debugging is reading code and reasoning to a
cause. It is also the least reliable, because the bug is usually where your model of the code is
wrong — exactly the place reasoning can't reach. Execution data is the one source that doesn't
share your wrong assumption.

## The loop

### 1. REPRODUCE — get a reliable trigger first

Build a command, input, or test that makes the failure happen on demand. Run it; paste the failing
output. A repro you can fire at will is the instrument every later phase reads from — without it you
are guessing twice (about the cause *and* about whether you fixed it).

If you **cannot** reproduce it, the discrepancy is itself what you've learned, not a dead end. The
"fails in CI, passes locally" / "the user sees it, I don't" gap points at an environment, version,
data, or timing difference. Hunt *that* gap (the differing input, the differing dependency
version, the differing config) rather than concluding the bug isn't real.

### 2. ISOLATE — narrow to the smallest failing surface

Shrink the repro until almost nothing is left and it still fails. Bisect: halve the input, disable
half the steps, `git bisect` across commits to find the one that introduced it, comment out
branches. Each halving either keeps the failure (the cause is in this half) or loses it (the cause
was in the other half). Stop when one more cut makes the failure disappear — the cause lives at
that boundary.

### 3. IDENTIFY — instrument, run, read the real values

Now place **temporary probes** — log statements, a debugger breakpoint, an assertion — at the
boundary isolation found, and *run again*. Read the actual runtime values: what is this variable
*really*, which branch *actually* executes, what is the real call order. The conclusion comes from
what the probe prints, not from what you expected it to print. The gap between the two is the bug.

Remove every probe once the cause is located. They are scaffolding, not the fix (see Gotchas).

### 4. VERIFY — the fix kills the repro *and* you understand why

Apply the fix, then run the phase-1 repro again: it must now pass. That is necessary but not
sufficient — also state, in one sentence, the causal chain from the fix to the observed value
change ("the timeout fired because the retry reset the deadline; resetting it inside the loop
keeps it bounded"). A fix that makes the repro pass but you can't explain may be masking the
symptom while the cause survives. Where the project has a test suite, bind a test that fails before
the fix and passes after — that is the durable proof the bug is dead, not merely quiet.

## Gotchas

- **Hypothesis-first editing.** You change code to *test* a guess — "let me just add a null check
  and see". The edit mutates the very state you are trying to observe, so a pass now tells you
  nothing about the original cause, and you may have papered over it. Observe first with a
  read-only probe; edit only to apply a fix to a *located* cause.
- **Fixing the symptom, not the cause.** The traceback points at line 200 where the `None`
  dereferences; you guard line 200. But the `None` was *produced* at line 40 by the real bug, which
  is still live and will resurface elsewhere. Trace upstream to where the bad value is *born*, not
  where it finally crashes.
- **Removing the repro before you understand why it fired.** The failure stops, so you call it
  fixed — but you changed three things and don't know which one mattered, or it stopped for an
  unrelated reason (a cache cleared, a race shifted). A repro that goes quiet without an
  explanation is not a fixed bug; it is a bug you've lost track of.
- **Leaving debug probes in.** The `console.log`/`print`/breakpoint that found the cause ships in
  the diff. At best it's noise; at worst it leaks data to logs, slows a hot path, or trips a linter
  in CI. Grep your diff for the probes before you hand off.
- **Trusting the first green as proof.** The repro passes once after the fix and you stop. If the
  original failure was rare or timing-dependent you've proven nothing — but note: if it's
  *intermittent*, you're in flaky-test territory, not here. For a deterministic bug, one faithful
  repro run is enough; just make sure it's the *same* repro from phase 1.

## What does not belong

- A proposed fix whose justification is "this is probably it" with no value read from a run.
- Editing production code as the *first* probe instead of an observation-only instrument.
- Bundling neighboring cleanups or refactors into the fix — they dilute the evidence that *this*
  change killed *this* bug.
- Diagnosing a flaky, non-deterministic failure here (no reliable repro to read) — that is
  stabilization, not debugging.

## Pairs with

- [`empirical-proof`](../empirical-proof/SKILL.md) — paste the verbatim repro output and the
  post-fix run; this skill says *gather* the runtime evidence, that skill governs *how you record*
  it.
- [`fix-flaky-test`](../fix-flaky-test/SKILL.md) — the sibling discipline for the intermittent
  case, where the first job is to *make* a repro reliable before any of the steps here apply.
- [`adversarial-review`](../adversarial-review/SKILL.md) — its refute-by-default stance, turned on
  your own diagnosis: assume your favorite cause is wrong until the runtime evidence forces it.
