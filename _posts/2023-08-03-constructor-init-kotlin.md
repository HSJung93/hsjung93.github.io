---
title: "Key Differences between constructor and init block in Kotlin"
date: 2023-08-03 19:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Data]
---

In Kotlin, both the constructor and init block are used for initializing the object of a class. However, there are key differences between them.

### Constructor

A constructor in Kotlin is a special method that is invoked when an object of a class is created. There are two types of constructors in Kotlin: primary and secondary. The primary constructor is part of the class header and cannot contain any code. The secondary constructor is part of the class body and can contain code. It's also important to note that if a class has a primary constructor, the secondary constructor needs to delegate to the primary constructor either directly or indirectly through another secondary constructor(s).

Here is an example of a class with a primary and secondary constructor:

```kotlin
class Sample(private var s : String) {
    constructor(t: String, u: String) : this(t) {
        this.s += u
    }
}

```

In this example, "T" is assigned to s from the primary constructor of the Sample class. Then the secondary constructor adds "U" to the s variable.

### Init Block

The init block in Kotlin is another way to initialize your objects. It is called after the primary constructor during object creation. The init block contains the code that is always executed after the primary constructor. If you have multiple init blocks in your class, they will be executed in the order they appear in the class definition. It's also important to note that the init blocks are executed before the secondary constructor.

Here is an example of a class with an `init` block:

```kotlin
class Sample(private var s : String) {
    init {
        s += "B"
    }
    constructor(t: String, u: String) : this(t) {
        this.s += u
    }
}

```

In this example, if you initialize the Sample class with Sample("T","U"), you will get a string response at variable s as "TBU". The "T" is assigned to s from the primary constructor of the Sample class. Then immediately the init block starts to execute; it will add "B" to the s variable. Next, it is the secondary constructor's turn; now "U" is added to the s variable to become "TBU".

In summary, the main difference is that the constructor is used to create an instance of a class while the init block is used to initialize the instance. The init block is always executed after the primary constructor and before the secondary constructor.