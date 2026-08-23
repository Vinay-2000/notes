
## 1. Interview Answer

If asked **"Explain the internal working of HashMap"**, answer:

> `HashMap` stores key-value pairs internally using an array of buckets. Each bucket can contain one or more nodes.
>
> When we call `put(key, value)`, HashMap first calculates the key's `hashCode()` and applies a hash-spreading operation. It then uses the hash and the table size to calculate the bucket index.
>
> If the bucket is empty, a new node is inserted.
>
> If the bucket already contains an entry, a collision has occurred. HashMap checks the hash and then uses `equals()` to determine whether the key already exists.
>
> If the key already exists, its value is replaced. Otherwise, the new entry is added to the bucket.
>
> In Java 8+, if a bucket becomes sufficiently large, its linked-list representation can be converted into a Red-Black Tree, improving lookup from O(n) to approximately O(log n).
>
> HashMap also uses a load factor, which is `0.75` by default. When the number of entries crosses the threshold (`capacity × load factor`), HashMap resizes the table, typically doubling its capacity, and redistributes the entries.

![[HashMapNode.png]]

![[HashmapStructure.png]]

---

## 2. Internal Structure

Conceptually, HashMap maintains an array called `table`.

```text
HashMap
   |
   ▼
table[]
   |
   ├── bucket 0
   ├── bucket 1
   ├── bucket 2
   ├── ...
   └── bucket N
```

Each entry is represented internally by a node containing approximately:

```text
Node
 ├── hash
 ├── key
 ├── value
 └── next
```

The `next` reference allows multiple entries to exist in the same bucket.

---

## 3. What Happens During `put()`?

Example:

```java
Map<String, Integer> map = new HashMap<>();

map.put("Vinay", 100);
```

The basic flow is:

```text
key
 ↓
hashCode()
 ↓
hash spreading
 ↓
bucket index
 ↓
check bucket
 ↓
insert / replace / handle collision
```

### Step 1: Calculate `hashCode()`

HashMap gets the hash code of the key:

```java
"Vinay".hashCode()
```

Modern Java HashMap also performs hash spreading. Conceptually:

```java
h ^ (h >>> 16)
```

The purpose is to spread the higher bits into lower bits and reduce collisions.

---

## 4. How Is the Bucket Index Calculated?

HashMap uses:

```java
index = (n - 1) & hash;
```

where:

```text
n = table.length
```

For example, if:

```text
table.length = 16
```

then:

```java
index = (16 - 1) & hash;
       = 15 & hash;
```

This produces an index from:

```text
0 to 15
```

### Why `&` instead of `%`?

HashMap keeps its capacity as a power of 2.

Therefore:

```java
hash % 16
```

can be efficiently represented as:

```java
hash & (16 - 1)
```

---

## 5. If the Bucket Is Empty

Suppose the calculated bucket is `5`:

```text
Bucket 5
   ↓
null
```

HashMap creates a new node:

```text
Bucket 5
   ↓
+------------------+
| hash             |
| key = "Vinay"    |
| value = 100      |
| next = null      |
+------------------+
```

---

# 6. Collision Handling

A collision occurs when different keys map to the same bucket.

Example:

```text
Key A ──┐
        ├──► Bucket 5
Key B ──┘
```

The keys can have the same bucket even though:

```java
A.equals(B) == false
```

Initially, HashMap handles collisions using a linked-list structure:

```text
Bucket 5
   ↓
Node A
   ↓
Node B
   ↓
Node C
```

Each node points to the next node.

---

## 7. How Does HashMap Know If the Key Already Exists?

HashMap uses both:

- `hashCode()`
- `equals()`

Think of it as:

```text
hashCode()
    ↓
Find the bucket
    ↓
equals()
    ↓
Find the exact key
```

Suppose:

```java
map.put("Vinay", 100);
map.put("Vinay", 200);
```

HashMap:

1. Calculates the hash.
2. Finds the bucket.
3. Finds the existing node.
4. Compares the hash.
5. Uses `equals()` to compare the keys.
6. Finds that the key already exists.
7. Replaces `100` with `200`.

Final result:

```text
Vinay → 200
```

It does not create a second entry.

---

# 8. Why Do We Need Both `hashCode()` and `equals()`?

`hashCode()` narrows down the search to a bucket.

`equals()` identifies the exact key inside that bucket.

Example:

```text
HashMap
   |
   | hashCode()
   ▼
Bucket 7
   |
   | equals()
   ▼
Exact key
```

### Important contract

If:

```java
a.equals(b) == true
```

then:

```java
a.hashCode() == b.hashCode()
```

must also be true.

However, the reverse is NOT required.

Two different objects can have the same hash code:

```text
A.hashCode() = 100
B.hashCode() = 100

A.equals(B) = false
```

This is a collision.

---

# 9. Java 8+ Treeification

If too many entries accumulate in the same bucket, searching a linked list becomes slower.

Linked list:

```text
Bucket
  ↓
Node
  ↓
Node
  ↓
Node
  ↓
Node
  ↓
...
```

Lookup can become:

```text
O(n)
```

Java 8 introduced tree bins.

A sufficiently large bucket can be converted into a **Red-Black Tree**:

```text
             Node
            /    \
         Node    Node
         /         \
      Node         Node
```

Lookup becomes approximately:

```text
O(log n)
```

### Important interview detail

Do not simply say:

