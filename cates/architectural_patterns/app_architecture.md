# Guide to app architecture


## Mobile app user experiences

In the majority of cases, desktop apps have a single entry point from a desktop or program launcher, then run as a single, monolithic process. Android apps, on the other hand, have a much more complex structure.

You declare most of these app components in your app manifest. The Android OS then uses this file to decide how to integrate your app into the device's overall user experience.

Keep in mind that mobile devices are also resource-constrained, so at any time, the operating system might kill some app processes to make room for new ones.

Given the conditions of this environment, it's possible for your app components to be launched individually and out-of-order, and the operating system or user can destroy them at any time. Because these events aren't under your control, you shouldn't store any app data or state in your app components, and your app components shouldn't depend on each other.


## Common architectural principles

If you shouldn't use app components to store app data and state, how should you design your app?

#### Separation of concerns

The most important principle to follow is [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns). It's a common mistake to write all your code in an `Activity` or a `Fragment`. These UI-based classes **should only contain logic that handles UI and operating system interactions**. By keeping these classes as lean as possible, you can avoid many lifecycle-related problems.

Keep in mind that you don't own implementations of `Activity` and `Fragment`, rather, these are just glue classes that represent the contract between the Android OS and your app. The OS can destroy them at any time based on user interactions or because of system conditions like low memory. To provide a satisfactory user experience and a more manageable app maintenance experience, it's best to **minimize your dependency on them**.


#### Drive UI from a model

Another important principle is that you should **drive your UI from a model**, preferably a persistent model. Models are components that are responsible for handling the data for an app. They're independent from the `View` and app components in your app, so they're unaffected by the app's lifecycle and the associated concerns.

Persistence is ideal for the following reasons:
* Your users don't lose data if the Android OS destroys your app to free up resources.
* Your app continues to work in cases when a network connection is flaky or not available.

By basing your app on model classes with the well-defined responsibility of managing the data, your app is more testable and consistent.


## Recommended app architecture

#### Overview

To start, consider the following diagram, which shows how all the modules should interact with one another after designing the app:

![App architecture overview](/resources/final-architecture.png)

Notice that **each component depends only on the component one level below it**. Ex: activities and fragments depend only on a view model. **The repository is the only class that depends on multiple other classes**. (the repository depends on persistent data model and remote backend data source,..)

**Repository**

Repository modules handle data operations. They provide a clean API so that the rest of the app can retrieve this data easily. They know where to get the data from and what API calls to make when data is updated. You can consider repositories to be mediators between different data sources, such as persistent models, web services, and caches.


#### Android Architecture Components   

Android architecture components are a collection of libraries that help you design robust, testable, and maintainable apps. Start with classes for managing your UI component lifecycle and handling data persistence.

Manage your app's lifecycle with ease. New lifecycle-aware components help you manage your activity and fragment lifecycles. Survive configuration changes, avoid memory leaks and easily load data into your UI.

**ViewModel**

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel): Stores UI-related data that isn't destroyed on app rotations.

A ViewModel object provides the data for a specific UI component, such as a fragment or activity, and contains data-handling business logic to communicate with the model. For example, the ViewModel can call other components to load the data, and it can forward user requests to modify the data. The ViewModel doesn't know about UI components, so it isn't affected by configuration changes, such as recreating an activity when rotating the device.

**LiveData**

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata): Use to build data objects that notify views when the underlying database changes.

LiveData is an observable data holder. Other components in your app can monitor changes to objects using this holder without creating explicit and rigid dependency paths between them. The LiveData component also respects the lifecycle state of your app's components—such as activities, fragments, and services—and includes cleanup logic to prevent object leaking and excessive memory consumption.

**Room**

