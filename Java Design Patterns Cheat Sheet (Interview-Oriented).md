## 1. Singleton

- **GoF Definition:** Ensure a class has only one instance and provide a global point of access to it.
    
- **Plain Definition:** Only one object of the class should ever exist.
    
- **Problem It Solves:** Shared resources, global configuration, avoiding multiple instances.
    
- **Real-World Examples:** Spring Singleton Beans, Logger, Cache Manager.
    

Java

```
class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }

    public void print() {
        System.out.println("Singleton Instance");
    }
}

public class Main {
    public static void main(String[] args) {
        Singleton s = Singleton.getInstance();
        s.print();
    }
}
```

## 2. Factory Method

- **GoF Definition:** Define an interface for creating an object, but let subclasses decide which class to instantiate.
    
- **Plain Definition:** Delegate object creation instead of using `new` everywhere.
    
- **Problem It Solves:** Tight coupling, centralized creation.
    
- **Real-World Examples:** `BeanFactory`, `Calendar.getInstance()`.
    

Java

```
interface Shape {
    void draw();
}

class Circle implements Shape {
    public void draw() {
        System.out.println("Circle");
    }
}

abstract class ShapeCreator {
    abstract Shape createShape();
}

class CircleCreator extends ShapeCreator {
    Shape createShape() {
        return new Circle();
    }
}

public class Main {
    public static void main(String[] args) {
        ShapeCreator creator = new CircleCreator();
        Shape shape = creator.createShape();
        shape.draw();
    }
}
```

## 3. Abstract Factory

- **GoF Definition:** Provide an interface for creating families of related objects.
    
- **Plain Definition:** Create matching groups of related objects.
    
- **Problem It Solves:** Ensures compatible product families.
    
- **Real-World Examples:** UI Themes, Spring `ApplicationContext`.
    

Java

```
interface Button {
    void render();
}

class WindowsButton implements Button {
    public void render() {
        System.out.println("Windows Button");
    }
}

interface UIFactory {
    Button createButton();
}

class WindowsFactory implements UIFactory {
    public Button createButton() {
        return new WindowsButton();
    }
}

public class Main {
    public static void main(String[] args) {
        UIFactory factory = new WindowsFactory();
        Button button = factory.createButton();
        button.render();
    }
}
```

## 4. Builder

- **GoF Definition:** Separate construction of a complex object from its representation.
    
- **Plain Definition:** Build objects step by step.
    
- **Problem It Solves:** Too many constructor parameters.
    
- **Real-World Examples:** Lombok `@Builder`, `HttpRequest.newBuilder()`.
    

Java

```
class User {
    private String name;
    private int age;
    private String email;

    private User(Builder b) {
        this.name = b.name;
        this.age = b.age;
        this.email = b.email;
    }

    static class Builder {
        private String name;
        private int age;
        private String email;

        Builder(String name, int age) {
            this.name = name;
            this.age = age;
        }

        Builder email(String email) {
            this.email = email;
            return this;
        }

        User build() {
            return new User(this);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        User user = new User.Builder("Vinay", 28)
                .email("vinay@email.com")
                .build();
    }
}
```

## 5. Prototype

- **GoF Definition:** Create objects by copying an existing instance.
    
- **Plain Definition:** Clone instead of creating from scratch.
    
- **Problem It Solves:** Expensive object creation.
    
- **Real-World Examples:** Spring prototype scope, Document templates.
    

Java

```
class User implements Cloneable {
    String name;

    User(String name) {
        this.name = name;
    }

    public User clone() throws CloneNotSupportedException {
        return (User) super.clone();
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        User u1 = new User("Vinay");
        User u2 = u1.clone();

        System.out.println(u1.name);
        System.out.println(u2.name);
    }
}
```

## 6. Adapter

- **GoF Definition:** Convert one interface into another expected by clients.
    
- **Plain Definition:** Translator between incompatible interfaces.
    
- **Problem It Solves:** Integrating third-party or legacy code.
    
