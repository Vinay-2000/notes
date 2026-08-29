## 138. Explain about ArrayList with an example?

An `ArrayList` is a resizable-array implementation of the `List` interface. Unlike standard arrays in Java which have a fixed length, an `ArrayList` grows dynamically as elements are added. It maintains insertion order, allows duplicate elements, and permits `null` values.

**Internal Working:**

- **Backing Array:** Under the hood, it uses a standard primitive Java array (`Object[]`).
- **Resizing Mechanism:** When the capacity of the internal array is exhausted, it automatically creates a new, larger array (typically **1.5 times** the original size, calculated via bitwise shift: `oldCapacity + (oldCapacity >> 1)`) and copies the old elements over using `System.arraycopy()`.
- **Time Complexity:**
    - Positional access (`get(index)`, `set(index)`) is $O(1)$.
    - Amortized addition (`add(element)`) at the end of the list takes $O(1)$ time.
    - Inserting or removing elements from arbitrary positions takes $O(n)$ time because all subsequent elements must be shifted.

```
ArrayList Layout

+-----+-----+-----+-----+-----+
|  A  |  B  |  C  |  D  |     |   <- Internal Object[] array
+-----+-----+-----+-----+-----+
   0     1     2     3     4      <- Indices (Capacity: 5, Size: 4)
```

### Example

Java

```
import java.util.ArrayList;
import java.util.List;

public class ArrayListExample {
    public static void main(String[] args) {
        // Instantiate using interface reference (Best Practice)
        List<String> frameworkList = new ArrayList<>();

        // Adding elements
        frameworkList.add("Spring Boot");
        frameworkList.add("Micronaut");
        frameworkList.add("Quarkus");

        // Fast O(1) random access
        String primaryTech = frameworkList.get(0);
        System.out.println("Primary Technology: " + primaryTech);
    }
}
```

> **Interview Tip:** Mention that `ArrayList` is **not synchronized**. If multiple threads access it concurrently and at least one modifies it structurally, it must be synchronized externally, or you should use `CopyOnWriteArrayList` / `Collections.synchronizedList()`.

## 139. Can an ArrayList have duplicate elements?

Yes, an `ArrayList` can absolutely store duplicate elements. The `List` interface contract guarantees the preservation of positional indexing and insertion order, which naturally accommodates identical elements.

`ArrayList` determines equality between elements using the object's `equals(Object o)` method. When you invoke methods like `remove(Object o)` or `indexOf(Object o)`, the list traverses its internal array sequentially and performs `o.equals(element)`.

### Example

Java

```
import java.util.ArrayList;
import java.util.List;

public class DuplicateExample {
    public static void main(String[] args) {
        List<String> roles = new ArrayList<>();
        roles.add("Developer");
        roles.add("Architect");
        roles.add("Developer"); // Allowed duplicate

        System.out.println("Total Roles: " + roles.size()); // Outputs 3
    }
}
```

> **Spring Boot Relevance:** When mapping `@OneToMany` relationships in JPA/Hibernate using a `List` collection type, Hibernate allows duplicate child entities if the list is structurally altered. However, best practices dictate using `Set` mappings if duplicates are business-illogical, to avoid N+1 query patterns and cartesian product issues during eager fetches.

## 140. How do you iterate around an ArrayList using iterator?

To iterate through an `ArrayList` using an `Iterator`, you call the `iterator()` method exposed by the `Collection` interface. The `Iterator` pattern provides a safe way to traverse elements sequentially via `hasNext()` and `next()`.

Critically, the `Iterator` provides a dedicated `remove()` method. This is the **only safe way** to remove elements from a collection during structural traversal. Modifying the `ArrayList` directly via its own `remove()` method while iterating with an explicit or implicit loop will instantly trigger a `ConcurrentModificationException`.

### Example

Java

```
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorExample {
    public static void main(String[] args) {
        List<Integer> scores = new ArrayList<>();
        scores.add(75);
        scores.add(40);
        scores.add(90);

        Iterator<Integer> iterator = scores.iterator();
        while (iterator.hasNext()) {
            Integer score = iterator.next();
            // Safe removal based on a condition
            if (score < 50) {
                iterator.remove();
            }
        }
        System.out.println("Passing scores: " + scores);
    }
}
```

> **Common Mistake:** Avoid using a Java `for-each` loop if you plan to delete elements conditionally during iteration. Under the hood, the `for-each` loop uses a hidden iterator. If you call `list.remove()`, the internal state tracking variable (`modCount`) changes out of sync with the iterator's expected count, causing a **fail-fast** crash.

## 141. How do you sort an ArrayList?

There are three standard, high-performance ways to sort an `ArrayList` in modern Java:

1. **`List.sort(Comparator<? super E> c)` (Preferred since Java 8):** Invoked directly on the list instance. Passing `null` implies sorting via the elements' natural ordering.
2. **`Collections.sort(List<T> list)`:** A legacy utility wrapper that routes calls directly to `list.sort(null)`.
3. **Stream API (`list.stream().sorted()`):** Returns a _new_, sorted stream pipeline without mutating the underlying original `ArrayList`.

Internally, Java uses an adaptive version of **Timsort** (a hybrid of Merge Sort and Insertion Sort) for object arrays, executing with $O(n \log n)$ time complexity.

### Example

Java

```
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class SortExample {
    public static void main(String[] args) {
        List<String> cities = new ArrayList<>();
        cities.add("New York");
        cities.add("Berlin");
        cities.add("Tokyo");

        // 1. In-place sorting using natural order (alphabetical)
        cities.sort(Comparator.naturalOrder());
        System.out.println("Natural: " + cities);

        // 2. In-place sorting using custom Lambda Comparator (by string length)
        cities.sort((c1, c2) -> Integer.compare(c1.length(), c2.length()));
        System.out.println("By Length: " + cities);
    }
}
```

## 142. How do you sort elements in an ArrayList using comparable interface?

The `Comparable` interface defines the **natural ordering** of an object. To sort an `ArrayList` of custom objects using this approach, the target class must implement `Comparable<T>` and override its single method: `compareTo(T o)`.

- **`compareTo` Contract:**
    - Returns a **negative integer** if `this` object is less than the argument object.
    - Returns **zero** if `this` object is equal to the argument object.
    - Returns a **positive integer** if `this` object is greater than the argument object.

Once implemented, sorting is triggered via `list.sort(null)` or `Collections.sort(list)`.

### Example

Java

```
import java.util.ArrayList;
import java.util.List;

class Employee implements Comparable<Employee> {
    private int id;
    private String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() { return id; }

    @Override
    public int compareTo(Employee other) {
        // Natural order sorted ascending by numeric ID
        return Integer.compare(this.id, other.id);
    }

    @Override
    public String toString() { return "EmpId: " + id; }
}

public class ComparableDemo {
    public static void main(String[] args) {
        List<Employee> staff = new ArrayList<>();
        staff.add(new Employee(103, "Alice"));
        staff.add(new Employee(101, "Bob"));

        staff.sort(null); // Triggers Comparable logic
        System.out.println(staff); // Output: [EmpId: 101, EmpId: 103]
    }
}
```

## 143. How do you sort elements in an ArrayList using comparator interface?

The `Comparator` interface is designed for **external, custom sorting strategies**. Unlike `Comparable`, which alters the target class code to provide one unified default sort order, a `Comparator` decouples the sorting logic from the domain object entirely. This allows you to define multiple, disparate sorting rules for the same class (e.g., sorting by name, then sorting by salary).

It requires implementing the `compare(T o1, T o2)` method, typically resolved using Lambda expressions or functional builder patterns (`Comparator.comparing()`) in modern enterprise Java applications.

### Example

Java

```
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

class Product {
    private String sku;
    private double price;

    public Product(String sku, double price) {
        this.sku = sku;
        this.price = price;
    }
    public String getSku() { return sku; }
    public double getPrice() { return price; }

    @Override
    public String toString() { return sku + ":" + price; }
}

public class ComparatorDemo {
    public static void main(String[] args) {
        List<Product> catalog = new ArrayList<>();
        catalog.add(new Product("ProdA", 99.99));
        catalog.add(new Product("ProdB", 49.50));

        // Sorting using a functional Comparator composition (Price Descending)
        catalog.sort(Comparator.comparingDouble(Product::getPrice).reversed());

        System.out.println(catalog);
    }
}
```

## 144. What is vector class? How is it different from an ArrayList?

`Vector` is a legacy, thread-safe dynamic array class introduced in Java 1.0 that was retrofitted to implement the `List` interface in Java 1.2. It achieves thread safety by applying the `synchronized` keyword globally on almost all of its structural methods (e.g., `add()`, `get()`, `remove()`).

Because lock acquisition carries a significant performance cost, `Vector` is generally considered obsolete for modern single-threaded execution contexts or high-performance concurrent architectures.

### Comparison Table

| **Feature**       | **ArrayList**                                               | **Vector**                            |
| ----------------- | ----------------------------------------------------------- | ------------------------------------- |
| **Thread Safety** | Non-synchronized (Not thread-safe)                          | Synchronized (Thread-safe)            |
| **Performance**   | High (No overhead from locking)                             | Lower due to synchronization overhead |
| **Growth Factor** | Grows by **50%** of current capacity                        | Grows by **100%** (doubles size)      |
| **Legacy Status** | Modern (Introduced in Collections Framework Framework v1.2) | Legacy (Introduced in v1.0)           |

