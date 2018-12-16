# Loader

The Loader API lets you load data from a content provider or other data source for display in an FragmentActivity or Fragment. It was a solution to a problem that we shouldn’t have really been trying to solve — running asynchronous tasks within a Fragment or Activity and handling the rotation change.

* What do we do with the threaded work when the activity that kicked it off is no longer alive? See [this](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-08).

* Help **loading data in worker thread** to avoid ANR due to performing potentially slow queries.

* Compare to `AsyncTask` (which also load data in worker thread) it help **simplying management of both the thread and the UI thread** (through various activity or fragment lifecycle events, such as `onDestroy()` and configurations changes) by providing callback methods when events occur.

* Loaders **survive across configuration changes** so it persist and cache results to prevent duplicate queries.

* Implement an observer to **listen for changes in the underlying data source**. Eg: `CursorLoader` automatically registers a `ContentObserver` to trigger a reload when data changes

* Loaders **don’t stay around forever**. They’ll be automatically cleaned up when the requesting Activity or Fragment is permanently destroyed. That means no lingering, unnecessary loads.


## Working with the rest of your app: the `LoaderManager`

Of course, even the best loader would be nothing if it wasn’t connected to something. That connection point for activities and fragments comes in the form of `LoaderManager`. You’ll call `FragmentActivity`’s `getSupportLoaderManager()` or a `Fragment`’s `getLoaderManager()` to get your instance.

In almost every case, there’s only one method you’ll need to call: `initLoader()`. This is generally called in `onCreate()` or `onActivityCreated()` — basically as soon as you know you’ll need to load some data. There’s a `restartLoader()` method which gives you the ability to force a reload but un-necessary in most cases.


## Loaders: for data only

If you’ve been using retained Fragments (those that call `setRetainInstance(true)`) to store data across configuration changes, strongly consider switching from a retained Fragment to a Loader. Retained fragments, while aware of the overall Activity lifecycle, should be viewed as completely independent entities, while Loaders are tied directly into an Activity or Fragment lifecycle (even child fragments!) and therefore much more appropriate for retrieving exactly the data needed for display.


## Loaders are reactive, recipients of data

They’re not responsible for changing the underlying data. 


## Note

Loaders have been **deprecated** as of Android P (API 28). The recommended option for dealing with loading data while handling the Activity and Fragment lifecycles is to use a combination of `ViewModels` and `LiveData`. 

ViewModels survive configuration changes like Loaders but with less boilerplate. LiveData provides a lifecycle-aware way of loading data that you can reuse in multiple ViewModels.

