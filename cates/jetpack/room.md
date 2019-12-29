# Room

## Overview

[Room](https://developer.android.com/topic/libraries/architecture/room) An a SQLite object mapping library. Use it to easily convert SQLite table data to Java objects and avoid boilerplate code.

`Room` is an **object-mapping library that provides local data persistence with minimal boilerplate code by abstracting away some of the underlying implementation details of working with raw SQLite tables and queries**. It also allows you to observe changes to the database's data, including collections and join queries, exposing such changes using `LiveData` objects. It even **explicitly defines execution constraints that address common threading issues, such as accessing storage on the main thread**.

**At compile time, it validates each query** against your data schema, so broken SQL queries result in compile-time errors instead of runtime failures.

`Room` can return `RxJava`, `Flowable` and `LiveData` observables.

If your app already uses another persistence solution, such as a SQLite object-relational mapping (ORM), you don't need to replace your existing solution with `Room`. However, if you're writing a new app or refactoring an existing app - because `Room` takes care of these concerns for you - we highly recommend using `Room` to persist your app's data.