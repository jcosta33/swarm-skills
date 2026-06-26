---
name: concise-output
type: agent-guide
description: Make agent output maximally readable and economical — signal-dense, evidence-first, structure over prose, no filler/hedging/ceremony. ALWAYS apply when a session values terse, scannable, token-economical output (status updates, reviews, summaries, findings). This is the strong dial above the baseline output-economy convention (Corpus ADR-0109). Never compress at the cost of correctness or safety — keep full, unambiguous prose for security notes, irreversible-action confirmations, and multi-step sequences where order matters. Skip when the reader explicitly wants narrative, teaching, or exploratory prose, and skip inside code, commits, and PR descriptions (write those normally).
---

# Skill: concise-output

## Purpose

Verbose output costs reader attention and tokens — and long persuasive prose raises *trust* without
raising *scrutiny* ([\[52\]](../../docs/sources.md#52)). This skill is the strong setting of the Corpus output-
economy convention ([ADR-0109]): emit the leanest output that still carries the full signal. It is a
dial, not a hook — you apply it; nothing rewrites your output.

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
on the *output*, never on the *reasoning* ([\[50\]](../../docs/sources.md#50) — format restriction
during reasoning degrades it).

### 5. Clarity outranks brevity (the hard limit)

Never compress at the cost of correctness or safety. Keep full, unambiguous prose for:

- security warnings and irreversible-action confirmations
- multi-step sequences where order matters or a fragment risks a misread

When in doubt between terse and clear, choose clear. Code, commits, and PR bodies are written normally,
not compressed.

## Relationship to Corpus

This is the opt-in amplifier of [ADR-0109]'s convention floor (which every Corpus workspace already
carries). Installing it is a team/agent choice — Corpus requires neither this skill nor any particular
output style.

[ADR-0109]: https://github.com/jcosta33/corpus/blob/main/docs/adrs/0109-output-economy-convention.md
