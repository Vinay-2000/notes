### ⚡ TL;DR (Executive Summary)

* The Executor framework separates submitting work from creating and managing threads, typically through reusable thread pools.
* `execute()` runs a `Runnable` without a result; `submit()` returns a `Future` for result, completion, cancellation, and failure handling.
* Select pool types by workload: fixed for bounded concurrency, cached for short bursty tasks, single-thread for ordered work, and scheduled for delayed or periodic work.
* Shut executors down deliberately: `shutdown()` lets submitted tasks finish, while `shutdownNow()` attempts interruption.

---

# Why Was Executor Framework Introduced?

Suppose you want to execute 100 tasks.

Without ExecutorService:

```java
for(int i = 0; i < 100; i++) {
    new Thread(task).start();
}
```

Looks fine.

But think deeper.

Every iteration creates:

```
New Thread
↓
Allocate Stack
↓
Create OS Thread
↓
Register with Scheduler
↓
Execute
↓
Destroy Thread
```

Now imagine:

```
1000 Requests
↓
1000 Threads
↓
1000 Thread Creations
```

Creating threads is expensive.

So Java asked:

> **Why create new workers every time?**

Instead:

```
Create Workers Once
↓
Reuse Them
↓
Give Them New Tasks
```

This is called a **Thread Pool**.

---

# What is a Thread Pool?

Think of a restaurant.

Without Thread Pool

```
Customer Arrives
↓
Hire New Chef
↓
Cook
↓
Fire Chef
```

Ridiculous.

Instead:

```
Restaurant
↓
5 Chefs
↓
Customers Keep Coming
↓
Chefs Keep Cooking
```

Workers are reused.

A Thread Pool works exactly like this.

---

# Executor Framework

Java introduced:

```
Executor
↓
ExecutorService
↓
Thread Pool
```

Instead of:

```java
new Thread(...)
```

you simply submit work.

---

# Executor

Smallest interface.

```java
Executor executor = ...
```

Has only one method.

```java
execute(Runnable task);
```

Think:

```
Here is work.
Please execute it.
```

Executor doesn't say **how**.

---

# ExecutorService

ExecutorService extends Executor.

Adds:

- Thread Pool Management
- Shutdown
- Submit Callable
- Future
- Task Cancellation

This is what we use in real applications.

---

# Creating Thread Pools

## Fixed Thread Pool

```java
ExecutorService executor =
Executors.newFixedThreadPool(3);
```

Meaning:

```
3 Worker Threads
↓
Reuse Forever
```

Suppose:

```
10 Tasks
```

Execution:

```
Task1
Task2
Task3
↓
Workers Busy
↓
Task4 waits
↓
Worker Free
↓
Task4 Executes
```

---

## Cached Thread Pool

```java
Executors.newCachedThreadPool();
```

Behavior:

```
Need Thread?
↓
Reuse Existing
↓
Else Create New
```

Can create many threads.

Good for:

```
Short-lived Tasks
```

Danger:

Too many requests.

↓

Too many threads.

---

## Single Thread Executor

```java
Executors.newSingleThreadExecutor();
```

Only one worker.

```
Task1
↓
Task2
↓
Task3
```

Guaranteed sequential execution.

Useful for:

- Logging
- File Writing
- Ordered Processing

---

## Scheduled Thread Pool

```java
Executors.newScheduledThreadPool(2);
```

Execute later.

Example:

```java
schedule(task,5,SECONDS);
```

or

```
Every 10 Seconds
```

Great for:

- Cleanup Jobs
- Health Checks
- Cache Refresh

---

# execute() vs submit()

## execute()

```java
executor.execute(task);
```

Accepts:

```
Runnable
```

Returns:

```
Nothing
```

---

## submit()

```java
Future<?> future =
executor.submit(task);
```

Accepts:

```
Runnable
Callable
```

Returns:

```
Future
```

Future lets us:

- Wait
- Get Result
- Cancel
- Check Completion

We'll study Future next.

---

# shutdown()

```java
executor.shutdown();
```

Meaning:

```
No New Tasks
↓
Finish Existing Tasks
↓
Terminate Pool
```

---

# shutdownNow()

```java
executor.shutdownNow();
```

Meaning:

```
Stop Accepting
↓
Interrupt Running Tasks
↓
Return Waiting Tasks
```

Not guaranteed.

Tasks may ignore interruption.

---

# Internal Working

Suppose:

```
Pool Size = 3
```

```
Task1
Task2
Task3
Task4
Task5
```

Pool:

```
Worker1 → Task1
Worker2 → Task2
Worker3 → Task3
Queue
↓
Task4
Task5
```

When Worker1 finishes:

```
Worker1
↓
Task4
```

No new thread is created.

Worker reused.

---

# Why Is Executor Better Than Thread?

Without Executor

```
Task
↓
New Thread
↓
Destroy
```

With Executor

```
Task
↓
Queue
↓
Existing Worker
↓
Next Task
```

Huge performance improvement.

---

# Real Example

Imagine a Spring Boot API.

100 users call:

```
GET /employees
```

Does Spring create:

```
100 New Threads
```

for every request?

No.

Tomcat already has a thread pool.

Each request is assigned to an existing worker thread.

After processing:

```
Worker
↓
Returned To Pool
↓
Ready For Next Request
```

This is exactly the Executor Framework idea.

---

# Common Interview Questions

## Why not create a new Thread every time?

Because thread creation is expensive.

Thread pools reuse existing threads, reducing creation and destruction overhead.

---

## Difference between execute() and submit()

execute():

- Runnable only
- No return value

submit():

- Runnable or Callable
- Returns Future

---

## Difference between shutdown() and shutdownNow()

shutdown():

Finish existing tasks.

Reject new tasks.

shutdownNow():

Attempt to interrupt running tasks.

Reject new tasks.

---

## Which Thread Pool should I use?

Fixed Thread Pool

- Most common
- Predictable
- Good for backend services

Cached Thread Pool

- Short tasks
- Can create many threads

Single Thread Executor

- Sequential processing

Scheduled Thread Pool

- Delayed or periodic tasks

---

# Memory Trick

```
Need Thread?
↓
Don't Create
↓
Reuse
↓
Thread Pool
↓
ExecutorService
↓
submit()
↓
Future
```
