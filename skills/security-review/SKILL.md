---
name: security-review
type: agent-guide
description: Security-review a change by reasoning about what it does to the system's trust boundaries — not by grepping for scary strings — and report only reachable, concrete-impact issues. ALWAYS apply when reviewing a diff/PR/branch that touches authn/authz, user input, secrets, file/path/SQL/command/template sinks, deserialization, crypto, network calls, or dependency/config changes; or when explicitly asked for a security pass. Drive a real scanner (Semgrep/CodeQL) when one is configured and reason about its output rather than reviewing prose-only. Skip pure-formatting/rename/comment/doc diffs with no data-flow change, and defer to the team's security on-call for live incidents.
---

# Skill: security-review

## Purpose

A general-purpose code review (see [`adversarial-review`](../adversarial-review/SKILL.md)) hunts
correctness; it is not tuned for the attacker's-eye question of *what an adversary can now do*. Two
failure modes show up when an agent does a security pass untuned:

1. **Regex theater.** The agent greps for `password`, `eval`, `exec`, `SELECT` and flags string
   matches — missing the actual bug, which is a *flow*: tainted input reaching a sink across functions
   the grep never connected.
2. **False-positive flood.** The agent dumps every theoretical concern (missing input validation,
   "could be a DoS", open-redirect maybe, no rate limit) as if all were findings. A reviewer who floods
   low-confidence flags gets muted — a check that fires on noise has under ~10% effective value and
   developers stop reading it. At that point the security review is worse than none, because it
   buries the one real flag.

This skill replaces both with **semantic data-flow review** plus a **tunable false-positive filter**:
review the PR diff by reasoning about taint flow, then actively filter low-signal classes before
reporting.

## When a scanner exists, drive it first

Prose-only review misses flows; a static analyzer was built to follow them. Before reasoning by hand,
check whether the repo already has **Semgrep** (`.semgrep.yml`, `semgrep.yml`, a `semgrep` CI step) or
**CodeQL** (`.github/workflows/codeql*`, a `codeql` config). If a scanner like Semgrep or CodeQL is
configured, run it against the change and reason about its output:

- A scanner **hit** is a lead, not a verdict — confirm reachability before you report it (see step 4).
- A scanner **miss** is not absolution — the rules cover known patterns, not your specific logic. Hand
  review still applies to the trust-boundary surfaces in step 2.
- No scanner configured → say so, then do the hand review below. Don't invent a `semgrep` invocation
  the repo doesn't support; a guessed scan that "finds nothing" is negative evidence.

## Core procedure

### 1. Read the stated intent, then map the trust boundaries the change crosses

Read the PR description / stated requirements first, then ask of the diff: **where does data or control
cross a trust boundary?** Untrusted → trusted is where vulnerabilities live. Tag every changed surface
that touches: request/CLI/file input, authn or authz decisions, secrets/keys/tokens, a sink (SQL,
shell, filesystem path, HTTP request, template render, deserializer, reflection), crypto, or
inter-service calls.

### 2. Trace taint flow across functions — not within the diff alone

For each tainted input, follow it to a sink **through the call graph**, not just the changed lines. The
vulnerability is frequently in an *unchanged caller* the diff never touched: the new function trusts an
argument that an old caller passes unsanitized. Grep the callers of every changed surface
(`git grep -n '<symbol>'`) and read them. Ask at the sink: is the input attacker-controlled, and is
there no sanitizer between source and sink on *any* path?

### 3. Calibrate for the language in front of you

Each stack has footguns a generic pass misses. A few high-yield ones — the full table is in
[`references/language-quirks.md`](./references/language-quirks.md):

- **Python** — `pickle`/`yaml.load`/`eval` on untrusted data; `subprocess(..., shell=True)`.
- **JS/TS** — prototype pollution; `child_process.exec` vs `execFile`; `dangerouslySetInnerHTML`.
- **Java** — unsafe deserialization; XXE in default XML parsers; reflection sinks.
- **Go** — `text/template` (no escaping) where `html/template` was meant; SSRF via unvalidated URLs.
- **SQL anywhere** — string-concatenated queries vs parameterized; ORM `.raw()` escapes.

