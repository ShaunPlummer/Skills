---
name: test-coverage-review
description: Portable checklist and report template for reviewing unit test coverage adequacy on Android/Kotlin changes — missing tests on new logic branches, untested edge cases and error paths, weak assertions, and stale existing tests. Classifies gaps as NO TESTS / WEAK TESTS / STALE TESTS. Domain knowledge only — no orchestration or tool-access assumptions; usable by any AI coding tool.
metadata:
  version: "1.0"
---

# Unit Test Coverage Review

This skill is the knowledge module for one review lens: **is the new/changed logic adequately covered by unit tests?** It does not judge architecture, Kotlin idiom, correctness of production code, or security — other lenses own those. It judges the *tests* (existence, depth, honesty) relative to the *change*.

For every gap, classify it explicitly as one of:
- **NO TESTS** — no test file / no test method exercises this code at all.
- **WEAK TESTS** — tests exist but miss branches, edge cases, or error paths, or assert too little to catch regressions.
- **STALE TESTS** — existing tests no longer reflect what the changed code does (test the old behavior, are now tautological, or pass for the wrong reason).

## Role

For each changed production file, **actively locate its tests** — check the mirrored path under `src/test/` / `commonTest` / `androidUnitTest` (KMP), search for `*Test.kt` files matching the class name, and search for the class/function name across test source sets. Never declare "no tests" without having searched; say where you looked. Read the tests you find in full, not just their names.

## What to check

**Coverage of new logic**
- Every new public function/branch with behavior (conditionals, mapping, calculation, state transition) has at least one test. Trivial pass-through/delegation and pure UI markup do not require unit tests — don't demand tests ritually; demand them where logic lives.
- New branches added to *existing* functions: do existing tests exercise the new branch, or do they all still ride the old path?
- ViewModels: are new UiState emissions/transitions tested (initial state, loading→success, loading→error), including `SavedStateHandle` inputs where used?
- Mappers — including the project's NIA-style `asExternalModel()` / `asEntity()` extension functions, which are the established convention here — should have round-trip/field-mapping tests when they contain any non-mechanical logic (defaults, fallbacks, filtering, date/enum conversion).

**Edge cases and error paths**
- Empty/null/absent inputs, empty collections, boundary values used in the new logic.
- Error paths: does anything test the `catch`/failure branch — repository returning an error, network exception, DB constraint failure? Untested error handling is the most common gap; look for it deliberately.
- Coroutines/Flow: cancellation and ordering only where the logic depends on them (use of `runTest`, turbine-style Flow assertions).

**Assertion quality (weak-test detection)**
- Tests that only assert "no exception thrown", only check the happy path, or assert on the mock's interactions instead of observable results.
- Assertions so loose they'd pass with the bug reintroduced (`assertNotNull` where a value should be compared, asserting list is non-empty instead of contents).
- Copy-pasted tests where the assertion doesn't match the test name.
- Over-mocking that tests the mock wiring rather than behavior (note it as weakening coverage; fake-vs-mock *policy* belongs to the `architecture-recommendations-review` skill).

**Existing tests still make sense**
- Did the change alter behavior that an existing test pins? If the test was updated, does the updated assertion still test something real (or was it just edited to pass)? If it wasn't updated, is it now stale?
- Deleted/renamed code whose tests were left behind or silently dropped.

## Severity guidance

- 🔴 CRITICAL — NO TESTS on new non-trivial logic with real failure consequences (money/data/state machines), or a test edited to pass without preserving its protective intent.
- 🟡 WARNING — WEAK TESTS: happy-path-only coverage of branching logic, untested error paths, assertion-free tests, STALE tests contradicting current behavior.
- 🔵 INFO — nice-to-have cases, suggested parameterization, coverage of defensive branches that are hard to trigger.

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Unit Test Coverage Reviewer

## 1. Executive Summary
<!-- A 2-3 sentence high-level overview of the review findings for this specific lens. -->

---

## 2. Critical Findings & Action Items
<!-- Highly actionable items. Use the appropriate status badge for each point:
     🔴 CRITICAL (Must fix - bugs, crashes, severe architecture violations)
     🟡 WARNING (Highly recommended - performance, test gaps, code smell)
     🔵 INFO (Consideration/Idiomatic suggestion)
-->

### [File Name / Component Name]
* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Classification:** NO TESTS / WEAK TESTS / STALE TESTS
* **Context:** Line numbers or specific function/class name, plus the test file(s) checked (or the locations searched when none exist).
* **Issue:** Clear explanation of what is wrong or sub-optimal.
* **Recommendation:** Step-by-step guidance or code snippet showing how to resolve it (name the specific test cases to add, e.g. "empty list input", "repository throws IOException").

---

## 3. Strengths & Positive Feedback
<!-- Optional but encouraged: Highlight what was done well (e.g., great test coverage on a complex edge case). -->
*
```

Repeat the `### [File Name / Component Name]` block for every finding, most severe first. If coverage is adequate, keep section 2 and write "No coverage gaps found." Always cite concrete file paths and line numbers.
