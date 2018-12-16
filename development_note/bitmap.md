# Bitmap

Bitmaps can very easily exhaust an app's memory budget. Loading bitmaps on the UI thread can degrade your app's performance, causing slow responsiveness or even ANR. And it even harder if your app is loading multiple bitmaps into memory.

It is therefore important to **skillfully manage threading, memory and disk caching** when working with bitmaps.


## Loading Large Bitmaps Efficiently

Images come in all shapes and sizes. In many cases they are larger than required for a typical app UI. Given that you are working with limited memory, ideally you only want to load a lower resolution version in memory. 

An image with a higher resolution does **NOT provide any visible benefit**, but still takes up precious memory and incurs additional performance overhead (additional on the fly scaling)

#### Read Bitmap Dimensions and Type

#### Choose Decode method.

#### Load a Scaled Down Version into Memory.

Decide if the full image should be loaded into memory or if a subsampled version should be loaded, using:

* Estimate mem used when loading the full image.
* Amount of mem willing to commit to loading this image.
* Dimensions of the target ImageView or UI component to be loaded into.
* Screen size and density of the current device.

Ex: It’s not worth loading a `1024x768` px image into mem if it will only be displayed in a `128x96` px thumbnail in an ImageView


## Caching Bitmaps

Memory usage is kept down with components by recycling the child views as they move off-screen. The GC also frees up your loaded bitmaps (assuming don't keep any long lived references). This's all good, BUT in order to keep a fluid and fast-loading UI you need to **avoid continually processing these images each time they come back on-screen**.

* Use [LRUCache](https://developer.android.com/reference/android/util/LruCache.html) to [cache bitmaps](https://developer.android.com/topic/performance/graphics/cache-bitmap.html#memory-cache), keep reference objects using strong reference in `LinkedHashMap`, clear the least used before the cache is full to make room for new bitmaps.
* Use [DiskLRUCache](https://github.com/JakeWharton/DiskLruCache) to cache bitmap on hard disk (avoid load again when process interupted).

Note: For most cases, we recommend that you use the **Glide** library to fetch, decode, and display bitmaps in your app. Glide abstracts out most of the complexity in handling these and other tasks related to working with bitmaps and other images on Android

## Managing Bitmap Memory

In addition there are specific things you can do to facilitate garbage collection and bitmap reuse.