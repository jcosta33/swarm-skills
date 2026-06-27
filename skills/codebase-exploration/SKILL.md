---
name: codebase-exploration
type: agent-guide
description: Map an unfamiliar codebase by delegating narrow recon to subagents and building a key-files map before you read source — keep the orchestrator's context clean. ALWAYS apply when you land in a repo you don't know and must answer "where is X wired / what calls Y / how does this build" before changing anything, or when onboarding to a large or multi-package codebase. Delegate 2-4 read-only recon slices, each owning one question; demand a compact path→role table back; read only the few files that matter, in the main context. Skip when you already know the file to open (Read it directly) or the repo is small enough to grep in a step or two — delegation overhead isn't worth it there.
---

# Skill: codebase-exploration

## Purpose

The default move in an unfamiliar repo is to start opening files — grep a symbol, read
the hit, follow an import, read that, and keep going. By the time you understand the
layout, you have pulled twenty files of source into the one context that has to do the
actual work, and your attention is spread across all of it. Long context degrades
attention non-uniformly — relevant facts buried in the middle of a bloated window get
missed even when they are present.

This skill changes that: **the orchestrator does not read source files to orient.** It
delegates 2-4 narrow, read-only recon slices to subagents, gets back a compact
**key-files map** (path → one-line role), and only then reads the handful of files that
actually matter — in a context that is still mostly empty. The recon cost is paid in
throwaway subagent contexts; the main context stays clean for the work.

## When to delegate vs. just look

- **Delegate** when the repo is unfamiliar *and* answering the orientation question would
  mean reading several files you'd then carry as dead weight: "where is auth wired?",
  "what's the build/test layout?", "who calls this exported function?".
- **Just look** when you already know the path (Read it), or one or two greps settle it.
  Delegation has fixed overhead — spinning a subagent to read a single known file is
  slower and pollutes nothing you'd have polluted anyway.

## Procedure

### 1. Frame the orientation questions

Before spawning anything, write the 2-4 questions that, once answered, let you start the
real work — and no more. Each must be **answerable independently** and own a distinct
slice of the repo. Good slices for a typical service:

- *Wiring*: "Where is `<feature>` entered and what does it call, end to end?"
- *Build/test layout*: "What are the build, test, and lint commands, and where do tests live?"
- *Blast radius*: "What calls `<symbol you'll change>`? List call sites with file:line."
- *Data/config*: "Where are config and the data model defined?"

### 2. Delegate each slice to its own read-only subagent

One subagent per question. Each prompt must:

- **State the single question** and forbid scope creep ("answer only this; do not explore
  adjacent areas").
- **Be read-only** — the recon agent greps and reads; it does not edit.
- **Demand a compact return shape**, not a narrative: a `path → one-line role` table for
  the files that matter to that question, plus 1-2 sentences of how they connect. Tell it
  to return file:line for any call site, and to name what it could *not* find.

Spawn the independent slices together, not one at a time — they share no state.

### 3. Assemble the key-files map, then read

Merge the returned tables into one **key-files map**: every path that matters, each with a
one-line role. This map is your reading list. Now — and only now — Read the few files the
map says are load-bearing for your task, directly in the main context. You are reading 3-5
files you already know are relevant, not 20 you're triaging.

A worked map:

```
| path                          | role                                          |
| ----------------------------- | --------------------------------------------- |
| src/server/router.ts          | HTTP routes → handler wiring (entry point)    |
| src/auth/session.ts           | session create/verify; called by the guard    |
| src/auth/guard.ts             | per-route auth middleware                     |
| test/auth/session.test.ts     | the behavior to preserve                      |
| package.json (scripts)        | build: `pnpm build` · test: `pnpm test`       |
```

### 4. (Optional) Persist a structure index for next time

For a repo you'll revisit, write the key-files map plus per-directory one-line signatures
to a small checked-in or local file (e.g. a structure-index note). The next session reads
the index instead of re-running recon — a cold-start skip. Keep it short and **dated**, so
a reader can tell whether it predates recent changes.

## Gotchas

- **The orchestrator sneaks a "quick look."** You spawn the recon slices, then — while
  waiting, or because one answer is "almost there" — you grep and open a few files
  yourself "just to confirm." Now the main context carries the exact source you delegated
  recon to keep out of it, and the discipline bought nothing. Wait for the map; resist the
  peek. If you must verify one fact, do it in a subagent too.
- **Overlapping slices burn budget silently.** Two recon prompts that both end up reading
  the router because the questions weren't disjoint do the same expensive reads twice, and
  you pay for both contexts. The failure is invisible — each subagent looks productive.
  Carve the questions so each owns a distinct region before spawning.
- **The map goes stale after edits.** A key-files map (or a persisted index) is a snapshot.
  Once you start changing files, paths move, symbols get renamed, and a call site the map
  listed no longer exists — but the map still reads as authoritative. Treat it as a
  starting reading list, not a live source of truth; re-grep a symbol before you act on a
  map entry that an edit could have invalidated, and date any persisted index.
- **Recon returns a narrative, not a map.** A subagent that hands back three paragraphs of
  prose has just moved the bloat into your context instead of compressing it. Insist on the
  `path → role` table in the prompt and reject a walls-of-text reply — re-ask for the table.
- **Over-delegating a small repo.** Five subagents to map a four-file CLI costs more than
  reading it. Below the "just look" threshold, skip the ceremony.

## What does not belong

- Reading source files into the main context to orient (that is the behavior this skill
  exists to stop) — read source only after the map names it load-bearing.
- Recon subagents that edit, refactor, or "fix while they're in there" — recon is
  read-only; a recon agent that changes code is doing unreviewed work.
- A vague "explore the codebase" subagent with no single question — it returns a tour, not
  an answer, and re-pollutes your context.
- Treating a stale map or index as ground truth after you've started editing.

## Pairs with

- [`adversarial-review`](../adversarial-review/SKILL.md) — its cross-module caller search
  (grep callers, read the calling code) is the same blast-radius question this skill
  delegates as a recon slice; use that map to scope a review.
- [`concise-output`](../concise-output/SKILL.md) — the return-a-compact-table discipline
  is the same economy applied to subagent output; the key-files map is concise-output for
  recon.
