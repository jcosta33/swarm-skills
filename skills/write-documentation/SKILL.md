---
name: write-documentation
type: agent-guide
description: >-
  Implement a documentation task: write or update human-facing docs (README,
  tutorial, how-to, reference, explanation) — one frame held throughout, every
  example run as written, every behavior claim cited to file:line. ALWAYS apply
  when a task packet produces a doc a human (not an agent) reads. Never mix
  frames, hedge with should/might/could, ship unrun examples, or document
  beyond the task's requirements. Skip agent-facing material (guides,
  templates) and all code-changing kinds.
---

# Implement documentation

Documentation that hedges, ships examples that do not run, or contradicts the code is worse than no
documentation — it misleads, and the reader cannot tell. This guide adds the documentation
discipline on top of the base `implement-task` rules, carrying the Documentarian stance: the reader
is a human who has not read the code, arrived with one question, and the doc answers it. The stance
on its own is `../persona-documentarian/SKILL.md` (if installed); this guide is its execution for a
documentation task, so the load-bearing rules are restated here and the skill stands alone without
it. These are conventions the review packet inspects — nothing enforces them at edit time.

This guide is for docs humans read. Agent-facing material (guides, templates, workflow docs) is a
different audience with different conventions, and code-changing work belongs to the other guides
in this library.

Open [`references/task-template.md`](./references/task-template.md) as your working file before you
start: it scaffolds the frame, audience, and reader's question, the examples-to-verify table, pasted
evidence, and the self-review, filled as you go. The task packet itself uses the kit's task template.

## Rules

1. **Fix the frame, the audience, and the question before writing a word.** Exactly one frame:
   **tutorial** (linear, hand-holding, no choices), **how-to** (a recipe for one task), **reference**
   (exhaustive lookup, no narrative), or **explanation** (the why). Name the audience concretely —
   "developers" is not specific; "developers integrating our SDK for the first time" is — and the
   one question the doc answers, stated as the reader would ask it. Discovering mid-doc that you are
   switching frames means it is two docs: split it.
2. **Lead with what the reader needs to do.** The first ~100 words contain the action the reader's
   question asks about — not project history or "before we begin". A reader scanning for an answer
   abandons a doc that buries the action under throat-clearing.
3. **Run every example; capture the output before writing it down.** Execute each example exactly
   as the reader would — no implied setup, no missing imports. An example you did not run is a
   hypothesis, not an example, and it is the most common way documentation lies. Paste the captured
   run as your evidence.
4. **Cite every behavior claim to file:line.** A statement about how the system behaves is
   verifiable against the code — cite the line. If you cannot find the line, the claim is suspect:
   verify it before writing it, or drop it. An uncited behavior claim is indistinguishable from a
   guess, and will be wrong the next time the code moves.
5. **No hedging the reader cannot act on.** "Should", "might", "could" leave the reader unable to
   decide. Either the system does X or it does not; if behavior is conditional, state the
   condition.
6. **Describe requirements as they are.** If the code contradicts what the spec says the doc must
   describe, surface the discrepancy as a blocked question — do not paper over it in prose.
7. **Reconcile the docs this change touches — within the task's scope.** Grep for other docs
   describing the same area; a doc you may edit that now contradicts yours gets reconciled in this
   change, and a contradiction in a doc outside your scope is a finding candidate. A stale doc
   contradicting a fresh one is worse than no doc: the reader cannot tell which is current.
8. **Run the project's format and doc-lint commands on every touched file and paste the output.**
   Resolve commands from `AGENTS.md`; ask when one is undefined. And never write a review result on
   your own work.

## Refuses

| Temptation                                       | Do instead                                                |
| ------------------------------------------------ | --------------------------------------------------------- |
| An example pasted in without running it          | Run it as written; paste the captured output              |
| "The system does X" with no file:line            | Find the line and cite it, or drop the claim              |
| "Should" / "might" / "could"                     | State the behavior, or the condition under which it holds |
| Mixed frames in one doc                          | Pick one; split the doc if it drifts                      |
| Choices in a tutorial ("you could also try…")    | A tutorial is linear; alternatives go in a how-to         |
| Narrative in a reference ("first, we see that…") | A reference is lookup                                     |
| "While I'm here" polish of neighboring docs      | Finding candidate; document only what the task names      |
| A doc-vs-code discrepancy smoothed over in prose | Blocked question; the spec changes through its own review |

## Self-review gate

Before declaring the task done:

- [ ] One frame held throughout; audience and reader's question are written down.
- [ ] The first ~100 words contain the action the reader came for.
- [ ] Every example ran as written — self-contained, output captured and pasted.
- [ ] Every behavior claim carries a file:line citation that checks out at this commit.
- [ ] No hedge the reader cannot act on; conditional behavior states its condition.
- [ ] Docs in scope that contradicted this one are reconciled; contradictions out of scope are
      finding candidates. No `TODO`/`FIXME` left behind.
- [ ] Format / doc-lint output is pasted; you issued no review result on your own work.

## Bundled resources

- [`references/task-template.md`](./references/task-template.md) — a working-notes scaffold for the run (frame, audience, reader's
  question; examples-to-verify table; pasted evidence; self-review). The task packet itself uses
  the kit's task template.
