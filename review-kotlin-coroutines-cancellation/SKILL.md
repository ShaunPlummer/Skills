---
name: review-kotlin-coroutines-cancellation
description: >-
  Reviews cooperative coroutine cancellation and cleanup in Android/Kotlin
  changes — scope/job cancel, isActive/ensureActive/yield, CancellationException
  handling, NonCancellable cleanup, suspendCancellableCoroutine. Use for
  cancellation review or as a panel lens from a code review coordinator.
disable-model-invocation: true
metadata:
  version: "1.1"
---

# Review Kotlin Coroutines Cancellation

## Role

Review **cooperative cancellation and cleanup** only. Skip general Kotlin idiom, Flow/`stateIn` polish, architecture, test coverage, crash/leak bugs, and security (those belong to other lenses).

Match existing project conventions; only flag broadly agreed cancellation practice.

## Cancellation checklist

**Scope and job cancel**
- Use a scope whose lifetime matches the work. Android scopes (`viewModelScope`, `lifecycleScope`, custom owned scopes) have different lifetimes — pick the one that ends when the work should stop. Avoid `GlobalScope` or unowned ad-hoc scopes that never cancel.
- Cancel the **scope** to stop all children; cancel a single `Job` only when intentional (siblings must keep running).
- A cancelled scope cannot launch new work — do not keep using it after `cancel()`.

**Cooperative cancellation**
- `cancel()` moves the job to Cancelling; CPU/blocking loops and multi-chunk IO keep running unless code checks for cancellation.
- Before each chunk of long-running non-suspend work (e.g. each file read), check with `isActive`, `ensureActive()`, or `yield()`.
- All `kotlinx.coroutines` suspend APIs (`delay`, `withContext`, etc.) are already cancellable — do **not** demand extra checks when those are the only suspend points.

**How to check**
- Prefer `ensureActive()` for fail-fast periodic checks (throws `CancellationException` when inactive).
- Use `isActive` when you need to leave the loop and run non-suspending cleanup/logging afterward.
- Use `yield()` when CPU-bound work should also give other coroutines a chance to run.

**join / cancelAndJoin / await after cancel**
- Prefer `cancelAndJoin()` when the caller must wait until the job has finished cancelling.
- `cancel` then `await` — `await` throws `CancellationException` (no result). That exception may represent either the caller’s cancellation or the `Deferred`’s; do not “fix” by swallowing it.
- `join`/`await` then `cancel` — no effect (already completed).

**Cleanup and side effects**
- Put required cleanup in `finally` (or after a cooperative `isActive` loop exits).
- Catch `CancellationException` only to rethrow or to run cleanup; never swallow it in a broad `catch (Exception)`.
- Use `withContext(NonCancellable)` only for **suspending** cleanup that must complete, and keep that block narrow. A coroutine in Cancelling cannot suspend otherwise.
- Warn against `launch(NonCancellable)` / `async(NonCancellable)` — they bypass structured cancellation; prefer a narrow `withContext(NonCancellable)` in `finally`.
- Callback bridges: prefer `suspendCancellableCoroutine` + `continuation.invokeOnCancellation` over plain `suspendCoroutine`.

## Severity guidance

Base severity on **demonstrated impact**, not solely on the pattern.

- 🔴 CRITICAL — cancellation failure with clear waste, stuck work, or broken lifecycle (e.g. non-cooperative long-running work that keeps running after the owning scope should end; cleanup that must run but is skipped because suspending work in Cancelling has no `NonCancellable`).
- 🟡 WARNING — scope lifetime mismatched to the work; `suspendCoroutine` where cancellable is appropriate; `await` after cancel without handling; swallowing `CancellationException` where it undermines structured concurrency (not automatically critical in every context).
- 🔵 INFO — prefer `ensureActive` over manual `isActive` boilerplate; optional `yield` for CPU fairness; overly broad `NonCancellable` blocks.

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Kotlin Coroutines Cancellation Reviewer

## 1. Executive Summary
<!-- 2-3 sentences -->

---

## 2. Findings & Action Items

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
