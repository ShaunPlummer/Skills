---
name: write-a-test-scope-dsl
description: Designs and implements Kotlin test scope DSLs that group setup, actions, and assertions for unit tests. Use when creating or updating unit tests, introducing a *TestScope helper, or refactoring ViewModel/use-case tests to a BDD-style DSL.
---

# Waymap Test Scope DSL

## Purpose

Design and implement **test scope DSLs** (e.g. `MainSettingsViewModelTestScope`, `SearchHomeViewModelTestScope`) that encapsulate:

- Setup of dependencies and test dispatchers
- Construction of the class under test
- High-level helpers:
  - “Given” setters (`setFoo`)
  - “When” actions (`clickX`, `load`, `submit`)
  - “Then” assertions (`assertX`)

so unit tests stay concise, BDD-style, and easy to read.

For concrete Waymap Android 2 examples, see [examples.md](examples.md).

## How to use this skill

When asked to create or refactor tests for a component:

1. **Identify the component under test**
   - Inspect its constructor and dependencies (use cases, repositories, etc.).
   - Respect existing clean architecture boundaries and module structure.

2. **Choose location and naming**
   - Name the DSL class `<ComponentName>TestScope` (e.g. `MainSettingsViewModelTestScope`).
   - Place it in the matching test source set/package as the production code.
   - Provide a `TestScope` extension in lowerCamelCase, e.g. `mainSettingsViewModelTestScope { ... }`.

3. **Design the test scope class**
   - Accept start-condition config as constructor parameters.
   - Use a `TestCoroutineScheduler`-based dispatcher, usually `StandardTestDispatcher(testScheduler)` or `UnconfinedTestDispatcher(testScheduler)`.
   - Use the scheduler from the active `TestScope` / rule; do not create a new scheduler unless necessary.
   - Prefer real use cases/repositories; mock datasources and SDK/system wrappers with `mockk` (Android) or fakes (KMP).
   - Reuse `testing-util` helpers (`MockHelper`, `MapUtils`, `MapCatalogue`) when available.
   - Lazily construct the class under test as `private val testObject by lazy { ... }`.
   - Expose action helpers and domain-level assertion helpers; hide direct `StateFlow` access from tests.
   - Use `verify` / `coVerify` only when the side effect is the behaviour under test; warn if assertions alone are not ideal.

4. **Shape the tests around the DSL**
   - Write tests with `runTest { ... }` and the `TestScope` extension.
   - Use BDD-style comments: `// GIVEN ...`, `// WHEN ...`, `// THEN ...`.
   - Keep the test body focused on intent.

5. **Comments and style**
   - Prefer short comments that explain *why*.
   - Apply default values on the entrypoint function, not the test scope class.
   - Match naming and structure to existing `*TestScope` utilities in the project.

## Checklist

- [ ] `<Component>TestScope` + `TestScope.<component>TestScope { }` entrypoint
- [ ] Shared test scheduler for all `TestDispatcher`s
- [ ] Real use cases where practical; mocks at datasource/SDK boundary
- [ ] Lazy `testObject`
- [ ] Given/When/Then helpers; tests assert user-visible state
