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

| Feature                           | `HashMap`                 | `Hashtable`                         | `ConcurrentHashMap`                                         |
| --------------------------------- | ------------------------- | ----------------------------------- | ----------------------------------------------------------- |
| **Thread-safe?**                  | ❌ No                      | ✅ Yes                               | ✅ Yes                                                       |
| **Synchronization**               | None                      | Synchronizes methods                | Fine-grained concurrency / CAS + synchronization internally |
| **Performance in concurrent use** | Unsafe                    | Lower due to coarse-grained locking | Better concurrency                                          |
| **Null key**                      | ✅ One                     | ❌ No                                | ❌ No                                                        |
| **Null values**                   | ✅ Multiple                | ❌ No                                | ❌ No                                                        |
| **Introduced**                    | Java 1.2                  | Java 1.0                            | Java 1.5                                                    |
| **Legacy?**                       | No                        | ✅ Yes                               | No                                                          |
| **Iterator behavior**             | Fail-fast, best-effort    | Fail-fast, best-effort              | Weakly consistent                                           |
| **Use case**                      | Normal/non-concurrent map | Legacy code                         | Concurrent applications                                     |
**`putIfAbsent` → give me a value if key doesn't exist**  
**`computeIfAbsent` → calculate a value if key doesn't exist**

---

#### `putIfAbsent()`

```
Map<String, Integer> map = new HashMap<>();

map.put("A", 100);

map.putIfAbsent("A", 200);
```
Result:
```
A → 100
```
Because `"A"` already exists, `200` is **not inserted**.

`ConcurrentHashMap.putIfAbsent()` provides the required atomicity for concurrent use.

---

#### `computeIfAbsent()`

Here, instead of giving the value directly, you give a **function that calculates the value**.

```
Map<String, Integer> map = new HashMap<>();

map.computeIfAbsent("A", key -> key.length());
```
The key `"A"` doesn't exist, so:
```
"A" → 1
```

The lambda:
```
key -> key.length()
```
is executed to calculate the value.
If `"A"` already exists: The calculation is **not performed**, because `"A"` already has a value.

---
`merge()` is a **Map method** used when you want to **combine a new value with an existing value for the same key**.

Think:
> **`merge` → "If key exists, combine old + new. If not, insert the new value."**

##### Basic example
```
Map<String, Integer> map = new HashMap<>();

map.put("A", 10);

map.merge("A", 5, (oldValue, newValue) -> oldValue + newValue);
```

Result:
```
A → 15
```

Because `"A"` already exists:
```
old value = 10
new value = 5

10 + 5 = 15
```

There is no `"A"`:

```
A → 5
```
