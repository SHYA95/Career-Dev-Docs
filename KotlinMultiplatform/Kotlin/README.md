# Kotlin

## 🚀 Overview
Kotlin is a modern, statically-typed programming language that runs on the Java Virtual Machine (JVM). Developed by JetBrains in 2011 and made open-source in 2012, Kotlin is designed to be fully interoperable with Java while offering more concise syntax, enhanced safety features, and functional programming capabilities. Since 2019, Google has designated Kotlin as the preferred language for Android development.

## 🔑 Key Features

- **Concise Syntax**: Reduces boilerplate code compared to Java
- **Null Safety**: Type system distinguishes between nullable and non-nullable references
- **Extension Functions**: Add methods to existing classes without inheritance
- **Coroutines**: Simplified asynchronous programming with lightweight threads
- **Smart Casts**: Automatic casting after type checks
- **Data Classes**: Built-in support for immutable value objects
- **Functional Programming**: First-class functions, lambdas, and higher-order functions
- **Multiplatform**: Can target JVM, JavaScript, Native, and WebAssembly

## 💡 Code Examples

### Basic Syntax
```kotlin
fun main() {
    val name = "Kotlin"
    println("Hello, $name!")
    
    // Function with expression body
    fun sum(a: Int, b: Int) = a + b
    
    // Using when expression (enhanced switch)
    val x = 1
    when (x) {
        1 -> println("x is 1")
        2 -> println("x is 2")
        else -> println("x is neither 1 nor 2")
    }
}
```

## Null Safety
```kotlin
// Nullable type
var nullableString: String? = "Hello"
nullableString = null // OK

// Non-nullable type
var nonNullString: String = "Hello"
// nonNullString = null // Compilation error

// Safe call operator
println(nullableString?.length) // Prints null if nullableString is null

// Elvis operator
val length = nullableString?.length ?: 0 // Default value if null
```
### Extension Functions
```kotlin
fun String.addExclamation(): String {
    return this + "!"
}

val message = "Hello".addExclamation() // Returns "Hello!"
```
## 🔗 Additional Resources
https://kotlinlang.org/docs/home.html
