
## Common Employee Data

The following `Employee` object is used in the examples:

```java
public Employee(int id, String name, String department, double salary) {
    this.id = id;
    this.salary = salary;
    this.department = department;
    this.name = name;
}
```

Sample data:

```java
List<Employee> employees = Arrays.asList(
    new Employee(1, "John", "IT", 75000),
    new Employee(2, "Alice", "HR", 65000),
    new Employee(3, "Bob", "IT", 90000),
    new Employee(4, "David", "Finance", 80000),
    new Employee(5, "Emma", "HR", 70000),
    new Employee(6, "Charlie", "IT", 85000),
    new Employee(7, "Sophia", "Finance", 95000),
    new Employee(8, "Mike", "IT", 75000),
    new Employee(9, "Olivia", "HR", 60000),
    new Employee(10, "James", "Finance", 85000)
);
```

---

# 1. Remove Duplicates from a List

### Question

Given:

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 2, 4, 1, 5, 3);
```

Remove duplicate elements using Java Streams.

### Answer

```java
List<Integer> uniqueNumbers = numbers.stream()
        .distinct()
        .toList();
```

Output:

```text
[1, 2, 3, 4, 5]
```

### Interview Point

`distinct()` removes duplicate elements from the stream.

For an ordered sequential stream, it preserves the encounter order.

---

# 2. Convert List to Map

### Question

Convert a `List<Employee>` into:

```java
Map<Integer, String>
```

where:

```text
key   = employee ID
value = employee name
```

### Answer

```java
Map<Integer, String> map = employees.stream()
        .collect(Collectors.toMap(
                Employee::getId,
                Employee::getName
        ));
```

### Duplicate Key Problem

If two employees have the same ID, `toMap()` throws:

```text
IllegalStateException: Duplicate key
```

Handle it using a merge function:

```java
Map<Integer, String> map = employees.stream()
        .collect(Collectors.toMap(
                Employee::getId,
                Employee::getName,
                (oldValue, newValue) -> newValue
        ));
```

Here, if the same key occurs, `newValue` wins.

### Interview Point

Remember the three important arguments of `toMap()`:

```java
Collectors.toMap(
    keyMapper,
    valueMapper,
    mergeFunction
)
```

---

# 3. Sort Employees by Salary

### Question

Sort employees by salary in descending order.

### Answer

```java
List<Employee> sortedEmployees = employees.stream()
        .sorted(
                Comparator.comparingDouble(Employee::getSalary)
                        .reversed()
        )
        .toList();
```

### Alternative

```java
List<Employee> sortedEmployees = employees.stream()
        .sorted((a, b) ->
                Double.compare(b.getSalary(), a.getSalary())
        )
        .toList();
```

### Interview Point

For finding the highest element, don't sort the entire list unnecessarily.

Sorting is `O(n log n)`, while finding maximum is `O(n)`.

So use:

```java
employees.stream()
        .max(Comparator.comparingDouble(Employee::getSalary));
```

when the requirement is only to find the maximum.

---

# 4. Sort by Multiple Fields

### Question

Sort employees by:

1. Department ascending
    
2. Salary descending
    
3. Name ascending
    

### Answer

```java
List<Employee> sortedEmployees = employees.stream()
        .sorted(
                Comparator.comparing(Employee::getDepartment)
                        .thenComparing(
                                Comparator.comparingDouble(Employee::getSalary)
                                        .reversed()
                        )
                        .thenComparing(Employee::getName)
        )
        .toList();
```

### Important Interview Point

Be careful where you use `reversed()`.

This:

```java
Comparator.comparing(Employee::getDepartment)
        .thenComparingDouble(Employee::getSalary)
        .reversed();
```

reverses the entire comparator built so far.

Therefore both department and salary become descending.

Instead, reverse only the salary comparator:

```java
.thenComparing(
    Comparator.comparingDouble(Employee::getSalary)
            .reversed()
)
```

Think:

```text
Department ASC
      ↓
Salary DESC
      ↓
Name ASC
```

---

# 5. Group Employees by Department

### Question

Group employees by department.

Expected type:

```java
Map<String, List<Employee>>
```

### Answer

```java
Map<String, List<Employee>> groupedEmployees =
        employees.stream()
                .collect(
                        Collectors.groupingBy(Employee::getDepartment)
                );
