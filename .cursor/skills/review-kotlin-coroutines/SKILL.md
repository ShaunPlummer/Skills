---
name: review-kotlin-coroutines
description: >-
  Reviews idiomatic Kotlin and coroutines/Flow usage in Android/Kotlin changes.
  Shared checklist for Cursor and Claude. Correctness crashes/leaks belong to
  built-in bug review (Cursor Bugbot / Claude /code-review), not this lens.
disable-model-invocation: true
metadata:
  version: "1.1"
---

# Review Kotlin & Coroutines

Shared knowledge module (Cursor + Claude). Harness-specific launch/scope lives in the Cursor section below or in `.claude/agents/review-kotlin-coroutines.md`.

## Cursor (single-lens)

Launch exactly one `kotlin-coroutines-reviewer` subagent with `readonly: true` and `run_in_background: false`.

- Default scope: working tree including uncommitted changes (staged, unstaged, untracked) compared against `origin/main`.

Prompt the subagent:

```text
Full Repository Path: <absolute repository path>
Diff: working tree including uncommitted changes vs origin/main
Base Branch: origin/main

Read ~/.cursor/skills/review-kotlin-coroutines/SKILL.md.
Follow Role through Output. Ignore the "Cursor (single-lens)" section.
Return only the completed report template.
```

Deliver the subagent's template. Do not fix findings unless asked.

## Role

Review **idiomatic Kotlin** and **coroutines/Flow best practice**. Severity here is *code quality*, not crash risk — concrete crash/leak/corruption belongs to built-in bug review. Skip architecture, test coverage, and security.

Match existing project conventions; only flag broadly agreed idiom.

## Idiomatic Kotlin checklist

**Immutability & types**
- `val` over `var`; read-only collections in public APIs; `data class` `copy()` for state.
- Sealed types for closed hierarchies; narrow exposed `MutableStateFlow` / mutable lists.

**Null handling**
- `?.` / `?:` / `let` over `!!` and null pyramids; `requireNotNull`/`checkNotNull` with messages for invariants.

**Collections**
- Prefer stdlib operators (`firstOrNull`, `any`, `associateBy`, …) over manual loops; `asSequence()` only when it pays off; `buildList`/`buildMap` when building.

**Scope functions & style**
- Use `let`/`run`/`apply`/`also`/`with` for their shapes; flag nested/chained confusion.
- **Project convention:** NIA `asExternalModel()` / `asEntity()` — review internals, do not flag the pattern.

## Coroutines & Flow checklist

**Structured concurrency**
- Lifecycle-owned or injected scopes — never `GlobalScope` or unowned ad-hoc `CoroutineScope`.
- `coroutineScope`/`supervisorScope` for parallel work inside suspend functions.

**Dispatchers**
- IO/Default at the lowest layer; inject dispatchers for tests; no `runBlocking` on production paths.

**Error handling**
- Do not swallow `CancellationException`; Flow `catch`/`retry` with bounds; surface errors in UiState.

**Flow / StateFlow**
- Cold flows for streams, `suspend` for one-shots; `stateIn(..., WhileSubscribed(5_000), …)` for UI; `update { }` for RMW; no `.value` polling.

## Severity guidance

- 🔴 CRITICAL — `GlobalScope`, hot-path `runBlocking`, swallowed `CancellationException`, hardcoded uninjectable dispatchers in heavily tested layers.
- 🟡 WARNING — unowned scopes, missing `WhileSubscribed` on UI flows, public mutable state, complex manual loops reinventing stdlib.
- 🔵 INFO — polish (scope-function choice, expression bodies, naming).

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Kotlin & Coroutines Reviewer

## 1. Executive Summary
<!-- 2-3 sentences -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]
* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** Line numbers or function/class name.
* **Issue:** What is wrong.
* **Recommendation:** How to fix.

---

## 3. Strengths & Positive Feedback
*
```

Repeat finding blocks, most severe first. Empty sections: write "No findings." Cite paths and line numbers.
