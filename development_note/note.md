# Android Development Notes


## Bitmap

* Choose decode method.
* Load a sealed down version to memory.
* Use [LRUCache](https://developer.android.com/reference/android/util/LruCache.html) to [cache bitmaps](https://developer.android.com/topic/performance/graphics/cache-bitmap.html#memory-cache), keep reference objects using strong reference in `LinkedHashMap`, clear the least used before the cache is full to make room for new bitmaps.
* Use [DiskLRUCache](https://github.com/JakeWharton/DiskLruCache) to cache bitmap on hard disk (avoid load again when process interupted).


## [Threading](threading.md)

* [Process and Thread](threading.md#process-and-thread)
* [Android Threads](threading.md#android-threads)
* [Ways to get work off main thread](threading.md#ways-to-get-work-off-main-thread)
* [Race Condition](threading.md#race-condition)


## [Memory Management](memory_management.md)

* [Stack vs Heap in memory](memory_management.md#stack-vs-heap-in-memory)
* [Android heap](memory_management.md#android-heap)
* [Java heap](memory_management.md#java-heap)
* [Native heap](memory_management.md#native-heap)
* [Ashmem (ashmem.c)](memory_management.md#ashmem-(ashmem.c))


## [Garbage Collector](gc.md)

Garbage collection (GC) is a form of automatic memory management. The garbage collector, or just collector, attempts to reclaim garbage, or memory occupied by objects that are no longer in use by the program. 

* [Garbage Collection Roots—The Source of All Object Trees](gc.md#garbage-collection-roots—the-source-of-all-object-trees)
* [Stop the world event!](gc.md#stop-the-world-event!)
* [Mark Sweep Compact (MSC)](gc.md#mark-sweep-compact-(MSC))
* [Concurrent Mark Sweep (CMS)](gc.md#concurrent-mark-sweep-(CMS))


## Memory leaks

* [Unregister listener](memory_leaks.md#unregister-listener)
* [Non-static nested class (inner class)](memory_leaks.md#non-static-nested-class-(inner-class))
* [Anonymous Class](memory_leaks.md#anonymous-class)
* [Bitmap](memory_leaks.md#bitmap)
* [Context](memory_leaks.md#context)
* [Resource not closed](memory_leaks.md#resource-not-closed)


## [Loader](loader.md)


## [Virtual Machine](vm.md)

* [Why Android use virtual machine](vm.md#why-android-use-virtual-machine)
* [Java Virtual Machine](vm.md#java-virtual-machine)
* [Dalvik Virtual Machine](vm.md#dalvik-virtual-machine)
* [JVM vs DVM](vm.md#jvm-vs-dvm)


## [How to build an APK](apk_build.md)


## [Android Runtime](art.md)

* [Android Runtime vs Dalvik Virtual Machine](art.md#android-runtime-vs-dalvik-virtual-machine)
* [Just-in-time Compile vs Ahead-of-time Compile](art.md#just-in-time-compile-vs-ahead-of-time-compile)


## [Android main components](android_main_components.md)

* [Context](android_main_components.md#context)
* [Activity](android_main_components.md#activity)
* [Service](android_main_components.md#service)
* [BroadcastReceiver](android_main_components.md#broadcastreceiver)
* [ContentProvider](android_main_components.md#contentprovider)
* [Intent](android_main_components.md#intent)
* [Handler](android_main_components.md#handler)
* [aidl](android_main_components.md#aidl)


## [Android Service](services.md)


## [Java collections](java_collections.md)

* [Overview](java_collections.md#overview)
* [Abstraction](java_collections.md#abstraction)
* [Map and Dictionary](java_collections.md#map-and-dictionary)