[Room](https://developer.android.com/topic/libraries/architecture/room) An a SQLite object mapping library. Use it to Avoid boilerplate code and easily convert SQLite table data to Java objects. Room provides compile time checks of SQLite statements and can return RxJava, Flowable and LiveData observables.

Room is an object-mapping library that provides local data persistence with minimal boilerplate code. At compile time, it validates each query against your data schema, so broken SQL queries result in compile-time errors instead of runtime failures. Room abstracts away some of the underlying implementation details of working with raw SQL tables and queries. It also allows you to observe changes to the database's data, including collections and join queries, exposing such changes using LiveData objects. It even explicitly defines execution constraints that address common threading issues, such as accessing storage on the main thread.


#### Manage dependencies between components

You can use the following design patterns to address this problem:

* Dependency injection (DI): Dependency injection allows classes to define their dependencies without constructing them. At runtime, another class is responsible for providing these dependencies. We recommend the Dagger 2 library for implementing dependency injection in Android apps. Dagger 2 automatically constructs objects by walking the dependency tree, and it provides compile-time guarantees on dependencies.
* Service locator: The service locator pattern provides a registry where classes can obtain their dependencies instead of constructing them.

It's easier to implement a service registry than use DI, so if you aren't familiar with DI, use the service locator pattern instead.

These patterns allow you to scale your code because they provide clear patterns for managing dependencies without duplicating code or adding complexity. Furthermore, these patterns allow you to quickly switch between test and production data-fetching implementations.


#### Single source of truth

It's common for different REST API endpoints to return the same data. For example, if our backend has another endpoint that returns a list of friends, the same user object could come from two different API endpoints, maybe even using different levels of granularity. If the UserRepository were to return the response from the Webservice request as-is, without checking for consistency, our UIs could show confusing information because the version and format of data from the repository would depend on the endpoint most recently called.

For this reason, our UserRepository implementation saves web service responses into the database. Changes to the database then trigger callbacks on active LiveData objects. Using this model, **the database serves as the single source of truth**, and other parts of the app access it using our UserRepository. Regardless of whether you use a disk cache, we recommend that your repository designate a data source as the single source of truth for the rest of your app.


## Best practices

Programming is a creative field, and building Android apps isn't an exception. There are many ways to solve a problem, be it communicating data between multiple activities or fragments, retrieving remote data and persisting it locally for offline mode, or any number of other common scenarios that nontrivial apps encounter.

Although the following recommendations aren't mandatory, it has been our experience that following them makes your code base more robust, testable, and maintainable in the long run:

**Avoid designating your app's entry points—such as activities, services, and broadcast receivers—as sources of data.**

Instead, they should only coordinate with other components to retrieve the subset of data that is relevant to that entry point. Each app component is rather short-lived, depending on the user's interaction with their device and the overall current health of the system.

**Create well-defined boundaries of responsibility between various modules of your app.**

For example, don't spread the code that loads data from the network across multiple classes or packages in your code base. Similarly, don't define multiple unrelated responsibilities—such as data caching and data binding—into the same class.

**Expose as little as possible from each module.**

Don't be tempted to create "just that one" shortcut that exposes an internal implementation detail from one module. You might gain a bit of time in the short term, but you then incur technical debt many times over as your codebase evolves.

**Consider how to make each module testable in isolation.**

For example, having a well-defined API for fetching data from the network makes it easier to test the module that persists that data in a local database. If, instead, you mix the logic from these two modules in one place, or distribute your networking code across your entire code base, it becomes much more difficult—if not impossible—to test.

**Focus on the unique core of your app so it stands out from other apps.**

Don't reinvent the wheel by writing the same boilerplate code again and again. Instead, focus your time and energy on what makes your app unique, and let the Android Architecture Components and other recommended libraries handle the repetitive boilerplate.

**Persist as much relevant and fresh data as possible.**

That way, users can enjoy your app's functionality even when their device is in offline mode. Remember that not all of your users enjoy constant, high-speed connectivity.

**Assign one data source to be the single source of truth.**

Whenever your app needs to access this piece of data, it should always originate from a single source of truth - the databases.