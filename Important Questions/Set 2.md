# Strings

## 16. Are all Strings immutable?

**Interview Answer**

Yes. Every `String` object in Java is immutable, meaning once it is
created, its value cannot be changed. Any operation like `concat()`,
`replace()`, or `substring()` creates a new String object instead of
modifying the existing one.

**Why Java made Strings immutable** - Enables String Pool
optimization. - Thread-safe without synchronization. - More secure for
passwords, file paths, URLs, and class loading. - Hash code can be
cached, improving HashMap performance.

``` java
String s1 = "Hello";
s1.concat(" World");

System.out.println(s1); // Hello
```

------------------------------------------------------------------------

## 17. Where are String values stored in memory?

Java stores Strings in two places:

-   **String Pool (Heap)** -- String literals.
-   **Heap** -- Strings created using `new`.

``` java
String s1 = "Java";
String s2 = "Java";          // Same pooled object
String s3 = new String("Java"); // New heap object
```

Memory:

``` text
String Pool
------------
"Java" <--- s1, s2

Heap
------------
new String("Java") <--- s3
```

------------------------------------------------------------------------

## 18. Why should you avoid String concatenation (+) inside loops?

Each concatenation creates a new String object because Strings are
immutable.

``` java
String result = "";

for(int i=0;i<1000;i++){
    result += i;
}
```

This creates thousands of temporary objects, increasing memory usage and
GC overhead.

Time complexity becomes approximately **O(n²)**.

------------------------------------------------------------------------

## 19. How do you solve the above problem?

Use **StringBuilder**.

``` java
StringBuilder sb = new StringBuilder();

for(int i=0;i<1000;i++){
    sb.append(i);
}

String result = sb.toString();
```

`StringBuilder` modifies the same object, making it much faster.

------------------------------------------------------------------------

## 20. Difference between String and StringBuffer

  -----------------------------------------------------------------------
  String                      StringBuffer
  --------------------------- -------------------------------------------
  Immutable                   Mutable

  Thread-safe because         Thread-safe using synchronized methods
  immutable                   

  New object on modification  Same object is modified

  Faster for read-only data   Slower due to synchronization
  -----------------------------------------------------------------------

Use **String** for constants and read-only text.

Use **StringBuffer** when multiple threads modify the same text.

------------------------------------------------------------------------

## 21. Difference between StringBuilder and StringBuffer

  StringBuilder                  StringBuffer
  ------------------------------ -----------------------------
  Not synchronized               Synchronized
  Faster                         Slightly slower
  Single-threaded applications   Multi-threaded applications

In most Spring Boot applications, **StringBuilder** is preferred because
request processing is generally single-threaded per request.

------------------------------------------------------------------------

## 22. Useful methods in String class

Some commonly used methods:

``` java
String name = "Interview";

name.length();              // 9
name.charAt(0);             // I
name.substring(0,5);        // Inter
name.toUpperCase();
name.toLowerCase();
name.equals("Interview");
name.equalsIgnoreCase("interview");
name.contains("view");
name.startsWith("Inter");
name.endsWith("view");
name.replace("view","test");
name.split("e");
name.trim();
name.isEmpty();
name.indexOf('v');
```

### Interview Tip

Interviewers often ask:

**Why is String immutable?**

Mention these four points in order:

1.  String Pool
2.  Security
3.  Thread Safety
4.  HashMap performance

That is usually considered a complete interview answer.
