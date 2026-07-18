# Part 11B - thenCompose() vs thenCombine()

### ⚡ TL;DR (Executive Summary)
* **`thenCompose()` (Sequential / Dependent):** Used when **Task B depends on the result of Task A**. It accepts a function returning a new `CompletableFuture` and automatically flattens it (like `flatMap` in Streams). Use case: Customer ID → Fetch Orders.
* **`thenCombine()` (Parallel / Independent):** Used when **Task A and Task B are completely independent** and should run simultaneously in parallel. It combines both results once they finish using a BiFunction. Use case: Fetch Weather + Fetch News.
* **`thenCombineAsync()`:** Similar to `thenCombine()`, but explicitly schedules the final combining function on a separate pool thread rather than executing it inline inside the completing stage thread.

| Method | Dependency Type | Concurrency Mode | Output Wrapping | Core Analogy |
| :--- | :--- | :--- | :--- | :--- |
| **`thenCompose()`** | 🔗 Dependent | ➡️ Sequential | Flattens (`CompletableFuture<T>`) | `flatMap()` |
| **`thenCombine()`** | 🚀 Independent | 🔀 Parallel | Combined (`CompletableFuture<V>`) | Parallel Join |

---

# thenCompose()

Think:

```
Task A
↓
Need Result
↓
Start Task B
```

Task B **cannot start** until Task A finishes.

Example

```
Get Employee
↓
Need Employee ID
↓
Get Salary
```

Salary API needs:

```
Employee ID
```

So it must wait.

---

## Example

```java
CompletableFuture<String> future =
        CompletableFuture
                .supplyAsync(() -> "Vinay")
                .thenCompose(name ->
                        CompletableFuture.supplyAsync(() ->
                                name + " Kumar"));
System.out.println(future.join());
```

Execution

```
Task 1
↓
"Vinay"
↓
Task 2
↓
"Vinay Kumar"
```

Notice

Task 2 is another async task.
It starts only after Task 1 completes.

---

# Why not thenApply?

Suppose:

```java
.thenApply(name ->
    CompletableFuture.supplyAsync(...)
)
```

Return type becomes:

```
CompletableFuture<
    CompletableFuture<String>
>
```

Nested Future.
Ugly.
thenCompose automatically flattens it.
Think:
```
FlatMap
```
if you've used Streams.

---

# Visual

thenApply

```
Future
↓
Future
↓
Future<Future<T>>
```

---

thenCompose

```
Future
↓
Future
↓
Future<T>
```

---

# Real Example

```
Get Employee
↓
Employee ID
↓
Call Salary Service
```

Because Salary depends on Employee,
use:
```
thenCompose()
```

---

# thenCombine()

Now suppose:

```
Get Employee
```

and

```
Get Projects
```

Need each other?
No.
Completely independent.
So start both immediately.

```
Employee
Projects
```

Both execute in parallel.
Later:
```
Combine
```

---

## Example

```java
CompletableFuture<String> employee =
        CompletableFuture.supplyAsync(() -> "Vinay");
CompletableFuture<Integer> age =
        CompletableFuture.supplyAsync(() -> 25);
CompletableFuture<String> result =
        employee.thenCombine(
                age,
                (name, a) ->
                        name + " : " + a
        );
System.out.println(result.join());
```

Output

```
Vinay : 25
```

Notice

```
Employee
Age
```
started together.
Only the combining waits.

---

# Visual

thenCombine

```
Future A
---------
Future B
↓
Combine
↓
Result
```

---

# Real Spring Boot Example

Need
```
Employee Service
Salary Service
```
Independent.
Start together.

```java
CompletableFuture<Employee> employee = ...;
CompletableFuture<Salary> salary = ...;
```

Later
```
thenCombine()
↓
EmployeeDTO
```
Huge performance improvement.

---

# Difference

thenCompose

```
Task B
Depends
On Task A
```
Sequential.

---

thenCombine

```
Task A
Task B
```
Independent.
Parallel.

---

# Interview Example

Question
Need:
```
Customer
↓
Orders
```
Which method?
Orders API needs:
```
Customer ID
```
Answer
```
thenCompose()
```

---

Need:
```
Weather
News
```
Independent.
Answer
```
thenCombine()
```

---

# Common Interview Questions

## Difference between thenApply() and thenCompose()

thenApply()
Returns normal value.

thenCompose()
Returns another CompletableFuture.

---

## Difference between thenCompose() and thenCombine()

thenCompose()
Dependent async tasks.

thenCombine()
Independent async tasks.

---

# Memory Trick

```
thenCompose()
↓
Compose
↓
Future inside Future
↓
Flatten
↓
Dependent
--------------------
thenCombine()
↓
Two Futures
↓
Combine
↓
Independent
```

**`thenCombine()`**
- Combines two completed futures.
- The combining function executes in the completion flow of the previous stages.

**`thenCombineAsync()`**
- Combines two completed futures.
- The combining function is **scheduled asynchronously** on the `ForkJoinPool.commonPool()` or on a custom `Executor`.
