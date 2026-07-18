# Problem

Suppose we have one object shared by all threads.

```java
class UserContext {

    String user;

}
```

Multiple threads:

```
Thread A

↓

user = "Vinay"
```

```
Thread B

↓

user = "Sam"
```

Since both use the same object:

```
Thread A

↓

Vinay

↓

Sam
```

The value gets overwritten.

We could use synchronization.

But what if each thread should have **its own copy**?

---

# ThreadLocal

Think of ThreadLocal as:

> **A variable whose value is private to each thread.**

Every thread sees its own value.

---

# Example

```java
ThreadLocal<String> currentUser =
        new ThreadLocal<>();
```

Thread A

```java
currentUser.set("Vinay");
```

Thread B

```java
currentUser.set("Sam");
```

Later

Thread A

```java
System.out.println(currentUser.get());
```

prints

```
Vinay
```

Thread B

```java
System.out.println(currentUser.get());
```

prints

```
Sam
```

No synchronization needed.

---

# Visual

Without ThreadLocal

```
Thread A

↓

Shared Variable

↑

Thread B
```

Everyone shares one variable.

---

With ThreadLocal

```
Thread A

↓

Vinay

----------------

Thread B

↓

Sam

----------------

Thread C

↓

John
```

Each thread has its own value.

---

# Complete Example

```java
ThreadLocal<String> user =
        new ThreadLocal<>();

Runnable task = () -> {

    user.set(Thread.currentThread().getName());

    System.out.println(user.get());

};

new Thread(task, "Vinay").start();

new Thread(task, "Sam").start();
```

Output

```
Vinay

Sam
```

Each thread sees only its own value.

---

# How Does It Work?

Many people think

```
ThreadLocal

↓

Stores Data
```

Actually

```
Thread

↓

ThreadLocalMap

↓

ThreadLocal

↓

Value
```

Each Thread object internally contains a:

```
ThreadLocalMap
```

Think:

```
Thread A

↓

ThreadLocalMap

↓

user → Vinay

--------------------

Thread B

↓

ThreadLocalMap

↓

user → Sam
```

The ThreadLocal object is just the **key**.

The value lives inside the thread.

---

# Common Methods

Set

```java
threadLocal.set(value);
```

Read

```java
threadLocal.get();
```

Remove

```java
threadLocal.remove();
```

Always remove when finished.

---

# Why remove()?

Suppose we're using a thread pool.

```
Thread 1

↓

Request A

↓

user = Vinay
```

Request finishes.

Thread goes back to pool.

Later

```
Thread 1

↓

Request B
```

If we never removed:

```
user = Vinay
```

Request B may accidentally see the previous user's data.

This is a memory leak / stale data problem.

Always do

```java
try {

    ...

} finally {

    threadLocal.remove();

}
```

---

# Real Spring Boot Example

Imagine a filter.

```java
ThreadLocal<String> currentUser =
        new ThreadLocal<>();
```

Authentication Filter

```java
currentUser.set(jwt.getUsername());
```

Service

```java
String user = currentUser.get();
```

DAO

```java
String user = currentUser.get();
```

No need to pass

```
username
```

through every method.

---

# Where Is ThreadLocal Used?

Very common in:

- Logging Context (MDC)
- Security Context
- Hibernate Session
- Transaction Context
- Request Context

Spring itself uses ThreadLocal internally.

---

# ThreadLocal with Virtual Threads

Works perfectly.

But...

Creating

```
1,000,000 Virtual Threads
```

means potentially

```
1,000,000 ThreadLocal Maps
```

Use carefully.

---

# Interview Questions

## Is ThreadLocal thread-safe?

Yes.

Because every thread has its own value.

---

## Is the ThreadLocal object copied?

No.

The ThreadLocal object is shared.

The values are different because each thread stores its own entry.

---

## Why call remove()?

To prevent stale data and memory leaks, especially in thread pools.

---

# Memory Trick

```
Shared Variable

↓

One Copy

↓

Everyone Shares

-------------------

ThreadLocal

↓

One Variable

↓

Many Values

↓

One Per Thread
```
