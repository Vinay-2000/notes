Java 11
 ├── String improvements
 ├── HTTP Client
 └── Files.readString/writeString

Java 17 ⭐⭐⭐
 ├── Records
 ├── Sealed Classes
 ├── Pattern Matching instanceof
 └── Switch Expressions

Java 21 ⭐⭐⭐⭐⭐
 ├── Virtual Threads
 ├── Pattern Matching switch
 ├── Record Patterns
 └── Sequenced Collections

Java 25
 ├── Scoped Values
 ├── Flexible Constructor Bodies
 └── Compact Source Files

Java 26
 └── Know the major changes/concepts


| Java   | Feature                                      | Description                                                                                                                                                                   | Code                                                                                                                                                                |
| ------ | -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **11** | **HTTP Client API**                          | Modern HTTP client supporting HTTP/1.1 and HTTP/2.                                                                                                                            | HttpClient client = HttpClient.newHttpClient();<br>HttpRequest req = HttpRequest.newBuilder(URI.create(url)).build();<br>client.send(req, BodyHandlers.ofString()); |
|        | **String methods**                           | Adds useful methods like `isBlank()`, `strip()`, `repeat()`, and `lines()`.                                                                                                   | " hello ".strip();<br>" ".isBlank();<br>"Hi ".repeat(3);                                                                                                            |
|        | **Files.readString/writeString**             | Simplifies reading and writing text files.                                                                                                                                    | String s = Files.readString(path);<br>Files.writeString(path, "Hello");                                                                                             |
|        | **`var` in lambda parameters**               | Allows `var` to be used in lambda parameters.                                                                                                                                 | list.stream().map((var x) -> x * 2);                                                                                                                                |
| **17** | **Records** ⭐                                | Concise syntax for immutable data-carrier classes.                                                                                                                            | record User(String name, int age) {}<br>User u = new User("John", 25);<br>u.name();                                                                                 |
|        | **Sealed Classes** ⭐                         | Restricts which classes can extend a class/interface.                                                                                                                         | sealed interface Shape permits Circle, Square {}<br>final class Circle implements Shape {}<br>final class Square implements Shape {}                                |
|        | **Pattern Matching for `instanceof`** ⭐      | Performs type checking and casting in one step.                                                                                                                               | if (obj instanceof String s) {  System.out.println(s.length());}                                                                                                    |
|        | **Switch Expressions** ⭐                     | `switch` can return a value and uses cleaner `->` syntax.                                                                                                                     | int days = switch(month) {  <br>case 1, 3, 5 -> 31; <br>case 2 -> 28;<br>default -> 30;<br>};                                                                       |
| **21** | **Virtual Threads** ⭐⭐⭐                      | Extremely lightweight threads designed for high-concurrency, especially I/O-bound applications.                                                                               | Thread.startVirtualThread(() -> {  doWork();});                                                                                                                     |
|        | **Pattern Matching for `switch`** ⭐          | Allows type patterns directly inside `switch`.                                                                                                                                | String result = switch(obj) {  <br>case String s -> s.toUpperCase();<br>case Integer i -> "Number"; <br>default -> "Other";<br>};                                   |
|        | **Record Patterns**                          | Destructures a record directly while pattern matching.                                                                                                                        | record Point(int x, int y) {}<br>if (obj instanceof Point(int x, int y)) <br>{  System.out.println(x + y);}                                                         |
|        | **Sequenced Collections**                    | Common APIs for collections with a defined encounter order.                                                                                                                   | list.getFirst();<br>list.getLast();<br>list.reversed();                                                                                                             |
| **25** | **Flexible Constructor Bodies**⭐             | Allows statements before an explicit `super()`/`this()` call, subject to restrictions.                                                                                        | class Child extends Parent { <br>   public Child(String name) {    <br>           var normalized = name.trim();<br>           super(normalized); <br>		   }<br>	}   |
|        | **Module Import Declarations**               | Import all accessible packages from a module.                                                                                                                                 | import module java.base;                                                                                                                                            |
|        | **Compact Source Files & Instance `main()`** | Reduces boilerplate for simple Java programs.Java essentially infers the surrounding class and the main method doesn't need to be public static void main(String[] args).<br> | void main() {  <br>println("Hello");<br>}<br>                                                                                                                       |
|        | **Scoped Values** ⭐                          | Efficiently shares immutable contextual data, particularly useful with virtual threads.                                                                                       | static final ScopedValue USER = <br>ScopedValue.newInstance();<br>ScopedValue.runWhere(USER, "John", () -> {  System.out.println(USER.get());<br>});                |
| **26** | **HTTP/3 support**                           | Adds HTTP/3 support to the Java HTTP Client.                                                                                                                                  | HttpClient client = HttpClient.newBuilder()  .version(HttpClient.Version.HTTP_3)  .build();                                                                         |
|        | **Primitive Types in Patterns** _(Preview)_  | Extends pattern matching to work with primitive types.                                                                                                                        | switch (x) {  case int i -> ...;  case long l -> ...;}                                                                                                              |
|        | **Ahead-of-Time Object Caching**             | Improves startup/warm-up by caching objects ahead of time.                                                                                                                    | **JVM feature — no normal application code**                                                                                                                        |
|        | **Lazy Constants** _(Preview)_               | Allows constants to be initialized lazily.                                                                                                                                    | **Preview API — not something I'd prioritize for interviews yet.**                                                                                                  |




