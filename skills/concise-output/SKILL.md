---
name: concise-output
type: agent-guide
description: Make agent output maximally readable and economical — signal-dense, evidence-first, structure over prose, no filler/hedging/ceremony. ALWAYS apply when a session values terse, scannable, token-economical output (status updates, reviews, summaries, findings). Never compress at the cost of correctness or safety — keep full, unambiguous prose for security notes, irreversible-action confirmations, and multi-step sequences where order matters. Skip when the reader explicitly wants narrative, teaching, or exploratory prose, and skip inside code, commits, and PR descriptions (write those normally).
---

# Skill: concise-output

## Purpose

A universal output-economy discipline: emit the leanest output that still carries the full signal.
Verbose output costs reader attention and tokens — and long persuasive prose raises *trust* without
raising *scrutiny*. This is a dial, not a hook — you apply it;
nothing rewrites your output.

## Core rules

### 1. Evidence first

Lead with the result, finding, or answer and its evidence. Put any "why" after — and only if it makes
the reader's check cheaper, not to convince them.

### 2. Structure over prose

Use a table or a tight list when it carries the same signal in less space. A coverage table beats a
paragraph describing coverage. One claim per line.

### 3. Cut filler

Drop pleasantries, hedging, restating the prompt, and "as you can see" narration. Prefer the short
exact word (big, not extensive; fix, not implement-a-solution-for). Fragments are fine.

### 4. Reason free, emit lean

Think in whatever form works; then emit the structured, compressed artifact. Format constraints belong
on the *output*, never on the *reasoning* — format restriction during reasoning degrades it.

### 5. Clarity outranks brevity (the hard limit)

Brevity that costs correctness or safety is a net loss, so it is the wrong trade. Keep full,
unambiguous prose for:

- security warnings and irreversible-action confirmations
- multi-step sequences where order matters or a fragment risks a misread

When in doubt between terse and clear, choose clear — a misread costs the reader more than the words you
saved. Code, commits, and PR bodies stay readable in their own conventions; compressing them works
against the people and tools that parse them, so write those normally.

## Gotchas

- Compressing a security note, an irreversible-action confirmation, or an ordered step sequence — these
  are the hard limit; clarity wins because a misread there is expensive.
- Dropping the evidence to *look* terse: terseness without the supporting result is just an unbacked
  claim, which is the opposite of the goal.
- Terse-but-ambiguous fragments the reader misreads — saving words but adding a round-trip is a loss,
  not a win.
- Compressing code, commits, or PR descriptions — those have their own conventions; write them normally.

## See also

Corpus ships a baseline output-economy convention; this skill is a self-contained, stronger dial that
needs no Corpus knowledge to apply. (Pointer, not a dependency.)
