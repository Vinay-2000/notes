# Advanced Object-Oriented Concepts & Modifiers

## 55. What is polymorphism?

**Interview Answer**

Polymorphism means **one interface, multiple implementations**. The same
method call can behave differently depending on the actual object.

Types: - Compile-time (Method Overloading) - Runtime (Method Overriding)

``` java
Animal animal = new Dog();
animal.sound();   // Bark
```

Runtime polymorphism is heavily used in Spring through Dependency
Injection.

------------------------------------------------------------------------

## 56. What is the use of `instanceof`?

It checks whether an object belongs to a particular class or interface.

``` java
if(obj instanceof String){
    System.out.println("It's a String");
}
```

Since Java 16:

``` java
if(obj instanceof String s){
    System.out.println(s.length());
}
```

------------------------------------------------------------------------

## 57. What is coupling?

Coupling measures how dependent one class is on another.

-   Tight coupling → Hard to maintain
-   Loose coupling → Easy to extend and test

Spring DI promotes loose coupling.

------------------------------------------------------------------------

## 58. What is cohesion?

Cohesion measures how closely related the responsibilities of a class
are.

High cohesion is desirable.

Example: - `EmailService` should only handle email logic.

------------------------------------------------------------------------

## 59. What is encapsulation?

Encapsulation means hiding implementation details and exposing only what
is necessary.

``` java
class Employee{
    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name){
        this.name = name;
    }
}
```

Benefits: - Data hiding - Validation - Better maintainability

------------------------------------------------------------------------

## 60. What is an inner class?

A non-static class declared inside another class.

``` java
class Outer{
    class Inner{
    }
}
```

Inner classes can directly access members of the outer class.

------------------------------------------------------------------------

## 61. What is a static inner class?

A nested class declared using `static`.

``` java
class Outer{
    static class Inner{
    }
}
```

It does not require an instance of the outer class.

------------------------------------------------------------------------

## 62. Can you create an inner class inside a method?

Yes. It is called a **local inner class**.

``` java
void display(){

    class Local{
        void print(){
            System.out.println("Hello");
        }
    }

    new Local().print();
}
```

------------------------------------------------------------------------

## 63. What is an anonymous class?

A class without a name used for one-time implementations.

``` java
Runnable r = new Runnable(){

    @Override
    public void run(){
        System.out.println("Running");
    }
};
```

Nowadays Lambdas are preferred for functional interfaces.

------------------------------------------------------------------------

# Modifiers

## 64. What is the default class modifier?

If no modifier is specified, the class has **package-private** access.

It is accessible only within the same package.

------------------------------------------------------------------------

## 65. What is the private access modifier?

Accessible only inside the same class.

Cannot be accessed outside directly.

------------------------------------------------------------------------

## 66. What is default (package-private) access?

Accessible only within the same package.

------------------------------------------------------------------------

## 67. What is protected access?

Accessible: - Same package - Subclasses in other packages

------------------------------------------------------------------------

## 68. What is public access?

Accessible from anywhere.

------------------------------------------------------------------------

## 69. Which access modifiers are accessible in the same package?

-   public
-   protected
-   default

private is not accessible.

| Modifier                     | Within Class | Within Package | Outside Package <br>(Subclass Only <br> class that extends this) | World (Everywhere) |
| ---------------------------- | ------------ | -------------- | ---------------------------------------------------------------- | ------------------ |
| **`private`**                | Yes          | No             | No                                                               | No                 |
| **`default`** _(No keyword)_ | Yes          | Yes            | No                                                               | No                 |
| **`protected`**              | Yes          | Yes            | Yes                                                              | No                 |
| **`public`**                 | Yes          | Yes            | Yes                                                              | Yes                |

------------------------------------------------------------------------

## 70. Which access modifiers are accessible in a different package?

Only: - public

protected is accessible only through inheritance.

------------------------------------------------------------------------

## 71. Which modifiers are accessible from a subclass in the same package?

-   public
-   protected
-   default

------------------------------------------------------------------------

## 72. Which modifiers are accessible from a subclass in another package?

-   public
-   protected

------------------------------------------------------------------------

## 73. Use of `final` on a class

A final class cannot be inherited.

``` java
final class Utility{
}
```

Example: - String

------------------------------------------------------------------------

## 74. Use of `final` on a method

Cannot be overridden.

------------------------------------------------------------------------

## 75. What is a final variable?

Can only be assigned once.

``` java
final int MAX = 100;
```

------------------------------------------------------------------------

## 76. What is a final argument?

Its value cannot be reassigned inside the method.

``` java
void display(final int x){
    // x = 20; // Compilation Error
}
```

------------------------------------------------------------------------

## 77. What happens when a variable is marked volatile?

`volatile` ensures visibility of changes across threads.

Every read comes directly from main memory.

It **does not provide atomicity**.

``` java
private volatile boolean running = true;
```

Common interview follow-up: \> Is volatile enough for count++?

No. `count++` is not atomic.

------------------------------------------------------------------------

## 78. What is a static variable?

A static variable belongs to the class rather than individual objects.

Only one copy exists.

``` java
class Employee{

    static int count = 0;

    Employee(){
        count++;
    }
}
```

Access using:

``` java
Employee.count;
```

------------------------------------------------------------------------

# Interview Cheat Sheet

  Concept         Meaning
  --------------- ------------------------------------------
  Polymorphism    Same interface, different implementation
  Coupling        Dependency between classes
  Cohesion        Relatedness of responsibilities
  Encapsulation   Data hiding
  volatile        Visibility only
  final class     Cannot inherit
  final method    Cannot override
  static          Belongs to class

## Spring Boot Interview Connection

Interviewers often ask:

**How does Spring achieve loose coupling?**

Answer: - Dependency Injection - Interfaces - IoC Container - Runtime
polymorphism