## 1. Java 11 New Features (LTS)

Java 11 cleaned up the language, introduced the modern HTTP client, and brought major developer ergonomics.

### Local-Variable Syntax for Lambda Parameters

Allows the use of `var` inside lambda expressions, enabling annotations on lambda parameters.

Java

```
// Java 11 allows annotating lambda parameters by utilizing 'var'
List<String> list = List.of("apple", "banana", "cherry");
String result = list.stream()
    .map((@Nonnull var s) -> s.toUpperCase())
    .collect(Collectors.joining(", "));
```

### New HttpClient API

Replaced the ancient, blocking `HttpURLConnection` with a modern, asynchronous, non-blocking HTTP client supporting HTTP/2 and WebSockets.

Java

```
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.github.com"))
    .GET()
    .build();

// Async response using CompletableFuture
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

### String & Files API Enhancements

Added practical utility methods like `isBlank()`, `lines()`, `repeat(n)`, and easy file reading/writing.

Java

```
String text = "  ";
boolean isBlank = text.isBlank(); // true (checks for whitespace)

// Quick file operations without complex boilerplate streams
Path path = Files.writeString(Files.createTempFile("test", ".txt"), "Hello Java 11!");
String fileContent = Files.readString(path);
```

## 2. Java 17 New Features (LTS)

Java 17 focused on modernizing Java's data modeling capabilities and pattern matching infrastructure.

### Records

Immutable data carriers that eliminate boilerplate for POJOs (Getters, `toString`, `equals`, `hashCode` are generated automatically).

Java

```
// One line defines an immutable data structure
public record User(Long id, String name, String email) {}

// Usage
User user = new User(1L, "Alice", "alice@gmail.com");
System.out.println(user.name()); // Automatically generated accessor (no 'get' prefix)
```

### Sealed Classes

Gives explicit control over inheritance. You can declare exactly which subclasses are allowed to extend or implement a class/interface.

Java

```
public sealed interface Shape permits Circle, Square {}

public final class Circle implements Shape { double radius; }
public final class Square implements Shape { double side; }
// Any other class attempting to implement Shape will cause a compilation error
```

### Pattern Matching for switch (Preview in 17, Final in 21)

Allows testing expressions against types and extracting data directly within switch cases.

Java

```
public static String formatShape(Object shape) {
    return switch (shape) {
        case Circle c -> "Circle with radius " + c.radius;
        case Square s -> "Square with side " + s.side;
        case null     -> "Null object";
        default       -> "Unknown shape";
    };
}
```

## 3. Java 21 New Features (LTS)

Java 21 is a monumental release, heavily expanding concurrency scales and data destructuring.

### Virtual Threads (Project Loom)

Lightweight, JVM-managed threads that drastically cut down the resource cost of concurrent applications. Instead of 1:1 mapping with OS threads, millions of virtual threads can run on a handful of platform threads.

Java

```
// Spawning 100,000 tasks without breaking your system memory
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 100_000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        });
    });
} // Automatic resource cleanup at the end of the block
```

### Record Patterns (Destructuring)

Allows you to break apart a record into its individual components directly within an `instanceof` or `switch` check.

Java

```
record Point(int x, int y) {}

Object obj = new Point(10, 20);

