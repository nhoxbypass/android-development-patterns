# Android Performance Patterns Season 03

## Ep 01: Fun with ArrayMaps

Consider the commonly used HashMap that **totally useful but also complete memory hog**. When putting item, HashMap do "hash-to-index" to convert "key obj" to array index that store "value obj", but the problem is collisions or when multiple key hash to same index location. Small array means more collisions, so most HashMap end up **allocating a large array & adding more crazy logic (chainning,..) to avoid collisions**. And because of that, it is NOT really ideal for low-mem devices.

That's why Android provide `ArrayMap`. ArrayMap **provide functionalities as a HashMap but avoid all crazy overhead by using two array instead of one large one**. The first array contain hashes of the keys in sorted order, the second array store keys/values interwoven to each other & with the order of the sorted keys array. When need to fetch a value, it create hash of the key and then binary search hash array to find the index, then use this index directly to find the location of key/value pair in the interwoven array. 

When HashMap empty the array is still be allocated, but when ArrayMap is empty nothing allocated.

But obviously it's NOT wise to use in every case. But there are some perfect situations like: When you have a small number of items (< 1000) with lots of acesses OR when the frequency of insertion/deletion is low (because ArrayMap cause overhead in these cases) OR when you have containers of maps (such as maps of maps,..) you should change to ArrayMap. Unless you can **stick with the HashMap to avoid some overhead**.


## Ep 02: Beware Autoboxing

Java provide "object version" of primitive type  (like java.lang.Integer,..) which give you the same functionalities but can be used with generic collections and this is where autoboxing come in.

Autoboxing convert from primitive types to object but it comes with some performance penalties. Ex: When add a primitive int to an Integer object, the runtime has to create a new Integer object, push the value into it then add back to other Integer object. Which means **everytime you do autoboxing conversion a new object allocation comes along with it**. 

This is really painful since these object are **larger in size** than primitive (16 bytes for Integer rather than 4 bytes for primitive int) and also required more performance overhead in order to access the underlying value. 


## Ep 03: SparseArray Family Ties

You can avoid one of the largest cause of autoboxing issues: HashMap containers. HashMap has to use the generic object version which eat up memory. And **anytime you fetch an primitive from a generic container, autoboxing happen**. 

Android provide a whole SparseArray family of generic containers built specific to **provide functionalities of the HashMap but allow use primitive types & avoid autoboxing**. SparseArray basically like a ArrayMaps, they reduce overall memory by using 2 tightly packed arrays rather than 1 large array but come with some overhead when fetching object. So only useful for containers with hundreds (< 1000) of objects rather than thousands or milions or when we have a container of container (maps contain maps,..). 

The main diff between SparseArray & ArrayMap that the keys array always contain primitive type rather than generic object type allow saving memory & avoid autoboxing. 


## Ep 04: The price of ENUMs

When your app loaded, Android provide a section of system memory as heap space then DEX code load into that space. When memory gets low, your app will be terminated to free up space. 

Adding enum - which take **extra memory than int constants** - will bloat your DEX file which eats away your heap space.

If enums are widely used across your app, all those small pieces of overhead quickly add up to a substantial amount, you don't really know that enum are causing a problem until they're already infecting your codebase. And at that point, try to fix it is a horrible process.

If you're using int constants instead of enums you can add `@IntDef` annotation. If your code already using many enums, ProGuard can (in many situations) optimize enums to int constants. 


## Ep 05: Trimming and Sharing Memory

Each running app taking a small piece of device limited resources to store state information, graphics resources, allocated heap objects,.. that **stick around in memory even when app is in background**. Overtime device will run out of memory & kill existing app to claim memory back. 

We DO keep background app in memory **for fast switching between background/foreground app**. But when your app get killed, and user navigate back to it, it starts slowly from scratch. 

Your app **does NOT have to be killed**. Instead when memory is low your app can **offer some of its allocated space back to the system in order to avoid being terminated**. 

To do this, when memory is low, the active app will get `onLowMemory()` callback to release resources to help stablize the system, but this callback only get called AFTER all other background apps have been killed. That is why `onTrimMemory()` callback born to allow active apps save background apps from being killed. This callback issued to all running apps, tell them to release memory rather than being killed and can be overriden on application, activity, fragment, service,..

But in order to produce the best UX, should NOT just be reactive to memory situation (in callbacks) but also proactive check if your app are running on a low memory device using `ActivityManager.isLowRamDevice()`


## Ep 06: DO NOT LEAK VIEWS

