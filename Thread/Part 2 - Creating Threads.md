
## Why Do We Need Multiple Ways to Create Threads?

Think about what a Thread actually is.

A Thread has **two responsibilities**:

1. **The Thread itself** (managed by the JVM and OS)
2. **The work to execute**

Java separates these two responsibilities.

Think of it like this:

```
Thread (Worker)

        +

Task (Work)
```

The worker and the work are different things.

This is exactly why Java provides `Runnable`, `Callable`, and `Thread`.

---

# 1. Extending Thread

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Running...");
    }

}

MyThread thread = new MyThread();
thread.start();
```

### What happens internally?

```
MyThread Object

↓

start()

↓

JVM creates a new OS Thread

↓

JVM invokes run() on that new thread
```

Notice:

We never call `run()` ourselves.

The JVM does.

---

## Why is this usually NOT preferred?

Because Java supports **single inheritance**.

```
class MyThread extends Thread
```

Now the class cannot extend anything else.

Example:

```
class EmployeeService extends Thread
```

Later:

```
class EmployeeService extends SomeBaseClass
```

Impossible.

Your inheritance is already consumed.

---

# 2. Implementing Runnable

```java
class MyTask implements Runnable {

    @Override
    public void run() {
        System.out.println("Running...");
    }

}

Thread thread = new Thread(new MyTask());

thread.start();
```

Notice something interesting.

The Thread is created separately.

The task is created separately.

```
Runnable

↓

Task

----------------

Thread

↓

Executes Task
```

Much cleaner.

---

## Why did Java introduce Runnable?

Imagine 5 different tasks.

```
Download File

Upload File

Send Email

Generate Report

Compress Images
```

Should all of these extend Thread?

No.

A Thread is just a worker.

The tasks are different.

Instead:

```
Task

↓

Runnable

↓

Thread executes Runnable
```

Now one Thread class can execute any Runnable.

This is called **Separation of Concerns**.

---

# Why is Runnable Preferred?

Because it separates:

```
What to execute

(Runnable)

from

Who executes it

(Thread)
```

This becomes even more important later with:

- Thread Pools
- ExecutorService
- CompletableFuture

Those APIs don't care about Thread subclasses.

They execute Runnable or Callable tasks.

---

# 3. Callable

Runnable has one limitation.

```
Runnable

↓

run()

↓

No return value

No checked exception
```

Suppose we want:

```
Calculate Salary

↓

Return Result
```

Runnable cannot do this.

Java introduced Callable.

```java
class SalaryTask implements Callable<Integer> {

    @Override
    public Integer call() {
        return 50000;
    }

}
```

Notice:

```
Runnable

↓

run()

↓

void
```

vs

```
Callable

↓

call()

↓

Returns Value
```

Callable can also throw checked exceptions.

---

# Runnable vs Callable

| Runnable | Callable |
|-----------|-----------|
| run() | call() |
| Returns void | Returns a value |
| Cannot throw checked exceptions | Can throw checked exceptions |
| Introduced in Java 1.0 | Introduced in Java 5 |
| Used with Thread | Used with ExecutorService |

---

# Thread.start() vs run()

This is probably the MOST ASKED thread question.

Suppose:

```java
Thread t = new Thread(() -> {
    System.out.println(Thread.currentThread().getName());
});
```

### Calling run()

```java
t.run();
```

What happens?

Nothing special.

It's just a normal method call.

```
Main Thread

↓

run()

↓

Still Main Thread
```

No new thread is created.

---

### Calling start()

```java
t.start();
```

```
Main Thread

↓

JVM

↓

Creates New Thread

↓

New Thread executes run()
```

Now two threads exist.

---

# Why can't we call start() twice?

```java
Thread t = new Thread(...);

t.start();

t.start();
```

Throws:

```
IllegalThreadStateException
```

Why?

A Thread object represents one execution.

Once that execution finishes:

```
NEW

↓

RUNNABLE

↓

TERMINATED
```

It cannot go back to NEW.

Create a new Thread instead.

---

# Understanding

Think of a Thread like a bullet.

```
Bullet

↓

Fire

↓

Finished
```

You cannot fire the same bullet again.

Create another bullet.

The same applies to Thread objects.

---

# Interview Questions

## Why is Runnable preferred over extending Thread?

Because it separates the task from the thread, supports single inheritance, and works seamlessly with ExecutorService and thread pools.

---

## Why was Callable introduced?

Runnable cannot return a value or throw checked exceptions. Callable solves both problems.

---

## Difference between start() and run()?

`start()` creates a new thread and then invokes `run()`.

Calling `run()` directly executes it as a normal method on the current thread.

---

## Can we restart a Thread?

No.

A Thread object represents a single execution. Once terminated, it cannot be started again.

---

# Memory Trick

```
Thread

↓

Worker

----------------

Runnable

↓

Work

----------------

Callable

↓

Work + Result
```