---
name: write-view-model-test
description: Creates or updates Android ViewModel unit tests for this project using coroutine test best practices, test scope DSL patterns, and UiState Flow assertions. Use when the user asks to write ViewModel tests, test StateFlow/Flow uiState, inject test dispatchers, or fix flaky coroutine-based unit tests.
version: 1.0.0
projectTypes: [android, kotlin]
autoTrigger: true
---

# Write ViewModel Test

## When to use this skill

Use this skill when implementing or updating **local JVM unit tests** for ViewModels that use coroutines, `StateFlow`, or `Flow`.

## Goals

- Produce deterministic ViewModel tests.
- Replace real dispatchers with `TestDispatcher` instances.
- Assert `UiState` emissions clearly.
- Follow Waymap conventions (BDD-style tests and test scope DSL where useful).

## Workflow

1. **Inspect the ViewModel contract**
   - Identify constructor deps, dispatcher/scope usage, and `uiState` type.
   - Confirm whether state is `StateFlow`, `Flow`, or composed from upstream flows.

2. **Set up coroutine test environment**
   - Wrap each test in `runTest`.
   - In local unit tests, replace Main with a `TestDispatcher` (rule preferred).
   - Ensure all `TestDispatcher` instances share one scheduler (`testScheduler`).

3. **Inject test-friendly dependencies**
   - Inject dispatcher/scope into production classes when possible.
   - Prefer real use cases with mocked repositories/datasources.
   - Avoid fire-and-forget APIs; if needed, wait with `advanceUntilIdle()`, `join()`, or `await()`.

4. **Collect and assert UiState**
   - For current state assertions, read `StateFlow.value` after advancing scheduler.
   - For stream assertions, collect from `uiState` using `first()`, `take(n)`, or list capture.
   - Assert user-visible states (`Loading`, `Success`, `Error`) rather than internals.

5. **Use DSL pattern when scope is non-trivial**
   - Create `<Feature>ViewModelTestScope` to hold setup/actions/assertions.
   - Keep test body intent-focused with Given/When/Then comments.

## Default patterns

- **Dispatcher choice**:
  - Use `StandardTestDispatcher(testScheduler)` for precise scheduling.
  - Use `UnconfinedTestDispatcher(testScheduler)` only for simple eager tests.
- **Progressing coroutines**:
  - `advanceUntilIdle()` to complete queued work.
  - `runCurrent()` for current-time tasks only.
- **Main replacement**:
  - Use a reusable `MainDispatcherRule` and reset in teardown.

## Output expectations

When asked to write tests, produce:

- A focused ViewModel test class with `runTest`.
- Main dispatcher override for JVM tests.
- Deterministic dispatcher injection and shared scheduler usage.
- Clear UiState assertions (single-state and/or emission sequence).
- Optional test scope DSL when it improves readability/reuse.

## Review checklist

- [ ] `runTest` used per coroutine test.
- [ ] `Dispatchers.Main` replaced for local ViewModel tests.
- [ ] Test dispatchers share the same scheduler.
- [ ] No flaky timing assumptions (`Thread.sleep`, real delays).
- [ ] Assertions verify UiState behavior from user perspective.
- [ ] Naming/comments follow Kotlin + project conventions.