```

Result conceptually:

```text
IT       → [John, Bob, Charlie, Mike]
HR       → [Alice, Emma, Olivia]
Finance  → [David, Sophia, James]
```

### Interview Point

`groupingBy()` uses the supplied classifier as the key and collects matching elements into a list.

---

# 6. Find Employee with Highest Salary

### Question

Find the employee with the highest salary.

Return the whole `Employee` object.

### Answer

```java
Employee highest = employees.stream()
        .max(Comparator.comparingDouble(Employee::getSalary))
        .orElse(null);
```

### Why `max()` instead of `sorted()`?

Avoid:

```java
employees.stream()
        .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
        .findFirst();
```

when you only need the maximum.

`max()` is `O(n)` while sorting is `O(n log n)`.

### Interview Point

Use `max()` when you need the maximum element.

---

# 7. Find Second Highest Salary

### Question

Find the employee with the second-highest distinct salary.

If salaries are duplicated, treat them as the same salary.

### Answer

First find the second-highest distinct salary:

```java
double secondHighestSalary = employees.stream()
        .mapToDouble(Employee::getSalary)
        .distinct()
        .boxed()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .findFirst()
        .orElse(-1);
```

Then find the employee:

```java
Employee secondHighest = employees.stream()
        .filter(e -> e.getSalary() == secondHighestSalary)
        .findFirst()
        .orElse(null);
```

### Important Interview Point

This:

```java
.sorted(...)
.skip(1)
.findFirst()
```

does not automatically mean second-highest distinct salary.

If the highest salary occurs multiple times, you need:

```java
.distinct()
```

before `skip(1)`.

### Common Trap

For example:

```text
95000
95000
90000
```

Without `distinct()`:

```text
skip(1) → 95000
```

With `distinct()`:

```text
95000
90000
```

Therefore:

```text
skip(1) → 90000
```

---

# 8. Find Frequency of Elements

### Question

Given:

```java
List<Integer> numbers = Arrays.asList(
    1, 2, 3, 2, 4, 1, 2, 5, 3, 1
);
```

Find the frequency of each element.

Expected:

```text
1 → 3
2 → 3
3 → 2
4 → 1
5 → 1
```

### Answer

```java
Map<Integer, Long> frequency = numbers.stream()
        .collect(
                Collectors.groupingBy(
                        Function.identity(),
                        Collectors.counting()
                )
        );
```

### Understanding

```java
Function.identity()
```

means:

```text
element → element
```

So the element itself becomes the key.

```java
Collectors.counting()
```

counts how many times that key occurs.

`counting()` returns a `Long`.

### Alternative Using `toMap()`

```java
Map<Integer, Long> frequency = numbers.stream()
        .collect(Collectors.toMap(
                Function.identity(),
                n -> 1L,
                Long::sum
        ));
```

This uses `toMap()`'s merge function to add duplicate values.

---

# 9. Merge Two Maps

### Question

Given:

```java
Map<String, Integer> map1 = Map.of(
    "A", 10,
    "B", 20,
    "C", 30
);

Map<String, Integer> map2 = Map.of(
    "B", 5,
    "C", 15,
    "D", 40
);
```

Merge the maps such that:

- Keys present in only one map are retained.
    
- Keys present in both maps have their values added.
    

Expected:

```text
A → 10
B → 25
C → 45
D → 40
```

### Stream Answer

```java
Map<String, Integer> merged = Stream.concat(
        map1.entrySet().stream(),
        map2.entrySet().stream()
)
.collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        Integer::sum
));
```

### Understanding the Merge Function

```java
Integer::sum
```

is equivalent to:

```java
(oldValue, newValue) -> oldValue + newValue
```

For:

```text
B → 20
B → 5
```

the merge function produces:

```text
20 + 5 = 25
```

### Alternative Without `Stream.concat()`

```java
Map<String, Integer> merged = new HashMap<>(map1);

map2.forEach((key, value) ->
        merged.merge(key, value, Integer::sum)
);
```

### Interview Point

`Map.merge()` is extremely useful when combining values for duplicate keys.

---

# 10. HashMap vs ConcurrentHashMap — Code

### Question

Demonstrate why `HashMap` is not thread-safe and how `ConcurrentHashMap` can safely handle concurrent updates.

### Example

```java
static Map<Integer, Integer> map;

public static void main(String[] args) throws InterruptedException {

    map = new ConcurrentHashMap<>();
    // map = new HashMap<>();

    Thread thread1 = new Thread(() -> {
        for (int i = 0; i < 100; i++) {
            increment();
        }
    });

    Thread thread2 = new Thread(() -> {
        for (int i = 0; i < 100; i++) {
            increment();
        }
    });

    thread1.start();
    thread2.start();

    thread1.join();
    thread2.join();

    System.out.println(map.get(20));
}

