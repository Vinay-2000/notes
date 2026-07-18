### ⚡ TL;DR (Executive Summary)

* These exercises cover common concurrency interview patterns: producer-consumer coordination, ordered odd/even printing, multi-thread sequencing, singleton safety, and deliberate deadlock creation.
* Focus on the coordination primitive each solution uses—monitors, `wait()`/`notify()`, locks, conditions, or executors—rather than memorizing syntax alone.
* Correct solutions must preserve ordering, visibility, mutual exclusion, and termination without busy waiting.
* For deadlocks, recognize the four required conditions and learn how consistent lock ordering prevents circular wait.

---

## Producer Consumer

Best - https://www.geeksforgeeks.org/java/producer-consumer-solution-using-threads-java/

```
import java.util.LinkedList;
import java.util.Queue;
public class ProducerConsumer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int CAPACITY = 5;
    public void produce(int value) throws InterruptedException {
        synchronized (queue) {
            while (queue.size() == CAPACITY) {
                queue.wait();
            }
            queue.offer(value);
            System.out.println("Produced: " + value);
            queue.notifyAll();
        }
    }
    public int consume() throws InterruptedException {
        synchronized (queue) {
            while (queue.isEmpty()) {
                queue.wait();
            }
            int value = queue.poll();
            System.out.println("Consumed: " + value);
            queue.notifyAll();
            return value;
        }
    }
    public static void main(String[] args) {
        ProducerConsumer pc = new ProducerConsumer();
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 20; i++) {
                    pc.produce(i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 20; i++) {
                    pc.consume();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        producer.start();
        consumer.start();
    }
}
```

```
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>();
Runnable producer = () -> {
    System.out.println("Producer Started");
    for (int i = 1; i <= 20; i++) {
        try {
            System.out.println("Producer adding: " + i);
            queue.put(i);
            Thread.sleep(200);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
Using Blocking Queue
Runnable consumer = () -> {
    int val = 0;
    try {
        while (val != 20) {
            val = queue.take();
            System.out.println("Consumer receiver: " + val);
        }
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}
Thread a = new Thread(producer, "Producer");
Thread b = new Thread(consumer, "Consumer");
a.start();
b.start();
a.join();
b.join();
```

```
Queue<Integer> queue = new LinkedList<>();
Runnable producer = () -> {
    System.out.println("Producer Started");
    for (int i = 1; i <= 20; i++) {
        queue.add(i);
        if (queue.size() >= 5) {
            synchronized (queue) {
                try {
                    System.out.println("Notifying...");
                    queue.notify();
                    queue.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
    System.out.println("Producer Down");
};
Runnable consumer = () -> {
    int val = 0;
    while (queue.isEmpty()) {
        try {
            synchronized (queue) {
                System.out.println("Consumer Waiting... ");
                queue.wait();
                while (!queue.isEmpty()) {
                    val = queue.poll();
                    System.out.println("Consumer receiver: " + val);
                }
                System.out.println("Notifying Producer");
                queue.notify();
                if (val == 20) {
                    break;
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
    System.out.println("Consumer Down");
};
Thread a = new Thread(producer, "Producer");
Thread b = new Thread(consumer, "Consumer");
a.start();
b.start();
a.join();
b.join();
System.out.println("End");
```

## Odd Even

Best - https://www.geeksforgeeks.org/java/print-even-and-odd-numbers-in-increasing-order-using-two-threads-in-java/

Other way

```
private int count = 1;
private final Object lock = new Object();
public void printOdd() {
    synchronized(lock){
        while(count <= 100){
            while(count % 2 == 0){
                lock.wait();
            }
            System.out.println(count++);
            lock.notifyAll();
        }
    }
}
```

## Print 1 to 100 using 3 threads

```
int count = 1;
public synchronized void print1() {
    while (count <= 100) {
        while (count % 3 != 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println("Thread Interrupted");
            }
        }
        if (count > 100) {
            notifyAll();
            return;
        }
        System.out.println(Thread.currentThread().getName() + " printing: " + count);
        count++;
        notifyAll();
    }
}
public synchronized void print2() {
    while (count <= 100) {
        while (count % 3 != 1) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println("Thread Interrupted");
            }
        }
        if (count > 100) {
            notifyAll();
            return;
        }
        System.out.println(Thread.currentThread().getName() + " printing: " + count);
        count++;
        notifyAll();
    }
}
public synchronized void print3() {
    while (count <= 100) {
        while (count % 3 != 2) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println("Thread Interrupted");
            }
        }
        if (count > 100) {
            notifyAll();
            return;
        }
        System.out.println(Thread.currentThread().getName() + " printing: " + count);
        count++;
        notifyAll();
    }
}
Counter count = new Counter();
Thread a = new Thread(count::print1, "A");
Thread b = new Thread(count::print2, "B");
Thread c = new Thread(count::print3, "C");
a.start();
b.start();
c.start();
a.join();
b.join();
c.join();
```

## Singleton class

```
Best production choice when lazy initialization is NOT required.
public enum Singleton {
    INSTANCE;
    public void doSomething() {
        System.out.println("Singleton working...");
    }
}
Best choice if you want lazy loading in production.
public class Singleton {
    private Singleton() {
    }
    // Loaded only when getInstance() is called
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
DCL
public class Singleton {
    private static volatile Singleton instance = null;
    private Singleton() {
    }
    public static Singleton getInstance() {
        if (instance == null) {          // First check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {  // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

## Create Deadlock

```
public class Sample {
    private final Object object1 = new Object();
    private final Object object2 = new Object();
    public void method1() throws InterruptedException {
        synchronized (object1) {
            System.out.println(Thread.currentThread().getName() + " Acquired object1 lock");
            Thread.sleep(1000);
            synchronized (object2) {
                System.out.println(Thread.currentThread().getName() + " Acquired object2 lock");
            }
        }
    }
    public void method2() throws InterruptedException {
        synchronized (object2) {
            System.out.println(Thread.currentThread().getName() + " Acquired object2 lock");
            Thread.sleep(1000);
            synchronized (object1) {
                System.out.println(Thread.currentThread().getName() + " Acquired object1 lock");
            }
        }
    }
}
```

> **What are the four Coffman conditions required for a deadlock?**

This is asked surprisingly often.

They are:

### 1. Mutual Exclusion

A resource can be held by only one thread at a time.

Example:

```
object1 lock
```

Only one owner.

---

### 2. Hold and Wait

A thread holds one resource while waiting for another.

Your example:

```
Thread 1
Holding object1
Waiting for object2
```

---

### 3. No Preemption

A lock cannot be forcibly taken away.

Only the owning thread can release it.

---

### 4. Circular Wait

```
T1
↓
Waiting for T2
↓
Waiting for T3
↓
Waiting for T1
```

Cycle formed.
