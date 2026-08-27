### Composition vs Inheritance

**Composition** means a class **contains/uses another class** to achieve its functionality. It represents a **"has-a"** relationship and is generally preferred because it gives more flexibility and reduces tight coupling.

```
class Car {
    private Engine engine;
    Car(Engine engine) {
        this.engine = engine;
    }
}
```

Here, `Car` **has an** `Engine`.
**Inheritance** means a class **extends another class** and inherits its properties and behavior. It represents an **"is-a"** relationship and creates tighter coupling between parent and child.

```
class Dog extends Animal {
}
```

Here, `Dog` **is an** `Animal`.

### Simple rule

> **Composition = has-a → use another object**  
> **Inheritance = is-a → extend another class**
> For LLD, prefer **composition over inheritance** unless there is a genuine **is-a** relationship and the inheritance makes the design clearer.

---

**Association** is a general relationship where **two classes are connected or interact with each other**. Neither class necessarily owns the other.
For example:

```
class Teacher {
    void teach(Student student) {
        // ...
    }
}
class Student {
}
```

A `Teacher` **is associated with** a `Student` because they interact, but the `Teacher` doesn't own the `Student`.

### Think of it as:

> **Association = "knows/uses/works with"**
> Examples:

```
Doctor ─── Patient
Teacher ─── Student
Customer ─── Bank
Driver ─── Car
```

The important distinction is:

```
Association   → general relationship
Composition   → strong ownership ("has-a")
Aggregation   → weaker ownership ("has-a")
Inheritance   → "is-a"
```

So **composition and aggregation are more specific forms of association**.

---
**Aggregation** is a **weak "has-a" relationship** where one object contains or uses another object, but the contained object can exist independently.

Example:
```
class Department {
    List<Teacher> teachers;
}
```

A `Department` **has Teachers**, but a `Teacher` can exist without the `Department`.
Think:
> **Aggregation = has-a, but both can exist independently.**

Example: `University → Professor`
If the university is deleted, the professors can still exist.

---
### Interface vs abstract class

|Feature|Interface|Abstract Class|
|---|---|---|
|Purpose|Defines a **contract**|Provides a **common base/class**|
|Methods|Abstract, `default`, `static`, and `private` methods|Abstract + concrete methods|
|Variables|`public static final` by default (constants)|Can have instance/static/final variables|
|Constructor|❌ Cannot have a constructor|✅ Can have a constructor|
|Constructor invocation|—|Cannot be instantiated directly; constructor is invoked when a subclass object is created, through `super()`|
|State|❌ Cannot have instance state|✅ Can maintain instance state|
|Inheritance|A class can implement **multiple interfaces**|A class can extend only **one class**|
|Access modifiers|Interface methods are generally `public` (except private helper methods)|Methods can be `private`, `protected`, `public`, etc.|
|Keyword|`implements`|`extends`|
|Relationship|Defines **"can do"** behavior|Defines **"is a"** common base|
|Best use|When different classes need to follow the same contract|When closely related classes share state/behavior|
|Example|`Payment`, `Runnable`, `Comparable`|`Animal`, `Vehicle`, `Employee`|

**Simple rule:** Interface → **contract/capability**. Abstract class → **shared base + common behavior/state**.

---
`final` in Java means **"cannot be changed further."** What exactly cannot change depends on where you use it.

|Usage|Meaning|Example|
|---|---|---|
|`final` variable|Value/reference cannot be reassigned|`final int x = 10;`|
|`final` method|Cannot be overridden by a subclass|`final void run() {}`|
|`final` class|Cannot be inherited/extended|`final class A {}`|

### 1. Final variable

```
final int x = 10;
x = 20; // ❌
```

For an object reference:

```
final List<String> list = new ArrayList<>();
list.add("A");       // ✅
list = new ArrayList<>(); // ❌
```

`final` prevents changing the **reference**, not the object's internal state.

### 2. Final method

```
class Animal {
    final void breathe() {
        System.out.println("Breathing");
    }
}

class Dog extends Animal {
    void breathe() { } // ❌ Cannot override
}
```
Although we can overload it

### 3. Final class

```
final class Animal {
}
```

You cannot extend it:

```
class Dog extends Animal { } // ❌
```

A common example is `String`, which is a `final` class.

**Easy way to remember:**

> `final variable` → cannot reassign  
> `final method` → cannot override  
> `final class` → cannot extend

---
`static` means the member **belongs to the class itself, rather than to individual objects**.

|Usage|Meaning|Example|
|---|---|---|
|`static` variable|One shared copy for the entire class|`static int count;`|
|`static` method|Can be called using the class without creating an object|`Math.max()`|
|`static` block|Runs once when the class is loaded/initialized|`static { ... }`|
|`static` nested class|Nested class doesn't require an outer-class object|`static class Helper {}`|

---
### Immutability

Immutability means that once an object is created, its state cannot be changed.

How to create an immutable class
- Make the class `final` so it can't be subclassed.
- Make fields `private`.
- Make fields `final`.
- Initialize fields through the constructor.
- Don't provide setters.
- If a field contains a **mutable object**, don't expose the original reference. Use defensive copies.
For Objects we can do this
```
final class Employee {
    private final String name;
    private final Address address;

    public Employee(String name, Address address) {
        this.name = name;
        this.address = new Address(address.getCity());
    }

    public Address getAddress() {
        return new Address(address.getCity()); //Even if user changes the state, Employee Object wont change
    }
}
```

---
## UML Unified Modeling Language


```
+----------------------+
|       User           |
+----------------------+
| - id: Long           |
| - name: String       |
| - email: String      |
+----------------------+
| + login(): void      |
| + logout(): void     |
+----------------------+
```

### UML visibility

```
+ public 
- private 
# protected 
~ package-private
```