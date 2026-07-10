---
name: review-test-coverage
description: >-
  Reviews unit test adequacy for Android/Kotlin changes — NO TESTS / WEAK TESTS /
  STALE TESTS. Shared checklist for Cursor and Claude.
disable-model-invocation: true
metadata:
  version: "1.1"
---

# Review Test Coverage

Shared knowledge module (Cursor + Claude). Harness-specific launch/scope lives in the Cursor section below or in `.claude/agents/review-test-coverage.md`.

## Cursor (single-lens)

Launch exactly one `test-coverage-reviewer` subagent with `readonly: true` and `run_in_background: false`.

- Default scope: working tree including uncommitted changes (staged, unstaged, untracked) compared against `origin/main`.

Prompt the subagent:

```text
Full Repository Path: <absolute repository path>
Diff: working tree including uncommitted changes vs origin/main
Base Branch: origin/main

Read ~/.cursor/skills/review-test-coverage/SKILL.md.
Follow Role through Output. Ignore the "Cursor (single-lens)" section.
Return only the completed report template.
```

Deliver the subagent's template. Do not fix findings unless asked.

## Role

Judge whether **new/changed logic** has adequate unit tests. Do not judge architecture, Kotlin idiom, production correctness, or security.

Classify every gap as:
- **NO TESTS** — nothing exercises this code.
- **WEAK TESTS** — exist but miss branches/edges/errors or assert too little.
- **STALE TESTS** — pin old behavior, tautological, or pass for the wrong reason.

For each changed production file, **search for tests** (`src/test/`, `commonTest`, `androidUnitTest`, `*Test.kt`, Grep class/function names). Never claim "no tests" without saying where you looked. Read test files in full. Do not run the suite.

## What to check

**Coverage of new logic**
- Behavioral public functions/branches need tests; trivial pass-through/UI markup do not.
- New branches in existing functions: do tests hit them?
- ViewModels: UiState transitions (initial, loading→success/error), `SavedStateHandle` when used.
- NIA mappers `asExternalModel()` / `asEntity()`: test non-mechanical logic (defaults, filtering, conversions).

**Edge cases and error paths**
- Empty/null/boundary inputs; `catch`/failure paths; Flow cancellation only when logic depends on it.

**Assertion quality**
- Not "no throw only", not mock-interaction-only, not assertions that would pass with the bug back.
- Over-mocking noted as weak coverage (fake-vs-mock policy → `review-architecture-recommendations`).

**Staleness**
- Behavior changed but tests not updated honestly; orphaned tests for deleted code.

## Severity guidance

- 🔴 CRITICAL — NO TESTS on consequential logic, or tests edited to pass without protecting intent.
- 🟡 WARNING — WEAK/STALE coverage on branching or error paths.
- 🔵 INFO — nice-to-have cases, parameterization.

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Unit Test Coverage Reviewer

## 1. Executive Summary
<!-- 2-3 sentences -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]
* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Classification:** NO TESTS / WEAK TESTS / STALE TESTS
* **Context:** Function/class + test files checked (or search locations).
* **Issue:** What is wrong.
* **Recommendation:** Specific cases to add (e.g. empty list, repository throws).

---

## 3. Strengths & Positive Feedback
*
```

Repeat finding blocks, most severe first. If adequate: "No coverage gaps found." Cite paths and line numbers.
