
# Ticket: add search functionality
<ticket>
<title>
Add search screen for finding podcasts
</title>

<user-story>
As a user
I want to search for podcasts by keyword
So that I can discover new shows
</user-story>

<description>
Add a data layer method to `PodcastRemoteDataSource` and `DefaultPodcastRepository` to search podcasts via the Podcast Index API. Follow the existing data-source, repository, and dependency-injection conventions. Reuse existing interfaces where available, and introduce or extend interfaces only as needed to substitute fake implementations in tests.

A `SearchPodcastsUseCase` should be created to execute search queries and return matching results. The use case should be responsible for trimming leading and trailing whitespace from search queries before submitting them. It should block searching when the request is empty of blank.

Add a new `SearchScreen` destination to the Jetpack Compose navigation graph backed by a `SearchScreenViewModel`. `SearchScreenViewModel` should be responsible for catching errors and showing the corresponding UI state.

The search functionality should implement a debounce to ensure a query is only submitted when a user stop typing for 300 ms.

The `SearchScreenViewModel` should implement search-as-you-type with a 300 ms debounce. It should initiate a search only after the query has remained unchanged for the debounce interval. Empty or whitespace-only queries must not trigger a network request. The `SearchPodcastsUseCase` is responsible for trimming leading and trailing whitespace and rejecting blank queries before invoking the repository.
</description>

<ui-spec>
# UI Requirements
## Top App Bar / Search Bar
- Component: `SearchBar` (Material 3)
- Placeholder: "Search podcasts..." using `MaterialTheme.typography.bodyLarge`
- Colors: `MaterialTheme.colorScheme.surfaceContainerHigh`

## Search Results List Layout (`Card` or `ListItem`)
- Cover Art: 56dp x 56dp image with `MaterialTheme.shapes.small` (8dp corners)
- Podcast Title: `MaterialTheme.typography.titleMedium` using `MaterialTheme.colorScheme.onSurface`
- Publisher: `MaterialTheme.typography.labelMedium` using `MaterialTheme.colorScheme.onSurfaceVariant`

## Empty / Initial State Screen
- Layout: Centered `Column` with 16dp horizontal padding
- Icon: `Icon(Icons.Default.Search)` sized at 64dp, tinted with `MaterialTheme.colorScheme.outline`
- Title: "Search for podcasts" using `MaterialTheme.typography.titleLarge`
- Body: "Find your favorite shows by title or keyword." using `MaterialTheme.typography.bodyMedium` tinted with `MaterialTheme.colorScheme.onSurfaceVariant`

## Loading and Error States
- Loading: Centered `CircularProgressIndicator` using `MaterialTheme.colorScheme.primary`
- Error State: Text using `MaterialTheme.typography.bodyLarge` with `MaterialTheme.colorScheme.error` and a retry `TextButton`

## No Results State
- Display a message indicating that no podcasts were found for the current query.
- Use the existing Material 3 typography and color scheme.
- Keep the search bar available so the user can modify the query.
- Distinguish this state from the initial search prompt and the error state.
</ui-spec>

<verification>
# Verification
1. Verify that `SearchScreenViewModel` exposes the correct UI state for initial, loading, results, no-results, and error scenarios. Verify retry, query normalization integration, blank-query handling, and debounced search behavior where appropriate. Use the real `SearchPodcastsUseCase` backed by a controllable `FakePodcastRepository`.
2. Unit Tests: Verify `SearchPodcastsUseCaseTest` returns correct search results for a given query string. Make use `FakePodcastRepository` to provide test data.
3. Unit Tests: Verify `DefaultPodcastRepository` returns correct search results for a given query string. Make use `FakePodcastRemoteDataSource` to provide test data.
4. Unit Tests: Verify `PodcastRemoteDataSource` returns correct search results for a given query string. Make of a mock http client to return mock responses to API requests for testing.
</verification>

<acceptance-criteria>
# Acceptance Criteria
## AC 1: Initial Load
Given a user navigates to the search screen 
When no query has been entered, 
Then an initial state screen is displayed prompting the user to search.

## AC 2: Loading
Given a user enters a search query
When the search request is in progress
Then a circular progress loading indicator is displayed using `MaterialTheme.colorScheme.primary`

