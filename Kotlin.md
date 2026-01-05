# Kotlin - Comprehensive Developer Guide

> **Estimated Reading Time:** 4-8 hours  
> **Target Audience:** Android Developers, Senior Engineers preparing for interviews  
> **Last Updated:** January 2026

---

## Table of Contents

1. [Kotlin Basics (1-2 hours)](#kotlin-basics)
2. [Advanced Kotlin Topics (2-3 hours)](#advanced-kotlin-topics)
3. [Common Kotlin Interview Questions (1 hour)](#common-kotlin-interview-questions)
4. [Advanced Kotlin Interview Questions (1-2 hours)](#advanced-kotlin-interview-questions)
5. [Quick Reference: Kotlin for DSA (20 minutes)](#quick-reference-kotlin-for-dsa)

---

## Kotlin Basics
*Senior Engineer Revision - Quick Reference*

### Variables & Types

| Declaration | Mutability | Scope | Compile-Time Constant |
|-------------|-----------|-------|----------------------|
| `val` | Immutable reference | Runtime | No |
| `var` | Mutable reference | Runtime | No |
| `const val` | Immutable | Compile-time | Yes (top-level/object only) |

```kotlin
val name: String = "John"           // Immutable reference
var age: Int = 25                   // Mutable reference
const val MAX_SIZE = 100            // Compile-time constant

// Type inference
val city = "NYC"                    // Type inferred as String
var count = 0                       // Type inferred as Int
```

**Key Points:**
- `val` → read-only reference (not necessarily immutable object)
- `const val` → compile-time constant, primitives & strings only
- Use `val` by default, `var` only when needed

---

### Null Safety

```kotlin
var nullable: String? = null        // Nullable type
var nonNull: String = "Hello"       // Non-null type

// Safe call
val length = nullable?.length       // Returns null if nullable is null

// Elvis operator
val len = nullable?.length ?: 0     // Returns 0 if null

// Not-null assertion
val l = nullable!!.length           // Throws NPE if null

// Safe cast
val str = obj as? String            // Returns null if cast fails
```

**Scope Functions:**

| Function | Context | Return | Use Case |
|----------|---------|--------|----------|
| `let` | `it` | Lambda result | Null checks, transformations |
| `also` | `it` | Object itself | Side effects, logging |
| `run` | `this` | Lambda result | Object config & compute |
| `apply` | `this` | Object itself | Object configuration |
| `with` | `this` | Lambda result | Multiple calls on object |

```kotlin
// let - execute on non-null
nullable?.let { value ->
    println(value.length)
}

// also - side effects
val list = mutableListOf<Int>()
    .also { println("List created") }

// apply - configuration
val person = Person().apply {
    name = "John"
    age = 30
}

// run - compute result
val result = person.run {
    "$name is $age years old"
}

// with - multiple operations
with(person) {
    name = "Jane"
    age = 25
}

// takeIf/takeUnless - conditional return
val adult = person.takeIf { it.age >= 18 }  // returns object if TRUE else null
val minor = person.takeUnless { it.age >= 18 } // returns object if FALSE else null
```

---

### Functions

```kotlin
// Basic function
fun sum(a: Int, b: Int): Int {
    return a + b
}

// Single expression
fun multiply(a: Int, b: Int) = a * b

// Default parameters
fun greet(name: String = "Guest") = "Hello, $name"

// Named arguments
fun createUser(name: String, age: Int, city: String) { }
createUser(name = "John", city = "NYC", age = 30)

// Extension function
fun String.removeWhitespace() = this.replace(" ", "")
"Hello World".removeWhitespace()  // "HelloWorld"

// Infix function
infix fun Int.times(str: String) = str.repeat(this)
3 times "Ha"  // "HaHaHa"

// Inline function
inline fun <T> measureTime(block: () -> T): T {
    val start = System.currentTimeMillis()
    val result = block()
    println("Time: ${System.currentTimeMillis() - start}ms")
    return result
}

// Variable arguments
fun sum(vararg numbers: Int): Int {
    return numbers.sum()
}
sum(1, 2, 3, 4)  // 10
```

---

### Classes & Objects

```kotlin
// Data class - auto-generates `equals`, `hashCode`, `toString`, `copy`
data class User(val name: String, val age: Int)

val user = User("John", 30)
val copy = user.copy(age = 31)

// Sealed class - restricted hierarchy
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    object Loading : Result()
}

// Object declaration - singleton
object DatabaseManager {
    fun connect() { }
}

// Companion object - static members
class MyClass {
    companion object {
        const val CONSTANT = "value"
        fun create() = MyClass()
    }
}

// Enum class
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}

// Value class (inline class) - zero-overhead wrapper
@JvmInline
value class UserId(val id: String)
```

---

### Control Flow

```kotlin
// If expression
val max = if (a > b) a else b

// When expression
when (x) {
    1 -> println("One")
    2, 3 -> println("Two or Three")
    in 4..10 -> println("Between 4 and 10")
    is String -> println("String: $x")
    else -> println("Other")
}

// When as expression
val result = when {
    x > 0 -> "Positive"
    x < 0 -> "Negative"
    else -> "Zero"
}

// For loop
for (i in 1..5) { }                 // 1, 2, 3, 4, 5
for (i in 1 until 5) { }            // 1, 2, 3, 4
for (i in 5 downTo 1) { }           // 5, 4, 3, 2, 1
for (i in 0..10 step 2) { }         // 0, 2, 4, 6, 8, 10

// Iterate collection
for (item in collection) { }
for ((index, value) in collection.withIndex()) { }

// While/do-while
while (condition) { }
do { } while (condition)
```

---

### Type System

```kotlin
// Type inference
val name = "John"                   // String inferred

// Smart casts
fun process(obj: Any) {
    if (obj is String) {
        println(obj.length)          // Automatically cast to String
    }
}

// Explicit cast
val str = obj as String             // ClassCastException if fails
val safe = obj as? String           // Returns null if fails

// Type checks
if (obj is String) { }
if (obj !is String) { }
```

---

### Collections

```kotlin
// List - ordered, allows duplicates
val readOnlyList = listOf(1, 2, 3)
val mutableList = mutableListOf(1, 2, 3)
mutableList.add(4)

// Set - unordered, unique elements
val set = setOf(1, 2, 3, 2)         // [1, 2, 3]
val mutableSet = mutableSetOf<Int>()

// Map - key-value pairs
val map = mapOf("a" to 1, "b" to 2)
val mutableMap = mutableMapOf<String, Int>()
mutableMap["c"] = 3

// Common operations
list.filter { it > 2 }              // [3]
list.map { it * 2 }                 // [2, 4, 6]
list.find { it > 2 }                // 3
list.any { it > 2 }                 // true
list.all { it > 0 }                 // true
list.reduce { acc, i -> acc + i }   // 6
list.fold(0) { acc, i -> acc + i }  // 6
```

---

### Generics Basics

```kotlin

/** 

https://typealias.com/start/kotlin-variance/

RULES OF SUBTYPE: So what makes a subtype a subtype? The ability to substitute it for one of its supertypes. A subtype must fully support the contract of its supertype. Specifically, this means it must obey the following three rules:1

1. The subtype must have all of the same public properties and functions as its supertype.
2. Function parameter types must be the SAME AS or MORE general than, those in the supertype. (This is the rule of contravariance.)
3. Its function return types MUST be the SAME AS or MORE specific than, those in its supertype.
*/

// Generic class
class Box<T>(val value: T)
val intBox = Box<Int>(42)
val strBox = Box("Hello")

// Generic function
fun <T> singletonList(item: T): List<T> {
    return listOf(item)
}

// Variance
// out - covariant (producer) . Think of it as, this type can be only used for returning. This ensures Rule #2
interface Producer<out T> {
    fun produce(): T
}

// in - contravariant (consumer). Think of it as, this type can be only used for parameter input. This ensures Rule #3
interface Consumer<in T> {
    fun consume(item: T)
}

// where - multiple bounds
fun <T> process(item: T) where T : Comparable<T>, T : Cloneable {
    // T must implement both Comparable and Cloneable
}

// Reified Type Parameters Example

/**
 * In Kotlin, generic type parameters are erased at runtime (type erasure), so you can't use
 * `is`, `as`, or access the class of a generic type directly. Marking a generic parameter as `reified`
 * in an `inline` function allows you to access the actual type at runtime.
 */

inline fun <reified T> isOfType(value: Any): Boolean {
    return value is T
}

inline fun <reified T> List<*>.filterOfType(): List<T> {
    return this.filterIsInstance<T>()
}

// Example Usages:
val items: List<Any> = listOf("Kotlin", 42, "Java", 17.5)

// Filtering only String values from the list
val strings: List<String> = items.filterOfType<String>()    // ["Kotlin", "Java"]

// Runtime type check using reified generic
val isString = isOfType<String>("hello")    // true
val isInt = isOfType<Int>("hello")          // false

/**
 * Explanation:
 * - `inline` with `reified` allows the compiler to keep the actual type argument at runtime (no type erasure).
 * - You can use type checks (`is T`, `as T`), get class references (`T::class`), etc. inside reified functions.
 * - `inline fun <reified T>` can only be used with inline functions.
 */


/** IN SUMMARY:

1. The out modifier can be used to ensure that the type parameter will only appear publicly in an out-position, which makes it safe for covariance.
2. Conversely, the in modifier can be used to ensure that it will only appear publicly in an in-position, so that it’s safe for contravariance.
*/
```

---

### Lambda & Higher-Order Functions

```kotlin
// Lambda expression
val sum = { a: Int, b: Int -> a + b }
sum(1, 2)  // 3

// Higher-order function
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}
calculate(5, 3) { x, y -> x + y }   // 8

// Function type
val operation: (Int, Int) -> Int = { a, b -> a * b }

// Lambda with receiver
fun buildString(action: StringBuilder.() -> Unit): String {
    val sb = StringBuilder()
    sb.action()
    return sb.toString()
}
buildString {
    append("Hello")
    append(" World")
}

// Anonymous function
val multiply = fun(a: Int, b: Int): Int {
    return a * b
}
```

---

### String Templates

```kotlin
val name = "John"
val age = 30

// Simple template
val greeting = "Hello, $name"

// Expression template
val message = "Next year, $name will be ${age + 1}"

// Multi-line string
val text = """
    Line 1
    Line 2
    Line 3
""".trimIndent()

// Raw string with $
val price = """Price: ${'$'}99"""
```

---

## Advanced Kotlin Topics
*Detailed Explanations with Examples*

### Coroutines Overview

> **Note:** For comprehensive coroutines coverage, see [Coroutines.md](coroutines/Coroutines.md)

**Quick Overview:**
```kotlin
// Launch coroutine
lifecycleScope.launch {
    val data = fetchData()  // Suspend function
    updateUI(data)
}

// Async/await for parallel execution
val deferred1 = async { fetchUser() }
val deferred2 = async { fetchPosts() }
val user = deferred1.await()
val posts = deferred2.await()

// Dispatchers
Dispatchers.Main      // UI thread
Dispatchers.IO        // Network/disk I/O
Dispatchers.Default   // CPU-intensive work
```

**Key Concepts:**
- **Suspend functions** - can pause and resume without blocking
- **Structured concurrency** - coroutines tied to lifecycle
- **Main-safety** - use `withContext(Dispatchers.IO)` for blocking operations

---

### Property Delegates

Delegates allow property behavior to be reused across classes.

**Built-in Delegates:**

```kotlin
// lazy - computed on first access
val expensiveValue: String by lazy {
    println("Computing...")
    "Expensive Result"
}

// observable - observe property changes
var name: String by Delegates.observable("Initial") { prop, old, new ->
    println("$old -> $new")
}

// vetoable - validate before setting
var age: Int by Delegates.vetoable(0) { prop, old, new ->
    new >= 0  // Only allow non-negative values
}

// notNull - late initialization for non-nullable types
var notNullValue: String by Delegates.notNull<String>()
```

**Custom Delegates:**

```kotlin
import kotlin.reflect.KProperty

class LoggingDelegate<T>(private var value: T) {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        println("Getting ${property.name}: $value")
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        println("Setting ${property.name}: $value -> $newValue")
        value = newValue
    }
}

// Usage
class User {
    var name: String by LoggingDelegate("John")
}

val user = User()
println(user.name)      // Getting name: John
user.name = "Jane"      // Setting name: John -> Jane
```

**Map Delegation:**

```kotlin
class UserData(map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}

val user = UserData(mapOf("name" to "John", "age" to 30))
println(user.name)  // John
```

---

### Advanced Generics

#### Variance: Covariance and Contravariance

**Problem with Invariance:**
```kotlin
class Box<T>(val item: T)

// This won't compile - Box<Dog> is not a subtype of Box<Animal>
val dogBox: Box<Dog> = Box(Dog())
val animalBox: Box<Animal> = dogBox  // ❌ Type mismatch
```

**Covariance (out):**
```kotlin
// Producer - only produces T, never consumes
interface Producer<out T> {
    fun produce(): T
}

class AnimalProducer : Producer<Animal> {
    override fun produce(): Animal = Animal()
}

class DogProducer : Producer<Dog> {
    override fun produce(): Dog = Dog()
}

// ✅ Works because Producer is covariant
val dogProducer: Producer<Dog> = DogProducer()
val animalProducer: Producer<Animal> = dogProducer
```

**Contravariance (in):**
```kotlin
// Consumer - only consumes T, never produces
interface Consumer<in T> {
    fun consume(item: T)
}

class AnimalConsumer : Consumer<Animal> {
    override fun consume(item: Animal) {
        println("Consuming animal")
    }
}

// ✅ Works because Consumer is contravariant
val animalConsumer: Consumer<Animal> = AnimalConsumer()
val dogConsumer: Consumer<Dog> = animalConsumer
dogConsumer.consume(Dog())  // Safe - Animal can consume Dog
```

**Star Projection:**
```kotlin
// When you don't know or care about the type
fun printList(list: List<*>) {
    for (item in list) {
        println(item)
    }
}

printList(listOf(1, 2, 3))
printList(listOf("a", "b", "c"))
```

**Reified Type Parameters:**
```kotlin
// Without reified - type erased at runtime
fun <T> isInstance(value: Any): Boolean {
    // return value is T  // ❌ Won't compile - T is erased
    return false
}

// With reified - type available at runtime
inline fun <reified T> isInstance(value: Any): Boolean {
    return value is T  // ✅ Works
}

isInstance<String>("Hello")  // true
isInstance<Int>("Hello")     // false

// Practical example - safer generic parsing
inline fun <reified T> String.parseJson(): T {
    return Gson().fromJson(this, T::class.java)
}

val user = jsonString.parseJson<User>()
```

---

### Inline Functions Deep Dive

**Why Inline?**
- Eliminates lambda object allocation overhead
- Allows non-local returns
- Enables reified type parameters

```kotlin
// Regular function - lambda creates object
fun regularMap(list: List<Int>, transform: (Int) -> Int): List<Int> {
    return list.map(transform)
}

// Inline function - lambda code inlined at call site
inline fun inlineMap(list: List<Int>, transform: (Int) -> Int): List<Int> {
    val result = mutableListOf<Int>()
    for (item in list) {
        result.add(transform(item))
    }
    return result
}
```

**noinline:**
```kotlin
// Some lambdas shouldn't be inlined (stored for later use)
inline fun process(
    inlineAction: () -> Unit,
    noinline storedAction: () -> Unit
) {
    inlineAction()  // Inlined
    saveForLater(storedAction)  // Not inlined, can be stored
}
```

**crossinline:**
```kotlin
// Lambda can't use non-local returns
inline fun runInBackground(crossinline action: () -> Unit) {
    thread {
        action()  // crossinline prevents return from action
    }
}

fun test() {
    runInBackground {
        // return  // ❌ Won't compile with crossinline
        println("Running")
    }
}
```

**When to Use Inline:**
- ✅ Higher-order functions with lambda parameters
- ✅ Functions needing reified type parameters
- ✅ Small, frequently called functions
- ❌ Large functions (increases code size)
- ❌ Functions accessed indirectly

---

### DSL Building (Domain Specific Languages)

**Type-Safe Builders:**

```kotlin
// HTML DSL example
class HTML {
    private val children = mutableListOf<Tag>()
    
    fun head(init: Head.() -> Unit) {
        val head = Head()
        head.init()
        children.add(head)
    }
    
    fun body(init: Body.() -> Unit) {
        val body = Body()
        body.init()
        children.add(body)
    }
    
    override fun toString(): String = 
        "<html>${children.joinToString("")}</html>"
}

abstract class Tag(val name: String) {
    private val children = mutableListOf<Tag>()
    private val attributes = mutableMapOf<String, String>()
    
    protected fun <T : Tag> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }
    
    operator fun String.unaryPlus() {
        children.add(TextTag(this))
    }
    
    override fun toString(): String {
        val attrs = attributes.map { """ ${it.key}="${it.value}"""" }.joinToString("")
        val content = children.joinToString("")
        return "<$name$attrs>$content</$name>"
    }
}

class Head : Tag("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title : Tag("title")

class Body : Tag("body") {
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
}

class H1 : Tag("h1")
class P : Tag("p")
class TextTag(val text: String) : Tag("") {
    override fun toString() = text
}

// DSL builder function
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}

// Usage - looks like HTML!
val page = html {
    head {
        title {
            +"My Page"
        }
    }
    body {
        h1 {
            +"Welcome"
        }
        p {
            +"This is a paragraph"
        }
    }
}
```

**Android View DSL Example:**

```kotlin
class LinearLayoutDSL {
    var orientation: Int = LinearLayout.VERTICAL
    private val views = mutableListOf<View>()
    
    fun textView(init: TextView.() -> Unit) {
        val tv = TextView(context)
        tv.init()
        views.add(tv)
    }
    
    fun button(init: Button.() -> Unit) {
        val btn = Button(context)
        btn.init()
        views.add(btn)
    }
    
    fun build(context: Context): LinearLayout {
        return LinearLayout(context).apply {
            orientation = this@LinearLayoutDSL.orientation
            views.forEach { addView(it) }
        }
    }
}

fun linearLayout(init: LinearLayoutDSL.() -> Unit): LinearLayoutDSL {
    val dsl = LinearLayoutDSL()
    dsl.init()
    return dsl
}

// Usage
val layout = linearLayout {
    orientation = LinearLayout.VERTICAL
    
    textView {
        text = "Hello"
        textSize = 20f
    }
    
    button {
        text = "Click Me"
        setOnClickListener { /* action */ }
    }
}
```

---

### Contracts

Contracts help the compiler understand function behavior for better smart casts.

```kotlin
import kotlin.contracts.*

// Without contract
fun String?.isValid(): Boolean {
    return this != null && this.isNotEmpty()
}

fun process(input: String?) {
    if (input.isValid()) {
        // println(input.length)  // ❌ Smart cast not possible
    }
}

// With contract
fun String?.isValid(): Boolean {
    contract {
        returns(true) implies (this@isValid != null)
    }
    return this != null && this.isNotEmpty()
}

fun processWithContract(input: String?) {
    if (input.isValid()) {
        println(input.length)  // ✅ Smart cast works!
    }
}

// Contract types
fun requireNotNull(value: Any?) {
    contract {
        returns() implies (value != null)  // If returns normally, value is not null
    }
    if (value == null) throw IllegalArgumentException()
}

fun callsInPlace(block: () -> Unit) {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
}
```

**Built-in Functions with Contracts:**
- `require()` - ensures condition is true
- `check()` - checks state
- `requireNotNull()` - ensures value is not null
- `let`, `also`, `run`, etc. - scope functions with contracts

---

### Tail Recursion

**Problem with Recursion:**
```kotlin
// Regular recursion - causes stack overflow for large n
fun factorial(n: Int): Long {
    return if (n <= 1) 1 else n * factorial(n - 1)
}

factorial(10000)  // ❌ StackOverflowError
```

**Tail Recursion Solution:**
```kotlin
// Tail recursive - optimized to loop by compiler
tailrec fun factorial(n: Long, accumulator: Long = 1): Long {
    return if (n <= 1) accumulator 
    else factorial(n - 1, n * accumulator)
}

factorial(10000)  // ✅ Works - no stack overflow
```

**Requirements for tailrec:**
1. Function must call itself as the last operation
2. Can't use try-catch-finally with recursive call
3. Only valid on member/extension functions

**More Examples:**

```kotlin
// Sum of list
tailrec fun sum(list: List<Int>, accumulator: Int = 0): Int {
    return if (list.isEmpty()) accumulator
    else sum(list.drop(1), accumulator + list.first())
}

// Find element
tailrec fun <T> findElement(
    list: List<T>, 
    predicate: (T) -> Boolean,
    index: Int = 0
): T? {
    return when {
        index >= list.size -> null
        predicate(list[index]) -> list[index]
        else -> findElement(list, predicate, index + 1)
    }
}

// GCD (Greatest Common Divisor)
tailrec fun gcd(a: Int, b: Int): Int {
    return if (b == 0) a else gcd(b, a % b)
}
```

---

### Operator Overloading

Kotlin allows overloading operators for custom types.

```kotlin
data class Point(val x: Int, val y: Int) {
    // Arithmetic operators
    operator fun plus(other: Point) = Point(x + other.x, y + other.y)
    operator fun minus(other: Point) = Point(x - other.x, y - other.y)
    operator fun times(scalar: Int) = Point(x * scalar, y * scalar)
    
    // Unary operators
    operator fun unaryMinus() = Point(-x, -y)
    operator fun inc() = Point(x + 1, y + 1)
    
    // Comparison
    operator fun compareTo(other: Point): Int {
        return (x + y).compareTo(other.x + other.y)
    }
}

// Usage
val p1 = Point(1, 2)
val p2 = Point(3, 4)

val p3 = p1 + p2        // Point(4, 6)
val p4 = p1 * 2         // Point(2, 4)
val p5 = -p1            // Point(-1, -2)
println(p1 < p2)        // true

// Index access operator
class Matrix(private val data: Array<IntArray>) {
    operator fun get(row: Int, col: Int): Int {
        return data[row][col]
    }
    
    operator fun set(row: Int, col: Int, value: Int) {
        data[row][col] = value
    }
}

val matrix = Matrix(arrayOf(intArrayOf(1, 2), intArrayOf(3, 4)))
println(matrix[0, 1])   // 2
matrix[1, 0] = 10

// Contains operator
class Range(private val start: Int, private val end: Int) {
    operator fun contains(value: Int): Boolean {
        return value in start..end
    }
}

val range = Range(1, 10)
println(5 in range)     // true

// Invoke operator - call object like function
class Greeter(private val greeting: String) {
    operator fun invoke(name: String) = "$greeting, $name!"
}

val greet = Greeter("Hello")
println(greet("John"))  // Hello, John!
```

**Common Operators:**

| Expression | Translates to |
|------------|---------------|
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)` |
| `a..b` | `a.rangeTo(b)` |
| `a in b` | `b.contains(a)` |
| `a[i]` | `a.get(i)` |
| `a[i] = b` | `a.set(i, b)` |
| `a()` | `a.invoke()` |
| `a > b` | `a.compareTo(b) > 0` |

---

### Destructuring Declarations

```kotlin
// Data class - automatic componentN functions
data class User(val name: String, val age: Int, val email: String)

val user = User("John", 30, "john@example.com")
val (name, age, email) = user
println("$name, $age, $email")

// Destructure in lambda
val users = listOf(
    User("John", 30, "john@example.com"),
    User("Jane", 25, "jane@example.com")
)
users.forEach { (name, age) ->
    println("$name is $age years old")
}

// Map entries
val map = mapOf("a" to 1, "b" to 2)
for ((key, value) in map) {
    println("$key -> $value")
}

// Custom destructuring
class Rational(val numerator: Int, val denominator: Int) {
    operator fun component1() = numerator
    operator fun component2() = denominator
}

val (num, den) = Rational(3, 4)
println("$num/$den")  // 3/4

// Underscore for unused variables
val (name, _, email) = user  // age is ignored
```

---

### Reflection

Reflection allows inspecting code structure at runtime.

```kotlin
import kotlin.reflect.*

// Class reference
val kClass: KClass<String> = String::class
println(kClass.simpleName)  // String

// Property reference
class Person(var name: String, val age: Int)

val nameProperty: KMutableProperty1<Person, String> = Person::name
val person = Person("John", 30)
println(nameProperty.get(person))  // John
nameProperty.set(person, "Jane")
println(person.name)  // Jane

// Function reference
fun isEven(n: Int) = n % 2 == 0

val function: (Int) -> Boolean = ::isEven
println(function(4))  // true

// Member function reference
val toUpperCase: (String) -> String = String::toUpperCase
println(toUpperCase("hello"))  // HELLO

// Checking type at runtime
fun processValue(value: Any) {
    when (value::class) {
        String::class -> println("String: $value")
        Int::class -> println("Int: $value")
        else -> println("Other: $value")
    }
}

// Get all properties
val properties: Collection<KProperty1<Person, *>> = Person::class.memberProperties
properties.forEach { prop ->
    println("${prop.name}: ${prop.get(person)}")
}
```

**When to Use Reflection:**
- Serialization/deserialization frameworks
- Dependency injection
- Testing frameworks
- Dynamic proxies
- Plugin systems

**⚠️ Caution:** Reflection is slower and breaks compile-time safety. Use sparingly.

---

### Flow (Cold Streams)

> See [Coroutines.md](coroutines/Coroutines.md) for comprehensive Flow coverage.

**Basic Flow:**

```kotlin
// Create a flow
fun numberFlow(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(100)
        emit(i)
    }
}

// Collect flow
lifecycleScope.launch {
    numberFlow().collect { value ->
        println(value)
    }
}

// Flow operators
numberFlow()
    .filter { it % 2 == 0 }
    .map { it * 2 }
    .collect { println(it) }  // 4, 8
```

**StateFlow vs SharedFlow:**

```kotlin
// StateFlow - hot, stateful, replay = 1
class ViewModel {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun updateState(newState: UiState) {
        _uiState.value = newState
    }
}

// SharedFlow - hot, configurable replay
private val _events = MutableSharedFlow<Event>(replay = 0)
val events: SharedFlow<Event> = _events.asSharedFlow()

suspend fun sendEvent(event: Event) {
    _events.emit(event)
}
```

**Key Differences:**

| Feature | Flow | StateFlow | SharedFlow |
|---------|------|-----------|------------|
| Cold/Hot | Cold | Hot | Hot |
| Replay | No | 1 (latest) | Configurable |
| Initial value | No | Yes | No |
| Use case | One-time operations | UI state | Events |

---

### Annotations

**Built-in Annotations:**

```kotlin
// @JvmStatic - generate static method for Java
object Utils {
    @JvmStatic
    fun doSomething() { }
}
// Java: Utils.doSomething();

// @JvmField - expose as public field
class Config {
    @JvmField
    val timeout = 5000
}
// Java: config.timeout

// @JvmOverloads - generate overloaded methods
class User @JvmOverloads constructor(
    val name: String,
    val age: Int = 0,
    val email: String = ""
)
// Generates 3 constructors for Java

// @Suppress - suppress warnings
@Suppress("UNCHECKED_CAST")
val list = obj as List<String>

// @Deprecated
@Deprecated("Use newFunction instead", ReplaceWith("newFunction()"))
fun oldFunction() { }
```

**Custom Annotations:**

```kotlin
// Define annotation
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class Loggable(val level: String = "INFO")

@Target(AnnotationTarget.PROPERTY)
@Retention(AnnotationRetention.RUNTIME)
annotation class JsonField(val name: String)

// Use annotations
@Loggable(level = "DEBUG")
class UserService {
    @Loggable
    fun createUser() { }
}

data class User(
    @JsonField("user_name") val name: String,
    @JsonField("user_age") val age: Int
)

// Process annotations with reflection
fun processAnnotations(obj: Any) {
    val kClass = obj::class
    
    // Class annotations
    kClass.annotations.forEach { annotation ->
        if (annotation is Loggable) {
            println("Logging level: ${annotation.level}")
        }
    }
    
    // Property annotations
    kClass.memberProperties.forEach { prop ->
        prop.annotations.forEach { annotation ->
            if (annotation is JsonField) {
                println("${prop.name} -> ${annotation.name}")
            }
        }
    }
}
```

---

## Common Kotlin Interview Questions

*Basic to Intermediate Level*

### 1. What is the difference between val, var, and const val?

**Answer:**

| Feature | val | var | const val |
|---------|-----|-----|-----------|
| Mutability | Read-only reference | Mutable reference | Compile-time constant |
| Can be reassigned | No | Yes | No |
| Value computation | Runtime | Runtime | Compile-time |
| Where declared | Anywhere | Anywhere | Top-level or object only |
| Type restrictions | Any | Any | Primitives & String only |

```kotlin
// val - immutable reference (can't reassign)
val name = "John"
// name = "Jane"  // ❌ Error

// var - mutable reference
var age = 25
age = 26  // ✅ OK

// const val - compile-time constant
const val MAX_SIZE = 100

// Note: val doesn't make object immutable
val list = mutableListOf(1, 2, 3)
list.add(4)  // ✅ OK
```

---

### 2. Explain Kotlin's null safety features

**Answer:**

Kotlin's type system distinguishes nullable and non-nullable types at compile time.

```kotlin
// Non-nullable type
var name: String = "John"

// Nullable type
var nullableName: String? = "John"
nullableName = null  // ✅ OK

// Safe call operator
val length = nullableName?.length

// Elvis operator
val len = nullableName?.length ?: 0

// Not-null assertion
val l = nullableName!!.length

// Safe cast
val str: String? = obj as? String
```

---

### 3. What is the difference between == and ===?

**Answer:**

```kotlin
// == (Structural equality) - calls equals()
val a = "Hello"
val b = "Hello"
println(a == b)  // true

// === (Referential equality) - compares references
val list1 = listOf(1, 2, 3)
val list2 = listOf(1, 2, 3)
println(list1 == list2)   // true - same content
println(list1 === list2)  // false - different objects
```

**Summary:**
- `==` → checks value equality
- `===` → checks reference equality

---

### 4. Explain extension functions vs member functions

**Answer:**

```kotlin
// Member function - defined inside class
class Person(val name: String) {
    fun greet() {
        println("Hello, I'm $name")
    }
}

// Extension function - defined outside class
fun Person.introduce() {
    println("My name is $name")
}

// Key differences:
// 1. Extensions cannot access private members
// 2. Extensions are resolved statically
// 3. Extensions can be defined for final classes
```

---

### 5. What are data classes? What methods do they generate?

**Answer:**

Data classes automatically generate boilerplate code.

```kotlin
data class User(val name: String, val age: Int)

// Generated methods:
// 1. equals() / hashCode()
val user1 = User("John", 30)
val user2 = User("John", 30)
println(user1 == user2)  // true

// 2. toString()
println(user1)  // User(name=John, age=30)

// 3. copy()
val user3 = user1.copy(age = 31)

// 4. componentN() for destructuring
val (name, age) = user1
```

---

### 6. Explain sealed classes vs enum classes

**Answer:**

**Enum Classes:**
```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}
// All instances known at compile time
```

**Sealed Classes:**
```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    object Loading : Result()
}

// Exhaustive when expressions
fun handle(result: Result) = when (result) {
    is Result.Success -> println(result.data)
    is Result.Error -> println(result.message)
    is Result.Loading -> println("Loading...")
}
```

---

### 7. What is the difference between object and companion object?

**Answer:**

**Object Declaration (Singleton):**
```kotlin
object DatabaseManager {
    fun connect() { }
}
DatabaseManager.connect()
```

**Companion Object:**
```kotlin
class User {
    companion object {
        fun create() = User()
        const val MAX_AGE = 150
    }
}
val user = User.create()
```

---

### 8. Explain let, also, run, apply, with

**Answer:**

| Function | Context | Return | Use Case |
|----------|---------|--------|----------|
| let | it | Lambda result | Null checks |
| also | it | Object itself | Side effects |
| run | this | Lambda result | Config + compute |
| apply | this | Object itself | Initialization |
| with | this | Lambda result | Group calls |

```kotlin
// let - null check
name?.let { println(it) }

// also - logging
val list = mutableListOf(1, 2, 3)
    .also { println("Created: $it") }

// apply - configuration
val person = Person().apply {
    name = "John"
    age = 30
}
```

---

### 9. What are inline functions? Why use them?

**Answer:**

Inline functions copy function body to call site, eliminating lambda object creation.

```kotlin
inline fun <T> measureTime(block: () -> T): T {
    val start = System.currentTimeMillis()
    val result = block()
    println("Took: ${System.currentTimeMillis() - start}ms")
    return result
}

// Benefits:
// 1. No lambda object allocation
// 2. Non-local returns allowed
// 3. Reified type parameters

inline fun <reified T> isInstance(value: Any) = value is T
```

---

### 10. Explain lateinit vs lazy

**Answer:**

**lateinit:**
```kotlin
// For var properties
class MyActivity {
    private lateinit var database: Database
    
    override fun onCreate() {
        database = DatabaseFactory.create()
    }
    
    fun query() {
        if (::database.isInitialized) {
            database.query()
        }
    }
}
```

**lazy:**
```kotlin
// For val properties
class DataProcessor {
    private val resource: Resource by lazy {
        Resource()  // Computed only once
    }
}
```

| Feature | lateinit | lazy |
|---------|----------|------|
| Property type | var | val |
| Initialization | Manual | Automatic (first access) |
| Thread safety | No | Yes (by default) |

---

### 11. What is the difference between List and MutableList?

**Answer:**

```kotlin
// List - read-only interface
val readOnlyList: List<Int> = listOf(1, 2, 3)
// readOnlyList.add(4)  // ❌ No add method

// MutableList - read-write interface
val mutableList: MutableList<Int> = mutableListOf(1, 2, 3)
mutableList.add(4)  // ✅ Can modify
```

**Important:** List is just a read-only view, not truly immutable.

---

### 12. Explain type inference and smart casts

**Answer:**

**Type Inference:**
```kotlin
val name = "John"  // Inferred as String
val age = 30       // Inferred as Int
fun add(a: Int, b: Int) = a + b  // Return type inferred
```

**Smart Casts:**
```kotlin
fun process(obj: Any) {
    if (obj is String) {
        println(obj.length)  // Auto cast to String
    }
}

// Works with when
fun describe(obj: Any) = when (obj) {
    is Int -> obj + 1
    is String -> obj.length
    else -> "Unknown"
}
```

---

### 13. What are higher-order functions?

**Answer:**

Higher-order functions take functions as parameters or return functions.

```kotlin
// Function as parameter
fun calculate(a: Int, b: Int, op: (Int, Int) -> Int): Int {
    return op(a, b)
}

val sum = calculate(5, 3) { x, y -> x + y }  // 8

// Function as return value
fun getOperation(type: String): (Int, Int) -> Int {
    return when (type) {
        "add" -> { a, b -> a + b }
        "multiply" -> { a, b -> a * b }
        else -> { a, b -> 0 }
    }
}
```

---

### 14. Explain lambda vs anonymous functions

**Answer:**

```kotlin
// Lambda
val lambda = { a: Int, b: Int -> a + b }

// Anonymous function
val anonymous = fun(a: Int, b: Int): Int {
    return a + b
}

// Key difference: return behavior
fun processNumbers(numbers: List<Int>) {
    numbers.forEach { num ->
        if (num < 0) {
            return  // Returns from processNumbers (non-local)
        }
    }
}

fun processNumbers2(numbers: List<Int>) {
    numbers.forEach(fun(num) {
        if (num < 0) {
            return  // Returns from anonymous function only (local)
        }
    })
}
```

---

### 15. What is tail recursion?

**Answer:**

Tail recursion optimizes recursive calls to prevent stack overflow.

```kotlin
// Regular recursion - stack overflow for large n
fun factorial(n: Int): Long {
    return if (n <= 1) 1 else n * factorial(n - 1)
}

// Tail recursive - optimized to loop
tailrec fun factorial(n: Long, acc: Long = 1): Long {
    return if (n <= 1) acc 
    else factorial(n - 1, n * acc)
}

// Requirements:
// 1. Recursive call must be last operation
// 2. Must be member/extension function
```

---


## Advanced Kotlin Interview Questions
*Senior to Lead Level*

### 1. Explain variance (invariant, covariant, contravariant) with practical examples

**Answer:**

Variance determines how subtyping between generic types relates to subtyping of their type parameters.

**Invariance (default):**
```kotlin
class Box<T>(var item: T)

// Box<Dog> is NOT a subtype of Box<Animal>
open class Animal
class Dog : Animal()

val dogBox: Box<Dog> = Box(Dog())
// val animalBox: Box<Animal> = dogBox  // ❌ Type mismatch
```

**Covariance (out):**
```kotlin
// Producer - only produces T, never consumes
interface Producer<out T> {
    fun produce(): T  // T only in "out" position
    // fun consume(item: T)  // ❌ Would not compile
}

class AnimalProducer : Producer<Animal> {
    override fun produce() = Animal()
}

class DogProducer : Producer<Dog> {
    override fun produce() = Dog()
}

// ✅ Works because Producer is covariant
val dogProducer: Producer<Dog> = DogProducer()
val animalProducer: Producer<Animal> = dogProducer  // Safe upcasting

// Real-world example: List<T> is covariant
val dogs: List<Dog> = listOf(Dog(), Dog())
val animals: List<Animal> = dogs  // ✅ Safe - List is read-only
```

**Contravariance (in):**
```kotlin
// Consumer - only consumes T, never produces
interface Consumer<in T> {
    fun consume(item: T)  // T only in "in" position
    // fun produce(): T  // ❌ Would not compile
}

class AnimalConsumer : Consumer<Animal> {
    override fun consume(item: Animal) {
        println("Consuming animal: ${item.javaClass.simpleName}")
    }
}

// ✅ Works because Consumer is contravariant
val animalConsumer: Consumer<Animal> = AnimalConsumer()
val dogConsumer: Consumer<Dog> = animalConsumer  // Safe - Animal can consume Dog
dogConsumer.consume(Dog())  // Works fine

// Real-world example: Comparator<T> is contravariant
val animalComparator: Comparator<Animal> = Comparator { a1, a2 -> 
    a1.name.compareTo(a2.name) 
}
val dogComparator: Comparator<Dog> = animalComparator  // ✅ Safe
```

**Mnemonic:**
- **out** (covariant): Producer - only produces/returns T (T in **out** position)
- **in** (contravariant): Consumer - only consumes/accepts T (T in **in** position)
- **No variance** (invariant): Both producer and consumer

---

### 2. What are reified type parameters? Why are they useful?

**Answer:**

Reified type parameters preserve type information at runtime in inline functions.

**Problem without reified:**
```kotlin
// Type erased at runtime
fun <T> isInstanceOf(value: Any): Boolean {
    // return value is T  // ❌ Cannot check erased type
    return false
}
```

**Solution with reified:**
```kotlin
inline fun <reified T> isInstanceOf(value: Any): Boolean {
    return value is T  // ✅ Type available at runtime
}

println(isInstanceOf<String>("Hello"))  // true
println(isInstanceOf<Int>("Hello"))     // false
```

**Practical Examples:**

```kotlin
// 1. Generic JSON parsing
inline fun <reified T> String.parseJson(): T {
    return Gson().fromJson(this, T::class.java)
}

val user: User = jsonString.parseJson<User>()

// 2. Type-safe intent extras
inline fun <reified T : Activity> Context.startActivity() {
    startActivity(Intent(this, T::class.java))
}

context.startActivity<MainActivity>()

// 3. Type-safe casting
inline fun <reified T> Any.safeCast(): T? {
    return this as? T
}

val str: String? = obj.safeCast<String>()

// 4. Filter by type
inline fun <reified T> List<Any>.filterIsInstance(): List<T> {
    return filter { it is T }.map { it as T }
}

val mixed: List<Any> = listOf(1, "hello", 2, "world")
val strings = mixed.filterIsInstance<String>()  // ["hello", "world"]

// 5. ViewModel factory
inline fun <reified VM : ViewModel> Fragment.viewModels(): Lazy<VM> {
    return lazy {
        ViewModelProvider(this)[VM::class.java]
    }
}

val viewModel: MyViewModel by viewModels()
```

**Requirements:**
- Function must be `inline`
- Type parameter marked with `reified`
- Cannot be used on classes or non-inline functions

---

### 3. Explain contracts in Kotlin - what problems do they solve?

**Answer:**

Contracts help the compiler understand function behavior for better smart casts and flow analysis.

**Problem without contracts:**
```kotlin
fun String?.isValid(): Boolean {
    return this != null && this.isNotEmpty()
}

fun process(input: String?) {
    if (input.isValid()) {
        // println(input.length)  // ❌ Smart cast not possible
    }
}
```

**Solution with contracts:**
```kotlin
import kotlin.contracts.*

fun String?.isValid(): Boolean {
    contract {
        returns(true) implies (this@isValid != null)
    }
    return this != null && this.isNotEmpty()
}

fun processWithContract(input: String?) {
    if (input.isValid()) {
        println(input.length)  // ✅ Smart cast works!
    }
}
```

**Contract Types:**

```kotlin
// 1. Returns implies
fun requireNotNull(value: Any?) {
    contract {
        returns() implies (value != null)
    }
    if (value == null) throw IllegalArgumentException()
}

// 2. Returns value
fun Boolean.isTrueAndNotNull(): Boolean {
    contract {
        returns(true) implies (this@isTrueAndNotNull)
    }
    return this == true
}

// 3. CallsInPlace
fun runOnce(block: () -> Unit) {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
}

fun initializeValue() {
    val value: Int
    runOnce {
        value = 42  // ✅ Compiler knows block called exactly once
    }
    println(value)  // ✅ value is definitely initialized
}

// InvocationKind options:
// - EXACTLY_ONCE: Called exactly once
// - AT_MOST_ONCE: Called 0 or 1 times
// - AT_LEAST_ONCE: Called 1 or more times
// - UNKNOWN: May be called any number of times
```

**Built-in functions with contracts:**
- `require()`, `check()`, `assert()`
- `requireNotNull()`, `checkNotNull()`
- `let`, `also`, `run`, `apply`, `with`

**Limitations:**
- Only in top-level functions or member functions
- Must be first statement in function
- Cannot have complex logic
- Currently experimental/restricted feature

---

### 4. How would you build a type-safe DSL in Kotlin?

**Answer:**

Type-safe DSLs use lambda with receivers, scope control, and builder patterns.

**Example: SQL Query DSL**

```kotlin
// DSL Definition
class Query {
    private val fields = mutableListOf<String>()
    private var tableName: String = ""
    private val conditions = mutableListOf<String>()
    
    fun select(vararg columns: String) {
        fields.addAll(columns)
    }
    
    fun from(table: String) {
        tableName = table
    }
    
    fun where(condition: String) {
        conditions.add(condition)
    }
    
    fun build(): String {
        val select = if (fields.isEmpty()) "*" else fields.joinToString(", ")
        val where = if (conditions.isEmpty()) "" 
                    else " WHERE " + conditions.joinToString(" AND ")
        return "SELECT $select FROM $tableName$where"
    }
}

fun query(init: Query.() -> Unit): String {
    val query = Query()
    query.init()
    return query.build()
}

// Usage
val sql = query {
    select("name", "age")
    from("users")
    where("age > 18")
    where("status = 'active'")
}
// Result: SELECT name, age FROM users WHERE age > 18 AND status = 'active'
```

**Advanced Example: HTML DSL**

```kotlin
@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
abstract class Tag(val name: String) {
    private val children = mutableListOf<Tag>()
    private val attributes = mutableMapOf<String, String>()
    
    protected fun <T : Tag> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }
    
    operator fun String.unaryPlus() {
        children.add(TextTag(this))
    }
    
    fun attribute(name: String, value: String) {
        attributes[name] = value
    }
    
    override fun toString(): String {
        val attrs = attributes.entries.joinToString("") { """ ${it.key}="${it.value}"""" }
        val content = children.joinToString("")
        return "<$name$attrs>$content</$name>"
    }
}

class HTML : Tag("html") {
    fun head(init: Head.() -> Unit) = initTag(Head(), init)
    fun body(init: Body.() -> Unit) = initTag(Body(), init)
}

class Head : Tag("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title : Tag("title")

class Body : Tag("body") {
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
    fun div(init: Div.() -> Unit) = initTag(Div(), init)
}

class H1 : Tag("h1")
class P : Tag("p")
class Div : Tag("div")

class TextTag(val text: String) : Tag("") {
    override fun toString() = text
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}

// Usage
val page = html {
    head {
        title {
            +"My Website"
        }
    }
    body {
        h1 {
            +"Welcome"
            attribute("class", "header")
        }
        div {
            p {
                +"First paragraph"
            }
            p {
                +"Second paragraph"
            }
        }
    }
}
```

**Key Techniques:**
1. **Lambda with receiver** - `init: Type.() -> Unit`
2. **@DslMarker** - prevents implicit receiver mixing
3. **Extension functions** - adds DSL operations
4. **Operator overloading** - `unaryPlus` for string content
5. **Scope control** - type-safe nesting

---

### 5. Explain the difference between suspend and blocking functions

**Answer:**

**Blocking Function:**
```kotlin
// Blocks the thread - thread cannot do other work
fun blockingDownload(): String {
    Thread.sleep(1000)  // Thread is blocked
    return "Data"
}

// On main thread - UI freezes
blockingDownload()  // ❌ ANR risk
```

**Suspend Function:**
```kotlin
// Suspends coroutine - thread is free for other work
suspend fun suspendingDownload(): String {
    delay(1000)  // Coroutine suspended, thread is free
    return "Data"
}

// On main thread - UI remains responsive
lifecycleScope.launch {
    suspendingDownload()  // ✅ No ANR
}
```

**Key Differences:**

| Feature | Blocking | Suspend |
|---------|----------|---------|
| Thread | Blocked | Free to do other work |
| Context | Any | Must be in coroutine |
| Performance | Expensive (thread waiting) | Efficient (thread reused) |
| UI Impact | Freezes UI | UI remains responsive |
| Cancellation | Hard to cancel | Easy to cancel |

**Practical Example:**

```kotlin
// Blocking approach - BAD
fun loadUserBlocking(): User {
    val user = networkService.getUser()  // Blocks thread
    val posts = networkService.getPosts()  // Blocks thread
    return user
}

// Suspending approach - GOOD
suspend fun loadUserSuspending(): User = withContext(Dispatchers.IO) {
    val user = networkService.getUser()  // Suspends, doesn't block
    val posts = networkService.getPosts()  // Suspends, doesn't block
    user
}

// Parallel execution with suspend
suspend fun loadDataParallel() = coroutineScope {
    val userDeferred = async { getUser() }      // Start immediately
    val postsDeferred = async { getPosts() }    // Start immediately
    UserData(
        user = userDeferred.await(),            // Wait for both
        posts = postsDeferred.await()
    )
}
```

**When to use each:**
- **Blocking**: Never in production Android code (except legacy Java interop)
- **Suspend**: Always for async operations in Kotlin

---

### 6. What is the difference between Flow and LiveData?

**Answer:**

| Feature | Flow | LiveData |
|---------|------|----------|
| Platform | Kotlin only | Android only |
| Lifecycle-aware | No (needs lifecycleScope) | Yes (built-in) |
| Cold/Hot | Cold (default) | Hot |
| Operators | Extensive (map, filter, etc.) | Limited |
| Thread safety | Yes | Yes |
| Backpressure | Yes (buffer, conflate) | No |
| Initial value | No | Optional |

**Flow:**
```kotlin
// Cold stream - emits only when collected
fun getUpdates(): Flow<Data> = flow {
    while (true) {
        emit(fetchData())
        delay(1000)
    }
}

// Usage in ViewModel
class MyViewModel : ViewModel() {
    val dataFlow: Flow<Data> = repository.getUpdates()
        .map { transform(it) }
        .filter { it.isValid }
        .flowOn(Dispatchers.IO)
}

// Collection in Fragment/Activity
lifecycleScope.launch {
    viewModel.dataFlow.collect { data ->
        updateUI(data)
    }
}

// StateFlow - hot, stateful
private val _uiState = MutableStateFlow(UiState.Loading)
val uiState: StateFlow<UiState> = _uiState.asStateFlow()
```

**LiveData:**
```kotlin
// Hot stream - always active
class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<Data>()
    val data: LiveData<Data> = _data
    
    fun loadData() {
        viewModelScope.launch {
            _data.value = repository.getData()
        }
    }
}

// Observation (lifecycle-aware automatically)
viewModel.data.observe(viewLifecycleOwner) { data ->
    updateUI(data)
}
```

**When to use:**
- **Flow**: New Kotlin projects, complex transformations, backpressure needed
- **LiveData**: Existing projects, simple UI updates, tight Android integration
- **StateFlow**: Replacement for LiveData in new code

**Migration pattern:**
```kotlin
// Convert Flow to LiveData
val liveData: LiveData<Data> = flow.asLiveData()

// Convert LiveData to Flow
val flow: Flow<Data> = liveData.asFlow()
```

---

### 7. Explain operator overloading - when should you use it?

**Answer:**

Operator overloading allows custom behavior for standard operators.

**Example: Vector Class**

```kotlin
data class Vector2D(val x: Double, val y: Double) {
    // Arithmetic operators
    operator fun plus(other: Vector2D) = 
        Vector2D(x + other.x, y + other.y)
    
    operator fun minus(other: Vector2D) = 
        Vector2D(x - other.x, y - other.y)
    
    operator fun times(scalar: Double) = 
        Vector2D(x * scalar, y * scalar)
    
    operator fun div(scalar: Double) = 
        Vector2D(x / scalar, y / scalar)
    
    // Unary operators
    operator fun unaryMinus() = Vector2D(-x, -y)
    
    operator fun inc() = Vector2D(x + 1, y + 1)
    
    // Comparison
    operator fun compareTo(other: Vector2D): Int {
        val thisMag = Math.sqrt(x * x + y * y)
        val otherMag = Math.sqrt(other.x * other.x + other.y * other.y)
        return thisMag.compareTo(otherMag)
    }
    
    // Index access
    operator fun get(index: Int) = when (index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException()
    }
}

// Usage - looks like mathematical notation
val v1 = Vector2D(3.0, 4.0)
val v2 = Vector2D(1.0, 2.0)

val sum = v1 + v2          // Vector2D(4.0, 6.0)
val diff = v1 - v2         // Vector2D(2.0, 2.0)
val scaled = v1 * 2.0      // Vector2D(6.0, 8.0)
val negated = -v1          // Vector2D(-3.0, -4.0)
println(v1 > v2)           // true (magnitude comparison)
println(v1[0])             // 3.0
```

**Money class example:**

```kotlin
data class Money(val amount: BigDecimal, val currency: String) {
    operator fun plus(other: Money): Money {
        require(currency == other.currency) { "Currency mismatch" }
        return Money(amount + other.amount, currency)
    }
    
    operator fun times(factor: Int): Money {
        return Money(amount * BigDecimal(factor), currency)
    }
    
    operator fun compareTo(other: Money): Int {
        require(currency == other.currency) { "Currency mismatch" }
        return amount.compareTo(other.amount)
    }
}

val price = Money(BigDecimal("19.99"), "USD")
val total = price * 3  // $59.97
```

**When to use operator overloading:**

✅ **Good use cases:**
- Mathematical types (Vector, Matrix, Complex numbers)
- Collection-like types (custom lists, maps)
- Domain types with natural operators (Money, Date ranges)
- DSLs where operators enhance readability

❌ **Bad use cases:**
- Operators with non-obvious meaning
- Side effects in operators
- Violating principle of least surprise

**Guidelines:**
- Keep operators intuitive
- Maintain mathematical properties (commutativity, associativity where expected)
- Don't overuse - prefer named methods for complex operations

---

### 8. How do sealed classes help with state machines?

**Answer:**

Sealed classes provide exhaustive when expressions and type-safe state representation.

**Example: Network Request State Machine**

```kotlin
sealed class NetworkState {
    object Idle : NetworkState()
    object Loading : NetworkState()
    data class Success(val data: String) : NetworkState()
    data class Error(val exception: Exception) : NetworkState()
}

class NetworkManager {
    private val _state = MutableStateFlow<NetworkState>(NetworkState.Idle)
    val state: StateFlow<NetworkState> = _state.asStateFlow()
    
    suspend fun fetchData() {
        _state.value = NetworkState.Loading
        try {
            val data = api.getData()
            _state.value = NetworkState.Success(data)
        } catch (e: Exception) {
            _state.value = NetworkState.Error(e)
        }
    }
}

// UI handles all cases exhaustively
fun renderState(state: NetworkState) = when (state) {
    NetworkState.Idle -> showIdleUI()
    NetworkState.Loading -> showLoading()
    is NetworkState.Success -> showData(state.data)
    is NetworkState.Error -> showError(state.exception.message)
    // No else needed - compiler ensures all cases handled
}
```

**Example: Authentication State Machine**

```kotlin
sealed class AuthState {
    object Unauthenticated : AuthState()
    data class Authenticating(val username: String) : AuthState()
    data class Authenticated(val user: User, val token: String) : AuthState()
    data class Failed(val reason: String) : AuthState()
    
    // State transitions
    fun canTransitionTo(newState: AuthState): Boolean = when (this) {
        is Unauthenticated -> newState is Authenticating
        is Authenticating -> newState is Authenticated || newState is Failed
        is Authenticated -> newState is Unauthenticated
        is Failed -> newState is Authenticating || newState is Unauthenticated
    }
}

class AuthStateMachine {
    private var currentState: AuthState = AuthState.Unauthenticated
    
    fun transitionTo(newState: AuthState) {
        require(currentState.canTransitionTo(newState)) {
            "Invalid transition from $currentState to $newState"
        }
        currentState = newState
    }
}
```

**Benefits:**
1. **Exhaustive checking** - compiler ensures all cases handled
2. **Type safety** - each state has its own data
3. **No invalid states** - only defined states possible
4. **Clear transitions** - state changes are explicit
5. **Easy to extend** - add new states without breaking existing code

---

### 9. Explain the difference between object and class with singleton pattern

**Answer:**

**Using object (Kotlin):**

```kotlin
// Thread-safe, lazy initialization by default
object DatabaseManager {
    private var connection: Connection? = null
    
    fun connect() {
        if (connection == null) {
            connection = createConnection()
        }
    }
    
    fun query(sql: String): Result {
        return connection!!.execute(sql)
    }
}

// Usage - no instantiation needed
DatabaseManager.connect()
val result = DatabaseManager.query("SELECT * FROM users")

// Compiled to:
// public final class DatabaseManager {
//     public static final DatabaseManager INSTANCE;
//     static { INSTANCE = new DatabaseManager(); }
//     private DatabaseManager() {}
// }
```

**Using class (manual singleton):**

```kotlin
// Not thread-safe
class DatabaseManager private constructor() {
    companion object {
        private var instance: DatabaseManager? = null
        
        fun getInstance(): DatabaseManager {
            if (instance == null) {
                instance = DatabaseManager()
            }
            return instance!!
        }
    }
}

// Thread-safe (double-checked locking)
class DatabaseManager private constructor() {
    companion object {
        @Volatile
        private var instance: DatabaseManager? = null
        
        fun getInstance(): DatabaseManager {
            return instance ?: synchronized(this) {
                instance ?: DatabaseManager().also { instance = it }
            }
        }
    }
}

// Thread-safe (lazy delegate)
class DatabaseManager private constructor() {
    companion object {
        val instance: DatabaseManager by lazy {
            DatabaseManager()
        }
    }
}
```

**Comparison:**

| Feature | object | class singleton |
|---------|--------|-----------------|
| Thread safety | Automatic | Manual |
| Initialization | Lazy (on first access) | Depends on implementation |
| Boilerplate | None | Significant |
| Parameters | Cannot pass | Can pass to constructor |
| Testing | Hard to mock | Can inject dependency |

**When to use each:**

**Use `object`:**
```kotlin
// Simple singleton without dependencies
object Logger {
    fun log(message: String) {
        println("[LOG] $message")
    }
}
```

**Use `class` singleton:**
```kotlin
// When you need dependency injection
class Repository private constructor(
    private val database: Database
) {
    companion object {
        @Volatile
        private var instance: Repository? = null
        
        fun getInstance(database: Database): Repository {
            return instance ?: synchronized(this) {
                instance ?: Repository(database).also { instance = it }
            }
        }
    }
}

// Better: Use dependency injection framework
class Repository(private val database: Database) {
    // Dagger/Hilt manages singleton
}
```

---

### 10. What are property delegates? Implement a custom delegate

**Answer:**

Property delegates allow reusing property behavior across classes.

**Custom Delegate Examples:**

**1. Logging Delegate:**

```kotlin
class LoggingDelegate<T>(private var value: T) {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        println("[GET] ${property.name} = $value")
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        println("[SET] ${property.name}: $value -> $newValue")
        value = newValue
    }
}

