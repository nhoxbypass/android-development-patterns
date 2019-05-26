# Testing terminology

## Test coverage

The percentage of your code that is executed by your tests. If you have 100 lines of code, and your tests run through 80 of them, then you have 80% coverage.


## Test Driven Development (TDD)

A school of programming thought that says instead of writing your feature code first, you write your tests first. Then you write your feature code with the goal of passing your tests. You can learn more about it [here](https://www.youtube.com/watch?v=pK7W5npkhho).


## Test doubles

**A test double is an object that can stand in for a real object in a test**, such as a networking class, when testing. You can swap in a fake networking class to provide speed and determinism at the expense of fidelity. 

These are sometimes all commonly referred to as “mocks”, but it's important to distinguish between the different types of test doubles since they all have different uses. **The most common types of test doubles are stubs, mocks, fakes and spies**.

#### Stubs

**A stub has no logic, and only returns what you tell it to return**. Stubs can be used when you need an object to return specific values in order to get your code under test into a certain state. While it's usually easy to write stubs by hand, using a mocking framework is often a convenient way to reduce boilerplate.

#### Mocks

**A mock has expectations about the way it should be called, and a test should fail if it’s not called that way**. Mocks are used to test interactions between objects, and are useful in cases where there are no other visible state changes or return results that you can verify (e.g. if your code reads from disk and you want to ensure that it doesn't do more than one disk read, you can use a mock to verify that the method that does the read is only called once).

#### Fakes

**A fake doesn’t use a mocking framework: it’s a lightweight implementation of an API that behaves like the real implementation, but isn't suitable for production (e.g. an in-memory database)**. Fakes can be used when you can't use a real implementation in your test (e.g. if the real implementation is too slow or it talks over the network). You shouldn't need to write your own fakes often since fakes should usually be created and maintained by the person or team that owns the real implementation.