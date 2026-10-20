---
name: review-coroutines-flow-tests
description: >-
  Review project tests that contain Kotlin coroutine flows for reliable collection, meaningful
  assertions, StateFlow semantics, and stateIn activation. Use for focused test
  reviews and diagnosis of hanging, flaky, or ineffective Flow tests.
disable-model-invocation: true
metadata:
  version: "1.1"
---

# Review Coroutines & Flow Tests

## Role

Review **Flow test correctness and testing practice**. Follow the checklist derived
from the official [Android Flow Testing Guidance](https://developer.android.com/kotlin/flow/test).
Match project conventions; do not require a particular assertion library or rewrite
valid tests for style alone.

Focus on existing tests and their supporting fakes or mocks. Inspect production code only
to establish the tested contract. Skip unrelated architecture, production Kotlin
idioms, security, and general coverage audits. Report review findings; edit code
only when requested.

## Coroutines & Flow testing checklist

**Controlled inputs & fakes**

- For consumers, inject deterministic fake or mock producers: `flow { emit(...) }` for predefined data, or `MutableSharedFlow` / `MutableStateFlow` for dynamic triggers.
- Avoid real network/database implementations in unit tests; prefer mock API responses.

**Emission assertions on finite streams**

- Use `runTest` for suspending tests to auto-skip delays and manage test virtual time.
- Use `first()` for picking a single item; it cancels the flow collection immediately afterward.
- Use `drop(n).first()` for a later item and `take(n).toList()` for a bounded sequence.
- Reserve `toList()`, `single()`, and `count()` exclusively for completing streams.

```kotlin
// DO use first() or take() for streams that do not complete naturally
@Test
fun testFirstEmission() = runTest {
    val value = repository.observeData().first()
    assertEquals(expected, value)
}
```

### Continuous collection & unconfined dispatchers

Launch continuous or non-terminating collectors inside `backgroundScope` so they are automatically cancelled when the test finishes.

Use `UnconfinedTestDispatcher` for collecting flows in tests so emissions are executed eagerly without needing manual dispatcher yields.

```kotlin
// DO launch background collection eagerly with UnconfinedTestDispatcher
@Test
fun testContinuousCollection() = runTest {
    val values = mutableListOf<Item>()
    backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect { values.add(it) }
    }
    viewModel.userAction()
    assertEquals(listOf(InitialState, UpdatedState), values)
}
```

### StateFlow & stateIn activation

StateFlow conflates emissions; assert on the latest `.value` or collect sequentially, acknowledging conflation.

`SharingStarted.WhileSubscribed` requires an active subscriber to begin upstream collection. Keep a collector active in `backgroundScope` before invoking triggering actions.

```kotlin
// DO activate WhileSubscribed flows before triggering state changes
@Test
fun testWhileSubscribedStateIn() = runTest {
    val collector = backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.stateFlow.collect {}
    }
    // Upstream is now active and ready to process actions
}
```

## Severity guidance

🔴 CRITICAL — Unbounded `toList()` on a hot flow causing infinite test hangs, unhandled background job leaks, or race conditions from unshared dispatchers.

🟡 WARNING — Missing subscriber for WhileSubscribed StateFlow, manual delays instead of virtual time advancement (`advanceUntilIdle`), or asserting intermediate states on a conflating StateFlow.

🔵 INFO — Style polish, adopting Turbine over verbose manual collectors, or replacing redundant `take(1).toList()` with `first()`.

## Output — complete this template exactly and return it as your entire final response

```markdown

# Code Review: Coroutines & Flow Tests Reviewer

## 1. Executive Summary

<!-- 2–3 sentences; describe scope and whether tests were run or only inspected. -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]

* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** File path and line numbers or function/class name.
* **Issue:** What is wrong, with the relevant execution sequence and impact.
* **Recommendation:** The smallest effective correction.

---

## 3. Strengths & Positive Feedback

* [A specific strength supported by the reviewed code.]
```

That's a great first start, but it's missing the https://developer.android.com/kotlin/flow/test#statein section.

The stateIn guidance from Google's official coroutine testing documentation has been incorporated into SKILL.md, aligning its layout with the structure of format-example-SKILL.md.
`MD`
`+ 1`

## Key stateIn Additions & Structural Enhancements

SharingStarted Execution Strategies: Explicitly differentiates how `SharingStarted.Eagerly`, `SharingStarted.Lazily`, and `SharingStarted.WhileSubscribed` behave during unit tests.
`MD`

Background Subscription Pattern: Demonstrates how to launch an active collector in `backgroundScope` using `UnconfinedTestDispatcher` to trigger upstream production in WhileSubscribed or Lazily flows before executing test actions.
`MD`
`+ 1`

Code Snippets: Adds concrete DO and DO NOT Kotlin code examples under the stateIn section, matching the code-driven layout of format-example-SKILL.md.
`MD`

Frontmatter Alignment: Added disable-model-invocation: true to the frontmatter header to prevent unintended automatic executions.
`MD`

## Updated SKILL.md

```yaml
---
name: review-coroutines-flow-tests
description: >-
  Review Android Kotlin coroutine Flow tests for reliable collection, meaningful
  assertions, StateFlow semantics, and stateIn activation. Use for focused test
  reviews and diagnosis of hanging, flaky, or ineffective Flow tests.
disable-model-invocation: true
metadata:
  version: "1.1"
---
```

# Review Coroutines & Flow Tests

## Role

Review **Flow test correctness and testing practice**. Follow the checklist derived
from [Testing Kotlin flows on Android](https://developer.android.com/kotlin/flow/test).
Match project conventions; do not require a particular assertion library or rewrite
valid tests for style alone.

Focus on existing tests and their supporting fakes. Inspect production code only
to establish the tested contract. Skip unrelated architecture, production Kotlin
idioms, security, and general coverage audits. Report review findings; edit code
only when requested.

## Coroutines & Flow testing checklist

**Controlled inputs & fakes**

- For consumers, inject deterministic fake producers: `flow { emit(...) }` for predefined data, or `MutableSharedFlow` / `MutableStateFlow` for dynamic triggers.
- Avoid real network or database implementations in unit tests.

**Emission assertions on finite streams**

- Use `runTest` for suspending tests to handle virtual time automatically.
- Choose `first()` for picking a single item; it cancels flow collection immediately afterward.
- Use `drop(n).first()` for a later item and `take(n).toList()` for a bounded sequence.
- Reserve unbounded `toList()`, `single()`, and `count()` exclusively for completing streams.

```kotlin
// DO use first() or take() for streams that do not complete naturally
@Test
fun testFirstEmission() = runTest {
    val value = repository.observeData().first()
    assertEquals(expected, value)
}
```

### Continuous collection

Interleave triggers and assertions with a separate collector.

Launch nonterminating collectors in `backgroundScope` so they are automatically cleaned up when the test finishes.

Use `UnconfinedTestDispatcher` for collecting flows in tests so emissions execute eagerly without manual dispatcher advancement.

### Turbine

Treat Turbine (`app.cash.turbine`) as optional but preferred for sequential assertions.

Trigger emissions inside `test {}`; consume expected items with `awaitItem()` and clean up with `cancelAndIgnoreRemainingEvents()`.

### StateFlow & stateIn testing

`SharingStarted.Eagerly`: Starts upstream collection immediately when created; does not require an active test subscriber.

`SharingStarted.Lazily` & WhileSubscribed: Require an active subscriber to begin upstream collection. Reading .value directly without an active collector will return the initial value without executing upstream logic.

Active Subscription Pattern: Launch an active background collector in `backgroundScope` using `UnconfinedTestDispatcher` prior to invoking test actions.

```kotlin
// DO launch background collection before triggering actions on stateIn flows
@Test
fun testWhileSubscribedStateIn() = runTest {
    // Activate the WhileSubscribed stateIn operator
    val collectJob = backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect {}
    }

    // Perform action and assert state change
    viewModel.loadData()
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

```kotlin
// DO NOT assert on WhileSubscribed stateIn flows without an active subscriber
@Test
fun testWhileSubscribedStateIn_Incorrect() = runTest {
    viewModel.loadData()
    // FAILS: Upstream flow was never started because no collector is subscribed
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

## Severity guidance

🔴 CRITICAL — Unbounded `toList()` on a hot flow causing test hangs, or evaluating `SharingStarted.WhileSubscribed` flows without an active subscriber resulting in false positives/negatives.

🟡 WARNING — Assertion mismatches due to StateFlow conflation, missing `backgroundScope` cleanup on continuous collectors, or hardcoded delay usage instead of virtual time.

🔵 INFO — Small maintainability improvements, adopting Turbine for complex sequential streams, or substituting redundant `take(1).toList()` with `first()`.

## Output — complete this template exactly and return it as your entire final response

```markdown

# Code Review: Coroutines & Flow Tests Reviewer

## 1. Executive Summary

<!-- 2–3 sentences; describe scope and whether tests were run or only inspected. -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]

* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** File path and line numbers or function/class name.
* **Issue:** What is wrong, with the relevant execution sequence and impact.
* **Recommendation:** The smallest effective correction.

---

## 3. Strengths & Positive Feedback

* [A specific strength supported by the reviewed code.]
```

Can you give me a complete example for downloading?

```yaml
---
name: review-coroutines-flow-tests
description: >-
  Review Android Kotlin coroutine Flow tests for reliable collection, meaningful
  assertions, StateFlow semantics, and stateIn activation. Use for focused test
  reviews and diagnosis of hanging, flaky, or ineffective Flow tests.
disable-model-invocation: true
metadata:
  version: "1.1"
---
```

# Review Coroutines & Flow Tests

## Role

Review **Flow test correctness and testing practice**. Follow the checklist derived
from [Testing Kotlin flows on Android](https://developer.android.com/kotlin/flow/test)[cite: 1].
Match project conventions; do not require a particular assertion library or rewrite
valid tests for style alone.

Focus on existing tests and their supporting fakes. Inspect production code only
to establish the tested contract. Skip unrelated architecture, production Kotlin
idioms, security, and general coverage audits[cite: 1, 2]. Report review findings; edit code
only when requested.

## Coroutines & Flow testing checklist

**Controlled inputs & fakes**

- For consumers, inject deterministic fake producers: `flow { emit(...) }` for predefined data, or `MutableSharedFlow` / `MutableStateFlow` for dynamic triggers.
- Avoid real network or database implementations in unit tests.

**Emission assertions on finite streams**

- Use `runTest` for suspending tests to handle virtual time automatically.
- Choose `first()` for picking a single item; it cancels flow collection immediately afterward.
- Use `drop(n).first()` for a later item and `take(n).toList()` for a bounded sequence.
- Reserve unbounded `toList()`, `single()`, and `count()` exclusively for completing streams.

```kotlin
// DO use first() or take() for streams that do not complete naturally
@Test
fun testFirstEmission() = runTest {
    val value = repository.observeData().first()
    assertEquals(expected, value)
}
```

### Continuous collection

Interleave triggers and assertions with a separate collector.
`MD`

Launch nonterminating collectors in `backgroundScope` so they are automatically cleaned up when the test finishes.
`MD`

Use `UnconfinedTestDispatcher` for collecting flows in tests so emissions execute eagerly without manual dispatcher advancement.
`MD`

### Turbine

Treat Turbine (`app.cash.turbine`) as optional but preferred for sequential assertions[cite: 1].

Trigger emissions inside `test {}`; consume expected items with `awaitItem()` and clean up with `cancelAndIgnoreRemainingEvents()`[cite: 1].

### StateFlow & stateIn testing

`SharingStarted.Eagerly`: Starts upstream collection immediately when created; does not require an active test subscriber[cite: 1].

`SharingStarted.Lazily` & WhileSubscribed: Require an active subscriber to begin upstream collection[cite: 1]. Reading .value directly without an active collector will return the initial value without executing upstream logic[cite: 1].

Active Subscription Pattern: Launch an active background collector in `backgroundScope` using `UnconfinedTestDispatcher` prior to invoking test actions[cite: 1].

```kotlin
// DO launch background collection before triggering actions on stateIn flows
@Test
fun testWhileSubscribedStateIn() = runTest {
    // Activate the WhileSubscribed stateIn operator
    val collectJob = backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect {}
    }

    // Perform action and assert state change
    viewModel.loadData()
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

```kotlin
// DO NOT assert on WhileSubscribed stateIn flows without an active subscriber
@Test
fun testWhileSubscribedStateIn_Incorrect() = runTest {
    viewModel.loadData()
    // FAILS: Upstream flow was never started because no collector is subscribed
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

## Severity guidance

🔴 CRITICAL — Unbounded `toList()` on a hot flow causing test hangs, or evaluating `SharingStarted.WhileSubscribed` flows without an active subscriber resulting in false positives/negatives[cite: 1].

🟡 WARNING — Assertion mismatches due to StateFlow conflation, missing `backgroundScope` cleanup on continuous collectors, or hardcoded delay usage instead of virtual time[cite: 1].

🔵 INFO — Small maintainability improvements, adopting Turbine for complex sequential streams, or substituting redundant `take(1).toList()` with `first()`[cite: 1].

## Output — complete this template exactly and return it as your entire final response

```markdown

# Code Review: Coroutines & Flow Tests Reviewer

## 1. Executive Summary

<!-- 2–3 sentences; describe scope and whether tests were run or only inspected. -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]

* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** File path and line numbers or function/class name.
* **Issue:** What is wrong, with the relevant execution sequence and impact.
* **Recommendation:** The smallest effective correction.

---

## 3. Strengths & Positive Feedback

* [A specific strength supported by the reviewed code.]
```

I need a complete SKILL.md file I can download.

### Continuous collection

Interleave triggers and assertions with a separate collector.

Launch nonterminating collectors in `backgroundScope` so they are automatically cleaned up when the test finishes.

Use `UnconfinedTestDispatcher` for collecting flows in tests so emissions execute eagerly without manual dispatcher advancement.

```kotlin
// DO launch background collection eagerly with UnconfinedTestDispatcher
@Test
fun testContinuousCollection() = runTest {
    val values = mutableListOf<Item>()
    backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect { values.add(it) }
    }
    viewModel.userAction()
    assertEquals(listOf(InitialState, UpdatedState), values)
}
```

### Turbine

Treat Turbine (`app.cash.turbine`) as optional but preferred for sequential assertions.

Trigger emissions inside `test {}`; consume expected items with `awaitItem()` and clean up with `cancelAndIgnoreRemainingEvents()`.

```kotlin
// DO use Turbine for sequential flow assertions
@Test
fun testWithTurbine() = runTest {
    viewModel.uiState.test {
        assertEquals(State.Loading, awaitItem())
        viewModel.load()
        assertEquals(State.Success, awaitItem())
        cancelAndIgnoreRemainingEvents()
    }
}
```

### StateFlow & stateIn testing

`SharingStarted.Eagerly`: Starts upstream collection immediately when created; does not require an active test subscriber.

`SharingStarted.Lazily` & WhileSubscribed: Require an active subscriber to begin upstream collection. Reading .value directly without an active collector will return the initial value without executing upstream logic.

Active Subscription Pattern: Launch an active background collector in `backgroundScope` using `UnconfinedTestDispatcher` prior to invoking test actions.

```kotlin
// DO launch background collection before triggering actions on stateIn flows
@Test
fun testWhileSubscribedStateIn() = runTest {
    // Activate the WhileSubscribed stateIn operator
    val collectJob = backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect {}
    }

    // Perform action and assert state change
    viewModel.loadData()
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

```kotlin
// DO NOT assert on WhileSubscribed stateIn flows without an active subscriber
@Test
fun testWhileSubscribedStateIn_Incorrect() = runTest {
    viewModel.loadData()
    // FAILS: Upstream flow was never started because no collector is subscribed
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

## Severity guidance

🔴 CRITICAL — Unbounded `toList()` on a hot flow causing test hangs, or evaluating `SharingStarted.WhileSubscribed` flows without an active subscriber resulting in false positives/negatives.

🟡 WARNING — Assertion mismatches due to StateFlow conflation, missing `backgroundScope` cleanup on continuous collectors, or hardcoded delay usage instead of virtual time.

🔵 INFO — Small maintainability improvements, adopting Turbine for complex sequential streams, or substituting redundant `take(1).toList()` with `first()`.

## Output — complete this template exactly and return it as your entire final response

```markdown

# Code Review: Coroutines & Flow Tests Reviewer

## 1. Executive Summary

<!-- 2–3 sentences; describe scope and whether tests were run or only inspected. -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]

* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** File path and line numbers or function/class name.
* **Issue:** What is wrong, with the relevant execution sequence and impact.
* **Recommendation:** The smallest effective correction.

---

## 3. Strengths & Positive Feedback

* [A specific strength supported by the reviewed code.]
```

```python
"""

with open("SKILL.md", "w", encoding="utf-8") as f:
    f.write(content)

print("File generated successfully: SKILL.md")
```

```text?code_stdout&code_event_index=1
File generated successfully: SKILL.md
```

Your Markdown file is ready.

`MD icon`
SKILL
`MD`

## Summary of Enhancements Included in this File

Structural Parity: Matches format-example-SKILL.md frontmatter (disable-model-invocation: true), heading structures, and output template requirements.
`MD`

Official Google Guidance Alignment: Fully incorporates all rules from Testing Kotlin flows on Android, including continuous collection using `backgroundScope` + `UnconfinedTestDispatcher`, finite vs. infinite stream assertions, Turbine usage, and explicit stateIn / `SharingStarted.WhileSubscribed` activation patterns.
`MD`
`+ 1`

Code Examples: Added concrete DO and DO NOT code snippets for flow testing patterns to mirror the structure of format-example-SKILL.md.
`MD`

Gemini is AI and can make mistakes, including about people. Your privacy & Gemini — Opens in a new window

Analyzing

```yaml
---
name: review-coroutines-flow-tests
description: >-
  Review Android Kotlin coroutine Flow tests for reliable collection, meaningful
  assertions, StateFlow semantics, and stateIn activation. Use for focused test
  reviews and diagnosis of hanging, flaky, or ineffective Flow tests.
disable-model-invocation: true
metadata:
  version: "1.1"
---
```

# Review Coroutines & Flow Tests

## Role

Review **Flow test correctness and testing practice**. Follow the checklist derived
from [Testing Kotlin flows on Android](https://developer.android.com/kotlin/flow/test).
Match project conventions; do not require a particular assertion library or rewrite
valid tests for style alone.

Focus on existing tests and their supporting fakes. Inspect production code only
to establish the tested contract. Skip unrelated architecture, production Kotlin
idioms, security, and general coverage audits. Report review findings; edit code
only when requested.

## Coroutines & Flow testing checklist

**Controlled inputs & fakes**

- For consumers, inject deterministic fake producers: `flow { emit(...) }` for predefined data, or `MutableSharedFlow` / `MutableStateFlow` for dynamic triggers.
- Avoid real network or database implementations in unit tests.

**Emission assertions on finite streams**

- Use `runTest` for suspending tests to handle virtual time automatically.
- Choose `first()` for picking a single item; it cancels flow collection immediately afterward.
- Use `drop(n).first()` for a later item and `take(n).toList()` for a bounded sequence.
- Reserve unbounded `toList()`, `single()`, and `count()` exclusively for completing streams.

```kotlin
// DO use first() or take() for streams that do not complete naturally
@Test
fun testFirstEmission() = runTest {
    val value = repository.observeData().first()
    assertEquals(expected, value)
}
```

**Continuous collection**

- Interleave triggers and assertions with a separate collector.
- Launch nonterminating collectors in `backgroundScope` so they are automatically cleaned up when the test finishes.
- Use `UnconfinedTestDispatcher` for collecting flows in tests so emissions execute eagerly without manual dispatcher advancement.

```kotlin
// DO launch background collection eagerly with UnconfinedTestDispatcher
@Test
fun testContinuousCollection() = runTest {
    val values = mutableListOf<Item>()
    backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect { values.add(it) }
    }
    viewModel.userAction()
    assertEquals(listOf(InitialState, UpdatedState), values)
}
```

**Turbine**

- Treat Turbine (`app.cash.turbine`) as optional but preferred for sequential assertions.
- Trigger emissions inside `test {}`; consume expected items with `awaitItem()` and clean up with `cancelAndIgnoreRemainingEvents()`.

```kotlin
// DO use Turbine for sequential flow assertions
@Test
fun testWithTurbine() = runTest {
    viewModel.uiState.test {
        assertEquals(State.Loading, awaitItem())
        viewModel.load()
        assertEquals(State.Success, awaitItem())
        cancelAndIgnoreRemainingEvents()
    }
}
```

**StateFlow & stateIn testing**

- **`SharingStarted.Eagerly`:** Starts upstream collection immediately when created; does not require an active test subscriber.
- **`SharingStarted.Lazily` & WhileSubscribed:** Require an active subscriber to begin upstream collection. Reading `.value` directly without an active collector will return the initial value without executing upstream logic.
- **Active Subscription Pattern:** Launch an active background collector in `backgroundScope` using `UnconfinedTestDispatcher` prior to invoking test actions.

```kotlin
// DO launch background collection before triggering actions on stateIn flows
@Test
fun testWhileSubscribedStateIn() = runTest {
    // Activate the WhileSubscribed stateIn operator
    val collectJob = backgroundScope.launch(UnconfinedTestDispatcher(testScheduler)) {
        viewModel.uiState.collect {}
    }

    // Perform action and assert state change
    viewModel.loadData()
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

```kotlin
// DO NOT assert on WhileSubscribed stateIn flows without an active subscriber
@Test
fun testWhileSubscribedStateIn_Incorrect() = runTest {
    viewModel.loadData()
    // FAILS: Upstream flow was never started because no collector is subscribed
    assertEquals(UiState.Success, viewModel.uiState.value)
}
```

## Severity guidance

- 🔴 CRITICAL — Unbounded `toList()` on a hot flow causing test hangs, or evaluating `SharingStarted.WhileSubscribed` flows without an active subscriber resulting in false positives/negatives.
- 🟡 WARNING — Assertion mismatches due to `StateFlow` conflation, missing `backgroundScope` cleanup on continuous collectors, or hardcoded delay usage instead of virtual time.
- 🔵 INFO — Small maintainability improvements, adopting Turbine for complex sequential streams, or substituting redundant `take(1).toList()` with `first()`.

## Output — complete this template exactly and return it as your entire final response

```markdown

# Code Review: Coroutines & Flow Tests Reviewer

## 1. Executive Summary

<!-- 2–3 sentences; describe scope and whether tests were run or only inspected. -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]

* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** File path and line numbers or function/class name.
* **Issue:** What is wrong, with the relevant execution sequence and impact.
* **Recommendation:** The smallest effective correction.

---

## 3. Strengths & Positive Feedback

* [A specific strength supported by the reviewed code.]
```

SKILL.md
Displaying SKILL.md.