class User {
    var name: String by LoggingDelegate("Unknown")
    var age: Int by LoggingDelegate(0)
}

val user = User()
user.name = "John"  // [SET] name: Unknown -> John
println(user.name)   // [GET] name = John
```

**2. Validation Delegate:**

```kotlin
class ValidatedDelegate<T>(
    private var value: T,
    private val validator: (T) -> Boolean
) {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        require(validator(newValue)) {
            "${property.name} validation failed for value: $newValue"
        }
        value = newValue
    }
}

class Person {
    var age: Int by ValidatedDelegate(0) { it >= 0 && it <= 150 }
    var email: String by ValidatedDelegate("") { it.contains("@") }
}

val person = Person()
person.age = 25      // ✅ OK
// person.age = -5   // ❌ IllegalArgumentException
person.email = "john@example.com"  // ✅ OK
// person.email = "invalid"        // ❌ IllegalArgumentException
```

**3. Cached/Memoized Delegate:**

```kotlin
class CachedDelegate<T>(
    private val loader: () -> T,
    private val expirationMs: Long = 60000
) {
    private var value: T? = null
    private var lastLoadTime: Long = 0
    
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        val now = System.currentTimeMillis()
        if (value == null || (now - lastLoadTime) > expirationMs) {
            value = loader()
            lastLoadTime = now
            println("Cache refreshed for ${property.name}")
        }
        return value!!
    }
}

