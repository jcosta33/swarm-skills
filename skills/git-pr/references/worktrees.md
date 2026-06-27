# Git worktrees for parallel branches

Detail for the "Isolate parallel work in separate worktrees" stage of the `git-pr` skill.

## Why a worktree instead of switching branches

A single checkout can only have one branch checked out at a time. Switching between two lines of
work means `git stash` / `git checkout` churn, and the working tree, build artifacts, and any
running dev server belong to whichever branch is currently out. That serializes parallel work and
invites two failure modes: carrying uncommitted changes across a branch boundary by accident, and a
long build/test run on branch A being invalidated the moment you switch to branch B.

A worktree gives each branch its **own directory** backed by the **same `.git`**. Branch A and
branch B each have a clean tree, their own `node_modules` / `target` / build output, and can run
tests or a dev server simultaneously without touching each other.

## When to spin one up

- An urgent fix interrupts in-progress feature work — fix in its own worktree, leave the feature
  tree untouched.
- Two independent tasks you want to progress in parallel (or hand to two agents).
- You want to keep a clean reference checkout of the base branch while you work on a branch.
- A long-running build/test on one branch that you don't want to interrupt to do something else.

If the work is strictly sequential and short, a plain branch switch is fine — don't add worktrees
for their own sake.

## Create, use, remove

```bash
# Create a worktree for a new branch off the current HEAD
git worktree add ../repo-feature-x -b feature-x

# Or for an existing branch
git worktree add ../repo-feature-x feature-x

# Work inside it like any checkout
cd ../repo-feature-x
# ... edit, run tests, commit, push from here ...

# See all worktrees and which branch each has out
git worktree list

# When the branch is merged/abandoned, remove the worktree
git worktree remove ../repo-feature-x

# Clean up administrative records for manually-deleted worktree dirs
git worktree prune
```

## Naming and placement

- Put worktrees **as siblings** of the main checkout (`../repo-feature-x`), not nested inside it —
  a nested worktree gets swept into the parent's file globs, test runners, and `git status`.
- Name the directory after the branch so `git worktree list` is self-explanatory.

## Cleanup discipline (the part that goes wrong)

Worktrees are easy to create and easy to forget. A dangling worktree:

- keeps a branch "checked out", so `git branch -d` of that branch and a fresh `git worktree add` for
  it both error with "already checked out";
- leaves build artifacts and dependencies on disk consuming space;
- becomes a stale copy you might edit by mistake, thinking it is the live tree.

Make removal part of finishing a branch: when the PR merges or the work is abandoned, `git worktree
remove` it the same way you'd delete the branch. Run `git worktree list` periodically to catch
strays, and `git worktree prune` after you've manually deleted any worktree directory so Git drops
its stale bookkeeping.
