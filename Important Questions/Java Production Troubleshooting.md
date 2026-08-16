## 2. You see high CPU usage but low traffic. What could be the reason?

Low traffic but high CPU usually means the CPU is being consumed by something other than normal request processing.

Check:

- Infinite loops or busy-waiting
- Expensive algorithms such as accidental `O(n²)` processing
- Excessive retries
- Excessive garbage collection
- Background jobs or scheduled tasks
- CPU-intensive serialization/deserialization
- Excessive logging
- Thread contention or application bugs

### How to investigate

First identify which process/threads are consuming CPU.

Use:

```bash
top
```

or Windows/Linux monitoring tools.

Then take a thread dump:

```bash
jstack <pid>
```

`jstack` tells you what the JVM's threads are doing at a particular moment. For high CPU, combine it with OS CPU/thread information to identify the problematic thread and then inspect its stack trace.

and correlate the high-CPU thread with the Java code.

For deeper investigation, use **JFR** or a CPU profiler such as async-profiler.

The key is to identify **what is actually consuming the CPU before changing configuration**.

---

## 3. A thread is stuck in `BLOCKED` state. How will you identify and fix it?

`BLOCKED` generally means a thread is waiting to acquire a monitor lock held by another thread.

Take a thread dump:

```bash
jstack <pid>
```

or use JFR/VisualVM.

Look for a pattern like:

```text
Thread A → waiting for Lock 1
Thread B → holding Lock 1
```

Then inspect the synchronized code.
Example
```
"Thread-B" #20
   java.lang.Thread.State: BLOCKED (on object monitor)
        at MyService.process(MyService.java:42)
        - waiting to lock <0x0000000123456789>
```

Possible fixes:

- Reduce the scope of synchronization.
- Avoid unnecessary locking.
- Reduce lock contention.
- Use more appropriate concurrent data structures.
- Use `ReentrantLock` where it provides a better locking strategy.
- Redesign shared mutable state where possible.

Do not simply add more threads; the problem is usually the lock being held for too long or too many threads competing for it.

---

## 4. Your application slows down after running for a few hours. What will you check?

This suggests a problem that **builds up over time**.

Check:

- Memory leaks
- Growing collections/caches
- Increasing GC frequency
- Thread leaks
- Database connection leaks
- Connection pool exhaustion
- File/socket/resource leaks
- Growing queues
- Increasing database query time
- External service latency
- Log/storage issues

Compare metrics from startup with metrics after several hours.

Useful things to collect include:

- Heap usage
- GC activity
- Thread count
- CPU
- Connection pool usage
- Request latency

Heap dumps and thread dumps taken at different points in time can help identify what is accumulating.

---

## 5. You are facing frequent GC pauses. How will you optimize it?

First determine **why GC is happening frequently** rather than immediately changing the GC settings.

Check:

- Heap size
- Allocation rate
- Young/old generation behavior
- GC frequency
- GC pause duration
- GC logs
- Object allocation patterns
- Memory leaks

Possible causes include:

- Excessive temporary object creation
- Large collections
- Large caches
- Memory leaks
- Heap that is too small for the application's workload
- High allocation rate

Possible improvements:

- Reduce unnecessary object creation.
- Fix memory leaks.
- Avoid unnecessarily large objects/collections.
- Tune `-Xms` and `-Xmx` based on observed behavior.
- Use an appropriate garbage collector, such as G1GC, when suitable.
- Tune only after measuring the actual GC behavior.

The important point is:

> GC itself is not a problem. Excessive frequency or long pauses are the problem.

---

## 6. A `HashMap` is causing performance issues under heavy load. Why?

A normal `HashMap` is **not thread-safe**.

If multiple threads are modifying it concurrently, race conditions and incorrect behavior can occur.

For concurrent access, consider:

```java
ConcurrentHashMap<K, V>
```

Also investigate other possible causes:

- Poor `hashCode()` implementation
- Excessive hash collisions
- Frequent resizing
- Very large map
- Mutable keys
- Excessive synchronization around the map

For example, if the application is doing:

```java
map.get(key);
map.put(key, value);
```

from many threads, blindly adding `synchronized` around the entire map can create significant contention.

Choose the data structure based on the actual concurrency pattern.

---

## 7. Multiple threads are updating shared data incorrectly. How will you fix it?

This is typically a **race condition**.

For example:

```java
count++;
```

is not atomic. It consists conceptually of:

```text
read count
   ↓
increment
   ↓
write count
```

Two threads can interleave these operations.

Depending on the situation, use:

### Atomic classes

```java
AtomicInteger count = new AtomicInteger();

count.incrementAndGet();
```

### Synchronization

```java
synchronized
```

### Locks

```java
ReentrantLock
```

### Concurrent collections

```java
ConcurrentHashMap
```

Also consider making data immutable or reducing shared mutable state.

The correct solution depends on what is being shared and what consistency guarantees are required.

---

## 8. Your API works fine locally but fails in production. What will you investigate?

Compare the environments systematically.

Check:

