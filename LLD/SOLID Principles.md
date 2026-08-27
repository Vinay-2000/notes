# SOLID Principles — LLD Interview Notes

SOLID is a set of five object-oriented design principles that help make code **maintainable, extensible, testable, and loosely coupled**.

- **S** — Single Responsibility Principle
- **O** — Open/Closed Principle
- **L** — Liskov Substitution Principle
- **I** — Interface Segregation Principle
- **D** — Dependency Inversion Principle

---

# 1. Single Responsibility Principle (SRP)

## Definition

> A class should have **one responsibility** and therefore **one reason to change**.
> The important phrase is **"one reason to change."**
> It does NOT mean a class can have only one method.

## Bad Example

```java
class InvoiceService {
    public void createInvoice() {
        // create invoice
    }
    public void saveToDatabase() {
        // save invoice
    }
    public void sendEmail() {
        // send email
    }
}
```

This class has multiple responsibilities:

- Invoice creation
- Database persistence
- Email notification

## Better Design

```java
class InvoiceService {
    public Invoice createInvoice() {
        // create invoice
    }
}
```

```java
class InvoiceRepository {
    public void save(Invoice invoice) {
        // save invoice
    }
}
```

```java
class EmailService {
    public void send(Invoice invoice) {
        // send email
    }
}
```

### Interview Point

Don't say:

> "A class should have only one method."
> Say:
> "A class should have one responsibility or one reason to change."

---

# 2. Open/Closed Principle (OCP)

## Definition

> Software entities should be **open for extension but closed for modification**.

You should be able to add new behavior without repeatedly modifying stable existing code.

## Bad Example

```java
class PaymentService {
    public void pay(String type) {
        if (type.equals("CARD")) {
            // card payment
        } else if (type.equals("UPI")) {
            // UPI payment
        } else if (type.equals("PAYPAL")) {
            // PayPal payment
        }
    }
}
```

Every new payment method requires modifying `PaymentService`.

## Better Design

```java
interface PaymentMethod {
    void pay();
}
```

```java
class CardPayment implements PaymentMethod {
    public void pay() {
        // card payment
    }
}
```

```java
class UpiPayment implements PaymentMethod {
    public void pay() {
        // UPI payment
    }
}
```

```java
class PaypalPayment implements PaymentMethod {
    public void pay() {
        // PayPal payment
    }
}
```

```java
class PaymentService {
    private final PaymentMethod paymentMethod;

    PaymentService(PaymentMethod paymentMethod) {
        this.paymentMethod = paymentMethod;
    }

    public void pay() {
        paymentMethod.pay();
    }
}
```

Now a new payment type can be added without changing `PaymentService`.

### Common ways to achieve OCP

- Interfaces
- Polymorphism
- Strategy Pattern
- Factory Pattern
- Dependency Injection

### Interview Point

OCP does NOT mean "never modify existing code."

It means designing stable parts so new behavior can often be added through extension rather than modifying existing logic.

---

# 3. Liskov Substitution Principle (LSP)

## Definition

> Objects of a child class should be usable wherever objects of the parent class are expected without breaking the correctness of the program.

In simple terms:

> If `B` is a subtype of `A`, replacing `A` with `B` should not break expected behavior.

## Bad Example

```java
class Bird {
    void fly() {
        System.out.println("Flying");
    }
}
```

```java
class Penguin extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException();
    }
}
```

Now:

```java
Bird bird = new Penguin();
bird.fly(); // breaks expected behavior
```

The problem is the abstraction is wrong: not every bird can fly.

## Better Design

```java
interface Bird {
    void eat();
}
```

```java
interface FlyingBird extends Bird {
    void fly();
}
```

```java
class Sparrow implements FlyingBird {
    public void eat() {
    }

    public void fly() {
    }
}
```

```java
class Penguin implements Bird {
    public void eat() {
    }
}
```

### LSP Violation Indicators

Watch for subclasses that:

- Throw `UnsupportedOperationException` for inherited behavior
- Return completely unexpected results
- Require stronger conditions than the parent
- Break assumptions made by the parent
- Change the meaning of inherited methods

### Interview Answer

> "LSP means a subclass should be substitutable for its parent without breaking the expected behavior."

---

# 4. Interface Segregation Principle (ISP)

## Definition

> Clients should not be forced to depend on methods they do not need.

In simple terms:

> Prefer **small, focused interfaces** over one huge interface.

## Bad Example

```java
interface Machine {
    void print();
    void scan();
    void fax();
}
```

```java
class SimplePrinter implements Machine {
    public void print() {
    }

    public void scan() {
        throw new UnsupportedOperationException();
    }

    public void fax() {
        throw new UnsupportedOperationException();
    }
}
```

