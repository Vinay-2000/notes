# Collections Framework Overview (Q134--137)

> **Interview Focus:** Understand *why* the Collections Framework
> exists, how it is organized, and when to use each interface.

------------------------------------------------------------------------

# 134. Why do we need Collections in Java?

## Interview Answer

Collections provide a standard way to store and manipulate groups of
objects dynamically. They solve many limitations of arrays and provide
optimized data structures and algorithms.

### Problems with Arrays

-   Fixed size
-   Can store only one type
-   No built-in searching or sorting
-   Insertion/deletion is expensive
-   Manual resizing required

``` java
int[] arr = new int[3];
// Need a new array if more elements are required.
```

### Advantages of Collections

-   Dynamic size
-   Generic support (type safety)
-   Rich API
-   Ready-made data structures
-   Built-in algorithms (sorting, searching, shuffling)
-   Better maintainability

------------------------------------------------------------------------

# Collection Framework Architecture

``` text
                          Iterable
                              │
                         Collection
      ┌──────────────────┼───────────────────┐
      │                  │                   │
     List               Set               Queue
      │                  │                   │
 ┌────┼─────┐       ┌────┼─────┐        ┌────┴─────┐
 │    │     │       │    │     │        │          │
ArrayList LinkedList Vector HashSet TreeSet PriorityQueue
                    │
             LinkedHashSet

Map (Separate Hierarchy)
│
├── HashMap
├── LinkedHashMap
├── TreeMap
├── Hashtable
└── ConcurrentHashMap
```

------------------------------------------------------------------------

## 135. What are the important interfaces in the Collection hierarchy?

### Iterable

Root interface that enables the enhanced for-loop.

``` java
for(String s : list){
    System.out.println(s);
}
```

Main method:

``` java
iterator()
```

------------------------------------------------------------------------

### Collection

Root interface for List, Set and Queue.

Provides common operations:

-   add()
-   remove()
-   contains()
-   size()
-   isEmpty()

------------------------------------------------------------------------

### List

Characteristics:

-   Ordered
-   Allows duplicates
-   Index based
-   Preserves insertion order

Common implementations:

-   ArrayList
-   LinkedList
-   Vector

Use when order matters.

------------------------------------------------------------------------

### Set

Characteristics:

-   No duplicates
-   No indexing

Implementations:

-   HashSet
-   LinkedHashSet
-   TreeSet

Use when uniqueness is required.

------------------------------------------------------------------------

### Queue

FIFO data structure.

Implementations:

-   PriorityQueue
-   ArrayDeque
-   LinkedList

Used for scheduling and producer-consumer problems.

------------------------------------------------------------------------

### Deque

Double-ended queue.

Supports insertion and deletion from both ends.

Used as: - Queue - Stack

------------------------------------------------------------------------

### Map (Separate Hierarchy)

Stores key-value pairs.

Important point:

**Map is NOT part of the Collection hierarchy** because it stores
mappings rather than individual elements.

Implementations:

-   HashMap
-   LinkedHashMap
-   TreeMap
-   Hashtable
-   ConcurrentHashMap

------------------------------------------------------------------------

## 136. Important methods in Collection interface

### Add Operations

``` java
add(E e)
addAll(Collection c)
```

### Remove Operations

``` java
remove(Object o)
removeAll(Collection c)
retainAll(Collection c)
clear()
```

### Search Operations

``` java
contains(Object o)
containsAll(Collection c)
```

### Utility Operations

``` java
size()
isEmpty()
iterator()
toArray()
```

Example

``` java
List<String> list = new ArrayList<>();

list.add("Java");
list.add("Spring");

System.out.println(list.contains("Java"));
System.out.println(list.size());
```

------------------------------------------------------------------------

## 137. Explain the List interface.

### Interview Answer

A List is an ordered collection that allows duplicate elements and
provides index-based access.

Example

``` java
List<String> names = new ArrayList<>();

names.add("John");
names.add("Alice");
names.add("John");

System.out.println(names.get(1));
```

Output

``` text
Alice
```

Characteristics

  Feature        Supported
  ------------- -----------
  Ordered           ✅
  Duplicates        ✅
  Null values       ✅
  Index based       ✅

Common implementations

  Implementation   Best Use Case
  ---------------- --------------------------
  ArrayList        Frequent reads
  LinkedList       Frequent insert/delete
  Vector           Legacy synchronized list

------------------------------------------------------------------------

# Frequently Asked Interview Questions

## Collection vs Collections

  Collection                   Collections
  ---------------------------- --------------------------
  Interface                    Utility class
  Represents data structures   Provides utility methods

Example

``` java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
```

------------------------------------------------------------------------

## Iterable vs Collection

  Iterable         Collection
  ---------------- -------------------------------------
  Only iteration   Full collection operations
  iterator()       add(), remove(), contains(), size()

------------------------------------------------------------------------

## Collection vs Arrays

  Arrays              Collections
  ------------------- -----------------------
  Fixed size          Dynamic
  Primitive support   Objects (or wrappers)
  Fewer utilities     Rich API

------------------------------------------------------------------------

## Iterator vs Enhanced for Loop

### Iterator

``` java
Iterator<String> itr = list.iterator();

while(itr.hasNext()){
    System.out.println(itr.next());
}
```

Can safely remove elements using `iterator.remove()`.

### Enhanced for Loop

Cleaner syntax but cannot structurally modify the collection during
iteration.

------------------------------------------------------------------------

# Time Complexity Overview

  Operation   List     HashSet    HashMap
  ----------- -------- ---------- ----------
  Search      O(n)     O(1) Avg   O(1) Avg
  Insert      O(1)\*   O(1) Avg   O(1) Avg
  Delete      O(n)     O(1) Avg   O(1) Avg

\* ArrayList append is amortized O(1).

------------------------------------------------------------------------

# Spring Boot Examples

``` java
List<Employee> employees;

Set<Role> roles;

Queue<Job> jobs;

Map<String, Object> response;
```

Typical usage:

-   **List** → REST API responses
-   **Set** → Roles/permissions
-   **Queue** → Background tasks
-   **Map** → Dynamic JSON responses

------------------------------------------------------------------------

# Interview Tips

### Q. Why isn't Map part of Collection?

Because Collection represents a group of individual elements, whereas
Map stores key-value mappings.

------------------------------------------------------------------------

### Q. Which interface is the root of the Collections Framework?

Technically:

-   `Iterable` is the root for iteration.
-   `Collection` is the root for List, Set, and Queue.

------------------------------------------------------------------------

### Q. When would you choose List over Set?

Choose List when: - Order matters - Duplicates are allowed - Index-based
access is required

Choose Set when uniqueness is required.

------------------------------------------------------------------------

# Quick Revision

-   Arrays are fixed size; Collections are dynamic.
-   Collection is an interface.
-   Collections is a utility class.
-   Map is a separate hierarchy.
-   List = Ordered + Duplicates.
-   Set = Unique elements.
-   Queue = FIFO.
-   Deque = Double-ended queue.
