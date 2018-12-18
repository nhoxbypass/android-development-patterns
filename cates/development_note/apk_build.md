# How to build an APK

The APK is built using Java Compiler (javac), DEX compiler (dx), Android Asset Packaging Tool (aapt) and APK Packager.

1. `javac` compile `R.java` + `aidl` files + Android Java classes to Java `.class` bytecode. 
2. `dx` tool convert them to `.dex` [Dalvik EXecutable](https://stackoverflow.com/questions/7750448/what-are-dex-files-in-android) bytecode files. 
3. `aapt` pack resources into binary assets and put to `APK Packager`. 
4. `APK packager` use bytecode, resources package to generate APK and sign APK using keystore.