# LiveData

## Overview

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata): An observable data holder to build data objects that **notify views when the underlying database changes**.

`LiveData` is an observable data holder class. Unlike a regular observable, **LiveData is lifecycle-aware**, meaning it respects the lifecycle of other app components, such as activities, fragments, or services. This awareness **ensures LiveData only updates app component observers that are in an active lifecycle state**. 

Other components in your app can **monitor changes to objects** using this holder without creating explicit and rigid dependency paths between them. The LiveData component also respects the lifecycle state of your app's components—such as activities, fragments, and services—and includes **cleanup logic to prevent object leaking and excessive memory consumption**.