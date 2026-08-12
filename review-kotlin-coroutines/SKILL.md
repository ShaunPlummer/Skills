---
name: review-kotlin-coroutines
description: >-
  Reviews idiomatic Kotlin and coroutines/Flow usage in Android/Kotlin changes.
  Correctness crashes/leaks belong to built-in bug review, not this lens.
metadata:
  version: "1.1"
---

# Review Kotlin & Coroutines

## Role

Review **idiomatic Kotlin** and **coroutines/Flow best practice**. Severity here is *code quality*, not crash risk — concrete crash/leak/corruption belongs to built-in bug review. Skip architecture, test coverage, and security. Cooperative cancellation and cleanup belong to `review-kotlin-coroutines-cancellation`.

Match existing project conventions; only flag broadly agreed idiom.

## Idiomatic Kotlin checklist

**Immutability & types**
- `val` over `var`; read-only collections in public APIs; `data class` `copy()` for state.
- Sealed types for closed hierarchies; narrow exposed `MutableStateFlow` / mutable lists.
- Prefer exposing immutable types to other classes, `StateFlow` over `MutableStateFlow`.

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

**Main safe**
- Suspend functions should be main-safe, meaning they're safe to call from the main thread.
- Functions performing long running operations are in charge of moving the execution off the main thread using `withContex`.

**Dispatchers**
- IO/Default at the lowest layer; inject dispatchers for tests; no `runBlocking` on production paths.

**Error handling**
- Flow `catch`/`retry` with bounds; surface errors in UiState. (`CancellationException` swallowing → `review-kotlin-coroutines-cancellation`.)

**Flow / StateFlow**
- Cold flows for streams, `suspend` for one-shots; `stateIn(..., WhileSubscribed(5_000), …)` for UI; `update { }` for RMW; no `.value` polling.

** ViewModel should create coroutines **
- ViewModel classes should prefer creating coroutines instead of exposing suspend functions to perform business logic. Suspend functions in the ViewModel can be useful if instead of exposing state using a stream of data, only a single value needs to be emitted.
- Views shouldn't directly trigger any coroutines to perform business logic. Instead, defer that responsibility to the ViewModel. This makes your business logic easier to test as ViewModel objects can be unit tested, instead of using instrumentation tests that are required to test views.

```Kotlin
// DO create coroutines in the ViewModel
class LatestNewsViewModel(
    private val getLatestNewsWithAuthors: GetLatestNewsWithAuthorsUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow<LatestNewsUiState>(LatestNewsUiState.Loading)
    val uiState: StateFlow<LatestNewsUiState> = _uiState

    fun loadNews() {
        viewModelScope.launch {
            val latestNewsWithAuthors = getLatestNewsWithAuthors()
            _uiState.value = LatestNewsUiState.Success(latestNewsWithAuthors)
        }
    }
}
```

```Kotlin
// Prefer observable state rather than suspend functions from the ViewModel
class LatestNewsViewModel(
    private val getLatestNewsWithAuthors: GetLatestNewsWithAuthorsUseCase
) : ViewModel() {
    // DO NOT do this. News would probably need to be refreshed as well.
    // Instead of exposing a single value with a suspend function, news should
    // be exposed using a stream of data as in the code snippet above.
    suspend fun loadNews() = getLatestNewsWithAuthors()
}
```

** The data and business layer should expose suspend functions and Flows**
Classes in the data and business layers generally expose functions to perform one-shot calls or to be notified of data changes over time. Classes in those layers should expose suspend functions for one-shot calls and Flow to notify about data changes.

```kotlin
// Classes in the data and business layer expose
// either suspend functions or Flows
class ExampleRepository {
    suspend fun makeNetworkRequest() { /* ... */ }

    fun getExamples(): Flow<Example> {
        /* ... */
    }
}
```

**Creating coroutines in the business and data layer**
For classes in the data or business layer that need to create coroutines for different reasons, there are different options.

If the work to be done in those coroutines is relevant only when the user is present on the current screen, it should follow the caller's lifecycle. In most cases, the caller will be the ViewModel, and the call will be cancelled when the user navigates away from the screen and the ViewModel is cleared. In this case, coroutineScope or supervisorScope should be used.


```kotlin
class GetAllBooksAndAuthorsUseCase(
    private val booksRepository: BooksRepository,
    private val authorsRepository: AuthorsRepository,
) {
    suspend fun getBookAndAuthors(): BookAndAuthors {
        // In parallel, fetch books and authors and return when both requests
        // complete and the data is ready
        return coroutineScope {
            val books = async { booksRepository.getAllBooks() }
            val authors = async { authorsRepository.getAllAuthors() }
            BookAndAuthors(books.await(), authors.await())
        }
    }
}
```

## Severity guidance

- 🔴 CRITICAL — `GlobalScope`, hot-path `runBlocking`, hardcoded uninjectable dispatchers in heavily tested layers.

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
