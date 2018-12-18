# Memory Management

## Stack vs Heap in memory

All stored in RAM. But thread have it own stack but share heap memory.

| Stack        | Heap          |
| -------------|---------------|
| Auto delete memory when variable go out of scope | Remain until collect by program or GC triggerd |
| Fixed [size](https://stackoverflow.com/questions/16843357/what-is-the-android-ui-thread-stack-size-limit-and-how-to-overcome-it) -> cause [StackOverflow](https://stackoverflow.com/questions/214741/what-is-a-stackoverflowerror) | Can get more memory from OS |
| Faster, only need to move pointer to allocate | Slower, synchronize to keep thread safe, complex flow to allocate |
| Attach to thread, when thread stop -> reclaim | Attach to app process, process exit -> reclaim |
| Store local data, return data, params address, primitive variables | Store block of data, all object instance, static variable (Perm Gen) |
| Fixed [size](https://stackoverflow.com/questions/16843357/what-is-the-android-ui-thread-stack-size-limit-and-how-to-overcome-it) vary by Android API | Max 64Mb default in Java, max Android heap size vary by device's screen size |
| Cause `StackOverflow` | Cause memory fragmentation, memory leaks, `OutOfMemoryError` |


## Android heap

**Android heap contain Java heap, native heap and ashmem**. 

Note that memory usage on modern operating systems like Linux is an extremely [complicated and difficult to understand area](https://stackoverflow.com/questions/2298208/how-do-i-discover-memory-usage-of-my-application-in-android). 

Android limit heap size per process.
Max heap size is vary [by screen resolution](https://stackoverflow.com/a/5352488/5282585). As higher-resolution screens tend to want to manipulate larger bitmaps -> need more heap -> Google makes heap size recommendations and hope device manufacturers will abide by.

`OutOfMemory` error mostly cause from bitmaps, leak memory that GC cannot free. Enable [hardware accelerate](https://developer.android.com/guide/topics/graphics/hardware-accel.html) (allow draw canvas using GPU) will consume more RAM memory.

* `getRuntime().maxMemory()` is total bytes of heap that our app is **allowed to use**.
* `ActivityManager.getMemoryClass()` is total MB of heap that our app **should use** to respect limit of device.
* Read `heapstartsize`, `heapgrowlimit`, `heapsize` information inside `system/build.prop` file.
* Put `android:largeHeap="true"` in `AndroidManifest.xml` let the heap grow from `heapgrowlimit` to `heapsize` (Android API >= 11). But no guarantee.
* Request native heap using NDK JNI.


## Java heap

| Eden  | S0   | S1      |  Old Gen | Perm Gen |
| -------|-------| ------- | ------- | ------- |
|        |       |         | Major GC |        |


## Native heap

The native heap used by the C++ `new` operator. There is much more memory available here. The app is limited only by the physical memory available on the device. There is no garbage collection and nothing to slow things down. However, C++ programs are responsible for freeing every byte of memory they allocate, or they will leak memory and eventually crash. 

The native heap is managed by `dlmalloc()`, which uses a combination of `mmap()` and standard calls like `sbrk()` to allocate memory. The managed ("Dalvik") heap is (mostly) one large chunk allocated with `mmap()`. It's all running on top of the Linux kernel.


## Ashmem (ashmem.c)

* short term for **Android Shared Memory**.
* This operates much like the native heap but has additional system calls.
* Share memory between process using memory maps by name.
* Android can "unpin" the memory rather than freeing it. This is a lazy free; the memory is freed only if the system actually needs more memory. Then "pin" later on when device have enough memory back.
* Better support low-mem devices because it can discard shared mem unit under memory pressure.
* Removed in Android 5.0 and higher.
* Using C.
