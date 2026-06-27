---
name: git-pr
type: agent-guide
description: Ship a change end to end with discipline — stage only what belongs, commit with an imperative subject and a why-body, push, and open a PR that states intent and how to verify. ALWAYS apply when committing, pushing, opening or updating a pull request, addressing review comments, fixing a failing CI job, or running parallel branches that risk clobbering each other. Run review-comment and CI work as a loop you close, not a fire-and-forget. Never force-push a shared branch, stage files you did not inspect, blind-retry a red CI job, or open a PR with no how-to-verify. Skip when only editing files locally with no intent to commit, and when the repo's own contribution guide prescribes a different flow (follow it).
---

# Skill: git-pr

## Purpose

The default agent behavior is to treat "ship it" as one undifferentiated act — `git add -A && git
commit -m "fix" && git push` — and to treat reviewer comments and red CI as someone else's problem.
Each shortcut fails silently in a way nobody notices until later: `add -A` sweeps a secret or an
unrelated edit into history; a one-word commit message erases the *why* the next reader needs; a PR
with no verification steps makes the reviewer reconstruct your intent from the diff; a blind CI
re-run burns minutes and hides a real bug; parallel branches in one checkout overwrite each other's
work. This guide makes shipping a **disciplined lifecycle** — stage, commit, push, open, then close
the loop on review and CI — so the change lands the way you meant it to.

The ship loop here is stage → commit → push → open a PR, plus address review comments and fix a failing
CI job; the parallel-work isolation uses git worktrees.

## Resolve the repo's flow first

Before committing, check the repo for a contribution guide (`CONTRIBUTING.md`, `AGENTS.md`, a PR
template, or the CI config). If it prescribes a branch-naming scheme, a commit-message convention
(Conventional Commits, a sign-off, an issue-key prefix), or a required PR section, follow that over
the generic flow below — a house convention beats a guessed one. Also confirm whether to commit or
push at all: do those only when the user asked, and never push straight to the default branch.

## The ship lifecycle

### 1. Stage only what belongs

Run `git status` and `git diff` and read them before staging. Stage the files that make up *this*
change by name (`git add path/...`), not `git add -A` / `git add .` — the catch-all is how a `.env`,
a credential, a debug print, or an unrelated edit ends up in the commit. After staging, run `git
diff --cached` and read the whole thing: that is exactly what you are about to commit. A change that
mixes two unrelated concerns should be two commits.

### 2. Commit with a message the next reader needs

- **Subject:** imperative mood, ~50 chars, no trailing period — "Add retry to the upload client",
  as if completing "If applied, this commit will…". It describes what the change *does*, not what
  you did ("Added", "Fixes").
- **Body (the part that matters):** explain the *why* — the problem, the constraint, the alternative
  you rejected. The diff already shows *what* changed; the body carries the intent the diff cannot.
  Wrap at ~72 chars, separated from the subject by a blank line. Skip the body only for a genuinely
  trivial change.

### 3. Push to a branch, never force a shared one

Work on a feature branch, not the default branch. Push with `git push -u origin <branch>`. If you
must rewrite history you already pushed, use `git push --force-with-lease`, never bare `--force`:
`--force-with-lease` refuses the push if someone else added commits you have not seen, which is the
exact accident bare `--force` causes. On a branch others may be using, prefer not to rewrite history
at all.

### 4. Open a PR that states intent and how to verify

The description has two non-negotiable parts: **what this changes and why** (the intent, linked to
the issue/requirement it satisfies), and **how to verify** (the exact commands or steps a reviewer
runs to confirm it works — the test command, the manual repro, the before/after). A PR with no
how-to-verify forces the reviewer to invent a check or rubber-stamp the diff. Paste the verification
output you already ran into the description.

## Close the loop on review comments

When a PR has review feedback, do not cherry-pick the easy ones and leave the rest dangling. Pull
the **unresolved** comments (`gh pr view <n> --comments`, or the review threads via the API), and
for each one: act on it *or* reply with why you are not, then resolve the thread. The loop is closed
only when every thread is either addressed-in-a-commit or answered — a reviewer who has to re-check
whether their three comments were seen has been given more work, not less. Push the follow-up commit
and leave a short comment pointing to it.

