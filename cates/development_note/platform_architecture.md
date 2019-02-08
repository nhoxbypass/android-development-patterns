# Platform Architecture

Android is an open source, Linux-based software stack created for a wide array of devices and form factors. The following diagram shows the major components of the Android platform.

**Android system architecture**

![Android system architecture](/resources/ape_fwk_all.png)

**Android software stack**

![Android software stack](/resources/android-stack_2x.png)


## The Linux Kernel

The foundation of the Android platform is the Linux kernel. For example, the Android Runtime (ART) relies on the Linux kernel for underlying functionalities such as threading and low-level memory management.

Using a Linux kernel allows Android to take advantage of [key security features](https://source.android.com/security/overview/kernel-security.html) (user-based permissions model, process isolation,..) and allows device manufacturers to develop hardware drivers for a well-known kernel.

Developing your device drivers is similar to developing a typical Linux device driver. **Android uses a version of the Linux kernel with a few special additions such as Low Memory Killer (a memory management system that is more aggressive in preserving memory), wake locks (a PowerManager system service), the Binder IPC driver, and other features important for a mobile embedded platform**.


## Hardware Abstraction Layer (HAL)

The [hardware abstraction layer (HAL)](https://source.android.com/devices/architecture/hal-types) **provides standard interfaces that expose device hardware capabilities to the higher-level Java API framework**. Using a HAL allows hardware vendors to implement functionality without affecting or modifying the higher level system.

The HAL consists of multiple library modules, each of which implements an interface for a specific type of hardware component, such as the [camera](https://source.android.com/devices/camera/index.html) or [bluetooth](https://source.android.com/devices/bluetooth.html) module. When a framework API makes a call to access device hardware, the Android system loads the library module for that hardware component.


## Android Runtime

Android Runtime (ART) is the managed runtime used by apps and some system services on Android. **Replacing the predecessor Dalvik, ART performs the translation of the app's bytecode into native instructions** that are later executed by the device's runtime environment. 

See [more](art.md)


## System Services

System services are modular, focused components such as **Window Manager**, **Search Service**, or **Notification Manager**. Functionality exposed by application framework APIs communicates with system services to access the underlying hardware. 

Android includes two groups of services: **system** (such as Window Manager and Notification Manager) and **media** (services involved in playing and recording media).


## Native C/C++ Libraries

Many core Android system components and services, such as ART and HAL, are built from native code that require native libraries written in C and C++. The Android platform provides Java framework APIs to expose the functionality of some of these native libraries to apps. For example, you can access [OpenGL ES](https://developer.android.com/guide/topics/graphics/opengl) through the Android framework’s [Java OpenGL API](https://developer.android.com/reference/android/opengl/package-summary.html) to add support for drawing and manipulating 2D and 3D graphics in your app.

If you are developing an app that requires C or C++ code, you can use the [Android NDK](https://developer.android.com/ndk/index.html) to access some of these [native platform libraries](https://developer.android.com/ndk/guides/stable_apis.html) directly from your native code.

## Binder IPC

The Binder Inter-Process Communication (IPC) mechanism **allows the application framework to cross process boundaries and call into the Android system services code**. This enables high level framework APIs to interact with Android system services. At the application framework level, this communication is hidden from the developer and things appear to "just work".


## Java API Framework

The entire feature-set of the Android OS is available to you through APIs written in the Java language. These APIs form the building blocks you need to create Android apps by simplifying the reuse of core, modular system components and services, which include the following:

* A rich and extensible [View system](https://developer.android.com/guide/topics/ui/overview.html) you can use to build an app’s UI, including lists, grids, text boxes, buttons, and even an embeddable web browser
* A [Resource Manager](https://developer.android.com/guide/topics/resources/overview.html), providing access to non-code resources such as localized strings, graphics, and layout files
* A [Notification Manager](https://developer.android.com/guide/topics/ui/notifiers/notifications.html) that enables all apps to display custom alerts in the status bar
* An [Activity Manager](https://developer.android.com/guide/components/activities.html) that manages the lifecycle of apps and provides a common navigation back stack
* [Content Providers](https://developer.android.com/guide/topics/providers/content-providers.html) that enable apps to access data from other apps, such as the Contacts app, or to share their own data

Developers have full access to the same [framework APIs](https://developer.android.com/reference/packages.html) that Android system apps use.


## System Apps

Android comes with a set of core apps for email, SMS messaging, calendars, internet browsing, contacts, and more. Apps included with the platform have no special status among the apps the user chooses to install. So a third-party app can become the user's default web browser, SMS messenger, or even the default keyboard (some exceptions apply, such as the system's Settings app).

The system apps function both as apps for users and to provide key capabilities that developers can access from their own app. For example, if your app would like to deliver an SMS message, you don't need to build that functionality yourself—you can instead invoke whichever SMS app is already installed to deliver a message to the recipient you specify.