# Android Architecture Components   

Android architecture components are a collection of libraries that help you design robust, testable, and maintainable apps. Start with classes for managing your UI component lifecycle and handling data persistence.

Manage your app's lifecycle with ease. New lifecycle-aware components help you manage your `Activity` and `Fragment` lifecycles. Survive configuration changes, avoid memory leaks and easily load data into your UI.

## ViewModel

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel): Stores and manage UI-related data in a lifecycle conscious way. Allows data to **survive configuration changes such as screen rotations**.

A `ViewModel` object **provides the data for a specific UI component (fragment, activity,..), and contains data-handling business logic to communicate with the model**. For example, the `ViewModel` can call other components to load the data, and it can forward user requests to modify the data. `ViewModel` doesn't know about UI components, so it isn't affected by configuration changes (recreating an activity when rotating the device).

If the system destroys or re-creates a UI controller, any transient UI-related data you store in them is lost. For simple data, the `Activity` can use the `onSaveInstanceState()` method and restore its data from the bundle in `onCreate()`, but this approach is only suitable for small amounts of data that can be serialized then deserialized, not for potentially large amounts of data like a list of users or bitmaps.

UI controllers such as activities and fragments are primarily intended to display UI data, react to user actions, or handle OS communication (such as permission requests). Requiring UI controllers to also be responsible for loading data from a database or network **adds bloating code to the class**. Assigning excessive responsibility to UI controllers can result in a **single class that tries to handle all of an app's work by itself, instead of delegating work to other classes, this way also makes testing a lot harder. It's easier and more efficient to separate out view data ownership from UI controller logic.**


## LiveData

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata): An observable data holder to build data objects that **notify views when the underlying database changes**.

`LiveData` is an observable data holder class. Unlike a regular observable, **LiveData is lifecycle-aware**, meaning it respects the lifecycle of other app components, such as activities, fragments, or services. This awareness **ensures LiveData only updates app component observers that are in an active lifecycle state**. 

Other components in your app can **monitor changes to objects** using this holder without creating explicit and rigid dependency paths between them. The LiveData component also respects the lifecycle state of your app's components—such as activities, fragments, and services—and includes **cleanup logic to prevent object leaking and excessive memory consumption**.

## Room

[Room](https://developer.android.com/topic/libraries/architecture/room) An a SQLite object mapping library. Use it to easily convert SQLite table data to Java objects and avoid boilerplate code.

`Room` is an **object-mapping library that provides local data persistence with minimal boilerplate code by abstracting away some of the underlying implementation details of working with raw SQLite tables and queries**. It also allows you to observe changes to the database's data, including collections and join queries, exposing such changes using `LiveData` objects. It even **explicitly defines execution constraints that address common threading issues, such as accessing storage on the main thread**.

**At compile time, it validates each query** against your data schema, so broken SQL queries result in compile-time errors instead of runtime failures.

`Room` can return `RxJava`, `Flowable` and `LiveData` observables.

If your app already uses another persistence solution, such as a SQLite object-relational mapping (ORM), you don't need to replace your existing solution with `Room`. However, if you're writing a new app or refactoring an existing app - because `Room` takes care of these concerns for you - we highly recommend using `Room` to persist your app's data.