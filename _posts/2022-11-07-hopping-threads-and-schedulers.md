---
title: Hopping Threads and Schedulers
date: 2022-11-07 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Reactor, Scheduler]
---
# The Threading Model
- Reactor operators generally are concurrent agnostic: they don’t impose a particular threading model and just run on the `Thread` on which their `onNext` method was invoked.
- data production process starts on the Thread that initiated the subscription.
- the subscribe calls are chained until a data-producing Publisher is reached (the leftmost part of the chain of operators)
- then this Publisher offers a Subscription through onSubscribe, in turn passed down the chain, requested, etc...

# The `Scheduler` abstraction
- In Reactor, a `Scheduler` is an abstraction that gives the user control about threading.
- A `Scheduler` can spawn `Worker` which are conceptually `Threads`, but are not necessarily backed by a `Thread`.
- A `Scheduler` also includes the notion of a clock, whereas the `Worker` is purely about scheduling tasks.

```java
interface Scheduler extends Disposable {
    
  Disposable schedule(Runnable task);
  Disposable schedule(Runnable task, long initialDelay, TimeUnit delayUnit);
  Disposable schedulePeriodically(Runnable task, long initialDelay, long period, TimeUnit unit);
  
  long now(TimeUnit unit);
  
  Worker createWorker();
  
  interface Worker extends Disposable {
    Disposable schedule(Runnable task);
    Disposable schedule(Runnable task, long initialDelay, TimeUnit delayUnit);
    Disposable schedulePeriodically(Runnable task, long initialDelay, long period, TimeUnit unit);
  }
}
```

## Rule of thumbs for `Schedulers` factory methods typical usage:
- `Schedulers.immediate()` can be used as a null object for when an API requires a `Scheduler` but you don’t want to change threads
- `Schedulers.single()` is for one-off tasks that can be run on a unique `ExecutorService`
- `Schedulers.parallel()` is good for CPU-intensive but short-lived tasks. It can execute N such tasks in parallel (by default `N == number of CPUs`)
- `Schedulers.elastic()` and `Schedulers.boundedElastic()` are good for more long-lived tasks (eg. blocking IO tasks). The `elastic` one spawns threads on-demand without a limit while the recently introduced `boundedElastic` does the same with a ceiling on the number of created threads.

- The `elastic` flavor is also backed by workers based on `ScheduledExecutorService`, except it creates these workers on demand and pools them.
- The `boundedElastic` flavor is very similar in concept to the `elastic` one except it places an upper bound to the number of `ScheduledExecutorService`-backed `Worker` it creates. 
- Past this point, its `createWorker()` method returns a facade `Worker` that will enqueue tasks instead of submitting them immediately. 

# Are Schedulers Always Backed by an ExecutorService?
- As we said above, no. We already saw an example actually: the `immediate() Scheduler`. This one doesn’t modify which `Thread` the code is running on.

> Going from the main to a scheduler is obviously possible, but going from an arbitrary thread to the main thread is not possible.
{: .prompt-tip }

# Applying Schedulers to Operators

## The `publishOn(Scheduler s)` operator
- Incoming signals from its source are published on the given `Scheduler`, effectively switching threads to one of that scheduler’s workers.
- every processing step that appears below this operator will execute on the new `Scheduler s`, until another operator switches again (eg. another `publishOn`).

```java
Flux.fromIterable(firstListOfUrls) //contains A, B and C
    .map(url -> blockingWebClient.get(url))
    .subscribe(body -> System.out.println(Thread.currentThread().getName + " from first list, got " + body));

Flux.fromIterable(secondListOfUrls) //contains D and E
    .map(url -> blockingWebClient.get(url))
    .subscribe(body -> System.out.prinln(Thread.currentThread().getName + " from second list, got " + body));
```

- In the above example, assuming this code is executed on the main thread, each `Flux.fromIterable` emits the content of its `List` on that same `Thread`. 
- We then use an imperative blocking web client inside a `map` to fetch the body of each `url`, which “inherits” that thread (and thus blocks it). 
- The data-consuming lambda in each `subscribe` is thus also running on the main thread.

As a consequence, all these urls are processed sequentially on the main thread:

```
main from first list, got A
main from first list, got B
main from first list, got C
main from second list, got D
main from second list, got E
```

If we introduce publishOn, we can make this code more performant, so that the Flux don’t block each other:

```java
Flux.fromIterable(firstListOfUrls) //contains A, B and C
    .publishOn(Schedulers.boundedElastic())
    .map(url -> blockingWebClient.get(url))
    .subscribe(body -> System.out.println(Thread.currentThread().getName + " from first list, got " + body));

Flux.fromIterable(secondListOfUrls) //contains D and E
    .publishOn(Schedulers.boundedElastic())
    .map(url -> blockingWebClient.get(url))
    .subscribe(body -> System.out.prinln(Thread.currentThread().getName + " from second list, got " + body));
```

