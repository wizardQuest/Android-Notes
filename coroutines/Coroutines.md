# Kotlin Coroutines on Android - Interview Preparation Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [Coroutine Scope](#coroutine-scope)
4. [Dispatchers](#dispatchers)
5. [Suspend Functions](#suspend-functions)
6. [Advanced Coroutines Concepts](#advanced-coroutines-concepts)
7. [Main-Safety](#main-safety)
8. [Exception Handling](#exception-handling)
9. [Cancellation](#cancellation)
10. [Testing Coroutines](#testing-coroutines)
11. [Best Practices](#best-practices)
12. [Common Patterns](#common-patterns)

---

## Introduction

**Coroutines** are a concurrency design pattern that you can use on Android to simplify code that executes asynchronously. They help manage long-running tasks that might otherwise block the main thread and cause your app to become unresponsive.

### Why Coroutines?

- **Lightweight**: Coroutines can run on a thread but are not bound to it. They can suspend their execution without blocking the thread.
- **Fewer memory leaks**: Use structured concurrency to run operations within a scope.
- **Built-in cancellation support**: Cancellation propagates automatically through the running coroutine hierarchy.
- **Jetpack integration**: Many Jetpack libraries include extensions that provide full coroutines support.

---

## Key Concepts

### Coroutine
A coroutine is an instance of suspendable computation. It is conceptually similar to a thread, in that it takes a block of code to run that works concurrently with the rest of the code. However, a coroutine is not bound to any particular thread. It may suspend its execution in one thread and resume in another one.

### Suspension
When a coroutine is suspended, it means the coroutine can pause execution at a suspension point and resume later. This is done without blocking the thread.

### Coroutine Builder
Functions that create coroutines:
- `launch`: Launches a new coroutine and returns a `Job`
- `async`: Launches a new coroutine and returns a `Deferred` (a non-blocking cancellable future)
- `runBlocking`: Blocks the current thread until the coroutine completes (mainly for testing)

---

## Coroutine Scope

A **coroutine scope** defines the scope within which new coroutines can be launched. All coroutines must be launched in a scope.

### Types of Scopes

#### 1. **GlobalScope**
- Lives for the entire application lifetime
- **⚠️ Avoid using GlobalScope** - it can lead to memory leaks
- Not bound to any lifecycle

```kotlin
GlobalScope.launch {
    // Coroutine code
}
```

#### 2. **viewModelScope**
- Bound to the ViewModel lifecycle
- Automatically cancelled when ViewModel is cleared
- Recommended for ViewModel operations

```kotlin
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // Coroutine code
        }
    }
}
```

#### 3. **lifecycleScope**
- Bound to LifecycleOwner (Activity, Fragment)
- Automatically cancelled when lifecycle is destroyed
- Available in `lifecycle-runtime-ktx` library

```kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            // Coroutine code
        }
    }
}
```

#### 4. **CoroutineScope**
- Custom scope you can create
- Must manually manage cancellation

```kotlin
val scope = CoroutineScope(Dispatchers.Main + SupervisorJob())
scope.launch {
    // Coroutine code
}
```

### Structured Concurrency
Coroutines follow a principle of structured concurrency which ensures that:
- When a scope cancels, all of its coroutines cancel
- When a coroutine fails, its parent scope is notified
- Operations are organized in a hierarchy

---

## Dispatchers

**Dispatchers** determine which thread pool a coroutine runs on. They tell the coroutine where to execute.

### Types of Dispatchers

#### 1. **Dispatchers.Main**
- Runs on the main/UI thread
- Use for UI operations, light work, and calling suspend functions
- Default for coroutines launched in UI scope

```kotlin
viewModelScope.launch(Dispatchers.Main) {
    // Update UI
    textView.text = "Updated"
}
```

#### 2. **Dispatchers.Main.immediate**
- Executes immediately on the main thread if already on main thread
- Otherwise dispatches to main thread
- Useful for immediate execution when already on main thread

#### 3. **Dispatchers.IO**
- Optimized for disk and network I/O operations
- Shares threads with a thread pool
- Use for database operations, networking, file I/O

```kotlin
viewModelScope.launch(Dispatchers.IO) {
    // Network call or file operation
    val data = repository.fetchData()
}
```

#### 4. **Dispatchers.Default**
- Optimized for CPU-intensive work
- Use for sorting, parsing, complex calculations
- Default dispatcher for coroutines that don't specify one

```kotlin
viewModelScope.launch(Dispatchers.Default) {
    // CPU-intensive work
    val result = performComplexCalculation()
}
```

#### 5. **Dispatchers.Unconfined**
- Starts execution immediately in the current call frame
- Resumes in whatever thread that the corresponding suspending function uses
- **⚠️ Use with caution** - mainly for testing

---

## Suspend Functions

A **suspend function** is a function that can be paused and resumed at a later time. Suspend functions can only be called from another suspend function or from a coroutine.

### Characteristics
- Marked with the `suspend` keyword
- Can call other suspend functions
- Can be suspended without blocking the thread
- Must be called from a coroutine or another suspend function

### Example

```kotlin
suspend fun fetchUserData(): User {
    // This can suspend execution
    return withContext(Dispatchers.IO) {
        // Network call
        api.getUser()
    }
}

// Usage
viewModelScope.launch {
    val user = fetchUserData() // Can suspend here
    updateUI(user)
}
```

### Sequential Execution

```kotlin
viewModelScope.launch {
    val user = fetchUser()        // Suspends until complete
    val posts = fetchPosts(user)  // Suspends until complete
    displayData(user, posts)      // Executes after both complete
}
```

### Parallel Execution

```kotlin
viewModelScope.launch {
    val userDeferred = async { fetchUser() }
    val postsDeferred = async { fetchPosts() }
    
    val user = userDeferred.await()  // Suspends until complete
    val posts = postsDeferred.await() // Suspends until complete
    displayData(user, posts)
}
```

---

## Advanced Coroutines Concepts

### coroutineScope Builder

The `coroutineScope` builder creates a scope and doesn't complete until all launched children complete. It's useful for parallel decomposition of work.

**Key Points:**
- Creates a scope that waits for all children to complete
- Automatically handles exceptions and propagates them
- Doesn't complete until all launched coroutines finish
- Used for parallel decomposition

#### Example: Parallel Decomposition

```kotlin
suspend fun fetchTwoDocs() = coroutineScope {
    val deferredOne = async { fetchDoc(1) }
    val deferredTwo = async { fetchDoc(2) }
    deferredOne.await()
    deferredTwo.await()
}
```

#### Using awaitAll()

```kotlin
suspend fun fetchTwoDocs() = coroutineScope {
    val deferreds = listOf(
        async { fetchDoc(1) },
        async { fetchDoc(2) }
    )
    deferreds.awaitAll() // Wait for all to complete
}
```

**Note:** Even if you don't call `awaitAll()`, `coroutineScope` doesn't resume until all children complete.

### Job

A `Job` is a handle to a coroutine. Each coroutine created with `launch` or `async` returns a `Job` instance that uniquely identifies the coroutine and manages its lifecycle.

```kotlin
class ExampleClass {
    fun exampleMethod() {
        // Handle to the coroutine
        val job = scope.launch {
            // New coroutine
        }

        if (someCondition) {
            // Cancel the coroutine
            job.cancel()
        }
    }
}
```

**Job States:**
- **New**: Just created, not started
- **Active**: Running (default state)
- **Completing**: Finishing its work
- **Completed**: Finished successfully
- **Cancelling**: Being cancelled
- **Cancelled**: Cancelled

### CoroutineContext

A `CoroutineContext` defines the behavior of a coroutine using these elements:

1. **Job**: Controls the lifecycle of the coroutine
2. **CoroutineDispatcher**: Dispatches work to the appropriate thread
3. **CoroutineName**: The name of the coroutine (useful for debugging)
4. **CoroutineExceptionHandler**: Handles uncaught exceptions

#### Context Inheritance

For new coroutines created within a scope:
- A new `Job` instance is assigned to the new coroutine
- Other `CoroutineContext` elements are inherited from the containing scope
- You can override inherited elements by passing a new `CoroutineContext` to `launch` or `async`

```kotlin
class ExampleClass {
    val scope = CoroutineScope(Job() + Dispatchers.Main)

    fun exampleMethod() {
        // Inherits Dispatchers.Main from scope
        val job1 = scope.launch {
            // New coroutine with default CoroutineName
        }

        // Overrides dispatcher and name
        val job2 = scope.launch(
            Dispatchers.Default + CoroutineName("BackgroundCoroutine")
        ) {
            // New coroutine with custom dispatcher and name
        }
    }
}
```

**Note:** Passing a `Job` to `launch` or `async` has no effect - a new `Job` instance is always assigned to a new coroutine.

### Creating Custom CoroutineScope

When you need to create your own scope to control the lifecycle of coroutines:

```kotlin
class ExampleClass {
    // Job and Dispatcher combined into CoroutineContext
    val scope = CoroutineScope(Job() + Dispatchers.Main)

    fun exampleMethod() {
        // Starts a new coroutine within the scope
        scope.launch {
            fetchDocs()
        }
    }

    fun cleanUp() {
        // Cancel the scope to cancel ongoing coroutines
        scope.cancel()
    }
}
```

**Important:**
- A cancelled scope cannot create more coroutines
- Call `scope.cancel()` only when the class controlling its lifecycle is being destroyed
- `viewModelScope` automatically cancels in `ViewModel.onCleared()`

### Parallel Decomposition

Parallel decomposition is breaking down a task into smaller concurrent tasks. Use `coroutineScope` to ensure all tasks complete before returning.

```kotlin
suspend fun loadUserData(userId: String) = coroutineScope {
    val userDeferred = async { fetchUser(userId) }
    val postsDeferred = async { fetchPosts(userId) }
    val friendsDeferred = async { fetchFriends(userId) }
    
    // All three run in parallel
    UserData(
        user = userDeferred.await(),
        posts = postsDeferred.await(),
        friends = friendsDeferred.await()
    )
}
```

---

## Main-Safety

A function is **main-safe** when it doesn't block UI updates on the main thread.

### Making Functions Main-Safe

Use `withContext()` to switch dispatchers within a suspend function:

```kotlin
class LoginRepository {
    suspend fun makeLoginRequest(jsonBody: String): Result<LoginResponse> {
        // Move execution to I/O dispatcher
        return withContext(Dispatchers.IO) {
            // Blocking network request code
            // This runs on I/O thread, not main thread
        }
    }
}
```

### Benefits
- The calling function can be called from the main thread
- The function handles thread switching internally
- UI remains responsive

### Example Pattern

```kotlin
// Repository - handles thread switching
class UserRepository {
    suspend fun getUser(id: String): User {
        return withContext(Dispatchers.IO) {
            // Network or database call
            api.getUser(id)
        }
    }
}

// ViewModel - can call from main thread
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    fun loadUser(id: String) {
        viewModelScope.launch {
            // This runs on main thread
            val user = repository.getUser(id) // Suspends, switches to IO
            // Resumes on main thread
            _user.value = user
        }
    }
}
```

### Testing Tip
For easier testing, inject `Dispatchers` into your Repository:

```kotlin
class UserRepository(
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun getUser(id: String): User {
        return withContext(ioDispatcher) {
            api.getUser(id)
        }
    }
}
```

---

## Exception Handling

### Try-Catch Block

Handle exceptions in coroutines using standard Kotlin exception handling:

```kotlin
viewModelScope.launch {
    val result = try {
        repository.makeLoginRequest(jsonBody)
    } catch (e: Exception) {
        Result.Error(Exception("Network request failed"))
    }
    
    when (result) {
        is Result.Success -> // Handle success
        is Result.Error -> // Handle error
    }
}
```

### Exception Propagation

- Exceptions in `launch` are thrown immediately
- Exceptions in `async` are deferred until `await()` is called
- Uncaught exceptions cancel the parent scope (unless using `SupervisorJob`)

### SupervisorJob

Use `SupervisorJob` when you want a child coroutine's failure not to cancel other children:

```kotlin
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)
scope.launch {
    // Child 1 - if this fails, child 2 continues
}
scope.launch {
    // Child 2 - if this fails, child 1 continues
}
```

### CoroutineExceptionHandler

Handle uncaught exceptions globally:

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e("Coroutine", "Uncaught exception", exception)
}

viewModelScope.launch(handler) {
    // Coroutine code
}
```

---

## Cancellation

Cancellation in coroutines is **cooperative**. A coroutine must check for cancellation and respond to it appropriately. Cancellation propagates through the coroutine hierarchy.

### How Cancellation Works

When you cancel a coroutine:
1. It sets the `Job` to the "cancelling" state
2. The coroutine should check `isActive` or handle `CancellationException`
3. All children of the cancelled coroutine are also cancelled
4. The coroutine cannot be used to start new coroutines

### Basic Cancellation

```kotlin
val job = viewModelScope.launch {
    // Long-running work
}

// Cancel the coroutine
job.cancel()
```

### Cooperative Cancellation

Coroutines must check for cancellation. Most suspend functions (like `delay`, `withContext`) are cancellable and throw `CancellationException` when cancelled.

```kotlin
viewModelScope.launch {
    while (isActive) { // Check if coroutine is still active
        // Do work
        delay(1000)
    }
    // Cleanup code here
}
```

### Checking isActive

Always check `isActive` in long-running loops:

```kotlin
viewModelScope.launch {
    var nextPrintTime = 0L
    while (isActive) {
        val currentTime = System.currentTimeMillis()
        if (currentTime >= nextPrintTime) {
            println("Job: I'm working...")
            nextPrintTime = currentTime + 500L
        }
    }
    println("Job: I'm done")
}
```

### CancellationException

When a coroutine is cancelled, it throws `CancellationException`. This exception should be handled differently from other exceptions:

```kotlin
viewModelScope.launch {
    try {
        work()
    } catch (e: CancellationException) {
        // Cancellation is normal - just rethrow
        throw e
    } catch (e: Exception) {
        // Handle other exceptions
        handleError(e)
    }
}
```

**Important:** `CancellationException` should be rethrown - it's used by the coroutines framework for control flow.

### Non-Cancellable Operations

If you need to perform cleanup that shouldn't be cancelled, use `NonCancellable`:

```kotlin
viewModelScope.launch {
    try {
        work()
    } finally {
        // This block will execute even if coroutine is cancelled
        withContext(NonCancellable) {
            cleanup() // This won't be cancelled
        }
    }
}
```

### Cancelling with Reason

You can provide a reason when cancelling:

```kotlin
job.cancel("User requested cancellation")
job.cancel(CancellationException("Custom reason"))
```

### Timeout

Use `withTimeout` to automatically cancel after a duration:

```kotlin
viewModelScope.launch {
    try {
        val result = withTimeout(5000) { // 5 seconds
            fetchData()
        }
    } catch (e: TimeoutCancellationException) {
        // Handle timeout
    }
}
```

### Cancellation in Loops

For loops that don't naturally suspend, check `isActive`:

```kotlin
viewModelScope.launch {
    for (i in 0..1000) {
        if (!isActive) break // Check cancellation
        // Do work
        heavyComputation(i)
    }
}
```

### Making Functions Cancellable

To make a function cancellable, it should:
1. Check `isActive` periodically
2. Call other cancellable suspend functions
3. Throw `CancellationException` when cancelled

```kotlin
suspend fun cancellableWork() {
    for (i in 0..100) {
        ensureActive() // Throws CancellationException if cancelled
        // Or check: if (!isActive) return
        doWork(i)
    }
}
```

### Cancellation Propagation

Cancellation propagates downward through the coroutine hierarchy:

```kotlin
val parentJob = Job()
val scope = CoroutineScope(parentJob + Dispatchers.Main)

scope.launch {
    launch { // Child 1
        delay(1000)
    }
    launch { // Child 2
        delay(1000)
    }
}

// Cancelling parent cancels all children
parentJob.cancel()
```

### SupervisorJob and Cancellation

With `SupervisorJob`, child failures don't cancel siblings, but parent cancellation still cancels all children:

```kotlin
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

scope.launch {
    launch { // Child 1 - if this fails, Child 2 continues
        throw Exception("Error")
    }
    launch { // Child 2 - continues even if Child 1 fails
        delay(1000)
    }
}
// But if scope is cancelled, both children are cancelled
```

### Key Cancellation Rules

1. **Cancellation is cooperative** - coroutines must check for cancellation
2. **CancellationException is special** - rethrow it, don't catch it as a regular exception
3. **Cancellation propagates** - cancelling a parent cancels all children
4. **Use `isActive` or `ensureActive()`** - check cancellation in loops
5. **Most suspend functions are cancellable** - `delay`, `withContext`, etc.

---

## Testing Coroutines

Testing coroutines requires special handling because they involve asynchronous execution, timing, and thread switching. The `kotlinx-coroutines-test` library provides tools for testing.

### Dependencies

Add to your `build.gradle`:

```gradle
dependencies {
    testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:$coroutines_version"
}
```

### runTest

`runTest` is the main coroutine builder for testing. It provides:
- Virtual time control
- Automatic skipping of delays
- Proper exception handling

```kotlin
@Test
fun testDataLoading() = runTest {
    val repository = TestRepository()
    val result = repository.loadData()
    assertEquals(expectedData, result)
}
```

### TestDispatchers

Test dispatchers allow you to control when coroutines execute:

#### StandardTestDispatcher

- Queues coroutines and executes them when explicitly advanced
- Gives you full control over execution order
- Use `advanceUntilIdle()` to run all queued coroutines

```kotlin
@Test
fun testWithStandardTestDispatcher() = runTest {
    val testDispatcher = StandardTestDispatcher()
    val repository = Repository(testDispatcher)
    
    val job = launch(testDispatcher) {
        repository.loadData()
    }
    
    // Advance time to execute coroutines
    testDispatcher.scheduler.advanceUntilIdle()
    
    // Assertions
    assertTrue(job.isCompleted)
}
```

#### UnconfinedTestDispatcher

- Runs coroutines eagerly in a blocking manner
- Simpler but less control over execution order
- Good for simple tests

```kotlin
@Test
fun testWithUnconfinedTestDispatcher() = runTest {
    val testDispatcher = UnconfinedTestDispatcher()
    val repository = Repository(testDispatcher)
    
    val result = repository.loadData()
    
    // Assertions - coroutines already executed
    assertEquals(expectedData, result)
}
```

### Injecting Test Dispatchers

For testability, inject dispatchers into your classes:

```kotlin
class UserRepository(
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun getUser(id: String): User = withContext(ioDispatcher) {
        // I/O operation
    }
}

// In tests
@Test
fun testGetUser() = runTest {
    val testDispatcher = StandardTestDispatcher()
    val repository = UserRepository(testDispatcher)
    
    val user = repository.getUser("123")
    
    assertEquals(expectedUser, user)
}
```

### Testing with Delays

`runTest` automatically skips delays, but you can control virtual time:

```kotlin
@Test
fun testWithDelay() = runTest {
    val testDispatcher = StandardTestDispatcher()
    
    var executed = false
    launch(testDispatcher) {
        delay(1000)
        executed = true
    }
    
    // Advance time by 1000ms
    testDispatcher.scheduler.advanceTimeBy(1000)
    
    assertTrue(executed)
}
```

### Testing Exceptions

Test exception handling in coroutines:

```kotlin
@Test
fun testExceptionHandling() = runTest {
    val repository = Repository()
    
    assertThrows<NetworkException> {
        repository.fetchData() // Throws NetworkException
    }
}
```

### Testing Cancellation

Test that coroutines handle cancellation properly:

```kotlin
@Test
fun testCancellation() = runTest {
    val job = launch {
        try {
            delay(1000)
        } catch (e: CancellationException) {
            // Cleanup
            cleanup()
            throw e
        }
    }
    
    job.cancel()
    job.join()
    
    // Verify cleanup was called
    assertTrue(cleanupCalled)
}
```

### Testing Flows

Test Flow emissions:

```kotlin
@Test
fun testFlow() = runTest {
    val flow = repository.getUserFlow()
    
    val results = flow.take(3).toList()
    
    assertEquals(3, results.size)
}
```

### Testing viewModelScope

For testing ViewModels, you can replace `viewModelScope`:

```kotlin
class MyViewModel(
    private val repository: Repository,
    private val viewModelScope: CoroutineScope = CoroutineScope(SupervisorJob())
) : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // Load data
        }
    }
    
    override fun onCleared() {
        super.onCleared()
        viewModelScope.cancel()
    }
}

// In test
@Test
fun testViewModel() = runTest {
    val testScope = this
    val viewModel = MyViewModel(repository, testScope)
    
    viewModel.loadData()
    advanceUntilIdle()
    
    // Assertions
}
```

### Best Practices for Testing

1. **Use `runTest`** for all coroutine tests
2. **Inject dispatchers** for testability
3. **Use `StandardTestDispatcher`** when you need control
4. **Use `UnconfinedTestDispatcher`** for simple tests
5. **Test cancellation** behavior
6. **Test exception handling** properly
7. **Use `advanceUntilIdle()`** to ensure all coroutines complete

### Common Testing Patterns

#### Pattern 1: Testing Suspend Functions

```kotlin
@Test
fun testSuspendFunction() = runTest {
    val result = repository.fetchData()
    assertEquals(expected, result)
}
```

#### Pattern 2: Testing Parallel Execution

```kotlin
@Test
fun testParallelExecution() = runTest {
    val testDispatcher = StandardTestDispatcher()
    
    val deferred1 = async(testDispatcher) { fetchData1() }
    val deferred2 = async(testDispatcher) { fetchData2() }
    
    testDispatcher.scheduler.advanceUntilIdle()
    
    val result1 = deferred1.await()
    val result2 = deferred2.await()
    
    // Assertions
}
```

#### Pattern 3: Testing Timeout

```kotlin
@Test
fun testTimeout() = runTest {
    val testDispatcher = StandardTestDispatcher()
    
    assertThrows<TimeoutCancellationException> {
        withTimeout(1000) {
            // Operation that takes longer than 1000ms
            delay(2000)
        }
    }
}
```

---

## Best Practices

### 1. **Inject Dispatchers for Testing**

Always inject dispatchers to make your code testable:

```kotlin
class Repository(
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun fetchData() = withContext(ioDispatcher) {
        // I/O operation
    }
}
```

**Benefits:**
- Easy to test with `TestDispatcher`
- Flexible for different environments
- Better separation of concerns

### 2. **Use Appropriate Scopes**

Choose the right scope for your use case:

- ✅ **`viewModelScope`** in ViewModels - automatically cancelled when ViewModel is cleared
- ✅ **`lifecycleScope`** in Activities/Fragments - tied to lifecycle
- ✅ **Custom scopes** for specific use cases (with proper cleanup)
- ❌ **Avoid `GlobalScope`** in production code - can cause memory leaks

```kotlin
// ✅ Good
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // Work
        }
    }
}

// ❌ Bad
class MyViewModel : ViewModel() {
    fun loadData() {
        GlobalScope.launch {
            // Work - not tied to ViewModel lifecycle
        }
    }
}
```

### 3. **Make Functions Main-Safe**

Functions should be safe to call from the main thread:

- Move thread switching to the data layer (Repository)
- Keep ViewModel code on main thread
- Use `withContext()` for dispatcher switching

```kotlin
// ✅ Good - main-safe
class Repository {
    suspend fun getUser(id: String): User = withContext(Dispatchers.IO) {
        api.getUser(id)
    }
}

class ViewModel {
    fun loadUser(id: String) {
        viewModelScope.launch {
            val user = repository.getUser(id) // Safe to call from main
            _user.value = user
        }
    }
}
```

### 4. **Handle Exceptions Properly**

Proper exception handling prevents crashes:

- Use try-catch for expected exceptions
- Use `CoroutineExceptionHandler` for uncaught exceptions
- Consider using `SupervisorJob` when children should be independent
- Always rethrow `CancellationException`

```kotlin
// ✅ Good
viewModelScope.launch {
    try {
        val data = repository.fetchData()
        _uiState.value = UiState.Success(data)
    } catch (e: CancellationException) {
        throw e // Always rethrow
    } catch (e: Exception) {
        _uiState.value = UiState.Error(e)
    }
}
```

### 5. **Avoid Blocking the Main Thread**

Never block the main thread:

- ❌ Never use `runBlocking` in production code
- ✅ Use suspend functions for async operations
- ✅ Switch to appropriate dispatchers for heavy work
- ✅ Use `withContext()` to switch threads

```kotlin
// ❌ Bad - blocks main thread
fun loadData() {
    runBlocking {
        val data = api.getData() // Blocks!
    }
}

// ✅ Good - non-blocking
suspend fun loadData() = withContext(Dispatchers.IO) {
    api.getData() // Doesn't block main thread
}
```

### 6. **Cancel Coroutines Properly**

Proper cancellation prevents resource leaks:

- Use scoped coroutines (viewModelScope, lifecycleScope)
- Manually cancel custom scopes when done
- Check `isActive` in long-running loops
- Handle `CancellationException` properly

```kotlin
// ✅ Good
viewModelScope.launch {
    while (isActive) {
        // Long-running work
        delay(1000)
    }
    // Cleanup code
}

// ✅ Good - custom scope
class MyClass {
    private val scope = CoroutineScope(Job() + Dispatchers.Main)
    
    fun doWork() {
        scope.launch { /* work */ }
    }
    
    fun cleanup() {
        scope.cancel() // Cancel when done
    }
}
```

### 7. **Use Structured Concurrency**

Follow structured concurrency principles:

- Launch coroutines within a scope
- Let cancellation propagate naturally
- Use `coroutineScope` for parallel decomposition
- Don't create "fire and forget" coroutines without a scope

```kotlin
// ✅ Good - structured
suspend fun loadUserData(userId: String) = coroutineScope {
    val user = async { fetchUser(userId) }
    val posts = async { fetchPosts(userId) }
    UserData(user.await(), posts.await())
}

// ❌ Bad - unstructured
suspend fun loadUserData(userId: String) {
    GlobalScope.launch { fetchUser(userId) } // No scope!
    GlobalScope.launch { fetchPosts(userId) } // No scope!
}
```

### 8. **Prefer async/await for Parallel Work**

Use `async` when you need results from parallel operations:

```kotlin
// ✅ Good - parallel execution
viewModelScope.launch {
    val userDeferred = async { fetchUser() }
    val postsDeferred = async { fetchPosts() }
    
    val user = userDeferred.await()
    val posts = postsDeferred.await()
}

// ❌ Bad - sequential (slower)
viewModelScope.launch {
    val user = fetchUser() // Waits
    val posts = fetchPosts() // Waits after user
}
```

### 9. **Don't Catch CancellationException as Regular Exception**

`CancellationException` is special - always rethrow it:

```kotlin
// ✅ Good
try {
    work()
} catch (e: CancellationException) {
    cleanup()
    throw e // Rethrow
} catch (e: Exception) {
    handleError(e)
}

// ❌ Bad
try {
    work()
} catch (e: Exception) {
    // This catches CancellationException too!
    handleError(e)
}
```

### 10. **Use SupervisorJob for Independent Children**

When child coroutines should be independent:

```kotlin
// ✅ Good - children are independent
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)
scope.launch {
    // Child 1 - failure doesn't cancel Child 2
}
scope.launch {
    // Child 2 - failure doesn't cancel Child 1
}
```

### 11. **Avoid Excessive Thread Switching**

Minimize dispatcher switches:

```kotlin
// ❌ Bad - multiple switches
suspend fun processData() = withContext(Dispatchers.IO) {
    val data = fetchData()
    withContext(Dispatchers.Default) {
        process(data)
    }
}

// ✅ Good - single switch
suspend fun processData() = withContext(Dispatchers.IO) {
    val data = fetchData()
    process(data) // If process is CPU-intensive, switch once
}
```

### 12. **Name Your Coroutines for Debugging**

Use `CoroutineName` for easier debugging:

```kotlin
viewModelScope.launch(CoroutineName("LoadUserData")) {
    // Coroutine name appears in logs
}
```

### 13. **Handle Timeouts Appropriately**

Use timeouts for operations that might hang:

```kotlin
viewModelScope.launch {
    try {
        val result = withTimeout(5000) {
            fetchData()
        }
    } catch (e: TimeoutCancellationException) {
        // Handle timeout
    }
}
```

### 14. **Don't Access UI from Background Threads**

Always switch back to Main for UI updates:

```kotlin
viewModelScope.launch {
    val data = withContext(Dispatchers.IO) {
        fetchData()
    }
    // Already on Main - safe to update UI
    updateUI(data)
}
```

### 15. **Use Flow for Streams of Data**

For continuous data streams, use Flow instead of multiple coroutines:

```kotlin
// ✅ Good - use Flow
fun getUserUpdates(): Flow<User> = flow {
    while (true) {
        emit(fetchUser())
        delay(1000)
    }
}

// ❌ Bad - multiple coroutines
fun getUserUpdates() {
    viewModelScope.launch {
        while (true) {
            val user = fetchUser()
            _user.value = user
            delay(1000)
        }
    }
}
```

---

## Common Patterns

### Pattern 1: Network Request with Loading State

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun loadUser(id: String) {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val user = repository.getUser(id)
                _uiState.value = UiState.Success(user)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

### Pattern 2: Parallel Network Requests

```kotlin
viewModelScope.launch {
    val userDeferred = async { repository.getUser(userId) }
    val postsDeferred = async { repository.getPosts(userId) }
    
    val user = userDeferred.await()
    val posts = postsDeferred.await()
    
    // Use both results
    displayData(user, posts)
}
```

### Pattern 3: Sequential Operations with Error Handling

```kotlin
viewModelScope.launch {
    try {
        val token = repository.login(username, password)
        val user = repository.getUserProfile(token)
        updateUI(user)
    } catch (e: NetworkException) {
        showError("Network error: ${e.message}")
    } catch (e: Exception) {
        showError("Unexpected error: ${e.message}")
    }
}
```

### Pattern 4: Timeout Operations

```kotlin
viewModelScope.launch {
    try {
        val result = withTimeout(5000) { // 5 second timeout
            repository.fetchData()
        }
        // Handle result
    } catch (e: TimeoutCancellationException) {
        // Handle timeout
    }
}
```

### Pattern 5: Retry Logic

```kotlin
suspend fun <T> retry(
    times: Int = 3,
    delay: Long = 1000,
    block: suspend () -> T
): T {
    repeat(times - 1) {
        try {
            return block()
        } catch (e: Exception) {
            delay(delay)
        }
    }
    return block() // Last attempt
}

// Usage
viewModelScope.launch {
    val data = retry(times = 3) {
        repository.fetchData()
    }
}
```

---

## Key Interview Points

### 1. **What are Coroutines?**
- Lightweight threads that can suspend without blocking
- Provide structured concurrency
- Built-in cancellation support

### 2. **Difference between Coroutines and Threads**
- Coroutines are lightweight (millions can run)
- Threads are expensive (limited by OS)
- Coroutines can suspend/resume
- Coroutines follow structured concurrency

### 3. **What is a Suspend Function?**
- Function that can pause and resume
- Can only be called from coroutine or another suspend function
- Doesn't block the thread when suspended

### 4. **What are Dispatchers?**
- Determine which thread pool a coroutine runs on
- Main: UI operations
- IO: Network/disk operations
- Default: CPU-intensive work

### 5. **What is Coroutine Scope?**
- Defines the scope where coroutines can be launched
- Manages lifecycle and cancellation
- Examples: viewModelScope, lifecycleScope

### 6. **How to Handle Exceptions?**
- Try-catch blocks
- CoroutineExceptionHandler
- SupervisorJob for independent children

### 7. **What is Main-Safety?**
- Function doesn't block UI thread
- Use `withContext()` to switch dispatchers
- Keep UI code on main thread

### 8. **Structured Concurrency**
- Coroutines organized in hierarchy
- Parent cancellation cancels children
- Child failure can cancel parent (unless SupervisorJob)

---

## References

- [Official Android Coroutines Documentation](https://developer.android.com/kotlin/coroutines)
- [Advanced Coroutines Concepts](https://developer.android.com/kotlin/coroutines/coroutines-adv)
- [Testing Coroutines](https://developer.android.com/kotlin/coroutines/test)
- [Coroutines Best Practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
- [Cancellation in Coroutines (Medium)](https://medium.com/androiddevelopers/cancellation-in-coroutines-aa6b90163629)
- [Coroutines Overview (JetBrains)](https://kotlinlang.org/docs/coroutines-overview.html)
- [Coroutines Guide (JetBrains)](https://kotlinlang.org/docs/coroutines-guide.html)

---

## Quick Reference

| Concept | Description |
|---------|-------------|
| `launch` | Fire-and-forget coroutine builder |
| `async` | Coroutine builder that returns a result |
| `suspend` | Keyword for functions that can suspend |
| `withContext` | Switch dispatcher within a coroutine |
| `viewModelScope` | Coroutine scope bound to ViewModel |
| `lifecycleScope` | Coroutine scope bound to Lifecycle |
| `Dispatchers.Main` | Main/UI thread dispatcher |
| `Dispatchers.IO` | I/O operations dispatcher |
| `Dispatchers.Default` | CPU-intensive work dispatcher |
| `SupervisorJob` | Job that doesn't cancel children on failure |
| `coroutineScope` | Builder that waits for all children to complete |
| `Job` | Handle to a coroutine, manages lifecycle |
| `CoroutineContext` | Defines coroutine behavior (Job, Dispatcher, Name, ExceptionHandler) |
| `isActive` | Property to check if coroutine is still active |
| `ensureActive()` | Throws CancellationException if coroutine is cancelled |
| `withTimeout` | Cancels coroutine after specified duration |
| `runTest` | Coroutine builder for testing |
| `TestDispatcher` | Dispatcher for testing coroutines |
| `NonCancellable` | Context for operations that shouldn't be cancelled |

---

## Senior/Lead Level Interview Questions

### 1. **Explain the difference between `coroutineScope` and `CoroutineScope`. When would you use each?**

**Answer:**

- **`coroutineScope`** (lowercase): A suspend function builder that creates a scope and waits for all children to complete. It's used for parallel decomposition within a suspend function. It automatically propagates exceptions and ensures structured concurrency.

```kotlin
suspend fun loadData() = coroutineScope {
    val data1 = async { fetchData1() }
    val data2 = async { fetchData2() }
    CombinedData(data1.await(), data2.await())
}
```

- **`CoroutineScope`** (uppercase): A class that defines a scope for launching coroutines. It's typically created once and reused, tied to a lifecycle (like ViewModel). You manually manage its lifecycle.

```kotlin
class MyRepository {
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.IO)
    
    fun loadData() {
        scope.launch { /* work */ }
    }
    
    fun cleanup() {
        scope.cancel()
    }
}
```

**Use `coroutineScope`** when you need parallel decomposition within a suspend function. **Use `CoroutineScope`** when you need a long-lived scope tied to a component's lifecycle.

---

### 2. **What happens when you cancel a coroutine that's inside a `withContext` block? Explain the cancellation behavior.**

**Answer:**

When a coroutine is cancelled:
1. The `withContext` block receives the cancellation signal
2. If the block is still executing, it should check `isActive` or handle `CancellationException`
3. Most suspend functions (like `delay`, network calls) are cancellable and will throw `CancellationException`
4. The coroutine resumes in the original context (the one before `withContext`)

**Important:** If you catch `CancellationException` in `withContext`, you must rethrow it:

```kotlin
suspend fun fetchData() = withContext(Dispatchers.IO) {
    try {
        api.getData() // This can throw CancellationException
    } catch (e: CancellationException) {
        cleanup()
        throw e // Must rethrow
    }
}
```

If you don't rethrow `CancellationException`, the cancellation won't propagate properly, and the coroutine may not cancel as expected.

---

### 3. **Explain the difference between `SupervisorJob` and regular `Job` in exception handling. Provide a concrete example.**

**Answer:**

- **Regular `Job`**: When a child coroutine fails, it cancels all siblings and propagates the exception to the parent, potentially cancelling the entire scope.

- **`SupervisorJob`**: When a child coroutine fails, it doesn't cancel siblings. The exception is still propagated, but other children continue running.

```kotlin
// Regular Job - one failure cancels all
val scope = CoroutineScope(Job() + Dispatchers.Main)
scope.launch {
    throw Exception("Error") // This cancels Child 2
}
scope.launch {
    delay(1000)
    println("This won't print") // Cancelled by Child 1's failure
}

