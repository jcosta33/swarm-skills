# AGENTS.md — swarm-skills

This repo is the optional-skills catalog for the Swarm framework: self-contained
agent guides in the Agent Skills format, one folder per skill under `skills/`.
It is a derived-content repo — it carries no Swarm workspace install; the work
of changing it is planned and reviewed in the family workspace (the sibling
`swarm-hq` repo).

## Editing rules

- One skill per folder: `skills/<name>/SKILL.md`, plus `references/` only for
  material the skill itself instantiates (session frames, templates).
- A skill is self-contained: it must read correctly with no other file from
  this repo installed. Two reference conventions are allowed: a sibling skill
  as `../<name>/SKILL.md` marked "if installed", and a kit template or
  reference card by its workspace-root-relative path (`advanced/audit.md`,
  `templates/spec.md` — these resolve in any Swarm-kit workspace). Anything
  else goes to the Swarm repo by name ("the Swarm repo's
  `docs/reference/checks.md`"), never by relative path.
- Skills name abstract command slots (`cmdTest`, `cmdLint`, …) — never a
  concrete toolchain command; the consuming repo's `AGENTS.md` supplies those.
- The frontmatter `description` is the trigger: it must say when to ALWAYS
  apply the skill and when to skip it.
- Markdown only — no scripts, no executables, no network calls.
- The catalog table in `README.md` gains/loses a row with every skill
  added/removed.

## Commands

| Slot | Command | Resolves |
|---|---|---|
| — | (none) | markdown-only repo; content is checked by review |