## AC 3: Search results state
Given a user enters a search query
When results are returned successfully
Then a list of matching podcasts is displayed
And tapping a list item does nothing

## AC 4: No results state
Given a user enters a search query that yields no results
When the search finishes
Then an empty state message is displayed indicating no podcasts were found.

## AC 5: Request error state
Given a user enters a search query
When a network error occurs
Then a user-friendly error message is displayed with a retry option.

### AC 6: Debounced Search
Given the user enters or modifies a non-blank search query,
When the user continues typing within the 300 ms debounce interval,
Then no search request is initiated for the intermediate query values.
And the latest valid query is submitted after 300 ms without further input.

### AC 10: Outdated Search Requests
Given a search request is in progress,
When the user enters a newer query or clears the search field,
Then results or errors from the outdated request must not overwrite the UI state for the current query.

### AC 7: Query Normalization
Given the user enters a query with leading or trailing whitespace,
When the 300 ms debounce interval elapses without further input,
Then the trimmed query is passed to `SearchPodcastsUseCase` for searching.

### AC 8: Empty Query Validation
Given the search query is empty or contains only whitespace,
When the query changes or the debounce interval elapses,
Then no search request is initiated.
And the initial state is displayed when no valid query has been submitted.


### AC 9: Clearing the Search Query
Given the user has entered a search query or received search results,
When the user clears the search field,
Then any pending debounced search is cancelled.
And no search request is made for the empty query.
And the initial search prompt is displayed.
And results from previous queries are no longer displayed.
</acceptance-criteria>
</ticket>

# Ticket: View Podcast Info
<ticket>
<title>
Add podcast detail screen 
</title>
<user-story>
As a user
I want to tap on a podcast from search results to view its details
So that I can learn more about the show
</user-story>

<description>
Update `PodcastRemoteDataSource` and `DefaultPodcastRepository` to expose functionality to fetch podcast metadata given a podcast ID.

Create a `GetPodcastDetailsUseCase` to orchestrate retrieving show information.

Add a new `PodcastDetailScreen` destination to the Jetpack Compose navigation graph backed by a `PodcastDetailScreenViewModel`, supporting navigation from the search results item click.

This ticket depends on #1 having been implemented as it requires updating the existing screen to navigate when a list item is selected.
</description>

<ui-spec>
# UI Requirements
## Top App Bar
- **Component:** `TopAppBar` with a back navigation button
- **Title:** Podcast title using `MaterialTheme.typography.titleLarge`

## Podcast Header Section
- **Cover Art:** 120dp x 120dp image with `MaterialTheme.shapes.medium`
- **Podcast Title:** `MaterialTheme.typography.headlineSmall` using `MaterialTheme.colorScheme.onSurface`
- **Publisher:** `MaterialTheme.typography.bodyMedium` using `MaterialTheme.colorScheme.onSurfaceVariant`
- **Description:** `MaterialTheme.typography.bodySmall` using `MaterialTheme.colorScheme.onSurface` with text truncation

## Loading & Error States
- **Loading:** Centered `CircularProgressIndicator` using `MaterialTheme.colorScheme.primary`
- **Error State:** Text using `MaterialTheme.typography.bodyLarge` with `MaterialTheme.colorScheme.error` and a retry `TextButton`
</ui-spec>

<verification>
1. Unit Tests: Verify `GetPodcastDetailsUseCaseTest` correctly returns data for a given podcast ID.
2. UI Tests: Verify state transitions on `PodcastDetailScreen` (Loading -> Success, Loading -> Error).
3. UI Tests: Confirm navigation from a search result item correctly passes the podcast identifier and opens the detail screen.
</verification>

<verification>
1. Verify that `PodcastDetailScreenViewModel` exposes the correct UI state for loading, success, and error scenarios when fetching podcast metadata by ID. Use the real `GetPodcastDetailsUseCase` backed by a `FakePodcastRepository`.
2. Verify `GetPodcastDetailsUseCaseTest` correctly returns metadata for a given podcast ID. Make use of `FakePodcastRepository` to provide test data.
3. Verify `DefaultPodcastRepository` returns correct podcast details. Make use of `FakePodcastRemoteDataSource` to provide test data.
4. Verify `PodcastRemoteDataSource` returns correct podcast metadata given a podcast ID. Make use of a mock http client to return mock responses to API requests for testing.
5. Confirm navigation from a search result item correctly passes the podcast identifier and opens the detail screen.
</verification>