- **Real-World Examples:** `InputStreamReader`, `HandlerAdapter`.
    

Java

```
interface PaymentProcessor {
    void pay();
}

class PaypalGateway {
    void makePayment() {
        System.out.println("Paypal Payment");
    }
}

class PaypalAdapter implements PaymentProcessor {
    private PaypalGateway gateway = new PaypalGateway();

    public void pay() {
        gateway.makePayment();
    }
}

public class Main {
    public static void main(String[] args) {
        PaymentProcessor p = new PaypalAdapter();
        p.pay();
    }
}
```

## 7. Decorator

- **GoF Definition:** Attach additional responsibilities dynamically.
    
- **Plain Definition:** Wrap an object and enhance it.
    
- **Problem It Solves:** Avoid subclass explosion.
    
- **Real-World Examples:** `BufferedInputStream`, Spring AOP.
    

Java

```
interface Coffee {
    int cost();
}

class BasicCoffee implements Coffee {
    public int cost() {
        return 100;
    }
}

class MilkDecorator implements Coffee {
    private Coffee coffee;

    MilkDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    public int cost() {
        return coffee.cost() + 20;
    }
}

public class Main {
    public static void main(String[] args) {
        Coffee coffee = new BasicCoffee();
        coffee = new MilkDecorator(coffee);

        System.out.println(coffee.cost());
    }
}
```

## 8. Facade

- **GoF Definition:** Provide a unified interface to a subsystem.
    
- **Plain Definition:** Hide complexity behind one simple method.
    
- **Problem It Solves:** Complex APIs.
    
- **Real-World Examples:** `JdbcTemplate`, Service layer wrapper.
    

Java

```
class DVD {
    void play() {
        System.out.println("Playing Movie");
    }
}

class Sound {
    void on() {
        System.out.println("Sound ON");
    }
}

class HomeTheater {
    private DVD dvd = new DVD();
    private Sound sound = new Sound();

    void watchMovie() {
        sound.on();
        dvd.play();
    }
}

public class Main {
    public static void main(String[] args) {
        new HomeTheater().watchMovie();
    }
}
```

## 9. Strategy

- **GoF Definition:** Define interchangeable algorithms.
    
- **Plain Definition:** Swap behavior at runtime.
    
- **Problem It Solves:** Large `if-else` blocks.
    
- **Real-World Examples:** `Comparator`, Payment gateways.
    

Java

```
interface PaymentStrategy {
    void pay(int amount);
}

class UpiPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("UPI " + amount);
    }
}

class PaymentService {
    private PaymentStrategy strategy;

    PaymentService(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    void pay(int amount) {
        strategy.pay(amount);
    }
}

public class Main {
    public static void main(String[] args) {
        PaymentService service = new PaymentService(new UpiPayment());
        service.pay(1000);
    }
}
```

## 10. Observer

- **GoF Definition:** Notify dependent objects automatically.
    
- **Plain Definition:** Publish-Subscribe mechanism.
    
- **Problem It Solves:** Event notification.
    
- **Real-World Examples:** Spring Events, UI Event Listeners.
    

Java

```
interface Observer {
    void update(int price);
}

class Display implements Observer {
    public void update(int price) {
        System.out.println(price);
    }
}

class Stock {
    private java.util.List<Observer> observers = new java.util.ArrayList<>();

    void addObserver(Observer o) {
        observers.add(o);
    }

    void setPrice(int price) {
        for (Observer o : observers) {
            o.update(price);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Stock stock = new Stock();
        stock.addObserver(new Display());
        stock.setPrice(500);
    }
}
```

## 11. Command

- **GoF Definition:** Encapsulate a request as an object.
    
- **Plain Definition:** Turn a method call into an object.
    
- **Problem It Solves:** Undo/redo, queueing, asynchronous execution.
    
- **Real-World Examples:** `Runnable`, `ExecutorService`.
    

Java

