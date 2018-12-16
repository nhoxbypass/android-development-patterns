# Garbage Collector

Garbage collection (GC) is a form of automatic memory management. The garbage collector, or just collector, attempts to reclaim garbage, or memory occupied by objects that are no longer in use by the program. 

## Garbage Collection Roots—The Source of All Object Trees

Every object tree **must** have one or more root objects. As long as the application can reach those roots, the whole tree is reachable. But when are those root objects considered reachable? Special objects called garbage-collection roots are always reachable and so is any object that has a garbage-collection root at its own root. There are four kinds of GC roots in Java:

* Local variables: are kept alive by the stack of a thread. This is not a real object virtual reference and thus is not visible. For all intents and purposes, local variables are GC roots.
* Active Java threads: are always considered live objects and are therefore GC roots. This is especially important for thread local variables.
* Static variables are referenced by their classes.
* JNI References: are Java objects that the native code has created as part of a JNI call.

![GC](../resources/gc_with_leaks.png "GC")


## Stop the world event!

Simple GCs are [stop-the-world event](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Stop-the-world_vs._incremental_vs._concurrent), it completely halt execution of the program to run a collection cycle, thus guaranteeing that new objects are not allocated and objects do not suddenly become unreachable while the collector is running.

In general GC does not require stop-the-world pause. Modern GC implementations try to minimize blocking "stop-the-world" stalls by doing as much work as possible on the background thread. Also there are JVM implementations which are (almost) pause free (e.g. [Azul Zing JVM](https://www.azul.com/products/zing/)). Whenever JVM require STW to collect garbage depends on algorithm it is using. 

#### Mark Sweep Compact (MSC)

Mark Sweep Compact (MSC) is popular algorithm used in HotSpot by default. It is implemented in STW fashion and has 3 phases:
* **MARK** - traverse live object graph to mark reachable objects
* **SWEEP** - scans memory to find unmarked memory
* **COMPACT** - relocating marked objects to defragment free memory
When relocating objects in the heap, the JVM should update all references to this object. During this process the object graph is inconsistent, that is why STW pause is required.

#### Concurrent Mark Sweep (CMS)

This is the default plan of ART. This is another algorithm in HotSpot JVM which does NOT utilize STW pause for old space collection (not exactly same thing as full collection). Also see: https://stackoverflow.com/a/21230308/5282585/. It scans only the portion of the heap that was modified since the last GC and can reclaim only the objects allocated since the last GC.

There is also **G1** which is an incremental variation of MSC.

GC also have many disadvantages such as consuming additional resources, performance impacts, possible stalls in program execution, and incompatibility with manual resource management. So you should **take care of memory yourself, not just rely on it**