// SupervisorJob - failures are independent
val supervisorScope = CoroutineScope(SupervisorJob() + Dispatchers.Main)
supervisorScope.launch {
    throw Exception("Error") // This doesn't cancel Child 2
}
supervisorScope.launch {
    delay(1000)
    println("This will print") // Still runs despite Child 1's failure
}
```

**Use `SupervisorJob`** when you have independent operations where one failure shouldn't affect others (e.g., uploading multiple files, fetching different data sources).

---

### 4. **How would you implement a retry mechanism with exponential backoff using coroutines? Show both a basic and advanced version.**

**Answer:**

**Basic Version:**
```kotlin
suspend fun <T> retryWithBackoff(
    times: Int = 3,
    initialDelay: Long = 1000,
    maxDelay: Long = 10000,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    var currentDelay = initialDelay
    repeat(times - 1) {
        try {
            return block()
        } catch (e: Exception) {
            if (e is CancellationException) throw e
            delay(currentDelay)
            currentDelay = (currentDelay * factor).toLong().coerceAtMost(maxDelay)
        }
    }
    return block() // Last attempt
}
```

**Advanced Version with Jitter:**
```kotlin
suspend fun <T> retryWithExponentialBackoff(
    times: Int = 3,
    initialDelay: Long = 1000,
    maxDelay: Long = 10000,
    factor: Double = 2.0,
    jitter: Boolean = true,
    block: suspend () -> T
): T {
    var currentDelay = initialDelay
    repeat(times - 1) {
        try {
            return block()
        } catch (e: Exception) {
            if (e is CancellationException) throw e
            val delayWithJitter = if (jitter) {
                (currentDelay * (0.5..1.0).random()).toLong()
            } else {
                currentDelay
            }
            delay(delayWithJitter)
            currentDelay = (currentDelay * factor).toLong().coerceAtMost(maxDelay)
        }
    }
    return block()
}
```

**Usage:**
```kotlin
viewModelScope.launch {
    val data = retryWithExponentialBackoff(
        times = 5,
        initialDelay = 500,
        maxDelay = 10000
    ) {
        repository.fetchData()
    }
}
```

---

### 5. **What is the difference between `Dispatchers.Main` and `Dispatchers.Main.immediate`? When would you use each?**

**Answer:**

- **`Dispatchers.Main`**: Always dispatches to the main thread, even if you're already on the main thread. This adds a dispatch operation to the queue.

- **`Dispatchers.Main.immediate`**: If already on the main thread, executes immediately without dispatching. Otherwise, dispatches to the main thread.

```kotlin
// Already on main thread
viewModelScope.launch(Dispatchers.Main) {
    println("Dispatched") // Queued, executes later
}