```
interface Command {
    void execute();
}

class Light {
    void on() {
        System.out.println("Light ON");
    }
}

class LightOnCommand implements Command {
    private Light light;

    LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.on();
    }
}

public class Main {
    public static void main(String[] args) {
        Command command = new LightOnCommand(new Light());
        command.execute();
    }
}
```

## 12. State

- **GoF Definition:** Allow an object to alter behavior when its internal state changes.
    
- **Plain Definition:** Object behavior changes with state.
    
- **Problem It Solves:** Large state-based `if-else` or `switch` statements.
    
- **Real-World Examples:** Order fulfillment workflows, ATM state machines.
    

Java

```
interface State {
    void publish(Document d);
}

class DraftState implements State {
    public void publish(Document d) {
        System.out.println("Published");
    }
}

class Document {
    private State state = new DraftState();

    void publish() {
        state.publish(this);
    }
}

public class Main {
    public static void main(String[] args) {
        new Document().publish();
    }
}
```

## 13. Template Method

- **GoF Definition:** Define algorithm skeleton and defer specific steps to subclasses.
    
- **Plain Definition:** Parent decides the execution flow; child customizes specific steps.
    
- **Problem It Solves:** Duplicate algorithm structure.
    
- **Real-World Examples:** `JdbcTemplate`, `HttpServlet`.
    

Java

```
abstract class DataProcessor {
    public final void execute() {
        read();
        process();
        save();
    }

    void read() {}
    abstract void process();
    void save() {}
}

class CsvProcessor extends DataProcessor {
    void process() {
        System.out.println("Processing CSV");
    }
}

public class Main {
    public static void main(String[] args) {
        new CsvProcessor().execute();
    }
}
```

## 14. Chain of Responsibility

- **GoF Definition:** Pass requests along a chain of handlers.
    
- **Plain Definition:** Multiple handlers process a request sequentially.
    
- **Problem It Solves:** Validation pipelines and request filtering.
    
- **Real-World Examples:** Spring Security filter chain, Servlet filters.
    

Java

```
abstract class Handler {
    Handler next;

    Handler setNext(Handler next) {
        this.next = next;
        return next;
    }

    void handle() {
        if (next != null) {
            next.handle();
        }
    }
}

class AuthHandler extends Handler {
    void handle() {
        System.out.println("Authentication");
        super.handle();
    }
}

class LoggingHandler extends Handler {
    void handle() {
        System.out.println("Logging");
        super.handle();
    }
}

public class Main {
    public static void main(String[] args) {
        Handler auth = new AuthHandler();
        Handler log = new LoggingHandler();

        auth.setNext(log);
        auth.handle();
    }
}
```

---
---
## 15. Bridge

- **GoF Definition:** Decouple an abstraction from its implementation so that the two can vary independently.
    
- **Plain Definition:** Split a large class into two separate hierarchies—Abstraction and Implementation—so they can be developed independently.
    
- **Problem It Solves:** Prevents class explosion when combining multiple dimensions of variations (e.g., 2 types of remotes $\times$ 2 types of TVs = 4 subclasses).
    
- **Real-World Examples:** JDBC (`DriverManager` & `Driver` implementations), SLF4J logging abstractions.
    

Java

```
// Implementation hierarchy
interface Device {
    void turnOn();
}

class TV implements Device {
    public void turnOn() {
        System.out.println("TV On");
    }
}

// Abstraction hierarchy
abstract class Remote {
    protected Device device;

    Remote(Device device) {
        this.device = device;
    }

    abstract void togglePower();
}

class BasicRemote extends Remote {
    BasicRemote(Device device) {
        super(device);
    }

    void togglePower() {
        device.turnOn();
    }
}

public class Main {
    public static void main(String[] args) {
        Remote remote = new BasicRemote(new TV());
        remote.togglePower();
    }
}
```

## 16. Composite

- **GoF Definition:** Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.
    
- **Plain Definition:** Treat individual leaf objects and groups of objects (containers) the exact same way.
    
- **Problem It Solves:** Eliminates conditional logic needed to distinguish single objects from complex nested structures.
    