> **Interview Tip:** If an interviewer asks how to get a thread-safe `ArrayList` without using legacy `Vector`, state that you would use `Collections.synchronizedList(new ArrayList<>())` for general safety, or `CopyOnWriteArrayList` if read operations vastly outnumber write operations.

## 145. What is linkedList? What interfaces does it implement? How is it different from an ArrayList?

`LinkedList` is a sequential access collection that stores elements inside distinct node containers. Each node retains explicit pointer references to both its predecessor and its successor, forming a structural **doubly-linked list**.

**Interfaces Implemented:**

- `List` (Provides index-based operations)
- `Deque` and `Queue` (Enables double-ended queue capabilities like `push()`, `pop()`, `peek()`, `poll()`)
- `Cloneable` and `Serializable`

```
LinkedList Architecture

        Node                 Node                 Node
   +----+---+----+      +----+---+----+      +----+---+----+
   |prev| A |next| <=>  |prev| B |next| <=>  |prev| C |next|
   +----+---+----+      +----+---+----+      +----+---+----+
```

### Comparison Table

| **Feature**                | **ArrayList**                         | **LinkedList**                                    |
| -------------------------- | ------------------------------------- | ------------------------------------------------- |
| **Underlying Structure**   | Dynamic Object Array (`Object[]`)     | Doubly Linked List Nodes                          |
| **Random Access (`get`)**  | $O(1)$ fast indexing                  | $O(n)$ slow sequential pointer traversal          |
| **Head Insertion/Removal** | $O(n)$ due to array element shifts    | $O(1)$ fast pointer adjustments                   |
| **Memory Footprint**       | Low (Stores data values contiguously) | High (Requires extra reference pointers per node) |

> **Best Practice:** In corporate microservices architectures using Spring Boot, `ArrayList` should almost always be favored over `LinkedList` by default. Modern CPU architectures benefit extensively from L1/L2 cache locality, making contiguous array iterations dramatically faster than jumping across disjointed heap locations via node pointers.

## 146. Can you briefly explain about the Set interface?

The `Set` interface extends `Collection` but models the mathematical set abstraction. Its primary distinguishing constraint is that **it cannot contain duplicate elements**. A `Set` can contain at most one `null` element (depending on the concrete implementation).

Unlike `List`, a `Set` does not guarantee any deterministic ordering of its elements out of the box (though specific sub-interfaces/implementations like `LinkedHashSet` or `TreeSet` do introduce order). The interface contract relies deeply on the proper overrides of `equals(Object)` and `hashCode()` methods of the stored objects to enforce its uniqueness constraint.

### Best Practices

- Always ensure objects stored inside a `Set` have an **immutable identity**. If an object's state changes while inside a `Set` such that its `hashCode()` output shifts, the collection will experience a memory leak or lookups will fail silently.
- Program against the interface reference:
    Java
    ```
    Set<String> uniqueIds = new HashSet<>();
    ```

## 147. What are the important interfaces related to the Set interface?

The `Set` ecosystem is enriched by two vital sub-interfaces that impose ordering contracts on the items held within them:

1. **`SortedSet` Interface:** Extends `Set` and guarantees that its elements are kept strictly in ascending order, sorted either by their natural ordering (`Comparable`) or by an explicitly injected `Comparator`.
2. **`NavigableSet` Interface:** Extends `SortedSet` and adds concrete navigation capabilities. It exposes high-utility structural search methods like `floor()`, `ceiling()`, `lower()`, and `higher()` to look up closest-match elements, along with methods to poll elements or invert the set's sort order.

```
Set Interface Hierarchy

        Collection
            ↑
           Set
            ↑
        SortedSet
            ↑
       NavigableSet
```

## 148. What is the difference between Set and sortedSet interfaces?

The fundamental differentiator lies in structural ordering guarantees and the richness of the API.

### Comparison Table

| **Feature**              | **Set**                                                                        | **SortedSet**                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| **Ordering**             | Does not guarantee any order by default. Elements appear randomized.           | Elements are strictly sorted in ascending order continuously.                                        |
| **Element Restrictions** | Can hold `null` values (implementation dependent).                             | Cannot hold `null` if natural ordering is used, as `null.compareTo()` throws `NullPointerException`. |
| **Specialized API**      | Inherits standard `Collection` API methods only (`add`, `remove`, `contains`). | Exposes range-view methods: `subSet()`, `headSet()`, `tailSet()`, `first()`, `last()`.               |

## 149. Can you give examples of classes that implement the Set interface?

The four core implementations of the `Set` interface commonly leveraged within high-scale enterprise applications are:

1. **`HashSet`:** Backed by a `HashMap`. Offers excellent performance with $O(1)$ time complexity for insertions, removals, and lookups. It provides no iteration order guarantees.
2. **`LinkedHashSet`:** A subclass of `HashSet` that uses a doubly-linked list running through its buckets. It preserves the **insertion order** of elements while maintaining near $O(1)$ performance.
3. **`TreeSet`:** A Red-Black tree structure implementing `NavigableSet`. Elements are stored in a strictly sorted order at the expense of logarithmic time complexity ($O(\log n)$) for standard operations.
4. **`ConcurrentSkipListSet`:** A thread-safe, concurrent alternative to `TreeSet` based on SkipLists, designed specifically for highly concurrent environments.

## 150. What is a HashSet?

`HashSet` is the most widely applied implementation of the `Set` interface. It relies entirely on a **`HashMap` instance internally** to preserve uniqueness. When you call `hashSet.add(element)`, the `HashSet` inserts that element as a **key** into the internal map, assigning a generic static dummy object (`new Object()`) as the map's associated value.

### Key Performance Specs

- **Time Complexity:** $O(1)$ constant time for basic CRUD operations (`add`, `remove`, `contains`), assuming a well-distributed hash function that avoids collisions.
- **Memory Overhead:** Higher than standard arrays because it must allocate backing structural map nodes (`Map.Entry`).
- **Ordering:** Iteration order is explicitly **not guaranteed** and may change over time as the collection resizes.

Java

```
// Conceptual view of HashSet internal storage mechanism:
private transient HashMap<E, Object> map;
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT) == null; // Key uniqueness enforces Set logic
}
```

## 151. What is a linkedHashSet? How is different from a HashSet?

`LinkedHashSet` is an extended variant of `HashSet` that maintains a running **doubly-linked list** across all of its entries. This structure records the precise sequence in which elements were added to the collection.

### Comparison Table

| **Feature**                 | **HashSet**                                               | **LinkedHashSet**                                                                      |
| --------------------------- | --------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Iteration Order**         | Completely undefined and non-deterministic.               | Guarantees iteration order matches **insertion order**.                                |
| **Internal Data Structure** | Backed strictly by a `HashMap`.                           | Backed by a combined `LinkedHashMap`.                                                  |
| **Performance Overhead**    | Marginally faster iterations and lower memory footprints. | Slightly slower operations due to the necessity of maintaining internal link pointers. |

### Example

Java

```
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;

public class SetComparison {
    public static void main(String[] args) {
        Set<String> hashSet = new HashSet<>();
        Set<String> linkedHashSet = new LinkedHashSet<>();

        for (String val : new String[]{"Z", "A", "M"}) {
            hashSet.add(val);
            linkedHashSet.add(val);
        }

        System.out.println("HashSet (Order Undefined): " + hashSet);      // Output could be random: e.g., [A, Z, M]
        System.out.println("LinkedHashSet (Insertion): " + linkedHashSet); // Guaranteed Output: [Z, A, M]
    }
}
```

## 152. What is a TreeSet? How is different from a HashSet?

`TreeSet` is a concrete implementation of the `NavigableSet` interface backed internally by a self-balancing **Red-Black Tree** structure (`TreeMap`). It ensures elements are organized into a strict sorted order dynamically as they are injected.

### Comparison Table

| **Feature**          | **HashSet**             | **TreeSet**                                        |
| -------------------- | ----------------------- | -------------------------------------------------- |
| **Internal Storage** | Hash Table (`HashMap`)  | Self-Balancing Binary Tree (`TreeMap`)             |
| **Time Complexity**  | $O(1)$ constant lookups | $O(\log n)$ logarithmic performance                |
| **Ordering**         | No ordering guarantees  | Strictly sorted (Natural or via `Comparator`)      |
| **Null Elements**    | Allows a `null` value   | Throws `NullPointerException` (cannot sort `null`) |

### Example

Java

```
import java.util.TreeSet;

public class TreeSetDemo {
    public static void main(String[] args) {
        TreeSet<Integer> scores = new TreeSet<>();
        scores.add(45);
        scores.add(99);
        scores.add(12);

        System.out.println("Sorted: " + scores); // Output: [12, 45, 99]
        System.out.println("Closest below 50: " + scores.floor(50)); // Output: 45
    }
}
```

## 153. Can you give examples of implementations of navigableSet?

In the core Java Standard Library, the premier implementation of `NavigableSet` is **`java.util.TreeSet`**.

For multi-threaded high-concurrency production landscapes, the equivalent implementation is **`java.util.concurrent.ConcurrentSkipListSet`**. It guarantees thread safety and lock-free concurrent writes while maintaining sorted navigation behaviors through a probabilistic concurrent data structure called a SkipList.

### Key Navigation API Methods Available:

