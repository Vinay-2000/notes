# Exception Handling

## 91. Why is exception handling important?

**Interview Answer**

Exception handling allows an application to handle unexpected runtime
problems gracefully instead of terminating abruptly.

Benefits: - Prevents application crashes - Separates normal logic from
error handling - Helps in debugging and logging - Enables recovery when
possible

------------------------------------------------------------------------

## 92. What design pattern is used in exception handling?

Java exception handling follows the **Chain of Responsibility** pattern.

When an exception is thrown, the JVM searches up the call stack until it
finds a matching `catch` block.

------------------------------------------------------------------------

## 93. What is the need for the `finally` block?

`finally` contains cleanup code that should execute regardless of
whether an exception occurs.

Typical uses: - Close files - Close database connections - Release locks

``` java
try {
    // business logic
} finally {
    connection.close();
}
```

------------------------------------------------------------------------

## 94. When is `finally` not executed?

`finally` usually executes, except in rare cases: - `System.exit()` -
JVM crash - Power failure - Infinite loop before reaching `finally`

------------------------------------------------------------------------

## 95. Will `finally` execute if there is a return?

Yes.

``` java
static int test() {
    try {
        return 10;
    } finally {
        System.out.println("Finally");
    }
}
```

Output:

``` text
Finally
```

------------------------------------------------------------------------

## 96. Is `try` without `catch` allowed?

Yes, if it has a `finally`.

``` java
try {
    System.out.println("Hello");
} finally {
    System.out.println("Cleanup");
}
```

------------------------------------------------------------------------

## 97. Is `try` without both `catch` and `finally` allowed?

No. Compilation error.

------------------------------------------------------------------------

## 98. Explain the exception hierarchy.

``` text
Object
   |
Throwable
 ├── Error
 └── Exception
      ├── RuntimeException
      └── Checked Exceptions
```

------------------------------------------------------------------------

## 99. Difference between Error and Exception

  Error                   Exception
  ----------------------- ---------------------------
  Serious JVM problem     Application-level problem
  Usually unrecoverable   Usually recoverable
  Don't handle            Handle appropriately

Example Errors: - OutOfMemoryError - StackOverflowError

------------------------------------------------------------------------

## 100. Checked vs Unchecked Exceptions

  Checked                   Unchecked
  ------------------------- -------------------------
  Checked at compile time   Runtime
  Must handle or declare    Optional
  Extend Exception          Extend RuntimeException

Examples: - IOException - SQLException

Unchecked: - NullPointerException - IllegalArgumentException -
ArithmeticException

------------------------------------------------------------------------

## 101. How do you throw an exception?

``` java
throw new IllegalArgumentException("Invalid Age");
```

------------------------------------------------------------------------

## 102. What happens when you throw a checked exception?

You must either: - Handle it using `try-catch` - Declare it using
`throws`

``` java
public void read() throws IOException {
}
```

------------------------------------------------------------------------

## 103. How do you resolve compilation errors for checked exceptions?

Two options:

1.  Handle

``` java
try{
}catch(IOException e){
}
```

2.  Declare

``` java
throws IOException
```

------------------------------------------------------------------------

## 104. How do you create a custom exception?

``` java
public class InvalidAgeException extends Exception{

    public InvalidAgeException(String message){
        super(message);
    }
}
```

Throw:

``` java
throw new InvalidAgeException("Age must be greater than 18");
```

For business validation in Spring Boot, extending `RuntimeException` is
more common.

------------------------------------------------------------------------

## 105. How do you handle multiple exception types?

Java 7 introduced multi-catch.

``` java
try{

}catch(IOException | SQLException e){
    e.printStackTrace();
}
```

------------------------------------------------------------------------

## 106. What is try-with-resources?

Introduced in Java 7.

Resources implementing `AutoCloseable` are closed automatically.

``` java
try(BufferedReader br =
        new BufferedReader(new FileReader("file.txt"))){

}
```

------------------------------------------------------------------------

## 107. How does try-with-resources work?

The compiler automatically generates a hidden `finally` block to close
resources in reverse order.

------------------------------------------------------------------------

## 108. Exception handling best practices

-   Catch specific exceptions.
-   Never swallow exceptions.
-   Log meaningful information.
-   Don't use exceptions for normal flow.
-   Prefer custom exceptions for business rules.
-   Wrap low-level exceptions when appropriate.
-   Close resources using try-with-resources.

### Spring Boot Interview Connection

In Spring Boot:

-   Create custom exceptions extending `RuntimeException`.
-   Handle them globally using `@ControllerAdvice`.
-   Use `@ExceptionHandler` to return consistent API responses.

``` java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handle(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }
}
```

## Interview Cheat Sheet

  Topic                 Remember
  --------------------- ----------------------
  Checked Exception     Compile-time
  Unchecked Exception   Runtime
  Error                 JVM issue
  finally               Cleanup
  throw                 Throw object
  throws                Declare method
  try-with-resources    Auto close resources
