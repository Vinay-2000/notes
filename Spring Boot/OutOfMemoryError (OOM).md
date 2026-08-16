# Java OutOfMemoryError (OOM) — Practical Debugging Notes

## Starting Question

> **Your Java application suddenly starts throwing `OutOfMemoryError`. How will you debug it?**

The first step is to identify **what kind of memory is exhausted** from the exact OOM message.

Common examples:

- `java.lang.OutOfMemoryError: Java heap space` → Java heap is exhausted.
- `java.lang.OutOfMemoryError: GC overhead limit exceeded` → JVM is spending excessive time doing GC but recovering very little memory.
- `java.lang.OutOfMemoryError: Metaspace` → class metadata/native Metaspace is exhausted.
- `java.lang.OutOfMemoryError: Direct buffer memory` → off-heap/direct buffer memory problem.

For a heap-related OOM:

1. Check heap and GC metrics.
2. Determine whether memory continuously grows.
3. Take a heap dump.
4. Analyze the heap dump using a tool such as Eclipse MAT.
5. Find which objects consume/retain the memory.
6. Trace why those objects are still reachable.
7. Fix the root cause and verify the behavior again.

---

# 1. What Is a Memory Leak?

A memory leak occurs when an object is **no longer logically needed by the application but is still reachable through references**, so the Garbage Collector cannot reclaim it.

The important distinction is:

> **GC only knows whether an object is reachable. It does not know whether the application still logically needs it.**

### Normal memory usage

```text
Object created
     ↓
Object used
     ↓
No references remain
     ↓
GC runs
     ↓
Object can be collected
```

### Memory leak

```text
Object created
     ↓
Object no longer logically needed
     ↓
Some long-lived object still references it
     ↓
GC sees it as reachable
     ↓
Object cannot be collected
     ↓
Memory keeps growing
     ↓
Eventually → OutOfMemoryError
```

High memory usage by itself does **not** necessarily mean a memory leak. If GC can reclaim unused objects and memory comes back down, that can be normal.

---

# 2. Common Causes of Java Memory Leaks

Typical causes include:

- Unbounded `List`, `Map`, `Set`, or `Queue`
- Unbounded caches
- Static collections holding objects forever
- `ThreadLocal` misuse
- Event listeners/callbacks that are never removed
- ClassLoader leaks
- Objects accidentally retained by long-lived application components

A `static` collection is not automatically a memory leak. The problem is when a long-lived collection continuously grows without removing objects.

---

# 3. How Can an OOM Occur?

There are several ways an application can run out of memory.

## A. Heap exhaustion

The application keeps creating/retaining objects until the Java heap cannot satisfy another allocation.

Example:

```java
List<byte[]> cache = new ArrayList<>();

while (true) {
    cache.add(new byte[1024 * 1024]);
}
```

The list keeps references to the byte arrays, so they remain reachable.

Eventually:

```text
java.lang.OutOfMemoryError: Java heap space
```

## B. Metaspace exhaustion

Class metadata is stored in Metaspace in modern Java.

Possible causes include:

- Excessive class generation
- ClassLoader leaks
- Dynamically generated classes that are never unloaded

The error can be:

```text
java.lang.OutOfMemoryError: Metaspace
```

## C. Direct/native memory exhaustion

Java processes also use memory outside the Java heap.

Examples:

- Direct byte buffers
- Thread stacks
- Metaspace
- Code cache
- Other native allocations

Therefore:

> `-Xmx` limits the Java heap, not the entire memory footprint of the JVM process.

---

# 4. `-X` and `-XX` JVM Options

JVM options are passed when starting the Java application.

## `-Xms`

Initial Java heap size:

```text
-Xms128m
```

This means the JVM starts with an initial heap size of approximately 128 MB.

## `-Xmx`

Maximum Java heap size:

```text
-Xmx256m
```

This means the Java heap can grow up to approximately 256 MB.

So:

```text
-Xms128m -Xmx256m
```

roughly means:

```text
Initial heap → 128 MB
       ↓
Heap can grow as needed
       ↓
Maximum heap → 256 MB
```

`-Xmx256m` does **not** mean the entire Java process is limited to 256 MB.

## `-XX`

`-XX` options are used for advanced JVM configuration/tuning.

Boolean option:

```text
-XX:+HeapDumpOnOutOfMemoryError
```

`+` means enable it.

A boolean option can be disabled with:

```text
-XX:-SomeOption
```

An option with a value looks like:

```text
-XX:HeapDumpPath=./heapdump.hprof
```

---

# 5. Creating a Heap Dump Automatically

The JVM can automatically create a heap dump when an OOM occurs.

Use:

```text
-XX:+HeapDumpOnOutOfMemoryError
```

To specify the dump location:

```text
-XX:HeapDumpPath=./heapdump.hprof
```

For our IntelliJ experiment we used:

```text
-Xms128m -Xmx256m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./heapdump.hprof
```

In IntelliJ:

**Run → Edit Configurations → Spring Boot/Java application → VM options**

Put the JVM options in **VM options**, not Program arguments or `application.properties`.

---

# 6. Important: What Does a Heap Dump Do?

A heap dump is a **snapshot of the objects in the JVM heap**.

It does not fix the memory leak.

The sequence is:

```text
Application
     ↓
Heap fills
     ↓
GC attempts to reclaim memory
     ↓
Still not enough memory
     ↓
OutOfMemoryError
     ↓
JVM creates heap dump
```

If the application terminates, the operating system eventually releases all memory belonging to the process.

The `.hprof` file itself is only evidence for investigation.

---

# 7. Practical Experiment We Created

We created a simple Java application that deliberately simulates an unbounded cache.

```java
import java.util.ArrayList;
import java.util.List;

public class MemoryLeakDemo {

    private static final List<byte[]> CACHE = new ArrayList<>();

    public static void main(String[] args) throws Exception {

        System.out.println("Application started...");

        while (true) {
            CACHE.add(new byte[1024 * 1024]); // 1 MB

            System.out.println("Cache size: " + CACHE.size() + " MB");

            Thread.sleep(1000);
        }
    }
}
```

Every iteration:

```java
CACHE.add(new byte[1024 * 1024]);
```

creates approximately 1 MB of byte-array data and stores it in `CACHE`.

Because `CACHE` is a long-lived static collection, the objects remain reachable.

Conceptually:

```text
MemoryLeakDemo
      ↓
    CACHE
      ↓
  ArrayList
      ↓
  byte[]
  byte[]
  byte[]
  ...
```

Therefore GC cannot remove those byte arrays.

---

# 8. Reproducing the OOM

We deliberately limited the heap:

```text
-Xms128m
-Xmx256m
```

and enabled automatic heap dumps:

```text
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./heapdump.hprof
```

The application eventually stopped with:

```text
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "main"
```

and created:

```text
./heapdump.hprof
```

We then used that file for analysis.

---

# 9. Analyzing the Heap Dump with Eclipse MAT

We used **Eclipse Memory Analyzer (MAT)**.

Open:

```text
heapdump.hprof
```

in MAT and inspect the **Dominator Tree**.

Useful columns include:

- Class Name
- Shallow Heap
- Retained Heap

### Shallow Heap

Memory used by the object itself.

### Retained Heap

The memory that would become reclaimable if that object/reference were removed.

For our example, MAT showed approximately:

```text
MemoryLeakDemo
    ↓
CACHE : java.util.ArrayList
    ↓
elementData : Object[]
    ↓
byte[]
byte[]
byte[]
...
```

The important observation was that approximately 99% of the heap was being retained through `CACHE`.

The result was effectively:

```text
MemoryLeakDemo
      ↓
    CACHE
      ↓
   ArrayList
      ↓
  Object[] elementData
      ↓
   byte[]
   byte[]
   byte[]
      ...
```

This proved **why the byte arrays were still alive**.

---

# 10. What MAT Told Us

MAT answered:

> **Why is the memory still being retained?**

The answer was:

```text
CACHE → ArrayList → byte[]
```

The byte arrays were still reachable through the static `CACHE`, so GC could not collect them.

This is the important distinction:

```text
JVM has a lot of objects
        ↓
Are they still reachable?
        ↓
YES
        ↓
GC cannot remove them
        ↓
Potential memory leak
```

---

# 11. JFR — Java Flight Recorder

We then investigated the application using **JFR (Java Flight Recorder)**.

JFR records JVM/application activity over time.

It can provide information about:

- Memory
- Object allocations
- Garbage collection
- Threads
- CPU
- Locks
- I/O
- Exceptions
- Other JVM events

JFR is useful because a heap dump is a **snapshot**, while JFR shows **what was happening over a period of time**.

---

# 12. Starting JFR with `jcmd`

First find the Java process:

```bash
jps -l
```

Example:

```text
12345 MemoryLeakDemo
```