- `lower(E e)`: Returns the greatest element strictly less than `e`.
- `floor(E e)`: Returns the greatest element less than or equal to `e`.
- `ceiling(E e)`: Returns the least element greater than or equal to `e`.
- `higher(E e)`: Returns the least element strictly greater than `e`.
- `pollFirst()` / `pollLast()`: Retrieves and removes the absolute boundaries.

## 154. Explain briefly about Queue interface?

The `Queue` interface is designed to hold elements prior to processing. Modeled typically on the standard computer science **FIFO (First-In, First-Out)** layout, it defines clear entry and exit behaviors for buffer collections.

However, queues are flexible and include architectures like Priority Queues (which order items based on custom priority rules rather than simple insertion age) and LIFO (Last-In, First-Out) structures.

### The Two Queue Behavior Matrix

The interface methods are categorized into two structural patterns based on error handling:

| **Operation Type** | **Throws Exception on Failure** | **Returns Special Value (false/null)** |
| ------------------ | ------------------------------- | -------------------------------------- |
| **Insert (Tail)**  | `add(e)`                        | `offer(e)`                             |
| **Remove (Head)**  | `remove()`                      | `poll()`                               |
| **Examine (Head)** | `element()`                     | `peek()`                               |

## 155. What are the important interfaces related to the Queue interface?

The `Queue` interface is extended by two major specialized interfaces that are highly relevant to enterprise application messaging and concurrency:

1. **`Deque` (Double Ended Queue):** Extends `Queue` and supports structural element insertion, inspection, and removal at **both boundaries** (head and tail). It allows the collection to act simultaneously as a FIFO Queue and a LIFO Stack.
2. **`BlockingQueue`:** Extends `Queue` and introduces operations that wait (block) for the queue to become non-empty when retrieving an element, or wait for space to become available when storing an element. This interface is the foundational building block for the Producer-Consumer architecture across concurrent thread execution environments.

## 156. Explain about the Deque interface?

`Deque` stands for **Double Ended Queue** (pronounced "deck"). It represents a linear collection that allows high-efficiency element insertion, deletion, and inspection at both the absolute front and absolute back edges.

### Core Architecture Roles

- **As a Queue (FIFO):** You call `addLast()` / `offerLast()` to add items to the back and `removeFirst()` / `pollFirst()` to extract them from the front.
- **As a Stack (LIFO):** It explicitly supersedes the legacy `java.util.Stack` class. You invoke `push(e)` (which translates internally to `addFirst(e)`) and `pop()` (which maps to `removeFirst()`).

The primary high-performance non-thread-safe implementations of `Deque` are **`ArrayDeque`** (which uses a highly optimized, resizable circular array) and **`LinkedList`**.

## 157. Explain the BlockingQueue interface?

`BlockingQueue` is a sub-interface of `Queue` situated inside the `java.util.concurrent` package. It is explicitly engineered to handle structural synchronization across thread boundaries for **Producer-Consumer patterns**.

### Blocking Capabilities

- **Bounded Capacity Support:** It can maintain strict max-size restrictions to prevent out-of-memory errors caused by runaway producers.
- **Thread Blocking:**
    - If a thread attempts to retrieve an item via `take()` and the queue is empty, the thread automatically yields execution and blocks until a producer injects an item.
    - If a thread attempts to insert an item via `put(e)` and the queue is completely full, the thread blocks until a consumer creates space.

```
Producer-Consumer Architecture via BlockingQueue

  [Producer Thread] ---> put() -> [ BLOCKING QUEUE ] -> take() ---> [Consumer Thread]
                                  (Thread safe Buffer)
```

## 158. What is a priorityQueue?

A `PriorityQueue` is an unbounded queue implementation where elements are processed based on their priority rather than their chronological arrival order.

### Internal Implementation

- **Data Structure:** It is backed by a balanced binary min-heap array.
- **Ordering:** Elements are continuously organized according to their **natural ordering** (`Comparable`) or an explicitly supplied **`Comparator`** instance at construction time.
- **Complexity:** The lowest element is always at the head of the queue, accessible in $O(1)$ time via `peek()`. Inserting elements (`offer()`) and extracting elements (`poll()`) require logarithmic time complexity ($O(\log n)$).
- **Null Constraint:** It does not allow `null` items, as it relies on object comparisons to maintain structure.

### Example

Java

```
import java.util.PriorityQueue;
import java.util.Queue;

public class PriorityQueueDemo {
    public static void main(String[] args) {
        // Naturally ordered priority queue (Ascending integers by default)
        Queue<Integer> urgentTasks = new PriorityQueue<>();
        urgentTasks.offer(5);
        urgentTasks.offer(1);
        urgentTasks.offer(3);

        while (!urgentTasks.isEmpty()) {
            System.out.print(urgentTasks.poll() + " "); // Output: 1 3 5
        }
    }
}
```

## 159. Can you give example implementations of the BlockingQueue interface?

The concurrent package provides several specialized implementations of `BlockingQueue`, selected based on specific architectural trade-offs:

1. **`ArrayBlockingQueue`:** A bounded blocking queue backed internally by a traditional array. The total capacity is locked at instantiation and cannot be altered. It uses a single lock for both read and write operations, making it deterministic but sometimes prone to contention.
2. **`LinkedBlockingQueue`:** An optionally bounded queue backed by linked nodes. It utilizes a two-lock queue architecture: one separate lock for insertions (`putLock`) and another for extractions (`takeLock`). This significantly enhances concurrent throughput by allowing producers and consumers to operate simultaneously.
3. **`SynchronousQueue`:** A zero-capacity structural rendezvous queue. Each insert operation must wait for a corresponding remove operation by another thread, and vice versa. It doesn't store data; it directly transfers it between threads.

## 160. Can you briefly explain about the Map interface?

The `Map` interface does not extend `Collection`. Instead, it defines an independent structure designed to store data as **Key-Value pairs**. A `Map` cannot contain duplicate keys; each discrete key can map to at most one corresponding value.

### Key Structural Facets

- **View Collections:** You interact with the internal map data through three distinct collection views:
    - `keySet()`: A `Set` tracking all keys.
    - `values()`: A `Collection` holding all mapped values.
    - `entrySet()`: A `Set` containing elements of type `Map.Entry<K, V>`, which provide direct structural access to both the key and value simultaneously.
- **Efficiency:** Maps are heavily relied upon in enterprise design patterns because they provide incredibly fast access to values when the unique identifier key is known.

## 161. What is difference between Map and sortedMap?

The primary differences center on the enforcement of sorting criteria and extended navigation capabilities.

### Comparison Table

| **Feature**      | **Map**                                                     | **SortedMap**                                                             |
| ---------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Key Ordering** | Order is completely undefined by default (e.g., `HashMap`). | Keys are strictly maintained in sorted order dynamically.                 |
| **Null Keys**    | Allows up to one `null` key (e.g., `HashMap`).              | Cannot accept `null` keys if it relies on natural sorting logic.          |
| **API Methods**  | Basic accessors: `put()`, `get()`, `containsKey()`.         | Range-view accessors: `subMap()`, `headMap()`, `tailMap()`, `firstKey()`. |

## 162. What is a HashMap?

`HashMap` is a high-efficiency implementation of the `Map` interface based on a hash table architecture.

### Internal Mechanics

- **Buckets:** It utilizes an array of nodes (commonly called buckets).
- **Hashing:** When `put(key, value)` is invoked, it runs the key through a hashing function to compute an index location within the array.
- **Collision Resolution:** If multiple distinct keys hash to the exact same array index, the collision is resolved using a linked list structure inside that bucket.
- **Treeification (Java 8+):** If a single bucket's linked list size exceeds a threshold of **8 elements** (`TREEIFY_THRESHOLD`) and the total map capacity is at least 64, Java converts that specific linked list into a self-balancing **Red-Black Tree**. This improves the worst-case lookup performance of that bucket from $O(n)$ down to a highly optimized $O(\log n)$.

```
HashMap Bucket Layout (Java 8+)

[Bucket Index]
   [0] -> Null
   [1] -> [Node: Key A] -> [Node: Key B] (Linked List due to collision)
   [2] -> [Red-Black Tree Root Node]     (Treeified due to high collisions)
```

## 163. What are the different methods in a Hash Map?

The `HashMap` API provides standard CRUD accessors along with several functional compute methods introduced in Java 8 to streamline state modifications:

### Core Classic Methods

- `V put(K key, V value)`: Associates the value with the key. If the key already exists, it overwrites the value and returns the old value.
- `V get(Object key)`: Fetches the value; returns `null` if not found.
- `boolean containsKey(Object key)` / `containsValue(Object value)`: Validates existence.
- `V remove(Object key)`: Purges the mapping.

### Functional Methods

- `V putIfAbsent(K key, V value)`: Inserts the mapping only if the key is missing or mapped to `null`.
- `V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)`: Dynamically computes a value using the provided function if the key isn't already present.
- `V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)`: Standardizes updates by combining an existing value with a new value using the provided merging logic.

## 164. What is a TreeMap? How is different from a HashMap?

`TreeMap` implements the `NavigableMap` interface using a self-balancing Red-Black tree structure. This ensures that all stored keys are kept in continuous sorted order.

### Comparison Table

