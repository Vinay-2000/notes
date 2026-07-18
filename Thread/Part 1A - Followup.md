# Interview Point 1: Why does each thread have its own stack?

Each thread executes independently. If threads shared the same stack, they would overwrite each other's method calls, local variables, and return addresses, making correct execution impossible.

---

# Interview Point 2: What does a thread's stack contain?

Each thread's stack contains **stack frames** for active method calls.

Each stack frame contains:

- Local variables
- Method parameters
- Operand stack (temporary values)
- Return address

The actual method code is **not** stored in the stack.

---

# Interview Point 3: Where is the method code stored?

Method bytecode is stored only once in the JVM's **Method Area (Metaspace)**.

All threads execute the same method code but have different stack frames.

---

# Interview Point 4: Why are local variables thread-safe?

Local variables are stored in the thread's private stack.

No other thread can access that stack.

Hence synchronization is not required.

---

# Interview Point 5: Why are instance variables not thread-safe?

Instance variables belong to objects stored in the heap.

Since the heap is shared by all threads, multiple threads can modify the same variable simultaneously, causing race conditions.

---

# Interview Point 6: Can two threads execute the same method simultaneously?

Yes.

The method code is shared, but each thread gets its own stack frame.

```
Method Code (Shared)

        calculate()

           ▲
           │
 ┌─────────┴─────────┐
 │                   │
Thread 1         Thread 2
 Stack             Stack
```

---

# Interview Point 7: What causes a Race Condition?

A race condition occurs when multiple threads access and modify the same shared data without proper synchronization, causing the result to depend on the execution order.

Example:

```
count++;
```

Internally:

```
Read↓Increment↓Write
```

These are three separate operations, not one atomic operation.

---

# Interview Point 8: Which memory is shared and which is private?

|Memory Area|Shared?|
|---|---|
|Heap|✅ Shared|
|Method Area (Metaspace)|✅ Shared|
|Stack|❌ One per thread|
|Program Counter (PC)|❌ One per thread|
|Native Method Stack|❌ One per thread|

---

# Interview Point 9: Why is context switching between threads cheaper than between processes?

Threads share the same heap and method area, so only thread-specific state (stack, program counter, registers) needs to be switched.

Processes have separate memory spaces, making context switching much more expensive.

---

# Interview Point 10: One-liner Interview Answer

> Each Java thread has its own private stack containing stack frames for active method calls, while all threads share the heap where objects and instance variables reside. This separation makes local variables inherently thread-safe, whereas shared objects on the heap require synchronization to prevent race conditions.

These 10 points cover nearly all the common follow-up questions interviewers ask immediately after introducing Java threads.