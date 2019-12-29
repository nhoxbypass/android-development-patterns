# Guide to app architecture

This guide encompasses best practices and recommended architecture for building robust, production-quality apps.


## Mobile app architecture

In the majority of cases, desktop apps have a single entry point from a desktop or program launcher, then run as a single, monolithic process. Android apps, on the other hand, have a much more complex structure.

You declare most of these app components in app manifest. The Android OS then uses this file to decide how to integrate your app into the device's overall UX.

Keep in mind that mobile devices are also **resource-constrained**, so at any time, the operating system (OS) might kill some app processes to make room for new ones. Because these events aren't under your control, you **shouldn't store any app data or state in your app components, and your app components shouldn't depend on each other**.


## Common architectural principles

If you shouldn't use app components to store app data and state, how should you design your app?

#### Separation of concerns

The most important principle to follow is [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns). It's a common **mistake** to write all your code in an `Activity` or a `Fragment`. **These UI-based classes should only contain logic that handles UI and device interactions**. By keeping these classes as lean as possible, you can avoid many lifecycle-related problems.

Keep in mind that you don't own implementations of `Activity` and `Fragment`, rather, these are just glue classes that represent the contract between the Android OS and your app. The OS can destroy them at any time based on user interactions or because of system conditions like low memory. To provide a satisfactory UX and a more manageable app maintenance experience, it's best to **minimize your dependency on them**.


#### Update UI base on Model

You should **change your UI base on state of a model**, preferably a persistent model. Models are components that are responsible for handling the data for an app. They're independent from the `View` and app components, so they're **unaffected by the app's lifecycle and the associated concerns**.

Persistence is ideal for the following reasons:
* Your users don't lose data if the Android OS destroys your app to free up resources.
* Your app continues to work in cases when a network connection is flaky or not available. User can still browse that content while they are offline, any user-initiated content changes are then synced to the server after the device is back online

By basing your app on model classes with the well-defined responsibility of managing the data, your app is more testable and consistent.


## Using data repository

Repository modules handle data operations. The Repository pattern **adds an abstraction layer over the top of data access, to provide a clean & centralising API so that the rest of the app can retrieve this data easily**. They know where to get the data from and what API calls to make when data is updated.

![Repository Pattern](/resources/repository_pattern.png)

**Even though the repository module looks unnecessary, it serves an important purpose: it abstracts the data sources from the rest of the app**. 

You can consider repositories to be mediators between different data sources:
* Remote: RestAPI or cloud services (Firebase, AWS,..).
* Local: database or file.
* In-memory cache.

It’s a good idea to have a data layer in your app, completely unaware of your presentation layer. Algorithms to keep cache and database in sync with the network are not trivial. Having a separate repository class as the single-point entry to your data.


#### Cache data in memory

The repository implementation abstracts the call to the "API helper" object, but because it relies on only one data source, it's not very flexible. After it fetches data from our API server, it doesn't store that data anywhere. Therefore, if the user home app, then returns, our app must re-fetch the data from server, even if it hasn't changed. This will wastes valuable network bandwidth and forces user to wait for the new query to complete.

So we need to add a new data source in our Reposistory to cache these value, or just use a `Map<>` to cache it. When the UI needs data, we check the cache first, it it's not exist --> query to server to fetch data.


#### Persist data in database

After added memory cache, if the user rotates the device or leaves and immediately returns to the app, the existing UI becomes visible instantly because the repository retrieves data from our in-memory cache.

However, what happens if the user leaves the app and comes back hours later, after the Android OS has killed the process? We need to fetch the data again from the network. This re-fetching process isn't just a bad UX; it's also wasteful because it consumes valuable mobile data.

You could fix this by caching the web requests, but that create new problem: What happens if the same user data shows up from another type of request, such as fetching a list of friends? The app would show inconsistent data. Our app would need to figure out how to merge this inconsistent data.

