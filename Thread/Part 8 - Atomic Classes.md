## Why Were Atomic Classes Introduced?

Suppose we have:

```java
int count = 0;
```

Two threads execute:

```java
count++;
```

We already know:

```
Read

↓

Increment

↓

Write
```

This is **not atomic**.

Possible execution:

```
count = 5

Thread A reads 5

Thread B reads 5

Thread A writes 6

Thread B writes 6
```

Expected:

```
7
```

Actual:

```
6
```

Race Condition.

---

# Solution 1

```java
synchronized(this){

    count++;

}
```

Works.

But locking has overhead.

Sometimes all we need is:

```
Increment One Integer
```

Using a monitor lock for such a small operation is unnecessary.

Java introduced:

```
AtomicInteger
```

---

# AtomicInteger

```java
AtomicInteger count = new AtomicInteger(0);
```

Increment:

```java
count.incrementAndGet();
```

Output:

```
1

2

3

...
```

Thread-safe.

No synchronized block.

---

# Common Methods

Increment

```java
count.incrementAndGet();
```

Return new value.

---

Increment

```java
count.getAndIncrement();
```

Return old value.

Then increment.

---

Decrement

```java
count.decrementAndGet();
```

---

Read

```java
count.get();
```

---

Set

```java
count.set(100);
```

---

# How Does AtomicInteger Work?

This is the interesting part.

It does **NOT** use:

```
synchronized
```

It does **NOT** use:

```
ReentrantLock
```

Instead it uses:

```
CAS

Compare And Swap
```

---

# Compare-And-Swap (CAS)

Suppose:

```
count = 5
```

Thread wants:

```
6
```

CAS works like this:

```
Expected = 5

↓

Current = 5 ?

↓

Yes

↓

Replace with 6
```

Done.

No lock.

---

Suppose another thread changed it.

Current value:

```
6
```

Thread still expects:

```
5
```

CAS says:

```
Expected = 5

↓

Current = 6

↓

Not Equal

↓

Fail
```

The operation retries.

---

# Visual

Thread A

```
Read 5

↓

CAS

↓

Success

↓

Write 6
```

Thread B

```
Read 5

↓

CAS

↓

Current is 6

↓

Failed

↓

Retry

↓

Read 6

↓

CAS

↓

Write 7
```

Eventually everyone succeeds.

No lock.

---

# compareAndSet()

You can use CAS directly.

```java
AtomicInteger count = new AtomicInteger(10);

boolean success =
count.compareAndSet(10,20);
```

Meaning:

```
If Current == 10

↓

Change To 20
```

Otherwise

```
Do Nothing
```

---

# Why Is AtomicInteger Faster?

Normally:

```
Acquire Lock

↓

Increment

↓

Release Lock
```

AtomicInteger:

```
CAS

↓

Done
```

No monitor.

No blocking.

No context switching.

---

# Can AtomicInteger Replace synchronized?

No.

Suppose:

```java
balance += amount;

history.add(amount);

sendNotification();
```

Three operations.

AtomicInteger only makes:

```
One Variable

↓

Atomic
```

It cannot make multiple operations atomic together.

For multiple related operations:

```
synchronized

or

Lock
```

is still needed.

---

# ABA Problem

Suppose:

```
Value = A
```

Thread 1 reads:

```
A
```

Thread 2 changes:

```
A

↓

B

↓

A
```

Thread 1 performs CAS.

```
Expected = A

↓

Current = A

↓

Success
```

CAS thinks nothing changed.

But something DID.

This is called the **ABA Problem**.

---

# How Is ABA Solved?

Java provides:

```
AtomicStampedReference
```

Instead of:

```
A
```

Store:

```
A

Version = 1
```

Update:

```
B

Version = 2
```

Again:

```
A

Version = 3
```

Now CAS checks:

```
Value

AND

Version
```

Thread 1 expects:

```
A

Version 1
```

Current:

```
A

Version 3
```

CAS fails.

Problem solved.

---

# AtomicInteger vs volatile

| AtomicInteger         | volatile             |
| --------------------- | -------------------- |
| Visibility            | Visibility           |
| Atomic Operations     | No Atomic Operations |
| CAS                   | No CAS               |
| Thread-safe Increment | Not Thread-safe      |

---

# Common Interview Questions

## Is AtomicInteger lock-free?

Yes.

It uses Compare-And-Swap (CAS), not synchronized.

---

## Does AtomicInteger use volatile?

Yes.

Internally, the stored value is declared volatile so updates are visible across threads, while CAS provides atomicity.

---

## When should we use AtomicInteger?

When a single variable is updated independently and thread safety is required without the overhead of locking.

---

## Can AtomicInteger replace synchronized?

No.

It only guarantees atomic operations on a single variable.

Complex business operations involving multiple variables or objects still require synchronization.

---

Internally, `AtomicInteger` has something like this (simplified):

```
public class AtomicInteger {

    private volatile int value;

}
```

Notice:

```
volatile int value;
```

The actual integer inside `AtomicInteger` is **volatile**.

So when you do:

```
count.get();
```

you're reading a **volatile** variable.

That guarantees you see the latest value.




# Memory Trick

```
volatile

↓

Visibility

-------------------

AtomicInteger

↓

Visibility

+

Atomicity

↓

CAS

↓

No Lock

-------------------

synchronized

↓

Visibility

+

Atomicity

+

Multiple Operations
```
