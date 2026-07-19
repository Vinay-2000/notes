## List implementations

| Feature               | ArrayList                                       | LinkedList                  | Vector                            | Stack                                                                                    | CopyOnWriteArrayList                    |
| --------------------- | ----------------------------------------------- | --------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------- |
| **Implementation**    | Resizable `Object[]`maintains size and capacity | Doubly Linked List (`Node`) | Synchronized Resizable `Object[]` | Extends `Vector` (LIFO)<br>has push pop peek methods which does the operation on Vector. | Copies entire `Object[]` on every write |
| **Backing Structure** | Dynamic Array                                   | Doubly Linked List          | Dynamic Array                     | Dynamic Array                                                                            | Dynamic Array                           |
| **Thread Safe**       | ❌                                              | ❌                          | ✅                                | ✅                                                                                       | ✅                                      |
| **Random Access**     | O(1)                                            | O(n)                        | O(1)                              | O(1)                                                                                     | O(1)                                    |
| **Add at End**        | Amortized O(1)                                  | O(1)                        | Amortized O(1)                    | O(1)                                                                                     | O(n)                                    |
| **Insert Middle**     | O(n)                                            | O(n)                        | O(n)                              | O(n)                                                                                     | O(n)                                    |
| **Iterator**          | Fail-Fast                                       | Fail-Fast                   | Fail-Fast                         | Fail-Fast                                                                                | Snapshot                                |
| **Recommended**       | ✅ Most cases                                   | Queue/Deque use cases       | Legacy                            | Legacy                                                                                   | Read-heavy concurrent apps              |

---
## Set Implementations

| Feature               | HashSet         | LinkedHashSet                   | TreeSet          | CopyOnWriteArraySet        | EnumSet          |
| --------------------- | --------------- | ------------------------------- | ---------------- | -------------------------- | ---------------- |
| **Implementation**    | `HashMap`       | `LinkedHashMap`                 | `TreeMap`        | `CopyOnWriteArrayList`     | Bit Vector       |
| **Backing Structure** | Hash Table      | Hash Table + Doubly Linked List | Red-Black Tree   | Dynamic Array              | Bit Mask         |
| **Ordering**          | ❌               | Insertion Order                 | Sorted           | Insertion Order            | Enum Order       |
| **Duplicates**        | ❌               | ❌                               | ❌                | ❌                          | ❌                |
| **Null Allowed**      | One             | One                             | ❌                | ❌                          | ❌                |
| **Thread Safe**       | ❌               | ❌                               | ❌                | ✅                          | ❌                |
| **Search**            | O(1)            | O(1)                            | O(log n)         | O(n)                       | O(1)             |
| **Recommended**       | General purpose | Preserve insertion order        | Need sorted data | Read-heavy concurrent apps | Enum collections |

---
## Map Implementations

| Feature                 | HashMap             | LinkedHashMap                   | TreeMap        | Hashtable   | ConcurrentHashMap       | WeakHashMap                  | IdentityHashMap                | EnumMap             |
| ----------------------- | ------------------- | ------------------------------- | -------------- | ----------- | ----------------------- | ---------------------------- | ------------------------------ | ------------------- |
| Internal Implementation | Hash Table          | Hash Table + Doubly Linked List | Red-Black Tree | Hash Table  | Concurrent Hash Table   | Hash Table + Weak References | Hash Table (`==`)              | Array               |
| Ordering                | None                | Insertion / Access              | Sorted         | None        | None                    | None                         | None                           | Enum Order          |
| Search                  | O(1)                | O(1)                            | O(log n)       | O(1)        | O(1)                    | O(1)                         | O(1)                           | O(1)                |
| Null Key                | 1                   | 1                               | ❌              | ❌           | ❌                       | Yes                          | Yes                            | ❌                   |
| Null Value              | Many                | Many                            | Yes            | ❌           | ❌                       | Yes                          | Yes                            | Yes                 |
| Thread Safe             | ❌                   | ❌                               | ❌              | ✅           | ✅                       | ❌                            | ❌                              | ❌                   |
| Best Use Case           | General-purpose map | Preserve order / LRU cache      | Sorted data    | Legacy code | Concurrent applications | Memory-sensitive caches      | Reference identity comparisons | Enum-keyed mappings |

|Implementation|Remember One Line|
|---|---|
|**HashMap**|Fast, unordered key-value map using a hash table.|
|**LinkedHashMap**|HashMap + Doubly Linked List → maintains insertion/access order.|
|**TreeMap**|Red-Black Tree → sorted keys with O(log n) operations.|
|**Hashtable**|Legacy synchronized HashMap; avoid in new code.|
|**ConcurrentHashMap**|Thread-safe HashMap using fine-grained concurrency; no nulls.|
|**WeakHashMap**|Keys are weakly referenced and can be garbage-collected automatically.|
|**IdentityHashMap**|Uses `==` instead of `equals()` for key comparison.|
|**EnumMap**|Array-backed map optimized for enum keys.|

