# Android Performance Notes

## Threading Performance 101


#### Ep 01

Android redraw the screen every `16.67 ms` to keep the app smoothly at `60 FPS`. If your LameWork is longer, it will cause **dropped frame**. So you must get all of your heavy workload **off** UI (main) thread and communicate back when all of the work are done. Android provide many ways to do so:

* [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html): Helps get workload on/off main thread the right way. Usually use for simple network operations which do NOT require downloading a lot of data or disk-bound tasks that might take more than a few milliseconds.
* [HandlerThread](https://developer.android.com/reference/android/os/HandlerThread.html): Dedicated thread for API callbacks to land on. Use for long tasks that requires sequentially communication between the worker thread and the caller thread
* [ThreadPool](https://developer.android.com/training/multiple-threads/run-code.html): Break your workload up into a really small package, and toss them to a bunch of threads. To exploit the powerful multi-core processor and run the tasks concurrently on more than one worker threads.
* [IntentService](https://developer.android.com/reference/android/app/IntentService.html): Helps get intents work off the UI thread. Use for long scheduled tasks that should run on the background, independent of your activity.

Another thing need to care is memory. Threading and memory never really played well together. It can cause **memory leaks** (using inner class `AsyncTask` then activity get destroyed but `AsyncTask`s & threads are still running and have have reference to this activity,..)


#### Ep 02

* Since thread will **die** when run out of work, so you need to ALWAYS have some sort of **loop running** on the thread.
  
  Eg: [Looper aka Event Loop](https://developer.android.com/reference/android/os/Looper.html) which keep the thread alive and pop tasks/messages off the waiting queue to execute on,..

* In addition, you need some sort of **waiting queue** that the loop can **pull blocks of work** out for execution, and some other **threads that create work** packages and **push them into the queue** for execution.
  
  Eg: [Handler](https://developer.android.com/reference/android/os/Handler.html) receive tasks/messages from other threads and add to the waiting queue, which give us control to push work at the head/tail or set a time-based delay for blocks of work,..

* And the unit/block of work in Android are explicitly defined as `Intent`s, `Runable`s or `Message`s,.. vary depend on who's issuing them and who's consuming them.

And the combination of all these things together is a `HandlerThread`.
When an app run, the system create for it a process which contain a thread of execution called main thread or UI thread, which is just a `HandlerThread`.


#### Ep 03

* View can be reference from worker thread to update UI after execute jobs, but this view can be **removed** from the view hierachy **before the jobs done**.

* If an activity is **destroyed** (by calling `finish()` or user rotate the screen or being killed by system,..) but there still exists a threaded block of work that references it, the Activity and entire view-tree will **NOT get collected by GC until that block of work finishes**.
  Eg: If you kick off some work and the user rotates the screen three times in-a-row before that work completes, you could end up with 3 instances of that Activity resident in memory!
  It's NOT just the explicit references to UI objects that you need to worry about but also the implicit references (non-static nested (inner) classes,..).

* Should NOT hold references to any type of UI objects in any of threading scenarios. But how? Use a unique **update function** to update new information for the views **or drop** the work if the view is NOT there anymore.


#### Ep 04

* All `AsyncTask` created will **share the same thread** and thus will [execute in a serial fashion](https://stackoverflow.com/questions/18661288/android-two-asynctasks-serially-or-parallel-execution-the-second-is-freezing) from a **single message queue**. So if the `AsyncTask` take too long to complete it will **freeze the thread** from doing future work (Unless you use `Executor` with thread pool).
  Eg: If you kick off 20 work orders and the 3rd take an hour -> the other 17 will be blocked and wait!

* To "cancel" some work: First create and check a `isCancelled` flag. Then report work result or drop.

* Non-static nested or **inner class** will create an **implicit references** to the outer enclosing class. So Activity and entire view hierachy that use inner `AsyncTask` (most famous, well-known case) will be **leaked** if it get **destroyed before the AsyncTask work completed**, GC will mark reachable and will NOT swipe the destroyed Activity instance because there is still an implicit reference from `AsyncTask` to it, and `AsyncTask` is **still running**.


#### Ep 05

* By default, `AsyncTask` execute serially on another thread (already note above), which mean that dealing with an 8Mpxs block of data might stall other `AsyncTask`'s packages that UI thread are waiting for. And this is exactly what `HandlerThread` is for, it effectively a long-running thread that grabs work from a message queue and operates on it, usually when need update multiple UI elements or have repeating tasks.
  Eg: Delegate Camera.open() to HandlerThread, so the preview frames callback will land on the HandlerThread rather than blocking UI or AsyncTask thread.

* When create a `HandlerThread` set thread priority for it based on the type of work is doing. Because CPU can only handle a small number of thread in parallel. By setting priority you help the system know the right ways to schedule jobs. (thread priority is important, explained below).

* Note that `HandlerThread`s run outside of your activity’s lifecycle, so they need to be cleaned up properly or else you will have thread leaks.


#### Ep 06

* `ThreadPoolExecutor` let you do more customization when using thread to make use of the number of processor available in the devices. It help you spin up a number of threads and toss blocks of work to execute on it, handle all heavy lifting of spinning up the threads, load balancing work across those threads, even killing those threads when they've been idle for a while.

* When creating thread pool, we can specify the number of initial threads and the number of maximum threads as the workload in the thread pool changes it will **scale** the number of alive thread to match.


#### Ep 07

[IntentService](https://developer.android.com/reference/android/app/IntentService.html) helps get intents work off the UI thread.

* `AsyncTask` does NOT really help when there is no UI. `HandlerThread` while you're NOT receiving intents, the thread is **still sitting around and taking up resources**. But there is another option - `IntentService` - a hybrid between a service and a `HandlerThread` (it extends the `Service` class but internally create and wrap a `HandlerThread` to deal with all coming intents).

* The `HandlerThread` process work from a work queue, it means a task from an intent A which take a long time to complete will block other queued up intents coming to `IntentService`.

* Can use `BroadcastReceiver` to communicate and update UI. But there is a more performant way is use `LocalBroadCastManager` to [report work status](https://developer.android.com/training/run-background-service/report-status.html#ReportStatus) or just use `runOnUiThread()` method after work complete to push a block of work into UI thread handler.

* Use `IntentService` put app into middle of two states "HAS foreground activity" and "NO foreground activity". Help app a little less likely to be killed by the system than just use only `Thread`.


#### Ep 08

What do we do with the threaded work when the activity that kicked it off is no longer alive? 

If a thread hold a **strong reference** to the views to update when work complete, BUT the activity end **before the work complete** -> the activity and entire views tree will be keep on memory until that work finished, and this is a memory leak. And even the late updates of the thread to these views are wasted, because they are no longer visible!

Use [Loaders](https://developer.android.com/guide/components/loaders.html)!

* Loaders are wise to the inner working of the activity lifecycle, so you can ensure your work end in the right place everytime! Resistant to memory leaks, always update the correct views and never repeat unnescessarily.

* Instead of kicking off your work on `AsyncTask` or some other threads, ask your activity for instance of a [LoaderManager](https://developer.android.com/reference/android/app/LoaderManager.html). Send the work to the manager and it will make sure your work will be handle properly. 

* The `LoaderManager` also cache the work results so that won't be repeated with future changes.

* When an activity with an active loader is popped out of the stack and never return, the `LoaderManager` made a callback saying the result will never be used. Use this callback to abort the work, clean up, and move on without waste anymore resources.


#### Ep 09

Spawn too many threads into not enough CPUs is an old problem, thread scheduling has solved this by using various metrics to determine which thread gets the next slice of CPU time. Every thread is assigned a priority, the scheduler will **prefer more** to thread that are more important but **still ballance** with the need to eventually get all it's work done.

Priorities are assigned in a couple of ways.
* Based on the life cycle activity state of your app. Active and visible apps are assigned to the foreground group and will get `90%` of total execution time for the device. So the background group only gets about `10%`.
* When a thread is created by default it's given the **same priority and group memberships** as the "spawner" thread. So if UI thread spawn 20 other worker thread, they will all compete equally for CPU time allocation. -> Explicitly set the priority for any thread that you created in your application.


#### Ep 10

See the reference video.


#### Ep 11

With pieces of data that will be used multiple times, fetch it from the network and cache on the device. But by default, HTTP Caching is disabled for Android apps.

Turn caching on using [HttpResponseCache](https://developer.android.com/reference/android/net/http/HttpResponseCache.html) which allow you to define a **location** on the device and the **max size** of the cache. From now, all the HTTP response will be cached on the file system.

With HTTP response cache, data is evicted from the device in `2` ways: First, if the cache fills up, the system will delete the oldest unused files to make room for the new. Second, files will be removed according to their `Cache-Control` header information which included in server's responses (Ex: `max-age=3600`).

The drawbacks is: API server can decide to cache or not, cache values can conflict with physical resources on device,.. -> Write your **own Disk Cache management** (clone and modify the `DiskLRUCache`) or creating your **own caching logic** based on the type of the data and state of device.


#### Ep 12

**Do not over sync** data thru network. Because networking is the single biggest battery hog. It's not only drain battery to initialize the chip but then it keep awake for an additional `20-60` seconds after request completed.

**Avoid pull server regularly for updates** (could wasting bandwidth, battery waiting for server telling you that nothing changed). Instead use [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) which will **let the server signal the app when there is new content**.

Or when in cases that you have to sync in an specific time interval, if no new data is available **double length of the time interval** that need to wait until check again. Or adjust the time interval base on user activity or device's state (user is busy, device sleep mode, wifi connected, charge plugged in,..).

Use [GCMNetworkManager](https://developers.google.com/android/reference/com/google/android/gms/gcm/GcmNetworkManager) to schedule network tasks and handle batching.


#### Ep 13

**Prefetching** is about predicting what data would be in future request and grabbing all data now while there is an active radio connection.

Like said above, each radio request has some overhead in term of time that it takes to wake up the radio and battery drain as a result of keep awake time. Being able to bundle future requests together and do them now means being able to reduce that overhead.

But prefetching is a tricky balancing problem. Prefetch too little and you'll end up not optimize your **bandwidth effectively**, but prefetch too much means the user is going to be **waiting even longer** for the result.

On 3G a quality prefetch is about `1-5Mb` of data that user might need in the next `1-2` minutes of their active session. Modifying prefetch code to adjust/optimize **based on the quality of the user's connection** (the easiest way to determine the health of the network is simply how long it takes for some well-known pieces of content).


#### Ep 14

To adapting to latency means adjust how apps work based on the connectivity of the device.

1. **Gather information about the speed and performance of the network**. Use [ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager.html) to determine if you are on mobile network, then get a network subtype to check with [this](/resources/network_subtype.PNG) handy look up table to know which kind of network you are dealing with and what to expect the **best possible bandwidth and latency**. But in the real world you'll get much worse result not only from **slow network** but also **slow data server**. So a more accurate (but complex) solution is evaluate the "real" time it takes to grab response from this network.
2. **Design apps to respond to changes in bad latency environments**. You can define `2` thresholds and `3` bucket of classification. If your latency is less than `60ms` to load an asset then user is on great connection, you may be able to be **more** aggressive about the amount of **prefetching data** and **less** aggressive about things like **caching**. If the latency is between `60ms` and `220ms` you need to become **more** aggressive about **batching and caching** and **defer some data requests**. If the latency is above the max `220ms` you should find ways to defer more data requests, **only fetch critical data** until later when there might be a better network connection.

Use [Emulator throttling](https://developer.android.com/studio/run/emulator.html#netspeed) to throttling the bandwidth or use [AT&T Network Attenuator](https://developer.att.com/blog/at-amp-t-network-attenuator) to test how your app respond when latency go sky high.



**References:**
1. [Threading Performance 101](https://www.youtube.com/watch?v=qk5F6Bxqhr4).
2. [Understanding Android Threading](https://www.youtube.com/watch?v=0Z5MZ0jL2BM)
3. [Memory & Threading](https://www.youtube.com/watch?v=tBHPmQQNiS8)
4. [Good AsyncTask Hunting](https://www.youtube.com/watch?v=jtlRNNhane0)
5. [Getting a HandlerThread](https://www.youtube.com/watch?v=adPLIAnx9og)
6. [Swimming in Threadpools](https://www.youtube.com/watch?v=uCmHoEY1iTM)
7. [The Zen of IntentService](https://www.youtube.com/watch?v=9FweabuBi1U)
8. [Threading and Loaders](https://www.youtube.com/watch?v=s4eAtMHU5gI)
9. [The Importance of Thread Priority](https://www.youtube.com/watch?v=NwFXVsM15Co)
10. [Profile GPU Rendering : M Update](https://www.youtube.com/watch?v=erGJw8WDV74)
11. [#Cachematters for networking](https://www.youtube.com/watch?v=7lxVqqWwTb0)
12. [Optimizing Network Request Frequencies](https://www.youtube.com/watch?v=nDHeuEM30ks)
13. [Effective Prefetching](https://www.youtube.com/watch?v=GajI0uKyAGE)
14. [Adapting to Latency](https://www.youtube.com/watch?v=uzboHWX3Kvc)