// Read-only property delegate
operator fun <T> CachedDelegate<T>.provideDelegate(
    thisRef: Any?,
    prop: KProperty<*>
): CachedDelegate<T> {
    return this
}

class DataRepository {
    val users: List<User> by CachedDelegate({ fetchUsersFromNetwork() }, 5000)
    val settings: Settings by CachedDelegate({ fetchSettings() })
}
```

**4. Thread-Safe Delegate:**

```kotlin
class SynchronizedDelegate<T>(initialValue: T) {
    private var value: T = initialValue
    private val lock = Any()
    
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        synchronized(lock) {
            return value
        }
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: T) {
        synchronized(lock) {
            value = newValue
        }
    }
}

class SharedResource {
    var counter: Int by SynchronizedDelegate(0)
}
```

**5. Bound Delegate (limits value range):**

```kotlin
class BoundDelegate(
    initialValue: Int,
    private val min: Int,
    private val max: Int
) {
    private var value: Int = initialValue.coerceIn(min, max)
    
    operator fun getValue(thisRef: Any?, property: KProperty<*>): Int {
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: Int) {
        value = newValue.coerceIn(min, max)
    }
}

class Volume {
    var level: Int by BoundDelegate(50, 0, 100)
}

val volume = Volume()
volume.level = 150  // Clamped to 100
println(volume.level)  // 100
```

**provideDelegate for initialization:**

```kotlin
class ResourceDelegate<T>(
    private val resourceName: String,
    private val loader: (String) -> T
) {
    private var value: T? = null
    
    operator fun provideDelegate(
        thisRef: Any?,
        prop: KProperty<*>
    ): ResourceDelegate<T> {
        // Validate at property declaration time
        println("Initializing ${prop.name} with resource $resourceName")
        return this
    }
    
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        if (value == null) {
            value = loader(resourceName)
        }
        return value!!
    }
}