viewModelScope.launch(Dispatchers.Main.immediate) {
    println("Immediate") // Executes immediately
}
```

**Use `Dispatchers.Main.immediate`** when:
- You want to avoid unnecessary dispatch overhead
- You're already on the main thread and want immediate execution
- Performance is critical

**Use `Dispatchers.Main`** when:
- You want consistent behavior regardless of current thread
- You want to ensure execution order in the main thread queue

---

### 6. **How would you handle a scenario where you need to cancel a coroutine but ensure cleanup code always runs, even if the coroutine is already cancelled?**

**Answer:**

Use `NonCancellable` context for cleanup operations:

```kotlin
viewModelScope.launch {
    val resource = acquireResource()
    try {
        // Work that can be cancelled
        processResource(resource)
    } finally {
        // Cleanup that must always run
        withContext(NonCancellable) {
            releaseResource(resource) // Won't be cancelled
        }
    }
}
```

**Alternative pattern with `invokeOnCompletion`:**
```kotlin
val job = viewModelScope.launch {
    val resource = acquireResource()
    processResource(resource)
}

job.invokeOnCompletion { exception ->
    // Always called, even on cancellation
    if (exception is CancellationException) {
        cleanup()
    }
}
```

**Key point:** `NonCancellable` ensures that even if the coroutine is cancelled, the cleanup code in that block will complete.

---

### 7. **Explain what happens when you use `async` but never call `await()`. What are the implications?**

**Answer:**

If you launch an `async` coroutine but never call `await()`:
1. The coroutine still runs
2. If it throws an exception, it's stored in the `Deferred`
3. The exception is only thrown when `await()` is called
4. If the parent scope is cancelled, the `async` coroutine is also cancelled
5. **Memory leak risk**: If you lose the reference to the `Deferred`, the coroutine may continue running

```kotlin
// Problematic code
fun loadData() {
    viewModelScope.launch {
        async { fetchData1() } // Deferred is lost!
        async { fetchData2() } // Deferred is lost!
        // These coroutines may continue running
    }
}

