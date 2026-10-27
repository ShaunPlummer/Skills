# Ticket: [Feature / Task Name]
<ticket>
<title>
[Concise title describing the ticket]
</title>

<user-story>
As a [type of user]
I want to [perform an action / achieve a goal]
So that [benefit / value]
</user-story>

<description>
[Detailed description of the architectural changes, layers to update (e.g., Data Source, Repository, Use Case, ViewModel, UI), dependencies, and business logic rules required to implement the ticket.]

<dependencies>
[A list of task IDs which must be completed before this ticket]
</dependencies>
</description>

<ui-spec>
# UI Requirements
## [Component / Section Name]
- Component: [e.g., TopAppBar, Card, ListItem]
- Typography: [e.g., MaterialTheme.typography.titleMedium]
- Colors: [e.g., MaterialTheme.colorScheme.onSurface]
- Layout & Dimensions: [e.g., 56dp x 56dp image, 16dp horizontal padding]

## Loading and Error States
- Loading: [e.g., Centered CircularProgressIndicator]
- Error State: [e.g., Text message with error color and retry button]
</ui-spec>

<verification>
# Verification
1. Unit Tests (Domain/Business Logic): Verify that `[UseCaseName]` or business logic correctly handles success, edge cases, and error states using `[FakeRepository/Mock]`.
2. Unit Tests (Data Layer): Verify that `[Repository/DataSource]` correctly fetches, parses, and maps data from `[RemoteDataSource/LocalDatabase/API]` using `[MockHttpClient/TestDatabase]`.
3. State Management / ViewModel Tests: Verify that `[ViewModelName]` properly exposes UI states across initial, loading, success, and error scenarios.
4. UI / Integration Tests: Confirm that `[ScreenName]` renders correctly, responds to user interactions, and handles navigation or side effects as specified.
</verification>

<acceptance-criteria>
# Acceptance Criteria
[The externally observable behaviour of the application]
## AC 1: [Initial / Base State]
Given [initial context or precondition]
When [action occurs]
Then [expected outcome]

## AC 2: [Loading State]
Given [context]
When [request is in progress]
Then [loading indicator behavior]

## AC 3: [Success State]
Given [context]
When [operation succeeds]
Then [success outcome]

## AC 4: [Error State]
Given [context]
When [error occurs]
Then [error message and retry option behavior]
</acceptance-criteria>
</ticket>