| **Feature**         | **HashMap**                                      | **TreeMap**                                  |
| ------------------- | ------------------------------------------------ | -------------------------------------------- |
| **Data Structure**  | Hash Table (Array + Linked List/Tree Nodes)      | Red-Black Tree                               |
| **Time Complexity** | $O(1)$ constant time for reads/writes            | $O(\log n)$ logarithmic time                 |
| **Ordering**        | Completely unordered                             | Sorted by keys (Natural or Custom)           |
| **Null Support**    | Allows one `null` key and multiple `null` values | No `null` keys allowed; values can be `null` |

## 165. Can you give an example of implementation of navigableMap interface?

The primary implementation of the `NavigableMap` interface in the standard Java core package is **`java.util.TreeMap`**. For thread-safe, concurrent enterprise applications, the equivalent implementation is **`java.util.concurrent.ConcurrentSkipListMap`**.

### Example

Java

```
import java.util.NavigableMap;
import java.util.TreeMap;

public class NavigableMapDemo {
    public static void main(String[] args) {
        NavigableMap<String, String> routingTable = new TreeMap<>();
        routingTable.put("10.0.0.1", "Gateway A");
        routingTable.put("192.168.0.1", "Gateway B");
        routingTable.put("172.16.0.1", "Gateway C");

        // Returns the closest entry strictly greater than the given key
        System.out.println("Higher than 10.0.0.1: " + routingTable.higherEntry("10.0.0.1"));

        // Reverse the sorting order instantly
        System.out.println("Descending Keys: " + routingTable.descendingKeySet());
    }
}
```

## 166. What are the static methods present in the collections class?

The `java.util.Collections` class is an uninstantiable utility class filled exclusively with static methods that operate on or return collections.

### Core High-Utility Methods

- **Sorting & Shuffling:** `Collections.sort(List)`, `Collections.shuffle(List)`.
- **Search:** `Collections.binarySearch(List, Key)` (Requires the list to be pre-sorted).
- **Thread-Safety Wrappers:** `Collections.synchronizedList(List)`, `Collections.synchronizedMap(Map)`. These wrap an unsynchronized collection inside a thread-safe proxy object where every method call is protected by a synchronized block.
- **Immutability:** `Collections.unmodifiableList(List)` / `unmodifiableMap(Map)`. These return a read-only view of the collection that throws an `UnsupportedOperationException` on any mutation attempt.

> **Modern Java Alternative:** Since Java 9, prefer using factory methods like `List.of()`, `Set.of()`, and `Map.of()` to create truly immutable collections rather than wrapping mutable collections using `Collections.unmodifiable...`.

# Advanced Collections

## 167. What is the difference between synchronized and concurrent collections in Java?

The difference centers on how concurrency control is managed and the level of throughput they allow across multi-threaded applications.

### Comparison Table

| **Feature**           | **Synchronized Collections (Collections.synchronizedList)**                                        | **Concurrent Collections (ConcurrentHashMap)**                                                   |
| --------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Locking Strategy**  | **Coarse-grained locking.** Locks the entire collection object for every read and write operation. | **Fine-grained locking** or lock-free operations using CAS (Compare-And-Swap).                   |
| **Thread Contention** | High contention. If Thread A is reading, Thread B must block and wait to write or read.            | Low contention. Multiple threads can read and write to different segments concurrently.          |
| **Iterator Behavior** | **Fail-fast.** Throws `ConcurrentModificationException` if modified mid-loop.                      | **Fail-safe / Weakly Consistent.** Safely tolerates modifications during loops without crashing. |

## 168. Explain about the new concurrent collections in Java?

Introduced within the `java.util.concurrent` package, concurrent collections are engineered to deliver high thread safety combined with high throughput, explicitly avoiding the global bottleneck of coarse-grained synchronization.

### Key Implementations

1. **`ConcurrentHashMap`:** Uses lock-free read operations and fine-grained locking on a per-bucket-node level for writes. This allows multiple threads to write to different segments of the map simultaneously without blocking each other.
2. **`CopyOnWriteArrayList`:** A thread-safe list variant where all mutative operations (`add`, `set`) create a completely fresh copy of the underlying array. It is highly optimized for scenarios where read operations vastly outnumber modifications.
3. **`ConcurrentSkipListMap` / `ConcurrentSkipListSet`:** Concurrent, sorted alternatives to `TreeMap` and `TreeSet`. They use a SkipList structure under the hood to perform lock-free operations with $O(\log n)$ efficiency.

## 169. Explain about copyonwrite concurrent collections approach?

The **Copy-on-Write (COW)** architecture states that whenever a structural mutation occurs (such as an `add()` or `remove()`), the collection duplicates its entire internal array, applies the modification to this new copy, and then atomically updates its internal reference point to point to this new array.

```
CopyOnWrite Mutate State

[Thread-1: read]  ---> Reads from [ Array V1: "A", "B", "C" ]  (No blocking)

[Thread-2: write] ---> Copies V1 -> Altered [ Array V2: "A", "B", "C", "D" ]
                       ---> Atomically flips internal reference to V2
```

### Architectural Trade-offs

- **Lock-Free Reads:** Read operations require no synchronization or locking whatsoever. They read directly from the current immutable snapshot array, making them incredibly fast.
- **Write Penalty:** Write operations are computationally expensive because they require duplicating the array. This can trigger significant garbage collection pressure if the array is large or writes are frequent.
- **Use Case:** Ideal for configurations like a cache of system configurations or event listener lists, where definitions are loaded once at startup, read millions of times, and updated rarely.

## 170. What is compareandswap approach?

**Compare-And-Swap (CAS)** is an atomic, lock-free CPU instruction designed to achieve safe concurrency without the high cost of traditional OS-level thread blocking. It operates using three values:

1. A **Memory Location (`V`)**: The target variable to update.
2. An **Expected Value (`A`)**: What the thread believes the current value is.
3. A **New Value (`B`)**: The value to write if the check passes.

### How it Works

The CPU executes the check and write atomically: it checks if the value at memory location `V` matches the expected value `A`. If it matches, the CPU updates `V` with the new value `B` and returns `true`. If another thread modified `V` in the meantime, the check fails, the update is rejected, and the calling thread typically retries the operation in a tight loop (known as a spin-lock).

In Java, this capability is exposed via the `VarHandle` API and internal `Unsafe` classes, powering the `java.util.concurrent.atomic` package.

## 171. What is a lock? How is it different from using synchronized approach?

While the `synchronized` keyword acts as an implicit, built-in monitor lock tied to an object, the explicit `Lock` interface (e.g., `ReentrantLock`) was introduced in Java 5 to provide fine-grained, flexible concurrency control.

### Comparison Table

| **Feature**          | **synchronized Block**                                                            | **Explicit Lock (ReentrantLock)**                                               |
| -------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Acquisition**      | Scoped strictly to blocks/methods. Releases automatically when exiting the scope. | Explicitly controlled via `lock()` and `unlock()`. Can span across methods.     |
| **Non-blocking Try** | Not supported. Threads block indefinitely if the lock is held.                    | Supported via `tryLock()`. The thread can walk away if the lock is unavailable. |
| **Fairness Policy**  | Unfair. No guarantee which waiting thread gets the lock next.                     | Optional fairness mode. Can pass `true` to favor the longest-waiting thread.    |
| **Interruptibility** | A thread waiting to enter a synchronized block cannot be interrupted.             | Supported via `lockInterruptibly()`. Threads can break out if interrupted.      |

### Example

Java

```
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockDemo {
    private final Lock lock = new ReentrantLock();

    public void safeMethod() {
        lock.lock();
        try {
            // Critical business logic goes here
        } finally {
            // Best Practice: Always unlock inside a finally block to prevent deadlocks
            lock.unlock();
        }
    }
}
```

## 172. What is initial capacity of a Java collection?

The initial capacity is the **number of elements the collection's internal data structure can hold at the moment it is created**, before any automatic resizing occurs.

For instance:

- `ArrayList` defaults to an initial capacity of **10**.
- `HashMap` defaults to an initial capacity of **16**.

### Interview Tip / Best Practice

If you know ahead of time that you need to load 10,000 items into an `ArrayList` or `HashMap`, you should explicitly set the initial capacity at instantiation:

Java

```
List<String> userList = new ArrayList<>(10000);
```

By allocating the required memory upfront, you prevent the collection from performing multiple expensive array copies and re-hashing cycles as it grows, which significantly improves performance in high-throughput applications.

## 173. What is load factor?

The load factor is a metric used by hash-based collections (like `HashMap` and `HashSet`) to determine when to expand their internal capacity. It is defined as:

$$\text{Load Factor} = \frac{\text{Size of the Map (Number of Occupied Entries)}}{\text{Total Bucket Capacity}}$$

The default load factor in Java is **0.75** ($75\%$). This value provides an excellent balance between time complexity and memory overhead.

### Resizing Trigger

When the number of entries in the map exceeds the **threshold** ($\text{Capacity} \times \text{Load Factor}$), the map automatically doubles its capacity (resizes) and re-hashes every existing entry into the new array buckets.

- A higher load factor (e.g., 0.90) reduces memory usage but increases collision frequency, degrading lookups toward $O(n)$.
- A lower load factor (e.g., 0.50) minimizes collisions but increases memory consumption due to frequent expansions.

## 174. When does a Java collection throw UnsupportedOperationException?

