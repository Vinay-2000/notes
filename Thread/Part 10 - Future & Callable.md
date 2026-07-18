 
# Why Do We Need Future?

Suppose you submit a task.

```java
executor.submit(task);
```

Questions:

- Did it finish?
- Did it fail?
- What value did it return?
- Can I cancel it?

`execute()` cannot answer these.

Java introduced:

```
Future
```

Think of Future as:

> **A placeholder for the result of a task that will complete in the future.**

---

# Runnable vs Callable

Runnable

```java
Runnable task = () -> {
    System.out.println("Hello");
};
```

- No return value
- Cannot throw checked exceptions

---

Callable

```java
Callable<String> task = () -> {
    return "Hello";
};
```

- Returns a value
- Can throw checked exceptions

---

# Why Callable?

Suppose:

```
Download File

↓

Need Downloaded Content
```

Runnable cannot return it.

Callable can.

---

# submit()

```java
ExecutorService executor =
        Executors.newFixedThreadPool(2);

Future<String> future =
        executor.submit(() -> {

            return "Vinay";

        });
```

Notice:

```
submit()

↓

Future
```

Now we have something representing that running task.

---

# get()

```java
String value = future.get();
```

Meaning:

```
Task Finished?

↓

No

↓

Wait

↓

Finished

↓

Return Result
```

This is very similar to:

```java
thread.join();
```

Difference:

```
join()

↓

Wait for Thread

--------------------

Future.get()

↓

Wait for Task

↓

Return Result
```

---

# Example

```java
ExecutorService executor =
        Executors.newSingleThreadExecutor();

Future<Integer> future =
        executor.submit(() -> {

            Thread.sleep(2000);

            return 100;

        });

System.out.println("Doing other work...");

System.out.println(future.get());

executor.shutdown();
```

Output

```
Doing other work...

(wait 2 sec)

100
```

Notice

The main thread continues doing other work until it actually needs the result.

---

# isDone()

Instead of blocking:

```java
future.get();
```

You can check:

```java
future.isDone();
```

Returns

```
true

false
```

Useful for polling.

---

# cancel()

Suppose:

```
Long Running Task
```

You don't need it anymore.

```java
future.cancel(true);
```

Meaning:

```
Interrupt Task

↓

Attempt Cancellation
```

Returns:

```
true

false
```

depending on whether cancellation succeeded.

---

# get(timeout)

Instead of waiting forever:

```java
future.get(5, TimeUnit.SECONDS);
```

Meaning:

```
Wait

↓

Maximum 5 Seconds

↓

Still Running?

↓

TimeoutException
```

Very useful for API calls.

---

# Exception Handling

Suppose:

```java
Callable<Integer> task =
        () -> {

            throw new RuntimeException();

        };
```

Task fails.

When you call:

```java
future.get();
```

Java throws:

```
ExecutionException
```

The original exception is wrapped inside it.

---

# Future Lifecycle

```
submit()

↓

Running

↓

Completed

↓

Future

↓

get()

↓

Result
```

or

```
submit()

↓

Running

↓

Cancelled

↓

CancellationException
```

---

# Real World Example

Suppose a Spring Boot API needs:

```
Employee Details

Salary

Projects
```

Three independent database calls.

Submit all three.

```
Future<Employee>

Future<Salary>

Future<List<Project>>
```

Later:

```java
employee.get();

salary.get();

projects.get();
```

This is much faster than executing them one after another.

---

# Limitations of Future

Future is useful but limited.

Suppose:

```
Task A

↓

Task B

↓

Task C
```

Can Future chain them?

No.

Can Future combine multiple tasks elegantly?

No.

Can Future attach callbacks?

No.

These limitations led to:

```
CompletableFuture
```

which we'll study next.

---

# Common Interview Questions

## Difference between Runnable and Callable?

| Runnable | Callable |
|-----------|----------|
| No return value | Returns value |
| Cannot throw checked exceptions | Can throw checked exceptions |
| execute() | submit() |

---

## Difference between execute() and submit()?

execute()

- Runnable only
- No Future

submit()

- Runnable or Callable
- Returns Future

---

## Difference between join() and Future.get()?

join()

- Waits for a Thread

Future.get()

- Waits for a Task
- Returns result

---

## What happens if get() is called before completion?

The calling thread blocks until the task completes.

---

## What exception does Future.get() throw?

- InterruptedException
- ExecutionException
- TimeoutException (for timed get)

---

# Memory Trick

```
Runnable

↓

No Result

-------------------

Callable

↓

Returns Result

↓

submit()

↓

Future

↓

get()

↓

Value
```