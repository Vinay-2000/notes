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
