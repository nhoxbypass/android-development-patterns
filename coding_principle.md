## Coding Principles

#### SOLID

* **Single responsibility**
  A class should have **only one** reason to change, only do the task which it has been designed.
  To reduce coupling, code easier to read and maintain.

* **Open closed**
  Class, module should be **open** for extension but **close** for modification.
  Help easy to reuse, extend and implement new feature without need to modify old code.

* **Liskov Substitution**
  Object must be replacable by instance of their sub-type without breaking correct.
  Help reuse code, class hierachy easy to understand.

* **Interface Segregation**
  A class should not implememnt interface that it does NOT use. We split a general interface to separate & specific interface.
  Help easy to refactor.

* **Dependency inversion**
  High-level module should NOT depend on low-level module but should depend on abstractions (interface, abstract class). Use dependency injection.
  Help reduce coupling.


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