<acceptance-criteria>
# Acceptance Criteria
## AC 1: Select a podcast
Given a user is viewing search results
When they tap on a podcast item
Then they navigate to the podcast detail screen
And the selected podcast's identifier is passed to the `PodcastDetailScreenViewModel`.

## AC 2: Loading state
Given a user navigates to the podcast detail screen
When the data is loading
Then a circular progress loading indicator is displayed using `MaterialTheme.colorScheme.primary`

## AC 2: Success state
Given a user navigates to the podcast detail screen
When data loads successfully
Then the podcast cover art, title, publisher, and description are displayed

## AC 3: Error state
Given a user navigates to the podcast detail screen
When a network error occurs
Then a user-friendly error message is displayed with a retry option
</acceptance-criteria>
</ticket>

# Ticket: View Podcast Recent Episode List
<ticket>
<title>
Add episode list to podcast detail screen
</title>
<user-story>
As a user
I want to view a list of episodes on a podcast's detail screen
So that I can choose content to listen to
</user-story>

<description>
Update `PodcastRemoteDataSource` and `DefaultPodcastRepository` to expose functionality to fetch episode lists given a podcast ID.

Create a `GetEpisodeListUseCase` to orchestrate retrieving associated episodes for a given podcast.

Update `PodcastDetailScreenViewModel` and `PodcastDetailScreen` to fetch and render the episode list below the podcast metadata section.

This ticket depends on the podcast detail screen metadata ticket having been implemented.
</description>

<ui-spec>
# UI Requirements
## Episode List Item Layout (`Card` or `ListItem`)
- **Episode Title:** `MaterialTheme.typography.titleMedium` using `MaterialTheme.colorScheme.onSurface` (max 2 lines, `TextOverflow.Ellipsis`)
- **Metadata (Date & Duration):** `MaterialTheme.typography.bodySmall` using `MaterialTheme.colorScheme.onSurfaceVariant`

## Loading & Error States
- **Loading:** Centered `CircularProgressIndicator` using `MaterialTheme.colorScheme.primary`
- **Error State:** Text using `MaterialTheme.typography.bodyLarge` with `MaterialTheme.colorScheme.error` and a retry `TextButton`
</ui-spec>

<verification>
# Verification
1. Verify that `PodcastDetailScreenViewModel` correctly handles loading, success, and error states for the episode list. Use the real `GetEpisodeListUseCase` backed by a controllable `FakePodcastRepository`.
2. Verify `GetEpisodeListUseCaseTest` correctly returns episode data for a given podcast ID. Make use of `FakePodcastRepository` to provide test data.
3. Verify `DefaultPodcastRepository` returns the correct episode list for a podcast ID. Make use of `FakePodcastRemoteDataSource` to provide test data.
4. Verify `PodcastRemoteDataSource` returns correct episode items from the API. Make use of a mock http client to return mock responses to API requests for testing.
5. Confirm the episode list renders correctly below the podcast metadata section and handles its own section loading/error states.
</verification>

<acceptance-criteria>
# Acceptance Criteria
## AC 1: Show episode list
Given a user is viewing a podcast detail screen
When the episode list is loading
Then a circular progress loading indicator is displayed for the episode section

## AC 2: Success state
Given a user is viewing a podcast detail screen
When episodes are loaded successfully
Then a scrolling list of episodes is displayed below the podcast metadata

## AC 3: Error state when all requests fail
Given a user is viewing a podcast detail screen
When an error occurs requesting the episode list
And an error occurs requesting the podcast info
Then a user-friendly error message is displayed with a retry option for the episode section

## AC 4: Error state when podcast info fails
Given a user is viewing a podcast detail screen
When an error occurs requesting the episode list regardless of if the episode list is available
Then a user-friendly error message is displayed with a retry option for the episode section

## AC 5: Error state when episode list only fails
Given a user is viewing a podcast detail screen
When podcast info is loaded successfully
And an error occurs requesting the episode list
Then the podcast info is displayed
And a user-friendly error message is displayed with a retry option for the episode section
</acceptance-criteria>
</ticket>
