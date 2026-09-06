Java memory management is handled by the JVM. 
The main memory areas I consider are the heap, stack, Metaspace, PC register and native method stack. 
Objects and arrays are generally allocated on the heap, which is shared across threads and managed by the Garbage Collector.
Each thread has its own stack containing method frames, local variables and references. 
Metaspace stores class metadata in modern HotSpot JVMs. 
When objects are no longer reachable from GC Roots, they become eligible for garbage collection and their heap memory can be reclaimed. Java manages memory automatically, but memory leaks can still occur when unnecessary objects remain reachable, for example through static collections or improperly managed caches.

![[MemoryManagement.png]]