# Self-containment

> **Why no `SKILL.md` references another skill, why project-specific values resolve through `AGENTS.md`, and why the personas live in individual folders rather than one shared index.**

Skills install individually. A user who installs `write-feature` does not have `empirical-proof` in their context unless they also install it. The repo's structure has to assume that — every shipped skill stands alone, with no implicit dependencies on its siblings.

---

## The progressive-disclosure model

The open spec [\[1\]](./sources.md#1) defines skills as **independently loaded units**:

```mermaid
flowchart LR
    M1[Skill A metadata] -.always.-> CTX[(Agent context)]
    M2[Skill B metadata] -.always.-> CTX
    M3[Skill C metadata] -.always.-> CTX
    B1[Skill A body] -->|trigger match| CTX
    B2[Skill B body] -->|trigger match| CTX
    R1[Skill A references/...] -.on demand.-> B1
```

Each skill's metadata is in context regardless of which other skills are installed. The body loads when the description matches. References load on demand, scoped to that skill.

The structural consequence is non-trivial: **a skill cannot assume any other skill is in context**. If `write-feature` *expects* `empirical-proof`'s discipline, that discipline has to be either restated inline or re-derivable from `write-feature` alone.

---

## Rule 1: no cross-skill references in `SKILL.md`

Anti-pattern catalogues converge on the same finding [\[6\]](./sources.md#6) — *"Reference Illusion"*: skills referencing files or skills that may not exist on the consumer's machine. The consequence is a dead reference that silently degrades behaviour.

| Source | Finding |
| --- | --- |
| [\[1\]](./sources.md#1) Open spec | Each skill's metadata loads independently; no implicit ordering between skills. |
| [\[2\]](./sources.md#2) Anthropic best practices | Recommends *"focused, composable skills"*. |
| [\[6\]](./sources.md#6) Skill anti-patterns | "Reference Illusion" — linking to skills/files that aren't guaranteed to be present. |

**Applied in this repo:** [`AGENTS.md`](../AGENTS.md) carries the rule: a skill must read
correctly with no other file from this repo installed. If a related discipline is load-bearing,
restate the rule inline; a sibling may be *mentioned* only as `../<name>/SKILL.md` with an
explicit "if installed" marker, so a missing sibling reads as an option, never a dead
dependency. Unmarked cross-skill links exist only in the repo's *meta* docs (this `docs/`
directory, `README.md`, `AGENTS.md`) — never inside a `SKILL.md` body.

> A linked skill that isn't installed is a dead reference; an inline one-sentence restatement is robust.

---

## Personas: the canonical worked example

The persona discipline is where the self-containment principle does its loudest work. Every persona is a fully standalone skill — installing the starter kit's `write-audit` guide does not pull in `persona-auditor`, and installing the persona does not require the guide.

```mermaid
flowchart TD
    UR[User: "audit the auth module"] --> AGT[Agent assesses task]
    AGT -->|matches description| WA["write-audit loads (starter kit)"]
    AGT -->|matches description| PA[persona-auditor loads]
    WA -.no link to.-> PA
    PA -.no link to.-> WA
    WA & PA --> CTX[(Both in context, neither depends on the other)]
```

| Property | How it's enforced |
| --- | --- |
| **Each persona is a separate skill folder** | `skills/persona-auditor/SKILL.md`, `skills/persona-skeptic/SKILL.md`, … one folder per persona, each ~115–135 lines. |
| **Each persona activates from task assessment, not from cross-skill mention** | Each `description` names the task type the persona is for, e.g. *"ALWAYS apply this skill when authoring an audit of present state"*. The agent loads the persona because the task matches, not because another skill mentioned it. |
| **No persona index / core / loader skill** | There is no `personas-core`, no `personas` monolith. Each persona is independently installable. |
| **Personas are not referenced from any other skill** | `Grep` over `skills/` for `persona-` returns zero hits inside non-persona `SKILL.md` files and zero hits inside any `references/task-template.md`. The only matches are within the persona files themselves, where "personas" appears as a concept word ("do not blend personas"). |

> A consumer who installs only the kit's `write-audit` and this catalog's `persona-auditor` gets the same behaviour as one with everything installed — neither file mentions the other.

---

## Rule 2: project-specific values come from `AGENTS.md`

Skills are universal. The consuming repo holds project-specific values. Hardcoding `pnpm tsc --noEmit && pnpm lint` into a skill couples it to one stack and breaks every other consumer.

The repo solves this with the `AGENTS.md` contract — a Commands table in the consuming repo's
`AGENTS.md` mapping abstract slot names to real commands (the
[Swarm starter kit](https://github.com/jcosta33/swarm-starter-kit/blob/main/AGENTS.md) ships
the template). Skills reference the slots by name and degrade gracefully when an entry is
missing.

```mermaid
flowchart LR
    SK[skills/write-feature/SKILL.md] -->|"references the cmdValidate slot"| AC[(Consuming repo's AGENTS.md Commands table)]
    AC -->|resolves to| CMD["pnpm tsc --noEmit && pnpm lint"]
    SK -. if missing or undefined .-> ASK[Skill asks user before declaring verification done]
```

The [open `AGENTS.md` convention](https://agents.md) is adopted by Cursor, Codex, Claude Code, OpenCode, and others [\[19\]](./sources.md#19); the contract format used here extends it with an explicit Commands table.

### What the contract guarantees

| Slot | Read by |
| --- | --- |
| `cmdValidate` (and `cmdTypecheck` / `cmdLint`) | Any skill whose discipline includes running typecheck + lint as part of self-review. |
| `cmdTest` | Any skill that runs the test suite (feature, fix, refactor, rewrite, migration, performance, testing, flaky-test stabilisation). |
| `cmdFormat` | Any skill that closes a session by formatting touched files. |
| Project facts (stack, architecture rules, conventions) | All skills, for orientation. |

### What it deliberately doesn't cover

Some skills reference values *outside* the standard contract — a benchmark command for perf work, a dependency-validator for refactor / migration / review work, a doc-lint command for documentation work. Convention used throughout the repo: **the skill names the descriptive concept ("the project's benchmark command"), tells the agent to ask the user for the concrete value, and flags it in the PR**. New `AGENTS.md` sections are not added unilaterally from inside a skill.

---

## Why the cost is worth it

| Benefit | Mechanism |
| --- | --- |
| **Selective install works** | Any subset of the catalogue can be installed; nothing breaks because no skill assumes a sibling is present. |
| **Skills are stack-agnostic** | The same skill body runs in a TypeScript monorepo, a Rust crate, or a Python service — only `AGENTS.md` differs. |
| **Reviews are local** | A skill's correctness is determined by reading just that skill plus the `AGENTS.md` contract. No need to model cross-skill interactions. |
| **Forks are cheap** | A team can vendor a single skill into their own collection without porting an entire dependency graph. |

The cost is restated rules. The same discipline (e.g. *paste the actual output, not a paraphrase*) appears in `empirical-proof`, in `write-feature`, in `write-fix`, etc. — usually as one or two inline sentences rather than a link. The duplication is deliberate, not a TODO.

---

## State externalisation makes self-containment work

Self-containment relies on each skill carrying its own state. If one skill had to rely on another keeping a *shared in-memory* picture of what was decided, the skills wouldn't actually be independent — they'd be implicitly coupled through the agent's working memory.

The repo's answer is **file-based state externalisation**: every long-running task writes to a local task file (e.g. a gitignored `.tasks/<slug>.md` on the dev's machine). Each skill reads the file, writes to it, and assumes nothing about what other skills did beyond what's recorded there. The file is personal working memory, not a team artefact — see [Task files § Where the task file lives](./task-files.md#where-the-task-file-lives-gitignored-local-personal).

The empirical case for this is unusually direct. InfiAgent [\[29\]](./sources.md#29) ran an ablation study removing file-based state externalisation from their long-horizon agent and measured a **21× performance degradation**. Anthropic's three-file pattern [\[20\]](./sources.md#20) and the Claude Code Tasks system [\[23\]](./sources.md#23) converge on the same shape from a different angle. The full case is in [Task files](./task-files.md).

For self-containment specifically, the implication is structural: **state is shared through the file, not through implicit context**. A skill that needs prior findings reads them from the task file. A skill that needs to record a decision writes it to the task file. No skill assumes another skill kept anything in attention.

> Without file-based state, "self-contained skills" would be a polite fiction — every skill would secretly depend on the agent's memory of what its siblings did. The task file is what makes the independence real.

---

## See also

- [Activation](./activation.md) — exclusion clauses name the *task types* a skill is not for, never sibling skill names. The agent disambiguates by matching the user's task to whichever sibling's `ALWAYS apply when…` clause triggers.
- [Body anatomy](./body-anatomy.md) — body length budgets factor in inline restatement.
- [Scope](./scope.md) — the same self-containment principle is what excludes any "personas-core" or "loader" index skill from the repo.
- [Task files](./task-files.md) — the file-based state externalisation that lets self-contained skills coordinate.
- [Sources](./sources.md) — full bibliography.
