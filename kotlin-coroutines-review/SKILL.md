---
name: kotlin-coroutines-review
description: Portable checklist and report template for reviewing idiomatic Kotlin usage (collections, scope functions, immutability, null-handling) and coroutines best practice — structured concurrency, dispatcher choice and injection, error handling, and Flow/StateFlow usage. Domain knowledge only — no orchestration or tool-access assumptions; usable by any AI coding tool.
metadata:
  version: "1.0"
---

# Kotlin & Coroutines Review

This skill is the knowledge module for one review lens: **idiomatic Kotlin** and **coroutines/Flow best practice**. This is not the bug lens: if a coroutine issue would concretely crash, leak, or corrupt state, the `bug-review` skill owns it — you may still note it briefly from the idiom angle, but severity here reflects *code quality*, not crash risk. Architecture layering, test coverage, and security belong to other lenses; skip them.

## Role

You are a Kotlin language specialist. Match the codebase's existing conventions; consistency beats personal taste — only flag idiom the Kotlin community and Android team broadly agree on.

## Idiomatic Kotlin checklist

**Immutability & types**
- `val` over `var`; `List`/`Map`/`Set` over their mutable variants in public signatures; `data class` `copy()` over in-place mutation for state.
- Data classes for value carriers; sealed interfaces/classes for closed hierarchies (UiState, results) instead of nullable-field bags or type-code enums with casts.
- Exposed mutable state (`MutableStateFlow`/`MutableList` in a public API) should be narrowed (`asStateFlow()`, read-only interfaces).

**Null handling**
- `?.`, `?:`, `let` over `!!` and null-checked `if` pyramids; `requireNotNull`/`checkNotNull` with a message where non-null is an invariant.
- Avoid nullable types where a sealed type or default expresses intent better.

**Collections & functional operators**
- Use the stdlib operator that says what it does: `firstOrNull`, `single`, `any`, `count`, `sumOf`, `associateBy`, `groupBy`, `partition`, `zip` — instead of manual loops with flags/accumulators.
- Chain judiciously: use `asSequence()` for long chains over large collections; conversely don't sequence 3-element lists.
- `buildList`/`buildMap` over temp mutable + `toList()` dances.

**Scope functions & style**
- `let`/`run`/`apply`/`also`/`with` used for their intended shapes; flag nested/chained scope functions that obscure the receiver (`it` vs `this` confusion), not every stylistic choice.
- Expression bodies for single-expression functions; `when` over if-else ladders; named/default arguments over telescoping overloads; extension functions for utility behavior on foreign types.
- **Project convention (respect it, do not flag it):** NIA-style mapper extensions `asExternalModel()` / `asEntity()` in data modules are the established mapping idiom — this is exactly the sanctioned use of extension functions here. Review their internals for idiom; never suggest replacing the pattern.

## Coroutines & Flow checklist

**Structured concurrency**
- Coroutines launched in a lifecycle-owned scope (`viewModelScope`, `lifecycleScope`, injected `CoroutineScope`) — never `GlobalScope`, never an ad-hoc `CoroutineScope(Dispatchers.X)` without an owner responsible for cancelling it.
- `coroutineScope { }`/`supervisorScope { }` for parallel decomposition inside suspend functions instead of passing scopes around.
- Suspend functions don't launch fire-and-forget work into scopes they received as parameters/properties unless that's the documented contract.

**Dispatchers**
- `Dispatchers.IO` for blocking disk/network, `Default` for CPU work, main-safety enforced *at the lowest layer* (`withContext` inside the repository/data source, `flowOn` at the flow source) — not sprinkled at call sites.
- Dispatchers injected (constructor parameter, e.g. `ioDispatcher: CoroutineDispatcher`) rather than hardcoded, for testability with `runTest`.
- No `withContext(Dispatchers.Main)` gymnastics where the caller is already main; no `runBlocking` in production code paths.

**Error handling**
- `try-catch` around specific suspend calls with specific exception types; never `catch (e: Exception)` that swallows `CancellationException` (rethrow it, or catch narrower).
- `CoroutineExceptionHandler` only where it belongs — top-level launched coroutines in owned scopes — not as a substitute for local handling.
- Flows: `catch` operator for upstream errors, `retry`/`retryWhen` with bounds, errors surfaced into the UiState rather than silently dropped.

**Flow / StateFlow efficiency**
- Cold flows for streams, `suspend` for one-shots — flag suspend-wrapped-in-flow-of-one and flow-collected-once-for-a-single-value shapes.
- `stateIn(scope, SharingStarted.WhileSubscribed(5_000), initial)` for flows exposed to UI, so upstream stops when the UI is gone; `shareIn` for cold→hot event streams with intentional replay/buffer.
- Combine streams with `combine`/`flatMapLatest`/`mapLatest` instead of nested `collect` blocks; `distinctUntilChanged` where re-emissions are noise (built into StateFlow — don't add it redundantly).
- `MutableStateFlow.update { }` for read-modify-write; `Flow` transforms kept pure — side effects in `onEach`/`collect`, not `map`.
- No `StateFlow.value` polling loops; no `first()` misused as "wait for valid state" without a filter.

## Severity guidance

- 🔴 CRITICAL — patterns the Kotlin/Android teams explicitly proscribe and that always bite eventually: `GlobalScope`, `runBlocking` on a hot path, swallowing `CancellationException`, hardcoded uninjectable dispatchers in a heavily-tested layer.
- 🟡 WARNING — non-idiomatic code with real cost: unowned ad-hoc scopes, missing `WhileSubscribed` sharing on UI-exposed flows, public mutable state, manual loops replicating stdlib operators in complex logic.
- 🔵 INFO — pure idiom polish: scope-function choice, expression bodies, operator suggestions, naming.

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Kotlin & Coroutines Reviewer

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
* **Context:** Line numbers or specific function/class name.
* **Issue:** Clear explanation of what is wrong or sub-optimal.
* **Recommendation:** Step-by-step guidance or code snippet showing how to resolve it.

---

## 3. Strengths & Positive Feedback
<!-- Optional but encouraged: Highlight what was done well (e.g., clean use of functional operators, well-structured Flow pipeline). -->
*
```

Repeat the `### [File Name / Component Name]` block for every finding, most severe first. If a section has nothing, keep it and write "No findings." Always cite concrete file paths and line numbers.