// Correct approach
fun loadData() {
    viewModelScope.launch {
        val deferred1 = async { fetchData1() }
        val deferred2 = async { fetchData2() }
        // Use await() or store references
        val result1 = deferred1.await()
        val result2 = deferred2.await()
    }
}
```

**Best practice:** Always either await the result or store the `Deferred` reference if you need it later. Use `coroutineScope` to ensure all children complete.

---

### 8. **How would you implement a debounce mechanism using coroutines? Show a reusable solution.**

**Answer:**

**Using `Job` cancellation:**
```kotlin
class Debouncer(private val delayMs: Long) {
    private var debounceJob: Job? = null
    
    fun debounce(scope: CoroutineScope, action: suspend () -> Unit) {
        debounceJob?.cancel()
        debounceJob = scope.launch {
            delay(delayMs)
            action()
        }
    }
}

// Usage
val debouncer = Debouncer(300)
debouncer.debounce(viewModelScope) {
    performSearch(query)
}
```

**More advanced version with Flow:**
```kotlin
fun <T> Flow<T>.debounce(timeout: Long): Flow<T> = flow {
    var lastValue: T? = null
    var lastEmitTime = 0L
    
    collect { value ->
        val currentTime = System.currentTimeMillis()
        lastValue = value
        
        if (currentTime - lastEmitTime >= timeout) {
            emit(value)
            lastEmitTime = currentTime
        } else {
            delay(timeout - (currentTime - lastEmitTime))
            if (lastValue == value) {
                emit(value)
                lastEmitTime = System.currentTimeMillis()
            }
        }
    }
}
```

**Using Kotlin Flow's built-in debounce:**
```kotlin
searchQueryFlow
    .debounce(300)
    .collect { query ->
        performSearch(query)
    }
