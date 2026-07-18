### ⚡ TL;DR (Executive Summary)

* **Why `Lock` exists:** `synchronized` is ideal for simple mutual exclusion, while `ReentrantLock` adds practical control: non-blocking attempts, timeouts, interruptible waits, and optional fairness.
* **Safe pattern:** Always pair `lock.lock()` with `unlock()` in a `finally` block so exceptions cannot leave other threads permanently blocked.
* **Key capabilities:** Use `tryLock()` to avoid indefinite waiting, timed `tryLock()` to give up after a limit, and `lockInterruptibly()` to support cancellation.
* **Read-heavy data:** `ReadWriteLock` lets multiple readers proceed together but keeps writers exclusive; `StampedLock` adds optimistic reads for high-performance read-mostly workloads.

| Need | Best Fit |
| :--- | :--- |
| Simple shared-data protection | `synchronized` |
| Timeout, interruption, fairness, or `tryLock()` | `ReentrantLock` |
| Many readers, occasional writers | `ReadWriteLock` |
| Advanced read-heavy optimization | `StampedLock` |

---

## Why Was Lock Introduced?

We already have:

```java
synchronized(this) {
    count++;
}
```

So why introduce:

```java
Lock lock = new ReentrantLock();
lock.lock();
try{
    count++;
}
finally{
    lock.unlock();
}
```

Because `synchronized` is simple, but sometimes **too simple**.
Java needed more control.

# What Problems Does synchronized Have?

## Problem 1 - Can't Try to Acquire a Lock

Suppose Thread A has the lock.
Thread B reaches:

```java
synchronized(this) {
    ...
}
```

What happens?

```
Thread B
↓
BLOCKED
↓
Wait Forever
```

Thread B has no choice.
It must wait.
Sometimes we don't want that.
Example:

```
ATM Machine
↓
If busy
↓
Try another ATM
```

This is where:

```java
tryLock()
```

comes in.

## Problem 2 - Can't Interrupt While Waiting

Suppose:

```
Thread A
↓
Has Lock
-
Thread B
↓
Waiting
```

Now someone says:

```
Cancel Thread B
```

With `synchronized`
Too bad.
Thread B keeps waiting.
With `ReentrantLock`

```java
lock.lockInterruptibly();
```

Now another thread can call:

```java
thread.interrupt();
```

Thread B immediately stops waiting.

## Problem 3 - Can't Choose Fairness

Imagine five people waiting.

```
A
B
C
D
E
```

With `synchronized`
The JVM decides.
Maybe:

```
A
↓
E
↓
C
↓
B
```

No guarantees.
Sometimes that's fine.
Sometimes it isn't.
`ReentrantLock` allows:

```java
new ReentrantLock(true);
```

Meaning:

```
First Come
↓
First Served
```

# ReentrantLock

Create one:

```java
Lock lock = new ReentrantLock();
```

Acquire it:

```java
lock.lock();
```

Release it:

```java
lock.unlock();
```

Always:

```java
lock.lock();
try{
    // Critical Section
}
finally{
    lock.unlock();
}
```

# Why finally?

Suppose:

```java
lock.lock();
count++;
throw new RuntimeException();
```

Without:

```java
unlock();
```

The lock is never released.
Every future thread blocks forever.
Using:

```java
finally
```

guarantees:

```
Lock
↓
Released
↓
Always
```

# Why is it called Reentrant?

"Reentrant" means:

> The thread that already owns the lock can acquire it again.
> Example:

```java
class Counter{
    Lock lock = new ReentrantLock();
    void increment(){
        lock.lock();
        try{
            helper();
        }finally{
            lock.unlock();
        }
    }
    void helper(){
        lock.lock();
        try{
            ...
        }finally{
            lock.unlock();
        }
    }
}
```

Notice:
Same thread.
Same lock.
Acquired twice.
Allowed.
The lock internally keeps a count.

```
First lock()
↓
Count = 1
Second lock()
↓
Count = 2
unlock()
↓
Count = 1
unlock()
↓
Count = 0
↓
Released
```

This is why it's called **Reentrant**.

# tryLock()

Instead of waiting forever:

```java
if(lock.tryLock()){
    try{
        // Work
    }finally{
        lock.unlock();
    }
}else{
    System.out.println("Lock Busy");
}
```

Think:

```
Try
↓
Success?
↓
Yes → Work
No → Do Something Else
```

# Timed tryLock()

```java
lock.tryLock(5, TimeUnit.SECONDS);
```

Meaning:

```
Wait
↓
Maximum 5 Seconds
↓
Still Busy?
↓
Give Up
```

# lockInterruptibly()

```java
lock.lockInterruptibly();
```

Suppose:

```
Thread A
↓
Owns Lock
```

Thread B

```
Waiting
```

Someone interrupts B.

```
threadB.interrupt();
```

Instead of waiting forever:

```
InterruptedException
```

is thrown.
Very useful for cancellation.

# Fair vs Non-Fair Lock

Fair

```
A
↓
B
↓
C
```

Everyone gets the lock in arrival order.
Non-Fair

```
A
↓
C
↓
B
```

Higher throughput.
This is Java's default.

# ReadWriteLock

Suppose:
100 Threads

```
Read Read Read Read Read
```

Should they block each other?
No.
Reading doesn't modify data.
ReadWriteLock provides:

