# Android Performance Notes

### Threading Performance 101

***Ep 01***

Android redraw the screen every `16.67 ms` to keep the app smoothly at `60 FPS`. If your LameWork is longer, it will cause **dropped frame**. So you must get all of your heavy workload **off** UI (main) thread and communicate back when all of the work are done. Android provide many ways to do so:

* [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html): Helps get workload on/off main thread the right way.
* [HandlerThread](https://developer.android.com/reference/android/os/HandlerThread.html): Dedicated thread for API callbacks to land on.
* [ThreadPool](https://developer.android.com/training/multiple-threads/run-code.html): Break your workload up into a really small package, and toss them to a bunch of threads.
* [IntentService](https://developer.android.com/reference/android/app/IntentService.html): Helps get intents work off the UI thread.

Another thing need to care is memory. Threading and memory never really played well together. It can cause **memory leaks** (inner class `AsyncTask`, `Activity` get destroyed when many threads are still running,..)

***Ep 02***

* Since thread will **die** when run out of work, so you need to always have some sort of **loop running** on the thread.
  
  Eg: [Looper aka Event Loop](https://developer.android.com/reference/android/os/Looper.html) which keep the thread alive and pop task/message off the waiting queue to execute on,..

* In addition, you need some sort of **waiting queue** that the loop can **pull blocks of work** from to execute on, and some other **threads that create work** packages and **push them into the queue** for execution.
  
  Eg: [Handler](https://developer.android.com/reference/android/os/Handler.html) receive task/message from other thread and add to the waiting queue, which give us control to push work at the head/tail or set a time-based delay for blocks of work,..

* And the unit/block of work in Android are explicitly defined as `Intent`s, `Runable`s or `Message`s,.. vary depend on who's issuing them and who's consuming them.

And the combination of all these things together is a `HandlerThread`.

When an app run, the system create for it a process which contain a thread of execution called main thread or UI thread, which is just a `HandlerThread`.

***Ep 03***

* View can be reference from worker thread to update UI after execute some jobs, but this view can be **removed** from the view hierachy **before the jobs done**.

* If an activity is **destroyed** (by calling `finish()` or user rotate the screen or being killed by system,..) but there still exists a threaded block of work that references it, the Activity will **NOT get collected until that block of work finishes**.
  Eg: If you kick off some work and the user rotates the screen three times in-a-row before that work completes, you could end up with 3 instances of that Activity object resident in memory!
  It's NOT just the explicit references to UI objects that you need to worry about but also the implicit references (non-static nested AsyncTask,..).

* Should NOT hold references to any type of UI objects in any of threading scenarios. But how? Use a unique **update function (WorkRecords)** to update new information for the views or drop the work if the view is NOT there anymore.

***Ep 04***

* All `AsyncTask` created will **share the same thread** and thus will [execute in a serial fashion](https://stackoverflow.com/questions/18661288/android-two-asynctasks-serially-or-parallel-execution-the-second-is-freezing) from a **single message queue**. So if the `AsyncTask` take too long to complete it will **freeze the thread** from doing future work (Unless you use `Executor` with thread pool).
  Eg: If you kick off 20 work orders and the 3rd take an hour -> the other 17 will be blocked and wait!

* To "cancel" some work: First create and check a "isCancelled" flag. Then report work result invalid.

* Non-static nested or **inner AsyncTask** will create an **implicit references** to the outer enclosing class. So Activity and entire view hierachy that use inner AsyncTask will be **leaked** if it get **destroyed before the AsyncTask work completed**, GC will NOT mark/swipe the destroyed Activity instance because there is still an implicit reference from AsyncTask to it, and AsyncTask is still running.

**References:**
1. [Threading Performance 101](https://www.youtube.com/watch?v=qk5F6Bxqhr4).
2. [Understanding Android Threading](https://www.youtube.com/watch?v=0Z5MZ0jL2BM)
3. [Memory & Threading](https://www.youtube.com/watch?v=tBHPmQQNiS8)
4. [Good AsyncTask Hunting](https://www.youtube.com/watch?v=jtlRNNhane0)