Pull the file up and check the rows for the languages the diff actually changes.

### 4. Apply the false-positive filter — report only reachable, concrete impact

This is the differentiator. Before a concern becomes a reported finding, it must clear the filter.
**Auto-drop these low-signal classes unless you can name a concrete, reachable exploit path:**

- Generic "should validate input" hand-wringing with no demonstrated sink.
- Theoretical DoS / resource exhaustion with no realistic trigger.
- Missing rate-limiting (an infra/ops control, rarely a code-diff finding).
- Open-redirect / verbose-error / missing-security-header noise with no concrete abuse.
- A flag on input the code does not actually expose to an attacker.

The filter is **tunable**: in a pre-auth, internet-facing path tighten it (theoretical reachability
counts); in an internal admin-only tool with trusted callers, drop more aggressively. State the tuning
you chose so the reader can re-tighten it. A reachable concrete bug that you down-ranked is the
expensive error — when unsure whether something is reachable, *investigate the path* before dropping or
reporting; don't resolve the doubt by flagging it anyway.

### 5. Report: each finding is reachable, evidenced, severity-rated

A finding carries: **severity** (by exploit impact, not a flat list) · **file:line of the sink** · the
**source→sink path** (the trace, the entry point, the missing sanitizer) · a **concrete impact** (what
an attacker gains) · a **fix sketch**. No source→sink path → it didn't clear step 4; cut it. Lead with
the path so the reader can check it cheaply.

## Gotchas

- **Regex theater.** Flagging a `"password"` string literal or an `exec` *appearing in* the diff while
  the real bug is a multi-hop taint flow the string match never followed. A match is a lead to trace,
  never a finding on its own.
- **Diff-only tunnel vision.** The changed function is safe in isolation; the vuln is that an
  *unchanged* caller feeds it unsanitized input. Reviewing only the diff misses it every time — trace
  out to the callers (step 2).
- **False-positive flood.** Shipping ten theoretical concerns to look thorough. The reader mutes the
  channel and the one real BLOCKER dies with the noise — the precise effect the <10% effective-value
  threshold predicts. Fewer, reachable findings beat a comprehensive-looking list.
- **Unreachable-as-blocker.** Reporting a textbook issue (missing header, theoretical timing leak) as a
  blocker when no attacker path reaches it. Severity is exploit impact on a *reachable* path; an
  unreachable issue is a note at most.
- **Scanner output trusted whole.** Treating every Semgrep/CodeQL hit as a confirmed vuln (they carry
  their own false positives) — or treating a clean scan as proof of safety. The scanner finds patterns;
  you confirm reachability and cover the logic its rules don't.
- **Sanitizer assumed, not verified.** "There's surely validation upstream" — assumed, not read. If you
  can't point at the sanitizer on the path, treat the path as unsanitized.

## What does not belong

- A finding without a source→sink path or concrete impact (it failed the filter — cut it).
- Generic input-validation, theoretical-DoS, rate-limiting, or header noise reported as blockers.
- Writing the fix here (that is a separate task) or self-issuing a merge verdict — you produce findings
  + a human-attention list; an accountable human owns shipping.
- Style/correctness nits unrelated to security (route those to the general review).

## Pairs with

- [`adversarial-review`](../adversarial-review/SKILL.md) — the refute-by-default review *procedure* and
  panel; run a security lens inside it, or this skill as the dedicated security pass.
- [`empirical-proof`](../empirical-proof/SKILL.md) — paste the verbatim scanner output and the
  reproduction of any exploit path you claim; don't assert "Semgrep is clean" without the run.
- [`concise-output`](../concise-output/SKILL.md) — keep findings lean, but security notes are the named
  exception to compression: full, unambiguous prose on the exploit path.

## Bundled resources

- [`references/language-quirks.md`](./references/language-quirks.md) — the per-language footgun table
  (sources, sinks, and the stack-specific traps), kept out of the body so it loads only when the diff's
  language warrants it.