A Java collection throws an `UnsupportedOperationException` when a modifying or mutative method (like `add()`, `remove()`, or `clear()`) is invoked on a collection that is explicitly configured as **read-only or immutable**.

This design pattern allows the Java Collections Framework to reuse standard interfaces like `List` and `Set` for both mutable and immutable structures. Instead of polluting the API with separate read-only interfaces, immutable implementations simply throw this runtime exception if a write method is called.

### Typical Scenarios

- Invoking modifications on views returned by factory utilities:
    Java
    ```
    List<String> fixedList = List.of("A", "B");
    fixedList.add("C"); // Throws UnsupportedOperationException
    ```
- Modifying an unmodifiable wrapper view:
    Java
    ```
    List<String> dynamicList = new ArrayList<>();
    List<String> readOnlyList = Collections.unmodifiableList(dynamicList);
    readOnlyList.clear(); // Throws UnsupportedOperationException
    ```

## 175. What is difference between fail-safe and fail-fast iterators?

The primary difference lies in how iterators handle structural modifications made to a collection while it is being traversed.

### Comparison Table

| **Feature**            | **Fail-Fast Iterators**                                                                                           | **Fail-Safe / Weakly Consistent Iterators**                                               |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Exception Behavior** | Throws `ConcurrentModificationException` immediately upon detecting an external change.                           | Does not throw any exception when the underlying collection is modified during traversal. |
| **Structural Basis**   | Iterates directly over the collection's live data structures using an internal modification counter (`modCount`). | Operates on a detached snapshot copy or walks a concurrent, weakly consistent structure.  |
| **Modifications**      | Reflects updates immediately but crashes if altered outside the iterator's own methods.                           | Allows concurrent additions or removals safely.                                           |
| **Typical Classes**    | All standard non-concurrent structures: `ArrayList`, `HashSet`, `HashMap`.                                        | Concurrent structures: `CopyOnWriteArrayList`, `ConcurrentHashMap`.                       |

## 176. What are atomic operations in Java?

An atomic operation is an operation that **executes entirely as a single, indivisible unit of work**. While it is running, no other thread can intercept it or see it in a partially completed state. It either succeeds completely or fails entirely, preventing data races without requiring explicit synchronized blocks.

In standard Java, reads and writes for object references and primitive variables (except `long` and `double`) are guaranteed to be atomic. However, compound statements like `count++` (which involves a read, a modification, and a write step) are **not atomic**.

### Atomic Enhancements

To perform compound operations safely across multiple threads, you should use the classes in the `java.util.concurrent.atomic` package (such as `AtomicInteger`, `AtomicLong`, and `AtomicReference`). These classes use underlying CPU atomic operations (like Compare-And-Swap) to ensure thread safety without the performance overhead of traditional locking.

Java

```
// Thread-Safe Atomic Increment Counter
private final AtomicInteger apiCounter = new AtomicInteger(0);

public void incrementRequestCount() {
    apiCounter.incrementAndGet(); // Atomically increments value and returns it
}
```

## 177. What is BlockingQueue in Java?

_(Note: This covers the essential technical details of the concept introduced in Q157)._

A `BlockingQueue` is an extended type of `Queue` specifically designed for concurrent architectures. It manages thread coordination by automatically blocking operations when the queue is full or empty.

It is widely used in enterprise frameworks like **Spring Boot's asynchronous task execution engines** and standard Java `ThreadPoolExecutor` systems. The queue serves as a resilient, thread-safe buffer that regulates task distribution between producer threads (which submit tasks) and worker threads (which process them).

### Summary of Core Operations

```
+-------------------------------------------------------------+
|               BlockingQueue Operation Matrix               |
+-------------------+--------------------+--------------------+
|  Method Strategy  | Queue is FULL      | Queue is EMPTY     |
+-------------------+--------------------+--------------------+
|  put(e) / take()  | BLOCKS Thread      | BLOCKS Thread      |
|  offer() / poll() | Returns false/null | Returns false/null |
+-------------------+--------------------+--------------------+
```

# Generics

## 178. What are Generics?

Introduced in Java 5, **Generics** add a layer of abstraction that allows types (classes and interfaces) to be parameterized when defining classes, interfaces, and methods. They enable you to design reusable components that can safely adapt to different data types.

The primary goal of Generics is to enforce **compile-time type safety**. By checking types at compile time, Generics allow the compiler to catch type mismatches early, preventing runtime `ClassCastException` failures.

Java

```
// Pre-Generics (Raw types) - Prone to errors
List openList = new ArrayList();
openList.add("Text");
Integer num = (Integer) openList.get(0); // Crashes at runtime with ClassCastException

// Post-Generics - Safe and robust
List<String> cleanList = new ArrayList<>();
cleanList.add("Text");
// cleanList.add(10); // Compiler blocks this immediately
String val = cleanList.get(0); // Safe assignment; no explicit casting needed
```

## 179. Why do we need Generics? Can you give an example of how Generics make a program more flexible?

We need Generics to eliminate two major issues common in older Java code: explicit, error-prone casting and the lack of compile-time type validation for collections. Generics make code highly flexible by decoupling processing logic from specific data models.

### Flexibility Example

Consider a generic response container commonly used in Spring Boot REST APIs to wrap different payload types while maintaining complete type safety:

Java

```
// Reusable, generic API wrapper
public class ApiResponse<T> {
    private String status;
    private T data; // Flexibly adapts to any object type

    public ApiResponse(String status, T data) {
        this.status = status;
        this.data = data;
    }
    public T getData() { return data; }
}

// Inside a controller service:
ApiResponse<UserDto> userResponse = new ApiResponse<>("Success", new UserDto("John"));
ApiResponse<OrderDto> orderResponse = new ApiResponse<>("Success", new OrderDto(99.0));
```

Without Generics, the `data` field would have to be defined as a generic `Object`, which would require developers to manually cast the payload back to its specific type every time it is used.

## 180. How do you declare a generic class?

To declare a generic class, you append a type parameter section enclosed in angle brackets (`<T>`) immediately after the class name. You can then use this type placeholder throughout the class body to define field types, method arguments, and return types.

By convention, standard single-letter placeholders are used to maximize readability:

- `T` for generic Type.
- `E` for Collection Element.
- `K` for Map Key.
- `V` for Map Value.

### Example

Java

```
public class RepositoryContainer<T> {
    private T entity;

    public void save(T entity) {
        this.entity = entity;
        System.out.println("Persisted entity: " + entity.getClass().getSimpleName());
    }

    public T getEntity() {
        return entity;
    }
}
```

## 181. What are the restrictions in using generic type that is declared in a class declaration?

Because Java implements Generics using **Type Erasure** (where the compiler removes all generic type information during compilation and replaces it with raw `Object` types or bound classes), several structural restrictions apply:

1. **No Static Context Access:** You cannot reference a class-level generic type parameter inside a `static` field or `static` method. Static members are shared across the class level, whereas generic types are bound to specific object instances.
2. **No Instantiation:** You cannot create an instance of a generic type directly (e.g., `new T()`). The type information is erased at runtime, so the JVM wouldn't know which constructor to execute.
3. **No Primitive Types:** You cannot use primitive types as generic arguments (e.g., `List<int>` is invalid). Generics require object types, so you must use wrapper classes like `List<Integer>` instead.
4. **No Generic Array Creation:** You cannot instantiate arrays of a generic type (e.g., `new T[10]` is invalid) because arrays enforce strict type checks at runtime, which conflicts with type erasure.

## 182. How can we restrict Generics to a subclass of particular class?

To restrict a generic type parameter to a specific subclass (or an implementation of a specific interface), you use **Bounded Wildcards** with the **`extends`** keyword. This configuration is known as an **Upper Bound**.

This ensures that the type parameter must either be the specified base class itself or one of its subclasses.

### Example

Java

```
abstract class Asset { public abstract double getValue(); }

class Stock extends Asset { public double getValue() { return 500.0; } }

// Restricting the generic manager to accept only Asset types and its subclasses
public class PortfolioManager<T extends Asset> {
    private final List<T> assets = new ArrayList<>();

    public double calculateTotalValue() {
        // Safe to call Asset methods because T is guaranteed to extend Asset
        return assets.stream().mapToDouble(Asset::getValue).sum();
    }
}
```

## 183. How can we restrict Generics to a super class of particular class?

To restrict a generic type parameter to be a specific class or any of its parent superclasses, you use the **`super`** keyword. This configuration is known as a **Lower Bound**.

This approach is typically used with wildcards (`<? super T>`) when designing methods that write data into collections, following the **PECS** guideline: **Producer Extends, Consumer Super**.

### Example

Java

```
import java.util.List;

class Manager extends Employee {}
class Executive extends Manager {}

public class ConsumerBoundDemo {
    // This method accepts lists of Manager, Employee, or Object
    public static void appendManagers(List<? super Manager> outputList) {
        outputList.add(new Manager());   // Safe to append a Manager
        outputList.add(new Executive()); // Safe because Executive is a subclass of Manager
        // outputList.add(new Employee()); // Compiler error! Cannot guarantee target list accepts general Employees
    }
}
```

## 184. Can you give an example of a generic method?

A generic method defines its own independent type parameters, which are scoped only to that specific method. The type parameter section `<T>` must be placed **before the method's return type**.

Generic methods can be defined within both standard classes and generic classes.

### Example

Java