class MyClass {
    val image: Bitmap by ResourceDelegate("logo.png") { name ->
        loadBitmap(name)
    }
}
```

---


### 11. Explain inline, crossinline, and noinline

**Answer:**

**inline:**
```kotlin
// Basic inline - copies lambda code to call site
inline fun measureTime(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()  // Inlined at call site
    println("Took: ${System.currentTimeMillis() - start}ms")
}

// Allows non-local returns
fun findFirst(list: List<Int>): Int? {
    list.forEach { item ->  // forEach is inline
        if (item > 5) {
            return item  // ✅ Returns from findFirst
        }
    }
    return null
}
```

**noinline:**
```kotlin
// Lambda stored or passed to non-inline function
inline fun process(
    inlineAction: () -> Unit,
    noinline storedAction: () -> Unit
) {
    inlineAction()  // Inlined
    saveForLater(storedAction)  // Not inlined - can be stored
}

fun saveForLater(action: () -> Unit) {
    // Store for later execution
}
```

**crossinline:**
```kotlin
// Lambda used in non-local context but can't use non-local returns
inline fun runAsync(crossinline action: () -> Unit) {
    thread {
        action()  // Used in different context (thread)
    }
}

fun test() {
    runAsync {
        println("Running")
        // return  // ❌ Compilation error - crossinline prevents non-local return
    }
}

