---
name: adversarial-review
type: agent-guide
description: "Review a change refute-by-default — a claim is unproven until evidence forces the opposite conclusion. The best-in-class form, for any repo. ALWAYS apply when reviewing a branch, a PR, a diff, a revision after a prior review, an audit you are deepening, a bug you are root-causing — even if it looks correct — and as the lens for ADVERSARIAL SELF-REVIEW on your own work before you declare it done. Run the project's tests yourself from a clean checkout first; treat confident prose, a green summary, and a small diff as starting points, never proof; cite file:line per finding. As a panel (the strong form): a lead runs three or more independent reviewers, each a distinct lens, who draft before they compare. You produce findings + evidence + a human-attention list; you do not own the ship/merge call — a human or an independent reviewer does. Skip when authoring the thing being reviewed (nothing exists yet to refute) and when writing the fix itself (that is a separate task)."
---

# Skill: adversarial-review

## Purpose

Reviews fail when the reviewer accepts the author's framing — the default behavior is to read a green
summary, a small diff, and confident prose as proof, and wave the change through. That is the exact path
a hole gets approved. This skill inverts the default: **refute by default** — a claim is unproven until
evidence forces the opposite conclusion. The lever is not the attitude, it is the evidence you re-run
_yourself_: a completion claim is worth only the checks you reproduce from a clean state, the output you
paste verbatim, and the finding you can pin to a `file:line`. Unaided doubt that grounds in no external
check changes nothing. The same hostility-to-plausible-explanations applies whether you are reviewing a
branch, deepening an audit, root-causing a bug, or turning the lens on your own work before handing off.
You produce _findings + evidence + a human-attention list_ — you do not decide whether it ships.

**Before you start, copy [`references/review-notes.md`](./references/review-notes.md) into your working
notes** — it is the session frame; fill it in as you go rather than reconstructing it from memory.

## Solo, or a panel of three lenses

- **Solo** — one reviewer, all the rules below, the adversarial questions answered explicitly.
- **Panel (the strong form)** — a **lead** runs **three or more independent reviewers**, each taking
  **one distinct lens**: _correctness · verification/evidence/repro · maintainability/design risk_ (add
  security, migration-safety, performance, data-integrity, concurrency as the change warrants). The lead
  reconciles, dedupes, and writes the one report. Diverse lenses beat redundant reviewers because
  **agreement is not a correctness signal** — models err in correlated ways, and the correlation grows
  with capability.

Two rules make a panel work:

- **Draft before you compare.** Each lens drafts its findings _before_ seeing the others — independence
  first, comparison second. Revealing early collapses the panel into one correlated opinion.
- **Vote, don't debate; refute first.** Resolve a disagreement by each reviewer **independently
  re-checking** then **voting**, not by argument — debate propagates the shared error. On a contested
  finding the reviewer writes the **refutation first** (why the claim is unproven); it survives only if
  the refutation fails against fresh evidence.

## Project commands

Resolve the project's commands from its `AGENTS.md` Commands table (or its README / CI config) — the
test, lint, build, and typecheck commands, plus any project-specific validation. If you can't resolve a
command, **ask** rather than guess one — a guessed command that "passes" is worse than no check, because
it greens on something unrelated and proves nothing.

## Core procedure

### 1. Bind to the stated intent first

Read the PR description / issue / stated requirements **before** the diff — a review unbound from intent
both over-corrects on style and misses real violations. Every finding later maps to a stated requirement
or is called out as out-of-scope. If you cannot restate in your own words what the change is supposed to
do, you have not read enough to judge it.

### 2. Run the checks yourself, from a clean checkout

Run install / test / lint in **your own clean checkout**, the branch checked out. The author's pasted
output is evidence they ran it _once_, not that it passes _now_ — it is the claim you are reviewing, not
its proof. Re-running is the lever: substantive engagement, not a sign-off, is what makes review catch
defects. Paste the output verbatim — last lines and exit status
included. Re-run from a fresh checkout, not the author's already-built artifacts or shell state, or it
"passes" for the wrong reason.

### 3. The adversarial questions (answer each explicitly)

Force these while judging; an unanswered one is a gap in the review, not a stylistic preference. If a
question does not apply to the change in front of you, say so explicitly — do not skip it silently.

1. **What was the intent?** State it in your own words. If you can't, you haven't read enough.
2. **What would falsify this?** Name the observation that would prove the claim false. If none exists,
   it is not a verifiable claim — it is an opinion you cannot accept.
