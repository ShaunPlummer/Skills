# Code Review Subagent System

A set of agent-agnostic code-review subagents for Android / Kotlin Multiplatform projects following (or aspiring to) Google's recommended app architecture, plus a coordinator that runs them in parallel and merges their reports.

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
```

- Each reviewer is an **independent lens** — it never sees the others' output. Synthesis happens only at the coordinator.
- Each reviewer returns the same completed report template (executive summary → severity-badged findings → strengths).
- The coordinator groups findings where two or more reviewers hit the same file/line into a **"Conflicting or Overlapping Findings"** section, showing each verdict and severity side by side. It never averages severities or picks a winner — a 🔴-vs-🔵 disagreement is surfaced as-is for the human to judge.

## The agents

| File | Agent | Lens |
|---|---|---|
| `review-coordinator.md` | `code-review-coordinator` | Dispatch + synthesis (no review of its own) |
| `review-architecture-guide.md` | `architecture-guide-reviewer` | Google's *Guide to app architecture*: layer separation, UDF, ViewModel, single source of truth |
| `review-architecture-recommendations.md` | `architecture-recommendations-reviewer` | Android's prescriptive *Recommendations*: lifecycle-aware collection, SavedStateHandle, DI, modularization, testing guidance |
| `review-bugs.md` | `bug-reviewer` | Correctness only: crashes, races, leaks, lifecycle bugs, edge cases |
| `review-test-coverage.md` | `test-coverage-reviewer` | Unit-test adequacy; classifies gaps as NO TESTS / WEAK TESTS / STALE TESTS |
| `review-kotlin-coroutines.md` | `kotlin-coroutines-reviewer` | Kotlin idiom; structured concurrency, dispatchers, error handling, Flow/StateFlow |
| `review-security.md` | `security-reviewer` | Secrets, insecure storage, network config, exported components, WebView, injection |

Severity badges are shared across all reviewers: 🔴 CRITICAL · 🟡 WARNING · 🔵 INFO.

## Usage

Copy this `.claude/agents/` directory into the root of the Android project (it's project-scoped, so check it into version control there). Then:

- **Full review:** ask for the coordinator — *"Use the code-review-coordinator to review my staged changes"* — or let it match automatically on "full code review".
- **Single lens:** invoke one reviewer directly — *"Run the bug-reviewer on this diff"*.
- Scope defaults, when you don't specify one: unstaged diff → staged diff → branch diff vs `origin/<default-branch>` (fetched fresh, then `git diff origin/main...HEAD` — never a possibly-stale local `main`), first non-empty wins. The coordinator pins the base to a commit SHA so all reviewers see the identical range.

## Design notes

- **Read-only:** every reviewer has only `Read`, `Grep`, `Glob`, and `Bash`; Bash is granted solely for read-only git commands (`git diff`, `git log`, `git show`) so agents can scope themselves. Instructions forbid modification. The coordinator additionally has `Task` (to dispatch) and `Write` (solely for the optional `code-review-report.md`).
- **Nested-agent caveat:** some harnesses don't let a subagent spawn subagents. In that case run the coordinator's instructions from the main conversation (or a slash command); the procedure is identical and noted in its file.
- **Project convention:** all agents are told that NIA-style mapper extension functions (`asExternalModel()` for DB/network → domain, `asEntity()` for domain → DB, living in the relevant data module) are the sanctioned pattern here — they review the mappers' internals but never flag the pattern itself.
- **Lane discipline:** each agent's prompt explicitly defers neighboring concerns to its siblings (the bug reviewer drops style notes, the Kotlin reviewer defers crash-risk severity to the bug reviewer, etc.) so the consolidated report doesn't quadruple-report one line — and when two lenses *do* converge, the coordinator surfaces that convergence deliberately instead of deduplicating it away.
