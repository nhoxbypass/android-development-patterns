# Android Performance Notes

## How to increase Android App Performance?

#### Avoid slow rendering:

* System draw app's screen after every 1000/60FPS = `16.7ms`. If our app CANNOT complete logic in `16.7ms` it will force system to **drop frame** which called a [jank](https://developer.android.com/topic/performance/vitals/render.html#identify). 
* To avoid it we can: use [Layout Inspector](https://developer.android.com/studio/debug/layout-inspector.html), [profile GPU rendering](https://developer.android.com/topic/performance/rendering/profile-gpu.html), use `ConstrainLayout` to reduce nested layout, move `onMeasure`, `onLayout` to background thread, use `match_parent`. 
* Long-running task should run asynchronous outside UI thread.
* Avoid nested `RecyclerView`, avoid call `notifyDatasetChanged()` in RV adapter with small update. Apply [DiffUtil.Callback](https://developer.android.com/reference/android/support/v7/util/DiffUtil.Callback) to calculate the diff between two lists to minimal number of updates.
* Use [RV Prefetch](https://medium.com/google-developers/recyclerview-prefetch-c2f269075710) to reduce the cost of inflation in most case by doing the work ahead of time, while UI thread is idle. 
* RV `onBindViewHolder()` or ListView `onBind()` called on UI thread -> it should be very simple, and take much less than one millisecond for all but the most complex items, it should only get data from POJO and update ui.

#### App lauching time:

We can reduce app lauching time by: Lazy load resources. Allow app to load and display views first, then update visual which depend on bitmap and other resources.

#### Layout:

Optimize layout hierachy to avoid nested `LinearLayout` that use `layout_weight` or nested `RelativeLayout`. Reuse layout using `<include>` or `<merge>`. Use dynamic views with views rarely use. Use `RecyclerView` instead of the old `ListView`. Apply `ViewHolder` pattern for lists. Use `lint` tool to check layouts.

#### Power usage:

Reduce network call. Avoid [wake lock](https://developer.android.com/training/scheduling/wakelock.html). Use GPS and `AlarmManager` carefully. Perform [batch scheduling](https://developer.android.com/topic/performance/scheduling.html).


## Performance Tips

#### Prefer static over virtual

In any class, if you don't need to access an object's fields, make your method static. Invocations will be about `15-20%` faster. And you can tell other devs that calling the method can't alter the object's state.

#### Use static final for constants

In any class, if use `static int val = 22` the compiler generates a class initializer method - called `<clinit>`, that is executed when the class is first used - to store `22` into `val` to access later with field lookup. 

If we add the `final` keyword like `static final int val = 22` this class no longer requires a `<clinit>` method, because the constants go into static field initializers in the dex file and can be refered directly later without field lookup.

Note: This optimization applies ONLY to primitive types and String constants, not arbitrary reference types. Still, it's good practice to declare constants static final whenever possible.

#### Use enhanced foreach syntax for loops

`foreach` loop can be used for collections that implement the Iterable interface and for arrays, should use it by default because it's faster (with JIT) or equal (without JIT) than normal counted for loop.

But with an `ArrayList`, a hand-written counted loop is about 3x faster than foreach loop.

#### Use package instead of private access modifier with inner class

Consider define an inner class (inner class cannot be static) `Foo$Inner` that directly accesses private methods/fields in the outer class. This is legal in Java. 

But the problem is that the VM considers direct access to `Foo`'s private members from `Foo$Inner` to be illegal because `Foo` and `Foo$Inner` are different classes. To bridge the gap, the compiler generates [synthetic methods](https://vanillajava.blogspot.com/2011/07/java-secret-generated-methods.html):

```
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mPrivateVal;
}
/*package*/ static void Foo.access$200(Foo foo, int val) {
    foo.privateMethod(val);
}
```

The inner class code calls these static methods whenever it needs to access the `mPrivateVal` or invoke the `privateMethod()` of the outer class. This is an "invisible" performance hit because accessors are **slower** than direct field accesses, and it will **hurt the 64K methods limit** of Android too.

#### Avoid using floating point

As a rule of thumb, floating-point is about `2x` slower than integer on Android-powered devices.

In speed terms, there's no difference between float and double on the more modern hardware. Space-wise, double is `2x` larger. As with desktop machines, assuming space isn't an issue, you should prefer double to float.

#### Know & use libraries

By prefering library code over your own code you can avoid "re-inventing the wheel" and bear in mind that the system is allowed to replace calls to library methods with hand-coded assembler, which may be better than the best code the JIT can produce for the equivalent Java. The typical example here is `String.indexOf()` and related methods, which Dalvik replaces with an inlined intrinsic. Similarly, the `System.arraycopy()` method is about `9x` faster than a hand-coded loop on a Nexus One with the JIT.

#### Use native methods carefully

Developing your app with native code using the Android NDK is NOT necessarily more efficient than programming with the Java language. 

For one thing, there's a cost associated with the Java-native transition, and the JIT can't optimize across these boundaries. If you're allocating native resources (native heap,..) it's more difficult to arrange timely collection of these resources. You also need to compile your code for each architecture you wish to run on so it will increase APK size. 

Native code is primarily useful when you have an **existing native codebase that you want to port to Android, not for "speeding up" parts of your Android app written with the Java**. 

#### Avoid using enum

Enum cause bigger apk size, 4 byte reference + enum size.
-> Use constants or [IntDef](https://stackoverflow.com/questions/32032503/enums-and-android-annotation-intdef) instead.
