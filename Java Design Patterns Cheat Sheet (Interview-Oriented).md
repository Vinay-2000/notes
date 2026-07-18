> Each pattern includes: - GoF Definition - Plain Definition - Problem

> it solves - Real-world examples - Complete runnable core example

> (Product + Implementation + Client)

---

# 1. Singleton

## GoF Definition

Ensure a class has only one instance and provide a global point of

access to it.

## Plain Definition

Only one object of the class should ever exist.

## Problem it solves

- Shared resources

- Global configuration

- Avoid multiple instances

## Real-world examples

- Spring Singleton Beans

- Logger

- Cache Manager

## Complete Example

```java

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

---

# 2. Factory Method

## GoF Definition

Define an interface for creating an object, but let subclasses decide

which class to instantiate.

## Plain Definition

Delegate object creation instead of using `new` everywhere.

## Problem it solves

- Tight coupling

- Centralized creation

## Real-world examples

- BeanFactory

- Calendar.getInstance()

## Complete Example

```java

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

---

# 3. Abstract Factory

## GoF Definition

Provide an interface for creating families of related objects.

## Plain Definition

Create matching groups of related objects.

## Problem it solves

Ensures compatible product families.

## Real-world examples

- UI Themes

- Spring ApplicationContext

## Complete Example

```java

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

---

# 4. Builder

## GoF Definition

Separate construction of a complex object from its representation.

## Plain Definition

Build objects step by step.

## Problem it solves

Too many constructor parameters.

## Real-world examples

- Lombok @Builder

- HttpRequest.newBuilder()

## Complete Example

```java

class User {



    private String name;

    private int age;

    private String email;



    private User(Builder b){

        this.name=b.name;

        this.age=b.age;

        this.email=b.email;

    }



    static class Builder{

        private String name;

        private int age;

        private String email;



        Builder(String name,int age){

            this.name=name;

            this.age=age;

        }



        Builder email(String email){

            this.email=email;

            return this;

        }



        User build(){

            return new User(this);

        }

    }

}



public class Main{

    public static void main(String[] args){

        User user=new User.Builder("Vinay",28)

                .email("vinay@email.com")

                .build();

    }

}

```

---

# 5. Prototype

## GoF Definition

Create objects by copying an existing instance.

## Plain Definition

Clone instead of creating from scratch.

## Problem it solves

Expensive object creation.

## Real-world examples

- Spring prototype scope

- Document templates

## Complete Example

```java

class User implements Cloneable{



    String name;



    User(String name){

        this.name=name;

    }



    public User clone() throws CloneNotSupportedException{

        return (User)super.clone();

    }

}



public class Main{

    public static void main(String[] args)throws Exception{

        User u1=new User("Vinay");

        User u2=u1.clone();



        System.out.println(u1.name);

        System.out.println(u2.name);

    }

}

```

---

# 6. Adapter

## GoF Definition

Convert one interface into another expected by clients.

## Plain Definition

Translator between incompatible interfaces.

## Problem it solves

Integrating third-party or legacy code.

## Real-world examples

- InputStreamReader

- HandlerAdapter

## Complete Example

```java

interface PaymentProcessor{

    void pay();

}



class PaypalGateway{

    void makePayment(){

        System.out.println("Paypal Payment");

    }

}



class PaypalAdapter implements PaymentProcessor{



    private PaypalGateway gateway=new PaypalGateway();



    public void pay(){

        gateway.makePayment();

    }

}



public class Main{

    public static void main(String[] args){

        PaymentProcessor p=new PaypalAdapter();

        p.pay();

    }

}

```

---

# 7. Decorator

## GoF Definition

Attach additional responsibilities dynamically.

## Plain Definition

Wrap an object and enhance it.

## Problem it solves

Avoid subclass explosion.

## Real-world examples

- BufferedInputStream

- Spring AOP

## Complete Example

```java

interface Coffee{

    int cost();

}



class BasicCoffee implements Coffee{

    public int cost(){

        return 100;

    }

}



class MilkDecorator implements Coffee{



    private Coffee coffee;



    MilkDecorator(Coffee coffee){

        this.coffee=coffee;

    }



    public int cost(){

        return coffee.cost()+20;

    }

}



public class Main{

    public static void main(String[] args){



        Coffee coffee=new BasicCoffee();

        coffee=new MilkDecorator(coffee);



        System.out.println(coffee.cost());

    }

}

```

---

# 8. Facade

## GoF Definition

Provide a unified interface to a subsystem.

## Plain Definition

Hide complexity behind one simple method.

## Problem it solves

Complex APIs.

## Real-world examples

- JdbcTemplate

- Service layer

## Complete Example

```java

class DVD{

    void play(){

        System.out.println("Playing Movie");

    }

}



class Sound{

