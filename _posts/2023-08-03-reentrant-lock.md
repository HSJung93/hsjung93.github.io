---
title: "What is Reentrant Lock"
date: 2023-08-03 19:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Data]
---

ReentrantLock is a class in the Java Concurrent package (java.util.concurrent.locks) and is an implementation of the Lock interface. It is a reentrant mutual exclusion lock, meaning a thread can acquire the same lock multiple times without causing a deadlock.

In a reentrant lock, when a thread requests a lock held by another thread, the requesting thread will be blocked. However, when a thread requests a lock held by itself, the request will succeed and the lock's counter will be incremented. This is called reentrancy.

Here is an example of using ReentrantLock:

```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantExample {
    private final ReentrantLock lock = new ReentrantLock();

    public void outer() {
        lock.lock();
        try {
            System.out.println("outer lock acquired");
            inner();
        } finally {
            lock.unlock();
            System.out.println("outer lock released");
        }
    }

    public void inner() {
        lock.lock();
        try {
            System.out.println("inner lock acquired");
        } finally {
            lock.unlock();
            System.out.println("inner lock released");
        }
    }

    public static void main(String[] args) {
        ReentrantExample example = new ReentrantExample();
        example.outer();
    }
}

```
In this example, both the outer() and inner() methods attempt to acquire the same lock. When we call the outer() method in the main() method, the lock is first acquired, and then the inner() method is called within outer(). Since the lock is reentrant, the inner() method can successfully acquire the lock without being blocked.

Compared to the built-in synchronization mechanism, ReentrantLock provides some more advanced features:

1. Interruptible lock acquisition: A thread waiting to acquire a lock can be interrupted.
2. Fairness: The lock can be set to fair, meaning the thread that has been waiting the longest time will be given priority to acquire the lock.
3. Lock surrender: A thread waiting to acquire a lock can give up waiting and do other things.
4. Multiple condition variables: ReentrantLock can have one or more condition variables. This allows for more fine-grained control of thread waiting.

In terms of implementation, ReentrantLock is a class in the JDK implemented by Java code. It provides more features than synchronized, such as interruptible lock acquisition, fair locks, multiple condition variables, etc.

You can create an instance of ReentrantLock using its constructors:

```java
ReentrantLock lock = new ReentrantLock(); // Creates an instance of ReentrantLock.
ReentrantLock fairLock = new ReentrantLock(true); // Creates an instance of ReentrantLock with the fairness policy.

```

In the first line, an instance of ReentrantLock is created. This is equivalent to using ReentrantLock(false). In the second line, an instance of ReentrantLock with the fairness policy is created. If the fairness policy is set to true, the lock is fair, meaning the thread that has been waiting for the longest time will be given priority to acquire the lock.