// Why crossinline is needed:
inline fun problematic(action: () -> Unit) {
    thread {
        action()  // If action has return, it returns from outer function, not thread
    }
}
```

**Comparison:**

| Modifier | Inlined | Non-local returns | Can be stored | Used in other contexts |
|----------|---------|-------------------|---------------|------------------------|
| inline | ✅ | ✅ | ❌ | ❌ |
| noinline | ❌ | ❌ | ✅ | ✅ |
| crossinline | ✅ | ❌ | ❌ | ✅ |

**Real-world examples:**

```kotlin
// Standard library: let (inline, non-local returns allowed)
inline fun <T, R> T.let(block: (T) -> R): R = block(this)

// Standard library: also (crossinline, can't use non-local returns)
inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}

// Custom: mixed lambdas
inline fun complexOperation(
    inlineTransform: (Int) -> Int,
    crossinline validate: (Int) -> Boolean,
    noinline onError: (Exception) -> Unit
) {
    try {
        val value = inlineTransform(42)
        thread {
            if (!validate(value)) {
                onError(ValidationException())
            }
        }
    } catch (e: Exception) {
        onError(e)
    }
}
```

---

### 12. How would you implement a thread-safe singleton in Kotlin?

**Answer:**

**1. Object Declaration (Recommended):**
```kotlin
// Thread-safe by default, lazy initialization
object DatabaseManager {
    private val connection by lazy {
        createConnection()
    }
    
