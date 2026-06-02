---
name: add-new-app-screen
description: Adds a new screen to the Waymap Android app by asking for the target module, package, screen name, and NavHost, then generating a minimal Hilt-enabled Compose screen, ViewModel, UI state, and NavController extension (plus graph entry) without adding DI modules or tests unless explicitly requested. Use when the user asks to create a new screen or navigation destination.
version: 1.0.0
autoTrigger: true
projectTypes: [android, kotlin]
---

# Add New App Screen

## When to use this skill

Use this skill whenever a **new screen** is requested for the Android app and there is not yet a corresponding Compose screen, ViewModel, and UI state.

If there is not enough information to proceed, the skill should **ask the user explicit questions** before making any changes.

### Required information to collect

Before generating files, confirm:

1. **Target module**
   - Ask: “Which module should this screen live in?”  
     Examples: `feat/search`, `feat/journeyplanning`, `app`.
2. **Target package**
   - Ask: “What package should I use?”  
     Follow the convention `com.waymap.<module>.<feature>.<component>`, for example `com.waymap.search.results`.
3. **Screen base name**
   - Ask: “What should the screen base name be?”  
     Examples: `SearchResults`, `JourneyPlanner`, `UserProfile`.
4. **Navigation host / graph**
   - Ask: “Which NavHost or navigation graph should this screen be added to?”  
     Examples: `BottomSheetNavHost` (search graph), `journeyPlannerGraph`, or another feature graph.

If any of these are ambiguous or missing, do not guess; ask for clarification.

## What this skill should create

When invoked, this skill should guide the creation of three core pieces plus a navigation helper:

1. **Jetpack Compose screen**
   - A stateful entry-point composable that:
     - Accepts a `Modifier` as the last parameter with default `Modifier`.
     - Obtains its ViewModel via `hiltViewModel()`.
     - Collects UI state using `collectAsStateWithLifecycle(...)`, with a sensible default such as `Loading`.
   - A stateless/internal composable that:
     - Receives the UI state and callbacks as parameters.
     - Applies `safeDrawingPadding()` and standard layout patterns (scrolling, size, etc.).
   - A `@PreviewCore` preview wrapped in `WaymapTheme`.

2. **UI state sealed interface**
   - A sealed interface named `<ScreenName>UiState` with, at minimum:
     - `Loading` state.
     - `Error` state.
     - `Success` or equivalent “loaded” state.
   - Prefer including a `companion object` with a sensible `DEFAULT` value when appropriate, following project conventions in `.cursorrules`.

3. **ViewModel using Hilt**
   - A `@HiltViewModel`-annotated class named `<ScreenName>ViewModel`.
   - Constructor-injected dependencies (if any), using `@Inject`.
   - A `StateFlow`/`Flow` named `uiState` of type `<ScreenName>UiState`, initialised to `Loading` at minimum.

4. **Navigation helper extension**
   - Add an **extension function** on `NavController` named `to<ScreenName>` or `navigateTo<ScreenName>` (match existing naming in the target module).
   - Place this extension in the appropriate navigation file for that feature (for example, alongside `toLocationDetail` and `toBuildingDirectory` in a `*Nav.kt` file).
   - The function should expose any required arguments as parameters and prepare a type-safe route object if the feature already uses `@Serializable` destinations.
   - When it is clear which NavHost / graph the screen belongs to, **add the corresponding `composable<Destination>` entry** into the relevant `*Graph` builder function (for example, `searchGraph` or `journeyPlannerGraph`). If the correct place is unclear, leave a clear `TODO` and instructions instead of modifying the graph.

## Scope and non-goals

To keep behaviour predictable:

- The skill **does not** add new DI modules or wiring outside the ViewModel unless the user explicitly requests it.
- The skill **does not** add tests automatically. If tests are desired, it should:
  - Ask which module/package tests should live in.
  - Follow the existing ViewModel test DSL patterns (for example, test scopes described in `.cursorrules`).
  - Only generate test scaffolding when clearly requested.

## Behaviour and workflow

When using this skill to add a new screen:

1. **Clarify naming and location**
   - Confirm the module (for example, `feat/search`) and package (for example, `com.waymap.search.home`).
   - Confirm the screen base name, for example `SearchResults` or `JourneyPlanner`, and derive:
     - Composable file name: `<BaseName>Screen.kt`
     - State file name: `<BaseName>UiState.kt`
     - ViewModel file name: `<BaseName>ViewModel.kt`

   | Base name       | Composable file           | State file                 | ViewModel file                 | Nav extension                                   | Example package                |
   |:----------------|:--------------------------|:---------------------------|:-------------------------------|:-----------------------------------------------|:-------------------------------|
   | `SearchResults` | `SearchResultsScreen.kt`  | `SearchResultsUiState.kt`  | `SearchResultsViewModel.kt`    | `fun NavController.toSearchResults(...)`       | `com.waymap.search.results`    |
   | `JourneyPlanner`| `JourneyPlannerScreen.kt` | `JourneyPlannerUiState.kt` | `JourneyPlannerViewModel.kt`   | `fun NavController.toJourneyPlanner(...)`      | `com.waymap.journey.planner`   |

2. **Follow existing navigation patterns**
   - Use the pattern from `SearchNav.kt` and `BottomSheetNavHost.kt`:
     - Define destinations as `@Serializable` data classes when appropriate.
     - Keep navigation helpers as `NavController` extensions (for example, `toLocationDetail`, `toBuildingDirectory`).
     - Let composables accept callbacks for navigation rather than calling `NavController` directly inside UI where possible.

3. **Respect Waymap conventions**
   - Follow Kotlin and Compose style rules from `.cursorrules`.
   - Use sealed interfaces for UI state, with `Loading`, `Error`, and `Success` (or similar) variants.
   - Separate stateful and stateless composables, with state collected using `collectAsStateWithLifecycle`.
   - Use `WaymapTheme` and `@PreviewCore` for previews.

4. **Navigation setup**
   - Create the `NavController` extension function for the new screen in the appropriate navigation file.
   - Ask which NavHost / graph the screen belongs to (for example, `BottomSheetNavHost` via `searchGraph`, or `journeyPlannerGraph`), then:
     - Add a new `@Serializable` destination data class if the feature uses typed destinations.
     - Add a `composable<Destination>` entry into the relevant `*Graph` function that calls the new screen composable.
   - If the correct graph or insertion point is not obvious from the user’s answer or the codebase, leave a `TODO` and clearly explain where and how the `composable<Destination>` entry should be added manually.

## Examples

For concrete examples based on the Waymap Android Studio file templates (including placeholders like `${NAME}` and `${PACKAGE_NAME}`), see:

- [examples.md](examples.md)

These templates show how to structure:

- The Compose screen with stateful and stateless layers.
- The sealed UI state interface.
- The Hilt-enabled ViewModel and its exposed `uiState`.
- A navigation pattern that mirrors `SearchNav.kt`, including a `@Serializable` destination, a `NavGraphBuilder` entry, and a `NavController` extension.