The worst thing that you can leak in Android app is Views object. 

By themselves view aren't much of a leak problem but what they reference can cause a horrible situation. GC can only reclaim objects that are no longer referenced by anything else. A leak is a object that is no longer needed (unused) but there's still a reference to it somewhere in the system. And this problem can **cascade**.

So here's the problem, **View contain a ref back to the Activity that created them**, and the Activity reference to lots of internal objects (fields, variables,..) and other memory items. This is why a leaked view can cause a big issue. Ex: When user rotate device a configuration changed is triggered causing current activity destroyed & recreate new instance of this activity. But if a View from that 1st activity leaked then the original activity can NOT be cleaned up but sit around in the memory. 

How to fix it: 
* Don't reference Views inside of async callbacks. Because the activity may be **killed before** the callback triggered so the Views & activity may be leaked and keep around in memory (for non-static callbacks). And worst situation your callback may be triggered after the View objects actually destroyed that will cause `IllegalStateException` & crash app.
* Don't reference Views from static object. Because the static object can persist for a lifetime of the entire process of your app but the lifetime of your activity is shorter. Ex: having a static object reference to View & the View still reference to the Activity so when rotate the device, the acitvity and entire view tree are remain in the memory. 
* Don't put View in collection that don't have clear memory patterns. Ex: `WeakHashmap` store Views as hard reference so can end up a bad spot anytime something destroy those views. 


## Ep 07: Location & Battery Drain

Location interval is the milis which your app prefer to receive a location update, the lower interval the more updates you get, but the more battery you burn. If you notice the **position has stayed the same for a while** so user may be stationary for a long duration. So try to increase interval to reduce battery churn. 

There are many ways to get location updates: GPS Provider - using satelite - more accurate but very battery intensive operation, Cell Network Provider - using nearby cell tower & Wifi access point - less accurate result but save battery. 

Using the `FusedLocationProvider` you can simplify the operation, Android will handle & **ballance between battery & location accuracy** for you. 


## Ep 08: Double Layout Taxation

Anytime the position or size of Views changed will affect other neighbor views in the hierachy. **When views changed, layout phases of the rendering pipeline will occur to re-calculate position & size for all views impacted by the change**. In general it's a reasonably fast process but may cause an expensive cascade of layout operations which can increase your frame time.

Ex: RelativeLayout allow to define the position of a view with respect to parent or position of some other views. The issue here is in order to properly position views in relation to another, container must kick of a second layout pass before finalize position & begin render. (The first pass will visit each view & calculate position & size base upon it's own request. Then the RelativeLayout use this data to figure out proper positions of correlated views & make boundary adjustment. Then a second layout pass will kicked off, recalculate to determine the final position to use for rendering). But it's not the only layout that can cause double layout pass, the LinearLayout with `measureWithLargestChild`, or nested LinearLayout.

The big problem is when it cascaded: A RelativeLayout (2x pass) which contain a ListView (2x pass) which every list item is a GridView (2x pass) will cause child views to have 8 times re-layouts for each change.


## Ep 09: Network Performance 101

Networking performance is about **reducing the amount of time between when user want data and when networking return it**. But there is also a second set of performance concern that devs should be aware of: using cell radio for networking is the number one **battery killer** & the more data you transfering the more **money user will be charged**.

So:
* Reduce the amount of time you keep the radio active.
* Reduce the size of data you fetching.

With the stuff that user ask to do (pull to refresh, update now,..) we cannot do anything. But with the stuff that servers need to update for you (notify new emails,..) and the stuff that need to be upload frequently (upload analytics, check device location,..) we can do optimize.

You should NEVER poll the server regularly for updates, because you could wasting bandwidth, battery waiting server for nothing changes. Instead **let the server signal the app when there's new content** (using FCM,..). If this technique cannot be use, you should reduce the frequency using **backoff pattern** that increase interval time when data not change. And rather that letting your request be strung out overtime, try to batch them together so they happen in a short burst to optimize active radio time. Or try **pre-fectching** your data to reduce future requests.


## Ep 10: Effective Network Batching

After a request radio still wait for extra `20-60 secs` to **keep alive** just in case the response from server come in, if nothing comes it will go back to sleep. 

Batching is about **grouping the requests together so you only have to pay the wake up & keep alive once**. Instead of execute network request immediately, store them in a pending queue to execute in the future when reaching threshold in queue size. But there is cases that network radio could be wake up by some other apps before the threshold reach. So you should implement a callback when network radio turn up to execute request queue.



