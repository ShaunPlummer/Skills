# Test scope DSL examples

Patterns below mirror Waymap Android 2 (`waymap-android-2`): real use cases over mocked datasources/SDKs, `testing-util` helpers where available, lazy `testObject`, Given setters / When actions / Then assertions, and a `TestScope` entrypoint with defaults on the extension (not the class).

## ViewModel test scope

Inspired by `MainSettingsViewModelTestScope`, `SearchHomeViewModelTestScope`, and `LegTransitionViewModelTestScope`.

```kotlin
package com.waymap.settings.main

import com.waymap.domain.catalogue.Deployment
import com.waymap.mapdata.usecase.deployment.GetActiveDeploymentUseCase
import com.waymap.testingutil.MapUtils
import com.waymap.testingutil.MockHelper
import com.waymap.userprofile.model.AccessKeyTypes
import com.waymap.userprofile.usecase.accesskeys.CheckHasAccessTypeUseCase
import io.mockk.every
import io.mockk.mockk
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.test.TestScope
import kotlinx.coroutines.test.UnconfinedTestDispatcher
import kotlinx.coroutines.test.advanceUntilIdle
import org.junit.Assert.assertEquals

/**
 * Test scope DSL for [MainSettingsViewModel].
 *
 * Sets up dependencies (mocked access checks; real [GetActiveDeploymentUseCase] over a
 * test map repository), exposes Given/When helpers, and asserts on emitted UiState.
 */
@OptIn(ExperimentalCoroutinesApi::class)
class MainSettingsViewModelTestScope(
    currentDeployment: Deployment?,
    deploymentList: List<Deployment>,
    testScope: TestScope,
) {
    private val dispatcher = UnconfinedTestDispatcher(testScope.testScheduler)

    private val checkHasAccessTypeUseCase = mockk<CheckHasAccessTypeUseCase> {
        every { this@mockk.invoke(any()) } returns flowOf(false)
    }

    private val mapSdk = MockHelper.mockkMapSdk(deploymentList)
    private val mapRepository = MapUtils.createMapRepository(mapSdk).also { repo ->
        currentDeployment?.let { repo.setAutoDetectedDeployment(it.id) }
    }

    private val testObject: MainSettingsViewModel by lazy {
        MainSettingsViewModel(
            getCurrentDeploymentUseCase = GetActiveDeploymentUseCase(mapRepository),
            checkHasAccessTypeUseCase = checkHasAccessTypeUseCase,
            dispatcher = dispatcher,
        )
    }

    /** Given: configure developer access before collecting state. */
    fun setDeveloperAccess(hasDevAccess: Boolean) {
        every { checkHasAccessTypeUseCase(AccessKeyTypes.Developer) } returns flowOf(hasDevAccess)
    }

    /** When: drive a ViewModel action, then advance the shared scheduler. */
    fun refresh() {
        testObject.onRefresh()
        testScope.advanceUntilIdle()
    }

    /** Then: assert user-visible UiState, not internals. */
    suspend fun assertDevSettingsDisplayed(expected: Boolean) {
        val state = testObject.uiState.first()
        check(state is MainSettingsUiState.Success) { "Expected Success, got: $state" }
        assertEquals(expected, state.hasDevAccess)
    }

    suspend fun assertCorrectCityDisplayed(expectedCity: String) {
        val state = testObject.uiState.first()
        check(state is MainSettingsUiState.Success) { "Expected Success, got: $state" }
        assertEquals(expectedCity, state.currentDeployment)
    }
}

@OptIn(ExperimentalCoroutinesApi::class)
suspend fun TestScope.mainSettingsViewModelTestScope(
    currentDeployment: Deployment?,
    deploymentList: List<Deployment>,
    block: suspend MainSettingsViewModelTestScope.() -> Unit,
) {
    MainSettingsViewModelTestScope(
        currentDeployment = currentDeployment,
        deploymentList = deploymentList,
        testScope = this,
    ).block()
}
```

### Consuming test

```kotlin
class MainSettingsViewModelTest {

    @Test
    fun `developer settings displayed when true`() = runTest(timeout = TestTimeout) {
        // GIVEN the settings screen is opened
        mainSettingsViewModelTestScope(
            currentDeployment = MapCatalogue.London.deployment,
            deploymentList = MapCatalogue.England.deployment,
        ) {
            // WHEN developer access is enabled
            setDeveloperAccess(true)

            // THEN developer settings are shown
            assertDevSettingsDisplayed(expected = true)
        }
    }
}
```

## Use case test scope

Inspired by `CheckIsFeatureAvailableUseCaseTestScope`.

```kotlin
package com.waymap.featconfig.usecase

import com.waymap.featconfig.FeatureConfigRepository
import com.waymap.featconfig.domain.Feature
import io.mockk.every
import io.mockk.mockk
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.test.TestScope
import kotlinx.coroutines.test.runCurrent
import org.junit.Assert.assertEquals

@OptIn(ExperimentalCoroutinesApi::class)
class CheckIsFeatureAvailableUseCaseTestScope(
    enableOptionalFeatures: Boolean?,
    private val testScope: TestScope,
) {
    private val featureConfigRepository = mockk<FeatureConfigRepository> {
        every { isAvailable(any()) } returns flowOf(enableOptionalFeatures)
    }

    private val testObject = CheckIsFeatureAvailableUseCase(featureConfigRepository)

    suspend fun assertFeatureAvailability(
        feature: Feature = Feature.SUPPORT,
        expected: Boolean,
    ) {
        val actual = testObject(feature).first()
        testScope.runCurrent()
        assertEquals(expected, actual)
    }
}

suspend fun TestScope.checkIsFeatureAvailableUseCaseTestScope(
    enableOptionalFeatures: Boolean? = null,
    block: suspend CheckIsFeatureAvailableUseCaseTestScope.() -> Unit,
) {
    CheckIsFeatureAvailableUseCaseTestScope(
        enableOptionalFeatures = enableOptionalFeatures,
        testScope = this,
    ).block()
}
```

## Patterns to copy from Waymap 2

| Pattern | Practice |
|---------|----------|
| Entrypoint | `suspend fun TestScope.<name>TestScope(... defaults ..., block)` |
| Under test | `private val testObject by lazy { ... }` |
| Dispatchers | Share `testScope.testScheduler` (`UnconfinedTestDispatcher` / `StandardTestDispatcher`) |
| Repos / use cases | Prefer real use cases; mock SDKs, datasources, and system/network wrappers via `mockk` / `MockHelper` / `MapUtils` |
| Fixtures | Reuse `MapCatalogue`, `MockJourneyHelpers`, and other `testing-util` helpers |
| Assertions | Prefer domain/UiState asserts; use `coVerify` only when the side effect is the behaviour under test |
| Advancing time | `advanceUntilIdle()` / `runCurrent()` after actions that enqueue coroutine work |
