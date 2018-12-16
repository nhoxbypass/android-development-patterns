# Android main components

## Context

* Abstract class to implement, support access to app resources, lauch activity/service/broadcast receiver. 
* Provide context of current state of app environment data.

## Activity

Main control unit. Provide UI for user to interact. Which included `Window` - carrier and `View`s - display.

## Service

Long-running operation in background (can be in UI thread or worker thread), no UI and user interact.

## BroadcastReceiver
  
Enable system to deliver events to the app and the OS (outside user's flow) or communicate between components.

## ContentProvider

Manage set of shared data in system. Share and access data of other app. Use `ContentResolver` to resolve `URI` to specific content provider.

## Intent

Message object to request action.

## Handler
  
Schedule `Message` and `Runable` to be execute at some point or enqueue action to perform in different thread. Main part of `HandlerThread` - a thread that has a looper.

## aidl

To generate IPC code. 