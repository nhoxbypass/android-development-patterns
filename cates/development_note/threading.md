# Threading

Making adept use of threads & processes on Android can help you boost your app’s performance.

## Process and Thread

* All are execution environment. Process can consist of 1 or many thread(s) but a thread can only belong to 1 process.
* Context switching: Store the state of process or thread for resume later. Allow multi process to share a single CPU.
* `AsyncTask` will be [execute serially](https://stackoverflow.com/a/18661403/5282585) by default on Android API >= 13. Because of that it will block other `AsyncTask`. To resolve this we use [ThreadpoolExecutor](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html) to execute parallel.

| Process        | Thread          |
| -------------|---------------|
| Heavy weight | Light weight |
| Process creation is costly | Thread creation is economy |
| Not share memory | Memory shared between threads in same process |
| IPC is slow because too many addresses | Communication between threads is fast because they shared mem address |
| Forced to communicate via messages |  |


## Android Threads

#### Runnable

A runnable isn't a background thread, it is a unit of work that can be run in a given thread.

#### Handler

...

#### When to use Handler#post() and when to use Thread#start().

* Use `Handler.post()` whenever you want to do operations in the UI thread. So let's say in the callback (which is running in **separate thread**) you want to change a TextView's text, you should use `Handler.post()`.

* If whatever you are doing is "heavy" you should be doing it in a Thread (That's why network operation is forced to do in a worker thread in Android). Interestingly when you are using a separate **worker thread** it is often useful to also use a `Handler` to communication between this working thread and the main UI thread.

* Must note that: `Handler` and `Thread` are really 2 different things and do NOT contradict each other. `Handler` can be used to attach to a thread and provides a simple channel to send data to this thread. And a `Thread` is basically the core element of multithreading which a developer can use to get heavy payload work off main thread.

* The difference between `Hander.post()` and `View.post()` is that Handler will run your code **on the thread the Handler instance was created on** (which is not necessarily the UI thread) because it will be attached to the only looper of this thread to communicate, while View will **always run it on the UI thread** (because views are bound to it).

#### HandlerThread

You would use `HandlerThread` in case that you want to perform long background tasks **sequentially**, one at a time and you want that those tasks will run at the order of execution. The HandlerThread has it's **own looper** and handlers could be created and post it, (so it would not block the main thread). And use the `Handler` inside it to communication between the worker thread and the caller thread.

For more information related to performance, see [this](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-02) and [this](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-05).

For real life example, look at [this](https://stackoverflow.com/questions/18149964/best-use-of-handlerthread-over-other-similar-classes).


## Ways to get work off main thread.

Every Android app has a main thread which is in charge of handling UI (including measuring and drawing views), coordinating user interactions, and receiving lifecycle events. If there is too much work happening on this thread, the app appears to hang or slow down. Any long-running computations/operations that takes more than a few millisecs (such as decoding a bitmap, accessing the disk, or performing network requests,..) should be done on a separate background thread

There are some ways for your app to perform operations in the background without hurting app performance:

#### Normal Thread

Implement a `Runnable` by overriding and place the code that need to be executed inside `Runnable.run()` method. Then create a new thread using `Thread t = new Thread(Runnable runnable);` and start it with `t.start()`. (Bad practice, thread creation is costly).

#### AsyncTask

[AsyncTask](https://developer.android.com/reference/android/os/AsyncTask) is a proper and easy use to perform background operations and publish results on the UI thread without having to manipulate threads and/or handlers.

Create a new class that extends `AsyncTask`, override `doInBackground(Params...)` and implement the code that need to get off main thread. Then if you want to use the result to update UI, override `onPostExecute(Result)`. Init and invoke `execute()` to start this task. 
   
See this [example](https://stackoverflow.com/questions/9671546/asynctask-android-example). (Quick solution but known source of memory leaks).

#### ThreadPools

Using `ThreadPools` - which providing a group of background threads that accept and enqueue submitted work -  with `Executor`. Create new group of ThreadPools base on your need (network, disk I/O, and computation,..) using `Executor executor = Executors.newSingleThreadExecutor()` and execute task using `executor.execute(new Runnable() {...})`. (Recommended - reuse threads to **avoid thread creation cost**)

#### Worker Thread

Create a custom [worker thread](https://stackoverflow.com/questions/13235312/what-are-worker-threads-and-what-is-their-role-in-the-reactor-pattern).

#### HandlerThread

Create a new `HandlerThread` and start the same as normal Thread. And use Handler to communicate. See [this](https://stackoverflow.com/questions/25094330/example-communicating-with-handlerthread).

#### IntentService

Create a new class that extends `IntentService` and implement it. Then trigger start using an Intent to pawns a new worker thread. See [this](https://code.tutsplus.com/tutorials/android-fundamentals-intentservice-basics--mobile-6183).

See [this](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-01) to see when to use which approach.

#### Using external libraries:  

1. [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager) is part of Android Jetpack, a lib that gracefully runs deferrable background work when the work's triggers (like appropriate network state and battery conditions) are satisfied. Recommended for work that must execute to completion and is deferrable.
2. [RxJava](https://github.com/ReactiveX/RxJava) Reactive Extensions for the JVM – a library for composing asynchronous and event-based programs using observable sequences for the Java VM.


## Race condition

A race condition occurs when two or more threads can access shared data and they try to change it at the same time. The shared data part is called "critical section".
To resolve conflict, race condition Java sync is NOT enough -> blocking thread.

* `vilotile`: Ensure anythread that read a field will see the most recently value.

| synchronize        | lock          |
| -------------|---------------|
| Java use monitor lock | Need help of system |
| Easy to use | Need to try/catch exception to **release** the lock |