```

---

### 9. **What is the difference between `runBlocking` and `coroutineScope`? When is it acceptable to use `runBlocking`?**

**Answer:**

- **`runBlocking`**: Blocks the current thread until the coroutine completes. It's a bridge between blocking and non-blocking code. **Never use in production Android code** - it blocks the thread.

- **`coroutineScope`**: Suspends the coroutine (doesn't block the thread) until all children complete. It's used for structured concurrency within suspend functions.

```kotlin
// runBlocking - BLOCKS the thread
fun main() {
    runBlocking {
        delay(1000) // Blocks the thread for 1 second
    }
}

// coroutineScope - SUSPENDS (non-blocking)
suspend fun loadData() = coroutineScope {
    delay(1000) // Suspends, doesn't block
    // Other coroutines can run on this thread
}
```

**Acceptable uses of `runBlocking`:**
- In `main()` function for simple scripts
- In unit tests (though `runTest` is preferred)
- In `@Test` functions for testing (but prefer `runTest`)

**Never use `runBlocking` in:**
- Android Activities/Fragments
- ViewModels
- Repository classes
- Any production code

---

### 10. **How would you implement a semaphore-like mechanism using coroutines to limit concurrent operations?**

**Answer:**

**Using `Mutex` (mutual exclusion):**
```kotlin
class RateLimiter(private val maxConcurrent: Int) {
    private val semaphore = Mutex()
    private var activeCount = 0
    
