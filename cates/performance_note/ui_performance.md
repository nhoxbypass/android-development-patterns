# UI Performance

## Avoid slow rendering:

* System draw app's screen after every 1000/60FPS = `16.7ms`. If our app CANNOT complete logic in `16.7ms` it will force system to **drop frame** which called a [jank](https://developer.android.com/topic/performance/vitals/render.html#identify). 
* To avoid it we can: use [Layout Inspector](https://developer.android.com/studio/debug/layout-inspector.html), [profile GPU rendering](https://developer.android.com/topic/performance/rendering/profile-gpu.html), use `ConstrainLayout` to reduce nested layout, move `onMeasure`, `onLayout` to background thread, use `match_parent`. 
* Long-running task should run asynchronous outside UI thread.
* Avoid nested `RecyclerView`, avoid call `notifyDatasetChanged()` in RV adapter with small update. Apply [DiffUtil.Callback](https://developer.android.com/reference/android/support/v7/util/DiffUtil.Callback) to calculate the diff between two lists to minimal number of updates.
* Use [RV Prefetch](https://medium.com/google-developers/recyclerview-prefetch-c2f269075710) to reduce the cost of inflation in most case by doing the work ahead of time, while UI thread is idle. 
* RV `onBindViewHolder()` or ListView `onBind()` called on UI thread -> it should be very simple, and take much less than one millisecond for all but the most complex items, it should only get data from POJO and update ui.


## Layout:

* Optimize layout hierachy to avoid nested `LinearLayout` that use `layout_weight` or nested `RelativeLayout`. 
* Reuse layout using `<include>` or `<merge>`. Use dynamic views with views rarely use. Use `RecyclerView` instead of the old `ListView`. 
* Apply `ViewHolder` design pattern for lists to reuse row item view instead of create new View object. This will avoid `findViewById()` overhead. (Number of row item views = number of rows inside screen + 2 (threshold) -> recycle row item views to reuse when scroll).
* Use `lint` tool to check layouts.