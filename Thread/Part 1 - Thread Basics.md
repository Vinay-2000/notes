
# What is a Process?

A **Process** is an independent program in execution.

When you open an application like Chrome, IntelliJ, or Spotify, the operating system creates a separate process for it.

Each process has its own:

- Memory (Heap)
    
- Code
    
- Resources
    
- Threads


Example:

```text
Chrome.exe

↓

Process
```

---

# What is a Thread?

A **Thread** is the smallest unit of execution within a process.

A process always has at least one thread called the **Main Thread**.

Example:

```text
Java Application

↓

Process

↓

Main Thread
```

A process can create multiple threads to perform tasks concurrently.

---

# Process vs Thread

|Process|Thread|
|---|---|
|Independent program|Smallest execution unit inside a process|
|Has its own memory|Shares process memory|
|Heavyweight|Lightweight|
|Communication is expensive (IPC)|Communication is easy (shared memory)|
|Context switch is slower|Context switch is faster|

---

# Example

Suppose you open IntelliJ.

```text
Operating System

↓

IntelliJ Process

↓

Main Thread

Background Indexing Thread

Auto Save Thread

UI Thread

Git Thread
```

All these threads belong to the same process and share its memory.

---

# Why Do We Need Multithreading?

Without multithreading:

```text
Download File

↓

Wait

↓

Update UI
```

The application freezes until the download completes.

With multithreading:

```text
Main Thread

↓

Update UI

------------

Worker Thread

↓

Download File
```

The UI remains responsive while the download happens in the background.

---

# Advantages

- Better CPU utilization
    
- Faster execution
    
- Responsive applications
    
- Concurrent task execution
    
- Improved throughput
    

---

# Does Multithreading Always Make Programs Faster?

No.

If tasks are:

- Small
    
- Sequential
    
- Highly dependent
    

Creating extra threads may actually reduce performance because of thread creation and context switching overhead.

---

# Thread Lifecycle

A thread goes through several states during its lifetime.

```text
NEW

↓

RUNNABLE

↓

RUNNING (chosen by CPU scheduler)

↓

BLOCKED / WAITING / TIMED_WAITING

↓

RUNNABLE

↓

TERMINATED
```

Java combines **READY** and **RUNNING** into the `RUNNABLE` state.

---

# Thread States

## NEW

Thread object is created but `start()` has not been called.

```java
Thread t = new Thread();
```

State:

```text
NEW
```

---

## RUNNABLE

`start()` is called.

The thread is ready to run and is waiting for CPU time.

```java
t.start();
```

State:

```text
RUNNABLE
```

---

## BLOCKED

The thread is waiting to acquire a monitor lock (`synchronized`).

Example:

Thread A owns the lock.

Thread B tries to enter the synchronized block.

Thread B becomes:

```text
BLOCKED
```

---

## WAITING

The thread waits indefinitely until another thread wakes it.

Examples:

```java
wait()

join()
```

---

## TIMED_WAITING

Waiting for a specified time.

Examples:

```java
Thread.sleep(1000)

join(1000)

wait(1000)
```

---

## TERMINATED

The thread finishes execution.

```text
run()

↓

Completed

↓

TERMINATED
```

---

# User Thread vs Daemon Thread

## User Thread

Performs application work.

The JVM waits for all user threads to finish before shutting down.

Examples:

- Main Thread
    
- Business Logic
    
- Request Processing
    

---

## Daemon Thread

Background support thread.

The JVM does **not** wait for daemon threads.

Examples:

- Garbage Collector
    
- Background Cleanup
    
- Monitoring
    

Example:

```java
Thread t = new Thread(task);

t.setDaemon(true);
```

---

# Common Interview Questions

## Does every Java program have a thread?

Yes.

Every Java application starts with one **Main Thread**.

---

## Can a process exist without a thread?

No.

Every process has at least one thread.

---

## Can threads communicate easily?

Yes.

Threads share the same process memory.

---

## Why are threads called lightweight?

Because creating and switching threads is much cheaper than creating and switching processes.

---

## Why is context switching between processes expensive?

Processes have separate memory spaces and resources, so the operating system must save and restore more state during a switch.

Threads share the same memory, making switching much faster.

---

# Memory Layout

```text
Java Process

+----------------------+
| Heap (Shared)        |
|                      |
| Thread A             |
| Thread B             |
| Thread C             |
|                      |
+----------------------+

Each Thread Has:

- Program Counter (PC)
- Java Stack
- Native Stack
```

The **Heap** is shared by all threads.

Each thread has its own **stack**, which stores method calls and local variables.

---

# Interview Answer

A process is an independent program in execution with its own memory and resources. A thread is the smallest unit of execution within a process. Multiple threads in the same process share memory but have separate stacks and program counters. Multithreading improves responsiveness and CPU utilization by allowing tasks to execute concurrently, but it also introduces challenges such as synchronization and thread safety.