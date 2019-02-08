# Android Development Notes

Discover surprise Android development knowledge, resources and theory with examples.

## Bitmap

Bitmaps can very easily exhaust an app's memory budget. Loading bitmaps on the UI thread can degrade your app's performance, causing slow responsiveness or even ANR. And it even harder if your app is loading multiple bitmaps into memory.

It is therefore important to skillfully manage threading, memory and disk caching when working with bitmaps.

* [Loading Large Bitmaps Efficiently](bitmap.md#loading-large-bitmaps-efficiently)
* [Caching Bitmaps](bitmap.md#caching-bitmaps)
* [Managing Bitmap Memory](bitmap.md#managing-bitmap-memory)


## Threading

Making adept use of threads & processes on Android can help you boost your app’s performance.

* [Process and Thread](threading.md#process-and-thread)
* [Android Threads](threading.md#android-threads)
* [Ways to get work off main thread](threading.md#ways-to-get-work-off-main-thread)
* [Race Condition](threading.md#race-condition)


## Memory Management

* [Stack vs Heap in memory](memory_management.md#stack-vs-heap-in-memory)
* [Android heap](memory_management.md#android-heap)
* [Java heap](memory_management.md#java-heap)
* [Native heap](memory_management.md#native-heap)
* [Ashmem (ashmem.c)](memory_management.md#ashmem-(ashmem.c))


## Garbage Collector

Garbage collection (GC) is a form of automatic memory management. The garbage collector, or just collector, attempts to reclaim garbage, or memory occupied by objects that are no longer in use by the program. 

* [Garbage Collection Roots—The Source of All Object Trees](gc.md#garbage-collection-roots—the-source-of-all-object-trees)
* [Stop the world event!](gc.md#stop-the-world-event!)
* [Mark Sweep Compact (MSC)](gc.md#mark-sweep-compact-(MSC))
* [Concurrent Mark Sweep (CMS)](gc.md#concurrent-mark-sweep-(CMS))


## Memory leaks

A memory leak happens when memory is allocated but never freed. This means the garbage collector is not able to take out the trash once we are done with the takeout. So the memory of our app will constantly increase until no more memory can be allocated to our app, leading to `OutOfMemoryError` which ultimately crashes the app

* [Unregister listener](memory_leaks.md#unregister-listener)
* [Non-static nested class (inner class)](memory_leaks.md#non-static-nested-class-(inner-class))
* [Anonymous Class](memory_leaks.md#anonymous-class)
* [Bitmap](memory_leaks.md#bitmap)
* [Context](memory_leaks.md#context)
* [Resource not closed](memory_leaks.md#resource-not-closed)


## Loader

The Loader API lets you load data from a content provider or other data source for display in an FragmentActivity or Fragment. It was a solution to a problem that we shouldn’t have really been trying to solve — running asynchronous tasks within a Fragment or Activity and handling the rotation change.

See [more](loader.md)


## Platform Architecture

Android is an open source, Linux-based software stack created for a wide array of devices and form factors. 

* [The Linux Kernel](platform_architecture.md#the-linux-kernel)
* [Hardware Abstraction Layer (HAL)](platform_architecture.md#hardware-abstraction-layer-hal)
* [Android Runtime](platform_architecture.md#android-runtime)
* [System Services](platform_architecture.md#system-services)
* [Native C/C++ Libraries](platform_architecture.md#native-c-c++-libraries)
* [Binder IPC](platform_architecture.md#binder-ipc)
* [Java API Framework](platform_architecture.md#java-api-framework)
* [System Apps](platform_architecture.md#system-apps)


## Virtual Machine

Virtual Machine allow Java/Kotlin programs to run on any device or operating system (known as the "Write once, run anywhere" principle), and to manage, optimize program resources & memory. 

* [Why Android use virtual machine](vm.md#why-android-use-virtual-machine)
* [Java Virtual Machine](vm.md#java-virtual-machine)
* [Dalvik Virtual Machine](vm.md#dalvik-virtual-machine)
* [JVM vs DVM](vm.md#jvm-vs-dvm)


## How to build an APK

The APK is built using Java Compiler (javac), DEX compiler (dx), Android Asset Packaging Tool (aapt) and APK Packager.

See [more](apk_build.md)


## Android Runtime

Android Runtime (ART) is the managed runtime used by applications and some system services on Android. Replacing the predecessor Dalvik, ART performs the translation of the application's bytecode into native instructions that are later executed by the device's runtime environment. 

ART also executes the Dex format and Dex bytecode specification.

* [Android Runtime vs Dalvik Virtual Machine](art.md#android-runtime-vs-dalvik-virtual-machine)
* [Just-in-time Compile vs Ahead-of-time Compile](art.md#just-in-time-compile-vs-ahead-of-time-compile)


## Android basic components

App components are the essential building blocks of an Android app. Each component is an entry point through which the system or a user can enter your app. Some components depend on others.

* [Context](android_main_components.md#context)
* [Activity](android_main_components.md#activity)
* [Service](android_main_components.md#service)
* [BroadcastReceiver](android_main_components.md#broadcastreceiver)
* [ContentProvider](android_main_components.md#contentprovider)
* [Intent](android_main_components.md#intent)
* [Handler](android_main_components.md#handler)
* [aidl](android_main_components.md#aidl)


## Android Services

A service is a general-purpose entry point for keeping an app running in the background for all kinds of reasons. It is a component that runs in the background to perform long-running operations or to perform work for remote processes. 

A service does not provide a UI.

See [more](services.md)


## Java collections

The Java collections framework is a set of classes and interfaces that implement commonly reusable collection data structures. 

* [Overview](java_collections.md#overview)
* [Abstraction](java_collections.md#abstraction)
* [Map and Dictionary](java_collections.md#map-and-dictionary)