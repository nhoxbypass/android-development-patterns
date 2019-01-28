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

1. Create new `Thread`: `Thread t = new Thread(new Runnable() {...});` then start it `t.start()`. (Bad practice)
2. Create a new [class](https://stackoverflow.com/questions/9671546/asynctask-android-example) that extends `AsyncTask`, and implement the code that need work off main thread inside `doOnBackground()` callback. Then write code to update result, UI inside `onPostExecute()` callback. Finally init and `execute()` this task. (Known source of memory leaks).
3. Create a custom [worker thread](https://stackoverflow.com/questions/13235312/what-are-worker-threads-and-what-is-their-role-in-the-reactor-pattern) using `Executor` `Executor executor = Executors.newSingleThreadExecutor()` and execute task using `executor.execute(new Runnable() {...})`. (Recommended - reuse threads to avoid thread creation cost)
4. Create a new `HandlerThread` and start the same as normal Thread. And use Handler to communicate. See [this](https://stackoverflow.com/questions/25094330/example-communicating-with-handlerthread).
5. Create a new class that extends `IntentService` and implement it. Then trigger start using an Intent to pawns a new worker thread. See [this](https://code.tutsplus.com/tutorials/android-fundamentals-intentservice-basics--mobile-6183).

See [this](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-01) to see when to use which approach.


## Race condition

A race condition occurs when two or more threads can access shared data and they try to change it at the same time. The shared data part is called "critical section".
To resolve conflict, race condition Java sync is NOT enough -> blocking thread.

* `vilotile`: Ensure anythread that read a field will see the most recently value.

| synchronize        | lock          |
| -------------|---------------|
| Java use monitor lock | Need help of system |
| Easy to use | Need to try/catch exception to **release** the lock |