    suspend fun <T> execute(block: suspend () -> T): T {
        semaphore.lock()
        try {
            while (activeCount >= maxConcurrent) {
                semaphore.unlock()
                delay(10) // Wait a bit
                semaphore.lock()
            }
            activeCount++
        } finally {
            semaphore.unlock()
        }
        
        return try {
            block()
        } finally {
            semaphore.lock()
            try {
                activeCount--
            } finally {
                semaphore.unlock()
            }
        }
    }
}
```

**Better approach using `Semaphore` (if available) or `Channel`:**
```kotlin
class ConcurrencyLimiter(private val maxConcurrent: Int) {
    private val channel = Channel<Unit>(maxConcurrent)
    
    init {
        repeat(maxConcurrent) {
            channel.trySend(Unit)
        }
    }
    
    suspend fun <T> execute(block: suspend () -> T): T {
        val permit = channel.receive()
        return try {
            block()
        } finally {
            channel.send(permit)
        }
    }
}

// Usage
val limiter = ConcurrencyLimiter(3)
viewModelScope.launch {
    limiter.execute {
        uploadFile(file)
    }
}
```

---

### 11. **Explain the execution order in this code. What will be printed and why?**

```kotlin
fun main() = runBlocking {
    println("1")
    launch {
        println("2")
        delay(100)
        println("3")
    }
    println("4")
    delay(50)
    println("5")
}
```

**Answer:**

Output: `1, 4, 2, 5, 3`

**Explanation:**
1. `1` prints immediately (main coroutine)
2. `launch` starts a new coroutine but doesn't wait - it's scheduled
3. `4` prints immediately (main coroutine continues)
4. `delay(50)` in main coroutine suspends for 50ms
5. During this suspension, the launched coroutine starts: `2` prints
6. Main coroutine resumes after 50ms: `5` prints
7. Launched coroutine's `delay(100)` completes after 100ms total: `3` prints

**Key concepts:**
- `launch` is non-blocking - it schedules the coroutine
- `delay` suspends the coroutine, allowing other coroutines to run
- Coroutines are concurrent, not parallel (unless using different dispatchers)

---

### 12. **How would you implement a timeout that doesn't cancel the operation but just stops waiting for it?**

**Answer:**

Use `select` with `onTimeout`:

```kotlin
suspend fun <T> fetchWithTimeout(
    timeout: Long,
    block: suspend () -> T
): T? = select {
    async { block() }.onAwait { it }
    onTimeout(timeout) { null }
}
```

**Or using `withTimeoutOrNull`:**
```kotlin
suspend fun <T> fetchWithTimeout(
    timeout: Long,
    block: suspend () -> T
): T? = withTimeoutOrNull(timeout) {
    block()
}
```

**If you want the operation to continue but return early:**
```kotlin
suspend fun <T> fetchWithEarlyReturn(
    timeout: Long,
    block: suspend () -> T
): T? = coroutineScope {
    val deferred = async { block() }
    val timeoutJob = launch {
        delay(timeout)
        // Operation continues, but we return null
    }
    
    select {
        deferred.onAwait { timeoutJob.cancel(); it }
        timeoutJob.onJoin { null }
    }
}
```

**Note:** In most cases, you want cancellation. Use `withTimeoutOrNull` if you just want to stop waiting without cancelling the operation (though the operation will still be cancelled in practice).

---

### 13. **What are the performance implications of using `Dispatchers.IO` vs `Dispatchers.Default`? When should you use each?**

**Answer:**

**`Dispatchers.IO`:**
- Optimized for I/O-bound operations
- Shares threads with a thread pool (can have many threads)
- Threads are designed to wait on I/O operations
- Use for: network calls, file I/O, database operations

**`Dispatchers.Default`:**
- Optimized for CPU-bound operations
- Limited to number of CPU cores
- Threads are designed for computation
- Use for: sorting, parsing, calculations, image processing

**Performance implications:**
- Using `Dispatchers.IO` for CPU work wastes threads (they'll block CPU cores)
- Using `Dispatchers.Default` for I/O work limits concurrency (limited threads waiting on I/O)

```kotlin
// ✅ Correct
suspend fun fetchData() = withContext(Dispatchers.IO) {
    api.getData() // I/O operation
}

