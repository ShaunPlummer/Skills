---
name: android-data-layer
description: Create, modify, refactor, or debug Android application data layers working with application data. Use whenever a task touches repositories, data sources, in-memory application state or caches or data persistence. When offline reads/writes, synchronization need to be modified. This includes when these changes are implicit in a feature request. Exclude purely visual UI layout changes with no data behavior impact.
---

# Android data layer

The data layer contains application data and business logic. The business logic is what gives value to your app—it's made of real-world business rules that determine how application data must be created, stored, and changed.

# Data layer architecture

The data layer is made of repositories that each can contain zero to many data sources. You should create a repository class for each different type of data you handle in your app.

The repository takes data sources as dependencies in its constructor. Each data source class should have the responsibility of working with only one source of data, which can be a file, a network source, or a local database.

Other layers in the hierarchy should never access data sources directly; the entry points to the data layer are always the repository classes.

The data exposed by this layer should be immutable so that it cannot be tampered with by other classes.

## Repository Responsibilities
- Exposing data to the rest of the app.
- Centralizing changes to the data.
- Resolving conflicts between multiple data sources.
- Abstracting sources of data from the rest of the app.
- Containing business logic.

```kotlin
class ExampleRepository(
    private val exampleRemoteDataSource: ExampleRemoteDataSource, // network
    private val exampleLocalDataSource: ExampleLocalDataSource // database
) { /* ... */ }
```

# Expose APIs
Classes in the data layer generally expose functions to perform one-shot CRUD operations or to be notified of data changes over time. The data layer should expose the following for each of these cases:

- For one-shot operations, expose suspend functions.
- To be notified of data changes over time, expose flows.

```kotlin
class ExampleRepository(
    private val exampleRemoteDataSource: ExampleRemoteDataSource, // network
    private val exampleLocalDataSource: ExampleLocalDataSource // database
) {

    val data: Flow<Example> = ...

    suspend fun modifyData(example: Example) { ... }
}
```

# Naming conventions

Repository classes are named after the data that they're responsible for. Typically in the format "<type of data> + Repository". For example: NewsRepository, MoviesRepository, or PaymentsRepository.

Data source classes are named after the data they're responsible for and the source they use. Typically in the format "<type of data> + <type of source> + DataSource.". For the type of data, use values like `Remote`, `Local` or `InMemory` to be more generic because implementations can change. To be more specific in case the source is important, use the type of the source. For example: `NewsNetworkDataSource` or `NewsDiskDataSource`.

Don't name the data source based on an implementation detail—for example, `UserSharedPreferencesDataSource`.


# Multiple levels of repositories

With complex business requirements, a repository might need to depend on other repositories. This could be because the data involved is an aggregation from multiple data sources, or because the responsibility needs to be encapsulated in another repository class.

For example, a repository that handles user authentication data, `UserRepository`, could depend on other repositories such as `LoginRepository` and `RegistrationRepository` to fulfill its requirements.

# Source of truth

Each repository defines a single source of truth. The source of truth always contains data that is consistent, correct, and up-to-date. In fact, the data exposed from the repository should always be the data coming directly from the source of truth.

The source of truth can be a data source—for example, the database—or even an in-memory cache that the repository might contain. Repositories combine different data sources and solve any potential conflicts between the data sources to update the single source of truth regularly or due to a user input event.

Different repositories in your app might have different sources of truth. For example, the `LoginRepository` class might use its cache as the source of truth and the `PaymentsRepository` class might use the network data source.

In order to provide offline-first support, a local data source—such as a database—is the recommended source of truth.

# Threading

Calling data sources and repositories should be main-safe. These classes are responsible for moving the execution of their logic to the appropriate thread when performing long-running blocking operations.

# Represent business models

The data models exposed from the data layer should return only the information your application needs. Often Use seperate model objects representing models used by different data sources. These should be kept internal to the data layer.

For example `NewsResponse` represents the data returned by a network datasource and `NewsArticleEntity` the model used by a local data source with `NewsArticle` being exposed to other layers.

```
data/
├─ local/
│ ├─ entities/
│ │ ├─ NewsArticleEntity
│ ├─ dao/
│ ├─ NewsArticleLocalDataSource
├─ network/
│ ├─ NewsNetworkDataSource`
│ ├─ models/
│ │ ├─ NewsResponse
├─ model/
│ ├─ NewsArticle
├─ repository/
│ ├─ NewRepository
```

# Lifecycle

If the class's responsibility is crucial for the whole application, you can scope an instance of that class to the `Application` class using the `@Singleton` hilt annotation. Alternatively, if you only need to reuse the same instance in a particular flow in your app—for example, the registration or login flow—then you should scope the instance to the class that owns the lifecycle of that flow. For example, you could scope a `RegistrationRepository` that contains in-memory data to a backstack entry using a `NavEntryDecorator`.

## Data Mapping

Make use of extension functions to map between different types. For example

```kotlin
/**
 * Converts the network model to the local model for persisting
 * by the local data source
 */
fun NewsResponse.asEntity() = NewsArticleEntity(
    id = id,
    name = name,
    imageUrl = imageUrl,
    twitter = twitter,
    mediumPage = mediumPage,
    bio = bio,
)

/**
 * Converts the local model to the external model for use
 * by layers external to the data layer
 */
fun NewsArticleEntity.asExternalModel() = NewsArticle(
    id = id,
    name = name,
    imageUrl = imageUrl,
    twitter = twitter,
    mediumPage = mediumPage,
    bio = bio,
)
```

Note: Mappers like the preceding example often map between models defined in different modules. As a result, it's often beneficial to define these mappers in the modules where they are used to avoid tightly coupled modules.

# Expose errors

Interactions with repositories and sources of data can either succeed or throw an exception when a failure occurs. For coroutines and flows, you should use Kotlin's built-in error-handling mechanism. For errors that could be triggered by suspend functions, use try/catch blocks when appropriate; and in flows, use the catch operator. With this approach, the UI layer is expected to handle exceptions when calling the data layer.

The data layer can understand and handle different types of errors and expose them using custom exceptions—for example, a UserNotAuthenticatedException.

Note: Another way to model the result of interactions with the data layer is by using a Result class. This pattern models errors and other signals that can happen as part of processing the result. In this pattern, the data layer returns a Result<T> type instead of T, making the UI aware of known errors that could occur in certain scenarios.


# Testing

Make use of interfaces to allow the creation of fakes for testig. This applied to using interfaces for datasources to enable testing of the repository and the of interfaces on the repository to allow it to be replaced in other tests.