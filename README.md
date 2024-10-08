# Kotlin-Coroutines
## What are they?
- A Kotlin specific library built by JetBrains to provide a way to write asynchronous code in a more readable and synchronous manner. Typical use cases include processes such as performing background tasks, network requests and any long-running tasks including reading and writing to a database like room. 
- What’s interesting is that Coroutines perform such tasks `off the main thread` and as such coroutines ensure that we don’t block the main thread while performing such processes.

## Why Coroutines?
-	`Lightweight and easy to use` - You can run many coroutines on a single thread due to support for suspension, which doesn't block the thread where the coroutine is running. Compared to callbacks and thtreads.
-	`Structured concurrency` hence fewer `memory leaks` - New coroutines can only be launched in a specific CoroutineScope
  >Think of structured concurrency as a	design principle in Kotlin coroutines that ensures coroutines are properly scoped and managed within a defined hierarchy, making it easier to track their lifecycle and avoid issues like memory leaks or orphaned coroutines.
`In structured concurrency, every coroutine is tied to a specific scope, and this scope determines its lifecycle. When a coroutine is launched, it must belong to a parent scope (such as a CoroutineScope or viewModelScope in Android). If the parent scope is canceled, all the child coroutines are also canceled. This creates a structured, tree-like relationship between coroutines. That way we don’t have memory leaks.`
-	`Less resource-intensive` compared to threads | Resource efficiency
-	`Have Built-in cancellation` support – You can always cancel them when you wish to 
-	`Jetpack Integration` - Many Jetpack libraries include extensions that provide full coroutines support for example Room. 

## Coroutine Basics
### 1. Suspend function
- A suspend function is a function that can be paused and resumed at a later time. it is good to note that a suspend function can only be called from another suspend function or from a coroutine.
```kotlin
suspend fun fetchData(): String {
    delay(1000L) // Simulates a network request
    return "Data from server"
}
```
### 2. Coroutine builders 
- In-built coroutine functions that help you launch/create a coroutine. They include the following:-
  - `launch` is used to create a coroutine that does not return a result while
  - `async` is used to create a coroutine that returns a result. The result is a Deferred object and we can use the `await()` method to get the result.
  - `runBlocking` – blocks current thread until coroutines inside it have completed. Primarily used for testing and when running functions in main functions
<details>
  <summary> <b>Example Code</b> </summary>

```kotlin
  fun main() = runBlocking {
    // Launch a coroutine that does not return a result
    launch {
        delay(1000L)
        println("Task from launch")
    }

    // Launch a coroutine that returns a result using async
    val result = async {
        delay(2000L)
        "Result from async"
    }

    println("Result: ${result.await()}") // Waiting for the async result
    println("Task from runBlocking") // Main thread is blocked until coroutines finish
  }
```
</details>

### 3. Coroutine Jobs
- A Job is a coroutine instance with a life cycle and can be canceled. The same job can be used to check if the coroutine has been cancelled os is still active. We can also use the job to wait for the coroutine to finish. When does a jkob end? Well, a job ends when it’s completed or canceled.
- Usuually, the coroutin builders return a job object that we can use to check if the coroutine is still active or if it has been canceled.
- A job can be
  - A Normal `Job` - This kind is canceled when any of its children fail
  - A `SupervisorJob` - This kind is not canceled when any of its children fail.
### 4. Coroutine Scope 
- A `structured concurrency construct` that defines the lifecycle of a coroutine. Ideally, it keeps track of all the coroutines we create using the launch or async builders.
- It is responsible for knowing how long a coroutine will live. Without it, coroutines can not be launched.
- A `corotinescope` can be
  - `GlobalScope` - a scope that is not tied to any life cycle. It is not recommended to use this scope as it can lead to memory leaks.
    ```kotlin
    GlobalScope.launch {
        delay(500L)
        println("Task from GlobalScope (Not recommended)")
    }
    ```
  - `viewModelScope`- a scope that is tied to ViewModel. Coroutines here are cancelled when the viewModel is cancelled.
    ```kotlin
    viewModelScope.launch {
            // DoThis()
        }
    ```
  - `lifecycleScope` - a scope that is tied to an activity or fragment life cycle.
    ```kotlin
    lifeCycleScope.launch {
            // DoThis()
        }
    ```
  - `Custom` coroutinescope. We can also create our own custom scopes if we want to launch coroutines that will be canceled when a custom life cycle is destroyed.
  <details>
  <summary> <b>Example Code</b> </summary>
    
    ```kotlin
    private val customScope = CoroutineScope(Dispatchers.Default)

    fun startTask() {
        customScope.launch {
            delay(1000L)
            println("Task from custom scope completed")
        }
    }
    ```
  </details>
    
### 5. Coroutine Context
- This is a collection of many elements. Ideally, a `CoroutineContext` defines the behavior of our coroutines using elements such as the following:
  - `Job`: This manages the life cycle of the coroutine.
  - `CoroutineDispatcher`: This defines the thread on which the coroutine will run.
  - `CoroutineName`: This defines the name of the coroutine.
  - `CoroutineExceptionHandler`: This handles uncaught exceptions in the coroutine.
### 6. Dispatchers
- These specify which thread the coroutines will run on. We have the following dispatchers:
  - `Dispatchers.Main`: This is the main thread. It is used when we need to interact with the UI in our coroutines.
  - `Dispatchers.IO`: This is a thread pool that is optimized for IO tasks such as reading and writing to a database or making network requests.
  - `Dispatchers.Default`: This is a thread pool that is optimized for CPU-intensive tasks.
  - `Dispatchers.Unconfined`: This is a dispatcher that is not confined to any thread. It is used to create a coroutine that inherits the context of the parent coroutine.
  <details>
  <summary> <b>Example Code</b> </summary>
    
  ```kotlin
      // Dispatchers.Main
      launch(Dispatchers.Main) {
          println("Running on Main")
      }

      // Dispatchers.IO
      launch(Dispatchers.IO) {
          println("Running on IO")
          delay(500L)
          println("IO task completed")
      }
  
      // Dispatchers.Default 
      launch(Dispatchers.Default) {
          println("Running on Default")
          for (i in 1..5) {
              delay(200L)
              println("Processing $i on Default")
          }
      }
  
      // Dispatchers.Unconfined
      launch(Dispatchers.Unconfined) {
          println("Running on Unconfined")
          delay(100L)
      }
  ```
  </details>
  
### 7. withContext()
- `withContext()` function is a coroutine suspend fucntion that helps you switch between different dispatchers.
- Ideally, you would do this inside a coroutine
<details>
  <summary> <b>Example Code</b> </summary>

  ```kotlin
launch(Dispatchers.Main) {
        println("Running on Main thread")
        // Switch to IO dispatcher using withContext for a network call or database operation
        val result = withContext(Dispatchers.IO) {
            println("Switching to IO thread")
            // Simulate network request
            delay(1000L)
            "Data from network"
        }
        // After completing the IO task, back on the Main thread
        println("Back to Main thread")
    }
```
</details>

## Practical Use Cases for Coroutines 
- Long Running tasks that may block the main thread
- Writing and Reading from a database such as room
- Fetching data from an API | Network calls 
- Running background tasks that require switching between threads

## Coroutines in Practice
[x] [Fetching data from the internet - Retrofit](https://medium.com/@samueljumawrites/fetching-data-from-an-api-using-retrofit-in-your-android-app-5ceddd1030b8)  
[x] [Fetching data from the internet - Ktor](https://medium.com/@samueljumawrites/fetching-data-from-an-api-using-ktor-in-your-android-app-4c914cfb4093) 

