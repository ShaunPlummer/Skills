# Code Review Subagent System

A set of code-review subagents for Android / Kotlin Multiplatform projects following (or aspiring to) Google's recommended app architecture, plus a coordinator that runs them in parallel and merges their reports.

The system is split into two layers on purpose:

- **Skills** (repo root: `<lens-name>/SKILL.md`, e.g. `bug-review/SKILL.md`) hold all the actual review knowledge — checklists, severity guidance, the report template. They're plain markdown with no Claude Code-specific mechanics, so they're transferable to any AI coding tool that can load a markdown skill/rule file.
- **Subagents** (`.claude/agents/*.md`) are thin Claude Code-specific wrappers: tool access (read-only), scope detection (which diff to review), parallel-dispatch eligibility, and the instruction to load the matching skill and apply it. They are Claude Code's mechanism for isolated-context, parallel execution — that part doesn't port to other tools, so it stays out of the skills.

## How it works

```
                    ┌────────────────────────────┐
 "review my diff" → │   code-review-coordinator  │ → one consolidated Markdown report
                    └─────────────┬──────────────┘
              dispatches in parallel, same scope, blind to each other
   ┌──────────┬──────────┬───────┴───┬───────────┬───────────┐
   ▼          ▼          ▼           ▼           ▼           ▼
 arch       arch       bug        test        kotlin &   security
 guide      recs                  coverage    coroutines
   │          │          │           │            │           │
   ▼          ▼          ▼           ▼            ▼           ▼
each subagent loads its matching skill (repo-root SKILL.md) for its checklist + template
```

- Each reviewer is an **independent lens** — it never sees the others' output. Synthesis happens only at the coordinator.
- Each reviewer returns the same completed report template (executive summary → severity-badged findings → strengths), defined once in its skill.
- The coordinator groups findings where two or more reviewers hit the same file/line into a **"Conflicting or Overlapping Findings"** section, showing each verdict and severity side by side. It never averages severities or picks a winner — a 🔴-vs-🔵 disagreement is surfaced as-is for the human to judge.

## The pieces

### Subagents — `.claude/agents/` (Claude Code-specific, not portable)

| File | Agent | Loads skill |
|---|---|---|
| `review-coordinator.md` | `code-review-coordinator` | — (dispatch + synthesis only, no skill of its own) |
| `review-architecture-guide.md` | `architecture-guide-reviewer` | `architecture-guide-review` |
| `review-architecture-recommendations.md` | `architecture-recommendations-reviewer` | `architecture-recommendations-review` |
| `review-bugs.md` | `bug-reviewer` | `bug-review` |
| `review-test-coverage.md` | `test-coverage-reviewer` | `test-coverage-review` |
| `review-kotlin-coroutines.md` | `kotlin-coroutines-reviewer` | `kotlin-coroutines-review` |
| `review-security.md` | `security-reviewer` | `android-security-review` |

### Skills — repo root (portable, agent-agnostic)

| Directory | Lens |
|---|---|
| `architecture-guide-review/` | Google's *Guide to app architecture*: layer separation, UDF, ViewModel, single source of truth |
| `architecture-recommendations-review/` | Android's prescriptive *Recommendations*: lifecycle-aware collection, SavedStateHandle, DI, modularization, testing guidance |
| `bug-review/` | Correctness only: crashes, races, leaks, lifecycle bugs, edge cases |
| `test-coverage-review/` | Unit-test adequacy; classifies gaps as NO TESTS / WEAK TESTS / STALE TESTS |
| `kotlin-coroutines-review/` | Kotlin idiom; structured concurrency, dispatchers, error handling, Flow/StateFlow |
| `android-security-review/` | Secrets, insecure storage, network config, exported components, WebView, injection |

`android-security-review` is deliberately not named `security-review` — that name collides with a generic security-review skill/command some environments already ship.

Severity badges are shared across all reviewers: 🔴 CRITICAL · 🟡 WARNING · 🔵 INFO.

## Usage

Copy `.claude/agents/` **and** the six top-level `*-review/` skill directories into the root of the Android project (both are meant to be checked into that project's version control). Then:

- **Full review:** ask for the coordinator — *"Use the code-review-coordinator to review my staged changes"* — or let it match automatically on "full code review".
- **Single lens:** invoke one reviewer directly — *"Run the bug-reviewer on this diff"*.
- Scope default, when you don't specify one: `git fetch origin <default-branch>` (never a possibly-stale local `main`), then `git diff origin/main` — single-ref form. That one command covers commits already made on the branch, staged changes, and unstaged edits all at once, so nothing in your current work is silently excluded. The coordinator resolves the fetched ref to a commit SHA and has every reviewer diff against that same SHA, so all six see the identical, working-tree-inclusive scope.
- **Reusing the skills elsewhere:** the six `*-review/SKILL.md` files have no Claude Code-specific content — drop them into any tool that can load a markdown skill/rules file (e.g. a single Cursor agent reading them one at a time) to get the same checklists without the parallel-subagent orchestration.

## Design notes

- **Read-only:** every reviewer subagent has only `Read`, `Grep`, `Glob`, `Bash`, and `Skill`; Bash is granted solely for read-only git commands (`git diff`, `git log`, `git show`, `git fetch`) so agents can scope themselves. Instructions forbid modification. The coordinator additionally has `Task` (to dispatch) and `Write` (solely for the optional `code-review-report.md`).
- **Skill-loading fallback:** each subagent is told to invoke the Skill tool for its named skill, and if that can't resolve a project skill in its execution context, to `Read` the skill file at the repo root directly instead — so the wrapper degrades gracefully rather than failing outright.
- **Nested-agent caveat:** some harnesses don't let a subagent spawn subagents. In that case run the coordinator's instructions from the main conversation (or a slash command); the procedure is identical and noted in its file. Where nesting *is* allowed, the coordinator dispatches reviewers **synchronously in one parallel batch** (not as background agents): background-completion notifications route to the top-level session, not to a nested coordinator, so backgrounded reviewers' results would never reach it. Synchronous parallel calls keep the fan-out concurrent while returning each completed template directly to the coordinator as a tool result.
- **Project convention:** every skill tells its reviewer that NIA-style mapper extension functions (`asExternalModel()` for DB/network → domain, `asEntity()` for domain → DB, living in the relevant data module) are the sanctioned pattern here — they review the mappers' internals but never flag the pattern itself.
- **Lane discipline:** each skill explicitly defers neighboring concerns to its siblings (the bug lens drops style notes, the Kotlin lens defers crash-risk severity to the bug lens, etc.) so the consolidated report doesn't quadruple-report one line — and when two lenses *do* converge, the coordinator surfaces that convergence deliberately instead of deduplicating it away.
