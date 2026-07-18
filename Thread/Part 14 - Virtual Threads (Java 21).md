
### ⚡ TL;DR (Executive Summary)

* Virtual threads are lightweight JVM-managed threads designed to let applications handle very large numbers of blocking, I/O-bound tasks with simple thread-per-request code.
* They are scheduled on a smaller set of platform-thread carriers and can unmount while blocked, freeing carriers for other work.
* They improve scalability for blocking I/O, not CPU-bound throughput; CPU-heavy work remains limited by processor cores.
* Avoid pinning virtual threads by holding monitors during blocking operations; use them thoughtfully with `ThreadLocal` and existing frameworks.

---

> ⭐⭐⭐⭐⭐ Very Important for Java 21 Interviews

---

# Why Were Virtual Threads Introduced?

Let's first understand the problem.

Suppose a Spring Boot application receives:

```
10,000 HTTP Requests
```

Traditional Thread Model

```
10,000 Requests
↓
Need Thousands of Platform Threads
↓
Huge Memory Usage
↓
OS Context Switching
↓
Slow
```

Remember:

Each platform thread has:

- Native OS Thread
- ~1 MB Stack (approximately, configurable)
- Context Switching Cost

Creating thousands of threads is expensive.

---

# The Idea

Instead of

```
1 Java Thread
↓
1 OS Thread
```

Java said:

> Why not create lightweight Java threads that are **scheduled by the JVM**, not directly by the operating system?

These are called

```
Virtual Threads
```

---

# Platform Thread (Old Model)

```
Java Thread
↓
OS Thread
↓
CPU
```

One Java thread owns one OS thread.

---

# Virtual Thread

```
Virtual Thread
↓
JVM Scheduler
↓
Small Pool of Platform Threads
↓
CPU
```

Thousands of virtual threads share a much smaller number of platform threads.

---

# Platform Thread vs Virtual Thread

| Platform Thread | Virtual Thread |
|-----------------|----------------|
| Heavyweight | Lightweight |
| 1:1 with OS Thread | Many-to-few mapping to platform threads |
| Expensive to Create | Very Cheap |
| High Memory Usage | Very Low Memory |
| Limited Scalability | Millions Possible |

---

# Creating Platform Thread

```java
Thread thread = new Thread(() -> {
    System.out.println("Platform Thread");
});
thread.start();
```

---

# Creating Virtual Thread

```java
Thread.startVirtualThread(() -> {
    System.out.println("Virtual Thread");
});
```

That's it.

---

Another way

```java
Thread.Builder builder = Thread.ofVirtual();
Thread thread = builder.start(() -> {
    System.out.println("Hello");
});
```

---

# Executor for Virtual Threads

Instead of

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);
```

Use

```java
try (ExecutorService executor =
         Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        System.out.println("Hello");
    });
}
```

Each submitted task gets its own virtual thread.

---

# Why Are They Faster?

Imagine

```
1000 API Calls
↓
Waiting for Database
```

Platform Threads

```
Thread
↓
Blocked
↓
OS Thread Also Blocked
```

Virtual Threads

```
Virtual Thread
↓
Blocked
↓
Platform Thread Released
↓
Another Virtual Thread Uses It
```

This is the biggest advantage.

---

# Mounting & Unmounting

Suppose

```
Virtual Thread
↓
Running
```

The JVM mounts it onto a platform thread.

```
Virtual Thread
↓
Platform Thread
↓
CPU
```

Now suppose

```java
Thread.sleep(5000);
```

The virtual thread blocks.

Instead of blocking the platform thread,

the JVM does:

```
Unmount
↓
Platform Thread Free
↓
Run Another Virtual Thread
```

Later

```
Wake Up
↓
Mount Again
↓
Continue
```

This makes blocking operations much cheaper.

---

# Example

Platform Thread

```
Database Call
↓
Waiting
↓
OS Thread Wasted
```

Virtual Thread

```
Database Call
↓
Waiting
↓
Platform Thread Returned To Pool
↓
Another Virtual Thread Runs
```

Huge scalability improvement.

---

# Real Spring Boot Example

Suppose

```
10,000 Users
↓
Calling REST API
```

Traditional

```
Need Thousands Of Platform Threads
```

Virtual Threads

```
Need Thousands Of Virtual Threads
↓
Only Small Number Of Platform Threads
```

Much less memory.

---

# Are Virtual Threads Faster?

Interview Trick Question.

Answer

```
Not Always.
```

Virtual Threads don't make your code execute faster.

They improve

```
Scalability
```

by reducing thread creation cost and efficiently handling blocking.

---

# When Should We Use Them?

Excellent For

- REST APIs
- Database Calls
- File I/O
- Network Calls
- HTTP Clients
- Microservices

Basically

```
Blocking I/O
```

---

# When NOT To Use Them?

CPU-intensive work.

Example

```
Image Processing
Machine Learning
Encryption
Video Encoding
```

Virtual threads don't speed up CPU-bound computations.

---

# ThreadLocal

Virtual Threads fully support

```java
ThreadLocal
```

but...

Creating millions of virtual threads means

```
Millions Of ThreadLocal Objects
```

Use carefully.

---

# Pinning ⭐⭐⭐⭐⭐

One interview topic.

Suppose

```java
synchronized(this){
    Thread.sleep(5000);
}
```

Normally

```
Sleep
↓
Unmount
```

But inside certain synchronized/native code situations,

the virtual thread becomes

```
Pinned
```

Meaning

```
Virtual Thread
↓
Cannot Unmount
↓
Platform Thread Blocked
```

Pinning reduces the benefits of virtual threads.

Prefer

```
ReentrantLock
```

over long-running synchronized blocks in virtual-thread-heavy applications.

---

# Virtual Threads vs ExecutorService

Old

```java
ExecutorService executor =
        Executors.newFixedThreadPool(100);