    fun query(sql: String) = connection.execute(sql)
}
```

**2. Lazy Delegate (for class with parameters):**
```kotlin
class Repository private constructor(private val api: Api) {
    companion object {
        @Volatile
        private var instance: Repository? = null
        
        fun getInstance(api: Api): Repository {
            return instance ?: synchronized(this) {
                instance ?: Repository(api).also { instance = it }
            }
        }
    }
}
```

**3. Double-Checked Locking:**
```kotlin
class DatabaseManager private constructor(private val config: Config) {
    companion object {
        @Volatile
        private var INSTANCE: DatabaseManager? = null
        
        fun getInstance(config: Config): DatabaseManager {
            // First check (no locking)
            val currentInstance = INSTANCE
            if (currentInstance != null) {
                return currentInstance
            }
            
            // Second check (with locking)
            return synchronized(this) {
                val newInstance = INSTANCE
                if (newInstance != null) {
                    newInstance
                } else {
                    DatabaseManager(config).also {
                        INSTANCE = it
                    }
                }
            }
        }
    }
}
```

**4. Lazy Delegate (thread-safe):**
```kotlin
class Repository private constructor(private val database: Database) {
    companion object {
        private var database: Database? = null
        
        val instance: Repository by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
            Repository(database ?: throw IllegalStateException("Initialize first"))
        }
        
        fun initialize(db: Database) {
            database = db
        }
    }
}

// Usage
Repository.initialize(databaseInstance)
val repo = Repository.instance
```

**5. Holder Pattern:**
```kotlin
class CacheManager private constructor() {
    companion object {
        fun getInstance(): CacheManager = Holder.instance
    }
    
    private object Holder {
        val instance = CacheManager()
    }
}
```

**Testing Considerations:**

```kotlin
// For testability - use interface + DI
interface UserRepository {
    fun getUsers(): List<User>
}

class UserRepositoryImpl(private val api: Api) : UserRepository {
    override fun getUsers() = api.fetchUsers()
}

// In production: use Dagger/Hilt
@Singleton
class UserRepositoryImpl @Inject constructor(
    private val api: Api
) : UserRepository

// In tests: easily mock
val mockRepo = mockk<UserRepository>()
```

**Best Practices:**
- Prefer `object` for simple singletons
- Use DI framework (Dagger, Hilt) for complex apps
- Avoid singletons when possible (hard to test)
- Use `@Volatile` with double-checked locking
- Consider lifecycle-scoped instances instead

---

### 13. How does Kotlin handle SAM (Single Abstract Method) conversions?

**Answer:**

SAM conversion allows using lambdas where functional interfaces are expected.

**Java Interfaces:**

```kotlin
// Java interface
public interface OnClickListener {
    void onClick(View view);
}

// Kotlin usage - SAM conversion
button.setOnClickListener { view ->
    println("Clicked!")
}

// Equivalent to:
button.setOnClickListener(object : OnClickListener {
    override fun onClick(view: View) {
        println("Clicked!")
    }
})
```

**Kotlin fun interfaces (1.4+):**

```kotlin
// Kotlin functional interface
fun interface StringProcessor {
    fun process(input: String): String
}

// SAM conversion works
fun transform(input: String, processor: StringProcessor): String {
    return processor.process(input)
}

// Lambda usage
val result = transform("hello") { it.uppercase() }

// Multiple function calls - separate objects created
transform("a") { it.uppercase() }
transform("b") { it.uppercase() }
```

**Limitations:**

```kotlin
// ❌ SAM conversion doesn't work for Kotlin interfaces (pre-1.4)
interface Processor {
    fun process(input: String): String
}

fun transform(input: String, processor: Processor): String {
    return processor.process(input)
}

// Must use object expression
transform("hello", object : Processor {
    override fun process(input: String) = input.uppercase()
})

// ✅ Works with fun interface
fun interface Processor {
    fun process(input: String): String
}

transform("hello") { it.uppercase() }
```

**Performance Considerations:**

```kotlin
// Lambda creates new object each time
repeat(1000) {
    button.setOnClickListener { view ->
        println("Clicked!")
    }
}
// Creates 1000 listener objects

// Reuse same listener
val listener = View.OnClickListener { view ->
    println("Clicked!")
}
repeat(1000) {
    button.setOnClickListener(listener)
}
// Creates 1 listener object
```

**Suspend SAM:**

```kotlin
// Suspend functional interface
fun interface SuspendProcessor {
    suspend fun process(input: String): String
}

suspend fun transform(input: String, processor: SuspendProcessor): String {
    return processor.process(input)
}

// Usage
lifecycleScope.launch {
    val result = transform("hello") { input ->
        delay(100)
        input.uppercase()
    }
}
```

---

### 14. What is the difference between internal and private visibility?

**Answer:**

**private:**
- Visible only within the same file/class
- Most restrictive visibility

**internal:**
- Visible within the same module
- Module = Gradle module, Maven project, IntelliJ module

```kotlin
// File: User.kt
private class InternalUser(val id: String)  // Visible only in this file

internal class PublicUser(val id: String)   // Visible in same module

class UserManager {
    private val secret = "secret"           // Only in UserManager
    internal val apiKey = "key"             // Visible in module
    
    private fun encrypt() { }               // Only in UserManager
    internal fun authenticate() { }         // Visible in module
}

// File: Main.kt (same module)
fun test() {
    // val user = InternalUser("123")       // ❌ Not visible
    val user = PublicUser("123")            // ✅ Visible (same module)
    
    val manager = UserManager()
    // manager.secret                       // ❌ Not visible
    manager.apiKey                          // ✅ Visible (same module)
}

// Different module
fun testFromOtherModule() {
    // val user = PublicUser("123")         // ❌ Not visible (different module)
}
```

**Use Cases:**

```kotlin
// private - implementation details
class Repository {
    private val cache = mutableMapOf<String, User>()
    
    private fun validateUser(user: User): Boolean {
        return user.email.contains("@")
    }
    
    fun saveUser(user: User) {
        if (validateUser(user)) {
            cache[user.id] = user
        }
    }
}

// internal - module API (not public)
internal class NetworkClient {
    internal fun makeRequest(url: String): Response {
        // Implementation
    }
}

// Used across module, but not exposed to external consumers
internal object Analytics {
    internal fun logEvent(event: String) {
        // Implementation
    }
}
```

**Module Structure:**

```
app (module)
├── User.kt
│   internal class User            // Visible in app module
│   private class UserImpl         // Visible only in User.kt
│
library (module)
├── Api.kt
│   internal class ApiClient       // Visible in library module only
│   private fun connect()          // Visible only in Api.kt
```

**Visibility Summary:**

| Modifier | Class/Top-level | Class member |
|----------|-----------------|--------------|
| private | Same file | Same class |
| internal | Same module | Same module |
| protected | N/A | Same class + subclasses |
| public | Everywhere | Everywhere |

---

### 15. Explain @JvmStatic, @JvmOverloads, @JvmField

**Answer:**

These annotations improve Java interoperability.

**@JvmStatic:**

```kotlin
// Kotlin
class User {
    companion object {
        @JvmStatic
        fun create(name: String) = User(name)
        
        fun validate(name: String) = name.isNotEmpty()
    }
}

object Logger {
    @JvmStatic
    fun log(message: String) {
        println(message)
    }
}
```

```java
// Java usage
// With @JvmStatic
User user = User.create("John");        // ✅ Static method
Logger.log("Message");                  // ✅ Static method

// Without @JvmStatic
boolean valid = User.Companion.validate("John");  // ❌ Must use Companion
```

**@JvmOverloads:**

```kotlin
// Kotlin - generates multiple constructors/methods
class User @JvmOverloads constructor(
    val name: String,
    val age: Int = 0,
    val email: String = ""
)

@JvmOverloads
fun greet(name: String, greeting: String = "Hello") {
    println("$greeting, $name")
}
```

```java
// Java usage - multiple overloads generated
User user1 = new User("John");
User user2 = new User("John", 30);
User user3 = new User("John", 30, "john@example.com");

greet("John");                    // Uses default greeting
greet("John", "Hi");              // Custom greeting
```

**Generated Java code:**
```java
public User(String name) {
    this(name, 0, "");
}

public User(String name, int age) {
    this(name, age, "");
}

public User(String name, int age, String email) {
    // Full constructor
}
```

**@JvmField:**

```kotlin
// Kotlin
class Config {
    @JvmField
    val timeout = 5000
    
    val retries = 3  // Without @JvmField
    
    companion object {
        @JvmField
        val MAX_SIZE = 1000
    }
}
```

```java
// Java usage
// With @JvmField - direct field access
int timeout = config.timeout;           // ✅ Direct field access
int maxSize = Config.MAX_SIZE;          // ✅ Static field

// Without @JvmField - must use getter
int retries = config.getRetries();      // ❌ Must use getter
```

**Real-World Example:**

```kotlin
// Kotlin Library
object Constants {
    @JvmField
    val API_URL = "https://api.example.com"
    
    @JvmStatic
    fun getVersion() = "1.0.0"
}

class Builder @JvmOverloads constructor(
    val name: String,
    val age: Int = 0,
    val email: String = ""
) {
    companion object {
        @JvmStatic
        fun create(name: String): Builder {
            return Builder(name)
        }
    }
}
```

```java
// Java Consumer
String url = Constants.API_URL;              // Direct access
String version = Constants.getVersion();     // Static method