- **Real-World Examples:** File System (Files vs. Folders containing Files/Folders), Swing UI components (`JPanel` containing `JButton`).
    

Java

```
import java.util.ArrayList;
import java.util.List;

interface FileSystemComponent {
    void showDetails();
}

class FileItem implements FileSystemComponent {
    private String name;

    FileItem(String name) {
        this.name = name;
    }

    public void showDetails() {
        System.out.println("File: " + name);
    }
}

class Directory implements FileSystemComponent {
    private List<FileSystemComponent> components = new ArrayList<>();

    void add(FileSystemComponent c) {
        components.add(c);
    }

    public void showDetails() {
        for (FileSystemComponent c : components) {
            c.showDetails();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Directory root = new Directory();
        root.add(new FileItem("doc.txt"));
        root.showDetails();
    }
}
```

## 17. Flyweight

- **GoF Definition:** Use sharing to support large numbers of fine-grained objects efficiently.
    
- **Plain Definition:** Share common parts of state across thousands of objects to drastically cut down memory usage.
    
- **Problem It Solves:** High RAM consumption caused by creating huge numbers of repetitive instances.
    
- **Real-World Examples:** `Integer.valueOf()` caching (-128 to 127), Java String Pool, Game engine rendering systems (e.g., trees in a forest).
    

Java

```
import java.util.HashMap;
import java.util.Map;

class TreeType {
    private String name; // Shared Intrinsic State

    TreeType(String name) {
        this.name = name;
    }

    void draw(int x, int y) { // Extrinsic State passed at runtime
        System.out.println("Drawing " + name + " at " + x + "," + y);
    }
}

class TreeFactory {
    private static final Map<String, TreeType> types = new HashMap<>();

    public static TreeType getTreeType(String name) {
        return types.computeIfAbsent(name, TreeType::new);
    }
}

public class Main {
    public static void main(String[] args) {
        TreeType oak = TreeFactory.getTreeType("Oak");
        oak.draw(10, 20);
        oak.draw(15, 25);
    }
}
```

## 18. Proxy

- **GoF Definition:** Provide a surrogate or placeholder for another object to control access to it.
    
- **Plain Definition:** Place an intermediary in front of a target object to intercept, check, cache, or defer operations.
    
- **Problem It Solves:** Need for lazy initialization, access control, logging, or remote calls without altering the real class code.
    
- **Real-World Examples:** Spring AOP `@Transactional` proxies, Hibernate lazy-loading proxies, Java `java.lang.reflect.Proxy`.
    

Java

```
interface Image {
    void display();
}

class RealImage implements Image {
    RealImage() {
        System.out.println("Loading heavy image from disk...");
    }

    public void display() {
        System.out.println("Displaying image");
    }
}

class ProxyImage implements Image {
    private RealImage realImage;

    public void display() {
        if (realImage == null) { // Lazy initialization
            realImage = new RealImage();
        }
        realImage.display();
    }
}

public class Main {
    public static void main(String[] args) {
        Image image = new ProxyImage();
        image.display(); // Loads and displays on first call
    }
}
```

## 19. Interpreter

- **GoF Definition:** Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.
    
- **Plain Definition:** Build a mini-rule engine or language parser by representing grammar rules as object structures.
    
- **Problem It Solves:** Evaluating complex domain-specific sentences, expressions, or logical rules without hardcoding endless parsing loops.
    
- **Real-World Examples:** `java.util.regex.Pattern`, Spring Expression Language (SpEL), SQL parsers.
    

Java

```
interface Expression {
    boolean interpret(String context);
}

class TerminalExpression implements Expression {
    private String data;

    TerminalExpression(String data) {
        this.data = data;
    }

    public boolean interpret(String context) {
        return context.contains(data);
    }
}

class OrExpression implements Expression {
    private Expression expr1;
    private Expression expr2;

    OrExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }

    public boolean interpret(String context) {
        return expr1.interpret(context) || expr2.interpret(context);
    }
}

public class Main {
    public static void main(String[] args) {
        Expression isJava = new TerminalExpression("Java");
        Expression isPython = new TerminalExpression("Python");
        Expression isProgrammer = new OrExpression(isJava, isPython);

        System.out.println(isProgrammer.interpret("I know Java"));
    }
}
```