```
public class UtilityFactory {

    // Generic method designed to transform an array into a List format safely
    public static <T> List<T> convertArrayToList(T[] inputArray) {
        List<T> resultList = new ArrayList<>();
        for (T element : inputArray) {
            resultList.add(element);
        }
        return resultList;
    }

    public static void main(String[] args) {
        String[] textArray = {"Spring", "Cloud", "Data"};
        // The compiler automatically infers the correct type argument based on parameters
        List<String> modernList = UtilityFactory.convertArrayToList(textArray);
        System.out.println(modernList);
    }
}
```

# Multi-threading

## 185. What is the need for threads in Java?

Threads are the fundamental units of concurrent execution within a Java application. They are essential for building high-performance applications for several key reasons:

1. **Concurrency & Multi-Core Utilization:** Threads allow an application to execute multiple tasks simultaneously across available CPU cores, maximizing hardware efficiency and increasing overall throughput.
2. **Asynchronous Execution:** Threads prevent long-running tasks (like database queries, external API integrations, or disk I/O operations) from blocking the main application thread. This keeps the application responsive to incoming user requests.
3. **Throughput in Web Architectures:** In web frameworks like Spring Boot, an underlying servlet container (such as Tomcat) automatically assigns an independent worker thread to manage each incoming HTTP request. This thread isolation allows the server to handle thousands of concurrent client requests efficiently.

## 186. How do you create a thread?

There are two classical approaches to creating a thread directly in Java:

1. Extending the structural `Thread` class.
2. Implementing the functional `Runnable` interface and passing it to a `Thread` constructor.

In modern enterprise applications, you should rarely create threads manually using these methods. Instead, you should use the **`ExecutorService` framework** to manage thread pools efficiently, or use **Virtual Threads** (introduced in Java 21) for lightweight, high-scale concurrency.

## 187. How do you create a thread by extending thread class?

To create a thread using this approach, you define a subclass that extends the `Thread` class and override its `run()` method. The `run()` method contains the code that will execute concurrently when the thread starts.

To launch the thread, you create an instance of your subclass and call its **`start()` method**. Calling `start()` signals the JVM to allocate internal system resources and spin up a new execution thread, which then runs your `run()` method.

### Example

Java

```
class TransactionWorker extends Thread {
    @Override
    public void run() {
        System.out.println("Processing async ledger update inside thread: " + Thread.currentThread().getName());
    }
}

public class ThreadExtendDemo {
    public static void main(String[] args) {
        TransactionWorker task = new TransactionWorker();
        task.start(); // Spins up a new thread asynchronously
    }
}
```

> **Common Mistake:** Never invoke `task.run()` directly instead of `task.start()`. Calling `run()` directly executes the code synchronously within the _current_ thread, completely bypassing concurrent execution.

## 188. How do you create a thread by implementing runnable interface?

This approach decouples the task logic from the execution mechanism. You implement the functional `Runnable` interface, define the task logic inside its single abstract `run()` method, and then pass that `Runnable` instance into a `Thread` constructor.

This method is highly preferred over extending the `Thread` class because Java supports only single class inheritance. By implementing an interface, your class remains free to extend another base class (such as a domain repository or controller).

### Example

Java

```
public class RunnableDemo {
    public static void main(String[] args) {
        // Modern Java definition using a Lambda expression
        Runnable alertTask = () -> System.out.println("Alert processed by: " + Thread.currentThread().getName());

        Thread executionEngine = new Thread(alertTask);
        executionEngine.start();
    }
}
```

## 189. How do you run a thread in Java?

To execute a thread, you must call its **`start()`** method.

Calling `start()` triggers an internal transition within the JVM: it moves the thread from the **New** state to the **Runnable** state and registers it with the operating system's thread scheduler. Once the scheduler allocates CPU time to the thread, the JVM invokes the thread's `run()` method automatically.

```
Thread Lifecycle Transition

  [ New Thread Instance ] --- explicit start() invocation ---> [ Runnable State ]
                                                                     |
                                                           (OS Scheduler Allocates CPU)
                                                                     v
                                                             [ Running run() ]
```

## 190. What are the different states of a thread?

A Java thread's lifecycle is governed by the JVM and can be in one of six distinct states, as defined by the `Thread.State` enum:

1. **NEW:** The thread instance has been created (e.g., via `new Thread()`) but its `start()` method has not yet been called.
2. **RUNNABLE:** The thread is actively executing its code or is ready and waiting for the operating system to allocate CPU cycles to it.
3. **BLOCKED:** The thread is paused, waiting to acquire a monitor lock to enter a synchronized block or method.
4. **WAITING:** The thread is waiting indefinitely for another thread to perform a specific action, triggered by calling methods like `Object.wait()`, `Thread.join()`, or `LockSupport.park()`.
5. **TIMED_WAITING:** The thread is paused for a specific period (e.g., via `Thread.sleep(ms)`, `Object.wait(timeout)`, or `Thread.join(timeout)`).
6. **TERMINATED:** The thread has completed its execution, either because its `run()` method finished normally or because an uncaught exception occurred.

## 191. What is priority of a thread? How do you change the priority of a thread?

A thread priority is an integer value that acts as a hint to the operating system's thread scheduler, indicating the relative importance of one thread compared to another.

Priorities range from a minimum value of **1 (`Thread.MIN_PRIORITY`)** to a maximum value of **10 (`Thread.MAX_PRIORITY`)**, with a default value of **5 (`Thread.NORM_PRIORITY`)**. You can adjust this value using the `setPriority(int)` method.

> **Interview Tip:** Emphasize that thread priorities are **non-deterministic** across different platforms. The JVM specification does not enforce how the operating system handles these values. A high-priority thread is not guaranteed to finish before a low-priority thread; it simply receives a probabilistic preference from the OS scheduler. Relying on thread priorities to coordinate application logic is a major anti-pattern.

## 192. What is executorservice?

Introduced in Java 5, `ExecutorService` is an advanced concurrency framework that replaces manual, ad-hoc thread creation with structured **Thread Pools**.

### Why it is Essential

Manually creating and destroying OS-level threads carries significant performance overhead. `ExecutorService` mitigates this by maintaining a pool of persistent, reusable worker threads. When a new task is submitted, the framework automatically assigns it to an available worker thread. This decoupled design simplifies thread lifecycle management, controls thread resource allocation, and protects the application from crashing due to sudden spikes in concurrent traffic.

```
ExecutorService Architecture

                +----------------+      +-------------------+
  Tasks ------> | Blocking Queue | ---> | Worker Thread Pool|
  (Runnable/    +----------------+      |  [T1] [T2] [T3]   |
   Callable)                            +-------------------+
```

## 193. Can you give an example for executorservice?

Here is a complete example demonstrating how to initialize an `ExecutorService`, submit asynchronous tasks, and perform a clean shutdown.

### Example

Java

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ExecutorDemo {
    public static void main(String[] args) {
        // Instantiate a fixed thread pool containing 3 reusable worker threads
        ExecutorService pool = Executors.newFixedThreadPool(3);

        try {
            for (int i = 0; i < 5; i++) {
                final int taskId = i;
                pool.submit(() -> {
                    System.out.println("Processing Task " + taskId + " via " + Thread.currentThread().getName());
                });
            }
        } finally {
            // Graceful shutdown sequence: stop accepting new tasks and wait for active tasks to finish
            pool.shutdown();
        }
    }
}
```

## 194. Explain different ways of creating executor services?

The `java.util.concurrent.Executors` utility class provides several pre-configured factory methods to initialize an `ExecutorService` based on specific application requirements:

1. **`newFixedThreadPool(int nThreads)`:** Creates a pool with a fixed number of threads and an unbounded input queue. It is ideal for predictable environments with stable baseline workloads.
2. **`newCachedThreadPool()`:** Creates a highly dynamic pool that scales out worker threads automatically as new tasks arrive, and cleans up idle threads after 60 seconds of inactivity. It is useful for handling short-lived, bursty asynchronous workloads.
3. **`newSingleThreadExecutor()`:** Creates a pool containing exactly one worker thread. It guarantees that all tasks are executed sequentially in the order they are received, preventing race conditions without needing explicit synchronization.
4. **`newScheduledThreadPool(int corePoolSize)`:** Returns an instance that can schedule tasks to run after a specified delay or execute them periodically at fixed intervals.

## 195. How do you check whether an executionservice task executed successfully?

When you submit a task to an `ExecutorService` using the `submit()` method, the framework returns a **`Future<?>`** proxy object. This object acts as a handle to track the status of the asynchronous task.

### Checking Success Criteria

- **`Future.isDone()`:** Returns `true` if the task has finished processing (whether it completed normally, failed with an exception, or was canceled).
- **Blocking Evaluation (`get()`):** Invoking `future.get()` blocks the calling thread until the asynchronous task completes.
    - If the task completes successfully, `get()` returns the computed result (or `null` if a `Runnable` was used).
    - If the task throws an unhandled exception during execution, `get()` wraps it and throws an **`ExecutionException`**. You can inspect the root cause by calling `exception.getCause()`.

## 196. What is callable? How do you execute a callable from executionservice?

`Callable` is a functional interface introduced in Java 5 to address the key limitations of `Runnable`. Unlike `Runnable`, a `Callable` task can **return a computed value** and is permitted to **throw checked exceptions**.

You execute a `Callable` by passing it to an `ExecutorService` via the `submit(Callable<T> task)` method. This returns a `Future<T>` instance that you can use to retrieve the computed value once execution completes.

### Example

Java

```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // Define an asynchronous task that returns a calculated integer value
        Callable<Integer> calculationTask = () -> {
            Thread.sleep(100); // Simulate processing
            return 42;
        };

        Future<Integer> evaluationHandle = executor.submit(calculationTask);

        // Retrieve the result. This block waits for the task to finish.
        Integer numericOutput = evaluationHandle.get();
        System.out.println("Returned Computation Value: " + numericOutput);

        executor.shutdown();
    }
}
```

## 197. What is synchronization of threads?

Synchronization is a mechanism that controls access to shared resources across multiple threads. It ensures that when multiple threads attempt to interact with a shared variable or block of code simultaneously, only **one thread can enter that critical section at any given time**.

### Why it is Needed

Without synchronization, concurrent operations can result in **Race Conditions** and data corruption, where threads overwrite each other's changes because they see stale, out-of-sync data.

Java implements synchronization using **Monitor Locks** (intrinsic locks). When a thread enters a synchronized zone, it automatically acquires the monitor lock for that object. Any other thread that attempts to enter the same zone is blocked until the holding thread finishes its work and releases the lock.

## 198. Can you give an example of a synchronized block?

A `synchronized` block provides fine-grained control over locking by wrapping **only the specific lines of code that modify shared state**, rather than locking the entire method. This minimizes the time a lock is held, which helps reduce thread contention.

### Example

Java

```
public class FinancialAccount {
    private double balance = 0.0;
    private final Object lockObject = new Object(); // Dedicated lock target object

