# Terminology

List of clarification about programming terms, definitions, commands, and glossary. Comparison between similar terminologies.

## General Java terms

| Term        | Exampple          |  Exampple          |
| -------------|---------------|---------------|
| Market annotation | `@Test` | Annotation with no parameter, simply "marks" the annotated element |
| Market interface | Serializable | An interface that contains no method declarations but merely "marks" a class that implements it as having some property. Marker interface type allows you to catch errors at compile time (compare to runtime of marker annotation |


## Serializable vs Parcelable

| Serializable        | Parcelable          |
| -------------|---------------|
| Only a marker interface | Provide skeleton to implement |
| Serialize object into stream of byte (`writeObject()`) | Write to `Parcel` container |
| Deserialize use no-arg constructor (`readObject()`) | Use `Parcelable.CREATOR` to generate instance and read data from `Parcel` |
| Use [Java reflection](https://stackoverflow.com/a/8586724/5282585) and use `serialVersionUid` to detect and build object instance | Use `IBinder`, high IPC transport |
| Slower | Faster (x10) |
| Create alot of temp object -> cause GC run many times |  |
| Easy to apply | Need effort to implement |


## Nested class

| Static        | Non-static (inner class)    |
| ------------- | --------------- |
| Behave like top-level class but nested inside another top-level class for package convenience |  |
| Respect encapsulation of outer class field & method | Can use outer class field & method even they have private access |
| Can have static field & method | Can NOT have static field & method |
| Can live without instance of outer class (but in same package) | Can only live with instance of outer class |
|  | Local class, anonymous class |
|  | Have implicit reference to outer class -> cause memory leaks if live longer than instance of outer class |


## Abstract Class vs Interface

| Abstract Class        | Interface          |
| -------------|---------------|
| `extends` | `implements` |
| Only need to provide abstract methods in subclass | Provide **all** methods declared in interface |
| Have constructor | No constructor |
| Cannot init but still be a ordinary Java class | Different |
| Can only extends 1 abstract class | Can implement multiple interface |
| Have default implementation of methods | Have default implement in Java |


## HashTable

* key -> value look up.
* `int hashCode()` function convert object to an int, then use this int to map to **index** of array.
* No guarantee provide unique int for unique object (to save memory). That can lead to collision.
* Unique key. Multiple key for a single value.


## Enumerator vs Iterator

| Enumerator        | Iterator          |
| -------------|---------------|
|  | have `remove()` |
| From Java 1.0 | From Java 1.2 |
| Legacy to support `HashTable`, `Stack`, `Vector` | New |
| Fail safe | Fail fast -> thread safe, secure |


## HashTable vs HashMap

| HashTable        | HashMap          |
| -------------|---------------|
| Legacy | New |
| NonNull | New |
| Synchronize | Non-sync | 


## Vector vs ArrayList

| Vector        | ArrayList          |
| -------------|---------------|
| Legacy/Deprecated | New |
| Synchronize invidual operation, not the whole sequence operations | Non-sync |
| Data growth | Data expand |


## Type inference

Type inference refers to the automatic detection of the data type of an expression in a programming language. It is a feature present in some strongly statically typed languages. 


## Java Generics

| Term        | Exampple          |
| -------------|---------------|
| Parameterized type | `List<String>` |
| Actual type parameter type | `String` |
| Generic type | `List<E>` |
| Formal type parameter | `E` |
| Unbounded wildcard type | `List<?>` |
| Raw type | `List` |
| Bounded type parameter | `<E extends Number>` |
| Recursive type bound | `<T extends Comparable<T>>` |
| Bounded wildcard type	 | `List<? extends Number>` |
| Generic method	| `static <E> List<E> asList(E[] a)` |
| Type token | `String.class`	|
