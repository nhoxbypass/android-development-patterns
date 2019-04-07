# Fundamentals of Testing

## Iterative development workflow (TDD)

When developing a feature iteratively, you start by either writing a new test or by adding cases and assertions to an existing unit test. The test fails at first because the feature isn't implemented yet.

For each unit, you write a corresponding unit test. Your unit tests should nearly exhaust all possible interactions with the unit, including standard interactions, invalid inputs, and cases where resources aren't available.

This workflow called [Test-Driven Development (TDD)](https://www.youtube.com/watch?v=pK7W5npkhho&start=111).

![The two cycles associated with iterative, test-driven development](/resources/testing-workflow.png)


## Testing Pyramid

The Testing Pyramid, illustrates how your app should include the three categories of tests: small, medium, and large:

* Small tests: unit tests that you can run in isolation from production systems. They typically mock every major component and should run quickly on your machine.
* Medium tests: integration tests that sit in between small tests and large tests. They integrate several components, and they run on emulators or real devices.
* Large tests: integration and UI tests that run by completing a UI workflow. They ensure that key end-user tasks work as expected on emulators or real devices.

![Testing Pyramid](/resources/pyramid_2x.png)

Small tests are fast and focused, allowing you to address failures quickly, but they're also low-fidelity and self-contained, making it difficult to have confidence that a passing test allows your app to work.

Because of the different characteristics of each test category, you should include tests from each layer of the test pyramid with following split among the categories: **70 percent small, 20 percent medium, and 10 percent large**.


## Threads in tests

AndroidX Test makes use of the following threads:

* The main thread, also known as the "UI thread" or the "activity thread", where UI interactions and activity lifecycle events occur.
* The instrumentation thread, where most of your tests run. When your test suite begins, the `AndroidJUnitTest` class starts this thread.

If you need a test to execute on the main thread, annotate it using `@UiThreadTest`.


## Write small tests

As you add and change your code, make sure that these features behave as intended by creating and running unit tests against them. Although it's possible to evaluate units on a device or emulator, it's usually quicker and easier to test the units in your development environment, adding stubbed or mocked methods as needed to interact with the Android system.

#### Robolectric

If your app's testing environment requires unit tests to interact more extensively with the Android framework, you can use [Robolectric](http://robolectric.org/). This tool executes real Android framework code and fakes of native framework code on your local JVM. 

Robolectric tests nearly match the full fidelity of running tests on an Android device:

* All AndroidX Test libraries
* Android 4.1 (API level 16) and higher
* Android Gradle Plugin version 2.4 and higher
* Component lifecycles
* Event loops
* All resources


#### Interact with the Android environment

You can control and verify the elements of the Android framework with which your app interacts by running unit tests against a modified version of `android.jar`, which doesn't contain any code. You need to stub out every one of these interactions by using a mocking framework, such as [Mockito](https://site.mockito.org/).

If your code contains references to resources or complex interactions with the Android framework, you should use a different form of unit testing instead, such as Robolectric.


#### Instrumented unit tests

You can also run instrumented unit tests on a physical device or emulator, which doesn't involve any mocking or stubbing of the framework. Because this form of testing involves significantly **slower execution times than local unit tests**, however, it's best to rely on this method only when you want to evaluate your app's behavior against actual device hardware (to find device-related bugs).


## Write medium tests

Medium tests allow you to verify that the components behave properly with other components when run on an emulator or device. These tests are particularly important to create and run if some of your app's components depend on physical hardware.

Medium tests evaluate how your app coordinates multiple units, but they don't test the full app. Examples of medium tests include service tests, integration tests, and hermetic UI tests that simulate the behavior of external dependencies.


## Write large tests

Although it's important to test each layer and feature within your app in isolation, it's just as important to test common workflows and use cases that involve the complete stack, from the UI through business logic to the data layer.

If your app is small enough, you might need only one suite of large tests to evaluate your app's functionality as a whole. Otherwise, you should divide your large test suites by team ownership, functional verticals, or user goals.

The `AndroidJUnitRunner` class defines an instrumentation-based [JUnit](https://junit.org/junit4/) test runner that lets you run test classes on Android devices. The test runner facilitates loading your `test` package onto a device or emulator, running your tests, and reporting the results.

The AndroidJUnitRunner class also supports the following tools and frameworks from AndroidX Test:


#### JUnit4 Rules

AndroidX Test includes code for managing the lifecycles of key app components involved in your tests, such as activities and services. See: [JUnit4 Rules](testing.md#junit4-rules-with-androidx-test) guide.


#### Espresso

Espresso synchronizes asynchronous tasks while automating the following in-app interactions, both on your development machine and on real devices:

* Performing actions on `View`.
* Completing workflows that cross your app's process boundaries. Available only on Android 8.0 (API level 26) and higher.
* Assessing how users with accessibility needs can use your app.
* Locating and activating items within `RecyclerView` and `AdapterView`.
* Validating the state of outgoing intents.
* Verifying the structure of a DOM within `WebView`.
* Tracking long-running background operations within your app.


## Create more readable assertions

The Guava team provides a fluent assertions library called [Truth](https://github.com/google/truth). You can use this library as an alternative to JUnit-based assertions when constructing the validation step—or then step—of your tests.

Usually, you use Truth to express that a particular object has a specific property using phrases that contain the conditions you're testing, such as the following:

* `assertThat(object).hasFlags(FLAGS)`
* `assertThat(object).doesNotHaveFlags(FLAGS)`
* `assertThat(intent).hasData(URI)`
* `assertThat(extras).string(string_key).equals(EXPECTED)`