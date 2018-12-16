# Virtual Machine

## Why Android use virtual machine

* Code **isolate** from the core OS -> code contain malicious won't affect system directly. Make app and system stable and reliable.
* Cross platform compatibility, platform independent.


## Java Virtual Machine

Java code use JVM compiler (`javac`) to compile Java code (`.java`) into Java bytecode (`.class`). On runtime environment (Web, PC,..) Java interpreter convert bytecode into machine specs instruction set by: Start JVM -> Run (Start main thread, link, load `.class` files, execute machine code, unload classes, terminate main thread) -> Shutdown JVM.

* Modern JVM use just-in-time (JIT) compilation to compile the bytecode to machine code on the fly at runtime.
* It's possible to [compile Java code down to native code](https://stackoverflow.com/questions/2991799/can-i-compile-java-to-native-code) ahead of time, then run.
* It's also possible to intepret Java code directly.


## Dalvik Virtual Machine

DVM is Android virtual machine which is optimized for mobile (memory, battery life, performance,..)


## JVM vs DVM

| JVM        | DVM          |
| -------------|---------------|
| Not free | Free |
| Use `.class` bytecode | Convert `.class` bytecode to `.dex` bytecode using dex compiler |
| Stack based | Register based |