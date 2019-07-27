# WorkManager

The **WorkManager API** makes it easy to schedule deferrable, asynchronous tasks that are expected to run even if the app exits or device restarts.

## Testing your workers

WorkManager provides a [work-testing](https://developer.android.com/jetpack/androidx/releases/work#declaring_dependencies) artifact which helps with unit testing of your workers for **Android Instrumentation tests**.

After WorkManager `v2.1` release there are now **two ways to test your workers**:

* [WorkManagerTestInitHelper](https://developer.android.com/reference/androidx/work/testing/WorkManagerTestInitHelper)
* [TestWorkerBuilder](https://developer.android.com/reference/androidx/work/testing/TestWorkerBuilder) and [TestListenableWorkerBuilder](https://developer.android.com/reference/androidx/work/testing/TestListenableWorkerBuilder)

#### WorkManagerTestInitHelper

Using `WorkManagerTestInitHelper` you can test your Worker classes by **simulating delays, meeting the constraints and the period requirements**. The `work-testing` also provides a `SynchronousExecutor` which makes it easier to write tests in a synchronous manner, without having to deal with multiple threads, locks or latches.

**Basic test:**

Testing this `SimpleWorker` in test mode is very similar to how you would use it in a real application.

```
@Test
@Throws(Exception::class)
fun testSimpleWorker() {
    // Define input data
    val input = workDataOf(KEY_1 to 1, KEY_2 to 2)

    // Create request
    val request = OneTimeWorkRequestBuilder<SimpleWorker>()
        .setInputData(input)
        .build()

    val workManager = WorkManager.getInstance()
    // Enqueue and wait for result. This also runs the Worker synchronously
    // because we are using a SynchronousExecutor.
    workManager.enqueue(request).result.get()

    // Get WorkInfo and outputData
    val workInfo = workManager.getWorkInfoById(request.id).get()
    val outputData = workInfo.outputData

    // Assert
    assertThat(workInfo.state, `is`(WorkInfo.State.SUCCEEDED))
    assertThat(outputData, `is`(expectedOutput))
}
```

**Test with constraints, delays and periodic work**

`WorkManagerTestInitHelper` provides you with an instance of [TestDriver](https://developer.android.com/reference/androidx/work/testing/TestDriver.html) which can be used to simulate `initialDelays`, conditions where `Constraints` are met, and intervals for `PeriodicWorkRequests`.

```
@Test
@Throws(Exception::class)
fun testWithInitialDelay() {
    // Define input data
    val input = workDataOf(KEY_1 to 1, KEY_2 to 2)

    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()

    // Create request
    val request = PeriodicWorkRequestBuilder<EchoWorker>(15, MINUTES)
        .setInputData(input)
        .setConstraints(constraints)
        .setInitialDelay(10, TimeUnit.SECONDS)
        .build()

    val workManager = WorkManager.getInstance()
    val testDriver = WorkManagerTestInitHelper.getTestDriver()

    // Enqueue and wait for result.
    workManager.enqueue(request).result.get()

    // Simulate delay, constrains condition,..
    testDriver.setInitialDelayMet(request.id)
    testDriver.setAllConstraintsMet(request.id)

    // Tells the testing framework the period delay is met
    testDriver.setPeriodDelayMet(request.id)
    
    // Get WorkInfo and outputData
    val workInfo = workManager.getWorkInfoById(request.id).get()
    val outputData = workInfo.outputData
    // Assert
    assertThat(workInfo.state, `is`(WorkInfo.State.SUCCEEDED))
    assertThat(outputData, `is`(input))
}
```

But, if you need to test a `CoroutineWorker`, `RxWorker` or `ListenableWorker`, using `WorkManagerTestInitHelper` has some **additional complexity** because you cannot rely on its `SynchronousExecutor`.

#### TestWorkerBuilder and TestListenableWorkerBuilder

To make testing these classes more straightforward, WorkManager `v2.1` includes a set of new `WorkRequest` builder:

* `TestWorkerBuilder` to invoke directly a `Worker` class
* `TestListenableWorkerBuilder` to invoke directly a `ListenableWorker` (`RxWorker` or `CoroutineWorker`)

These have the advantage that you can test any kind of Worker classes because in this case you’re running it directly.

**Testing Workers**. 

Let’s say we have a `Worker` which looks like this:

```
class SleepWorker(context: Context, parameters: WorkerParameters) : Worker(context, parameters) {

    companion object {
        const val SLEEP_DURATION = "SLEEP_DURATION"
    }

    override fun doWork(): Result {
        // Sleep on a background thread.
        val sleepDuration = inputData.getLong(SLEEP_DURATION, 1000)
        Thread.sleep(sleepDuration)
        return Result.success()
    }
}
```

To test this worker, you can now use `TestWorkerBuilder`. 

```
// Kotlin code can use the TestWorkerBuilder extension to
// build the Worker
@RunWith(AndroidJUnit4::class)
class SleepWorkerTest {
    private lateinit var context: Context
    private lateinit var executor: Executor

    @Before
    fun setUp() {
        context = ApplicationProvider.getApplicationContext()
        executor = Executors.newSingleThreadExecutor()
    }

    @Test
    fun testSleepWorker() {
        val worker = TestWorkerBuilder<SleepWorker>(
            context = context,
            executor = executor,
            inputData = workDataOf("SLEEP_DURATION" to 10000L)
        ).build()

        val result = worker.doWork()
        assertThat(result, `is`(Result.success()))
    }
}
```

**Testing ListenableWorker**

Suppose we need to test a `CoroutineWorker` which looks like this:

```
class SleepWorker(context: Context, parameters: WorkerParameters) : CoroutineWorker(context, parameters) {
    override suspend fun doWork(): Result {
        delay(1000) // milliseconds
        return Result.success()
    }
}
```

To test `SleepWorker`, we first create an instance of the worker using `TestListenableWorkerBuilder`. This builder can also be used to set tags, inputData, runAttemptCount, and so on. 

```
@RunWith(AndroidJUnit4::class)
class SleepWorkerTest {
    private lateinit var context: Context

    @Before
    fun setUp() {
        context = ApplicationProvider.getApplicationContext()
    }

    @Test
    fun testSleepWorker() {
        // Kotlin code can use the TestListenableWorkerBuilder extension to
        // build the ListenableWorker
        val worker = TestListenableWorkerBuilder<SleepWorker>(context).build()
        runBlocking {
            val result = worker.doWork()
            assertThat(result, `is`(Result.success()))
        }
    }
}
```

The main difference between `TestWorkerBuilder` and a `TestListenableWorkerBuilder` is, `TestWorkerBuilder` lets you specify the background `Executor` used to run the worker.

You can read more about these in the [documentation](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/testing) and you can see an example of a test using these new builders in the [Sunflower sample application](https://github.com/googlesamples/android-sunflower/blob/master/app/src/androidTest/java/com/google/samples/apps/sunflower/worker/SeedDatabaseWorkerTest.kt):

```
@RunWith(JUnit4::class)
class RefreshMainDataWorkTest {
  private lateinit var context: Context
  @Before
  fun setup() {
    context = ApplicationProvider.getApplicationContext()
  }
  @Test
  fun testRefreshMainDataWork() {
    // Get the ListenableWorker
    val worker = TestListenableWorkerBuilder<SeedDatabaseWorker>(context).build()
    // Start the work synchronously
    val result = worker.startWork().get()
    assertThat(result, `is`(Result.success()))
  }
}
```