Builder builder1 = new Builder("John");      // Default parameters
Builder builder2 = Builder.create("Jane");   // Static factory
```

**When to Use:**
- **@JvmStatic**: Methods that should be static in Java
- **@JvmOverloads**: Functions/constructors with default parameters used from Java
- **@JvmField**: Properties that should be fields (not getters) in Java

---

### 16. How does Kotlin handle checked exceptions differently than Java?

**Answer:**

Kotlin doesn't have checked exceptions - all exceptions are unchecked.

**Java:**
```java
// Java - must declare or catch checked exceptions
public String readFile() throws IOException {  // Must declare
    return Files.readString(Path.of("file.txt"));
}

public void process() {
    try {
        readFile();  // Must catch or declare
    } catch (IOException e) {
        // Handle
    }
}
```

**Kotlin:**
```kotlin
// Kotlin - no checked exceptions
fun readFile(): String {
    return File("file.txt").readText()  // May throw IOException, no declaration needed
}

fun process() {
    readFile()  // No forced try-catch
}

// Can still catch if needed
fun processSafely() {
    try {
        readFile()
    } catch (e: IOException) {
        // Handle
    }
}
```

**Interop with Java:**

```kotlin
// Kotlin function called from Java
fun save(data: String) {
    throw IOException("Failed")  // Unchecked in Kotlin
}
```

```java
// Java code
public void caller() {
    save("data");  // Java compiler doesn't force try-catch
}
```

**@Throws Annotation:**

```kotlin
// Tell Java about potential exceptions
@Throws(IOException::class, SQLException::class)
fun saveToDatabase(data: String) {
    // Implementation
}
```

```java
// Java code - now aware of exceptions
public void caller() {
    try {
        saveToDatabase("data");  // Java compiler now requires handling
    } catch (IOException | SQLException e) {
        // Handle
    }
}
```

**Rationale:**

```kotlin
// Java's checked exceptions often lead to:

// 1. Empty catch blocks
try {
    operation();
} catch (Exception e) {
    // Ignored
}

// 2. Wrapped exceptions
try {
    operation();
} catch (CheckedException e) {
    throw new RuntimeException(e);  // Just wrapping
}

// 3. Large try-catch blocks
try {
    operation1();
    operation2();
    operation3();
} catch (Exception1 | Exception2 | Exception3 e) {
    // Handle all
}

// Kotlin's approach: handle exceptions when meaningful
fun process() {
    val data = readFile()  // Crash if fails - expected behavior
    processData(data)
}

fun processWithRecovery() {
    val data = try {
        readFile()
    } catch (e: IOException) {
        getDefaultData()  // Meaningful recovery
    }
    processData(data)
}
```

**Best Practices:**

```kotlin
// 1. Use Result type for expected failures
fun loadData(): Result<Data> {
    return try {
        Result.success(fetchData())
    } catch (e: Exception) {
        Result.failure(e)
    }
}

// 2. Use sealed classes for errors
sealed class DataResult {
    data class Success(val data: Data) : DataResult()
    data class Error(val exception: Exception) : DataResult()
}

// 3. Document exceptions in KDoc
/**
 * Loads user data from the network.
 * @throws IOException if network request fails
 * @throws JsonException if parsing fails
 */
fun loadUser(): User {
    // Implementation
}
```

---

### 17. Explain the difference between coroutineScope and CoroutineScope

**Answer:**

**coroutineScope (lowercase) - Suspend function:**

```kotlin
// Creates a scope and suspends until all children complete
suspend fun loadData() = coroutineScope {
    val user = async { fetchUser() }
    val posts = async { fetchPosts() }
    
    UserData(user.await(), posts.await())
}  // Suspends until all async complete

// Used for parallel decomposition
suspend fun processMultiple() = coroutineScope {
    launch { process1() }
    launch { process2() }
    launch { process3() }
}  // Returns when all complete
```

**CoroutineScope (uppercase) - Interface:**

```kotlin
// Creates a long-lived scope
class Repository {
    private val scope = CoroutineScope(
        SupervisorJob() + Dispatchers.IO
    )
    
    fun loadData() {
        scope.launch {
            // Coroutine runs in scope
        }
    }
    
    fun cleanup() {
        scope.cancel()  // Cancel all coroutines
    }
}
```

**Key Differences:**

| Feature | coroutineScope | CoroutineScope |
|---------|----------------|----------------|
| Type | Suspend function | Interface |
| Lifetime | Until children complete | Until manually cancelled |
| Usage | Parallel decomposition | Long-lived operations |
| Exception handling | Propagates to caller | Needs handler |
| When to use | Inside suspend functions | Class-level scope |

**Practical Examples:**

```kotlin
// coroutineScope - structured concurrency
suspend fun fetchAllData(): Data = coroutineScope {
    val users = async { api.getUsers() }
    val posts = async { api.getPosts() }
    val comments = async { api.getComments() }
    
    Data(
        users = users.await(),
        posts = posts.await(),
        comments = comments.await()
    )
}  // If any fails, all are cancelled

// CoroutineScope - lifecycle-bound
class ViewModel : ViewModel() {
    private val viewModelScope = CoroutineScope(
        SupervisorJob() + Dispatchers.Main
    )
    
    fun loadData() {
        viewModelScope.launch {
            // Long-running work
        }
    }
    
    override fun onCleared() {
        viewModelScope.cancel()
    }
}

// Combining both
class Repository(private val scope: CoroutineScope) {
    fun getData() = scope.launch {
        // Parallel decomposition within scope
        val result = coroutineScope {
            val a = async { fetchA() }
            val b = async { fetchB() }
            combine(a.await(), b.await())
        }
        processResult(result)
    }
}
```

**Exception Handling:**

```kotlin
// coroutineScope - exception cancels all siblings
suspend fun process() = coroutineScope {
    launch { task1() }  // If this fails
    launch { task2() }  // This is cancelled
    launch { task3() }  // This is cancelled too
}

// supervisorScope - exceptions don't cancel siblings
suspend fun processIndependent() = supervisorScope {
    launch { task1() }  // If this fails
    launch { task2() }  // This continues
    launch { task3() }  // This continues too
}

// CoroutineScope with SupervisorJob
class Worker {
    private val scope = CoroutineScope(SupervisorJob())
    
    fun start() {
        scope.launch { task1() }  // Independent
        scope.launch { task2() }  // Independent
    }
}
```

---

### 18. Explain the difference between tailrec and regular recursion with performance implications

**Answer:**

**Regular Recursion:**

```kotlin
// Stack frames accumulate
fun factorial(n: Int): Long {
    return if (n <= 1) 1 
    else n * factorial(n - 1)
}

// Stack trace for factorial(4):
// factorial(4) = 4 * factorial(3)
//   factorial(3) = 3 * factorial(2)
//     factorial(2) = 2 * factorial(1)
//       factorial(1) = 1

// Memory: O(n) stack frames
// factorial(10000)  // ❌ StackOverflowError
```

**Tail Recursion:**

```kotlin
// Compiler optimizes to loop
tailrec fun factorial(n: Long, accumulator: Long = 1): Long {
    return if (n <= 1) accumulator
    else factorial(n - 1, n * accumulator)
}

// Compiled to (approximately):
fun factorial(n: Long, accumulator: Long = 1): Long {
    var currentN = n
    var currentAcc = accumulator
    while (currentN > 1) {
        currentAcc = currentN * currentAcc
        currentN = currentN - 1
    }
    return currentAcc
}

// Memory: O(1) - no stack frames
// factorial(10000)  // ✅ Works
```

**Performance Comparison:**

```kotlin
// Benchmark
fun benchmarkFactorial() {
    // Regular recursion
    val time1 = measureTimeMillis {
        regularFactorial(5000)  // May crash
    }
    
    // Tail recursion
    val time2 = measureTimeMillis {
        tailrecFactorial(5000)  // Fast and safe
    }
    
    println("Regular: $time1ms")
    println("Tailrec: $time2ms (typically faster)")
}
```

**Requirements for tailrec:**

```kotlin
// ✅ Valid - recursive call is last operation
tailrec fun sum(n: Int, acc: Int = 0): Int {
    return if (n == 0) acc else sum(n - 1, acc + n)
}

// ❌ Invalid - operation after recursive call
fun factorial(n: Int): Int {
    return if (n <= 1) 1 
    else n * factorial(n - 1)  // Multiplication happens after call
}

// ❌ Invalid - multiple recursive calls
fun fibonacci(n: Int): Int {
    return if (n <= 1) n
    else fibonacci(n - 1) + fibonacci(n - 2)  // Two calls
}
```

**Converting to Tail Recursion:**

```kotlin
// Regular → Tailrec: Fibonacci
// Regular (exponential time)
fun fibonacci(n: Int): Int {
    return if (n <= 1) n
    else fibonacci(n - 1) + fibonacci(n - 2)
}

// Tailrec (linear time)
tailrec fun fibonacci(n: Int, a: Long = 0, b: Long = 1): Long {
    return when (n) {
        0 -> a
        1 -> b
        else -> fibonacci(n - 1, b, a + b)
    }
}

// Regular → Tailrec: List sum
// Regular
fun sum(list: List<Int>): Int {
    return if (list.isEmpty()) 0
    else list.first() + sum(list.drop(1))
}

// Tailrec
tailrec fun sum(list: List<Int>, acc: Int = 0): Int {
    return if (list.isEmpty()) acc
    else sum(list.drop(1), acc + list.first())
}
```

**Performance Metrics:**

| Aspect | Regular Recursion | Tail Recursion |
|--------|-------------------|----------------|
| Stack space | O(n) | O(1) |
| Speed | Slower (function calls) | Faster (loop) |
| Max depth | ~10,000 calls | Unlimited |
| Debugging | Easier (stack trace) | Harder (loop) |

---


---

## Quick Reference: Kotlin for DSA
*20-Minute Revision Guide*

### Essential Syntax

```kotlin
// Variables
val immutable = 10              // Read-only
var mutable = 20                // Mutable
const val CONSTANT = 100        // Compile-time constant

// Functions
fun add(a: Int, b: Int): Int = a + b
fun greet(name: String = "Guest") = "Hello, $name"

// Loops
for (i in 0..10) { }           // 0 to 10 inclusive
for (i in 0 until 10) { }      // 0 to 9
for (i in 10 downTo 0) { }     // 10 to 0
for (i in 0..10 step 2) { }    // 0, 2, 4, 6, 8, 10
repeat(5) { }                  // Execute 5 times

// Conditions
val max = if (a > b) a else b
when (x) {
    in 1..10 -> println("1-10")
    else -> println("Other")
}
```

---

### Collections Cheat Sheet

#### Array Operations

```kotlin
// Creation
val arr = arrayOf(1, 2, 3, 4, 5)
val intArr = intArrayOf(1, 2, 3)
val arr2D = Array(n) { IntArray(m) }

// Access
arr[0]                          // Get
arr[0] = 10                     // Set
arr.size                        // Length
arr.indices                     // 0 until size
arr.lastIndex                   // size - 1

// Common operations
arr.sort()                      // In-place sort
val sorted = arr.sorted()       // New sorted list
arr.reverse()                   // In-place reverse
arr.sum()                       // Sum of elements
arr.max()                       // Maximum
arr.min()                       // Minimum
arr.average()                   // Average
```

#### List Operations

```kotlin
// Creation
val list = listOf(1, 2, 3)              // Immutable
val mutableList = mutableListOf(1, 2, 3) // Mutable

// Add/Remove (MutableList only)
mutableList.add(4)                      // Add to end
mutableList.add(0, 5)                   // Add at index
mutableList.remove(3)                   // Remove element
mutableList.removeAt(0)                 // Remove at index
mutableList.clear()                     // Remove all

// Access
list[0]                                 // Get
list.first()                            // First element
list.last()                             // Last element
list.getOrNull(10)                      // Safe get

// Search
list.indexOf(3)                         // Index of element (-1 if not found)
list.contains(3)                        // Check existence
3 in list                               // Same as contains
list.binarySearch(3)                    // Binary search (must be sorted)

