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
Add a data layer method to `PodcastRemoteDataSource` and `DefaultPodcastRepository` to search podcasts via the Podcast Index API.

A `SearchPodcastsUseCase` should be created to execute search queries and return matching results.

Add a new `SearchScreen` destination to the Jetpack Compose navigation graph backed by a `SearchScreenViewModel`.
</description>

<ui-spec>
### Top App Bar / Search Bar
- Component: `SearchBar` (Material 3)
- Placeholder: "Search podcasts..." using `MaterialTheme.typography.bodyLarge`
- Colors: `MaterialTheme.colorScheme.surfaceContainerHigh`

### Search Results List Layout (`Card` or `ListItem`)
- Cover Art: 56dp x 56dp image with `MaterialTheme.shapes.small` (8dp corners)
- Podcast Title: `MaterialTheme.typography.titleMedium` using `MaterialTheme.colorScheme.onSurface`
- Publisher: `MaterialTheme.typography.labelMedium` using `MaterialTheme.colorScheme.onSurfaceVariant`

### Empty / Initial State Screen
- Layout: Centered `Column` with 16dp horizontal padding
- Icon: `Icon(Icons.Default.Search)` sized at 64dp, tinted with `MaterialTheme.colorScheme.outline`
- Title: "Search for podcasts" using `MaterialTheme.typography.titleLarge`
- Body: "Find your favorite shows by title or keyword." using `MaterialTheme.typography.bodyMedium` tinted with `MaterialTheme.colorScheme.onSurfaceVariant`

### Loading and Error States
- Loading: Centered `CircularProgressIndicator` using `MaterialTheme.colorScheme.primary`
- Error State: Text using `MaterialTheme.typography.bodyLarge` with `MaterialTheme.colorScheme.error` and a retry `TextButton`
</ui-spec>

<verification>
1. Unit Tests: Verify `SearchPodcastsUseCaseTest` returns correct search results for a given query string.
2. UI Tests: Verify state transitions on `SearchScreen` (Initial to Loading to Success, Loading to Empty, Loading to Error).
3. UI Tests: Confirm search query and scroll position are preserved across configuration changes.
4. Manual QA: Perform searches and verify responsiveness and accurate handling of empty and network error states.
</verification>

<acceptance-criteria>
- Given a user navigates to the search screen, when no query has been entered, then an initial state screen is displayed prompting the user to search.
- Given a user enters a search query, when the search request is in progress, then a circular progress loading indicator is displayed using `MaterialTheme.colorScheme.primary`.
- Given a user enters a search query, when results are returned successfully, then a list of matching podcasts is displayed.
- Given a user enters a search query that yields no results, when the search finishes, then an empty state message is displayed indicating no podcasts were found.
- Given a user enters a search query, when a network error occurs, then a user-friendly error message is displayed with a retry option.
</acceptance-criteria>
</ticket>
