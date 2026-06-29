# Changelog

All notable changes to the Suspec skills catalog are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the catalog is versioned with
[semantic versioning](https://semver.org/spec/v2.0.0.html): a **major** bump removes or
renames a skill or breaks a skill's contract, **minor** adds a skill or non-breaking guidance,
**patch** is a fix or wording change.

The catalog is pull-updatable — you install it with `npx skills add jcosta33/suspec-skills` (pin to
a tag or commit for stability) and re-run to re-fetch. Watch the
[releases](https://github.com/jcosta33/suspec-skills/releases) and re-pull when a bump matters.

## [Unreleased]

### Changed

- **Re-baselined the catalog to the universal set (7 → 11 skills), 2026-06-27.** The catalog is now
  universal-only — skills that hold for any repo regardless of the Suspec workflow — per
  **[Suspec ADR-0112](https://github.com/jcosta33/suspec/blob/main/docs/adrs/0112-two-tier-skills.md)**.
  Added from a live adoption census: `codebase-exploration`, `concise-output`, `debugging`, `git-pr`,
  `planning-spec`, `security-review`, and `adversarial-review` (which now carries both the
  refute-by-default stance and the review procedure). The catalog ships **11 universal skills**:
  `adversarial-review`, `codebase-exploration`, `concise-output`, `debugging`, `empirical-proof`,
  `fix-flaky-test`, `git-pr`, `persona-challenger`, `persona-surveyor`, `planning-spec`,
  `security-review` — plus `persona-skeptic` kept as a retired redirect to `adversarial-review`.

### Moved

- **The `write-*` depth guides and `implement-task` moved to the starter kit** (ADR-0112): the
  Suspec-workflow guides — `write-feature`, `write-fix`, `write-refactor`, `write-rewrite`,
  `write-migration`, `write-performance`, `write-testing`, `write-documentation`, and
  `implement-task` — now live in `suspec-starter-kit` at `.agents/skills/`, not in this catalog.

## [1.0.0] - 2026-06-22

First versioned release. The catalog ships **14 skills**: the cross-cutting conditioning stances
`persona-skeptic`, `persona-challenger`, `persona-surveyor`, and `empirical-proof`; the
code-authoring depth guides `write-feature`, `write-fix`, `write-refactor`, `write-rewrite`,
`write-migration`, `write-performance`, `write-testing`, `write-documentation`, `fix-flaky-test`;
and `implement-task`.

This baseline reflects two recent decisions:

- **[Suspec ADR-0093](https://github.com/jcosta33/suspec/blob/main/docs/adrs/0093-collapse-1to1-personas.md)** — the four 1:1 authoring personas (architect / auditor / researcher / documentarian) were
  collapsed into their work guides (single source); the catalog keeps only the cross-cutting trio plus
  `empirical-proof`, each rebuilt grounding-first (a stance's leverage is the external evidence it
  forces, not a role label).
- **Reference-load wiring** — every guide that bundles a `references/` file now carries a Rule-0
  load directive (a point-of-need 1-hop link; top-of-body for the work guides), so the reference is
  actually loaded rather than merely listed.