- Environment variables
- Application configuration
- Database connectivity
- Database credentials
- Secrets
- Network connectivity
- DNS
- Firewall/security groups
- External service URLs
- Authentication/JWT configuration
- CORS
- File paths
- File permissions
- Java/Spring versions
- Timeouts
- Container configuration
- Production logs

First look at the actual production exception/stack trace.

Then reproduce the same request and trace it through:

```text
Client
  ↓
Load Balancer/API Gateway
  ↓
Application
  ↓
Database / External Services
```

Do not assume the code itself is different just because it works locally. Environment/configuration differences are common causes.

---

## 9. You suspect a memory leak. How will you confirm it?

A memory leak should be **confirmed with evidence**, not just assumed from high memory usage.

First observe heap behavior over time.

If memory grows continuously and does not return after GC, investigate further.

Take heap dumps at different times:

```text
T0 → heap dump
T1 → heap dump
T2 → heap dump
```

Analyze them using a tool such as **Eclipse MAT**.

Look for:

- Objects whose count keeps increasing
- Large retained heap
- Static collections
- Unbounded caches
- `ThreadLocal` references
- Listeners that were never removed
- ClassLoader leaks
- Objects that should have been released but remain reachable

The key question is:

> **Why are these objects still reachable?**

A heap being full does not automatically mean there is a leak. If GC successfully removes unused objects, the behavior may be normal.

---

## 10. A service becomes unresponsive randomly. What could be happening?

Look at the complete request path.

Possible causes:

- Thread pool exhaustion
- Database connection pool exhaustion
- Deadlock
- Long-running database queries
- Slow external APIs
- Network problems
- GC pauses
- CPU saturation
- Memory pressure
- Queue/backlog growth
- Circuit breaker/retry problems

During the incident, collect:

- Thread dump
- JVM metrics
- GC metrics
- CPU
- Memory
- Thread pool metrics
- Database pool metrics
- Request latency
- External dependency latency

For example, if all request threads are waiting for database connections, increasing the HTTP thread pool won't solve the problem.

---

## 11. You see a deadlock in your system. How will you detect and resolve it?

A deadlock occurs when threads wait for locks held by each other.

Example:

```text
Thread A
holds Lock 1
   ↓
waits for Lock 2

Thread B
holds Lock 2
   ↓
waits for Lock 1
```

### Detect it

Take a thread dump:

```bash
jstack <pid>
```

The JVM can often identify monitor deadlocks in the thread dump.

JFR can also help investigate lock contention.

### Resolve it

A common solution is to enforce a **consistent lock acquisition order**.

For example:

```text
Always acquire Lock 1
        ↓
then Lock 2
```

Do not have another code path acquire:

```text
Lock 2
   ↓
Lock 1
```

Other solutions include:

- Reduce lock scope.
- Avoid unnecessary nested locks.
- Use `tryLock()` where appropriate.
- Redesign shared-state handling.

---

## 12. Your logs show inconsistent behavior across requests. Why?

One common reason is that you cannot properly correlate logs belonging to the same request.

Use a **correlation/request ID**:

```text
Request ID: abc123
```

Then include it throughout the request:

```text
API Gateway
    ↓
Controller
    ↓
Service
    ↓
Repository
    ↓
External Service
```

In distributed systems, use distributed tracing with:

- Trace ID
- Span ID

Other possible causes include:

- Shared mutable state
- Race conditions
- Async processing
- Thread-local/context problems
- Different configuration across instances
- Different application instances behaving differently

---

## 13. Your application crashes without any clear error. How will you debug it?

First determine **what actually terminated the process**.

Check:

- Application logs
- JVM logs
- Container logs
- Kubernetes/ECS events
- Process exit code
- OS logs
- OOM killer events
- Health checks
- Startup/shutdown logs

In Kubernetes, for example:

```bash
kubectl describe pod <pod>
kubectl logs <pod>
```

Look for conditions such as:

```text
OOMKilled
CrashLoopBackOff
SIGTERM
SIGKILL
```

Also check whether the JVM itself crashed and produced an error log such as a fatal error file.

The important first step is determining whether the application crashed because of a Java exception, JVM failure, OS termination, resource limit, or infrastructure action.

---

## 14. A database call is slowing down your Java service. How will you optimize it?

First determine where the time is actually being spent:

```text
Java service
    ↓
Connection acquisition
    ↓
Database query
    ↓
Database processing
    ↓
Result transfer
    ↓
Java processing
```

Investigate:

- SQL execution plan
- Missing indexes
- Full table scans
- Expensive joins
- Large result sets
- N+1 queries
- Lock contention
- Connection pool exhaustion
- Database CPU/memory
- Network latency

For a slow SQL query, inspect the actual execution plan rather than blindly adding indexes.

Also check whether the application is requesting more data than it needs.

Possible improvements:

- Add appropriate indexes.
- Optimize SQL.
- Reduce returned columns/rows.
- Fix N+1 queries.
- Use pagination.
- Tune connection pooling.
- Cache appropriate read-heavy data.
- Optimize transactions.

