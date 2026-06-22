# Changelog

All notable changes to the Swarm skills catalog are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the catalog is versioned with
[semantic versioning](https://semver.org/spec/v2.0.0.html): a **major** bump removes or
renames a skill or breaks a skill's contract, **minor** adds a skill or non-breaking guidance,
**patch** is a fix or wording change.

The catalog is pull-updatable — you install it with `npx skills add jcosta33/swarm-skills` (pin to
a tag or commit for stability) and re-run to re-fetch. Watch the
[releases](https://github.com/jcosta33/swarm-skills/releases) and re-pull when a bump matters.

## [Unreleased]

## [1.0.0] - 2026-06-22

First versioned release. The catalog ships **14 skills**: the cross-cutting conditioning stances
`persona-skeptic`, `persona-challenger`, `persona-surveyor`, and `empirical-proof`; the
code-authoring depth guides `write-feature`, `write-fix`, `write-refactor`, `write-rewrite`,
`write-migration`, `write-performance`, `write-testing`, `write-documentation`, `fix-flaky-test`;
and `implement-task`.

This baseline reflects two recent decisions:

- **[Swarm ADR-0093](https://github.com/jcosta33/swarm/blob/main/docs/adrs/0093-collapse-1to1-personas.md)** — the four 1:1 authoring personas (architect / auditor / researcher / documentarian) were
  collapsed into their work guides (single source); the catalog keeps only the cross-cutting trio
  plus `empirical-proof`, each rebuilt grounding-first (a stance's leverage is the external
  evidence it forces, not a role label).
- **[swarm-hq #45/#56](https://github.com/jcosta33/swarm-hq/issues/45)** — every guide that bundles a
  `references/` file now carries a Rule-0 load directive (a top-of-body, point-of-need 1-hop link),
  so the reference is actually loaded rather than merely listed.
