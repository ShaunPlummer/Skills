---
name: viewmodel-navigation-ui-state
description: Models ViewModel-driven navigation as nullable screen-scoped navigation state in UiState (sealed type), with a clear consume callback on the ViewModel. Use when implementing unidirectional data flow, handling navigation from a ViewModel, replacing navigation Channels/SharedFlows, or when the user mentions UiState, StateFlow navigation, or one-shot nav from ViewModel.
---

# ViewModel navigation via UI state

## Goal

Every navigation intent that originates from **business logic** in the ViewModel is represented as **UI state**, not as a fire-and-forget event stream. The UI observes state, performs `navigate`, then tells the ViewModel to clear that field so navigation stays reproducible (configuration changes, optional saved state).

## Naming conventions

Use the **same screen prefix** for the UI state type, navigation type, and ViewModel. Replace `Login` with your screen name (PascalCase prefix).

| Concept | Pattern | Example |
|--------|---------|---------|
| UI state data class | `<Screen>UiState` | `LoginUiState`, `CheckoutUiState` |
| Navigation sealed type | `<Screen>Navigation` | `LoginNavigation`, `CheckoutNavigation` |
| ViewModel class | `<Screen>ViewModel` | `LoginViewModel` |
| State flow property | `uiState` | `val uiState: StateFlow<LoginUiState>` |
| Private mutable backing | `_uiState` | `private val _uiState = MutableStateFlow(...)` |
| Clear after navigation | `onNavigationConsumed()` | `onNavigationConsumed()` |

**Sealed destination names:** use noun or imperative route style inside `<Screen>Navigation`: `Home`, `Help`, `OrderDetails(orderId: String)`.

## Model `<Screen>Navigation`

Prefer a **`sealed class` or `sealed interface`** (not a flat `enum`) when destinations carry different payloads.

```kotlin
sealed interface LoginNavigation {
    data object Home : LoginNavigation
    data object Help : LoginNavigation
    data class VerifyEmail(val token: String) : LoginNavigation
}
```

If every destination is argument-free, `enum` is acceptable; still use the same `<Screen>Navigation` name if the team standardizes on it.

## Model `<Screen>UiState>`

Hold navigation as **nullable** meaning “no pending navigation.”

```kotlin
data class LoginUiState(
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val navigation: LoginNavigation? = null,
)
```

Property name **`navigation`** is the default; alternatives like `pendingNavigation` are fine if the codebase already uses that consistently.

## ViewModel responsibilities

1. Expose `StateFlow<<Screen>UiState>`.
2. When business logic decides to navigate, **`update`** (or `combine` + `copy`) so `navigation` is non-null.
3. Provide **`on<Screen>NavigationConsumed()`** (or `on<Screen>NavigationHandled()`) that sets `navigation` back to `null` after the UI has performed navigation.

```kotlin
class LoginViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun onSignInSuccess() {
        _uiState.update { it.copy(navigation = LoginNavigation.Home) }
    }

    fun onHelpRequested() {
        _uiState.update { it.copy(navigation = LoginNavigation.Help) }
    }

    fun onNavigationConsumed() {
        _uiState.update { it.copy(navigation = null) }
    }
}
```

### Combining with other flows

If `UiState` is built with **`combine`**, keep navigation as one of the inputs (e.g. `MutableStateFlow<LoginNavigation?>` merged into the full `LoginUiState`) so a single `StateFlow` remains the UI’s source of truth.

```kotlin
// Conceptual: merge isLoading, error, and navigation into LoginUiState
val uiState: StateFlow<LoginUiState> = combine(
    isLoadingFlow,
    errorFlow,
    navigationFlow,
) { loading, error, nav ->
    LoginUiState(isLoading = loading, errorMessage = error, navigation = nav)
}.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), LoginUiState())
```

After navigation, clear **`navigationFlow`** / merged field via `onNavigationConsumed()`.

## UI responsibilities

1. Collect `uiState` with a **lifecycle-aware** API (`repeatOnLifecycle` / `collectAsStateWithLifecycle`).
2. When `navigation != null`, map it to `NavController` routes (or equivalent).
3. Call **`viewModel.on<Screen>NavigationConsumed()`** after initiating navigation so the same destination is not re-triggered on recomposition or collector replay.

**Compose (illustrative):**

```kotlin
val uiState by viewModel.uiState.collectAsStateWithLifecycle()

LaunchedEffect(uiState.navigation) {
    val target = uiState.navigation ?: return@LaunchedEffect
    when (target) {
        LoginNavigation.Home -> navController.navigate(HOME_ROUTE) { popUpTo(LOGIN_ROUTE) { inclusive = true } }
        LoginNavigation.Help -> navController.navigate(HELP_ROUTE)
        is LoginNavigation.VerifyEmail -> navController.navigate("verify/${target.token}")
    }
    viewModel.onNavigationConsumed()
}
```

**Views (`lifecycleScope` + `repeatOnLifecycle`):** same rule: read `uiState.navigation`, navigate, then `onNavigationConsumed()`.

## When not to use this alone

If the **current screen stays on the back stack** and valid state can become true again without meaning “navigate again,” add **UI-scoped gating** (e.g. a “user tapped Continue” flag) per [Android guidance on navigation and back stack](https://developer.android.com/topic/architecture/ui-layer/events#navigation-events). The ViewModel still exposes domain state; the extra bit is UI responsibility.

## Checklist

- [ ] Types named `<Screen>UiState` and `<Screen>Navigation`.
- [ ] Nullable `navigation` (or team-standard equivalent) on `UiState`.
- [ ] ViewModel exposes `on<Screen>NavigationConsumed()` (or agreed synonym).
- [ ] UI clears navigation **after** calling `navigate`.
- [ ] Lifecycle-aware collection; avoid navigating blindly while in background if product requires it.

## Anti-patterns

- **Primary** navigation from ViewModel via `Channel` / `SharedFlow` without a state representation: loses easy replay under UDF.
- Leaving `navigation` non-null after navigate: causes duplicate navigation on rotation or re-collection.
- Putting **NavController** inside the ViewModel: navigation side effects and back-stack policy belong in the UI layer.
