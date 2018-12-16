## Terminology

#### Nested class

| Static        | Non-static (inner class)    |
| ------------- | --------------- |
| Behave like top-level class but nested inside another top-level class for package convenience |  |
| Respect encapsulation of outer class field & method | Can use outer class field & method even they have private access |
| Can have static field & method | Can NOT have static field & method |
| Can live without instance of outer class (but in same package) | Can only live with instance of outer class |
|  | Local class, anonymous class |
|  | Have implicit reference to outer class -> cause memory leaks if live longer than instance of outer class |


#### Abstract Class vs Interface

| Abstract Class        | Interface          |
| -------------|---------------|
| `extends` | `implements` |
| Only need to provide abstract methods in subclass | Provide **all** methods declared in interface |
| Have constructor | No constructor |
| Cannot init but still be a ordinary Java class | Different |
| Can only extends 1 abstract class | Can implement multiple interface |
| Have default implementation of methods | Have default implement in Java |


#### HashTable

* key -> value look up.
* `int hashCode()` function convert object to an int, then use this int to map to **index** of array.
* No guarantee provide unique int for unique object (to save memory). That can lead to collision.
* Unique key. Multiple key for a single value. 