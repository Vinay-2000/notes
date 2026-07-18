There are four major concurrency problems:

1. Race Condition
2. Deadlock
3. Starvation
4. Livelock

---

# 1. Race Condition ⭐⭐⭐⭐⭐

## What is it?

Multiple threads access shared mutable data simultaneously and the final result depends on the order of execution.

Example

```java
count++;
```

Internally

```
Read

↓

Increment

↓

Write
```

Thread A

```
Read 5
```

Thread B

```
Read 5
```

Thread A

```
Write 6
```

Thread B

```
Write 6
```

Expected

```
7
```

Actual

```
6
```

---

Solution

```
synchronized

ReentrantLock

AtomicInteger
```

---

# 2. Deadlock ⭐⭐⭐⭐⭐

## What is it?

Two or more threads permanently wait for each other.

Nobody can proceed.

Example

Thread A

```
Lock A

↓

Waiting for Lock B
```

Thread B

```
Lock B

↓

Waiting for Lock A
```

Result

```
Forever Waiting
```

---

Example

```java
Object lock1 = new Object();
Object lock2 = new Object();

Thread t1 = new Thread(() -> {

    synchronized(lock1){

        synchronized(lock2){

        }

    }

});

Thread t2 = new Thread(() -> {

    synchronized(lock2){

        synchronized(lock1){

        }

    }

});
```

Possible

```
T1 gets lock1

↓

T2 gets lock2

↓

T1 waits lock2

↓

T2 waits lock1

↓

Deadlock
```

---

Solution

Always acquire locks in the same order.

Example

```
lock1

↓

lock2
```

Everywhere.

---

# 3. Starvation ⭐⭐⭐⭐☆

## What is it?

A thread never gets CPU time or a resource because other threads continuously get preference.

Example

```
High Priority Thread

↓

Runs Again

↓

Runs Again

↓

Runs Again
```

Low priority thread

```
Waiting...

Waiting...

Waiting...
```

Forever.

---

Example

Non-fair ReentrantLock

```
Thread A waiting

Thread B waiting

New Thread C arrives

↓

C gets lock
```

If this keeps happening,

A may starve.

---

Solution

```
Fair Lock

new ReentrantLock(true)
```

---

# 4. Livelock ⭐⭐⭐⭐☆

Looks like progress.

Actually no progress.

Threads keep responding to each other.

Example

Imagine two people in a hallway.

```
Person A

↓

Moves Left
```

Person B

```
Moves Left
```

Still blocked.

A

```
Moves Right
```

B

```
Moves Right
```

Still blocked.

They keep moving forever.

Nobody passes.

---

Programming Example

Thread A

```
tryLock()

↓

Failed

↓

Release

↓

Retry
```

Thread B

```
tryLock()

↓

Failed

↓

Release

↓

Retry
```

Both keep being polite.

Nobody finishes.

---

Solution

Random Backoff

Thread waits

```
100ms

200ms

Random Delay
```

before retrying.

---

# Summary

Race Condition

```
Wrong Data
```

Deadlock

```
No Thread Moves
```

Starvation

```
One Thread Never Gets Chance
```

Livelock

```
Threads Keep Moving

But No Progress
```

---

# Interview Table

| Problem        | Threads Running? | Progress?                    |
| -------------- | ---------------- | ---------------------------- |
| Race Condition | ✅ Yes           | ❌ Wrong Result              |
| Deadlock       | ❌ No            | ❌ None                      |
| Starvation     | Some             | One Thread Never Gets Chance |
| Livelock       | ✅ Yes           | ❌ None                      |

---

# Prevention

Race Condition

```
Synchronization
```

Deadlock

```
Lock Ordering

tryLock()

Timeout
```

Starvation

```
Fair Scheduling

Fair Lock
```

Livelock

```
Random Delay

Backoff Strategy
```
