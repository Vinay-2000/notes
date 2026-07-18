## Why Do We Need wait() and notify()?

Suppose two threads are working together.

```
Producer
↓
Produces Data
-
Consumer
↓
Consumes Data
```

Imagine there is one shared queue.

```java
Queue<Integer> queue = new LinkedList<>();
```

Initially:

```
Queue
↓
Empty
```

Consumer starts first.

```
Consumer
↓
Remove Item
```

But...
There is nothing to remove.
What should the consumer do?

## Option 1 - Keep Checking (Busy Waiting)

```java
while(queue.isEmpty()) {
}
```

This is called **Busy Waiting**.
The thread continuously checks:

```
Empty?
↓
Yes
↓
Check Again
↓
Yes
↓
Check Again
```

CPU usage becomes:

```
100%
```

Terrible design.

## What We Really Want

Consumer should say:

> "I'm going to sleep until someone adds data."
> That's exactly what `wait()` does.

# wait()

```java
synchronized(queue) {
    while(queue.isEmpty()) {
        queue.wait();
    }
}
```

Think of it as:

```
Queue Empty
↓
Consumer Sleeps
↓
Consumes NO CPU
```

Unlike sleep(),
the consumer doesn't wake after a fixed time.
It waits until someone wakes it.

# notify()

Producer adds data.

```java
synchronized(queue) {
    queue.add(10);
    queue.notify();
}
```

Meaning:

```
Producer
↓
Adds Data
↓
Wake One Waiting Thread
```

The consumer wakes up.

# Internal Flow

Initially

```
Queue Empty
↓
Consumer
↓
wait()
↓
WAITING
```

Producer

```
Add Data
↓
notify()
↓
Consumer becomes RUNNABLE
↓
Consumer Continues
```

# Why Must wait() Be Inside synchronized?

This is probably the most asked question.
Suppose:

```java
queue.wait();
```

without synchronization.
Which thread owns the queue's monitor?
Nobody.
Now imagine:

```
Consumer
↓
wait()
```

Who should release the lock?
There isn't one.
Java therefore requires:

```
Acquire Monitor
↓
wait()
↓
Release Monitor
↓
Sleep
```

Only the thread holding the monitor can call `wait()`.
Otherwise:

```
IllegalMonitorStateException
```

is thrown.

# The Most Important Difference

## sleep()

```
Thread Sleeps
↓
Keeps Lock
↓
Other Threads Block
```

## wait()

```
Thread Waits
↓
Releases Lock
↓
Other Threads Can Enter
```

This is the biggest conceptual difference.

# Why Does wait() Release the Lock?

Imagine it didn't.
Consumer:

```
Acquire Lock
↓
Queue Empty
↓
wait()
```

If it kept the lock:
Producer

```
Needs Lock
↓
Blocked Forever
```

Producer could never add data.
Consumer could never wake.
Deadlock.
Therefore Java does:

```
Acquire Lock
↓
wait()
↓
Release Lock
↓
Sleep
```

Now producer can acquire the lock.

# notify()

Wakes **one** waiting thread.

```java
lock.notify();
```

Think:

```
Wake One
```

# notifyAll()

Wakes **all** waiting threads.

```java
lock.notifyAll();
```

Think:

```
Wake Everyone
```

Only one thread will eventually acquire the monitor.
The others compete for it.

# Why Use while Instead of if?

Bad

```java
if(queue.isEmpty()) {
    queue.wait();
}
```

Good

```java
while(queue.isEmpty()) {
    queue.wait();
}
```

Why?
A thread may wake up even though the condition is still false.
This is called a **spurious wakeup**.
Always recheck the condition.

# Producer Consumer Example

Consumer

```java
synchronized(queue){
    while(queue.isEmpty()){
        queue.wait();
    }
    queue.remove();
}
```

Producer

```java
synchronized(queue){
    queue.add(10);
    queue.notify();
}
```

# Thread State

Consumer

```
RUNNABLE
↓
wait()
↓
WAITING
↓
notify()
↓
BLOCKED (waiting for monitor)
↓
RUNNABLE
↓
Running
```

Notice:
After notify(),
the thread does NOT immediately continue.
It must reacquire the monitor first.

# Common Interview Questions

## Does wait() release the lock?

Yes.
Immediately.

## Does sleep() release the lock?

No.
The thread sleeps while still owning the monitor.

## Why must wait() be inside synchronized?

Because the thread must own the object's monitor before it can release it and enter the waiting state.

## Does notify() release the lock?

No.
It only wakes a waiting thread.
The notifying thread continues executing until it exits the synchronized block.
Only then can the awakened thread reacquire the monitor.

## notify() vs notifyAll()

`notify()` wakes one waiting thread.
`notifyAll()` wakes all waiting threads.
The awakened threads still compete to reacquire the monitor.

# Memory Trick

```
sleep()
↓
Pause
↓
Keep Lock
wait()
↓
Pause
↓
Release Lock
↓
Wait for notify()
notify()
↓
Wake One
notifyAll()
↓
Wake Everyone
```