public static void increment() {
    map.merge(20, 1, Integer::sum);
}
```

With `ConcurrentHashMap`, the expected result is:

```text
200
```

### Why `merge()`?

```java
map.merge(20, 1, Integer::sum);
```

means:

```text
If key 20 doesn't exist:
    insert 20 → 1

If key 20 exists:
    oldValue + 1
```

With `ConcurrentHashMap`, `merge()` provides the atomic update needed for this concurrent operation.

---

# HashMap vs ConcurrentHashMap

|Feature|HashMap|ConcurrentHashMap|
|---|---|---|
|Thread-safe|❌ No|✅ Yes|
|Concurrent access|Not safe for concurrent modification|Designed for concurrent access|
|Null key|✅ One null key|❌ Not allowed|
|Null values|✅ Allowed|❌ Not allowed|
|Suitable for|Single-threaded/general use|Multi-threaded applications|
|Atomic operations|Basic Map operations aren't thread-safe|Provides atomic methods like `putIfAbsent()`, `compute()`, `merge()`|

### Important Interview Point

Do not say:

> "ConcurrentHashMap makes every sequence of operations thread-safe."

It doesn't.

For example:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

is still a compound operation.

Instead, use:

```java
map.putIfAbsent(key, value);
```

when appropriate.

---

# `merge()` vs `compute()`

These are important `Map` methods that often come up in interviews.

## `merge()`

Syntax:

```java
map.merge(key, newValue, remappingFunction);
```

You supply the new value.

Example:

```java
Map<String, Integer> map = new HashMap<>();

map.put("A", 10);

map.merge("A", 5, Integer::sum);

System.out.println(map); // {A=15}
```

Conceptually:

```text
Existing value = 10
Supplied value = 5

10 + 5 = 15
```

If the key doesn't exist:

```java
map.merge("B", 5, Integer::sum);
```

Result:

```text
B → 5
```

The supplied value is inserted.

---

## `compute()`

Syntax:

```java
map.compute(key, remappingFunction);
```

You calculate the new value inside the lambda.

Example:

```java
Map<String, Integer> map = new HashMap<>();

map.put("A", 10);

map.compute("A", (key, oldValue) -> oldValue + 5);

System.out.println(map); // {A=15}
```

You don't supply `5` separately.

You calculate the new value:

```text
oldValue = 10
       ↓
10 + 5
       ↓
newValue = 15
```

`compute()` also works when the key is absent. In that case:

```text
oldValue = null
```

---

# `merge()` vs `compute()` — Quick Comparison

|Feature|`merge()`|`compute()`|
|---|---|---|
|Supply new value separately|✅ Yes|❌ No|
|Lambda receives|`oldValue`, `newValue`|`key`, `oldValue`|
|Main idea|Combine old + supplied value|Calculate new value|
|Missing key|Inserts supplied value|Lambda receives `null`|
|Common use|Counters, combining values|Recalculating/updating values|

### Easy Way to Remember

```text
merge()
    new value is supplied
          ↓
    combine old + new

compute()
    no new value supplied
          ↓
    calculate the new value
```

Examples:

```java
map.merge(20, 1, Integer::sum);
```

means:

> Add this new `1` to the existing value.

While:

```java
map.compute(20, (key, value) -> value + 1);
```

means:

> Calculate what the new value should be.

---

# Must-Remember Stream/Collection Patterns

```java
// Remove duplicates
stream.distinct()

// Convert List → Map
Collectors.toMap()

// Sort
Comparator.comparing()

// Sort primitive double
Comparator.comparingDouble()

// Reverse order
.reversed()

// Multiple sorting conditions
.thenComparing()

// Group elements
Collectors.groupingBy()

// Find maximum
stream.max()

// Count frequency
Collectors.groupingBy(
    Function.identity(),
    Collectors.counting()
)

// Merge duplicate keys
Collectors.toMap(
    keyMapper,
    valueMapper,
    Integer::sum
)

// Map-level merge
map.merge(key, value, Integer::sum)

// Calculate map value
map.compute(key, (key, value) -> ...)
```

## High-Value Interview Tip

Don't just memorize the code. For each solution, be able to explain **why that particular Stream/Map operation is being used**.

For example:

```java
groupingBy()
```

→ "I want to group multiple objects under the same key."

```java
toMap()
```

→ "I want one map entry per element and I need to define the key/value."

```java
merge()
```

→ "I have an existing value and a new value and I want to combine them."

```java
compute()
```

→ "I want to calculate the new value based on the current key/value."

```java
max()
```

→ "I only need the maximum, so there's no reason to sort the entire collection."