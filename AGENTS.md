# AGENTS.md — swarm-skills

This repo is the optional-skills catalog for the Swarm framework: self-contained
agent guides in the Agent Skills format, one folder per skill under `skills/`,
with the evidence behind their design under `docs/`. It is a derived-content
repo — it carries no Swarm workspace install; the work of changing it is
planned and reviewed in the family workspace (the sibling `swarm-hq` repo).

## Editing rules

The rationale and evidence for each rule live in `docs/` (the rule names its page).

- One skill per folder: `skills/<name>/SKILL.md`, plus `references/` only for
  material the skill itself instantiates (session frames, templates).
- **The description rule** (`docs/activation.md`): the frontmatter `description`
  is the trigger and must be directive — open with the verb of the work, say
  when to ALWAYS apply, name what the skill refuses, end with a `Skip for …`
  clause naming task types (never sibling skill names).
- **The body-length rule** (`docs/body-anatomy.md`): bodies stay under the
  500-line hard cap, target ~200; depth moves to `references/`. Rules are
  numbered, each with a one-line rationale.
- **The forced-output rule** (`docs/execution.md`): a verification step must
  require visible output (a paste, a table, a marker in the deliverable) — a
  step that only asks the agent to "verify" gets silently skipped.
- **Self-containment** (`docs/self-containment.md`): a skill must read
  correctly with no other file from this repo installed. Restate a load-bearing
  sibling rule inline; point at a sibling that *carries* a discipline only as
  `../<name>/SKILL.md` with an explicit "if installed" marker. A plain mention
  that merely sends a different task elsewhere (`load fix-flaky-test, not this`)
  carries no dependency and needs no marker — the test is whether the skill
  still works when the named guide is absent. Kit templates and reference cards
  are referenced by their workspace-root-relative path (`advanced/audit.md`,
  `templates/spec.md` — these resolve in any Swarm-kit workspace). Anything
  else goes to the Swarm repo by name, never by relative path.
- Skills name abstract command slots (`cmdTest`, `cmdLint`, `cmdValidate`, …) —
  never a concrete toolchain command; the consuming repo's `AGENTS.md` supplies
  those. An empty slot means ask.
- Markdown only — no scripts, no executables, no network calls
  (`docs/scope.md`).
- The catalog table in `README.md` gains/loses a row with every skill
  added/removed.

## Commands

| Slot | Command | Resolves |
|---|---|---|
| — | (none) | markdown-only repo; content is checked by review |