Start a JFR recording:

```bash
jcmd 12345 JFR.start name=OOMDemo settings=profile
```

Let the application run for a while.

Then explicitly save the recording while the JVM is still alive:

```bash
jcmd 12345 JFR.dump name=OOMDemo filename=oom-demo.jfr
```

This produced:

```text
oom-demo.jfr
```

### Why we used `JFR.dump`

Initially, we tried a timed recording:

```bash
jcmd <PID> JFR.start name=OOMDemo duration=2m filename=oom-demo.jfr
```

The application reached OOM before the recording was successfully written, resulting in a 0 KB file.

Using:

```text
JFR.start
    ↓
let application run
    ↓
JFR.dump
```

lets us explicitly save the recording while the JVM is still alive.

---

# 13. Analyzing JFR with JMC

We opened the `.jfr` recording using **JDK Mission Control (JMC)**.

In JMC, the important section for our experiment was **Memory**.

The JFR result showed:

```text
Class: byte[]
Allocation: 148 MiB
Total Allocation: 100%
```

So the captured allocations were entirely `byte[]`.

The memory graph showed memory continuously increasing:

```text
~64 MB
   ↓
~80 MB
   ↓
~100 MB
   ↓
~130 MB
   ↓
~160 MB
   ↓
~190 MB
   ↓
~220+ MB
```

The graph was essentially a continuous upward trend.

---

# 14. What JFR Told Us

JFR answered:

> **What is happening over time?**

We saw:

```text
byte[] allocations
       ↓
Heap usage increases
       ↓
GC occurs
       ↓
Memory is not sufficiently reclaimed
       ↓
Heap continues increasing
       ↓
Eventually OOM
```

This complements the MAT result.

MAT told us:

> The `byte[]` objects are retained by `CACHE`.

JFR told us:

> The application keeps allocating `byte[]`, and memory usage continuously increases.

---

# 15. MAT vs JFR

| Tool | Main Question |
|---|---|
| **Heap Dump + MAT** | What objects are occupying/retaining memory right now? |
| **JFR + JMC** | What is happening in the JVM over time? |

### MAT

Good for:

- Finding retained objects
- Finding memory leaks
- Dominator tree
- Retained heap
- Reference chains
- Finding GC roots

### JFR/JMC

Good for:

- Memory behavior over time
- Allocation rates
- GC behavior
- CPU
- Threads
- Locks
- I/O
- Exceptions
- Finding where activity is occurring

They complement each other.

---

# 16. Our Complete Investigation

We started with:

```text
OutOfMemoryError
```

Then:

```text
1. Configure JVM
       ↓
-Xms128m
-Xmx256m
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./heapdump.hprof
       ↓
2. Reproduce OOM
       ↓
3. Get heapdump.hprof
       ↓
4. Open in MAT
       ↓
5. Dominator Tree
       ↓
6. Find byte[]
       ↓
7. Follow references
       ↓
CACHE → ArrayList → byte[]
       ↓
8. Root cause identified
```

Then we used JFR:

```text
1. Start application
       ↓
2. jps -l
       ↓
3. jcmd <PID> JFR.start
       ↓
4. Let application run
       ↓
5. jcmd <PID> JFR.dump
       ↓
6. Open .jfr in JMC
       ↓
7. Memory → Allocations
       ↓
byte[] = 100% of captured allocation
       ↓
Heap continuously increases
```

Together, we established:

```text
Continuous byte[] allocation
          +
byte[] retained by CACHE
          +
Heap continuously growing
          ↓
       Memory leak
          ↓
   OutOfMemoryError
```

---

# 17. Practical Mental Model

When you see:

```text
OutOfMemoryError
```

think:

```text
                 OOM
                  ↓
        What memory is exhausted?
                  ↓
       ┌──────────┴──────────┐
       ↓                     ↓
      Heap              Metaspace/native
       ↓
  Heap Dump + MAT
       ↓
What objects dominate?
       ↓
Why are they retained?
       ↓
Find root cause
```

And if you need to understand the **behavior leading up to the problem**:

```text
JFR
 ↓
Memory
 ↓
Allocation
 ↓
GC
 ↓
Threads / CPU / other events
```

### The core takeaway

> **Heap dump + MAT tells you what is retained and why.**
>
> **JFR + JMC tells you what the JVM was doing over time.**

For a real production memory problem, using both gives you a much stronger diagnosis than simply increasing `-Xmx`.