---

## 15. Your thread pool gets exhausted under load. What's your approach?

First find out **why the threads are not completing**.

Common causes:

- Slow database operations
- Slow external APIs
- Blocking I/O
- Deadlocks
- Long-running tasks
- Infinite loops
- Too many tasks being submitted

Check:

```text
Active threads
Queue size
Pool size
Task execution time
Rejected tasks
```

Take a thread dump during the problem to see what the threads are doing.

Do not immediately increase the thread pool size.

For example:

```text
500 application threads
       ↓
only 50 DB connections
       ↓
many threads wait for DB connections
       ↓
more threads added
       ↓
more contention
       ↓
system gets worse
```

Thread pool size should be designed around the workload and downstream capacity.

---

## 16. You need to handle high concurrency safely. What will you use?

It depends on what is being shared.

Common approaches:

- Immutable objects
- Stateless services
- `AtomicInteger`, `AtomicLong`, etc.
- `synchronized`
- `ReentrantLock`
- `ConcurrentHashMap`
- Concurrent queues
- Database transactions/locking
- Idempotency
- Proper thread/connection pool limits

For example:

```java
AtomicInteger counter = new AtomicInteger();

counter.incrementAndGet();
```

For distributed systems, in-memory synchronization is not enough because multiple application instances may be running.

In those cases, concurrency control may need to involve:

- Database constraints/locks
- Distributed locks
- Idempotency keys
- Message ordering/partitioning
- Transactional mechanisms

---

## 17. Your system processes duplicate requests. How will you handle it?

Make the operation **idempotent** where possible.

A common approach is an idempotency key:

```text
Idempotency-Key: abc123
```

Store the processing status/result associated with that key.

If the same request arrives again:

```text
abc123 → already processed
```

return the existing result rather than performing the operation again.

Database constraints can also protect against duplicates.

For example:

```sql
UNIQUE(request_id)
```

For messaging systems such as Kafka, design the consumer and database operations so that duplicate messages do not produce duplicate business effects.

The key idea is:

> The same operation can safely be performed/retried without producing an unintended second business effect.

---

## 18. A cache is giving stale data. How will you fix it?

First determine the required **consistency level**.

Possible approaches:

- Reduce TTL
- Explicit cache invalidation
- Cache-aside pattern
- Write-through cache
- Versioned cache entries
- Event-driven cache invalidation

For example:

```text
UPDATE database
      ↓
Invalidate Redis key
      ↓
Next GET
      ↓
Cache miss
      ↓
Read database
      ↓
Put fresh value in cache
```

In distributed systems, cache invalidation needs to account for multiple application instances.

Do not simply reduce the TTL without understanding the consistency requirement. A short TTL can reduce staleness but increase database load.

---

## 19. Your application is not scaling even after adding instances. Why?

Adding application instances only helps if the bottleneck is actually in the application instances.

Check for bottlenecks such as:

- Database capacity
- Database connection pool
- Shared locks
- Session state stored locally
- Local/in-memory caches
- Synchronized code
- External API limits
- Queue bottlenecks
- Load balancer configuration
- CPU/memory limits

For example:

```text
Instance 1 ─┐
Instance 2 ─┼──→ Database
Instance 3 ─┘
```

If the database is already at maximum capacity, adding more application instances can make the situation worse.

Also check whether the application is actually **stateless**. Local session state or local caches can prevent effective horizontal scaling.

---

## 20. You need to trace a request across multiple layers. How will you do it?

Use **distributed tracing**.

A request can travel through:

```text
Client
  ↓
API Gateway
  ↓
Service A
  ↓
Service B
  ↓
Database
```

A distributed tracing system assigns a **Trace ID** to the request and creates **Spans** for individual operations.

For example:

```text
Trace ID: abc123

API Gateway       20 ms
Service A        100 ms
Service B        600 ms
Database         550 ms
```

This tells us where the time is actually being spent.

Common technologies/tools include:

- OpenTelemetry
- Jaeger
- Zipkin
- AWS X-Ray
- Datadog
- New Relic

The key benefit is that instead of saying:

> "The API is slow."

you can determine:

> "Service B is taking 600 ms, and most of that is a database query."

---

# Quick Troubleshooting Toolkit

| Problem                       | Useful tool/data                     |
| ----------------------------- | ------------------------------------ |
| OutOfMemoryError              | Heap dump + MAT                      |
| Memory leak                   | Heap dumps + MAT                     |
| High CPU                      | JFR / profiler / thread dump         |
| BLOCKED thread                | Thread dump                          |
| Deadlock                      | Thread dump / JFR                    |
| Frequent GC                   | JFR + GC logs                        |
| Thread pool exhaustion        | Thread dump + pool metrics           |
| Slow DB                       | Execution plan + DB metrics          |
| Unresponsive service          | Thread dump + JVM/dependency metrics |
| Inconsistent distributed logs | Correlation ID + distributed tracing |
| Request tracing               | OpenTelemetry / distributed tracing  |
