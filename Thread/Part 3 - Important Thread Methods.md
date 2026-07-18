Before learning synchronization, we need to understand the methods available on a Thread.
We'll cover:

- start()
- run()
- sleep()
- join()
- yield()
- interrupt()
- isAlive()
- setDaemon()

# 1. start()

Creates a **new thread** and asks the JVM to schedule it.

```java
Thread t = new Thread(() -> {
    System.out.println(Thread.currentThread().getName());
});
t.start();
```

Internally:

```
Main Thread
↓
start()
↓
JVM creates a new OS Thread
↓
New Thread executes run()
```

Notice:
You **never call `run()` yourself** after `start()`.
The JVM calls it.

## Why not just call run()?

```java
t.run();
```

This is just a normal method call.

```
Main Thread
↓
run()
↓
Still Main Thread
```

No new thread is created.

## Interview Question

### What is the difference between start() and run()?

`start()` creates a new thread and then invokes `run()` on that thread.
Calling `run()` directly executes it like any other method on the current thread.

# 2. sleep()

Pauses the **current thread** for a specified duration.

```java
Thread.sleep(2000);
```

Meaning:

```
Current Thread
↓
TIMED_WAITING
↓
2 seconds
↓
RUNNABLE
```

## Important

`sleep()` does **NOT** release any locks.
Suppose:

```java
synchronized(lock) {
    Thread.sleep(5000);
}
```

The thread still owns the lock for the full 5 seconds.
Other threads remain blocked.
This is one of the most asked interview questions.

## Why use sleep()?

Usually for:

- Delay
- Retry
- Polling
- Demonstrations
    Not for synchronization.

# 3. join()

Waits for another thread to finish.
Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Child");
});
t.start();
t.join();
System.out.println("Main");
```

Output:

```
Child
Main
```

Without join():

```
Main
Child
```

or

```
Child
Main
```

Scheduling is unpredictable.

## What actually happens?

```
Main Thread
↓
join()
↓
WAITING
↓
Child finishes
↓
Main resumes
```

## Why use join()?

Suppose:

```
Download File
↓
Process File
```

Processing should not start until downloading finishes.
`join()` coordinates this dependency.

# 4. yield()

Hints to the scheduler:

> "I'm willing to let another thread run."

```java
Thread.yield();
```

Important:
It is only a **hint**.
The scheduler may ignore it.
Never depend on `yield()` for correctness.

# 5. interrupt()

Requests a thread to stop what it's doing.

```java
thread.interrupt();
```

Notice:
It **does not forcefully stop** the thread.
It simply sets the interrupt flag.
The thread must cooperate.
Example:

```java
while (!Thread.currentThread().isInterrupted()) {
    // work
}
```

## Why?

Forcefully killing a thread could leave:

- Half-written files
- Open database transactions
- Locked resources
    Java prefers cooperative cancellation.

# 6. isAlive()

Checks whether a thread is still running.

```java
Thread t = new Thread(...);
t.start();
System.out.println(t.isAlive());
```

Returns:

```
true
```

After completion:

```
false
```

# 7. setDaemon()

Marks a thread as a daemon thread.

```java
Thread t = new Thread(task);
t.setDaemon(true);
t.start();
```

Daemon threads perform background work.
Examples:

- Garbage Collector
- Monitoring
- Cleanup
    The JVM exits when all **user threads** finish, even if daemon threads are still running.

# Thread States Revisited

```
NEW
↓
start()
↓
RUNNABLE
↓
Running
↓
sleep()
↓
TIMED_WAITING
↓
RUNNABLE
↓
Finished
↓
TERMINATED
```

`join()` causes the **calling thread** to enter `WAITING`.

# Common Interview Questions

## Does sleep() release the lock?

No.
The thread sleeps while still owning any synchronized locks.

## Does join() release the lock?

`join()` itself internally uses `wait()`, so the **calling thread waits** for the target thread to finish. We'll understand the lock behavior better when we cover `wait()`.

## Can we call start() twice?

No.
A Thread object represents one execution.
Calling `start()` again throws:

```
IllegalThreadStateException
```

## Does interrupt() stop a thread immediately?

No.
It only requests interruption by setting the interrupt flag.
The thread must check the flag or respond to interruption.

# Summary

| Method      | Purpose                                  |
| ----------- | ---------------------------------------- |
| start()     | Starts a new thread                      |
| run()       | Contains the task executed by the thread |
| sleep()     | Pause the current thread                 |
| join()      | Wait for another thread to finish        |
| yield()     | Hint to the scheduler                    |
| interrupt() | Request thread interruption              |
| isAlive()   | Check if a thread is still running       |
| setDaemon() | Mark thread as a background thread       |

# Understanding

Think of these methods as controlling a worker.

```
start()
↓
Hire Worker
--
sleep()
↓
Take a Nap
--
join()
↓
Wait for Worker
--
interrupt()
↓
Ask Worker to Stop
--
yield()
↓
Let Someone Else Work
--
isAlive()
↓
Is Worker Still Working?
--
setDaemon()
↓
Background Worker
```
