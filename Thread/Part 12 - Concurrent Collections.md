

# Why Were Concurrent Collections Introduced?

Suppose we have:

```java
Map<Integer, String> map = new HashMap<>();
```

Thread 1

```java
map.put(1, "A");
```

Thread 2

```java
map.put(2, "B");
```

At the same time.

Question:

Is HashMap thread-safe?

```
No
```

Problems can occur:

- Race Conditions
- Lost Updates
- Corrupted Internal Structure
- ConcurrentModificationException (during iteration)

Java introduced thread-safe collections.

---

# Concurrent Collections

Most commonly used:

```
ConcurrentHashMap

CopyOnWriteArrayList

BlockingQueue
```

These are the three you should know very well.

---

# 1. ConcurrentHashMap ⭐⭐⭐⭐⭐

Most asked interview question.

---

## Why not Hashtable?

Old Java had:

```java
Hashtable
```

It synchronized every operation.

```
put()

↓

Entire Table Locked
```

Even two threads writing to different keys had to wait.

Poor scalability.

---

ConcurrentHashMap improves this.

Instead of locking everything:

```
Thread A

↓

Bucket 1

----------------

Thread B

↓

Bucket 10
```

Can proceed simultaneously.

Much higher throughput.

---

## Example

```java
ConcurrentHashMap<Integer, String> map =
        new ConcurrentHashMap<>();

map.put(1, "Java");

map.put(2, "Spring");
```

Safe even with many threads.

---

## Why Faster?

Hashtable

```
Entire Map Locked
```

ConcurrentHashMap

```
Fine-grained locking

+

CAS

+

Lock only when needed
```

Much better concurrency.

---

# Can ConcurrentHashMap Store null?

```java
map.put(null, "Java");
```

❌ Exception

```
NullPointerException
```

Unlike HashMap.

Reason:

Suppose

```java
map.get(1)
```

returns

```
null
```

Question:

Did

```
Key Not Exist
```

or

```
Value Is Null
```

ConcurrentHashMap cannot distinguish this safely during concurrent access.

So nulls are prohibited.

---

# 2. CopyOnWriteArrayList ⭐⭐⭐⭐☆

Think:

```
Many Reads

Very Few Writes
```

Perfect example:

```
Application Configuration

Country List

Product Categories
```

Mostly read.

Rarely modified.

---

How does it work?

Suppose

```
[1,2,3]
```

Thread writes:

```java
list.add(4);
```

Java creates:

```
Old

[1,2,3]

↓

New Copy

[1,2,3,4]
```

Readers continue using the old array until the new one is ready.

No locking for readers.

---

Advantage

```
Reads

Very Fast
```

Disadvantage

```
Writes

Expensive

(New Array Every Time)
```

---

# Example

```java
CopyOnWriteArrayList<String> list =
        new CopyOnWriteArrayList<>();

list.add("Java");

list.add("Spring");
```

Perfect for read-heavy applications.

---

# 3. BlockingQueue ⭐⭐⭐⭐⭐

Probably the most practical concurrent collection.

Imagine Producer Consumer.

Producer

```
Create Tasks
```

Consumer

```
Process Tasks
```

Need a queue.

---

Without BlockingQueue

Consumer keeps checking:

```java
while(queue.isEmpty()){

}
```

Busy waiting.

Waste of CPU.

---

BlockingQueue solves this.

Producer

```java
queue.put(task);
```

Consumer

```java
queue.take();
```

If queue empty

↓

Consumer sleeps automatically.

No busy waiting.

---

Example

```java
BlockingQueue<Integer> queue =
        new LinkedBlockingQueue<>();

queue.put(10);

int value = queue.take();
```

---

# put()

Queue Full?

↓

Wait.

---

# take()

Queue Empty?

↓

Wait.

Automatic synchronization.

---

# Common Implementations

```
LinkedBlockingQueue

ArrayBlockingQueue

PriorityBlockingQueue

DelayQueue
```

---

# Which One Is Used Most?

```
LinkedBlockingQueue
```

Very common.

---

# Interview Questions

## HashMap vs ConcurrentHashMap

| HashMap | ConcurrentHashMap |
|----------|-------------------|
| Not Thread-safe | Thread-safe |
| Allows null | No nulls |
| No Synchronization | Fine-grained locking + CAS |

---

## ArrayList vs CopyOnWriteArrayList

| ArrayList | CopyOnWriteArrayList |
|------------|----------------------|
| Not Thread-safe | Thread-safe |
| Fast Writes | Slow Writes |
| Normal Reads | Extremely Fast Reads |

---

## Queue vs BlockingQueue

Queue

```
poll()

↓

Returns null
```

BlockingQueue

```
take()

↓

Wait Until Item Exists
```

---

# Memory Trick

ConcurrentHashMap

↓

Concurrent Map

-------------------

CopyOnWriteArrayList

↓

Read Heavy

-------------------

BlockingQueue

↓

Producer Consumer

| Problem                                  | Solution               |
| ---------------------------------------- | ---------------------- |
| Multiple threads updating a map          | `ConcurrentHashMap`    |
| Many readers, very few writers           | `CopyOnWriteArrayList` |
| Producer and Consumer need to coordinate | `BlockingQueue`        |