## Fix failing CI by reading it, not retrying it

A red CI job is a result to read, not a button to mash. **Read the failing job's logs first** (`gh
run view <run-id> --log-failed`), find the actual failing step and its error. Then **reproduce it
locally** — run the same command the job ran — so you are fixing the cause, not guessing. Fix it,
re-run that command locally to confirm green, then push. Re-running the CI job without a change is a
blind retry: if it greens, you have shipped a flake or a load-dependent bug; if it fails again, you
have learned nothing and spent the minutes anyway. Retry-without-a-change is only ever justified for
a known-infrastructure outage, and then say so.

## Isolate parallel work in separate worktrees

When you have more than one line of work in flight (a feature branch and an urgent fix, or two
independent tasks), do **not** stash-and-switch branches in a single checkout — that serializes the
work and risks carrying uncommitted state across branches. Use a separate **git worktree** per
branch: `git worktree add ../repo-feature-x feature-x` gives each branch its own directory sharing
one `.git`, so builds, test runs, and edits never clobber each other. Remove it when done with `git
worktree remove ../repo-feature-x`. The deeper worktree workflow (when to spin one up, naming,
cleanup discipline) is in [`references/worktrees.md`](./references/worktrees.md).

## Gotchas

- **`--force` clobbering a shared branch.** A bare force-push silently discards commits a teammate
  pushed while you were rebasing — they are simply gone from the remote, no warning. Use
  `--force-with-lease`, which aborts when the remote moved under you.
- **Secrets and stray files swept in by `add -A`.** The catch-all stages everything dirty in the
  tree — a generated `.env`, a credential, a scratch file, an unrelated edit. It commits clean and
  pushes clean; the leak surfaces only when someone reads the history. Stage by name and read `git
  diff --cached`.
- **Blind CI retry that "fixes" a flake.** Re-running a red job until it greens hides a real
  intermittent bug behind a passing badge — the failure is load- or order-dependent and will return
  in production. Read the log; reproduce locally. (A flaky test is its own job — pair with
  `fix-flaky-test`.)
- **Dangling worktrees.** A worktree you forgot to remove keeps a stale branch checked out and its
  build artifacts on disk; later `git worktree add` for the same branch errors, or you edit the
  wrong copy thinking it is the live one. List with `git worktree list`; prune with `git worktree
  remove` / `git worktree prune`.
- **PR description that restates the diff.** "Updated upload.ts" tells the reviewer nothing the file
  list already showed — the intent (why) and the verification (how to check) are the parts only you
  can supply, and the parts the reviewer actually needs.
- **Review thread left open after the fix.** You fixed the issue in a commit but never replied or
  resolved the thread, so the reviewer cannot tell addressed from ignored and re-reviews everything.
  Closing the loop is part of the fix.

## What does not belong

- `git add -A` / `git add .` without reading the staged diff; bare `git push --force` on any branch
  others touch; pushing to the default branch directly; committing when the user did not ask to.
- A commit subject that says what you did ("Added X") rather than what the commit does ("Add X"); a
  body that paraphrases the diff instead of giving the why.
- A PR with no how-to-verify; review comments addressed silently with no reply/resolve; a CI re-run
  with no code change behind it.

## Pairs with

- [`empirical-proof`](../empirical-proof/SKILL.md) — paste the verbatim test/CI output into the
  commit/PR/review reply, rather than asserting "tests pass".
- [`adversarial-review`](../adversarial-review/SKILL.md) — the receiving side: how the reviewer reads
  your PR. Open the PR so that review is cheap.
- [`fix-flaky-test`](../fix-flaky-test/SKILL.md) — when the red CI job is an intermittent failure,
  not a deterministic one, stabilize it there instead of retrying.
- [`concise-output`](../concise-output/SKILL.md) — for the status update around the ship; the commit
  body and PR description themselves stay in their normal full prose.

## Bundled resources

- [`references/worktrees.md`](./references/worktrees.md) — the full git-worktree workflow for
  parallel branches: when to create one, naming and placement, running tools inside it, and the
  cleanup discipline that keeps worktrees from dangling.