The proper way to handle this situation is to use a persistent model. This is where the [Room](https://github.com/nhoxbypass/android-development-patterns/blob/master/cates/jetpack/architecture_component.md#room) persistence library comes to the rescue.


#### Single source of truth

Instead of directly access data from model and push to the UI, **we let them observe data in the Repository and then react to data changed events later**.

![Single source of truth](/resources/single_source_of_truth.jpeg)

**Whenever Repository saves server API responses into the database, these changes then trigger callbacks on active "observer" objects**. Using this model, the database serves as the single source of truth, and other parts of the app access it using our Repository.


## Recommended app architecture

#### Overview

To start, consider the following diagram, which shows how all the modules should interact with one another after designing the app:

![App architecture overview](/resources/final-architecture.png)

Notice that **each component depends only on the component one level below it**. (Ex: activities and fragments depend only on a view model). **The Repository is the only class can depends on multiple other classes**. (repository can depends on persistent data model and remote backend data source,..).


#### Distributing responsibilities

![Typical interaction of entities in an app built with Architecture Components](/resources/viewmodel_distributing_res.png)

Ideally, **ViewModels contain logic-only and shouldn’t know anything about Android**. This improves testability, leak safety and modularity. A general rule of thumb is to make sure there are no `android.*` imports in your `ViewModel`s (with exceptions like `android.arch.*`). 

Keep the logic in `Activity` and `Fragment` to a minimum. Conditional statements, loops,.. should be done in `ViewModel`s or other layers of an app. The View layer is usually **not unit tested** (unless you use `Robolectric`) so the fewer lines of code the better. Views should only know how to display data and send user events to the `ViewModel`. This is called the [Passive View pattern](https://martinfowler.com/eaaDev/PassiveScreen.html).


#### Observer pattern with LiveData

![ViewModel Observer Pattern](/resources/viewmodel_observer.png)

**Instead of actively pushing data to the UI, let the UI observe changes to it**.

View (activity or fragment) should observe (subscribe to changes in) the `ViewModel`. Since the `ViewModel`doesn’t know about Android, it doesn’t know how Android likes to kill Views frequently. This has some advantages:

* `ViewModel`s are persisted over configuration changes, so there’s no need to re-query an external source for data (such as a database or the network) when a screen-rotation happens.
* When long-running operations finish, the observables in the `ViewModel` are updated. It doesn’t matter if the data is being observed or not. No NPEs happen when trying to update the nonexistent `View`.
* `ViewModel`s don’t reference views so there’s less risk of memory leaks.

```
private void subscribeToModel() {
  // Observe product data
  viewModel.getObservableProduct().observe(this, new Observer<Product>() {
      @Override
      public void onChanged(@Nullable Product product) {
        tvTitle.setText(product.title);
      }
  });
}
```


#### View references in ViewModels

Avoid references to View-layer in `ViewModel`s.

**ViewModels have different scopes than activities or fragments**. While a `ViewModel` is alive and running, an activity can be in any of its [lifecycle states](https://developer.android.com/guide/components/activities/activity-lifecycle.html). Activities and fragments can be destroyed or re-created while the `ViewModel` is unaware.

![ViewModels persist configuration changes](/resources/viewmodel_scope.png)

Passing a reference of the View (activity or fragment) to the `ViewModel` is a serious risk. Let’s assume the `ViewModel` requests data from the network and the data comes back some time later. At that moment, **the View reference might be destroyed or might be an old activity that is no longer visible, generating a memory leak** and, possibly, a crash.

The recommended way to communicate between `ViewModel`s and View-layer is the observer pattern (said above), using `LiveData` or observables from other libraries (`RxJava`,..).


#### Leaking ViewModels

Consider edge cases, mem leaks and how long-running operations can affect the instances in your architecture.

The reactive paradigm works well in Android because it allows for a convenient connection between UI and the rest of the layers of your app. `LiveData` is the key component of this structure so normally your activities and fragments will observe `LiveData` instances.

Consider this diagram where the Presentation-layer is using the observer pattern and the Data-Layer is using callbacks:

![Observer pattern in the UI and callbacks in the data layer](/resources/leaking_viewmodel.png)

If the user exits the app, the View-layer will be gone so the `ViewModel` is not observed anymore. **If the repository is a singleton or otherwise scoped to the application, the repository will not be destroyed until the process is killed**. (only when system needs resources or the user force stop the app). Now if the **repository is holding a reference to a callback in the ViewModel, the ViewModel will be temporarily leaked**.

![The activity is finished but the ViewModel is still around](/resources/leaking_viewmodel_1.png)

This leak is not a big deal if the `ViewModel` is light or the operation is guaranteed to finish quickly. However, this is not always the case. Ideally, **ViewModels should be free to go whenever they don’t have any Views observing them**:

![The activity is finished and the ViewModel is cleared](/resources/leaking_viewmodel_2.png)

You have many options to achieve this:

* With `ViewModel.onCleared()` you can tell the repository to drop the callback to the `ViewModel`.
* In the repository you can use a `WeakReference` or you can use an `Event Bus` (both easy to misuse and even considered harmful).
* Use the `LiveData` in the Repository to communicate with `ViewModel` (see section below).

Don’t put logic in the ViewModel that is critical to saving clean state or related to data. Because any call you make from a `ViewModel` can be the last one.


#### LiveData in repository

To avoid leaking `ViewModel`s and callback hell, repositories can be observed. We can use the `LiveData` to communicate between the Repository and `ViewModel` in a similar way to using `LiveData` between the View and the `ViewModel`.


![LiveData in repository](/resources/livedata_repository.png)

When the `ViewModel` is cleared or when the lifecycle of the view is finished, the subscription is cleared:

![LiveData in repository](/resources/livedata_repository_1.png)


#### Manage dependencies between components

You can use the following design patterns to address this problem:

* Dependency injection (DI): Dependency injection allows classes to define their dependencies without constructing them. At runtime, another class is responsible for providing these dependencies. We recommend the `Dagger 2` library for implementing dependency injection in Android apps. `Dagger 2` automatically constructs objects by walking the dependency tree, and it provides compile-time guarantees on dependencies.
* Service locator: The service locator pattern provides a registry where classes can obtain their dependencies instead of constructing them.

It's easier to implement a service registry than use DI, so if you aren't familiar with DI, use the service locator pattern instead.

These patterns allow you to scale your code because they provide clear patterns for managing dependencies without duplicating code or adding complexity. Furthermore, these patterns allow you to quickly switch between test and production data-fetching implementations.


## Best practices

Programming is a creative field, and building Android apps isn't an exception. There are many ways to solve a problem, be it communicating data between multiple activities or fragments, retrieving remote data and persisting it locally for offline mode, or any number of other common scenarios that nontrivial apps encounter.

Although the following recommendations aren't mandatory, it has been our experience that following them makes your code base more robust, testable, and maintainable in the long run:

**Avoid designating your app's entry points (activities, services, and broadcast receivers,..) as sources of data.**

Instead, they should only coordinate with other components to retrieve the subset of data that is relevant to that entry point. Each app component is rather **short-lived**, depending on the user's interaction with their device and the overall current health of the system.

**Create well-defined boundaries of responsibility between various modules of your app.**

For example, don't spread the code that loads data from the network across multiple classes or packages in your code base. Similarly, don't define multiple unrelated responsibilities—such as data caching and data binding—into the same class.

**Expose as little as possible from each module.**

Don't be tempted to create "just that one" shortcut that exposes an internal implementation detail from one module. You might gain a bit of time in the short term, but you then incur technical debt many times over as your codebase evolves.

**Consider how to make each module testable in isolation.**

For example, having a well-defined API for fetching data from the network makes it easier to test the module that persists that data in a local database. If, instead, you mix the logic from these two modules in one place, or distribute your networking code across your entire code base, it becomes much more difficult (or even impossible) to test.

**Focus on the unique core of your app so it stands out from other apps.**

Don't reinvent the wheel by writing the same boilerplate code again and again. Instead, focus your time and energy on the features that makes your app unique, and let the Android Architecture Components and other recommended libraries handle the repetitive boilerplate.

**Persist as much relevant and fresh data as possible.**

That way, users can enjoy your app's functionality even when their device is in offline mode. Remember that not all of your users enjoy constant, high-speed connectivity.

**Assign one data source to be the single source of truth.**

Whenever your app needs to access this piece of data, it should always originate from a single source of truth - the databases.