## 20. Iterator

- **GoF Definition:** Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.
    
- **Plain Definition:** Traverses through a collection without exposing whether it uses an array, tree, or linked list under the hood.
    
- **Problem It Solves:** Exposing data structure internals to client code during traversal.
    
- **Real-World Examples:** `java.util.Iterator`, Java enhanced `for` loops.
    

Java

```
interface Iterator<T> {
    boolean hasNext();
    T next();
}

class NameRepository {
    private String[] names = {"Vinay", "Alex", "John"};

    public Iterator<String> getIterator() {
        return new NameIterator();
    }

    private class NameIterator implements Iterator<String> {
        int index = 0;

        public boolean hasNext() {
            return index < names.length;
        }

        public String next() {
            return hasNext() ? names[index++] : null;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        NameRepository repo = new NameRepository();
        Iterator<String> iter = repo.getIterator();
        while (iter.hasNext()) {
            System.out.println(iter.next());
        }
    }
}
```

## 21. Mediator

- **GoF Definition:** Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly.
    
- **Plain Definition:** An "air traffic control tower" where objects communicate through a central coordinator rather than talking directly to each other.
    
- **Problem It Solves:** Tightly coupled "spiderweb" dependencies where changing one component breaks many others.
    
- **Real-World Examples:** Chat room servers, Spring MVC `DispatcherServlet`, Java `ExecutorService`.
    

Java

```
class ChatMediator {
    public static void showMessage(User user, String message) {
        System.out.println(user.getName() + ": " + message);
    }
}

class User {
    private String name;

    User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void send(String message) {
        ChatMediator.showMessage(this, message);
    }
}

public class Main {
    public static void main(String[] args) {
        User user1 = new User("Vinay");
        User user2 = new User("Rahul");

        user1.send("Hello!");
        user2.send("Hi Vinay!");
    }
}
```

## 22. Memento

- **GoF Definition:** Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.
    
- **Plain Definition:** Take a snapshot of an object's state so you can restore or "undo" it later.
    
- **Problem It Solves:** Implementing undo/redo functionality without breaching private field encapsulation.
    
- **Real-World Examples:** Text editor Undo (`Ctrl+Z`), database transactions / savepoints.
    

Java

```
class Memento {
    private final String state;

    Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}

class TextEditor {
    private String text = "";

    public void write(String newText) {
        text += newText;
    }

    public Memento save() {
        return new Memento(text);
    }

    public void restore(Memento memento) {
        text = memento.getState();
    }

    public String getText() {
        return text;
    }
}

public class Main {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        editor.write("Hello ");
        Memento snapshot = editor.save();

        editor.write("World!");
        editor.restore(snapshot); // Reverts back to "Hello "

        System.out.println(editor.getText());
    }
}
```

## 23. Visitor

- **GoF Definition:** Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.
    
- **Plain Definition:** Attach new behaviors/operations to an existing object structure without modifying the classes themselves.
    
- **Problem It Solves:** Violating the Open-Closed Principle when adding new behaviors to an established class hierarchy.
    
- **Real-World Examples:** Abstract Syntax Tree (AST) visitors in compiler design, `java.nio.file.FileVisitor`.
    

Java

```
interface Visitor {
    void visit(Book book);
}

interface Item {
    void accept(Visitor visitor);
}

class Book implements Item {
    private int price = 100;

    public int getPrice() {
        return price;
    }

    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

class DiscountVisitor implements Visitor {
    public void visit(Book book) {
        System.out.println("Discounted Price: " + (book.getPrice() - 10));
    }
}

public class Main {
    public static void main(String[] args) {
        Item item = new Book();
        Visitor visitor = new DiscountVisitor();
        item.accept(visitor);
    }
}
```