# Memory leaks

A memory leak happens when memory is allocated but never freed. This means the garbage collector is not able to take out the trash once we are done with the takeout. So the memory of our app will constantly increase until no more memory can be allocated to our app, leading to `OutOfMemoryError` which ultimately crashes the app

## Unregister listener

Without calling the unregister method, the instance will probably **keep a reference** around long after the referenced object has been terminated and will thus start leaking memory. But it [depends](https://stackoverflow.com/a/5010949/5282585) on what those listeners are registered upon.
Eg: a simple well-written `OnClickListener` for a button should not result in a memory leak. However, a `LocationListener`, registered with the `LocationManager` - which is a **system service & keep living until device shutdown** - is held by the process. Hence, even if the activity is destroyed, the **listener will remain registered**. If that listener is an **inner class**, it will continue to hold an implicit reference to the activity, and you will have a memory leak.


## Non-static nested class (inner class)

Inner class hold a **strong implicit reference** to outer enclosing class. So Activity and entire view hierachy that use inner AsyncTask (most famous, well-known case) will [be leaked](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-04) if it get **destroyed before** the AsyncTask work completed.
-> Use static nested class, send objects that needed from instance of outer class through constructor and hold inside `<WeakReference>`.


## Anonymous Class

Also an inner class. It usually cause memory leaks by an object have reference to this anonymous class without any information about it's outer container object, **when the outer container object not use anymore, but the anonymous class still alive** (because of still have a reference from outside) then this anonymous class and entire outer container object leaks and cannot be GC.


## Bitmap

-> Remember to `setBackgroundResource(null)` and `recycle()` bitmaps after use.


## Context

Can leak entire Activity + view tree when device roration.
-> Avoid static context, use `getApplicationContext()` for long-live object.


## Resource not closed