    public void depositFunds(double amount) {
        // Non-critical operations can run concurrently here...

        synchronized(lockObject) {
            // Critical Section: Only one thread can modify the balance at a time
            this.balance += amount;
        }
    }
}
```

## 199. Can a static method be synchronized?

Yes, a `static` method can be synchronized. When you apply the `synchronized` keyword to a static method, the thread does not lock an individual object instance (`this`). Instead, it locks the unique **`java.lang.Class` object** associated with that class layout across the entire JVM.

As a result, if Thread A is executing a synchronized static method, all other threads attempting to execute _any_ synchronized static method within that same class will be blocked, even if they are targeting completely different object instances.

Java

```
public class GlobalCounter {
    private static int globalCount = 0;

    // Locks GlobalCounter.class object reference
    public static synchronized void incrementGlobal() {
        globalCount++;
    }
}
```

## 200. What is the use of join method in threads?

The `join()` method allows one thread to pause execution and **wait until another target thread finishes running**.

For example, if the main application thread executes `threadB.join()`, the main thread pauses its work and enters the _Waiting_ (or _Timed_Waiting_) state. It will not resume until `threadB` runs to completion and terminates. This is highly useful for coordinating tasks where one thread depends on the data or results computed by another.

### Example

Java

```
public class JoinDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread downloadWorker = new Thread(() -> {
            System.out.println("Downloading media asset data chunks...");
            try { Thread.sleep(500); } catch (Exception ignored) {}
        });

        downloadWorker.start();

        // Wait until the download worker finishes before continuing
        downloadWorker.join();

        System.out.println("Download complete. Proceeding with file rendering.");
    }
}
```

## 201. Describe a few other important methods in threads?

The `Thread` class provides several core static and instance methods to control thread execution and scheduling:

- **`Thread.sleep(long millis)` (Static):** Pauses the currently executing thread for a specified duration, transitioning it to the `TIMED_WAITING` state. It does **not** release any monitor locks it currently holds.
- **`Thread.yield()` (Static):** Provides a hint to the operating system's thread scheduler that the current thread is willing to yield its current processor allocation, giving other threads of equal priority a chance to run. The scheduler is free to ignore this hint.
- **`interrupt()` (Instance):** Sends an interruption signal to the target thread. If the target thread is blocked in a method like `sleep()` or `wait()`, it wakes up instantly and throws an `InterruptedException`.
- **`isAlive()` (Instance):** Returns a boolean indicating whether the thread has been started and has not yet terminated.

## 202. What is a deadlock?

A **Deadlock** is a concurrency failure that occurs when two or more threads are unable to make progress because **each is waiting for a lock held by the other**, creating a permanent standstill.

```
Deadlock Condition

  [Thread 1] --- holds ---> [Lock A] --- waiting for ---> [Lock B]
      ^                                                      |
      |                                                      v
  [Lock A] <--- waiting for --- [Lock B] <--- holds --- [Thread 2]
```

### The Four Necessary Conditions for a Deadlock

1. **Mutual Exclusion:** Only one thread can hold a resource at a time.
2. **Hold and Wait:** A thread holds a resource while waiting to acquire another.
3. **No Preemption:** Resources cannot be forcibly taken from a thread.
4. **Circular Wait:** Thread A waits for Thread B, which is waiting for Thread A.

### Best Practices for Prevention

- **Lock Ordering:** Ensure all threads acquire locks in the exact same predefined sequence.
- **Use `tryLock()`:** Use explicit locks with timeouts instead of implicit `synchronized` blocks. This allows a thread to back out safely if it cannot acquire a lock within a certain timeframe, breaking the deadlock condition.

## 203. What are the important methods in Java for inter-thread communication?

Inter-thread communication is achieved using three core methods defined directly within the base **`java.lang.Object`** class:

1. **`wait()`**
2. **`notify()`**
3. **`notifyAll()`**

### Crucial Architectural Rules

These methods can **only be invoked from within a synchronized context** (a synchronized block or method). The calling thread must own the monitor lock of the object it is calling these methods on. If a thread attempts to call them outside of a synchronized context, it will throw an immediate **`IllegalMonitorStateException`** at runtime.

## 204. What is the use of wait method?

The `wait()` method instructs the currently executing thread to **immediately release its monitor lock** and enter the object's wait queue, transitioning to the `WAITING` state.

The thread remains paused in this wait queue until another thread explicitly calls `notify()` or `notifyAll()` on the same monitor object. Once awakened, the thread must compete with other threads to re-acquire the monitor lock before it can resume execution from the point where it was paused.

## 205. What is the use of notify method?

The `notify()` method **wakes up a single thread** that is currently waiting in the monitor object's wait queue.

### Interview Tip / Critical Nuance

If multiple threads are waiting in the queue, `notify()` picks one thread to wake up arbitrarily; the selection is entirely non-deterministic. Additionally, calling `notify()` does **not** release the lock immediately. The notifying thread continues running and holds the lock until it exits its synchronized block. The awakened thread cannot resume execution until the notifying thread releases the lock and it successfully re-acquires it.

## 206. What is the use of notifyall method?

The `notifyAll()` method **wakes up all threads** currently waiting in the monitor object's wait queue.

### Why it is Preferred Over `notify()`

In almost all production environments, **`notifyAll()` is strongly preferred over `notify()`**. Because `notify()` wakes up only one thread arbitrarily, it can cause problems if that thread is unable to make progress based on the application's current state. The thread will simply go back to waiting, but since the notification signal was consumed, no other threads will wake up to process the work, leaving the application stalled.

`notifyAll()` avoids this risk by waking up all waiting threads, ensuring that any thread capable of making progress has the opportunity to do so.

## 207. Can you write a synchronized program with wait and notify methods?

Here is a classic implementation of a thread-safe, single-capacity Buffer that coordinates a Producer and a Consumer using `wait()` and `notifyAll()`.

### Example

Java

```
public class SharedBuffer {
    private String contentData;
    private boolean isEmpty = true;

    // Method for the Producer thread
    public synchronized void produce(String data) throws InterruptedException {
        // Best Practice: Always check the condition inside a while loop, never an if statement,
        // to protect against spurious wakeups.
        while (!isEmpty) {
            wait(); // Voluntarily release lock and wait for consumer
        }

        this.contentData = data;
        this.isEmpty = false;
        System.out.println("Produced Data: " + data);

        notifyAll(); // Wake up waiting consumer threads
    }

    // Method for the Consumer thread
    public synchronized String consume() throws InterruptedException {
        while (isEmpty) {
            wait(); // Voluntarily release lock and wait for producer
        }

        String consumedValue = this.contentData;
        this.isEmpty = true;
        System.out.println("Consumed Data: " + consumedValue);

        notifyAll(); // Wake up waiting producer threads
        return consumedValue;
    }
}
```

# Functional Programming

## 208. What is functional programming?

Functional programming is a declarative programming paradigm where applications are built by combining **pure, stateless functions** rather than executing sequences of imperative statements that mutate state.

### Key Concepts in Java (Java 8+)

- **First-Class Functions:** Functions can be treated as data—they can be assigned to variables, passed as arguments to other functions, and returned from methods.
- **Immutability:** Instead of modifying an existing collection or object in place, functional operations produce new, independent objects, reducing side effects and making code safer for concurrent execution.
- **Declarative Style:** You describe _what_ outcome you want to achieve rather than writing explicit instructions for _how_ to loop and update states manually.

## 209. Can you give an example of functional programming?

This example demonstrates the difference between the traditional imperative approach (using explicit loops and state mutations) and the modern functional approach (using declarative stream pipelines).

### Example

Java

```
import java.util.List;
import java.util.stream.Collectors;