    void on(){

        System.out.println("Sound ON");

    }

}



class HomeTheater{



    private DVD dvd=new DVD();

    private Sound sound=new Sound();



    void watchMovie(){

        sound.on();

        dvd.play();

    }

}



public class Main{

    public static void main(String[] args){

        new HomeTheater().watchMovie();

    }

}

```

---

# 9. Strategy

## GoF Definition

Define interchangeable algorithms.

## Plain Definition

Swap behavior at runtime.

## Problem it solves

Large if-else blocks.

## Real-world examples

- Comparator

- Payment gateways

## Complete Example

```java

interface PaymentStrategy{

    void pay(int amount);

}



class UpiPayment implements PaymentStrategy{

    public void pay(int amount){

        System.out.println("UPI "+amount);

    }

}



class PaymentService{



    private PaymentStrategy strategy;



    PaymentService(PaymentStrategy strategy){

        this.strategy=strategy;

    }



    void pay(int amount){

        strategy.pay(amount);

    }

}



public class Main{

    public static void main(String[] args){

        PaymentService service=new PaymentService(new UpiPayment());

        service.pay(1000);

    }

}

```

---

# 10. Observer

## GoF Definition

Notify dependent objects automatically.

## Plain Definition

Publish-Subscribe.

## Problem it solves

Event notification.

## Real-world examples

- Spring Events

- UI Listeners

## Complete Example

```java

interface Observer{

    void update(int price);

}



class Display implements Observer{

    public void update(int price){

        System.out.println(price);

    }

}



class Stock{



    private java.util.List<Observer> observers=new java.util.ArrayList<>();



    void addObserver(Observer o){

        observers.add(o);

    }



    void setPrice(int price){

        for(Observer o:observers){

            o.update(price);

        }

    }

}



public class Main{

    public static void main(String[] args){

        Stock stock=new Stock();

        stock.addObserver(new Display());

        stock.setPrice(500);

    }

}

```

---

# 11. Command

## GoF Definition

Encapsulate a request as an object.

## Plain Definition

Turn a method call into an object.

## Problem it solves

Undo, queueing, async execution.

## Real-world examples

- Runnable

- ExecutorService

## Complete Example

```java

interface Command{

    void execute();

}



class Light{

    void on(){

        System.out.println("Light ON");

    }

}



class LightOnCommand implements Command{



    private Light light;



    LightOnCommand(Light light){

        this.light=light;

    }



    public void execute(){

        light.on();

    }

}



public class Main{

    public static void main(String[] args){

        Command command=new LightOnCommand(new Light());

        command.execute();

    }

}

```

---

# 12. State

## GoF Definition

Allow an object to alter behavior when its internal state changes.

## Plain Definition

Behavior changes with state.

## Problem it solves

Large state-based if-else blocks.

## Real-world examples

- Order workflow

- ATM

## Complete Example

```java

interface State{

    void publish(Document d);

}



class DraftState implements State{

    public void publish(Document d){

        System.out.println("Published");

    }

}



class Document{



    private State state=new DraftState();



    void publish(){

        state.publish(this);

    }

}



public class Main{

    public static void main(String[] args){

        new Document().publish();

    }

}

```

---

# 13. Template Method

## GoF Definition

Define algorithm skeleton and defer specific steps to subclasses.

## Plain Definition

Parent decides flow, child customizes steps.

## Problem it solves

Duplicate algorithm structure.

## Real-world examples

- JdbcTemplate

- HttpServlet

## Complete Example

```java

abstract class DataProcessor{



    public final void execute(){

        read();

        process();

        save();

    }



    void read(){}



    abstract void process();



    void save(){}

}



class CsvProcessor extends DataProcessor{



    void process(){

        System.out.println("Processing CSV");

    }

}



public class Main{

    public static void main(String[] args){

        new CsvProcessor().execute();

    }

}

```

---

# 14. Chain of Responsibility

## GoF Definition

Pass requests along a chain of handlers.

## Plain Definition

Multiple handlers process a request one after another.

## Problem it solves

Validation pipelines and filters.

## Real-world examples

- Spring Security filters

- Servlet Filters

## Complete Example

```java

abstract class Handler{



    Handler next;



    Handler setNext(Handler next){

        this.next=next;

        return next;

    }



    void handle(){

        if(next!=null){

            next.handle();

        }

    }

}



class AuthHandler extends Handler{



    void handle(){

        System.out.println("Authentication");

        super.handle();

    }

}



class LoggingHandler extends Handler{



    void handle(){

        System.out.println("Logging");

        super.handle();

    }

}



public class Main{

    public static void main(String[] args){



        Handler auth=new AuthHandler();

        Handler log=new LoggingHandler();



        auth.setNext(log);



        auth.handle();

    }

}

```