3. **Does the code do it — for the _exact_ requirement?** For each stated requirement in scope, point at
   the lines that address _that_ one, not a neighbour, not the change in general. The first plausible
   match is how a hole gets approved.
4. **What didn't change that should have?** Renamed types, callers, tests, docs, cross-references — the
   bug is often in _unchanged_ code that needed updating.
5. **What edge cases are unhandled?** Empty / max / concurrent / partial-state / unicode / time-zone.
6. **What production failure modes are possible?** Network, races, resource exhaustion, retry storms.
7. **What was claimed but not verified?** Comb the author's notes for "should never", "harmless", "by
   happy accident", "edge case unlikely to fire" — each confesses an unverified assumption, not an
   assurance.
8. **Did the change alter behavior outside its stated scope?** Walk the diff for changes that trace to no
   stated requirement.

### 4. Cross-module caller search

For every changed public surface, grep for callers and read the calling code, not just the changed
module: `git grep -n '<changed-symbol>'`. Paste the output or summarise the call-site count and read each.

### 5. Required evidence — no evidence, no acceptance

Accept a claim only when its evidence is in front of you:

- **Checks you re-ran yourself, mapped to the stated requirements.** Re-run in your own checkout (§2);
  paste the verbatim tail + exit status; tie each to the requirement it proves.
- **The diff read directly.** `git diff` / `git status` read yourself; an empty or trivial diff, or one
  whose shape does not match the stated scope, is itself a finding.
- **Preservation evidence.** That the change did not break a constraint or invariant in scope — not
  merely that the new behavior works.
- **Every finding cites file and line.** A finding names a specific surface and a specific issue; a vague
  concern is not a finding. Sharpen to file:line or drop it.

### 6. Every finding carries evidence and a false-positive risk

A finding is: **severity** (BLOCKER / MAJOR / MINOR) · **file:line** · **the specific issue** ·
**the evidence** (the diff line, the re-run output, the caller) · **an FP-risk** (high / medium / low) ·
**a fix sketch**. The FP-risk earns the finding its place: a reviewer who floods low-confidence flags
gets tuned out, so a noisy finding is worse than silence — keep effective false positives low.
Vague concerns are not findings; sharpen to file:line or drop.

### 7. Mistrust confident language; don't trust the diff to be the work

"Should never happen", "harmless", "unlikely to fire" are assumptions to investigate, not assurances. A
diff that touches 3 files when the stated change needed 8 is _evidence_ something was missed — walk the
default questions on a small diff rather than skimming it through.

## The Refuses table

The default behavior is to accept these at face value. Each is a red flag with a fixed response.

| Red flag                                                                                      | Action                                                                                                          |
| --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Summary-only proof ("tests pass", "all green").                                               | Reject; demand the artifact — command, exit code, output.                                                       |
| "Tests passed" with no command, exit code, or output.                                         | Reject as unverified.                                                                                           |
| A claim that a requirement is met with missing or mismatched evidence.                        | Reject as unverified; the evidence must prove _that_ requirement.                                               |
| Acceptance resting on the worker's own pasted verification.                                   | Reject; re-run the checks yourself, then judge.                                                                 |
| The implementer recording the result on their own change.                                     | Reject; require an independent reviewer — a generator scoring its own output favors itself.                     |
| A judgment call with no recorded reasoning, or a judge tied to whoever produced the work.     | Reject; it cannot be trusted to disagree.                                                                       |
| Confident language ("harmless", "should never", "by happy accident") standing in for a check. | Reject; investigate the assumption, then judge on evidence.                                                     |
| A small diff skimmed and waved through.                                                       | Reject; walk the adversarial questions — small diffs hide subtle defects.                                       |
| "I can't reproduce; must be environment-specific."                                            | The discrepancy is itself a finding; do not dismiss it.                                                         |
| Schema-valid / well-formed output offered as proof of correctness.                            | Reject; shape is not truth.                                                                                     |
| A finding demoted in severity, or softened to "maybe consider", to avoid blocking.            | Reject the softening; optimizing throughput over correctness is the exact failure this skill exists to prevent. |
| Source files edited during a review.                                                          | Refuse; review judges, it does not repair. The fix is a separate task.                                          |

## Adversarial self-review at completion

This same refute-by-default stance is **also** the lens you turn on your _own_ work before marking it
done — because the default behavior is to ship your own green result unexamined, and turning the stance
on yourself first yields fixes you would otherwise hand off. Same questions, aimed at yourself: re-run
your own checks from a clean state, hunt the path you did not exercise, look for where your green result
could still be wrong. Then a fast pass over the review itself:

