---
name: write-a-test-scope-dsl
description: Designs and implements Kotlin test scope DSL to use when creating or updating unit tests. All unit tests should make use of a test scope to contain unit test helpers that group setup, dependencies, and assertion functions for ViewModels, use cases, and other classes under test.
version: 1.0.0
projectTypes: [android, kotlin]
autoTrigger: true
---

# Waymap Test Scope DSL

## Purpose

This skill helps design and implement **test scope DSLs** (e.g. `UserHeightViewModelTestScope`, `MapViewModelTestScope`) that encapsulate:
- Setup of dependencies and test dispatchers
- Construction of the class under test
- High-level actions and assertion helpers, including:
	- “Given” setters (setFoo)
	- “When” actions (clickX, load, submit)
	- “Then” assertions (assertX)
so unit tests stay concise, BDD-style, and easy to read.


## How to use this skill

When asked to create or refactor tests for a component:

1. **Identify the component under test**
   - Inspect its constructor and dependencies (use cases, repositories, etc.).
   - Respect the existing clean architecture boundaries and module structure.

2. **Choose location and naming**
   - Name the DSL class `<ComponentName>TestScope` (e.g. `UserHeightViewModelTestScope`).
   - Place it in the appropriate test module, reflecting the package structure of the production code.
   - Provide a `TestScope` extension function named in lowerCamelCase, e.g. `userHeightViewModelTestScope { ... }`.

3. **Design the test scope class**

   In the `*TestScope` class:
   - Accept any required configuration (e.g. saved state, seed data, flags) as constructor parameters. This should allow unit tests to set the start conditions for any test steps.
   - Create a `TestCoroutineScheduler`-based dispatcher, usually:
     - `private val dispatcher = StandardTestDispatcher(testScheduler)`
   - Use the scheduler from the active TestScope / rule; don’t instantiate a new scheduler unless you have a strong reason.”
   - Where possible use real instances of constructor arguments such as use cases or repositories. If this is not possible, ask for guidance.
   - Within Android applications create `mockk` instances datasources or classes that wrap system APIS or network access.
   - Within Kotlin Multiplatform projects use fakes.
   - Prefer explicit stubs for “inputs”
   - Prefer state assertions for “outputs”
   - Build **real** use case instances that wrap the mocked repositories.
   - Lazily construct the class under test:
     - `private val testObject: <TypeUnderTest> by lazy { ... }`
   - Expose:
     - **Action helpers** to drive behaviour (e.g. `setMeasurementUnits(units: MeasurementUnits)`).
     - **Assertion helpers** that verify state in terms of your domain (e.g. `assertIsMetric()`, `assertIsImperial()`), hiding direct `StateFlow` access from tests.
   - Use verify {} only when behavior/side-effect is the purpose of the test. Warn me the code has been written in a way assertions are not the ideal solution.
4. **Shape the tests around the DSL**

   - Write tests using `runTest { ... }` and the `TestScope` extension, e.g.:
     - `userHeightViewModelTestScope { /* given / when / then */ }`
   - Follow BDD-style comments:
     - `// Given ...`, `// When ...`, `// Then ...`
   - Keep the test body focused on **intent**, delegating mechanical setup and assertions to the DSL.

5. **Comments and style**

   - Write **clear, concise code comments** that explain _why_ rather than _what_.
   - Prefer short inline comments over long blocks; use KDoc only when it adds meaningful context.
   - Keep the DSL API self-explanatory so tests read like a narrative.

## Template for a new test scope DSL
For examples, see [examples.md](examples.md).
Use this as a starting point when generating a new test scope; adapt names, types, and dependencies to the specific component:

```kotlin
/**
 * Creates and executes an [ExampleViewModelTestScope] inside a coroutine test.
 *
 * Use this DSL to write concise, behaviour-focused tests without manually
 * constructing dependencies or accessing internal flows.
 *
 * ### Example
 *
 * ```
 * class ExampleViewModelTest {
 *
 *     @Test
 *     fun `given initial state when updating something then state is updated`() =
 *         // Given
 *         exampleViewModelTestScope(
 *             initialState = SomeState.Initial
 *         ) {
 *
 *             // When
 *             updateSomething(SomeValue("42"))
 *
 *             advanceUntilIdle()
 *
 *             // Then
 *             assertState(SomeState.Updated("42"))
 *         }
 * }
 * ```
 *
 * Tests should express intent using the scope's action and assertion helpers
 * rather than directly interacting with the ViewModel or its internal state.
 */
class ExampleViewModelTestScope(
    private val initialState: SomeState?,
    private val testScheduler: TestCoroutineScheduler,
) {
    private val dispatcher = StandardTestDispatcher(testScheduler)

    private val exampleRepository = mockk<ExampleRepository> {
        every { getSomething() } returns initialState
    }

    private val exampleUseCase = ExampleUseCase(exampleRepository)

    private val testObject: ExampleViewModel by lazy {
        ExampleViewModel(
            exampleUseCase = exampleUseCase,
            dispatcher = dispatcher,
        )
    }

    fun updateSomething(value: SomeValue) {
        testObject.onSomethingChanged(value)
    }

    suspend fun assertState(expected: SomeState) {
        // if state can change due to coroutines, advance first in the test
        val state = testObject.uiState.first()
        assertEquals(expected, state)
    }
}

// Test DSL entrypoint
suspend fun TestScope.exampleViewModelTestScope(
    initialState: SomeState? = null,
    block: suspend ExampleViewModelTestScope.() -> Unit,
) {
    ExampleViewModelTestScope(
        initialState = initialState,
        testScheduler = testScheduler,
    ).block()
}
```

When generating a real DSL, always:
- Match naming and structure to existing `*TestScope` utilities in the project.
- Reuse existing helpers from the `testing-util` module when possible.
- Keep the public DSL surface small, intention-revealing, and well-commented but not verbose.
- Apply default values to the entrypoint function, not the test scope class
