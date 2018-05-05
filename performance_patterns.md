# Android Performance Patterns

### Season 5 Ep 01: Threading Performance 101

Android redraw the screen every `16.67 ms` to keep the app smoothly at `60 FPS`. If your LameWork is longer, it will cause **dropped frame**. So you must get all of your heavy workload **off** UI (main) thread and communicate back when all of the work are done. Android provide many ways to do so:

* [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html): Helps get workload on/off main thread the right way. Usually use for simple network operations which do NOT require downloading a lot of data or disk-bound tasks that might take more than a few milliseconds.
* [HandlerThread](https://developer.android.com/reference/android/os/HandlerThread.html): Dedicated thread for API callbacks to land on. Use for long tasks that requires sequentially communication between the worker thread and the caller thread
* [ThreadPool](https://developer.android.com/training/multiple-threads/run-code.html): Break your workload up into a really small package, and toss them to a bunch of threads. To exploit the powerful multi-core processor and run the tasks concurrently on more than one worker threads.
* [IntentService](https://developer.android.com/reference/android/app/IntentService.html): Helps get intents work off the UI thread. Use for long scheduled tasks that should run on the background, independent of your activity.

Another thing need to care is memory. Threading and memory never really played well together. It can cause **memory leaks** (using inner class `AsyncTask` then activity get destroyed but `AsyncTask`s & threads are still running and have have reference to this activity,..)


### Season 5 Ep 02: Understanding Android Threading

* Since thread will **die** when run out of work, so you need to ALWAYS have some sort of **loop running** on the thread.
  
  Eg: [Looper aka Event Loop](https://developer.android.com/reference/android/os/Looper.html) which keep the thread alive and pop tasks/messages off the waiting queue to execute on,..

* In addition, you need some sort of **waiting queue** that the loop can **pull blocks of work** out for execution, and some other **threads that create work** packages and **push them into the queue** for execution.
  
  Eg: [Handler](https://developer.android.com/reference/android/os/Handler.html) receive tasks/messages from other threads and add to the waiting queue, which give us control to push work at the head/tail or set a time-based delay for blocks of work,..

* And the unit/block of work in Android are explicitly defined as `Intent`s, `Runable`s or `Message`s,.. vary depend on who's issuing them and who's consuming them.

And the combination of all these things together is a `HandlerThread`.
When an app run, the system create for it a process which contain a thread of execution called main thread or UI thread, which is just a `HandlerThread`.


### Season 5 Ep 03: Memory & Threading

* View can be reference from worker thread to update UI after execute jobs, but this view can be **removed** from the view hierachy **before the jobs done**.

* If an activity is **destroyed** (by calling `finish()` or user rotate the screen or being killed by system,..) but there still exists a threaded block of work that references it, the Activity and entire view-tree will **NOT get collected by GC until that block of work finishes**.
  Eg: If you kick off some work and the user rotates the screen three times in-a-row before that work completes, you could end up with 3 instances of that Activity resident in memory!
  It's NOT just the explicit references to UI objects that you need to worry about but also the implicit references (non-static nested (inner) classes,..).

* Should NOT hold references to any type of UI objects in any of threading scenarios. But how? Use a unique **update function** to update new information for the views **or drop** the work if the view is NOT there anymore.


### Season 5 Ep 04: Good AsyncTask Hunting

* All `AsyncTask` created will **share the same thread** and thus will [execute in a serial fashion](https://stackoverflow.com/questions/18661288/android-two-asynctasks-serially-or-parallel-execution-the-second-is-freezing) from a **single message queue**. So if the `AsyncTask` take too long to complete it will **freeze the thread** from doing future work (Unless you use `Executor` with thread pool).
  Eg: If you kick off 20 work orders and the 3rd take an hour -> the other 17 will be blocked and wait!

* To "cancel" some work: First create and check a `isCancelled` flag. Then report work result or drop.

* Non-static nested or **inner class** will create an **implicit references** to the outer enclosing class. So Activity and entire view hierachy that use inner `AsyncTask` (most famous, well-known case) will be **leaked** if it get **destroyed before the AsyncTask work completed**, GC will mark reachable and will NOT swipe the destroyed Activity instance because there is still an implicit reference from `AsyncTask` to it, and `AsyncTask` is **still running**.


### Season 5 Ep 05: Getting a HandlerThread

* By default, `AsyncTask` execute serially on another thread (already note above), which mean that dealing with an 8Mpxs block of data might stall other `AsyncTask`'s packages that UI thread are waiting for. And this is exactly what `HandlerThread` is for, it effectively a long-running thread that grabs work from a message queue and operates on it, usually when need update multiple UI elements or have repeating tasks.
  Eg: Delegate Camera.open() to HandlerThread, so the preview frames callback will land on the HandlerThread rather than blocking UI or AsyncTask thread.

* When create a `HandlerThread` set thread priority for it based on the type of work is doing. Because CPU can only handle a small number of thread in parallel. By setting priority you help the system know the right ways to schedule jobs. (thread priority is important, explained below).

* Note that `HandlerThread`s run outside of your activity’s lifecycle, so they need to be cleaned up properly or else you will have thread leaks.


### Season 5 Ep 06: Swimming in Threadpools

* `ThreadPoolExecutor` let you do more customization when using thread to make use of the number of processor available in the devices. It help you spin up a number of threads and toss blocks of work to execute on it, handle all heavy lifting of spinning up the threads, load balancing work across those threads, even killing those threads when they've been idle for a while.

* When creating thread pool, we can specify the number of initial threads and the number of maximum threads as the workload in the thread pool changes it will **scale** the number of alive thread to match.


### Season 5 Ep 07: The Zen of IntentService

[IntentService](https://developer.android.com/reference/android/app/IntentService.html) helps get intents work off the UI thread.

* `AsyncTask` does NOT really help when there is no UI. `HandlerThread` while you're NOT receiving intents, the thread is **still sitting around and taking up resources**. But there is another option - `IntentService` - a hybrid between a service and a `HandlerThread` (it extends the `Service` class but internally create and wrap a `HandlerThread` to deal with all coming intents).

* The `HandlerThread` process work from a work queue, it means a task from an intent A which take a long time to complete will block other queued up intents coming to `IntentService`.

* Can use `BroadcastReceiver` to communicate and update UI. But there is a more performant way is use `LocalBroadCastManager` to [report work status](https://developer.android.com/training/run-background-service/report-status.html#ReportStatus) or just use `runOnUiThread()` method after work complete to push a block of work into UI thread handler.

* Use `IntentService` put app into middle of two states "HAS foreground activity" and "NO foreground activity". Help app a little less likely to be killed by the system than just use only `Thread`.


### Season 5 Ep 08: Threading and Loaders

What do we do with the threaded work when the activity that kicked it off is no longer alive? 

If a thread hold a **strong reference** to the views to update when work complete, BUT the activity end **before the work complete** -> the activity and entire views tree will be keep on memory until that work finished, and this is a memory leak. And even the late updates of the thread to these views are wasted, because they are no longer visible!

Use [Loaders](https://developer.android.com/guide/components/loaders.html)!

* Loaders are wise to the inner working of the activity lifecycle, so you can ensure your work end in the right place everytime! Resistant to memory leaks, always update the correct views and never repeat unnescessarily.

* Instead of kicking off your work on `AsyncTask` or some other threads, ask your activity for instance of a [LoaderManager](https://developer.android.com/reference/android/app/LoaderManager.html). Send the work to the manager and it will make sure your work will be handle properly. 

* The `LoaderManager` also cache the work results so that won't be repeated with future changes.

* When an activity with an active loader is popped out of the stack and never return, the `LoaderManager` made a callback saying the result will never be used. Use this callback to abort the work, clean up, and move on without waste anymore resources.


### Season 5 Ep 09: The Importance of Thread Priority

Spawn too many threads into not enough CPUs is an old problem, thread scheduling has solved this by using various metrics to determine which thread gets the next slice of CPU time. Every thread is assigned a priority, the scheduler will **prefer more** to thread that are more important but **still ballance** with the need to eventually get all it's work done.

Priorities are assigned in a couple of ways.
* Based on the life cycle activity state of your app. Active and visible apps are assigned to the foreground group and will get `90%` of total execution time for the device. So the background group only gets about `10%`.
* When a thread is created by default it's given the **same priority and group memberships** as the "spawner" thread. So if UI thread spawn 20 other worker thread, they will all compete equally for CPU time allocation. -> Explicitly set the priority for any thread that you created in your application.


### Season 5 Ep 10: Profile GPU Rendering : M Update

See the reference video.


### Season 4 Ep 1: #Cachematters for networking

With pieces of data that will be used multiple times, fetch it from the network and cache on the device. But by default, HTTP Caching is disabled for Android apps.

Turn caching on using [HttpResponseCache](https://developer.android.com/reference/android/net/http/HttpResponseCache.html) which allow you to define a **location** on the device and the **max size** of the cache. From now, all the HTTP response will be cached on the file system.

With HTTP response cache, data is evicted from the device in `2` ways: First, if the cache fills up, the system will delete the oldest unused files to make room for the new. Second, files will be removed according to their `Cache-Control` header information which included in server's responses (Ex: `max-age=3600`).

The drawbacks is: API server can decide to cache or not, cache values can conflict with physical resources on device,.. -> Write your **own Disk Cache management** (clone and modify the `DiskLRUCache`) or creating your **own caching logic** based on the type of the data and state of device.


### Season 4 Ep 2: Optimizing Network Request Frequencies

**Do not over sync** data thru network. Because networking is the single biggest battery hog. It's not only drain battery to initialize the chip but then it keep awake for an additional `20-60` seconds after request completed.

**Avoid pull server regularly for updates** (could wasting bandwidth, battery waiting for server telling you that nothing changed). Instead use [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) which will **let the server signal the app when there is new content**.

Or when in cases that you have to sync in an specific time interval, if no new data is available **double length of the time interval** that need to wait until check again. Or adjust the time interval base on user activity or device's state (user is busy, device sleep mode, wifi connected, charge plugged in,..).

Use [GCMNetworkManager](https://developers.google.com/android/reference/com/google/android/gms/gcm/GcmNetworkManager) to schedule network tasks and handle batching.


### Season 4 Ep 3: Effective Prefetching

**Prefetching** is about predicting what data would be in future request and grabbing all data now while there is an active radio connection.

Like said above, each radio request has some overhead in term of time that it takes to wake up the radio and battery drain as a result of keep awake time. Being able to bundle future requests together and do them now means being able to reduce that overhead.

But prefetching is a tricky balancing problem. Prefetch too little and you'll end up not optimize your **bandwidth effectively**, but prefetch too much means the user is going to be **waiting even longer** for the result.

On 3G a quality prefetch is about `1-5Mb` of data that user might need in the next `1-2` minutes of their active session. Modifying prefetch code to adjust/optimize **based on the quality of the user's connection** (the easiest way to determine the health of the network is simply how long it takes for some well-known pieces of content).


### Season 4 Ep 4: Adapting to Latency

To adapting to latency means adjust how apps work based on the connectivity of the device.

1. **Gather information about the speed and performance of the network**. Use [ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager.html) to determine if you are on mobile network, then get a network subtype to check with [this](/resources/network_subtype.PNG) handy look up table to know which kind of network you are dealing with and what to expect the **best possible bandwidth and latency**. But in the real world you'll get much worse result not only from **slow network** but also **slow data server**. So a more accurate (but complex) solution is evaluate the "real" time it takes to grab response from this network.
2. **Design apps to respond to changes in bad latency environments**. You can define `2` thresholds and `3` bucket of classification. If your latency is less than `60ms` to load an asset then user is on great connection, you may be able to be **more** aggressive about the amount of **prefetching data** and **less** aggressive about things like **caching**. If the latency is between `60ms` and `220ms` you need to become **more** aggressive about **batching and caching** and **defer some data requests**. If the latency is above the max `220ms` you should find ways to defer more data requests, **only fetch critical data** until later when there might be a better network connection.

Use [Emulator throttling](https://developer.android.com/studio/run/emulator.html#netspeed) to throttling the bandwidth or use [AT&T Network Attenuator](https://developer.att.com/blog/at-amp-t-network-attenuator) to test how your app respond when latency go sky high.


### Season 4 Ep 5: Minimizing Asset Payload

Big asset files drain more battery & cost user money. So we need to **minimize the assets payload**. And there is two biggest candidates: Images and serialized data.

1. For images, just change the type that you are using. If you don't need transparency **avoid using PNG file**, because they don't compress like JPEG or WEBP. And if you are already using JPEG, remember that a **small changes in quality can have a huge changes in file size** so you can crank down the quality settings of an image a significant amount before user start to notice any issues. So finding the right trade off between quality and size can be a huge win. There's really no reason to send a `4 Mpx` image down to device for only use as a thumbnail or smaller screen devices (like smart watches) can't even display full of it. So you can **store different quality & resolution of an image on your server** to optimize for the smallest possible file to be sent to the users.
2. Serialized formats data (JSON, XML,..) jammed too much un-needed data to make them more readable by human, instead leveraging binary serialization formats as [proto buffs](https://developers.google.com/protocol-buffers/), [flat buffers](https://google.github.io/flatbuffers/) are all accessible on Android that can reduce data foodprint significantly. Any of data you serialize is going to be [GZIP compressed by HTTP stack](https://en.wikipedia.org/wiki/HTTP_compression) so you should adopting a [Struct-of-Arrays](https://www.youtube.com/watch?v=qBxeHkvJoOQ) format to help bundle similar typed fields together so the LZ stage of the GZIP compressor can do a better job finding symbol matches.


### Season 4 Ep 6: Service Performance Patterns

Service aren't free (cost time & memory), service also run on UI thread so it can cause dropping frame. So **don't use Services of you don't have to!**

If you must use Serivce, follow the one primary rule: **do not let services live longer than they are needed**. There are 2 distinct type of services with 2 distinct ways to terminate them. Started services that use `startService()` stay alive until they are explicitly stop with a `stopSelf()` or `stopService()` call (or your app ended). Bound services that use `bindService()` stays around consume resources until all of it's client unbind from it by calling `unnindService()` (or your app ended).

Mixing these 2 types of service is useful but it's easy to cause error. Eg: create service using `startService()` then call `bindService()` for IPC communication, the problem is even client called `unbindService()` it will NOT terminate yet because it's waiting around for a `stopService()` to be called.


### Season 4 Ep 7: Removing unused code

3rd party libraries help you with the heavy lifting, this is totally fine because many of these libs are heavily tested and have proven the stress of production, the problem is you may **have to import the entire lib when you're just use a subset**. The extra code is called code bloat and turned into overhead that get shipped with your APK. 

At the simplest level, it will **increase the size of your APK**. Even more is the [64K method limit](https://developer.android.com/studio/build/multidex.html), because the Android Runtime assigns a numeric indentifier to each method with `16` bits wide so if you have more than `2^16` methods in your app, it will NOT be compiled. 

To overcome this, you need **MultiDex**, and trust me you do NOT want to do that. 

Fortunately there is a tool in the Android tool chain that great for hunting down unused code and stripping it from you build. [Proguard](https://developer.android.com/studio/build/shrink-code.html) is a tool that **shrinks, optimizes and obfuscates** your code by removing the unused parts. It also renames classes, fields and methods with semantically obscure names to make it **harder to reverse engineer** your code.

Proguard is NOT so great sorting out other situations like code that uses reflection or code that get called from native code. This may end up giving some false positive when some code is removed and some is not (you can get `NoClassDefFoundError` or `MethodNotFoundException`,..). So you need to adjust Proguard settings based on what lib you are including.


### Season 4 Ep 8: Removing unused resources

Just because you're being frugal your resources doesn't mean that the libs you are included will do the same, and stuff like this that leads to bloated APKs.

Gradle toolchain can **statically analyze** all of your code to find the assets that **aren't being reference** and **automatically** pull them out of your build. To do this, enable the shink resources flag `shrinkResources true` inside your `app/build.gradle`.

There is might be some false positives or false negatives when found some of your assets are getting cut when you want them kept and some of them kept but you want them cut. We can fix this will the `tools:keep` and `tools:discard` attributes to setup desired behavior. And note that Gradle will **ignore resources inside resolutions or multiple language folders** (drawable-hdpi, drawable-xhdpi, values-es, values-fr,..) because these can be needed at **runtime** and the compiler can't really know what user is going to need.


### Season 4 Ep 9: Perf Theory: Caching

Why computer have RAMs? They act as a data cache to **access recent information FAST - like SUPER fast - compared to having to get it from the hard drive or from the Internet**. This is basically what a cache is - **a place to store data that frequently used so that future uses happen as fast as possible**. Anytime that you have an overhead cost for computing, loading or finding a piece of data, a cache can help you do it faster and more effectively.

The most common place for caching is when you have data that is **calculated multiple times but te result always be the same**. Eg: Place a variable that will NOT change the value inside a loop, and every single loop app will need to re-calculate it -> Wasted. Instead calculate one time and place it outside the loop.

Cache helps manage resources for our limited resource environments. See [The Magic of LRU Cache](https://www.youtube.com/watch?v=R5ON3iwx78M).

Caching by pre-computing. Which spend time **offline ahead-of-time to calculate** a huge look up table or massive XML schema, so at runtime you can **simply fetch** that data rather than executing all that expensive overhead and let users wait.


### Season 4 Ep 10: Perf Theory: Approximation

Approximation is all about c**utting corners and making things good enough for both users and developers**. **Do less, when you can**: Using less time, calculating a less precise result that still meets the user's current need. 

Eg: Consider a trip planner app. While users are driving on a long road it's not necessary to poll the server for updates 
exactly every 5 minutes. Instead, use batching and schedule request less pricisely and less frequently to [save battery](https://www.youtube.com/watch?v=fEEulSk1kNY). The app could also calculate a path between 2 cities on the trip and then use the estimated travel time to set an alarm to wake up, turn on the GPS and start offering suggestions again. 

There are approximations as simple as **allowing images to be slightly lower resolution** in order to save memory space and increase rendering speed. Because when user is scrolling thru a list they don't need the fully detailed image yet. You can load that when they're stopping at some rows in the list. 

For a large majority of your code, you don't need 100% exact result. By demanding less from the hardware and still giving users everytihing they need, you can potentially gain a lot of **frame rates** and device's **battery life**. 


### Season 4 Ep 11: Perf Theory: Culling

**Avoid doing unnecessary work**.

Drawing take time, memory and battery to perform. [Overdrawn](https://developer.android.com/studio/profile/inspect-gpu-rendering.html#debug_overdraw) is where pixels are only drawn to the screen to be drawn over by some other pixels later, so all the overlaped pixels are completely wasted because user CANNOT see it. 

A technique to fix this called **Culling** which is essentially means **removing the code or situations when you are wasting processing time** (in any scenario). 

You should identify views that won't contribute to the final scene and avoid drawing them. You also need to avoid drawing objects that are off-screen using [ClipRect](https://developer.android.com/reference/android/graphics/Canvas.html#clipRect(float,%20float,%20float,%20float,%20android.graphics.Region.Op)).

But **culling isn't just for drawing**. Let's consider searching a database for a people information, unimaginative devs might start to begin in the database and search every entry for each criteria but that can take forever! We need to order & optimize the queries to reduce the set of people that need to be searched for each successive criteria.

Culling can apply everywhere, such as realtime services like location. Eg: An app alerts users based on local happening events then it doesn't make sense to check for updates to events outside the area where users live.


### Season 4 Ep 12: Perf Theory: Threading

For computations that are taking a long time, consider **calling in reinforcements with threads to operate data in parallel** and reduce the overall time required to complete the task.

Because entire app run default on the main thread which used for updating the UI, so to avoid ANR you should move extra complex work off the main thread so users can continue to interact with app. You have to rethink the entire approach in oder to properly integrate threads, so to avoid the rabit hole, take advantages of the [Android framework](https://developer.android.com/guide/components/processes-and-threads.html) which has been built to help you out.


#### Season 4 Ep 13: Perf Theory: Batching

Almost **everything in computing will cause performance overhead**. This could be something big (Eg: decompressing images which required a lot of memory allocations to store intermediate data,..) or can be something small (Eg: extra memory copies inside loop or recursive,..).

These performance taxes aren't much of a concern, BUT when it executed MULTIPLE time it will become a big memory and cause a serious performance problem. So we will use `batching` to handle these cases.

Batching is a process of **grouping identical** instances of a task together to let the overhead **happen once**, **not once for each** instance or simply **avoid nesting computation tasks**. Eg: If need to render the same image 20 times (inside a loop or `onDraw()`,..), try loading it once and save in a variable before begin (outside the loop, `onDraw()`), instead of loading it every render time. 

For Android, the most important places you can apply batching is with [networking requests](https://www.youtube.com/watch?v=Ecz5WDZoJok). There is an overhead cost after each network call to "keep alive" (see SS4 Ep02 above). So instead of sending request after n seconds, you should group them together, turn the network on and send all those requests at the same time!

Batching also help when rendering custom views. Rather than computing a transformation matrix for every single item, you should group and just make a small change to this larger transform matrix.

Batching is so important that all modern processors now come equipped with mathematical batching support. 


### Season 4 Ep 14: Serialization performance

**Serialization is a process of taking some in-memory objects and convert it to a formatted chunk of data and they can be converted back to in-memory objects later**. Serialization is everywhere: sending package between servers and devices, sending data between processes, store preferences to disk. 

You could implements `Serializable` to apply it but in term of **performance it's the worst solution**. So use [Gson](https://github.com/google/gson) which produces much faster serialization and much more memory-efficient results, but it require the formatted data to be JSON and JSON is bloated format (jammed too much extra data cause slower decode, see SS4Ep5, note that Android layout/drawable,.. XML file don't have this problem because it compiled at build time). 

For starters, the [google/protobuf](https://github.com/google/protobuf) library get a lot of recognition for being very compact, flexible format for serialization but the Java implement of this lib has memory & code size overhead. This is why the nano-proto-buffers (protobuf but optimize for Android) was made. If you want to focus on performance (like games) use [google/flatbuffers](https://github.com/google/flatbuffers) library, it will produce smaller files and minimize encoding/decoding time compare to protobuf. 

The most performant way is **don't serialize**. Eg: Use [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html) API instead of serialize preferences then save to disk. Or use `Parcelable` instead of `Serializable` when passing data between activities/processes which give a slightly serialized format but win a **huge performance boost**. Or if you you got a lot of data, don't serialize it instead create a local database using `SQLite`. 


### Season 04 Ep 15: Smaller Serialized Data

In most of the serialized form of data **property name duplicate** often and that will cause larger files. GZIP is NOT enough because it compresses data by **finding duplicate strings** in your file as long as they are within a window of 32K characters, so the larger your serialized classes are, the **longer distance between 2 duplicate** data will be. So cause less duplicate data which resulting in **less compression savings**.

Using the Arrays-of-Struct create larger serialized class & less compression savings, to solve this we use **Struct-of-Arrays** form. Struct-of-Arrays is take all of **one property from every element & list them together in an array of their type** and then do this for each property in the class. 

You can get even better compression & better serialization by **adopting binary serialization formats** (protobuffs, nanobuffs or FlatBuffer,..).


### Season 04 Ep 16: Caching UI data

At some point in the past, you've actually grabbed a valid block of UI information (except first load), although the data is outdated your app still CAN **use that cached information when the fresh data hasn't been fetched yet**. 

Upon a successful fetch of some UI data, **serialize it to persistence storage** (or memory for quick load but will be erase when app's process stop) along with timestamp to know how old the information is. When do a [cold start](https://developer.android.com/topic/performance/launch-time.html#cold) you can use this data to **start drawing UI immediately while also kicking off network request to fetch the freshest data**. But you need to notify user for this. 

There are right and wrong ways to do caching data. Firstly you should use binary serialization format to have smaller file & better performance than human readable formats like JSON or XML. And if UI is depent on a lot of complicated queries accross this data serialzation may NOT be the best idea compare to storing in SQLite.