```
Many Readers
One Writer
```

Multiple readers can proceed simultaneously.
Only writers require exclusive access.
Useful for:

- Caches
- Configuration
- Dictionaries

# StampedLock

An advanced version of ReadWriteLock.
Adds:

```
Optimistic Read
```

Meaning:

```
Read
↓
Assume No Writer
↓
Validate
↓
Still Safe?
↓
Use Result
```

Much faster for read-heavy workloads.
Mostly used in high-performance systems.

# synchronized vs ReentrantLock

| synchronized                    | ReentrantLock                 |
| ------------------------------- | ----------------------------- |
| Keyword                         | Class                         |
| JVM releases lock automatically | Must call unlock() manually   |
| Simpler                         | More flexible                 |
| No tryLock()                    | Supports tryLock()            |
| No fairness                     | Fair or non-fair              |
| No interruptible locking        | lockInterruptibly() supported |
| Less code                       | More code                     |

# Which Should We Use?

If all you need is:

```
Protect Shared Data
```

Use:

```java
synchronized
```

Simple.
Safe.
Easy.
If you need:

- Timeout
- Fairness
- Interruptible waiting
- Non-blocking lock attempts
    Use:

```java
ReentrantLock
```

# Common Interview Questions

## Why was ReentrantLock introduced?

Because synchronized cannot provide timeout, fairness, interruptible locking, or non-blocking lock attempts.

## Why is unlock() placed inside finally?

To guarantee that the lock is released even if an exception occurs.

## What does Reentrant mean?

The same thread can acquire the same lock multiple times without deadlocking itself.
The lock keeps an internal acquisition count.

## Which is faster?

For simple synchronization, modern JVMs have optimized `synchronized` heavily, so performance is often comparable.
Choose based on the features you need, not assumed speed.

# Memory Trick

```
synchronized
↓
Simple
-
ReentrantLock
↓
More Control
↓
tryLock()
↓
Fairness
↓
Interruptible
↓
Timeout
```

# Fair vs Non-Fair Lock

## Problem

Suppose three threads want the same lock.

```
Thread A
Thread B
Thread C
```

---

## Non-Fair Lock (Default)

```java
Lock lock = new ReentrantLock();
```

Timeline

```
A arrives
↓
Gets Lock
-------------------
B arrives
↓
Waiting
-------------------
C arrives
↓
Waiting
```

A releases the lock.

Who gets it?

```
Could be B
Could be C
Could even be a new Thread D
```

Java doesn't guarantee the order.

Example:

```
Arrival
A
B
C
--------
Execution
A
C
B
```

Why?

The JVM optimizes for **throughput**, not fairness.

If a thread happens to be running on the CPU when the lock becomes free, it may "barge" in ahead of waiting threads.

---

## Fair Lock

```java
Lock lock = new ReentrantLock(true);
```

Now Java keeps a queue.

```
A
↓
B
↓
C
```

When A releases:

```
B gets lock
↓
C gets lock
```

Strict FIFO (First In First Out).

---

## Should we always use Fair Lock?

No.

Fair locks:

- Less starvation
- More context switching
- Lower throughput

Most applications use the default **non-fair** lock.

```
Lock lock = new ReentrantLock(true); // change to false
Runnable task = () -> {
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName());
        Thread.sleep(1000);
    } catch (InterruptedException e) {
    } finally {
        lock.unlock();
    }
};
for (int i = 1; i <= 5; i++) {
    new Thread(task, "T" + i).start();
}
```

# ReadWriteLock

Think about a library.

```
100 people
↓
Reading books
```

Can they all read at once?

Yes.

Now suppose:

```
One librarian
↓
Updating books
```

Can people read while shelves are changing?

No.

---

ReadWriteLock separates:

```
Read Lock
Write Lock
```

---

Read Lock

```
Reader A
↓
Allowed
```

```
Reader B
↓
Allowed
```

```
Reader C
↓
Allowed
```

Multiple readers can work simultaneously.

---

Write Lock

```
Writer
↓
Exclusive
```

No reader or writer can proceed.

---

Truth Table

| Current | New Reader | New Writer |
|----------|------------|------------|
| Readers Present | ✅ Allowed | ❌ Wait |
| Writer Present | ❌ Wait | ❌ Wait |
## Code

```
ReadWriteLock rw = new ReentrantReadWriteLock();
Lock read = rw.readLock();
Lock write = rw.writeLock();
```

Reader

```
read.lock();
try {
    System.out.println("Reading");
} finally {
    read.unlock();
}
```

Writer

```
write.lock();
try {
    System.out.println("Writing");
} finally {
    write.unlock();
}
```

---

## Visual

```
Reader A
↓
Read Lock
----------------
Reader B
↓
Read Lock
----------------
Reader C
↓
Read Lock
```

All run together.

Now Writer comes.

```
Writer
↓
Wait
↓
Readers Finish
↓
Writer Runs
```

| Current State     | Can Read Lock Be Acquired?   | Can Write Lock Be Acquired? |
| ----------------- | ---------------------------- | --------------------------- |
| No locks          | ✅ Yes                        | ✅ Yes                       |
| Read lock(s) held | ✅ Yes (more readers allowed) | ❌ No                        |
| Write lock held   | ❌ No                         | ❌ No (by another thread)    |
