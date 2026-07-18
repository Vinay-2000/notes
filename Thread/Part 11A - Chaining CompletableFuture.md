### ⚡ TL;DR (Executive Summary)
* **Core Distinction:** The difference lies entirely in two things: **Does it consume the previous result?** and **Does it return a new value?**
* **`thenApply()` (Transform):** Takes the previous result, transforms it using a `Function<T, R>`, and returns a new `CompletableFuture<R>`. Use for **data pipelining/mapping** (e.g., converting an Entity to a DTO).
* **`thenAccept()` (Consume):** Takes the previous result, consumes it using a `Consumer<T>`, and returns `CompletableFuture<Void>`. Use for **terminal actions** like logging, saving, or printing.
* **`thenRun()` (Trigger):** Completely **ignores** the previous result, executes a `Runnable`, and returns `CompletableFuture<Void>`. Use for running an independent side-effect after a task completes.

| Method | Input (Consumes Result?) | Output (Returns Value?) | Functional Interface | Cheat Sheet / Memory Trick |
| :--- | :--- | :--- | :--- | :--- |
| **`thenApply()`** | ✅ Yes (`T`) | ✅ Yes (`R`) | `Function<T, R>` | **Apply** a transformation |
| **`thenAccept()`** | ✅ Yes (`T`) | ❌ No (`void`) | `Consumer<T>` | **Accept** and consume |
| **`thenRun()`** | ❌ No (`void`) | ❌ No (`void`) | `Runnable` | **Run** blindly next |

---

We already know:

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> "Vinay");
```

Suppose this finishes.

Now we want to do something with the result.

Java gives us three methods:

```
thenApply()
thenAccept()
thenRun()
```

The entire difference is based on **whether you need the previous result** and **whether you return something**.

---

# thenApply()

Think:

```
Input
↓
Transform
↓
Output
```

Example

```java
CompletableFuture<String> future =
        CompletableFuture
                .supplyAsync(() -> "vinay")
                .thenApply(String::toUpperCase);
System.out.println(future.join());
```

Output

```
VINAY
```

Notice

```
vinay
↓
toUpperCase()
↓
VINAY
```

It transforms one value into another.

---

## Signature

Conceptually

```java
T
↓
Function<T,R>
↓
R
```

Input
↓
Returns another value.

---

# Another Example

```java
CompletableFuture<Integer> future =
        CompletableFuture
                .supplyAsync(() -> 10)
                .thenApply(x -> x * 2);
System.out.println(future.join());
```

Output

```
20
```

```
10
↓
20
```

---

# thenAccept()

Think:

```
Input
↓
Consume
↓
No Return
```

Example

```java
CompletableFuture
        .supplyAsync(() -> "Vinay")
        .thenAccept(System.out::println);
```

Output

```
Vinay
```

Notice

The value is used.
But nothing is returned.

---

## Signature

```
T
↓
Consumer<T>
↓
void
```

---

# Example

```java
CompletableFuture
        .supplyAsync(() -> 50)
        .thenAccept(x ->
                System.out.println(x * 2));
```

Output

```
100
```

---

# thenRun()

Think

```
Ignore Previous Result
↓
Just Run Something
```

Example

```java
CompletableFuture
        .supplyAsync(() -> "Vinay")
        .thenRun(() ->
                System.out.println("Finished"));
```

Output

```
Finished
```

Notice

The previous result
`Vinay`
is ignored.

---

## Signature

```
No Input
↓
Runnable
↓
void
```

---

# Visual Difference

thenApply

```
Result
↓
Modify
↓
New Result
```

---

thenAccept

```
Result
↓
Use It
↓
Done
```

---

thenRun

```
Ignore Result
↓
Run Task
```

---

# Example Pipeline

```java
CompletableFuture<String> future =
        CompletableFuture
                .supplyAsync(() -> "Java")
                .thenApply(String::toUpperCase)
                .thenApply(s -> s + " 21")
                .thenApply(s -> "[" + s + "]");
System.out.println(future.join());
```

Execution

```
Java
↓
JAVA
↓
JAVA 21
↓
[JAVA 21]
```

Each thenApply receives the previous result.

---

# Real Spring Boot Example

Suppose

```
Database
↓
Employee
```

Need DTO

```java
CompletableFuture<EmployeeDTO> future =
        CompletableFuture
                .supplyAsync(employeeService::getEmployee)
                .thenApply(employeeMapper::toDTO);
```

Very common in enterprise applications.

---

# Common Interview Questions

## Difference between thenApply() and thenAccept()

thenApply()
Returns another value.

thenAccept()
Consumes the value and returns nothing.

---

## Difference between thenAccept() and thenRun()

thenAccept()
Receives the previous result.

thenRun()
Ignores the previous result.

---

# Memory Trick

```
thenApply()
↓
Apply Function
↓
Return New Value
-------------------
thenAccept()
↓
Accept Value
↓
Return Nothing
-------------------
thenRun()
↓
Ignore Value
↓
Just Run
```