if (obj instanceof Point(int x, int y)) {
    System.out.println("Coordinates are: " + x + ", " + y); // No explicit casting needed!
}
```

### Sequenced Collections

Introduced a unified interface hierarchy (`SequencedCollection`, `SequencedSet`, `SequencedMap`) to handle first/last element retrieval deterministically across Collections.

Java

```
LinkedHashSet<String> set = new LinkedHashSet<>(List.of("A", "B", "C"));
String first = set.getFirst(); // "A"
String last = set.getLast();   // "C"
List<String> reversed = set.reversed(); // Built-in reverse viewing
```

## 4. Java 25 New Features (LTS)

Released in September 2025, Java 25 is the newest LTS milestone. It bridges the gap between scripting ease and heavy framework infrastructure.

### Flexible Constructor Bodies

Relaxes the rigid rule that `super(...)` or `this(...)` _must_ be the absolute first line of a constructor. You can now validate inputs or calculate variables _before_ invoking the superclass constructor.

Java

```
public class CustomOrder extends BaseOrder {
    private final long timestamp;

    public CustomOrder(Map<String, String> context) {
        // Prepare or validate data BEFORE calling super()
        if (context == null || !context.containsKey("id")) {
            throw new IllegalArgumentException("Invalid context");
        }
        long id = Long.parseLong(context.get("id"));

        super(id); // super call is no longer strictly forced onto line 1!
        this.timestamp = System.currentTimeMillis();
    }
}
```

### Scoped Values (Production Ready)

An elegant replacement for `ThreadLocal`, specifically built to minimize memory overhead when paired with thousands of Virtual Threads. Data is passed immutably down a call-stack bound by lexical scope.

Java

```
public class RequestHandler {
    // Declare a ScopedValue container
    public final static ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

    public void handle(User user) {
        // Binds the user immutably within this block
        ScopedValue.where(CURRENT_USER, user).run(() -> {
            executeBusinessLogic();
        });
        // Out of this scope, CURRENT_USER is automatically unmapped and safe
    }

    private void executeBusinessLogic() {
        // Read down the call tree without explicit parameter passing
        User loggedInUser = CURRENT_USER.get();
        System.out.println("Processing for: " + loggedInUser.name());
    }
}
```

### Compact Source Files & Instance Main Methods

Reduces Java's historically verbose script structure. You can omit `public static void main(String[] args)` and write classless files entirely for quick tasks, leveraging the new built-in `java.lang.IO` utility.

Java

```
// Save this directly as script.java (No explicit class definition or static wrappers needed)
void main() {
    String name = IO.readln("Enter name: ");
    IO.println("Hello, " + name);
}
```

## 💡 Quick Summary Cheat Sheet for Your Interview

| **Version** | **Main Theme**                      | **Killer Features to Mention**                                                        |
| ----------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| **Java 11** | Modernizing Ecosystem               | Modern `HttpClient`, `var` in Lambdas, Local Single-File execution.                   |
| **Java 17** | Data Modeling                       | `Records`, `Sealed Classes`, Pattern Matching Foundation.                             |
| **Java 21** | Extreme Scalability                 | `Virtual Threads`, `Record Patterns`, Sequenced Collections.                          |
| **Java 25** | Developer Ergonomics & Architecture | Flexible Constructors, `Scoped Values` (optimized context tracking), Classless files. |

| Java Version            | Key Features                                                                                                        | Example                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Java 8 (2014)**       | • Lambda Expressions<br>• Stream API<br>• Optional                                                                  | ```java\nlist.stream().filter(x -> x > 10).toList();\n\nOptional<String> name = Optional.of(\"Vinay\");\n```                        |
| **Java 11 (2018, LTS)** | • `var` in Lambda Parameters<br>• New String Methods (`isBlank()`, `lines()`, `repeat()`)<br>• New HTTP Client API  | ```java\n\"  \".isBlank();\n\"Hi\".repeat(3);\n\nHttpClient client = HttpClient.newHttpClient();\n```                               |
| **Java 17 (2021, LTS)** | • Sealed Classes<br>• Pattern Matching for `instanceof`<br>• Enhanced Random Generator                              | ```java\nif (obj instanceof String s) {\n    System.out.println(s.length());\n}\n```                                                |
| **Java 21 (2023, LTS)** | • Virtual Threads ⭐<br>• Record Patterns<br>• Sequenced Collections                                                 | ```java\nThread.startVirtualThread(() -> {\n    System.out.println(\"Hello\");\n});\n\nrecord Employee(int id, String name) {}\n``` |
| **Java 25 (2025, LTS)** | • Stable evolution of Structured Concurrency<br>• Scoped Values improvements<br>• Performance & GC/JVM enhancements | ```java\n// Mainly JVM/runtime improvements.\n// No major syntax changes.\n```                                                      |