- **Did I re-run the checks myself, or lean on pasted output?** Confirm every check was re-run in a clean
  checkout with the verbatim tail + exit status recorded. If any acceptance rests on a paste, re-run it.
- **Does every finding cite file:line, and every accepted requirement map to evidence for _that_
  requirement?** Scan for vague concerns never sharpened, and for requirements waved through on a
  neighbour's evidence or a plausible first match. Sharpen or drop.
- **Did I soften anything to avoid blocking?** Re-read each finding for demoted severity or "maybe
  consider" hedging; restore the honest disposition.
- **Did I leave any default question silently skipped?** Each must be answered or explicitly marked not
  applicable.

The one boundary that holds: self-review **produces fixes and a recorded critique, never a self-issued
verdict.** It does not substitute for independent review, and an approval you issue on your own change is
inadmissible — an implementer scoring their own work cannot be trusted to disagree with it. Withholding
the verdict is about the _verdict_; it is not an excuse to skip attacking your own work first.

## Output — economical

Lead with the finding and its evidence; file:line throughout; no soft language ("maybe consider possibly
…"), no praise. Justify a finding to make the reader's check **cheap**, not to convince — long persuasive
prose raises trust without raising scrutiny. Pairs with the
[`concise-output`](../concise-output/SKILL.md) skill.

## You don't own the ship decision

You produce **findings + evidence + a human-attention list** — not an approve/merge/reject verdict. Why:
the person who can be accountable for shipping owns that call; a reviewer who self-issues "approved"
removes the human judgment the review existed to inform. A human or an independent reviewer owns the
ship/merge call. And **never review your own work** for a verdict — an author scoring their own change
cannot be trusted to disagree with it (this is the boundary the self-review subsection above respects).

## Gotchas

- **Re-running dirty.** You re-run the author's command but against their already-built artifacts or a
  branch with their uncommitted state — it "passes" for the wrong reason. Re-run from a clean checkout.
- **Pasted output mistaken for proof.** Accepting the author's paste instead of re-running — their paste
  is the claim under review, not its proof, and it only proves the command ran _at some past moment_.
- **"I can't reproduce" treated as case closed.** The discrepancy is itself a finding to chase down, not
  a reason to dismiss the concern.
- **Early reveal on a panel.** The three lenses share a thread before drafting, and all three anchor on
  the first reviewer's framing — you get one opinion in triplicate, not three (the correlated-opinion
  trap). Keep stage one private.
- **Severity laundering.** A BLOCKER gets filed as MINOR to avoid blocking the author. Severity is the
  consequence if it ships, not the social cost of saying so.
- **Diff tunnel vision.** You review only the changed lines; the defect is in an unchanged caller the
  diff never touched (question 4). Read the callers, not just the diff.
- **Guessed command.** No resolvable test command, so you assume one; it runs something unrelated and
  greens. A guessed check is negative evidence — ask instead.
- **Self-review collapsing into a self-verdict.** You attack your own work, find nothing, and quietly
  declare it approved. Self-review yields fixes + a critique; the ship call still belongs to someone else.

## Pairs with

- [`empirical-proof`](../empirical-proof/SKILL.md) — the verbatim-pasted-output discipline this skill
  leans on (re-run yourself; paste the tail; one verification per claim).
- [`concise-output`](../concise-output/SKILL.md) — the output-economy dial for the report you write.

(The former `persona-skeptic` stance is now folded into this skill — its spine is the refute-by-default
Purpose, the Refuses table, and the self-review subsection above. The `persona-skeptic` entry is a
redirect to here.)

## What does not belong

- Style preference as a finding (unless it violates a project convention); soft-language findings;
  findings without a fix sketch; signing off on the author's claims without re-running the checks;
  inheriting the author's framing; demoting a finding to dodge blocking the author; editing source files
  during the review (the fix is a separate task); issuing a verdict on a change you authored.
- Original authoring (writing the spec, the research, the audit, the bug report) — there is no claim yet
  to falsify; that needs the constructive stances.

## Bundled resources

- [`references/review-notes.md`](./references/review-notes.md) — the fillable review session frame: diff
  overview, the findings table (severity · file:line · evidence · FP-risk · fix), the cross-module caller
  log, a suggested recommendation (not a verdict), and a self-review gate. Copy it into your notes,
  substitute the project commands, fill it as you go.
