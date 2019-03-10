# Difference between Kotlin and Java in Android


## Kotlin solved Java problems

* Null safety which is [bilions dollar mistake](https://android.jlelse.eu/how-kotlin-addresses-the-billion-dollar-mistake-27609c82703e).
* Safe cast to avoid `ClassCastExeption`.
* Array are [invariant](https://kotlinlang.org/docs/reference/generics.html#variance) which represent by `Array` class.
* [Function type](https://kotlinlang.org/docs/reference/lambdas.html) replaced [Java SAM](https://stackoverflow.com/questions/17913409/what-is-a-sam-type-in-java)
* Have [data class](https://kotlinlang.org/docs/reference/data-classes.html) which is mainly for holding data.


## New language features

* Have extension function (Java do NOT have).
* Have string template. Eg: "$i" -- i=10 --> "10".
* Have primary constructor.
* Have [range](https://kotlinlang.org/docs/reference/ranges.html) expression.


## Comparison table

| Feature | Java | Kotlin |
| ------------- | ------------- | ------------- |
| Return type | void | Unit |
| Platform type | Object | Any! |
| Primitives | Yes | [No](https://stackoverflow.com/questions/45420806/are-kotlin-data-types-built-off-primitive-or-non-primitive-java-data-types) |
| Checked exception | Yes | No |
|  | Static method | package-level method |
| Non-private field | Yes | No |
| Wild card type `<?>` | Yes | No |
| Generic type `<T>` | Yes | Yes |