The printer is forced to implement functionality it does not support.

## Better Design

```java
interface Printer {
    void print();
}
```

```java
interface Scanner {
    void scan();
}
```

```java
interface Fax {
    void fax();
}
```

```java
class SimplePrinter implements Printer {
    public void print() {
    }
}
```

A multifunction printer can implement all three.

### ISP vs SRP

- **SRP** → focuses on a class/module having a focused responsibility.
- **ISP** → focuses on interfaces not forcing clients to depend on unnecessary methods.

---

# 5. Dependency Inversion Principle (DIP)

## Definition

> High-level modules should not depend directly on low-level modules. Both should depend on abstractions.

Also:

> Abstractions should not depend on details. Details should depend on abstractions.

## Bad Example

```java
class MySQLDatabase {
    void save() {
        // save to MySQL
    }
}
```

```java
class UserService {
    private MySQLDatabase database = new MySQLDatabase();

    void saveUser() {
        database.save();
    }
}
```

`UserService` is tightly coupled to `MySQLDatabase`.

## Better Design

```java
interface UserRepository {
    void save();
}
```

```java
class MySQLUserRepository implements UserRepository {
    public void save() {
        // MySQL
    }
}
```

```java
class PostgreSQLUserRepository implements UserRepository {
    public void save() {
        // PostgreSQL
    }
}
```

```java
class UserService {
    private final UserRepository repository;

    UserService(UserRepository repository) {
        this.repository = repository;
    }

    void saveUser() {
        repository.save();
    }
}
```

Now `UserService` depends on `UserRepository`, not directly on MySQL.

---

# DIP vs Dependency Injection

These are related but NOT the same.

### Dependency Inversion Principle

A **design principle**:

> High-level code should depend on abstractions rather than concrete implementations.

### Dependency Injection

A **technique** for providing dependencies from outside the class.

```java
class UserService {
    private final UserRepository repository;

    UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

So:

> **DIP = design principle**  
> **DI = technique used to achieve loose coupling**

---

# SOLID Quick Comparison

| Principle | Main Idea                                          | Typical Problem                                              |
| --------- | -------------------------------------------------- | ------------------------------------------------------------ |
| SRP       | One responsibility / reason to change              | God class                                                    |
| OCP       | Extend without unnecessarily modifying stable code | Huge `if/else` or `switch` for new behavior                  |
| LSP       | Subtypes must be safely substitutable              | Child breaks parent contract                                 |
| ISP       | Small, focused interfaces                          | Classes forced to implement unused methods                   |
| DIP       | Depend on abstractions                             | High-level class directly creates/uses concrete dependencies |

---

# How to Identify SOLID Violations

### SRP

Ask:

> "Does this class have multiple unrelated reasons to change?"

If yes → possible SRP violation.

### OCP

Ask:

> "Do I need to keep modifying this class every time I add a new type or behavior?"

If yes → possible OCP violation.

### LSP

Ask:

> "Can I replace the parent object with this child without breaking expected behavior?"

If no → LSP violation.

### ISP

Ask:

> "Is this class forced to implement methods it doesn't need?"

If yes → ISP violation.

### DIP

Ask:

> "Does my high-level business logic directly depend on a concrete implementation?"

If yes → possible DIP violation.

---

# SOLID and Common Design Patterns

| Principle | Commonly Used With                                 |
| --------- | -------------------------------------------------- |
| SRP       | Service/Repository separation, Facade              |
| OCP       | Strategy, Factory, Template Method                 |
| LSP       | Proper inheritance and polymorphism                |
| ISP       | Small role-based interfaces                        |
| DIP       | Dependency Injection, Strategy, Repository Pattern |

---

# Interview-Ready Definitions

### SRP

> A class should have one responsibility and therefore one reason to change.

### OCP

> Software entities should be open for extension but closed for modification.

### LSP

> Subtypes should be substitutable for their base types without breaking the expected behavior of the program.

### ISP

> Clients should not be forced to depend on methods they do not use.

### DIP

> High-level modules should depend on abstractions rather than concrete low-level implementations.

---

# Most Important Things to Remember

- [ ] SRP → **One reason to change**
- [ ] OCP → **Extend behavior without unnecessarily modifying stable code**
- [ ] LSP → **Child must honor parent's contract**
- [ ] ISP → **Don't force clients to depend on unused methods**
- [ ] DIP → **Depend on abstractions, not concrete implementations**
- [ ] DIP ≠ DI
- [ ] Composition + interfaces are frequently used to achieve SOLID designs
- [ ] SOLID principles are guidelines, not absolute rules
- [ ] Don't add abstractions just to "follow SOLID"
- [ ] Always explain the trade-off behind your design
