# Android Runtime

Android Runtime (ART) is the managed runtime used by applications and some system services on Android. Replacing the predecessor Dalvik, ART performs the translation of the application's bytecode into native instructions that are later executed by the device's runtime environment. 

ART also executes the Dex format and Dex bytecode specification.

## Android Runtime vs Dalvik Virtual Machine

| ART        | DVM          |
| -------------|---------------|
| Ahead-of-time compile together with JIT (Android 7.0) with code profiling | Just-in-time compile |
| Replace Dalvik from Android 5.0 (Introduced from 4.4) |  |
| Faster, reduce start up time (because of AOT) |  |
| Take time and storage space to translate `.dex` file during install | No extra installation time and storage space |
| Use `dex2oat` to generate `.oat` native code from `.dex` bytecode | Use `dex opt` to convert `.dex` bytecode to `.odex` bytecode |
| Improve battery life |  |
| [Improve GC](https://source.android.com/devices/tech/dalvik/gc-debug) throughput, number of GC pauses is reduced from two to one compared to Dalvik |  |


## Just-in-time Compile vs Ahead-of-time Compile

| JIT        | AOT          |
| -------------|---------------|
| **Dynamic** translate **part** of bytecode to machine code and **cache in memory** when app run | Statically translate bytecode to machine code at **installation** time and **store in storage** |
| Small memory | One time event, code execute faster but need extra space and time |