# Object-Oriented Programming Basics

## 23. What is a class?

**Interview Answer**

A class is a blueprint or template used to create objects. It defines
the data (fields) and behavior (methods) that objects of that type will
have.

``` java
class Employee {
    int id;
    String name;

    void work() {
        System.out.println("Working...");
    }
}
```

------------------------------------------------------------------------

## 24. What is an object?

An object is a runtime instance of a class. It occupies memory and has
its own state and behavior.

``` java
Employee emp = new Employee();
```

------------------------------------------------------------------------

## 25. What is the state of an object?

The **state** is represented by the values stored in its instance
variables.

Example:

``` java
Employee emp = new Employee();
emp.id = 101;
emp.name = "Vinay";
```

State = `id=101`, `name=Vinay`

------------------------------------------------------------------------
### Access Modifiers
![[AccessModifiers.png]]

---

## 26. What is the behavior of an object?

Behavior is defined by the methods that an object can perform.

``` java
emp.work();
```

------------------------------------------------------------------------

## 27. What is the superclass of every class in Java?

Every Java class directly or indirectly extends **Object**.

Common methods inherited: - `toString()` - `equals()` - `hashCode()` -
`wait()` - `notify()` - `getClass()`

------------------------------------------------------------------------

## 28. Explain the `toString()` method.

`toString()` returns the string representation of an object.

Override it for meaningful logging.

``` java
@Override
public String toString() {
    return "Employee{id=" + id + ", name='" + name + "'}";
}
```

------------------------------------------------------------------------

## 29. What is the use of `equals()`?

`equals()` compares **logical equality**, not object references.

``` java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

------------------------------------------------------------------------

## 30. Important rules while overriding `equals()`

-   Reflexive
-   Symmetric
-   Transitive
-   Consistent
-   Return false for null

**Always override `hashCode()` whenever you override `equals()`.**

------------------------------------------------------------------------

## 31. What is `hashCode()` used for?

It returns an integer hash value used by hash-based collections like
`HashMap` and `HashSet` for fast lookup.

Contract: - Equal objects must have equal hash codes. - Unequal objects
may still have the same hash code.

------------------------------------------------------------------------

## 32. Explain inheritance.

Inheritance allows one class to acquire the properties and methods of
another class.

``` java
class Animal {
    void eat() {}
}

class Dog extends Animal {
    void bark() {}
}
```

Benefits: - Code reuse - Extensibility - Polymorphism

------------------------------------------------------------------------

## 33. What is method overloading?

Same method name with different parameter lists in the same class.

``` java
void add(int a,int b){}
void add(double a,double b){}
```

Resolved at **compile time**.

------------------------------------------------------------------------

## 34. What is method overriding?

Subclass provides its own implementation of a superclass method.

``` java
class Animal{
    void sound(){ }
}

class Dog extends Animal{
    @Override
    void sound(){
        System.out.println("Bark");
    }
}
```

Resolved at **runtime** (dynamic polymorphism).

------------------------------------------------------------------------

## 35. Can a superclass reference hold a subclass object?

Yes.

``` java
Animal a = new Dog();
a.sound();
```

This enables runtime polymorphism.

------------------------------------------------------------------------

## 36. Is multiple inheritance allowed?

Java does **not** support multiple inheritance of classes to avoid the
Diamond Problem.

However, a class can implement multiple interfaces.

------------------------------------------------------------------------

## 37. What is an interface?

An interface defines a contract that implementing classes must follow.

------------------------------------------------------------------------

## 38. How do you define an interface?

``` java
interface PaymentService {
    void pay(double amount);
}
```

------------------------------------------------------------------------

## 39. How do you implement an interface?

``` java
class CardPayment implements PaymentService {

    @Override
    public void pay(double amount){
        System.out.println(amount);
    }
}
```

------------------------------------------------------------------------

## 40. Tricky things about interfaces

-   Variables are implicitly `public static final`.
-   Methods are `public abstract` by default.
-   Java 8 introduced `default` and `static` methods.
-   Java 9 introduced `private` methods.

------------------------------------------------------------------------

## 41. Can an interface extend another interface?

Yes.

``` java
interface A {}
interface B extends A {}
```

------------------------------------------------------------------------

## 42. Can a class implement multiple interfaces?

Yes.

``` java
class Demo implements Runnable, AutoCloseable {
    public void run(){}
    public void close(){}
}
```

------------------------------------------------------------------------

## 43. What is an abstract class?

A class declared with the `abstract` keyword. It cannot be instantiated
and may contain both abstract and concrete methods.

------------------------------------------------------------------------

## 44. When do you use an abstract class?

Use it when multiple related classes share common code and state.

------------------------------------------------------------------------

## 45. How do you define an abstract method?

``` java
abstract class Shape{
    abstract double area();
}
```

------------------------------------------------------------------------

## 46. Abstract class vs Interface

  Abstract Class          Interface

  Can have state          No instance state
  Constructors allowed    No constructors
  Single inheritance      Multiple implementation
  Shared implementation   Contract


| Feature | Abstract Class | Interface |
|---------|----------------|-----------|
| **State (Instance Variables)** | ✅ Can have instance state (fields) | ❌ No instance state (only `public static final` constants) |
| **Constructors** | ✅ Constructors allowed | ❌ No constructors |
| **Inheritance** | ❌ Single inheritance (`extends` one class) | ✅ Multiple implementation (`implements` multiple interfaces) |
| **Purpose** | Shared implementation + partial abstraction | Contract / capability definition |
| **Methods** | Can have abstract and concrete methods | Can have abstract, `default`, `static`, and `private` methods (Java 8/9+) |
| **Access Modifiers** | Methods/fields can have any access modifier | Abstract methods are `public` by default; fields are `public static final` |
| **Fields** | Can have mutable instance variables | Only constants (`public static final`) |
| **Object Creation** | ❌ Cannot be instantiated | ❌ Cannot be instantiated |
| **When to Use** | When classes share common state and behavior | When unrelated classes should follow the same contract |

------------------------------------------------------------------------

## 47. What is a constructor?

A constructor initializes an object and has the same name as the class.

------------------------------------------------------------------------

## 48. What is a default constructor?

If no constructor is written, Java provides a no-argument constructor
automatically.

------------------------------------------------------------------------

## 49. Will code compile if a parameterized constructor exists but no no-arg constructor?

No. Java does not generate the default constructor once any constructor
is defined.

------------------------------------------------------------------------

## 50. How do you call a superclass constructor?

Using `super()`.

``` java
class Dog extends Animal{
    Dog(){
        super();
    }
}
```

------------------------------------------------------------------------

## 51. Can `super()` and `this()` be used together?

No. Both must be the first statement of the constructor, so only one can
be used.

------------------------------------------------------------------------

## 52. What is `this()`?

It invokes another constructor in the same class.

``` java
Employee(){
    this(101);
}

Employee(int id){
    this.id=id;
}
```

------------------------------------------------------------------------

## 53. Can a constructor be called directly from a method?

No. Constructors are invoked only during object creation.

------------------------------------------------------------------------

## 54. Is superclass constructor called automatically?

Yes. If you don't explicitly call `super()`, Java inserts it
automatically as the first statement.

------------------------------------------------------------------------

## Interview Summary

Remember these frequently asked distinctions:

  Concept       Compile Time   Runtime
  ------------- -------------- ---------
  Overloading   ✔              
  Overriding                   ✔

  Keyword     Purpose
  ----------- -----------------------------------------
  `this()`    Calls another constructor in same class
  `super()`   Calls parent constructor