Which could give us something like the following output:

```
boundedElastic-1 from first list, got A
boundedElastic-2 from second list, got D
boundedElastic-1 from first list, got B
boundedElastic-2 from second list, got E
boundedElastic-1 from first list, got C
```

- First list and second list are interleaved now
- `publishOn` could be used to offset blocking work on a separate Thread, by switching the publication of the triggers for that blocking work (the urls to fetch) on a provided `Scheduler`.
- Since the `map` operator runs on its source thread, switching that source thread by putting a `publishOn` before the `map` works as intended.

## The `subscribeOn(Scheduler s)` operator
- what if that url-fetching method was written by somebody else, and they regrettably forgot to add the `publishOn`? 
- Is there a way to influence the `Thread` upstream? In a way, there is. That’s where subscribeOn can come in handy.
- since the subscribe signal flows upward, it directly influences where the source Flux subscribes and starts generating data.
- As a consequence, it can seem to act on the parts of the reactive chain of operators upward and downward

```java
//code provided in library you have no write access to
final Flux<String> fetchUrls(List<String> urls) {
  return Flux.fromIterable(urls)
    .map(url -> blockingWebClient.get(url)); //oops!
}

//your code:
fetchUrls(A, B, C)
  .subscribeOn(Schedulers.boundedElastic())
  .subscribe(body -> System.out.println(Thread.currentThread().getName + " from first list, got " + body));

fetchUrls(D, E)
  .subscribeOn(Schedulers.boundedElastic())
  .subscribe(body -> System.out.prinln(Thread.currentThread().getName + " from second list, got " + body));
```

- Like in our second publishOn example, that code will correctly output something like:

```
boundedElastic-1 from first list, got A
boundedElastic-2 from second list, got D
boundedElastic-1 from first list, got B
boundedElastic-2 from second list, got E
boundedElastic-1 from first list, got C
```

- The `subscribe` calls are still running on the main thread, but they propagate a `subscribe` signal to their source, `subscribeOn`. 
- In turn `subscribeOn` propagates that same signal to its own source from `fetchUrls`, but on a boundedElastic `Worker`.

> It is important to distinguish the act of subscribing and the lambda passed to the `subscribe()` method. This method subscribes to its source `Flux`, but the lambda are executed at the end of processing, when the data has flown through all the steps (including steps that hop to another thread),. So the `Thread` on which the lambda is executed might be different from the subscription `Thread` , ie. the thread on which the `subscribe` method is called.
{: .prompt-info }

- And if we were the author of the `fetchUrls` library, we could make the code even more performant by letting each fetch run on its own `Worker`, by leveraging `subscribeOn`.

```java
final Flux<String> betterFetchUrls(List<String> urls) {
  return Flux.fromIterable(urls)
    .flatMap(url -> 
             //wrap the blocking call in a Mono
             Mono.fromCallable(() -> blockingWebClient.get(url))
             //ensure that Mono is subscribed in an boundedElastic Worker
             .subscribeOn(Schedulers.boundedElastic())
    ); //each individual URL fetch runs in its own thread!
}
```

# And What If I Mix the Two?
- `subscribeOn` will act throughout the subscribe phase, from bottom to top, then on the data path until it encounters a `publishOn` (or a time based operator).
- Let’s consider the following example:

```java
Flux.just("hello")
    .doOnNext(v -> System.out.println("just " + Thread.currentThread().getName()))
    .publishOn(Scheduler.boundedElastic())
    .doOnNext(v -> System.out.println("publish " + Thread.currentThread().getName()))
    .delayElements(Duration.ofMillis(500))
    .subscribeOn(Schedulers.elastic())
    .subscribe(v -> System.out.println(v + " delayed " + Thread.currentThread().getName()));
```

This will print:

```
just elastic-1
publish boundedElastic-1
hello delayed parallel-1
```

## Unpack what happened step by step:
- Here `subscribe` is called on the main thread, but subscription is rapidly switched to the `elastic` scheduler due to the `subscribeOn` immediately above.
- All the operators above it are also subscribed on `elastic`, from bottom to top.
- `just` emits its value on the `elastic` scheduler.
- the first `doOnNext` receives that value on the same thread and prints it out: "just elastic-1"
- then on the top to bottom data path, we encounter the `publishOn`: data from `doOnNext` is propagated downstream on the `boundedElastic` scheduler.
- the second `doOnNext` receives its data on `boundedElastic` and prints "publish bounderElastic-1" accordingly.
- `delayElements` is a time operator, so by default it publishes data on the `Schedulers.parallel()` scheduler.
- on the data path, `subscribeOn` does nothing but propagating signal on the same thread.
- on the data path, the lambda(s) passed to `subscribe(...)` are executed on the thread in which data signals are received, so the lambda prints "hello delayed parallel-1"


# Reference
- https://spring.io/blog/2019/12/13/flight-of-the-flux-3-hopping-threads-and-schedulers