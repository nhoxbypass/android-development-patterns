# WorkManager

The **WorkManager API** makes it easy to schedule deferrable, asynchronous tasks that are expected to run even if the app exits or device restarts.


## Overview

Key features:

* Backwards compatible up to API 14. Uses `JobScheduler` on devices with API 23+. Uses a combination of `BroadcastReceiver` + `AlarmManager` on devices with API 14-22
* Add work constraints like network availability or charging status
* Chain tasks together
* Ensures task execution, even if the app or device restarts
* Adheres to power-saving features like Doze mode

WorkManager is intended for tasks that are deferrable—that is, not required to run immediately—and required to run reliably even if the app exits or the device restarts. For example:

* Sending logs or analytics to backend services
* Periodically syncing application data with a server

You can specify a 24 hours period, but because the work is executed **respecting Android’s battery optimization strategies**, you can only expect your worker to be executed around that time. You can then have an execution at 5:00AM the first day, 5:25AM the second day, 5:15AM the third, then 5:30AM the following one and so on.


## Create and execute background work

#### Creating background works

A task is defined using the [Worker](https://developer.android.com/reference/androidx/work/Worker) class. The `doWork()` (or `createWork()`) method is run synchronously on a background thread provided by WorkManager.

To create your background work, extend the `Worker` class and override the `doWork()` method.

```
class UploadWorker(appContext: Context, workerParams: WorkerParameters) : Worker(appContext, workerParams) {
    override fun doWork(): Result {
        // Do the work here. Ex: upload the images.

        uploadImages()

        // Indicate whether the task finished successfully with the Result
        return Result.success()
    }
}
```

To informs WorkManager whether the task needs to be retried at a later time, return `Result.retry()`

#### Configure how and when to run the task

While a `Worker` defines the unit of work, a `WorkRequest` defines **how and when work should be run**. 

Tasks may be one-off or periodic. For one-off work requests, use `OneTimeWorkRequest` and for periodic work `PeriodicTimeWorkRequest`.

```
val oneTimeWorkRequest = OneTimeWorkRequestBuilder<UploadWorker>()
        .build()

val periodicWorkRequest = PeriodicWorkRequestBuilder<UploadWorker>(15, MINUTES)
        .build()
```

Or you can customize work requests to handle common use cases:

* Provide device's conditions (network availabilty, charging,..) to work to indicate when it can run, using `Constraints`
* Guarantee a minimum delay in task execution
* Handle task retries and back-off
* Group tasks with tagging

#### Hand off your task to the system

Once you have defined your work request, you can now schedule it with `WorkManager#enqueue()` method.

```
WorkManager.getInstance().enqueue(uploadWorkRequest)
```

The exact time that the worker is going to be executed **depends on the constraints that are used in your work request and system optimizations (like Doze mode)**. WorkManager is designed to give the best possible behavior under these restrictions.


## Observing status of Work

As your work goes through its lifetime, it goes through various States.

* `BLOCKED`: if it has prerequisite work that hasn't finished yet.
* `ENQUEUED`: Work that is eligible to run as soon as its constraints and timing are met.
* `RUNNING`: When a worker is actively being executed.
* `SUCCEEDED`: A worker that has returned `Result.success()`. This is a terminal state; only `OneTimeWorkRequest`s may enter this state.
* `FAILED`: A worker that returned `Result.failure()`. This is also a terminal state; only `OneTimeWorkRequest`s may enter this state. All dependent work will also be marked as `FAILED` and will not run.
* `CANCELLED`: When you explicitly cancel a work request that hasn't already terminated. All dependent work will also be marked as CANCELLED and will not run.

If you need to check on the task status, you can get a `WorkInfo` object which includes the id of the work, its tags, its current `State`, and any output data.

```
WorkManager.getInstance().getWorkInfoByIdLiveData(uploadWorkRequest.id)
        .observe(lifecycleOwner, Observer { workInfo ->
            if (workInfo != null && workInfo.state == WorkInfo.State.SUCCEEDED) {
                displayMessage("Work finished!")
            }
        })
```

## Chaining Work together

Your app might need to run several tasks in a particular order. WorkManager allows you to create and enqueue a work sequence that specifies multiple tasks, and what order they should run in.

To create a chain of work, you can use `WorkManager.beginWith()` which return an instance of `WorkContinuation`. It can then be used to add dependent `OneTimeWorkRequest`s using `WorkContinuation.then()`.

```
WorkManager.getInstance()
    .beginWith(workA)
    .then(workB)    // then() returns a new WorkContinuation instance
    .then(workC)
    .enqueue();
```

If any task returns `Worker.WorkerResult.FAILURE`, the whole sequence ends.


## Canceling a Work

You can cancel a task after you enqueue it. To cancel the task, you need its work ID.

```
WorkManager.cancelWorkById(workRequest.id)
```


## Unique work sequences

We know that WorkManager guarantees the execution of your work even if your application is closed or the device is restarted. So, **enqueueing your worker at each start of your application can cause to add a new WorkRequest each time**. If you’re using a `OneTimeWorkRequest`, it’s probably not a big deal. But for periodic Work, **you can easily end up with multiple periodic work request being enqueued**.

Unique work is a concept that **guarantees that you only have one chain of work with a particular name at a time**.

You can enqueue your work request as a unique work by calling `WorkManager.enqueueUniqueWork()` or `WorkManager.enqueueUniquePeriodicWork()`. Or with chaining works, create the sequence with a call to `beginUniqueWork()` instead of `beginWith()`.


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

To make testing these classes more straightforward, WorkManager `v2.1` includes a set of new WorkRequest builder:

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
