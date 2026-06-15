# swarm-skills

> The optional skills catalog for [Swarm](https://github.com/jcosta33/swarm) — conditioning stances and code-authoring depth in the open Agent Skills format, installable into any agent CLI.

Each skill is a self-contained folder under [`skills/`](./skills/): one `SKILL.md` with a trigger description and the working rules, plus bundled `references/` where a skill ships a fillable session frame. No scripts, no runtime — markdown an agent loads when the work matches.

The [Swarm starter kit](https://github.com/jcosta33/swarm-starter-kit) ships the guides the workflow itself requires — the core loop (`write-spec`, `implement-task`, `review-output`) and the workspace authoring guides (`write-audit`, `write-research`, `write-rfc`, `write-prd`, `write-bug-report`, `write-change-plan`, `write-inventory`, `spec-check`, `split-work`, `save-findings`, `adversarial-review`). Everything here is the optional layer on top: install only what your work calls for.

## Install

With the [Vercel skills CLI](https://github.com/vercel-labs/skills) (works with Claude Code, Cursor, Codex, OpenCode, Gemini CLI):

```bash
# list what's available
npx skills add jcosta33/swarm-skills --list

# install one skill into the current repo
npx skills add jcosta33/swarm-skills --skill persona-skeptic

# install globally, or for a specific agent
npx skills add jcosta33/swarm-skills --skill persona-skeptic -g
npx skills add jcosta33/swarm-skills --skill persona-skeptic -a claude-code
```

No CLI? Copy the folder: `cp -R skills/persona-skeptic <your-repo>/.agents/skills/` (point your tool's skills directory at the same folder — e.g. a `.claude/skills` symlink).

## The AGENTS.md contract

Skills name abstract command slots — `cmdTest`, `cmdLint`, `cmdTypecheck`, `cmdValidate` — never concrete commands. The consuming repo's `AGENTS.md` Commands table supplies the implementations. That split is what makes a skill portable: the guide carries the discipline, your repo carries the toolchain. An empty slot means **ask** — a skill never invents a command. The [Swarm starter kit](https://github.com/jcosta33/swarm-starter-kit) sets this contract up for you.

## Where to start

You don't need any of these to run Swarm — the [starter kit](https://github.com/jcosta33/swarm-starter-kit)
already ships the core loop. Add skills only as a specific need shows up, in roughly this order:

1. **Nothing.** Run the loop with the kit's core guides. Most changes never need more.
2. **`persona-skeptic`** — the first one most teams want. Load it when an agent *judges another
   agent's* completion claims, so the review refutes by default and re-runs the checks rather than
   trusting them.
3. **`empirical-proof`** — pair it with any completion claim to force verbatim pasted output; the
   fastest cure for "done" that was never actually checked.
4. **A code-authoring guide** matching the change shape — `write-fix` for a reproduced defect,
   `write-refactor` for behavior-pinned restructuring, `write-migration` for an A→B move. Install
   the one the task calls for, not the set.
5. **A `persona-*` stance** when the *kind of thinking* matters more than the procedure —
   `persona-architect` while shaping a spec, `persona-auditor` while mapping a brownfield codebase.

Rule of thumb: install the fewest skills that name the discipline your current task is missing.

## Catalog

### Conditioning (stances)

Cognitive postures loaded *alongside* a work guide — they tilt what the agent looks for and refuses, while the guide carries the procedure.

| Skill | Use it when |
|---|---|
| `persona-architect` | shaping a spec or design — requirements free of smuggled implementation, each one verifiable |
| `persona-auditor` | recording present state — observation not prescription, file:line per finding, severity by blast radius |
| `persona-documentarian` | writing human-facing docs — one frame throughout, every example run as written |
| `persona-researcher` | depth inquiry against primary sources, committing to no decision |
| `persona-skeptic` | judging another agent's completion claims — refute by default, re-run the checks yourself |
| `persona-surveyor` | breadth research — what prevails across many products, patterns, or users |
| `empirical-proof` | any completion claim — bind it to verbatim pasted output, or it reads unverified |

### Code authoring

| Skill | Use it when |
|---|---|
| `implement-task` | implementing a Swarm task packet, long form — supersedes the kit's core guide when you want the full session frame |
| `write-feature` | net-new behavior behind a defined surface |
| `write-fix` | a reproduced defect with a root cause |
| `write-refactor` | restructuring with behavior pinned by tests |
| `write-rewrite` | re-implementing code whose behavior changes on purpose, with the delta recorded and the rest proven preserved |
| `write-migration` | moving the code from API A to API B — green after every wave, old callsites grepped to zero |
| `write-performance` | a measured bottleneck with a target, baseline first |
| `write-testing` | adding the tests an area should already have |
| `write-documentation` | human-facing docs for a reader who hasn't read the code |
| `fix-flaky-test` | a test that fails intermittently — diagnose, don't retry-loop |

## The science

[`docs/`](./docs/) documents the empirical evidence behind every structural choice in these skills — why descriptions are directive ([activation](./docs/activation.md)), why bodies stay under 200 lines ([body anatomy](./docs/body-anatomy.md)), why verification steps force visible output ([execution](./docs/execution.md)), why skills don't depend on each other ([self-containment](./docs/self-containment.md)), when a skill ships a task template ([task files](./docs/task-files.md)), and what deliberately stays out ([scope](./docs/scope.md)) — with the full bibliography in [sources](./docs/sources.md).

## Security

Read a skill before installing it — a skill is instructions your agent will follow. Everything here is plain markdown: no scripts, no network calls, no executables. Pin to a commit if you need a stable install.

## Relationship to the Swarm framework

These skills assume nothing about Swarm — each stands alone in any repo with an `AGENTS.md`. They pair naturally with the Swarm working discipline (specs with verifiable requirements, task packets with evidence-backed claims, review packets as the durable record); the framework and its docs live at [jcosta33/swarm](https://github.com/jcosta33/swarm), the copy-whole workspace at [jcosta33/swarm-starter-kit](https://github.com/jcosta33/swarm-starter-kit). This catalog is curated: skill content is edited here, and changes are planned and reviewed in the Swarm project's workspace.
