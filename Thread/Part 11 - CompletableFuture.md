

# Why Was CompletableFuture Introduced?

We already have:

```
ExecutorService

↓

Future
```

Question:

Why wasn't Future enough?

Because Future has several limitations.

---

# Problem 1 - Future Blocks

Suppose:

```java
Future<String> future =
executor.submit(() -> {

    Thread.sleep(5000);

    return "Hello";

});
```

Later:

```java
String value = future.get();
```

If the task isn't finished:

```
Main Thread

↓

WAITING

↓

5 Seconds

↓

Gets Result
```

This is blocking.

---

# Problem 2 - Can't Chain Tasks

Suppose we want:

```
Download Employee

↓

Extract Department

↓

Send Email
```

With Future:

```java
Employee e = future.get();

Department d = ...

sendMail();
```

Everything becomes sequential.

There is no elegant way to say:

> "When this finishes, automatically do the next step."

---

# Problem 3 - Can't Combine Futures

Suppose we have:

```
Future<Employee>

Future<Salary>

Future<Project>
```

We want:

```
Wait For All

↓

Create Response
```

Future has no elegant API.

---

# Problem 4 - Poor Exception Handling

With Future:

```
ExecutionException
```

gets thrown by `get()`.

Handling asynchronous errors becomes messy.

---

# CompletableFuture

Java 8 introduced:

```
CompletableFuture
```

Think of it as:

```
Future

+

Callbacks

+

Pipelines

+

Composition

+

Exception Handling
```

---

# The Biggest Idea

Future asks:

```
Has Task Finished?

↓

Yes?

↓

Give Result
```

CompletableFuture asks:

```
When Task Finishes

↓

Automatically Do Next Step
```

This is asynchronous programming.

---

# Two Ways to Create

## runAsync()

For:

```
Runnable
```

No return value.

```java
CompletableFuture<Void> future =
CompletableFuture.runAsync(() -> {

    System.out.println("Hello");

});
```

---

## supplyAsync()

For:

```
Callable
```

Returns a value.

```java
CompletableFuture<String> future =
CompletableFuture.supplyAsync(() -> {

    return "Vinay";

});
```

This is used far more often.

---

# runAsync vs supplyAsync

| runAsync  | supplyAsync         |
| --------- | ------------------- |
| Runnable  | Callable (Supplier) |
| No Return | Returns Value       |

---

# Where Do They Run?

If you don't specify an Executor:

```java
CompletableFuture.supplyAsync(...)
```

uses:

```
ForkJoinPool.commonPool()
```

Internally.

Later we'll see how to use our own ExecutorService.

---

# Example

```java
CompletableFuture<String> future =
CompletableFuture.supplyAsync(() -> {

    System.out.println(Thread.currentThread().getName());

    return "Java";

});
```

Notice:

Main thread continues immediately.

The task executes asynchronously.

---

# join()

CompletableFuture also has:

```java
future.join();
```

Similar to:

```java
future.get();
```

Difference:

| get() | join() |
|--------|---------|
| Checked Exception | Unchecked Exception |

We'll usually use:

```
join()
```

inside CompletableFuture chains.

---

# Visual

```
Main Thread

↓

Start Async Task

↓

Continue Working

---------------------

Worker Thread

↓

Computes Result

↓

Completes Future
```

Only when we call:

```
join()

or

get()
```

do we wait.

---

# Real Example

Suppose:

```
API Request
```

Needs:

```
Employee

Salary

Projects
```

All independent.

Instead of:

```
Employee

↓

Salary

↓

Projects
```

Run all together.

```
Employee

Salary

Projects
```

Each starts asynchronously.

Huge performance improvement.

We'll combine them in the next section.

---

# Common Interview Questions

## Why was CompletableFuture introduced?

Because Future lacks task composition, chaining, callbacks, and elegant exception handling.

---

## Difference between Future and CompletableFuture?

Future only represents a pending result.

CompletableFuture also lets you build asynchronous pipelines.

---

## Difference between runAsync() and supplyAsync()?

runAsync()

No result.

supplyAsync()

Returns a result.

---

# Memory Trick

```
Future

↓

Wait

↓

Get Result

-----------------------

CompletableFuture

↓

Start Async

↓

Continue

↓

Automatically Process Result
```