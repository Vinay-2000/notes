# Part 11E - Custom Executor with CompletableFuture

### ⚡ TL;DR (Executive Summary)
* **The Default Risk:** By default, standard async operations utilize `ForkJoinPool.commonPool()`. If mismatched tasks (like intense image processing and lightweight email triggers) share this single pool, a heavy process can completely stall unrelated components.
* **Resource Isolation:** Passing a custom `ExecutorService` parameter splits workloads into isolated thread ecosystems.
* **The `Async` Naming Rule:** Standard methods (e.g., `thenApply()`) execute code in the thread context of the *current completing task*. Switching to `thenApplyAsync(fn, executor)` forces it to jump tasks to your explicitly designated custom thread pool.

| Context Engine | Thread Target Pool | Isolation Capacity | Performance Impact |
| :--- | :--- | :--- | :--- |
| **Default Methods** | `ForkJoinPool.commonPool()` | ❌ Poor (Shared globally) | Heavy operations delay light tasks |
| **Custom Executors** | Custom Dedicated Pools | ✅ Excellent (Segmented) | Clear workload tuning controls |

---

## Default Behavior

Suppose you write:

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            return "Hello";
        });
```

Output:
```
ForkJoinPool.commonPool-worker-1
```

Notice:
```
ForkJoinPool.commonPool()
```
is used automatically.

---

# Why Not Use Common Pool?

Imagine your application has:
```
Image Processing
Email Sending
PDF Generation
Database Calls
API Calls
```
If everything uses:
```
ForkJoinPool.commonPool()
```
all tasks compete for the same shared pool.
One heavy task can slow down unrelated work.

---

# Create Your Own Executor

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);
```

Now tell CompletableFuture to use it.

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            return "Hello";
        }, executor);
System.out.println(future.join());
executor.shutdown();
```

Output:
```
pool-1-thread-1
```

Notice
No ForkJoinPool anymore.

---

# Async Chain Using Custom Executor

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);
CompletableFuture<String> future =
        CompletableFuture
                .supplyAsync(() -> "Java", executor)
                .thenApplyAsync(
                        String::toUpperCase,
                        executor
                )
                .thenApplyAsync(
                        s -> s + " 21",
                        executor
                );
System.out.println(future.join());
executor.shutdown();
```

Every async stage runs using your executor.

---

# Why Pass Executor Again?

Remember:

```
thenApply()
↓
Current Thread
```

```
thenApplyAsync()
↓
ForkJoinPool
```

Unless you specify:
```java
.thenApplyAsync(fn, executor)
```
then your own executor is used.

---

# Spring Boot Example

Usually we create:
```java
@Bean
public ExecutorService executor() {
    return Executors.newFixedThreadPool(10);
}
```

or more commonly:
```java
@Bean
public ThreadPoolTaskExecutor taskExecutor() {
    ThreadPoolTaskExecutor executor =
            new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(10);
    executor.setMaxPoolSize(20);
    executor.setQueueCapacity(100);
    executor.initialize();
    return executor;
}
```

Then
```java
CompletableFuture
        .supplyAsync(
                employeeService::getEmployee,
                executor
        );
```
Every async task uses the application's thread pool.

---

# Why Is This Better?

Instead of:
```
Everyone
↓
ForkJoinPool
```

You can have:
```
API Calls
↓
API Executor
----------------
Email
↓
Email Executor
----------------
Reports
↓
Report Executor
```
Each workload gets its own thread pool.
Heavy report generation won't delay email sending.

---

# Interview Question

## Why use a custom Executor with CompletableFuture?

Because the default `ForkJoinPool.commonPool()` is shared. Using a custom executor gives you:
- Better resource isolation
- Control over thread count
- Better performance tuning
- Separation of different workloads

---

# Memory Trick

```
Default
↓
ForkJoinPool.commonPool()
--------------------
Custom
↓
ExecutorService
↓
Your Thread Pool
```
