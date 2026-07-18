# Part 11C - allOf() and anyOf()

### ⚡ TL;DR (Executive Summary)
* **`CompletableFuture.allOf()` (Join All):** Blocks or waits until **every single provided future completes**. It returns a type-erased `CompletableFuture<Void>`, meaning you must pull the specific values directly out of your original task variables. Great for building combined Response DTOs from independent calls.
* **`CompletableFuture.anyOf()` (Fastest Wins):** Resolves as soon as **the very first future finishes**, ignoring all slower tasks. It returns `CompletableFuture<Object>` to accommodate diverse return types. Great for fetching identical data from backup clusters or secondary APIs.

| Method | Return Type | Resolution Criteria | Main Target Pattern |
| :--- | :--- | :--- | :--- |
| **`allOf()`** | `CompletableFuture<Void>` | Waits for **All** to finish | Scatter-Gather (Aggregating DTOs) |
| **`anyOf()`** | `CompletableFuture<Object>` | Waits for **First** to finish | Speed Optimization / Redundant APIs |

---

# Why Do We Need allOf()?

Suppose we have three independent API calls.

```
Employee Service
Salary Service
Project Service
```

All start together.

```java
CompletableFuture<Employee> employee = ...;
CompletableFuture<Salary> salary = ...;
CompletableFuture<Project> project = ...;
```

Question:
How do we wait for ALL of them?
Future has no elegant solution.
CompletableFuture provides:
```
allOf()
```

---

# allOf()

Example

```java
CompletableFuture<String> employee =
        CompletableFuture.supplyAsync(() -> "Vinay");
CompletableFuture<Integer> age =
        CompletableFuture.supplyAsync(() -> 25);
CompletableFuture<Void> all =
        CompletableFuture.allOf(employee, age);
all.join();
System.out.println(employee.join());
System.out.println(age.join());
```

Execution

```
Employee
Age
↓
Wait For Both
↓
Continue
```

---

## Return Type

Notice

```java
CompletableFuture<Void>
```

Why not:
```
Employee
Age
```
Because Java doesn't know:
- How many futures
- What types
So it simply says:
```
I'll tell you when ALL are finished.
```
You retrieve the individual results from the original futures.

---

# Real Example

```
Employee API
Salary API
Project API
Leave API
```
All execute simultaneously.

```
allOf()
↓
Everything Finished
↓
Build Response DTO
```
Very common in microservices.

---

# anyOf()

Suppose we have:
```
Server A
Server B
Server C
```
All contain the same data.
We only need the fastest response.
Use:
```java
CompletableFuture.anyOf(...)
```

---

Example

```java
CompletableFuture<String> a =
        CompletableFuture.supplyAsync(() -> "A");
CompletableFuture<String> b =
        CompletableFuture.supplyAsync(() -> "B");
CompletableFuture<Object> first =
        CompletableFuture.anyOf(a, b);
System.out.println(first.join());
```

Output
```
A
```
or
```
B
```
Whoever finishes first.

---

# Visual

allOf()

```
Task A
Task B
Task C
↓
Wait
↓
Continue
```

---

anyOf()

```
Task A
Task B
Task C
↓
First One Wins
↓
Continue
```

---

# Return Type

Notice

```java
CompletableFuture<Object>
```

Why Object?
Because:
```
String
Employee
Integer
```
may all be mixed.
Java has no common type except:
```
Object
```

---

# Real World Example

Need weather.
Call:
```
Primary Weather API
Backup API
Third API
```
Use:
```
anyOf()
```
Return whichever responds first.

---

# Combining allOf()

Very common.

```java
CompletableFuture<Employee> employee = ...;
CompletableFuture<Salary> salary = ...;
CompletableFuture<Project> project = ...;
CompletableFuture<EmployeeDTO> dto =
        CompletableFuture
                .allOf(employee, salary, project)
                .thenApply(v ->
                        new EmployeeDTO(
                                employee.join(),
                                salary.join(),
                                project.join()
                        )
                );
```

Notice

```
All Finish
↓
Create DTO
```
This pattern is extremely common in enterprise applications.

---

# Common Interview Questions

## Difference between allOf() and anyOf()

allOf()
Waits for every future.

anyOf()
Completes when the first future completes.

---

## Why does allOf() return CompletableFuture<Void>?

Because it only represents the completion of all tasks.
Results must be obtained from the original futures.

---

## Why does anyOf() return CompletableFuture<Object>?

Because the futures may have different result types.

---

# Memory Trick

```
allOf()
↓
Wait Everyone
--------------------
anyOf()
↓
First One Wins
```
