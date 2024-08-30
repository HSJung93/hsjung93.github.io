---
title: Functional Programming in Kotlin
date: 2023-01-08 11:52:00 +0900
categories: [SlipBox, Programming]
tags: [Functional, Kotlin]
---

# 1. What is functional programming?

- Functional programming(FP) is based on a simple premise with far-reaching implications: we construct our programs using only pure functions—in other words, functions with no side effects.

- What are side effects? A function has a side effect if it does something other than simply return a result. For example:
    - Modifying a variable beyond the scope of the block where the change occurs
    - Modifying a data structure in place
    - Setting a field on an object
    - Throwing an exception or halting with an error
    - Printing to the console or reading user input 
    - Reading from or writing to a file
    - Drawing on the screen

## The benefits of FP: A simple example

```kotlin
// ASIS: A Kotlin program with side effects
class Cafe {
    fun buyCoffee(cc: CreditCard): Coffee {
        val cup = Coffee()
        cc.charge(cup.price)
        return cup
    }
}

// INPROGRESS: Adding a Payments object
class Cafe {
    fun buyCoffee(cc: CreditCard, p: Payments): Coffee {
        val cup = Coffee()
        p.charge(cc, cup.price)
        return cup
    }
}
```

- Although side effects still occur when we call p.charge(cc, cup.price), we have at least regained some testability.
- Payments can be an interface, and we can write a mock implementation of this interface suitable for testing. But that isn’t ideal either. We’re forced to make Payments an interface when a concrete class might have been fine oth- erwise, and any mock implementation will be awkward to use.
- Separate from the concern of testing, there’s another problem: it’s challenging to reuse buyCoffee.

```kotlin
// TOBE: 
class Cafe {
    fun buyCoffee(cc: CreditCard): Pair<Coffee, Charge> {
        val cup = Coffee()
        return Pair(cup, Charge(cc, cup.price))
    }
}
```

- We’ve separated the concern of `creating` a charge from the `processing` or `interpretation` of that charge. The `buyCoffee` function now returns a Charge as a value along with `Coffee`. 

```kotlin
// Data class declaration with a constructor and immutable fields
data class Charge(val cc: CreditCard, val amount: Float) {
    fun combine(other: Charge) : Charge = 
    if ( cc == other.cc )
        Charge(cc, amount + other.amount)
    else throw Exception(
        "Cannot combine charges to different cards"
    )
}
```

- But what is `Charge`? It’s a data type we just invented, containing a `CreditCard` and an `amount`, equipped with a handy `combine` function for combining charges with the same `CreditCard`.
- A handy method is also exposed that allows this `Charge` to be combined with another `Charge` instance.

```kotlin
class Cafe {
    fun buyCoffee(cc: CreditCard) : Pair<Coffee, Charge> = TODO()

    fun buyCoffees(
        cc: CreditCard,
        n: Int
    ): Pair<List<Coffee>, Charge> {
        val purchases: List<Pair<Coffee, Charge>> = List(n) { buyCoffee(cc)}
        val (coffees, charges) = purchases.unzip()
        return Pair(
            coffees,
            charges.reduce { c1, c2 -> c1.combine(C2) }
        )
    } 
}
```

- The list is initialized using the `List(n) { buyCoffee(cc) }` syntax, where `n` describes the number of coffees and `{ buyCoffee(cc) }` is a function that ini tializes each element of the list.

- An `unzip` is then used to `destructure` the list of pairs into two separate lists, each representing one side of the `Pair`.

- This is done by constructing a `Pair` of `List<Coffee>`s mapped to the combined Charges for all the Coffees in the list.