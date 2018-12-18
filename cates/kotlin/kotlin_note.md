# Kotlin

Write better Android apps faster with Kotlin - a modern statically typed programming language - and boost your productivity, increase your developer happiness.

Kotlin is a statically typed programming language that runs on the Java virtual machine and also can be compiled to JavaScript source code or use the LLVM compiler infrastructure. 

## Difference between Kotlin and Java in Android.

* Null safety which is [bilions dollar mistake](https://android.jlelse.eu/how-kotlin-addresses-the-billion-dollar-mistake-27609c82703e).
* Safe cast to avoid `ClassCastExeption`.
* Return `Unit` -> return `void` in Java.
* `Any!` (platform type) -> `Ojbect` in Java.
* NO [primitives](https://stackoverflow.com/questions/45420806/are-kotlin-data-types-built-off-primitive-or-non-primitive-java-data-types).
* Array are [invariant](https://kotlinlang.org/docs/reference/generics.html#variance) which represent by `Array` class.
* [Function type](https://kotlinlang.org/docs/reference/lambdas.html) replaced [Java SAM](https://stackoverflow.com/questions/17913409/what-is-a-sam-type-in-java)
* Don't have checked exception.
* NO static methods -> package-level method instead.
* Don't have non-private field.
* No wild card type `<?>` (but still have generic type `<T>`).
* Have extension function (Java do NOT have).
* Have string template. Eg: "$i" -- i=10 --> "10".
* Have primary constructor.
* Have [range](https://kotlinlang.org/docs/reference/ranges.html) expression.
* Have [data class](https://kotlinlang.org/docs/reference/data-classes.html) which is mainly for holding data.