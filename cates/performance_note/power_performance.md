# Power Performance

Battery life is the single most important aspect of the mobile user experience. A device without power offers no functionality at all. For this reason, it is critically important that apps be as respectful of battery life as possible.

There are three important things to keep in mind in keeping your app power-thrifty:

* Make your apps Lazy First.
* Take advantage of platform features that can help manage your app's battery consumption.
* Use tools that can help you identify battery-draining culprits.

Ex:

* Reduce network call. 
* Avoid [wake lock](https://developer.android.com/training/scheduling/wakelock.html). 
* Use GPS and `AlarmManager` carefully. 
* Perform [batch scheduling](https://developer.android.com/topic/performance/scheduling.html).
