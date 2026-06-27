---
name: planning-spec
type: agent-guide
description: Plan a non-trivial change before breaking it into steps — draft the approach against the project's durable principles, name what you will deliberately NOT do, and get an explicit human "go" on the plan before fanning it out into tasks. ALWAYS apply when a request is large enough to need more than one step, touches multiple files or modules, or is open-ended ("add X", "support Y", "rework Z"). Skip for a one-line fix, a question, or a change whose shape is already fully decided. Redirecting a plan is cheap; redirecting after the work has fanned out is not.
---

# Skill: planning-spec

## Purpose

Left to default behavior, an agent jumps straight from a request to a task list and starts editing. That
fails in three quiet ways: it plans in a vacuum (ignoring how *this* project is already built), it never
states what it won't do (so scope drifts as the work proceeds), and it commits to an approach no human
ever approved (so a wrong direction is discovered only after the work has multiplied into a dozen edits).

This skill inserts a planning stage with three checkpoints the default skips: **check the approach against
the project's durable principles**, **write the out-of-scope line**, and **stop for a human "go" before
the plan becomes a task breakdown**. The leverage is timing — a plan is a paragraph you can rewrite in
seconds; a fanned-out breakdown with half the edits done is expensive to unwind.

## When a plan is worth it

Plan when the change is open-ended or spans more than one file/module/step. Skip the ceremony for a
one-line fix, a lookup, or a change whose shape is already pinned down — a plan that only restates an
obvious task list is pure overhead (see Gotchas).

## The shape

### 1. Clarify intent before drafting

State the goal in your own words and surface the ambiguities *now*: what does "done" look like, who is
the user, what is the one outcome this change must produce. An assumption you smuggle past this step
becomes a wrong plan you discover three checkpoints later. If a load-bearing answer is missing, ask —
don't guess and build on the guess.

### 2. Draft the approach — checked against the project's durable principles

Write the *approach*, not yet the steps: which existing module gets extended, which boundary is
respected, what the data flows through. Then check that draft against the project's **persistent
principles** — the durable constraints that govern how this project is built, not preferences invented
fresh for this change.

Look for those principles where the project records them: an `AGENTS.md` / `CONTRIBUTING.md`, an
architecture or decision-record directory (`docs/adr`, `decisions/`), a `CLAUDE.md`, or a stated set of
conventions. Validating a plan against the project's enduring principles before writing it keeps the
plan constrained by those rules, so it stays consistent with the rest of the codebase instead of
reintroducing a pattern the project deliberately moved away from. If the project records no such
principles, infer them from the prevailing structure —
and say which ones you inferred, so the human can correct you at the "go".

### 3. State what this will deliberately NOT do

Write the out-of-scope line explicitly: the adjacent things a reader might reasonably expect this change
to cover, that it will *not*. Scope creep enters through the omission, not the inclusion — an
unmentioned boundary is one the work silently crosses later. Naming the non-goals up front is the
out-of-scope discipline; it also gives the human something concrete to push back on ("no, the migration
*should* be in scope") while
the change is still a paragraph.

### 4. Stop for an explicit human "go"

Present intent + approach + out-of-scope and **wait for explicit approval before breaking the plan into
steps**. This human checkpoint between plan and breakdown is the gate: the plan is the cheapest artifact
to redirect, and once
it fans out into a task list with edits underway, every correction costs more. "Looks fine, proceed" is
approval; silence is not. If you cannot get a human in the loop, say so and treat the plan as
provisional rather than settled.

### 5. Only then break it into steps

After "go", and not before, decompose the approved approach into ordered, individually checkable steps.
The breakdown inherits the approved approach and the out-of-scope line — it does not get to quietly
expand them. If decomposition reveals the approach was wrong, that is a *new* plan: go back to step 2,
not forward into the wrong steps.

## Gotchas

- **Planning in a vacuum.** The plan is internally coherent but ignores a convention the project already
  settled — it reintroduces a discarded pattern, or extends the wrong module. The plan reads fine in
  isolation and only collides with reality at review. The fix is step 2: read the project's recorded
  principles *before* drafting, not after.
- **No out-of-scope line, so the plan silently grows.** Without a stated boundary, every "while I'm here"
  is defensible, and the change quietly absorbs the refactor next door. The absence of the non-goals list
  is itself the failure — it looks like nothing is wrong because nothing was written down to violate.
- **Skipping the checkpoint and fanning out a wrong plan.** The agent reads "plan, then implement" as one
  motion and never actually pauses for a human. The cost is invisible until the wrong direction surfaces
  with twelve edits already made — exactly the expensive case the checkpoint exists to prevent.
- **A "plan" that is just a restated task list.** "Step 1: edit the parser. Step 2: add a test." names
  *what to touch* but never states the *approach* (why this boundary, what flows where) — so there is
  nothing for the principle-check or the human to actually evaluate. A plan with no approach is a
  to-do list wearing a plan's name; the checkpoints pass vacuously and catch nothing.
- **Treating inferred principles as confirmed.** When the project records no constitution and you infer
  the conventions from the code, an inferred rule can be a coincidence, not a rule. Flag what you
  inferred at the "go" so a human can correct it — an unflagged inference hardens into a constraint
  nobody chose.

## What does not belong

- Decomposing into steps before the approach has a human "go" — that is the failure this skill exists to
  prevent.
- Inventing per-change conventions when the project already records durable ones — defer to the recorded
  principles; propose a change to them separately, don't quietly override them in one plan.
- A plan with no out-of-scope line, or a plan that is only a task list with no stated approach.
- The implementation itself — this skill ends at the approved, decomposed plan; carrying out the steps is
  separate work.

## Pairs with

- [`adversarial-review`](../adversarial-review/SKILL.md) — review binds findings to the *stated intent*;
  a plan with an explicit approach and out-of-scope line is what gives that review something precise to
  bind against.
- [`concise-output`](../concise-output/SKILL.md) — present the plan at the "go" checkpoint lean and
  scannable (intent · approach · out-of-scope), so the human can approve or redirect cheaply.