suspend fun processImage() = withContext(Dispatchers.Default) {
    imageProcessor.process() // CPU operation
}

// ❌ Wrong
suspend fun processImage() = withContext(Dispatchers.IO) {
    imageProcessor.process() // Wastes I/O threads
}
```

**Rule of thumb:** If it waits (network, disk), use IO. If it computes (CPU), use Default.

---

### 14. **How would you implement a coroutine that can be paused and resumed on demand?**

**Answer:**

Using a `Channel` or `Mutex` as a pause mechanism:

```kotlin
class PausableCoroutine {
    private val pauseChannel = Channel<Unit>()
    private var isPaused = false
    
    suspend fun pause() {
        if (!isPaused) {
            isPaused = true
        }
    }
    
    suspend fun resume() {
        if (isPaused) {
            isPaused = false
            pauseChannel.send(Unit)
        }
    }
    
    suspend fun execute(block: suspend () -> Unit) {
        viewModelScope.launch {
            while (true) {
                if (isPaused) {
                    pauseChannel.receive() // Wait for resume
                }
                block()
                delay(1000) // Your work interval
            }
        }
    }
}
```

**Using `Job` for simpler cases:**
```kotlin
class PausableWork {
    private var workJob: Job? = null
    
    fun start(scope: CoroutineScope) {
        workJob = scope.launch {
            while (isActive) {
                doWork()
                delay(1000)
            }
        }
    }
    
    fun pause() {
        workJob?.cancel()
    }
    
    fun resume(scope: CoroutineScope) {
        if (workJob?.isActive != true) {
            start(scope)
        }
    }
}
```

---

### 15. **Explain what happens with exceptions in this code. Will both coroutines complete?**

```kotlin
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

scope.launch {
    throw Exception("Error 1")
}

scope.launch {
    delay(1000)
    println("Completed")
}
```

**Answer:**

**Yes, both coroutines will attempt to run, but:**
1. The first coroutine throws an exception immediately
2. Because of `SupervisorJob`, the exception doesn't cancel the second coroutine
3. The second coroutine will complete and print "Completed"
4. However, the exception from the first coroutine needs to be handled, or it will be reported to the exception handler

**Important:** Uncaught exceptions in coroutines launched with `launch` need a `CoroutineExceptionHandler`, or they'll crash the app (in some contexts) or be logged.

**Better version:**
```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e("Coroutine", "Uncaught exception", exception)
}

val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main + handler)

scope.launch {
    throw Exception("Error 1") // Handled by handler
}

