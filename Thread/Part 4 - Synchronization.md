## Why Do We Need Synchronization?

Remember:

```
Thread 1
↓
Stack (Private)
-
Thread 2
↓
Stack (Private)
-
Heap (Shared)
```

The problem is **not** the stack.
The problem is the **shared heap**.
Suppose we have:

```java
class Counter {
    int count = 0;
}
```

Both threads share the same object.

```
          Heap
      Counter
     count = 0
        ▲    ▲
        │    │
 Thread1    Thread2
```

# Race Condition

Suppose both threads execute:

```java
counter.count++;
```

Looks like one statement.
It is NOT.
Internally it becomes:

```
Read count
↓
Add 1
↓
Write count
```

Imagine:
Current value:

```
count = 0
```

Thread 1

```
Read
↓
0
```

Before it writes...
CPU switches.
Thread 2

```
Read
↓
0
↓
Increment
↓
Write 1
```

CPU switches back.
Thread 1

```
Increment
↓
Write 1
```

Expected:

```
2
```

Actual:

```
1
```

This is called a **Race Condition**.
The final value depends on the order in which threads execute.

# Critical Section

A **Critical Section** is any code that accesses shared mutable data.
Example:

```java
counter.count++;
```

Only one thread should execute this at a time.

# Synchronization

Synchronization ensures:

> Only one thread enters the critical section at a time.
> Java provides this using:

```java
synchronized
```

# synchronized Method

```java
class Counter {
    int count;
    synchronized void increment() {
        count++;
    }
}
```

Now:

```
Thread 1
↓
increment()
↓
Lock Acquired
↓
count++
↓
Lock Released
Thread 2
↓
Wait
↓
Gets Lock
↓
count++
```

Only one thread executes the method at a time.

# synchronized Block

Instead of locking the whole method:

```java
void increment() {
    synchronized(this) {
        count++;
    }
}
```

Only the important code is synchronized.
This is preferred when only a small portion needs protection.

# What is Being Locked?

This is the most misunderstood topic.
Java does NOT lock methods.
Java locks **objects**.
Example:

```java
synchronized(this)
```

means

```
Lock this object
```

Suppose:

```java
Counter c = new Counter();
```

```
Counter Object
↓
Monitor Lock
```

Every Java object has an intrinsic monitor (lock).
When a thread enters:

```java
synchronized(this)
```

it acquires that object's monitor.

# Object Lock

```
Counter counter
↓
Monitor
```

Thread 1

```
Acquire Monitor
↓
Execute
↓
Release Monitor
```

Thread 2

```
Blocked
↓
Waiting
↓
Acquire Monitor
```

# What if We Have Two Objects?

```java
Counter c1 = new Counter();
Counter c2 = new Counter();
```

Each object has its own lock.

```
c1 Lock
↓
Thread 1
```

```
c2 Lock
↓
Thread 2
```

Both can execute simultaneously.
Synchronization only blocks threads competing for the **same object**.

# Static synchronized

```java
class Counter {
    static synchronized void test() {
    }
}
```

This does NOT lock an object.
It locks the **Class object**.

```
Counter.class
↓
Monitor Lock
```

Only one thread can execute any static synchronized method of that class at a time.

# Object Lock vs Class Lock

Object Lock

```
Counter object
↓
Lock
```

Class Lock

```
Counter.class
↓
Lock
```

Completely different locks.

# Intrinsic Lock (Monitor)

Every Java object automatically has a monitor.
You never create it.
The JVM creates it.

```
Object
↓
Monitor
↓
synchronized
```

This is why synchronized works without creating any Lock object.

# Can synchronized Prevent Race Conditions?

Yes.
Without synchronization:

```
Read
↓
Increment
↓
Write
```

Multiple threads may interleave.
With synchronization:

```
Acquire Lock
↓
Read
↓
Increment
↓
Write
↓
Release Lock
```

The entire sequence becomes atomic with respect to other threads using the same lock.

# Does synchronized Stop All Threads?

No.
It only blocks threads trying to acquire the **same monitor**.
Different objects have different monitors.

# Interview Questions

## What does synchronized lock?

Not methods.
Not code.
It locks an object's monitor.

## Why is synchronized needed?

Because multiple threads share objects on the heap, leading to race conditions.
Synchronization ensures only one thread accesses a critical section at a time.

## Difference between synchronized method and synchronized block?

A synchronized method locks the entire method.
A synchronized block locks only the specified section of code, reducing lock contention.

## Can two synchronized methods execute simultaneously?

Yes, if they belong to different objects.
No, if they synchronize on the same object.

# Memory Trick

```
Shared Heap
↓
Race Condition
↓
Need Lock
↓
synchronized
↓
Object Monitor
↓
One Thread at a Time
```
