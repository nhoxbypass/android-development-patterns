# Android Runtime

Android Runtime (ART) is the managed runtime used by apps and some system services on Android. **Replacing the predecessor Dalvik, ART performs the translation of the application's bytecode into native instructions** that are later executed by the device's runtime environment. 

ART is **written to run multiple virtual machines on low-memory devices** by executing DEX files, a bytecode format designed specially for Android that's optimized for minimal memory footprint. 

For devices running Android version 5.0 (API level 21) or higher, each app runs in its own process and with its own instance of the Android Runtime. 

ART also executes the Dex format and Dex bytecode specification.

Prior to Android version 5.0 (API level 21), Dalvik was the Android runtime. If your app runs well on ART, then it should work on Dalvik as well, but [the reverse may not be true](https://developer.android.com/guide/practices/verifying-apps-art).


## Android Runtime vs Dalvik Virtual Machine

| ART        | DVM          |
| -------------|---------------|
| Ahead-of-time (AOT) and just-in-time (JIT) compilation (Android 7.0) with code profiling | JIT compilation |
| Replace Dalvik from Android 5.0 (introduced from 4.4) |  |
| Faster, reduce start up time (because of AOT) |  |
| Take time and storage space to translate `.dex` file during install | No extra installation time and storage space |
| Use `dex2oat` to generate `.oat` native code from `.dex` bytecode | Use `dex opt` to convert `.dex` bytecode to `.odex` bytecode |
| Improve battery life |  |
| [Optimized garbage collection](https://source.android.com/devices/tech/dalvik/gc-debug) throughput, number of GC pauses is reduced from two to one compared to Dalvik |  |
| On Android 9 (API level 28) and higher, conversion of an app package's [DEX files to more compact machine code](https://developer.android.com/about/versions/pie/android-9.0#art-aot-dex) |  |
| Better debugging support (dedicated sampling profiler, detailed diagnostic exceptions and crash reporting, and ability to set watchpoints to monitor specific fields) |  |


## Just-in-time Compile vs Ahead-of-time Compile

| JIT        | AOT          |
| -------------|---------------|
| **Dynamic** translate **part** of bytecode to machine code and **cache in memory** when app run | Statically translate bytecode to machine code at **installation** time and **store in storage** |
| Small memory | One time event, code execute faster but need extra space and time |