scope.launch {
    delay(1000)
    println("Completed") // Still executes
}
```

---

### 16. **How would you implement a cache that fetches data asynchronously and serves stale data while fetching fresh data?**

**Answer:**

```kotlin
class AsyncCache<T>(
    private val fetch: suspend () -> T,
    private val scope: CoroutineScope
) {
    @Volatile
    private var cachedValue: T? = null
    private val mutex = Mutex()
    @Volatile
    private var refreshJob: Job? = null
    
    suspend fun get(): T {
        // Return cached value immediately if available
        cachedValue?.let { return it }
        
        // Otherwise, fetch and cache
        return mutex.withLock {
            cachedValue ?: fetch().also { cachedValue = it }
        }
    }
    
    fun refresh() {
        refreshJob?.cancel()
        refreshJob = scope.launch {
            val freshValue = fetch()
            mutex.withLock {
                cachedValue = freshValue
            }
        }
    }
    
    suspend fun getOrFetch(): T {
        // Start refresh in background
        refresh()
        // Return stale data immediately, or wait for fresh if no cache
        return cachedValue ?: mutex.withLock {
            cachedValue ?: fetch().also { cachedValue = it }
        }
    }
}
```

**More advanced with Flow:**
```kotlin
class FlowCache<T>(
    private val fetch: suspend () -> T
) {
    private val _data = MutableStateFlow<T?>(null)
    val data: StateFlow<T?> = _data.asStateFlow()
    
    suspend fun refresh() {
        _data.value = fetch()
    }
    
    fun getFlow(): Flow<T> = flow {
        // Emit cached value immediately
        _data.value?.let { emit(it) }
        // Fetch and emit fresh value
        emit(fetch().also { _data.value = it })
    }
}
```

---

### 17. **What is the difference between `launch` and `async` in terms of exception handling? Provide examples.**

**Answer:**

**`launch`:**
- Exceptions are thrown immediately and can crash the coroutine
- Uncaught exceptions need `CoroutineExceptionHandler`
- Exceptions propagate to the parent scope (unless `SupervisorJob`)

**`async`:**
- Exceptions are deferred until `await()` is called
- If you don't call `await()`, the exception is stored in the `Deferred`
- Calling `await()` will throw the exception

```kotlin
// launch - exception thrown immediately
viewModelScope.launch {
    throw Exception("Error") // Thrown immediately
    // This line never executes
}

// async - exception deferred
viewModelScope.launch {
    val deferred = async {
        throw Exception("Error") // Stored, not thrown yet
    }
    // This executes
    println("Before await")
    deferred.await() // Exception thrown here
    // This never executes
}

// async without await - exception never thrown
viewModelScope.launch {
    async {
        throw Exception("Error") // Lost exception!
    }
    // Continues normally, exception is lost
}
```

**Best practice:** Always handle exceptions from `async` when calling `await()`, or use `supervisorScope` to isolate failures.

---

### 18. **How would you implement a producer-consumer pattern using coroutines?**

**Answer:**

**Using `Channel`:**
```kotlin
class ProducerConsumer<T> {
    private val channel = Channel<T>(Channel.UNLIMITED)
    
    suspend fun produce(item: T) {
        channel.send(item)
    }
    
    fun consume(scope: CoroutineScope, consumer: suspend (T) -> Unit) {
        scope.launch {
            for (item in channel) {
                consumer(item)
            }
        }
    }
    
    fun close() {
        channel.close()
    }
}
```

**Using `Flow`:**
```kotlin
class FlowProducer<T> {
    private val _flow = MutableSharedFlow<T>(replay = 0, extraBufferCapacity = 64)
    val flow: SharedFlow<T> = _flow.asSharedFlow()
    
    suspend fun produce(item: T) {
        _flow.emit(item)
    }
}

// Usage
val producer = FlowProducer<String>()
producer.flow.collect { item ->
    process(item)
}
```

**Bounded channel with backpressure:**
```kotlin
class BoundedProducerConsumer<T>(capacity: Int) {
    private val channel = Channel<T>(capacity)
    
    suspend fun produce(item: T) {
        channel.send(item) // Suspends if channel is full
    }
    
    suspend fun consume(): T {
        return channel.receive()
    }
}
```

---

### 19. **Explain the memory and performance implications of using `GlobalScope` vs `viewModelScope`. Why is `GlobalScope` problematic?**

**Answer:**

**`GlobalScope`:**
- Lives for the entire application lifetime
- Not tied to any lifecycle
- Can cause memory leaks if it holds references to Activities/Fragments
- Coroutines launched in `GlobalScope` are not automatically cancelled
- Hard to test and reason about

**`viewModelScope`:**
- Tied to ViewModel lifecycle
- Automatically cancelled when ViewModel is cleared
- Prevents memory leaks
- Easier to test (can be replaced in tests)
- Follows structured concurrency

**Memory leak example:**
```kotlin
// ❌ Bad - memory leak
class MyActivity : AppCompatActivity() {
    fun loadData() {
        GlobalScope.launch {
            // Holds reference to Activity
            updateUI(this@MyActivity) // Activity can't be garbage collected
        }
    }
}

// ✅ Good - no leak
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // Cancelled when ViewModel is cleared
            updateUI()
        }
    }
}
```

**Performance:** `GlobalScope` itself doesn't have performance issues, but the memory leaks and uncancelled coroutines can lead to:
- Increased memory usage
- Battery drain (coroutines continue running)
- Slower garbage collection

**Best practice:** Never use `GlobalScope` in Android production code. Always use lifecycle-aware scopes.

---

### 20. **How would you implement a circuit breaker pattern using coroutines to handle failing services gracefully?**

**Answer:**

```kotlin
enum class CircuitState {
    CLOSED,    // Normal operation
    OPEN,      // Failing, reject requests
    HALF_OPEN  // Testing if service recovered
}

class CircuitBreaker(
    private val failureThreshold: Int = 5,
    private val timeout: Long = 60000, // 1 minute
    private val halfOpenTimeout: Long = 30000 // 30 seconds
) {
    @Volatile
    private var state = CircuitState.CLOSED
    
    @Volatile
    private var failureCount = 0
    
    @Volatile
    private var lastFailureTime = 0L
    
    @Volatile
    private var halfOpenAttemptTime = 0L
    
    suspend fun <T> execute(block: suspend () -> T): T {
        when (state) {
            CircuitState.CLOSED -> {
                return try {
                    val result = block()
                    onSuccess()
                    result
                } catch (e: Exception) {
                    onFailure()
                    throw e
                }
            }
            CircuitState.OPEN -> {
                if (System.currentTimeMillis() - lastFailureTime > timeout) {
                    state = CircuitState.HALF_OPEN
                    halfOpenAttemptTime = System.currentTimeMillis()
                } else {
                    throw CircuitBreakerOpenException("Circuit breaker is OPEN")
                }
            }
            CircuitState.HALF_OPEN -> {
                if (System.currentTimeMillis() - halfOpenAttemptTime > halfOpenTimeout) {
                    // Try again
                    return try {
                        val result = block()
                        onSuccess()
                        result
                    } catch (e: Exception) {
                        onFailure()
                        throw e
                    }
                } else {
                    throw CircuitBreakerOpenException("Circuit breaker is HALF_OPEN")
                }
            }
        }
    }
    
    private fun onSuccess() {
        failureCount = 0
        state = CircuitState.CLOSED
    }
    
    private fun onFailure() {
        failureCount++
        lastFailureTime = System.currentTimeMillis()
        if (failureCount >= failureThreshold) {
            state = CircuitState.OPEN
        }
    }
}

class CircuitBreakerOpenException(message: String) : Exception(message)

// Usage
val circuitBreaker = CircuitBreaker(failureThreshold = 5, timeout = 60000)

viewModelScope.launch {
    try {
        val data = circuitBreaker.execute {
            repository.fetchData()
        }
        // Handle success
    } catch (e: CircuitBreakerOpenException) {
        // Service is down, use cached data or show error
        showFallbackUI()
    } catch (e: Exception) {
        // Other errors
        handleError(e)
    }
}
```

This pattern:
- Opens the circuit after N failures
- Rejects requests when open
- Periodically tries to close (half-open state)
- Closes when service recovers

---

## Summary

These 20 questions cover:
- Advanced coroutine concepts and internals
- Exception handling and cancellation
- Performance optimization
- Design patterns with coroutines
- Memory management and leaks
- Testing strategies
- Real-world problem solving

Mastering these topics demonstrates deep understanding of coroutines suitable for Senior/Lead level positions.

