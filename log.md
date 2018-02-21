# Android Performance Notes

### Threading Performance 101

***Ep 01***

Android redraw the screen every `16.67 ms` to keep the app smoothly at `60 FPS`. If your LameWork is longer, it will cause **dropped frame**. So you must get all of your heavy workload **off** UI (main) thread and communicate back when all of the work are done. Android provide many ways to do so:

* [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html): Helps get workload on/off main thread the right way.
* [HandlerThread](https://developer.android.com/reference/android/os/HandlerThread.html): Dedicated thread for API callbacks to land on.
* [ThreadPool](https://developer.android.com/training/multiple-threads/run-code.html): Break your workload up into a really small package, and toss them to a bunch of threads.
* [IntentService](https://developer.android.com/reference/android/app/IntentService.html): Helps get intents work off the UI thread.

Another thing need to care is memory. Threading and memory never really played well together. It can cause **memory leaks** (inner class AsyncTask, Activity get destroyed when many threads are still running,..)

***Ep 02***

* Since thread die when run out of work, so you need to always have some sort of loop running on the thread.
  
  Eg: [Looper aka Event Loop](https://developer.android.com/reference/android/os/Looper.html) which keep the thread alive and pop task/message off the waiting queue to execute on,..
* In addition, you need some sort of waiting queue that the loop can pull blocks of work from to execute on, and some other threads that create work packages and push them into the queue for execution.
  
  Eg: [Handler](https://developer.android.com/reference/android/os/Handler.html) receive task/message from other thread and add to the waiting queue, which give us control to push work at the head/tail or set a time-based delay for blocks of work,..
* And the unit/block of work in Android are explicitly defined as Intents, Runables or Messages,.. depend on who's issuing them and who's consuming them.

And the combination of all these things together is a HandlerThread.

When an app run, the system create for it a process which contain a thread of execution called main thread or UI thread, which is just a HandlerThread.

**References:**
1. [Threading Performance 101. (Android Performance Patterns Season 5, Ep. 1)](https://www.youtube.com/watch?v=qk5F6Bxqhr4).
2. [Understanding Android Threading. (Android Performance Patterns Season 5, Ep. 2)](https://www.youtube.com/watch?v=0Z5MZ0jL2BM)