// Sublist
list.subList(1, 3)                      // Elements from index 1 to 2
list.take(3)                            // First 3 elements
list.drop(2)                            // Skip first 2 elements
list.slice(1..3)                        // Elements at indices 1, 2, 3
```

#### Common Collection Functions

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)

// Transform
numbers.map { it * 2 }                  // [2, 4, 6, 8, 10, 12]
numbers.filter { it % 2 == 0 }          // [2, 4, 6]
numbers.filterIndexed { i, v -> i % 2 == 0 }

// Aggregate
numbers.sum()                           // 21
numbers.count()                         // 6
numbers.count { it > 3 }                // 3
numbers.maxOrNull()                     // 6
numbers.minOrNull()                     // 1
numbers.average()                       // 3.5
numbers.reduce { acc, n -> acc + n }    // 21
numbers.fold(10) { acc, n -> acc + n }  // 31

// Search
numbers.find { it > 3 }                 // 4 (first match)
numbers.findLast { it > 3 }             // 6 (last match)
numbers.any { it > 5 }                  // true
numbers.all { it > 0 }                  // true
numbers.none { it < 0 }                 // true

// Group
numbers.groupBy { it % 2 }              // {1=[1,3,5], 0=[2,4,6]}
numbers.partition { it % 2 == 0 }       // Pair([2,4,6], [1,3,5])
numbers.chunked(2)                      // [[1,2], [3,4], [5,6]]
numbers.windowed(3)                     // [[1,2,3], [2,3,4], [3,4,5], [4,5,6]]

// Other
numbers.distinct()                      // Remove duplicates
numbers.sorted()                        // Sort ascending
numbers.sortedDescending()              // Sort descending
numbers.reversed()                      // Reverse
numbers.shuffled()                      // Random order
numbers.zip(listOf("a", "b"))           // [(1,a), (2,b)]
listOf(listOf(1, 2), listOf(3)).flatten() // [1, 2, 3]
```

#### Map Operations

```kotlin
// Creation
val map = mapOf("a" to 1, "b" to 2)
val mutableMap = mutableMapOf<String, Int>()

// Add/Update
mutableMap["c"] = 3                     // Put
mutableMap.put("d", 4)                  // Put (returns old value)
mutableMap.putIfAbsent("e", 5)          // Only if key absent

// Access
map["a"]                                // Get (returns null if absent)
map.getValue("a")                       // Get (throws if absent)
map.getOrDefault("z", 0)                // Get with default
map.getOrElse("z") { 0 }                // Get with lambda default

// Remove
mutableMap.remove("a")                  // Remove by key
mutableMap.clear()                      // Remove all

// Iterate
for ((key, value) in map) {
    println("$key -> $value")
}
map.forEach { (k, v) -> println("$k -> $v") }

// Keys/Values
map.keys                                // Set of keys
map.values                              // Collection of values
map.entries                             // Set of entries

// Check
map.containsKey("a")                    // Check key
map.containsValue(1)                    // Check value
"a" in map                              // Same as containsKey
```

#### Set Operations

```kotlin
// Creation
val set = setOf(1, 2, 3, 2)             // [1, 2, 3] - no duplicates
val mutableSet = mutableSetOf<Int>()

// Add/Remove
mutableSet.add(1)                       // Add element
mutableSet.remove(1)                    // Remove element
mutableSet.addAll(listOf(2, 3))         // Add multiple

// Operations
val set1 = setOf(1, 2, 3)
val set2 = setOf(3, 4, 5)

set1.union(set2)                        // [1, 2, 3, 4, 5]
set1.intersect(set2)                    // [3]
set1.subtract(set2)                     // [1, 2]
```

---

### String Operations

```kotlin
// Creation
val str = "Hello World"
val multiline = """
    Line 1
    Line 2
""".trimIndent()

// Access
str[0]                                  // 'H'
str.first()                             // 'H'
str.last()                              // 'd'
str.length                              // 11

// Substring
str.substring(0, 5)                     // "Hello"
str.take(5)                             // "Hello"
str.drop(6)                             // "World"
str.slice(0..4)                         // "Hello"

// Search
str.indexOf("World")                    // 6
str.contains("World")                   // true
"World" in str                          // true
str.startsWith("Hello")                 // true
str.endsWith("World")                   // true

// Transform
str.uppercase()                         // "HELLO WORLD"
str.lowercase()                         // "hello world"
str.capitalize()                        // "Hello World" (deprecated)
str.replaceFirstChar { it.uppercase() } // "Hello World"
str.replace("World", "Kotlin")          // "Hello Kotlin"
str.trim()                              // Remove whitespace
str.split(" ")                          // ["Hello", "World"]
str.reversed()                          // "dlroW olleH"

// Check
str.isEmpty()                           // false
str.isNotEmpty()                        // true
str.isBlank()                           // false (has non-whitespace)
str.all { it.isLetter() }               // false (has space)
str.any { it.isDigit() }                // false

// Convert
str.toIntOrNull()                       // null (not a number)
"123".toInt()                           // 123
"123".toLong()                          // 123L
str.toCharArray()                       // Array of chars
str.toList()                            // List of chars
```

---

### Sorting & Searching

```kotlin
val list = mutableListOf(5, 2, 8, 1, 9)

// Sorting
list.sort()                             // In-place ascending
list.sortDescending()                   // In-place descending
val sorted = list.sorted()              // New sorted list
val reversed = list.sortedDescending()  // New reversed sorted list

// Custom comparator
list.sortWith(compareBy { it })         // Ascending
list.sortWith(compareByDescending { it }) // Descending

// Complex sorting
data class Person(val name: String, val age: Int)
val people = listOf(Person("John", 30), Person("Jane", 25))

people.sortedBy { it.age }              // Sort by age
people.sortedByDescending { it.age }    // Sort by age desc
people.sortedWith(compareBy({ it.age }, { it.name })) // Multiple criteria

// Binary search (list must be sorted)
val index = list.binarySearch(5)        // Returns index or -insertionPoint - 1
val found = list.binarySearch(5) >= 0   // Check if found
```

---

### Common DSA Patterns

#### Two Pointers

```kotlin
// Find pair with sum
fun twoSum(arr: IntArray, target: Int): Pair<Int, Int>? {
    var left = 0
    var right = arr.lastIndex
    
    while (left < right) {
        val sum = arr[left] + arr[right]
        when {
            sum == target -> return Pair(left, right)
            sum < target -> left++
            else -> right--
        }
    }
    return null
}
```

#### Sliding Window

```kotlin
// Maximum sum of k consecutive elements
fun maxSum(arr: IntArray, k: Int): Int {
    var windowSum = arr.take(k).sum()
    var maxSum = windowSum
    
    for (i in k until arr.size) {
        windowSum = windowSum - arr[i - k] + arr[i]
        maxSum = maxOf(maxSum, windowSum)
    }
    return maxSum
}
```

#### Prefix Sum

```kotlin
// Range sum queries
class PrefixSum(arr: IntArray) {
    private val prefix = IntArray(arr.size + 1)
    
    init {
        for (i in arr.indices) {
            prefix[i + 1] = prefix[i] + arr[i]
        }
    }
    
    fun rangeSum(left: Int, right: Int): Int {
        return prefix[right + 1] - prefix[left]
    }
}
```

#### Binary Search

```kotlin
// Find first occurrence
fun firstOccurrence(arr: IntArray, target: Int): Int {
    var left = 0
    var right = arr.lastIndex
    var result = -1
    
    while (left <= right) {
        val mid = left + (right - left) / 2
        when {
            arr[mid] == target -> {
                result = mid
                right = mid - 1  // Continue searching left
            }
            arr[mid] < target -> left = mid + 1
            else -> right = mid - 1
        }
    }
    return result
}
```

---

### Useful Functions Reference

```kotlin
// Min/Max
minOf(a, b)                             // Minimum of two
maxOf(a, b)                             // Maximum of two
minOf(a, b, c)                          // Minimum of three
list.minOrNull()                        // Minimum in collection
list.maxOrNull()                        // Maximum in collection

// Math
abs(-5)                                 // 5
Math.pow(2.0, 3.0)                      // 8.0
Math.sqrt(16.0)                         // 4.0
Math.ceil(3.2)                          // 4.0
Math.floor(3.8)                         // 3.0

// Range
(1..10)                                 // 1 to 10 inclusive
(1 until 10)                            // 1 to 9
(10 downTo 1)                           // 10 to 1
(1..10 step 2)                          // 1, 3, 5, 7, 9

// Grouping
list.groupBy { it % 2 }                 // Group by predicate
list.associateBy { it.id }              // Map by key
list.partition { it > 5 }               // Split into two lists
list.chunked(3)                         // Split into chunks

// Combinations
list.zip(other)                         // Combine two lists
list.zipWithNext()                      // Pairs of consecutive elements
list.flatten()                          // Flatten nested lists
list.flatMap { it.toList() }            // Map and flatten

// Type conversions
str.toInt()                             // String to Int
str.toIntOrNull()                       // Safe conversion
int.toString()                          // Int to String
charArray.concatToString()              // CharArray to String
"123".toCharArray()                     // String to CharArray
list.toIntArray()                       // List to IntArray
intArray.toList()                       // IntArray to List
```

---

### Quick Tips

#### Null Handling

```kotlin
val length = str?.length ?: 0           // Elvis operator
str?.let { process(it) }                // Execute if not null
val nonNull = str!!                     // Assert non-null (throws NPE)
```

#### Range Operations

```kotlin
if (x in 1..10) { }                     // Check if in range
if (x !in 1..10) { }                    // Check if not in range
for (i in 1..n) { }                     // Iterate range
(1..n).forEach { }                      // Functional iteration
```

#### Type Conversions

```kotlin
// Number conversions
int.toLong()
long.toInt()
double.toInt()
string.toIntOrNull()

// Collection conversions
list.toSet()                            // Remove duplicates
set.toList()                            // Set to List
list.toMutableList()                    // Immutable to mutable
mutableList.toList()                    // Mutable to immutable
```

#### Common Mistakes to Avoid

```kotlin
// ❌ Modifying list while iterating
for (item in list) {
    list.remove(item)  // ConcurrentModificationException
}

// ✅ Use iterator or create new list
list.removeAll { it > 5 }
val filtered = list.filter { it <= 5 }

// ❌ Comparing with ==  for reference types without equals
if (arr1 == arr2) { }  // Compares references

// ✅ Use contentEquals for arrays
if (arr1.contentEquals(arr2)) { }

// ❌ Forgetting to handle null
val length = nullable.length  // Compilation error

// ✅ Use safe calls
val length = nullable?.length ?: 0
```

---

### DSA Time Complexities

| Operation | Array | ArrayList | LinkedList | HashSet | HashMap | TreeSet/Map |
|-----------|-------|-----------|------------|---------|---------|-------------|
| Access | O(1) | O(1) | O(n) | - | O(1) | O(log n) |
| Search | O(n) | O(n) | O(n) | O(1) | O(1) | O(log n) |
| Insert | - | O(n) | O(1) | O(1) | O(1) | O(log n) |
| Delete | - | O(n) | O(1) | O(1) | O(1) | O(log n) |
| Add (end) | - | O(1)* | O(1) | O(1) | O(1) | O(log n) |

*Amortized

---

### Quick Practice Template

```kotlin
fun main() {
    // Read input
    val n = readLine()!!.toInt()
    val arr = readLine()!!.split(" ").map { it.toInt() }.toIntArray()
    
    // Process
    val result = solve(arr, n)
    
    // Output
    println(result)
}

fun solve(arr: IntArray, n: Int): Int {
    // Your solution here
    return 0
}
```

---

**End of Quick Reference**

> **Pro Tip:** Bookmark this section for quick revision before coding interviews!

---

## References

- [Official Kotlin Documentation](https://kotlinlang.org/docs/home.html)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Android Kotlin Guides](https://developer.android.com/kotlin)
- [Kotlin for Competitive Programming](https://kotlinlang.org/docs/competitive-programming.html)

---

## Summary

This guide covers:
- ✅ Kotlin Basics (1-2 hours) - Quick revision for senior engineers
- ✅ Advanced Topics (2-3 hours) - Deep dives with examples
- ✅ Common Interview Questions (1 hour) - Basic to intermediate
- ✅ Advanced Interview Questions (1-2 hours) - Senior to lead level
- ✅ Quick Reference for DSA (20 minutes) - Ultra-concise guide

**Total Reading Time:** 4-8 hours (depending on depth)

**Best Practice:** Review basics quickly, focus on advanced topics and interview questions, use DSA reference for quick lookups.

---

*Happy Learning! 🚀*