public class FunctionalDemo {
    public static void main(String[] args) {
        List<Integer> values = List.of(1, 2, 3, 4, 5, 6);

        // Imperative Approach (Tracks state and loops manually)
        int imperativeSum = 0;
        for (int num : values) {
            if (num % 2 == 0) {
                imperativeSum += num * 2;
            }
        }

        // Functional / Declarative Approach (Stateless and clear)
        int functionalSum = values.stream()
                .filter(n -> n % 2 == 0)  // Filter even numbers
                .mapToInt(n -> n * 2)     // Double the values
                .sum();                   // Terminate and aggregate

        System.out.println("Functional calculation output: " + functionalSum);
    }
}
```

## 210. What is a stream?

A **Stream** in Java is a sequence of elements supporting sequential and parallel aggregate operations. It is **not a data structure**; it does not store elements or modify the underlying data source (like a `Collection` or an array). Instead, it acts as a computational pipeline that pulls data from a source, transforms it through a series of intermediate operations, and aggregates it using a terminal operation.

### Crucial Execution Mechanics

- **Lazy Evaluation:** Intermediate operations are never executed until a terminal operation is explicitly invoked.
- **Single-Use / Consumable:** A stream pipeline can only be traversed once. Once a terminal operation is executed, the stream is consumed and closed. Any attempt to reuse it will throw an `IllegalStateException`.

## 211. Explain about streams with an example? What are intermediate operations in streams?

A stream pipeline consists of a data source, zero or more intermediate operations, and a single terminal operation.

**Intermediate Operations** transform a stream into another stream. They are **lazy**; they record the transformation logic but do not process the elements immediately.

- Examples include `filter()`, `map()`, `sorted()`, and `distinct()`.

### Example

Java

```
import java.util.List;

public class StreamWorkflowDemo {
    public static void main(String[] args) {
        List<String> frameworks = List.of("Spring Boot", "Spring Cloud", "Hibernate", "Quarkus");

        frameworks.stream()
            .filter(name -> {
                // This print statement proves laziness; it won't execute without a terminal operation.
                System.out.println("Filtering pipeline element: " + name);
                return name.startsWith("Spring");
            })
            .map(String::toUpperCase) // Intermediate transformation
            .forEach(result -> System.out.println("Terminal Output: " + result)); // Terminal trigger
    }
}
```

## 212. What are terminal operations in streams?

A **Terminal Operation** is the final step in a stream pipeline. Invoking a terminal operation triggers the lazy intermediate operations, processes the elements through the pipeline, and produces a final result (such as a primitive value, a new collection, or a side effect).

Once a terminal operation executes, the stream pipeline is fully consumed and closed.

### Common Terminal Operations

- `collect(Collector)`: Collects the elements into a container like a `List`, `Set`, or `Map`.
- `forEach(Consumer)`: Iterates over each element (typically used to trigger side effects like logging).
- `reduce(BinaryOperator)`: Combines the elements into a single aggregate value.
- `count()`: Returns the total number of elements as a `long`.
- `findFirst()` / `findAny()`: Short-circuiting operations that return an `Optional` containing a matching element.

## 213. What are method references?

Method references provide a shorthand syntax to implement functional interfaces using existing methods. They make your code more readable by eliminating the boilerplate syntax of lambda expressions when a lambda simply passes its arguments directly to an existing method.

They use the double colon operator (`::`) and come in four distinct forms:

### Method Reference Typology

| **Category Type**                      | **Lambda Form**              | **Method Reference Translation** |
| -------------------------------------- | ---------------------------- | -------------------------------- |
| **Static Method**                      | `(x) -> Math.abs(x)`         | `Math::abs`                      |
| **Instance Method (Specific Object)**  | `(x) -> myPrinter.print(x)`  | `myPrinter::print`               |
| **Instance Method (Arbitrary Object)** | `(str) -> str.toLowerCase()` | `String::toLowerCase`            |
| **Constructor Reference**              | `() -> new ArrayList()`      | `ArrayList::new`                 |

## 214. What are lambda expressions?

A lambda expression is an anonymous function—a concise block of code that accepts parameters and returns a value, without being bound to a formal class structure.

Lambda expressions provide a clean way to implement **Functional Interfaces** inline, allowing you to treat blocks of behavior as data. This eliminates the boilerplate code required by traditional anonymous inner classes.

```
Lambda Structural Layout

   (argument list parameters) -> { operational function body execution }
```

## 215. Can you give an example of lambda expression?

This example shows how a lambda expression simplifies code compared to a traditional anonymous inner class when defining a custom sorting rule.

### Example

Java

```
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class LambdaDemo {
    public static void main(String[] args) {
        List<String> frameworkList = new ArrayList<>(List.of("Quarkus", "Spring", "Micronaut"));

        // Legacy approach using an Anonymous Inner Class
        frameworkList.sort(new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        });

        // Modern approach using a Lambda Expression
        frameworkList.sort((s1, s2) -> Integer.compare(s1.length(), s2.length()));

        System.out.println(frameworkList);
    }
}
```

## 216. Can you explain the relationship between lambda expression and functional interfaces?

A lambda expression can only be used where a **Functional Interface** is expected. A functional interface is an interface that contains **exactly one abstract method** (it can optionally include multiple `default` or `static` methods).

The compiler maps the lambda expression's parameters and return type directly to the single abstract method defined by the functional interface. You can add the optional `@FunctionalInterface` annotation to an interface to signal its intent; this instructs the compiler to generate an error if anyone accidentally adds a second abstract method to it.

Java

```
@FunctionalInterface
public interface TokenGenerator {
    String generate(User user); // Single Abstract Method (SAM)
}

// Assignment binding:
TokenGenerator jwtEngine = (u) -> "JWT_" + u.getName();
```

## 217. What is a predicate?

`Predicate<T>` is a pre-defined functional interface located in the `java.util.function` package. It represents a single-argument function that returns a **boolean value**.

Its single abstract method is `boolean test(T t)`. It also provides default methods like `and()`, `or()`, and `negate()`, which allow you to easily chain and combine multiple conditions together.

### Example

Java

```
import java.util.function.Predicate;

public class PredicateDemo {
    public static void main(String[] args) {
        Predicate<String> isLongerThanFive = str -> str.length() > 5;
        Predicate<String> startsWithS = str -> str.startsWith("S");

        // Combining individual predicates using conditional default methods
        Predicate<String> combinedFilter = isLongerThanFive.and(startsWithS);

        System.out.println(combinedFilter.test("Spring"));    // Returns true
        System.out.println(combinedFilter.test("Spark"));     // Returns false (length less than 5)
    }
}
```

## 218. What is the functional interface - function?

`Function<T, R>` is a built-in functional interface that represents an operation that **accepts an input argument of type `T` and transforms it into a result of type `R`**.

Its single abstract method is `R apply(T t)`. It is widely used in stream transformations, such as the `map()` operation, to convert elements from one data type to another.

### Example

Java

```
import java.util.function.Function;

public class FunctionDemo {
    public static void main(String[] args) {
        // Accepts an Integer and returns a String transformation representation
        Function<Integer, String> currencyConverter = amount -> "$" + amount + ".00";

        String parsedOutput = currencyConverter.apply(250);
        System.out.println(parsedOutput); // Output: $250.00
    }
}
```

## 219. What is a consumer?

`Consumer<T>` is a pre-defined functional interface that represents an operation that **accepts a single input argument of type `T` but returns no result** (`void`). It is designed to perform side effects, such as writing data to a database, printing messages to the console, or sending notifications.

Its single abstract method is `void accept(T t)`. It is commonly used as the argument for the `forEach()` terminal operation in the Stream API.

### Example

Java

```
import java.util.function.Consumer;

public class ConsumerDemo {
    public static void main(String[] args) {
        Consumer<String> logger = message -> System.out.println("LOG INFO: " + message);

        logger.accept("Async billing batch completed successfully.");
    }
}
```

## 220. Can you give examples of functional interfaces with multiple arguments?

When you need to pass more than one argument to a functional interface, you can use the built-in **Bi-variants** provided in the standard `java.util.function` package:

1. **`BiFunction<T, R U,>`:** Accepts two input arguments (type `T` and type `U`) and returns a result of type `R`. Its abstract method is `R apply(T t, U u)`.
2. **`BiPredicate<T, U>`:** Accepts two input arguments and returns a `boolean` result. Its abstract method is `boolean test(T t, U u)`.
3. **`BiConsumer<T, U>`:** Accepts two input arguments and returns no result (`void`). Its abstract method is `void accept(T t, U u)`. It is commonly used to iterate over maps via `map.forEach((key, value) -> ...)`.

### Example

Java

```
import java.util.function.BiFunction;
import java.util.function.BiPredicate;

public class BiInterfacesDemo {
    public static void main(String[] args) {
        // BiPredicate example: Check if a string matches a specified length criteria
        BiPredicate<String, Integer> validateLength = (text, target) -> text.length() == target;
        System.out.println("Validation: " + validateLength.test("Cloud", 5)); // Returns true

        // BiFunction example: Concatenate a prefix string and a numeric code
        BiFunction<String, Integer, String> systemIdBuilder = (prefix, code) -> prefix + "-" + code;
        String generatedId = systemIdBuilder.apply("ERR", 503);
        System.out.println("Generated Identifier: " + generatedId); // Output: ERR-503
    }
}
```
