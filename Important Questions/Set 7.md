# Miscellaneous Java Topics

## 109. What are the default values in an array?

**Interview Answer**

When an array is created, Java initializes its elements with default
values based on the data type.

  Data Type   Default Value
  ----------- ------------------
  byte        0
  short       0
  int         0
  long        0L
  float       0.0f
  double      0.0
  char        '`\u0`{=tex}000'
  boolean     false
  Object      null

``` java
int[] arr = new int[3];

System.out.println(arr[0]); // 0
```

------------------------------------------------------------------------

## 110. How do you loop through an array using an enhanced for loop?

``` java
int[] numbers = {10,20,30};

for(int number : numbers){
    System.out.println(number);
}
```

Best when you don't need the index.

------------------------------------------------------------------------

## 111. How do you print the contents of an array?

Using `Arrays.toString()`.

``` java
int[] arr = {1,2,3};

System.out.println(Arrays.toString(arr));
```

For multidimensional arrays:

``` java
Arrays.deepToString(array);
```

------------------------------------------------------------------------

## 112. How do you compare two arrays?

Use `Arrays.equals()`.

``` java
int[] a = {1,2,3};
int[] b = {1,2,3};

System.out.println(Arrays.equals(a,b));
```

For multidimensional arrays:

``` java
Arrays.deepEquals(a,b);
```

Avoid using `==` because it compares references.

------------------------------------------------------------------------

## 113. What is an enum?

An enum represents a fixed set of constants.

``` java
enum Status{
    NEW,
    ACTIVE,
    CLOSED
}
```

Benefits: - Type safety - Readability - Switch support

------------------------------------------------------------------------

## 114. Can switch be used with enums?

Yes.

``` java
switch(status){

    case ACTIVE:
        break;

    case CLOSED:
        break;

    default:
}
```

------------------------------------------------------------------------

## 115. What are varargs?

Variable arguments allow passing any number of parameters.

``` java
public int sum(int... numbers){

    int total = 0;

    for(int num : numbers){
        total += num;
    }

    return total;
}
```

Usage:

``` java
sum();
sum(10);
sum(10,20,30);
```

Only one varargs parameter is allowed and it must be the last parameter.

------------------------------------------------------------------------

## 116. What are assertions?

Assertions are used to verify assumptions during development.

``` java
assert age >= 18;
```

If the condition is false, an `AssertionError` is thrown.

Enable assertions:

``` text
java -ea MyProgram
```

------------------------------------------------------------------------

## 117. When should assertions be used?

Use assertions: - Internal testing - Development - Debugging

Do **not** use assertions for: - User input validation - Business
validation

------------------------------------------------------------------------

## 118. What is Garbage Collection?

Garbage Collection (GC) automatically removes objects that are no longer
reachable, freeing heap memory.

Benefits: - Prevents memory leaks - Eliminates manual memory
management - Reduces dangling pointers

------------------------------------------------------------------------

## 119. Explain Garbage Collection with an example.

``` java
Employee emp = new Employee();

emp = null;
```

After `emp` becomes unreachable, the object is eligible for garbage
collection.

Eligibility does **not** guarantee immediate collection.

------------------------------------------------------------------------

## 120. When does Garbage Collection run?

Java does not guarantee when GC runs.

The JVM decides based on: - Heap usage - Memory pressure - GC algorithm

Calling:

``` java
System.gc();
```

is only a request.

------------------------------------------------------------------------

## 121. Garbage Collection best practices

-   Remove unnecessary object references.
-   Avoid creating excessive temporary objects.
-   Prefer StringBuilder over String concatenation in loops.
-   Close resources properly.
-   Don't call `System.gc()` manually.

------------------------------------------------------------------------

## 122. What are initialization blocks?

Initialization blocks execute whenever an object is created.

``` java
class Employee{

    {
        System.out.println("Instance Block");
    }
}
```

Runs before the constructor.

------------------------------------------------------------------------

## 123. What is a static initializer?

Runs once when the class is loaded.

``` java
class Demo{

    static{
        System.out.println("Static Block");
    }
}
```

Useful for static initialization.

------------------------------------------------------------------------

## 124. What is an instance initializer block?

A non-static initialization block.

Runs every time an object is created before the constructor.

------------------------------------------------------------------------

## 125. What is tokenizing?

Tokenizing means splitting text into smaller pieces called tokens.

------------------------------------------------------------------------

## 126. Example of tokenizing

``` java
String sentence = "Java Spring Boot";

String[] words = sentence.split(" ");
```

Alternative:

``` java
StringTokenizer tokenizer =
    new StringTokenizer(sentence);
```

------------------------------------------------------------------------

## 127. What is serialization?

Serialization converts an object into a byte stream so it can be stored
or transmitted.

``` java
class Employee implements Serializable{

}
```

------------------------------------------------------------------------

## 128. How do you serialize an object?

``` java
ObjectOutputStream out =
    new ObjectOutputStream(
        new FileOutputStream("emp.ser"));

out.writeObject(employee);
```

------------------------------------------------------------------------

## 129. How do you deserialize?

``` java
ObjectInputStream in =
    new ObjectInputStream(
        new FileInputStream("emp.ser"));

Employee emp =
    (Employee) in.readObject();
```

------------------------------------------------------------------------

## 130. How do you serialize only part of an object?

Use the `transient` keyword.

``` java
class Employee implements Serializable{

    private transient String password;
}
```

Transient fields are ignored during serialization.

------------------------------------------------------------------------

## 131. How do you serialize an object hierarchy?

Every non-transient object in the hierarchy must implement
`Serializable`.

------------------------------------------------------------------------

## 132. Are constructors executed during deserialization?

No.

Objects are recreated without invoking constructors.

------------------------------------------------------------------------

## 133. Are static variables serialized?

No.

Static variables belong to the class, not individual objects.

------------------------------------------------------------------------

# Interview Cheat Sheet

  Topic             Key Point
  ----------------- ------------------------------------------
  Arrays            Default values initialized automatically
  Arrays.equals()   Compare array contents
  Enum              Fixed constants
  Varargs           Variable number of arguments
  Assertion         Development only
  GC                Automatic memory management
  Static Block      Runs once
  Instance Block    Runs before constructor
  Serialization     Object → Byte Stream
  Deserialization   Byte Stream → Object
  transient         Skip field during serialization
  static            Never serialized

## Spring Boot Interview Connection

These topics frequently appear in Spring Boot:

-   `enum` for API status, order state, payment state.
-   `transient` in JPA entities (non-persistent fields).
-   Serialization in distributed systems, caching (Redis), messaging
    (Kafka), and session management.
