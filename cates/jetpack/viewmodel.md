# ViewModel

## Overview

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel): Stores and manage UI-related data in a lifecycle conscious way. Allows data to **survive configuration changes such as screen rotations**.

A `ViewModel` object **provides the data for a specific UI component (fragment, activity,..), and contains data-handling business logic to communicate with the model**. For example, the `ViewModel` can call other components to load the data, and it can forward user requests to modify the data. `ViewModel` doesn't know about UI components, so it isn't affected by configuration changes (recreating an activity when rotating the device).

If the system destroys or re-creates a UI controller, any transient UI-related data you store in them is lost. For simple data, the `Activity` can use the `onSaveInstanceState()` method and restore its data from the bundle in `onCreate()`, but this approach is only suitable for small amounts of data that can be serialized then deserialized, not for potentially large amounts of data like a list of users or bitmaps.

UI controllers such as activities and fragments are primarily intended to display UI data, react to user actions, or handle OS communication (such as permission requests). Requiring UI controllers to also be responsible for loading data from a database or network **adds bloating code to the class**. Assigning excessive responsibility to UI controllers can result in a **single class that tries to handle all of an app's work by itself, instead of delegating work to other classes, this way also makes testing a lot harder. It's easier and more efficient to separate out view data ownership from UI controller logic.**