---
title: "crossline modifier"
date: 2023-06-13 22:00:00 +0900
categories: [Level, Junior]
tags: [Kotlin, crossline]
publish: false
---

In Kotlin, the crossinline modifier is used in function or lambda expressions to specify that the function or lambda should not allow non-local returns. This means that the function or lambda cannot use the return keyword to exit the enclosing function or lambda expression.

By default, Kotlin allows non-local returns from lambda expressions. For example, you can use the return keyword inside a lambda to exit the enclosing function or method in which the lambda is defined. However, there are situations where non-local returns are not desirable, such as when working with higher-order functions that need to enforce control flow within their body.

To address this, the crossinline modifier can be used on lambda parameters of higher-order functions to ensure that the lambda cannot use non-local returns. When a lambda parameter is marked as crossinline, the compiler enforces that the lambda can only return from itself and cannot affect the control flow of the enclosing function or method.

Here's an example to illustrate the usage of crossinline:

```kotlin
inline fun higherOrderFunction(crossinline lambda: () -> Unit) {
    val runnable = Runnable {
        lambda() // Invoking the lambda
    }
    // ...
    runnable.run() // Running the runnable
    // ...
}

fun main() {
    higherOrderFunction {
        // Some code
        // return // Error: Return is not allowed here
    }
}

```

In the example, the higherOrderFunction is an inline function that takes a lambda parameter. By marking the lambda parameter as crossinline, the compiler ensures that the lambda cannot use non-local returns, as demonstrated by the commented-out return statement inside the lambda in the main function.