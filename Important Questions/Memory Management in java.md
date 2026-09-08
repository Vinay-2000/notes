Java memory management is handled by the JVM. 
The main memory areas I consider are the heap, stack, Metaspace, PC register and native method stack. 
Objects and arrays are generally allocated on the heap, which is shared across threads and managed by the Garbage Collector.
Each thread has its own stack containing method frames, local variables and references. 
Metaspace stores class metadata in modern HotSpot JVMs. 
When objects are no longer reachable from GC Roots, they become eligible for garbage collection and their heap memory can be reclaimed. Java manages memory automatically, but memory leaks can still occur when unnecessary objects remain reachable, for example through static collections or improperly managed caches.

---

| Java 7 and earlier                                           | Java 8+                                                                                  |
| ------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| **PermGen** used for class metadata                          | **Metaspace** used for class metadata                                                    |
| PermGen was part of JVM-managed heap space                   | Metaspace is in **native memory**                                                        |
| PermGen had a fixed/configured maximum (`-XX:MaxPermSize`)   | Metaspace can grow using native memory, with `-XX:MaxMetaspaceSize` as an optional limit |
| String pool was in PermGen                                   | **String pool moved to the heap**                                                        |
| Class metadata could cause `OutOfMemoryError: PermGen space` | Excessive class metadata can cause `OutOfMemoryError: Metaspace`                         |

```
Employee class
│
├── static int count = 10
│       └── primitive value
│
└── static ArrayList names
        └── reference ──────────────┐
                                    ↓
                              Heap
                              ┌───────────────┐
                              │ ArrayList     │
                              │ elementData ──┼──→ String objects
                              └───────────────┘
```

There is **no separate heap `Integer` object** here because `int` is primitive.

---
![[MemoryManagement.png]]