> "After 8 nodes HashMap converts the list to a tree."

That's incomplete.

Treeification depends on additional conditions, including the table's capacity. In current OpenJDK implementations, the minimum table capacity for treeification is **64**. If the table is still smaller, HashMap prefers resizing.

---

# 10. What Happens During `get()`?

Example:

```java
map.get("Vinay");
```

HashMap follows essentially the same path:

```text
"Vinay"
   ↓
hashCode()
   ↓
hash spreading
   ↓
calculate bucket index
   ↓
go to bucket
   ↓
compare hash
   ↓
equals()
   ↓
return value
```

If the bucket contains a linked list:

```text
Bucket
   ↓
Node A
   ↓
Node B
   ↓
Node C
```

HashMap searches the nodes until the correct key is found.

If the bucket has been treeified, it searches the Red-Black Tree.

---

# 11. Resizing

HashMap uses:

```text
Default initial capacity = 16
Default load factor      = 0.75
```

The threshold is:

```text
threshold = capacity × load factor
```

For the default capacity:

```text
16 × 0.75 = 12
```

So the threshold is initially `12`.

When the number of entries crosses the threshold, HashMap resizes.

```text
Capacity = 16
Threshold = 12

       ↓

entries cross threshold

       ↓

resize

       ↓

Capacity = 32
```

The entries are redistributed into the new table.

---

# 12. Why Does HashMap Resize?

As the number of entries grows, collisions can increase.

Resizing increases the number of buckets:

```text
Before:

16 buckets
 ↓
more collisions


After:

32 buckets
 ↓
better distribution
```

This helps maintain efficient average lookup and insertion.

---

# 13. What Happens With `null`?

HashMap allows one `null` key:

```java
map.put(null, 100);
```

A `null` key cannot call:

```java
null.hashCode()
```

so HashMap handles it specially.

The `null` key is effectively associated with hash `0`, which places it in bucket `0` under the normal indexing calculation.

Therefore:

```java
map.put(null, 100);
map.get(null);
```

works.

HashMap allows:

```text
One null key
Multiple null values
```

Example:

```java
Map<String, Integer> map = new HashMap<>();

map.put(null, 100);
map.put(null, 200);
```

There is still only one `null` key:

```text
null → 200
```

---

# 14. Complete Internal Flow

```text
                  put(key, value)
                         │
                         ▼
                  key.hashCode()
                         │
                         ▼
                   Hash spreading
                         │
                         ▼
                Calculate bucket index
                         │
                         ▼
                  ┌─────────────┐
                  │   Bucket    │
                  └─────────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
            Empty                Existing
              │                     │
              ▼                     ▼
        Insert Node             Compare hash
                                      │
                                      ▼
                                  equals()?
                                 /        \
                               Yes         No
                                │           │
                                ▼           ▼
                         Replace value   Collision
                                            │
                                            ▼
                                       Linked List
                                            │
                                      Large enough?
                                            │
                                            ▼
                                      Red-Black Tree
```

---

# 15. Time Complexity

| Operation | Average | Worst Case |
|---|---:|---:|
| `put()` | O(1) | O(log n)* |
| `get()` | O(1) | O(log n)* |
| `remove()` | O(1) | O(log n)* |

`*` With tree bins. Poor hashing or pathological conditions can affect practical performance.

---

# 16. Most Important Interview Points

Remember these:

1. **HashMap internally uses an array of buckets.**
2. `hashCode()` is used to determine the bucket.
3. Hash spreading helps distribute hashes.
4. Bucket index is calculated using:
   ```java
   (n - 1) & hash
   ```
5. Collisions are handled using linked structures.
6. Java 8+ can convert a heavily populated bucket into a **Red-Black Tree**.
7. `equals()` identifies the exact key.
8. Equal objects must have equal hash codes.
9. Default load factor is **0.75**.
10. When the threshold is crossed, HashMap resizes, typically doubling capacity.
11. HashMap permits **one null key** and multiple null values.
12. Average `get()` / `put()` complexity is **O(1)**.

---

# 17. 30-Second Version

If the interviewer wants a short answer:

> HashMap internally uses an array of buckets. When we insert a key-value pair, HashMap calculates the key's `hashCode()`, performs hash spreading, and calculates the bucket index using the hash and table size. If the bucket is empty, it inserts a node. If there is already an entry, HashMap handles the collision and uses `equals()` to determine whether the key already exists. If it exists, the value is replaced; otherwise, another node is added to that bucket. In Java 8+, heavily populated buckets can be converted from a linked list to a Red-Black Tree. HashMap uses a default load factor of 0.75, and when the threshold is crossed it resizes the table, typically doubling its capacity. This gives average O(1) lookup and insertion.

---

# 18. Common Follow-Up Questions

Be ready for these immediately after explaining HashMap:

- Why does HashMap use both `hashCode()` and `equals()`?
- What happens if two keys have the same hash code?
- What happens if two keys are equal?
- What happens if we override `equals()` but not `hashCode()`?
- Why should HashMap capacity be a power of 2?
- Why is the default load factor `0.75`?
- What happens during resizing?
- What is treeification?
- Why was Red-Black Tree introduced in Java 8?
- What happens when we use `null` as a key?
- Why is HashMap not thread-safe?
- HashMap vs ConcurrentHashMap?
- HashMap vs Hashtable?
- Why is HashMap lookup O(1) on average?