```

Need to tune

```
Pool Size
Queue
Max Threads
```

Virtual

```java
Executors.newVirtualThreadPerTaskExecutor();
```

Usually

```
One Virtual Thread
↓
One Task
```

No complicated sizing.

---

# Spring Boot

Spring Boot 3.2+

Supports virtual threads.

Example

```properties
spring.threads.virtual.enabled=true
```

Tomcat can process requests using virtual threads (with compatible Java/Spring versions).

---

# Common Interview Questions

## Why were Virtual Threads introduced?

To make blocking applications highly scalable by reducing the cost of creating and managing threads.

---

## Do Virtual Threads replace Platform Threads?

No.

Virtual threads still run on platform threads.

The JVM schedules them.

---

## Are Virtual Threads daemon threads?

No.

They behave like normal threads unless configured otherwise.

---

## Can we create millions of Virtual Threads?

Yes.

That's their primary advantage.

---

## Do Virtual Threads improve CPU performance?

No.

They improve scalability for blocking workloads.

---

## Difference between Platform Thread and Virtual Thread?

| Platform Thread | Virtual Thread |
|-----------------|----------------|
| OS Managed | JVM Managed |
| Heavy | Lightweight |
| Expensive | Cheap |
| Limited | Millions Possible |

---

# Memory Trick

```
Platform Thread
↓
Heavy
↓
OS Thread
--------------------
Virtual Thread
↓
Lightweight
↓
JVM Scheduler
↓
Platform Thread
--------------------
Best For
↓
Blocking I/O
```

---

# One Complete Flow

```
HTTP Request
↓
Virtual Thread Created
↓
Calls Database
↓
Waiting
↓
Virtual Thread Unmounted
↓
Platform Thread Free
↓
Another Request Runs
↓
Database Returns
↓
Virtual Thread Mounted Again
↓
Response Sent
```

---

# Interview Summary

| Feature | Platform Thread | Virtual Thread |
|----------|-----------------|----------------|
| Managed By | Operating System | JVM |
| Creation Cost | High | Very Low |
| Memory | Higher | Lower |
| Scalability | Thousands | Hundreds of thousands to millions |
| Best Use | CPU-bound or legacy workloads | Blocking I/O, web servers, microservices |

---

# Important Interview Takeaways

- Virtual threads are **not faster threads**; they are **cheaper threads**.
- They shine for **blocking I/O**, not CPU-intensive work.
- Existing synchronous code often benefits without rewriting it to reactive programming.
- They simplify concurrent programming because you can write straightforward blocking code while achieving high scalability.
- They are a major feature of **Project Loom**, introduced as a standard feature in Java 21.

Virtual threads are lightweight threads managed by the JVM. When a virtual thread blocks on I/O, the JVM unmounts it from its underlying platform thread, allowing that platform thread to execute other virtual threads. This greatly improves scalability for blocking applications, though it doesn't make the actual business logic execute faster.
