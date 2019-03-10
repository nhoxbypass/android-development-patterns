# When You Should Use Null in Kotlin

"Null has got a bad rap".  This realization of the danger of the null reference had dawned on the industry way before its inventor, Tony Hoare, had admitted in 2009 that null reference was his [“Billion Dollar Mistake”](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare).

People do these kind of mistakes all the time when programming in Java. It turns out that most of the time our API functions are **not supposed and are not expected by other developers to return null**. In corner cases, like the absence of data, it is a convention to return some "placeholder object” (empty collection, un-filled data object,..).

So when null does appear as a result of a function in some special case (performance optimization, uninitialized reference field,..) it often **catches other code off-guard, unprepared to deal with it**. Not only it is rare to be having to deal with nulls, but the code you have to write is long and verbose.

So yes, it’s to be avoided in most code, but the truth is that **the concept of null is not mistake, but Java’s type-system - which considers null to be a member of every type - is**. However, in more type-safe languages - like Kotlin - fixed this flaw by **incorporating the concept of null into the type system and and the compiler will tell you when you haven’t handled it appropriately**. So there are still some perfectly appropriate safe uses of null in Kotlin.


## Valid usages of null in Kotlin

#### When Representing “No Value”

```
 var user: User? = null
```

If you need a way to represent whether a value is initialized or whether it has no value, then null is appropriate in this case.


#### Consuming External Data Sources

If you need a way to represent whether a value is available or unavailable, then null is appropriate in this case as well.

Apps must consume external data sources that we **don’t have full control over**.  When operating in a non-hermetic environment, we run into cases where **some data sources are not reliable and may provide null content**.  Whether it is a REST API, or an Android System Service, representing data that may not be available as a nullable type is important because it **allows us to create code paths to handle all the cases**.


## Functional Purity

While it seems ideal to never have null in your code, it’s not always possible to avoid, and sometimes shouldn’t be avoided.

#### Using a Sealed Class to Avoid Using Null

```
sealed class UserWrapper {
  object UninitializedUser : UserWrapper()
  data class InitializedUser(val user: User): UserWrapper()
}
```

Creating a sealed class for the sole purpose of avoiding the use of null introduces **unnecessary boilerplate code**.

Don’t get me wrong, sealed classes are one of my favorite features of Kotlin though, and are great if you need to handle more scenarios than just initialized or uninitialized, but if that’s all you need, then using null is a valid approach.

#### Using a Nullable Field

```
val user : User?
```

Using a nullable field is much more concise in your definition than a sealed class.  It’s also more concise to use a `?.` operator instead of an `if` or `when` statement to process your sealed class.


## Offensive Programming

If you really don't care, you can run around with a bunch of `!!` (double bangs) in your code.  However, you are **living dangerously and your code will crash if the value is null**.  This may be what you are looking for though if you are trying to follow practices of Offensive Programming.

```
// This will crash if the user is null
handleNonNullUser(user!!)
```

[Defensive Programming](https://en.wikipedia.org/wiki/Defensive_programming) has us cover all possible code paths just in case it might occur.  When you do have these extra code paths, provide some fallback logic or warning logs.


## Ways of Handling Null

If you check once that the value is non-null, the Kotlin compiler is smart enough for you to write your code without providing any `!!`.  Then process the non-null `val` after it’s been checked once, to avoid multiple checks throughout my code. Here are some ways to do this:

#### Null Comparison

```
if (user != null) {
  handleNonNullUser(user)
} else {
  Log.w(TAG, "user was null")
  handleNullUser(user)
}
```

This is very Java-esque, but it does the trick.

#### Let

```
user?.let {
  // Work with non-null user
  handleNonNullUser(user)
}
```

I like using `let` because it allows us to write much more idiomatic Kotlin.  But it does not allow us to handle the else case (when the value is null)

#### Early Exit

```
fun handleUser(user : User?) {
  user ?: return //exit the function if user is null
  //Now the compiler knows user is non-null
}
```

If the value is null, you can exit the method immediately, otherwise you can operate on the non-null value (because the Kotlin compiler is that smart).

#### Immutable Shadows

Using immutable “variable shadows” a really cool way of providing null safety.  It will do an **Early Exit** if the value is null, otherwise it will assign the non-null value of the `var` to an immutable `val` allowing the compiler to know the value is non-null.

```
var user : User? = null

fun handleUser() {
  val user = user ?: return //Return if null, otherwise create immutable shadow
  //Work with a local, non-null variable named user
}
```

## Conclusion

There is no doubt that you will write better code if you try and stick with non-null and `val`s where you can. However, if you run into a scenario where using null is the right thing to do, then don’t fight it.

These techniques allow you to avoid having to use the `!!` operator which can lead to `NullPointerException`s.  The whole point of having a nullable type in Kotlin is to allow you to avoid NPEs at compile time, instead of them sneaking up on your at run time.

See more in Roman Elizarov’s post [“Null is your friend, not a mistake“](https://medium.com/@elizarov/null-is-your-friend-not-a-mistake-b63ff1751dd5).