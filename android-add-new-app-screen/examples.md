# Examples for `add-new-app-screen`

These examples are based on your Android Studio file templates and follow the Waymap conventions described in `.cursorrules`. They are written with template placeholders such as `${NAME}` and `${PACKAGE_NAME}` so they can be adapted per screen.

`${NAME}` is the **screen base name**, for example `SearchResults`. This means:
- Composable file: `SearchResultsScreen.kt`
- State file: `SearchResultsUiState.kt`
- ViewModel file: `SearchResultsViewModel.kt`
- Nav extension: `fun NavController.toSearchResults(...)`

## Screen template

Typically placed in a feature module source set, for example:
`feat/search/src/main/java/com/waymap/search/results/SearchResultsScreen.kt`.
```kotlin
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.safeDrawingPadding
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.waymap.common.theme.PreviewCore
import com.waymap.common.theme.WaymapTheme

#end
#parse("File Header.java")
@Composable
fun ${NAME}Screen(
    modifier: Modifier = Modifier,
    viewModel: ${NAME}ViewModel = hiltViewModel(),
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle(${NAME}UiState.Loading)

    ${NAME}Screen(
        modifier = modifier,
        uiState = uiState,
    )
}

@Composable
private fun ${NAME}Screen(
    uiState: ${NAME}UiState,
    modifier: Modifier = Modifier,
) {
    val scroll = rememberScrollState()
    Column(
        modifier = modifier
            .safeDrawingPadding()
            .fillMaxSize()
            .verticalScroll(scroll),
    ) {
        // TODO: Implement screen content
    }
}

@PreviewCore
@Composable
private fun Preview() {
    WaymapTheme {
        ${NAME}Screen(
            uiState = ${NAME}UiState.Loading,
        )
    }
}
```

## State class template

Typically placed alongside the screen in the same package, for example:
`feat/search/src/main/java/com/waymap/search/results/SearchResultsUiState.kt`.
```kotlin
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}

#end
#parse("File Header.java")
/**
 * UI state for the ${NAME} screen.
 */
sealed interface ${NAME}UiState {
    /**
     * Loading.
     */
    data object Loading : ${NAME}UiState

    /**
     * Error.
     */
    data object Error : ${NAME}UiState

    /**
     * Screen successfully loaded.
     */
    data object Success : ${NAME}UiState

    /**
     * Default state used for previews and initial loading.
     */
    companion object {
        val DEFAULT: ${NAME}UiState = Loading
    }
}
```

## ViewModel template

Typically placed alongside the screen in the same package, for example:
`feat/search/src/main/java/com/waymap/search/results/SearchResultsViewModel.kt`.
```kotlin
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}

import androidx.lifecycle.ViewModel
import dagger.hilt.android.lifecycle.HiltViewModel
import javax.inject.Inject
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

#end
#parse("File Header.java")
@HiltViewModel
class ${NAME}ViewModel @Inject constructor(
) : ViewModel() {
    private val _uiState = MutableStateFlow(${NAME}UiState.DEFAULT)
    val uiState: StateFlow<${NAME}UiState> = _uiState.asStateFlow()
}
```

## NavController extension example

Place navigation helpers alongside existing ones (for example, in a `*Nav.kt` file such as `SearchNav.kt`), and follow the naming and destination patterns already in use:

```kotlin
import androidx.navigation.NavController

/**
 * Navigate to the ${NAME} screen.
 *
 * This helper prepares the route for ${NAME} but leaves wiring into the navigation graph
 * (for example, adding the composable<...> entry) to be implemented separately.
 */
fun NavController.to${NAME}() {
    // TODO: Implement route object or string for ${NAME} and call navigate(...)
}
```

Use this pattern as a starting point and adapt arguments and destination types to the specific feature module where the new screen lives.

## Navigation pattern mirroring `SearchNav.kt`

The following non-template example shows the typical structure used in Waymap’s navigation code:

```kotlin
import androidx.navigation.NavController
import androidx.navigation.NavGraphBuilder
import androidx.navigation.compose.composable
import kotlinx.serialization.Serializable

/**
 * Destination for the NewScreen composable.
 */
@Serializable
data class NewScreenDestination(
    val someArg: String,
)

fun NavGraphBuilder.newScreenGraph(
    navController: NavController,
) {
    composable<NewScreenDestination> {
        NewScreen(
            // Pass callbacks and state as needed
        )
    }
}

fun NavController.toNewScreen(
    someArg: String,
) {
    navigate(NewScreenDestination(someArg))
}
```

When adding a new screen with this skill, the agent should:

- Define a `@Serializable` destination data class when the feature uses typed destinations.
- Add a `composable<NewScreenDestination>` (or equivalent) into the appropriate `*Graph` function for the chosen NavHost (for example, the graph used from `BottomSheetNavHost` or `journeyPlannerGraph`).
- Use a `NavController` extension (like `toNewScreen`) to navigate to the new destination.

