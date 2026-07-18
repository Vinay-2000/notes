# Part 11D - Exception Handling

### ⚡ TL;DR (Executive Summary)
* **`exceptionally()` (Fallback):** Runs **only** if an exception is thrown upstream. It consumes the exception and provides a default fallback value to keep the application moving forward gracefully. Use case: Database down → return cached data.
* **`handle()` (Transform Both):** **Always runs** regardless of success or failure. It accepts both `(value, exception)` as parameters where one is always null, allowing you to intercept, recover, or map results to a different class type.
* **`whenComplete()` (Side-Effect Action):** **Always runs** like a `finally` block, but **cannot modify the result or stop exceptions**. It only views the outcome, making it optimal strictly for cleanup, metrics tracking, or logging.

| Method | Trigger Condition | Can Mutate Result? | Can Stop Exceptions? | Signature Parameters |
| :--- | :--- | :--- | :--- | :--- |
| **`exceptionally()`** | ❌ On Failure Only | ✅ Yes (Returns Default) | ✅ Yes (Swallows error) | `(ex)` |
| **`handle()`** | 🔄 Always (Success/Fail) | ✅ Yes (Can map to Type `R`) | ✅ Yes (Swallows error) | `(value, ex)` |
| **`whenComplete()`** | 🔄 Always (Success/Fail) | ❌ No (Read-Only) | ❌ No (Propagates out) | `(value, ex)` |

---

Suppose:

```java
CompletableFuture<Integer> future =
        CompletableFuture.supplyAsync(() -> {
            throw new RuntimeException("Database Error");
        });
```

Without handling:
```java
future.join();
```
throws
```
CompletionException
```
The original exception is wrapped inside it.

---

# exceptionally()

Think:

```
Something Failed
↓
Recover
↓
Return Default Value
```

Example

```java
CompletableFuture<Integer> future =
        CompletableFuture
                .supplyAsync(() -> {
                    throw new RuntimeException();
                })
                .exceptionally(ex -> {
                    System.out.println(ex.getMessage());
                    return 0;
                });
System.out.println(future.join());
```

Output
```
java.lang.RuntimeException
0
```

Notice
Instead of failing,
the pipeline continues.

---

# Visual

```
Task
↓
Exception
↓
exceptionally()
↓
Default Value
↓
Continue
```

---

# Real Example

Database unavailable.
Instead of:
```
500 Error
```
Return
```
Cached Data
```

---

# handle()

Think
```
Success?
↓
Yes
↓
Process
------------
Failure?
↓
Process
```

Unlike exceptionally(),
handle() always runs.

---

Example

```java
CompletableFuture<Integer> future =
        CompletableFuture
                .supplyAsync(() -> 10)
                .handle((value, ex) -> {
                    if(ex != null){
                        return 0;
                    }
                    return value * 2;
                });
System.out.println(future.join());
```

Output
```
20
```

Now failure:

```java
CompletableFuture<Integer> future =
        CompletableFuture
                .supplyAsync(() -> {
                    throw new RuntimeException();
                })
                .handle((value, ex) -> {
                    if(ex != null){
                        return 0;
                    }
                    return value;
                });
System.out.println(future.join());
```

Output
```
0
```

Notice
handle()
receives BOTH
```
Value
Exception
```
One of them is null.

---

# whenComplete()

Think
```
Finally
```
Runs
Whether success
or
failure.

But unlike handle(),
it cannot change the result.
Perfect for:
```
Logging
Metrics
Cleanup
```

---

Example

```java
CompletableFuture<Integer> future =
        CompletableFuture
                .supplyAsync(() -> 100)
                .whenComplete((value, ex) -> {
                    System.out.println("Finished");
                });
System.out.println(future.join());
```

Output
```
Finished
100
```

Notice
The result
```
100
```
did not change.

---

Now failure

```java
CompletableFuture<Integer> future =
        CompletableFuture
                .supplyAsync(() -> {
                    throw new RuntimeException();
                })
                .whenComplete((value, ex) -> {
                    System.out.println("Logging Error");
                });
future.join();
```

Output
```
Logging Error
CompletionException
```

Notice
The exception still propagates.
whenComplete()
does NOT recover.

---

# Difference

exceptionally()

```
Exception
↓
Recover
↓
Continue
```

---

handle()

```
Success?
↓
Transform
------------
Failure?
↓
Recover
↓
Continue
```

---

whenComplete()

```
Success?
↓
Log
------------
Failure?
↓
Log
↓
Original Result/Exception Continues
```

---

# Real Spring Boot Example

```java
CompletableFuture
        .supplyAsync(employeeService::getEmployee)
        .exceptionally(ex -> {
            return DEFAULT_EMPLOYEE;
        });
```
Fallback.

---

```java
CompletableFuture
        .supplyAsync(employeeService::getEmployee)
        .whenComplete((emp, ex) -> {
            log.info("Employee API Completed");
        });
```
Logging.

---

```java
CompletableFuture
        .supplyAsync(employeeService::getEmployee)
        .handle((emp, ex) -> {
            if(ex != null){
                return DEFAULT_EMPLOYEE;
            }
            return mapper.toDTO(emp);
        });
```
Transform both success and failure.

---

# Interview Questions

## Difference between exceptionally() and handle()

exceptionally()
Runs only when there is an exception.

handle()
Runs on both success and failure.

---

## Difference between handle() and whenComplete()

handle()
Can transform the result.

whenComplete()
Cannot change the result.
Mostly used for logging and cleanup.

---

## Does whenComplete() recover from exceptions?

No.
The exception continues downstream.

---

# Memory Trick

```
exceptionally()
↓
Recover
--------------------
handle()
↓
Always
↓
Transform
--------------------
whenComplete()
↓
Always
↓
Observe
↓
Don't Change
```
