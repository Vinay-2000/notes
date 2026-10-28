
**Target profile:** Java Full Stack Developer (\~3--4 years) --- Java,
Spring Boot, React, AWS, Kafka
**How to use:** These are spoken-answer templates. Start with the direct
answer, then explain the mechanism and give a practical example. Adapt
experience-based answers to what you actually did.

> Scope: System Design / HLD and the Progress checklist are
> intentionally omitted. Answers focus on the remaining question
> sections. For implementation questions, code is kept compact and
> interview-oriented.

------------------------------------------------------------------------

# 1. Java Core & Collections

## 1. How does HashMap work internally?

`HashMap` stores key-value pairs in an array of buckets. It calculates a
hash from the key, maps that hash to a bucket, and stores a node there.
If multiple keys map to the same bucket, the bucket holds multiple
nodes, or a tree for sufficiently large collision chains. `put()`
inserts or replaces a value; `get()` finds the bucket and then uses hash
plus `equals()` to identify the key. It resizes when the number of
entries exceeds its threshold, which is usually capacity × load factor.

## 2. How does HashMap calculate the bucket index?

For a power-of-two table capacity, Java uses a spread hash and a bit
mask:

``` java
int hash = key.hashCode();
hash = hash ^ (hash >>> 16);
int index = (table.length - 1) & hash;
```

The actual implementation handles `null` keys and internal details. The
bitwise mask is efficient and uses the lower bits while spreading
higher-bit information into them.

## 3. What happens during a hash collision?

A collision occurs when different keys map to the same bucket. `HashMap`
stores both entries in that bucket. It compares hashes first and then
uses `equals()` to distinguish keys. In modern Java, a heavily populated
bucket may become a red-black tree to improve lookup performance.

## 4. What changed in Java 8? (Treeification at 8 entries + capacity \>= 64)

Java 8 introduced tree bins for heavily collided `HashMap` buckets. When
a bucket reaches the treeification threshold (commonly described as 8
entries), the implementation attempts to convert its linked list to a
red-black tree if the table capacity is at least 64. If capacity is
smaller, it resizes instead. Treeification improves worst-case lookup
from linear to logarithmic for suitable keys.

## 5. Why is the treeification threshold 8?

It is an implementation tuning choice informed by the fact that long
collision chains should be rare with well-distributed hashes. The JDK
source comments refer to a Poisson distribution: under typical
assumptions, buckets with many entries are extremely uncommon. Eight is
a practical point to consider treeification without paying tree-node
overhead for normal buckets. It is not a guarantee that every bucket of
exactly eight entries immediately becomes a tree; capacity conditions
and implementation details matter.

## 6. Explain the `hashCode()` and `equals()` contract.

If two objects are equal according to `equals()`, they **must** return
the same `hashCode()`. Unequal objects may have the same hash code.
`equals()` should be reflexive, symmetric, transitive, consistent, and
return false for null. Hash-based collections use the hash to locate a
bucket and `equals()` to find the exact key.

## 7. What happens if you override `equals()` but not `hashCode()`?

Two logically equal objects may have different identity-based hash
codes. A `HashMap` can put them in different buckets, so a lookup using
an equal object may fail. Whenever you override `equals()`, override
`hashCode()` consistently as well.

## 8. Objects "disappear" from HashMap --- why?

A common cause is mutating a key after inserting it, especially a field
used by `hashCode()` or `equals()`. The map placed the key in the bucket
derived from its old hash, but a later lookup uses the new hash and
searches another bucket. Use immutable keys, or never mutate
equality/hash fields while a key is stored in a hash-based collection.

## 9. HashMap vs ConcurrentHashMap --- when to use each?

I use `HashMap` when a map is confined to one thread or safely guarded
externally. I use `ConcurrentHashMap` when multiple threads access and
update a shared map concurrently. `ConcurrentHashMap` supports
thread-safe operations and high concurrent reads/updates, but it does
not make compound application logic automatically atomic; use methods
such as `computeIfAbsent()` or appropriate synchronization for
multi-step invariants. It also disallows null keys and values.

## 10. How does ConcurrentHashMap work internally?

In modern Java, `ConcurrentHashMap` uses a bucket array, CAS for some
updates, and synchronization at the individual-bin level when needed.
Reads are generally non-blocking and use volatile/atomic visibility
guarantees. Contended bins can be treeified. Resizing can be
cooperatively performed by multiple threads. The exact implementation
differs by Java version; Java 8 removed the old fixed-segment design.

## 11. Java 7 segment locking vs Java 8 CAS approach

Java 7 `ConcurrentHashMap` divided the map into segments, each with its
own lock, so different segments could update concurrently. Java 8
removed segments and uses CAS for certain operations and synchronized
locking on a bucket/bin for contended updates, alongside volatile fields
and cooperative resizing. This gives more granular concurrency and
avoids the fixed-segment architecture.

## 12. Does ConcurrentHashMap ever lock the entire map?

Normal operations do not acquire one global lock for the whole map.
Updates usually coordinate at bucket/bin level, and some use CAS.
Resizing may involve coordination and multiple threads, but it is not
equivalent to locking the entire map for every operation.

## 13. Fail-Fast vs "Fail-Safe" iterators

A fail-fast iterator, such as a typical `ArrayList` iterator, detects
structural modification outside the iterator and may throw
`ConcurrentModificationException`. This is best-effort bug detection,
not a thread-safety guarantee. "Fail-safe" is informal terminology often
used for iterators over snapshots or concurrent collections. For
example, `CopyOnWriteArrayList` iterates over a snapshot, while
`ConcurrentHashMap` iterators are weakly consistent.

## 14. What is a weakly consistent iterator?

It can traverse a concurrent collection while updates happen without
throwing `ConcurrentModificationException`. It may reflect some updates
made after iteration starts, but does not promise a snapshot of the
collection at one instant. `ConcurrentHashMap` iterators are weakly
consistent.

## 15. Difference between Class, Object, and Struct in Java

A **class** is a blueprint defining state and behavior. An **object** is
a runtime instance of a class. Java does not have C-style user-defined
`struct` types; classes are used to group fields and methods. Java
records are concise data-carrier classes, but they are still reference
types, not C structs.

------------------------------------------------------------------------

# 2. Java 8+ & Concurrency

## 1. What are Functional Interfaces? Give examples.

A functional interface has exactly one abstract method, so it can be
implemented with a lambda expression or method reference. It may still
contain default and static methods. Examples are `Predicate<T>`
(`test`), `Function<T,R>` (`apply`), `Consumer<T>` (`accept`),
`Supplier<T>` (`get`), and `Runnable` (`run`). `@FunctionalInterface`
asks the compiler to validate the contract.

## 2. Explain Lambda Expressions with a real use case.

A lambda is a concise implementation of a functional interface:

``` java
List<String> names = List.of("Maya", "Arun", "Anu");
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);
```

Here, `name -> name.startsWith("A")` implements a predicate. Lambdas
reduce boilerplate when passing behavior to streams, callbacks, or
asynchronous tasks.

## 3. Stream API --- lazy evaluation, intermediate vs terminal operations

Intermediate operations such as `filter`, `map`, and `sorted` build a
pipeline and are generally lazy. A terminal operation such as `collect`,
`count`, or `forEach` triggers evaluation. For example, a `filter()`
alone does not process elements until a terminal operation runs. Streams
are intended for processing data declaratively and should generally not
be reused after a terminal operation.

## 4. When do parallel streams hurt performance?

They can hurt when the collection is small, tasks are cheap, work is
blocking, splitting is inefficient, or shared mutable state causes
contention. Parallel execution also adds scheduling and coordination
overhead and commonly uses the shared common ForkJoinPool. I benchmark
before using it and prefer explicit executors for blocking or
workload-isolated tasks.

## 5. CompletableFuture --- merge responses from 3 APIs in parallel

I start each independent API call asynchronously, then combine the
results after all complete:

``` java
CompletableFuture<User> user = CompletableFuture.supplyAsync(
    () -> userClient.getUser(), executor);
CompletableFuture<List<Order>> orders = CompletableFuture.supplyAsync(
    () -> orderClient.getOrders(), executor);
CompletableFuture<Profile> profile = CompletableFuture.supplyAsync(
    () -> profileClient.getProfile(), executor);

CompletableFuture<Dashboard> dashboard =
    user.thenCombine(orders, UserOrders::new)
        .thenCombine(profile, (uo, p) ->
            new Dashboard(uo.user(), uo.orders(), p));
```

In production I configure timeouts, error handling, and a dedicated
executor. The calls must actually be independent for parallel execution
to help.

## 6. `supplyAsync` vs `runAsync`

`supplyAsync()` runs a supplier and returns a `CompletableFuture`
containing its result. `runAsync()` runs a `Runnable` and returns
`CompletableFuture<Void>` because there is no result. Both accept an
optional `Executor`.

## 7. `thenCombine()` vs `allOf()`

`thenCombine()` combines the results of two futures when both complete,
returning a result from a combining function. `allOf()` creates a future
that completes when all supplied futures complete, but its result type
is `Void`; I then read results from the original futures, usually after
`allOf().join()`. Use `thenCombine` when composing two values and
`allOf` to wait for many independent tasks.

## 8. What thread pool does `supplyAsync` use by default?

Without an executor argument, `CompletableFuture.supplyAsync()` normally
uses `ForkJoinPool.commonPool()`. That pool is shared across the
application, so unrelated tasks can affect one another.

## 9. Why can the common pool be dangerous? How do you fix it?

Blocking I/O or long-running work can occupy common-pool workers and
starve unrelated tasks. I use a dedicated, bounded `ExecutorService` for
blocking calls, configure its capacity and rejection behavior, and shut
it down cleanly. I also set timeouts so tasks do not occupy threads
indefinitely.

## 10. What happens if one API times out?

I can apply `orTimeout()` to complete a future exceptionally after a
deadline, or `completeOnTimeout()` to provide a fallback value:

``` java
future.orTimeout(2, TimeUnit.SECONDS)
      .exceptionally(ex -> fallbackValue);
```

A timeout does not necessarily cancel the underlying network operation.
I also configure the HTTP client's own connect/read/request timeouts and
decide whether to retry or return a partial response.

## 11. Future vs CompletableFuture

`Future` represents a result that may become available and mainly offers
blocking retrieval, cancellation, and status checks. `CompletableFuture`
implements `Future` and `CompletionStage`, so it supports non-blocking
continuations, composition, combining independent tasks, and exception
handling.

## 12. Why is `Future.get()` blocking a problem?

It blocks the calling thread until the result is ready. If I call
`get()` immediately after submitting each task, I may lose the benefit
of parallelism and tie up request threads. I prefer composing futures
and waiting only at the boundary where the result is genuinely needed,
with a timeout when appropriate.

## 13. Explain ExecutorService and thread pools

`ExecutorService` separates task submission from thread management. A
thread pool reuses threads rather than creating one for every task. I
choose pool size and queue capacity based on whether tasks are CPU-bound
or I/O-bound, set a rejection policy, monitor saturation, and shut down
the executor gracefully. An unbounded queue can hide overload and cause
memory pressure.

## 14. Core multithreading concepts: synchronization, volatile, atomic operations

-   **`synchronized`:** provides mutual exclusion and visibility through
    monitor acquire/release.
-   **`volatile`:** guarantees visibility and ordering for reads/writes
    of that variable, but does not make compound operations such as
    `count++` atomic.
-   **Atomic classes:** classes such as `AtomicInteger` use atomic
    operations, commonly CAS, for lock-free updates to a value. I select
    the mechanism based on whether I need visibility, a single atomic
    update, or protection of a multi-step invariant.

## 15. What are Virtual Threads in Java 21?

Virtual threads are lightweight threads managed by the JVM rather than
one-to-one by the operating system. They make thread-per-request code
practical for applications with many concurrent blocking I/O tasks. They
are not inherently faster for CPU-bound work; they mainly improve
scalability by reducing the cost of blocked threads.

## 16. When are virtual threads not the right choice?

They are not a way to speed up CPU-bound work, and they do not create
more database connections or increase downstream capacity. I still need
connection-pool limits, backpressure, timeouts, and concurrency
controls. They can also be less beneficial when a framework/library
blocks in ways that pin carrier threads, depending on the JDK and code
path.

## 17. What is Project Loom?

Project Loom is the OpenJDK project that introduced virtual threads and
related concurrency improvements. Its goal is to make high-concurrency
Java applications easier to write using straightforward thread-per-task
programming rather than requiring reactive APIs everywhere.

------------------------------------------------------------------------

# 3. Spring Boot & Hibernate

## 1. `@Component` vs `@Service` vs `@Repository` --- does it matter?

All three register Spring-managed beans through component scanning.
`@Service` communicates business-layer intent, and `@Repository`
communicates persistence-layer intent and enables persistence exception
translation for supported technologies. The annotations improve
readability and can affect framework behavior, so I use the one that
matches the layer.

## 2. What happens when a Spring Boot application starts?

The `main()` method calls `SpringApplication.run()`. Spring Boot
prepares the environment and application context, loads configuration,
registers bean definitions, performs component scanning and
auto-configuration, creates and wires beans, runs lifecycle callbacks,
and starts the embedded web server for a web application. Startup
failures typically come from configuration errors, missing beans,
dependency conflicts, or failed external connections.

## 3. How does auto-configuration decide which beans to create? (`spring.factories`)

Auto-configuration is conditional. Classes use conditions such as
`@ConditionalOnClass`, `@ConditionalOnMissingBean`, and
`@ConditionalOnProperty` to decide whether to configure beans. In modern
Spring Boot, auto-configuration candidates are listed in
`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`;
older versions commonly used `spring.factories`. Auto-configuration
backs off when the application supplies its own bean in many cases.

## 4. Explain Dependency Injection in Spring

Dependency Injection means an object receives its dependencies from the
container instead of constructing them itself. Spring creates and wires
beans. I prefer constructor injection because dependencies are explicit,
required fields can be final, and unit testing is easier:

``` java
@Service
class OrderService {
    private final OrderRepository repository;
    OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

## 5. Bean lifecycle --- creation to destruction

At a high level, Spring instantiates a bean, injects dependencies, runs
aware callbacks, invokes bean post-processors, calls initialization
callbacks such as `@PostConstruct`, and makes the bean available. On
context shutdown, destruction callbacks such as `@PreDestroy` run for
managed singleton beans. Exact ordering has additional extension points,
including `BeanPostProcessor` and `InitializingBean`.

## 6. `@Autowired` internals --- how does Spring resolve dependencies?

Spring resolves the injection point by type first. If multiple
candidates exist, it can use `@Qualifier`, `@Primary`, bean names, and
other resolution rules. Constructor injection is the clearest choice for
required dependencies. If no candidate exists, startup usually fails
with an unsatisfied dependency error.

## 7. BeanNotFoundException --- common causes and fixes

In Spring the common exception is `NoSuchBeanDefinitionException` or an
unsatisfied dependency error. Causes include missing component scanning,
a class not annotated or registered as a bean, a profile/conditional
configuration not being active, a dependency missing from the classpath,
or a qualifier mismatch. I check the exact bean type/name, package scan,
active profile, conditions report, and configuration.

## 8. `@Transactional` --- what happens under the hood?

Spring usually creates an AOP proxy around the bean. When a call enters
a proxied transactional method, the transaction interceptor obtains or
joins a transaction through the configured transaction manager, then
commits on success or rolls back according to the rollback rules. By
default, unchecked exceptions and `Error`s trigger rollback; checked
exceptions do not unless configured. The transaction manager must match
the resource being used.

## 9. What happens when you call a `@Transactional` method from within the same class?

Self-invocation usually bypasses the Spring proxy, so the inner method's
transactional annotation is not applied independently. The outer method
may still have a transaction, but the inner method's declared
propagation/isolation settings will not be intercepted. A common fix is
to move the transactional operation to another Spring bean or call
through a proxy; redesigning the transaction boundary is often best.

## 10. Lazy Loading vs Eager Loading in Hibernate

Lazy loading fetches an association only when it is accessed; it can
reduce unnecessary database work. Eager loading fetches it as part of
the entity load plan, potentially causing larger joins or extra queries.
I generally prefer lazy associations and explicitly fetch what each use
case needs, because globally eager relationships can create performance
and N+1 issues.

## 11. How do you handle lazy loading in a REST API response?

I avoid returning JPA entities directly. Instead, I fetch the required
fields within a service transaction and map them to DTOs. I can use
`JOIN FETCH`, `@EntityGraph`, or projections to load the exact data.
This avoids `LazyInitializationException`, accidental serialization of
huge object graphs, and leaking persistence details into the API.

## 12. Explain the N+1 Query Problem --- detect and fix

N+1 occurs when one query loads a list of parent entities and then one
additional query is executed for each parent to load an association. I
detect it through Hibernate SQL logs, query metrics, tracing, and tests.
Fixes include `JOIN FETCH`, `@EntityGraph`, DTO projections, or
`@BatchSize`, chosen based on result size and relationship cardinality.
A join is not always best if it multiplies rows significantly.

## 13. How to detect slow SQL: `EXPLAIN ANALYZE`, Hibernate logging

I first identify the slow endpoint/query using APM, database monitoring,
or request traces. I enable targeted Hibernate SQL/bind logging in a
safe environment and inspect the database execution plan.
`EXPLAIN ANALYZE` executes the query in databases that support it, so I
use it carefully in production. I compare estimated and actual rows,
scans, joins, sorts, and time spent.

## 14. Fixes for N+1: `JOIN FETCH`, `@EntityGraph`, `@BatchSize`

-   `JOIN FETCH` loads an association in the same query.
-   `@EntityGraph` specifies which associations should be fetched for a
    repository operation.
-   `@BatchSize` lets Hibernate fetch multiple lazy associations in
    batches, reducing round trips. I choose based on pagination,
    relationship cardinality, and query size rather than applying one
    solution everywhere.

## 15. Hibernate caching --- first-level vs second-level

The first-level cache is the persistence context
(`EntityManager`/`Session`) and is enabled by default; within that
context, an entity identity is normally represented by one managed
instance. The second-level cache is optional and shared across sessions,
requiring configuration and a supported cache provider. Query caching is
a separate feature. I only enable shared caching when data access
patterns and invalidation requirements justify it.

## 16. Build a Spring Boot CRUD app with an in-memory rate limiter (10 req/min, return 429)

For a single-instance demonstration, a simple per-client fixed-window
limiter can track request counts and reset them every minute. In
production I would use a maintained library such as Bucket4j or a
distributed store such as Redis because an in-memory limiter resets on
restart and each application instance has separate counters. The
endpoint should return HTTP `429 Too Many Requests` when the limit is
exceeded.

Minimal service logic using a synchronized fixed window:

``` java
@Component
class RateLimiter {
    private static final int LIMIT = 10;
    private static final long WINDOW_MS = 60_000;
    private final Map<String, Window> clients = new ConcurrentHashMap<>();

    boolean allow(String key) {
        long now = System.currentTimeMillis();
        Window w = clients.compute(key, (k, old) -> {
            if (old == null || now - old.start >= WINDOW_MS)
                return new Window(now, 1);
            return new Window(old.start, old.count + 1);
        });
        return w.count <= LIMIT;
    }
    record Window(long start, int count) {}
}
```

Controller sketch:

``` java
@GetMapping("/items")
ResponseEntity<?> items(HttpServletRequest request) {
    String client = request.getRemoteAddr();
    if (!limiter.allow(client))
        return ResponseEntity.status(429).body("Too many requests");
    return ResponseEntity.ok(service.getItems());
}
```

For a real app, define the client identity carefully (often
authenticated user/API key, not blindly trusting forwarded IP headers),
expire old map entries, and use a correct rolling/token-bucket algorithm
if the policy requires it.

------------------------------------------------------------------------

# 4. Database & SQL

## 1. What is a database index? Trade-offs on write-heavy tables?

An index is a data structure that helps the database locate rows without
scanning the entire table. It speeds up matching, joins, and sometimes
sorting, but consumes storage and adds work to inserts, updates, and
deletes because indexes must be maintained. I create indexes based on
actual query patterns and verify their benefit with execution plans.

## 2. What is cardinality and why does it matter for index selection?

Cardinality can mean the number of distinct values in a column, or the
number of rows returned by a query, depending on context. A
high-distinct-value column often makes a selective index more useful,
while a low-cardinality column may be less useful alone. The optimizer
also considers data distribution, table size, composite-index order, and
query predicates.

## 3. Clustered vs Non-Clustered Index

A clustered index determines or closely corresponds to the
physical/logical row organization of a table, depending on the database
engine; a table typically has one such organization. A non-clustered
index is a separate structure containing indexed values and row
locators. Terminology and exact behavior vary by database---for example,
Oracle uses a different storage model and does not use the same
clustered-index concept as SQL Server.

## 4. Table with 50M rows, queries are slow --- investigation

I identify the slow query and its frequency, inspect the execution plan
and actual row counts, check indexes and statistics, and look for full
scans, inefficient joins, sorts, lock waits, and excessive data
retrieval. I verify whether predicates are selective and index-friendly,
review connection-pool and database resource metrics, and test changes
against representative data. I measure before and after rather than
adding indexes blindly.

## 5. Normalization vs Denormalization

Normalization reduces duplication and update anomalies by separating
entities into related tables. Denormalization duplicates or precomputes
data to make reads faster, at the cost of storage and consistency
complexity. I start with a normalized transactional model, then
denormalize only when measured read-performance needs justify it.

## 6. Partitioning vs Sharding

Partitioning divides a table or dataset into smaller pieces, often
within one database system, by range, list, or hash. Sharding
distributes data across separate database nodes or instances.
Partitioning can improve maintenance and query pruning; sharding adds
horizontal capacity but also adds routing, rebalancing, cross-shard
query, and operational complexity.

## 7. How do you choose a shard key?

I look for high cardinality, even distribution, and alignment with the
most common query patterns. I avoid keys that create hotspots, such as
monotonically increasing values when they concentrate writes. I also
consider whether queries can be routed to one shard and how hard it will
be to rebalance data as it grows.

## 8. ACID properties with examples

-   **Atomicity:** an order and its inventory update either both commit
    in the transaction or neither does.
-   **Consistency:** a transaction preserves defined constraints, such
    as stock not becoming negative.
-   **Isolation:** concurrent transactions do not observe invalid
    intermediate states; the exact guarantee depends on isolation level.
-   **Durability:** after commit, committed data survives failures
    according to the database's durability guarantees.

## 9. Optimistic vs Pessimistic Locking

Optimistic locking assumes conflicts are uncommon and detects them at
update time, often using a version column (`@Version`). Pessimistic
locking locks rows while a transaction works on them, preventing
conflicting updates but potentially increasing waits and deadlocks. I
choose based on contention and the cost of conflicts.

## 10. E-commerce inventory updates --- which locking strategy and why?

I would use an atomic conditional update or a row-level pessimistic lock
for highly contended stock, depending on throughput and business
requirements. For example:

``` sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = ? AND quantity > 0;
```

I check the affected-row count; zero means no stock was available. This
avoids a read-then-write race. Optimistic versioning can work when
contention is low, with conflict handling and retries.

## 11. How to debug a slow database query?

Find the exact query and timing first, then inspect its execution plan,
actual rows, indexes, statistics, joins, sorts, locks, and I/O. Check
whether the application fetches too many columns/rows or runs the query
repeatedly. Reproduce with representative parameters and data, make one
change at a time, and compare latency and resource use.

## 12. Reading EXPLAIN plans --- what to look for?

I look at access paths (index scan vs table/full scan), join order and
join type, estimated versus actual rows, filter selectivity, sorts,
temporary spills, and cost/time. A full scan is not automatically bad;
it can be optimal for small tables or queries returning a large fraction
of rows. I focus on unexpected row counts and expensive operations.

## 13. Full table scan causes and how to eliminate them?

A scan may happen because no useful index exists, a predicate is not
selective, statistics are stale, a function/cast prevents index use, or
the optimizer determines scanning is cheaper. I review the plan and
query predicates, update statistics when appropriate, add or adjust an
index if justified, and avoid wrapping indexed columns in functions
unless a matching function-based index is available. I do not eliminate
every full scan by force.

## 14. SQL query optimization techniques

Select only required columns, filter early, use suitable composite
indexes, make joins use appropriate keys, avoid unnecessary subqueries
and repeated queries, paginate large results, and use batch operations
when appropriate. Keep statistics current and validate with plans and
measurements. Avoid premature optimization and indexes that slow writes
without improving important queries.

## 15. Why use separate read and write stores? (CQRS)

CQRS separates the command model that changes state from the query model
optimized for reads. In larger systems, the read model may be a
projection or separate store tailored to search/reporting. It can scale
reads independently and simplify query models, but adds replication lag,
operational cost, and consistency complexity. It is not necessary for
every application.

## 16. How to manage consistency between read and write stores?

I generally update the authoritative write store and publish an event
reliably, often with the Transactional Outbox Pattern. A consumer
updates the read projection idempotently. The read store may be
eventually consistent, so APIs should define how they handle lag,
retries, duplicate events, and rebuilds. For operations requiring
immediate correctness, read from the authoritative store or provide a
read-your-writes strategy.

------------------------------------------------------------------------

# 5. Microservices & Distributed Systems

## 1. Monolith vs Microservices --- trade-offs

A monolith deploys the application as one unit and is simpler to
develop, test, and operate initially. Microservices split capabilities
into independently deployable services, allowing independent scaling and
team ownership, but introduce network failures, distributed tracing,
deployment coordination, and data consistency challenges. I choose
microservices when domain and organizational needs justify the
operational cost, not simply because they are fashionable.

## 2. Monolith to Microservices migration strategies

I would first understand domain boundaries and dependencies, improve
observability and automated tests, then extract one low-coupling
capability at a time. I can use the Strangler Fig approach, route
selected traffic to the new service, and migrate data carefully. I avoid
a big-bang rewrite and define rollback, ownership, and consistency
behavior for each extraction.

## 3. Explain the Strangler Fig Pattern

The Strangler Fig Pattern gradually replaces parts of a legacy system. A
routing layer sends selected functionality to new services while the
remaining features continue to use the old application. Over time, more
capabilities move until the legacy component can be retired. This
reduces the risk of a complete rewrite.

## 4. API Gateway --- what is it and why is it needed?

An API Gateway is an entry point for client requests. It can route
requests, terminate TLS, authenticate/authorize, apply rate limits,
transform requests, and collect metrics. It simplifies client access but
can become a bottleneck or single point of failure if not deployed and
operated redundantly. Business logic generally belongs in services, not
the gateway.

## 5. Service Discovery --- how do services find each other?

Services can use client-side discovery, where a client queries a
registry, or server-side discovery, where a load balancer routes to
healthy instances. In Kubernetes, Services and DNS commonly provide
stable names while the platform tracks endpoints. Discovery must account
for health, scaling, network failures, and timeouts.

## 6. Database-per-Service pattern --- why and how?

Each microservice owns its data and exposes it through APIs or events
rather than allowing other services to access its tables directly. This
reduces coupling and lets services evolve independently. Cross-service
workflows require APIs, events, projections, or sagas instead of a
single database join/transaction. Data duplication may be acceptable for
read models.

## 7. Why doesn't `@Transactional` work across microservices?

Spring's `@Transactional` manages a local transaction through a
transaction manager and resource, such as one database connection. A
remote HTTP call or message broker is outside that local transaction. If
one service commits and another fails, the first commit is not
automatically rolled back. Cross-service workflows need patterns such as
sagas, outbox, idempotency, or a distributed transaction protocol where
appropriate.

## 8. What are Distributed Transactions? Why do they fail at scale?

A distributed transaction coordinates updates across multiple
independent resources/services so they act as one logical unit. Network
partitions, process crashes, coordinator failures, long-held locks, and
partial availability make them complex. They can be appropriate for
tightly controlled systems, but at scale many architectures prefer
eventual consistency and compensating actions.

## 9. Two-Phase Commit (2PC) --- why is it problematic?

In phase one, a coordinator asks participants to prepare and hold the
resources needed to commit. In phase two, it tells them to commit or
abort. It can provide atomicity across participants, but participants
may hold locks while waiting, and coordinator/network failures can leave
transactions blocked or in-doubt. It also requires compatible
transaction managers and adds latency.

## 10. Explain the Saga Pattern

A saga breaks a business transaction into a sequence of local
transactions across services. Each successful step triggers the next. If
a later step fails, compensating transactions attempt to undo the
business effect of earlier steps---for example, release reserved
inventory if payment fails. Compensation is a new business action, not a
database rollback, and may itself need retries and manual handling.

## 11. Choreography vs Orchestration Saga

In choreography, services react to events and trigger subsequent
actions; it is loosely coupled but can be hard to understand as
workflows grow. In orchestration, a coordinator explicitly directs each
step and handles success, failure, retries, and compensation. I prefer
choreography for short, simple flows and orchestration for complex
workflows with visible state and branching.

## 12. Compensation transactions

A compensation transaction counteracts a previously completed business
step. For example, if a hotel booking succeeds but payment fails,
compensation may cancel the booking. It is not always an exact
reversal---an email cannot be unsent, and money may require a refund.
Compensation must be idempotent, observable, and retried or escalated
when it fails.

## 13. Tools for orchestration

Tools include Temporal, AWS Step Functions, and Camunda. They help model
workflow state, retries, timeouts, branching, and recovery. The choice
depends on deployment environment, workflow complexity, durability
requirements, team skills, and integrations. The application still needs
idempotent activities and carefully designed compensation logic.

## 14. Event-Driven Architecture with a real example

In an event-driven system, a service publishes an event describing
something that happened, and interested consumers react asynchronously.
For example, after an order is placed, the order service publishes
`OrderCreated`; inventory reserves stock, payment processes the charge,
and notifications send an update. This decouples services but requires
handling duplicates, ordering, retries, and eventual consistency.

## 15. How do you ensure data consistency with events? (Transactional Outbox)

The dual-write problem occurs when a service updates its database and
publishes an event separately; one operation can succeed while the other
fails. With the Transactional Outbox Pattern, the service writes
business data and an outbox row in the same database transaction. A
publisher or CDC tool then publishes the outbox event. Consumers should
be idempotent because delivery may be repeated.

------------------------------------------------------------------------

# 6. Resiliency & Reliability Patterns

## 1. Circuit Breaker --- explain the three states

-   **Closed:** requests flow; failures are counted.
-   **Open:** calls are rejected or fail fast after the failure
    threshold is reached.
-   **Half-open:** a limited number of trial calls test whether the
    dependency has recovered. If trials succeed, it closes; if they
    fail, it opens again. This prevents repeated calls to an unhealthy
    dependency from exhausting resources.

## 2. Circuit breaker vs retry

A retry repeats an operation that may have failed transiently. A circuit
breaker stops calls to a dependency that is failing repeatedly. They can
work together: retry a small number of transient failures, while the
circuit breaker protects the service from sustained outages. Retrying
non-idempotent operations without safeguards can duplicate effects.

## 3. What tool do you use?

For Spring Boot, I would use Resilience4j for circuit breakers, retries,
rate limiters, bulkheads, and time limiters. Netflix Hystrix is in
maintenance mode and is no longer the preferred choice for new systems.

## 4. Retry --- when to retry and when not to?

Retry transient failures such as temporary network errors, throttling,
or selected 5xx responses, with a bounded attempt count and backoff. Do
not retry validation errors or other permanent failures. For
non-idempotent operations, use an idempotency key or another
deduplication mechanism. Coordinate retries across layers to avoid
multiplying attempts.

## 5. Exponential Backoff --- why not retry immediately?

Exponential backoff increases the delay between attempts, giving an
overloaded or temporarily unavailable dependency time to recover.
Immediate retries can add more load during an outage and worsen the
failure.

## 6. Exponential Backoff with Jitter

Jitter adds a random component to the backoff delay. Without jitter,
many clients that fail together can retry together, causing a thundering
herd. A common approach is to choose a random delay within a bounded
range derived from the exponential delay.

## 7. Bulkhead Pattern --- what problem does it solve?

A bulkhead isolates resources so one slow dependency or workload cannot
consume all capacity. For example, separate thread pools or concurrency
limits for payment and profile calls prevent a slow profile service from
exhausting all workers. Resilience4j supports semaphore and thread-pool
bulkheads.

## 8. Timeout Pattern --- how to set appropriate timeouts?

Set explicit connect, read/response, and overall request deadlines based
on latency objectives and downstream behavior. A downstream timeout
should fit inside the caller's deadline, leaving time to return a
response. Use measurements and percentiles, not guesses. Propagate
deadlines where possible and ensure timed-out work is not left running
unnecessarily.

## 9. Dead Letter Queue (DLQ) --- when and why?

A DLQ stores messages that cannot be processed after the configured
retry policy or are identified as permanently invalid. It preserves the
failed message and metadata for investigation and controlled replay
instead of blocking the main flow indefinitely. I monitor DLQ depth and
age and define ownership and replay procedures.

## 10. DLQ vs logging errors

Logs describe an error but may not preserve a replayable message or the
processing state. A DLQ retains the message for inspection and possible
reprocessing. I use both: structured logs and metrics for diagnosis, and
a DLQ for failed asynchronous messages.

------------------------------------------------------------------------

# 7. Messaging Systems

## 1. Kafka vs RabbitMQ --- when to use each?

Kafka is a distributed event log designed for high-throughput streams,
retention, replay, and multiple independent consumer groups. RabbitMQ is
a message broker well suited to work queues, flexible routing,
acknowledgements, and per-message delivery workflows. The decision
depends on replay requirements, throughput, routing needs, latency, and
operational expertise---not on one being universally better.

## 2. When to use Kafka? When to use RabbitMQ?

I choose Kafka when multiple consumers need to independently replay
retained events, when throughput is high, or when event streaming/stream
processing is central. I choose RabbitMQ for task distribution, complex
routing, and broker-managed queue semantics. SNS/SQS may be a better fit
in AWS-native architectures.

## 3. Explain Consumer Groups in Kafka

A consumer group is a set of consumers sharing consumption of a topic.
Each partition is assigned to at most one consumer within a group at a
time, so partitions provide the parallelism limit for that group.
Different groups independently consume the same topic. If there are more
consumers than partitions, some consumers remain idle.

## 4. Partitions --- how do they enable horizontal scaling?

Kafka splits a topic into partitions distributed across brokers.
Consumers in a group can process different partitions concurrently. More
partitions can increase parallelism, but add overhead and do not
guarantee proportional throughput. Ordering is guaranteed within a
partition, not across the entire topic.

## 5. Offsets --- what if a consumer crashes before committing?

If processing completed but the offset was not committed, the message
may be delivered again after restart or reassignment, giving
at-least-once behavior. The consumer should be idempotent. If the offset
is committed before processing is safely completed, a crash can cause
message loss from the application's perspective. Commit strategy must
match the processing guarantees.

## 6. At-most-once, at-least-once, exactly-once

-   **At-most-once:** a message is processed zero or one time; loss is
    possible.
-   **At-least-once:** retries/redelivery prevent loss in many failure
    scenarios, but duplicates are possible.
-   **Exactly-once:** the defined effect is applied once within a
    specified boundary, often using Kafka transactions for
    Kafka-to-Kafka processing. External databases or APIs still need
    coordination or idempotency; it is not a universal end-to-end
    guarantee.

## 7. Which delivery guarantee do you use in production and why?

For many business integrations, I choose at-least-once delivery with
idempotent consumers because it is practical and recoverable. I use a
stable event/business ID and record processed IDs or enforce a unique
constraint. Where Kafka-to-Kafka atomic processing is needed, Kafka
transactions can atomically publish output records and commit consumed
offsets.

## 8. What is the Poison Pill problem in Kafka?

A poison pill is a message that repeatedly fails processing, potentially
blocking progress on its partition if the consumer cannot move past it.
I classify the failure, retry transient errors with limits, and route
permanent failures to a DLQ or retry topic while preserving the original
payload and metadata. The consumer must continue safely without silently
losing the message.

## 9. Distinguish transient vs permanent failures

Transient failures include temporary timeouts, network issues,
throttling, and some 5xx responses. Permanent failures include invalid
schema, missing required fields, or business-rule violations. I retry
only transient failures with bounded backoff; permanent failures go to a
DLQ or rejection flow. Some errors need domain-specific classification
rather than status-code-only rules.

## 10. Transactional Outbox Pattern --- why and how?

Write the business change and an outbox event in the same local database
transaction. A polling publisher or CDC connector publishes the event
later. This avoids losing an event between a successful database commit
and a failed broker publish. Publication may still be duplicated, so
consumers should be idempotent.

## 11. Polling Publisher vs CDC (Debezium)

A polling publisher periodically queries unpublished outbox rows, marks
or tracks publication, and is simple to understand but adds polling load
and latency. CDC tools such as Debezium read database change logs and
can publish outbox events with lower polling overhead, but add connector
and infrastructure complexity. Both require handling duplicates, schema
evolution, monitoring, and recovery.

## 12. What about duplicate events? (Idempotent consumers)

Give each event a stable unique ID. The consumer can store processed IDs
in the same transaction as its business update, or enforce a unique
business constraint. If the ID was already processed, it
acknowledges/skips the duplicate. The deduplication record and business
side effect must be atomic where possible.

## 13. What about event ordering?

Kafka preserves order within a partition. To preserve ordering for a
business entity, use a stable key such as `orderId` so related events go
to the same partition. Ordering across partitions is not guaranteed.
Also consider retries, multiple consumers, and out-of-order events from
upstream systems; sequence numbers or version checks may be needed.

------------------------------------------------------------------------

# 8. AWS & Cloud

## 1. What is Serverless?

Serverless means the cloud provider manages much of the underlying
infrastructure and scaling, and the user pays based on usage or
provisioned capacity rather than managing servers directly. It does not
mean there are no servers. Lambda and many managed event/database
services are examples.

## 2. ECS vs Lambda --- when to choose which?

I choose Lambda for event-driven, short-lived functions with variable
traffic and supported runtime limits. I choose ECS for long-running
services, custom runtime needs, sustained workloads, or more control
over process lifecycle. The choice also depends on startup latency,
concurrency, networking, cost profile, and deployment requirements.

## 3. Is ECS serverless?

ECS is a container orchestration service, not inherently serverless. ECS
on EC2 means I manage the underlying instances. ECS on Fargate lets AWS
manage the compute infrastructure for tasks, so it provides a serverless
container experience.

## 4. Lambda cold start --- when is it a problem?

A cold start occurs when Lambda initializes a new execution environment,
adding latency before the handler runs. It matters for latency-sensitive
requests, infrequently invoked functions, and functions with heavy
initialization or large dependencies. I reduce it by keeping
initialization lean, reusing clients outside the handler, choosing
appropriate runtimes and memory, and considering provisioned concurrency
where justified.

## 5. ECS vs Fargate

ECS is the container orchestration/control plane that schedules and
manages tasks. Fargate is a serverless compute option on which ECS tasks
can run without managing EC2 instances. ECS can run on either EC2 or
Fargate. EC2 may provide more control and can be economical for steady
high utilization; Fargate reduces host management.

## 6. Lambda internals --- how does it work?

An event source or direct invocation triggers Lambda. AWS allocates or
reuses an execution environment, initializes the runtime and function,
and invokes the handler with the event. Environments may be reused, so
clients can be initialized outside the handler, but code must not assume
local memory persists or that execution happens only once. Retries and
delivery semantics depend on the invocation source.

## 7. SQS vs SNS --- when to use each?

SQS is a durable queue that buffers work for consumers, allowing
messages to be processed independently and retried. SNS is a
publish/subscribe notification service that fans a message out to
multiple subscribers. They can be combined: publish to SNS and deliver
to multiple SQS queues so each consumer has its own buffer and retry
behavior.

## 8. What is a DLQ in SQS?

An SQS dead-letter queue receives messages that exceed the source
queue's configured `maxReceiveCount`. It isolates repeatedly failing
messages so healthy messages can continue. I monitor its depth and age,
investigate the root cause, and redrive messages only after fixing the
issue and validating replay safety.

## 9. API Gateway --- how does it handle millions of requests?

API Gateway is a managed, scalable front door, but throughput is
governed by service quotas, account/region limits, route configuration,
and downstream capacity. I design for throttling, caching where
appropriate, authentication, payload limits, and backpressure. I request
quota increases when justified and load-test the entire path; API
Gateway scaling does not mean the backend automatically scales.

## 10. EventBridge --- when to use over SNS?

EventBridge is useful for event buses, content-based filtering, routing
events from AWS services or applications, and scheduled rules. SNS is a
simpler pub/sub fan-out service, especially for direct notifications to
subscribers. EventBridge is often a better fit for rich event routing
and integrations; SNS is often a straightforward fan-out choice.

## 11. CloudWatch --- metrics, logs, alarms

CloudWatch collects metrics and logs, and alarms evaluate metrics or
metric expressions against thresholds. I use dashboards for operational
visibility, alarms for actionable conditions, structured logs for
debugging, and correlation/request IDs to connect events across
services. Alerts should have clear ownership and avoid excessive noise.

## 12. Auto Scaling --- how does it work?

Auto Scaling adjusts capacity based on policies and signals such as CPU,
request count, queue depth, or custom metrics. In ECS, service auto
scaling can change desired task count; EC2 Auto Scaling changes instance
capacity. I set minimum/maximum capacity, cooldown or stabilization
behavior, health checks, and alarms, and ensure the application can
handle scale-in safely.

## 13. ALB vs NLB

An Application Load Balancer operates at Layer 7 and routes HTTP/HTTPS
traffic using host, path, and other application-level rules. A Network
Load Balancer operates at Layer 4 for TCP/UDP/TLS traffic and is
designed for very high performance and low latency with static IP
options. I choose based on protocol, routing features, client-IP needs,
and performance requirements.

## 14. Design an email notification system using AWS services

I would accept notification requests through an API or application
event, persist the request/status, and enqueue work in SQS. Workers on
Lambda or ECS consume messages, apply templates and preferences, and
send through Amazon SES or an email provider. I would add retries with
backoff, an SQS DLQ, idempotency keys, rate controls, CloudWatch
metrics/alarms, and delivery/bounce tracking. For scheduled emails, a
scheduler or database-driven dispatcher creates queue messages.

## 15. How to handle 1M emails/hour?

One million per hour is about 278 emails/second on average. I would
verify provider sending quotas and warm-up/reputation requirements
first, then scale workers horizontally based on queue depth and age. SQS
buffers bursts; consumers use bounded concurrency and provider-aware
rate limiting. I would track accepted, delivered, bounced, throttled,
retried, and DLQ counts, and design templates and recipient data lookup
to avoid a database bottleneck.

## 16. Failure handling --- DLQ strategy

Retry transient provider failures with exponential backoff and jitter,
respecting provider retry-after guidance. After a bounded number of
attempts, move the message to a DLQ with error metadata. Alert on DLQ
depth and age, provide a controlled redrive process, and make sends
idempotent as far as the provider/API allows. Avoid endless retries that
harm sender reputation.

## 17. Scheduled email notifications from DB records --- design

Use EventBridge Scheduler or a scheduled job to trigger a dispatcher
that queries records due for notification, claims them safely, and
enqueues messages in SQS. Workers send the emails and update status. Use
a unique notification ID, transactional claiming or locking to prevent
duplicate scheduling, pagination/batching for large datasets, and
retries/DLQ for failures. Do not hold a database transaction open while
sending network requests.

## 18. Handling millions of requests through API Gateway

I would confirm quotas and throttling limits, use caching for safe
repeatable reads, authenticate and validate requests early, and use
Lambda/ECS auto scaling behind the gateway. I would protect dependencies
with timeouts, bulkheads, rate limits, and queues for asynchronous work.
Load testing, dashboards, alarms, and graceful degradation are
essential; the gateway alone cannot compensate for an undersized
downstream system.

------------------------------------------------------------------------

# 9. NoSQL Databases

## 1. MongoDB vs DynamoDB --- when to use each?

MongoDB is a document database with flexible querying, secondary
indexes, and rich document-oriented operations. DynamoDB is a managed
key-value/document database designed for predictable low-latency access
at scale using known access patterns and partition/sort keys. I choose
MongoDB for flexible document queries and aggregation; DynamoDB for
managed scale and key-oriented access patterns, especially in AWS.

## 2. When to use MongoDB? When to use DynamoDB?

Use MongoDB when the domain naturally maps to documents and queries need
flexible filtering, indexes, and aggregation. Use DynamoDB when access
patterns are well defined, traffic can be large or spiky, and managed
scaling/availability are priorities. DynamoDB data modeling is
access-pattern-first; it is not a relational database where arbitrary
joins are expected.

## 3. Partition Keys in DynamoDB --- how to choose?

Choose a key with high cardinality and an even distribution of
reads/writes to avoid hot partitions. It should also support the primary
access patterns. A composite key can use a partition key plus sort key
for related items and range queries. I test likely traffic distributions
and use write sharding or key design changes if a hot key is
unavoidable.

## 4. MongoDB Replica Sets --- primary/secondary roles and election

A replica set maintains multiple copies of data. The primary accepts
writes and records operations in the oplog; secondaries replicate those
operations and can serve reads if configured. If the primary fails,
eligible members hold an election and a new primary is chosen.
Replication is typically asynchronous, so read preferences and write
concern affect consistency and durability.

## 5. Sharding in MongoDB --- when and how to pick shard keys

Shard when one server's storage, throughput, or working set is no longer
sufficient and the operational complexity is justified. Choose a shard
key that distributes writes and data evenly and supports common queries.
Monotonically increasing keys can create hotspots in some patterns; poor
shard keys cause scatter-gather queries or uneven chunks. Plan for
resharding and operational monitoring.

## 6. CAP Theorem with real-world examples

During a network partition, a distributed system must trade off between
strong consistency and availability for affected operations.
**Consistency** in CAP means each successful read sees the latest
successful write according to a single-copy model; **availability**
means every request to a non-failing node receives a response;
**partition tolerance** means the system continues despite communication
partitions. Since partitions can occur, systems choose how to behave
during them. CAP does not mean a database permanently chooses only two
properties under normal conditions.

## 7. Eventual Consistency --- what does it mean?

After a write, replicas or projections may temporarily return stale
data, but if no new updates occur and replication succeeds, they
converge. The application must tolerate a lag window and possible
read-after-write inconsistency. Eventual consistency is a guarantee
about convergence, not a promise of a specific delay.

## 8. When is eventual consistency acceptable? When is it not?

It is often acceptable for feeds, analytics, search indexes, and
notification status, where slight lag is tolerable. It may not be
acceptable for checking available funds or inventory if stale data could
violate a business invariant. Even there, a system can use eventual
projections for display while enforcing correctness in the authoritative
transactional service.

------------------------------------------------------------------------

# 10. React & Frontend

## 1. React lifecycle --- mounting, updating, unmounting

For a functional component, mounting means it first renders and is
committed to the DOM; updating occurs when props or state changes cause
another render; unmounting removes it. `useEffect` can perform side
effects after commit and return cleanup logic. Class components expose
lifecycle methods such as `componentDidMount`, `componentDidUpdate`, and
`componentWillUnmount`.

## 2. Functional vs Class Components --- why functional won?

Functional components are simpler to compose and use Hooks to manage
state and side effects without lifecycle-method complexity. They work
naturally with custom Hooks for reusable logic. Class components remain
supported, but most modern React development favors function components.

## 3. `useState` --- how does it work?

`useState` gives a component state value and a setter. Calling the
setter schedules a render; React may batch updates. When the next state
depends on the previous state, I use the functional updater:

``` jsx
setCount(previous => previous + 1);
```

I do not mutate objects or arrays in state; I create new references so
React can detect changes correctly.

## 4. `useEffect` --- dependencies, cleanup, pitfalls

`useEffect` synchronizes a component with external systems, such as
subscriptions, timers, or network requests. The dependency array
determines when the effect is re-run; cleanup runs before the effect
re-runs and when the component unmounts. Common issues include missing
dependencies, stale closures, effects that loop because they update a
dependency, and failing to cancel/ignore outdated requests. I avoid
using effects for values that can be calculated during render.

## 5. `useMemo` vs `useCallback`

`useMemo` memoizes a computed value; `useCallback` memoizes a function
reference. I use them when a measured expensive calculation or
referential equality helps avoid work in memoized children or
dependencies. They are performance optimizations, not correctness tools,
and React may still render in cases where memoization is not applicable.

## 6. When can `useMemo` hurt performance?

It adds dependency comparisons and retains cached values, increasing
complexity and memory use. For cheap calculations, the overhead can
exceed the benefit. I use it when profiling identifies meaningful work
or when stable references prevent costly downstream renders.

## 7. `React.memo` --- when to use and avoid?

`React.memo` can skip rendering a function component when its props are
shallowly equal. It helps when a component renders often with unchanged
props and rendering is expensive. It is less useful if props are newly
created objects/functions every time or the component is cheap. It does
not prevent renders caused by the component's own state or consumed
context changing.

## 8. Context API vs Redux

Context is useful for relatively low-frequency shared values such as
theme, locale, or authenticated user information. Redux Toolkit is
useful for complex shared application state, predictable update flows,
middleware, devtools, and larger state domains. Context is not
automatically a full state-management replacement; updates to a provider
value can re-render consumers.

## 9. What about Zustand or Jotai?

Zustand offers a lightweight store with selector-based subscriptions and
minimal boilerplate. Jotai uses small composable atoms for state. They
can be good alternatives when their simpler models fit the app. I choose
based on team familiarity, debugging needs, state complexity, and
ecosystem rather than adopting a library just because it is newer.

## 10. React reconciliation and Virtual DOM

React creates an in-memory representation of the UI and compares the new
render output with the previous one. Reconciliation determines which
parts need updating, and React commits necessary DOM changes. The
Virtual DOM is not inherently faster than every direct DOM operation; it
gives React a declarative model for calculating updates. Keys help React
match list items across renders.

## 11. When does React bail out of re-rendering?

React can skip some work when state is set to an `Object.is`-equal
value, and `React.memo` can skip a child render when props compare
equal. However, parent renders, context changes, state changes, and
concurrent scheduling can still lead to rendering. A render does not
necessarily mean every DOM node is updated.

## 12. Client-side vs Server-side routing

Client-side routing changes the displayed view in the browser without a
full page reload, often improving navigation experience after the app
loads. Server-side routing sends a request to the server for a page or
route and can return HTML directly. Client-side routing needs server
fallback configuration for deep links; server rendering can improve
initial content and SEO depending on the application.

## 13. BrowserRouter vs HashRouter

`BrowserRouter` uses the History API and clean paths, but the server
must serve the app entry point for frontend routes. `HashRouter` stores
the route after `#`, so the fragment is not sent to the server; this can
simplify static hosting but produces less clean URLs. I choose based on
deployment/server configuration and URL requirements.

## 14. Dynamic, nested, protected, and lazy-loaded routes

Dynamic routes use parameters such as `/users/:id`; nested routes render
child routes inside a parent layout. Protected routes check
authentication/authorization in the UI, but the backend must still
enforce access control. Lazy-loaded routes use `React.lazy` and
`Suspense` to split bundles and load code when needed.

## 15. Optimize a React app rendering 10,000 list items

I would first profile the render. The biggest improvement is usually
list virtualization, rendering only visible rows using a library such as
`react-window`. I would also paginate or filter server-side when
appropriate, use stable keys, avoid unnecessary state updates, memoize
expensive children only when helpful, and split large bundles.

## 16. Virtualization, code splitting, `React.lazy` + `Suspense`

Virtualization renders only the visible part of a large list. Code
splitting creates separate bundles so users do not download all features
at startup. `React.lazy` loads a component on demand and `Suspense`
provides a fallback while it loads. They address different bottlenecks:
list rendering versus JavaScript download/initialization.

## 17. Debouncing vs throttling --- implement from scratch

Debouncing runs a function after calls stop for a specified delay;
throttling limits it to at most once per interval. Debounce suits search
input; throttle suits scroll/resize or frequent pointer events.

``` js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

function throttle(fn, delay) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= delay) {
      last = now;
      fn(...args);
    }
  };
}
```

A production implementation may need cancellation, preserving `this`,
trailing calls, and cleanup.

## 18. When does debounce hurt UX?

Debounce delays execution until input stops, so it can make direct
actions such as form submission or a critical button feel unresponsive.
I generally debounce noisy input-driven work like search suggestions,
but I do not delay explicit user actions unnecessarily. Submit handlers
should also guard against duplicate submissions.

## 19. Image optimization, CDN, and state management optimization

I use appropriately sized responsive images, modern formats, lazy
loading below the fold, and caching headers. A CDN serves static assets
closer to users. For state, I keep state near where it is used, avoid
unnecessarily broad context updates, select only required Redux fields,
and normalize large collections when appropriate. I measure bundle size
and render performance.

## 20. SSR vs CSR vs SSG

-   **CSR:** the browser renders the UI from JavaScript; useful for
    highly interactive apps but may delay initial content.
-   **SSR:** the server renders HTML per request; useful for dynamic
    pages and initial content/SEO, but adds server work.
-   **SSG:** HTML is generated at build time; fast for mostly static
    pages, but data can become stale until regenerated. Frameworks such
    as Next.js support combinations based on route and data needs.

## 21. What is hydration?

Hydration is when client-side JavaScript attaches React behavior and
event handlers to HTML that was already rendered on the server. Server
and client output must match closely; mismatches can cause warnings or
extra work. Hydration is different from rendering the page from scratch
in the browser.

## 22. How does Next.js mix SSR, CSR, and SSG?

Next.js supports static generation, server rendering, client components,
and data revalidation patterns. A page can deliver pre-rendered HTML for
initial display and then hydrate interactive client components. The
exact options depend on whether the project uses the App Router or Pages
Router and the relevant Next.js version.

------------------------------------------------------------------------

# 11. JavaScript Fundamentals

## 1. How does asynchronous processing work in JavaScript?

JavaScript typically executes application code on a single main thread,
while the host environment handles operations such as timers and network
I/O. When those operations complete, callbacks or promise reactions are
queued for execution. Async APIs let code avoid blocking the main thread
while waiting for external work.

## 2. Explain the Event Loop

The call stack runs synchronous JavaScript. Browser Web APIs or the
runtime handle supported asynchronous operations. When ready, callbacks
are queued. Promise reactions go to the microtask queue, which is
generally drained before the next task such as a timer callback. The
event loop coordinates when queued work runs; exact details vary by host
environment.

## 3. Output: `console.log(1); setTimeout(...2...); Promise.resolve().then(...3...)`

Output:

``` text
1
3
2
```

The synchronous log runs first. The promise callback is a microtask and
runs before the timer task. `setTimeout(..., 0)` means eligible after a
minimum delay, not immediately.

## 4. Promises vs Async/Await

A Promise represents eventual success or failure and can be composed
with `.then()` and `.catch()`. `async/await` is syntax built on promises
that makes asynchronous control flow read more like synchronous code. I
use `try/catch` around awaited operations and `Promise.all()` for
independent tasks that should run concurrently.

## 5. Explain Closures with a practical example

A closure is a function together with access to variables from its
lexical scope, even after the outer function returns:

``` js
function createCounter() {
  let count = 0;
  return () => ++count;
}
const next = createCounter();
next(); // 1
next(); // 2
```

The returned function retains access to `count`, which is useful for
encapsulating state and creating callbacks.

## 6. Classic `var` vs `let` closure problem in loops

`var` is function-scoped, so callbacks created in a loop share the same
binding; after the loop they may all observe the final value. `let`
creates a new binding for each loop iteration:

``` js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 0, 1, 2
```

## 7. Event Bubbling vs Event Capturing

Capturing travels from the outer ancestors toward the target. Bubbling
travels from the target back up through ancestors. Most event listeners
run in the bubbling phase by default; `{ capture: true }` registers a
capturing listener.

## 8. Event Delegation --- why useful?

Event delegation attaches one listener to a parent and handles events
from its children by inspecting `event.target` or using `closest()`. It
reduces listener count and works for dynamically added children,
provided the event bubbles.

## 9. `stopPropagation()` --- when to use?

It prevents an event from continuing through the propagation path. I use
it only when nested interactions should not trigger ancestor handlers,
because excessive use can make event flows difficult to reason about. It
does not prevent the browser's default action; `preventDefault()` does
that.

## 10. Type coercion gotchas

JavaScript may implicitly convert values based on the operator and
context. For example, `"5" + 1` produces `"51"`, while `"5" - 1`
produces `4`. I prefer explicit conversions and strict equality to avoid
surprising behavior.

## 11. `==` vs `===`

`==` permits type coercion before comparison; `===` compares without
coercing types. I use `===` by default because it is more predictable.
Loose equality has a few intentional cases, such as `value == null`
matching both `null` and `undefined`, but strict comparisons are clearer
for most code.

## 12. `delete` vs `splice`

`delete array[index]` removes the property but leaves a hole and does
not change array length. `array.splice(index, 1)` removes the element
and shifts later elements, changing length. For arrays, use `splice()`
or create a new array with `filter()` when immutability matters.

## 13. `this` in different contexts

In a regular function, `this` depends on how the function is called; in
a method call, it normally refers to the receiver. `call`, `apply`, and
`bind` can set it explicitly. Arrow functions do not have their own
`this`; they capture it lexically. In classes, methods use the instance
as `this` when called as instance methods, but passing an unbound method
as a callback can lose that receiver.

## 14. Common JavaScript output-question approach

I trace synchronous code first, then identify promise microtasks,
timers, and other host callbacks. I check scope (`var` vs `let`),
coercion (`==` vs `===`), `this` binding, mutation/reference behavior,
and evaluation order. I avoid guessing and write down each step.

------------------------------------------------------------------------

# 12. TypeScript

## 1. `type` vs `interface`

Both can describe object shapes. `interface` supports declaration
merging and is often useful for object contracts; `type` can represent
unions, intersections, primitives, tuples, and mapped/conditional types.
For ordinary object contracts, either can work, so I follow the team's
convention and choose `type` when union/composition features are needed.

## 2. When to use `type` vs `interface`?

I use `interface` for extensible object contracts and public shapes that
may be augmented. I use `type` for unions such as `Success | Failure`,
intersections, tuple types, and more complex type expressions. This is a
convention rather than a strict rule.

## 3. Explain Generics with a real example

Generics allow reusable code to preserve the relationship between input
and output types:

``` ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}
const name = first<string>(["Ana", "Raj"]); // string | undefined
```

They provide type safety without falling back to `any`. In a React app,
generics are also useful for reusable tables, API response wrappers, and
hooks.

## 4. Utility types: `Partial`, `Pick`, `Omit`, `Record`

-   `Partial<T>` makes all properties optional, useful for patch/update
    inputs.
-   `Pick<T, K>` selects specified properties.
-   `Omit<T, K>` removes specified properties.
-   `Record<K, V>` maps a key type to a value type.

``` ts
type User = { id: number; name: string; email: string };
type UserPatch = Partial<Pick<User, "name" | "email">>;
type PublicUser = Omit<User, "email">;
type StatusLabels = Record<"open" | "closed", string>;
```

## 5. What are Decorators in TypeScript?

Decorators are functions that can annotate or modify classes and,
depending on the decorator model and compiler configuration, class
members or other declarations. They are used by frameworks and libraries
for metadata and cross-cutting behavior. TypeScript has legacy
experimental decorators and newer standard-aligned decorator support, so
behavior and syntax depend on the compiler/framework version. They are
not the same as Java annotations.

## 6. Implement a Promise polyfill from scratch

In an interview, I would first clarify whether the interviewer wants a
simplified educational implementation or full Promises/A+ compliance. A
complete polyfill must handle pending/fulfilled/rejected states,
asynchronous callback execution, chaining, thenable assimilation,
exception propagation, and static methods such as `resolve`, `reject`,
`all`, and `race`. It is substantially more than a few lines; I would
implement the core state machine and chaining first, then add edge
cases.

## 7. Handle async callbacks, chainability, state, callback queue, static methods

The core design has a state (`pending`, `fulfilled`, `rejected`), a
result value/reason, and queues for fulfillment/rejection handlers.
Settlement is allowed once. `.then()` returns a new promise; its
callback result resolves the new promise, and thrown errors reject it.
If a callback returns a promise/thenable, the implementation must adopt
its eventual state. Handlers should run asynchronously, and
`Promise.all()` must preserve input order while rejecting if one input
rejects.

------------------------------------------------------------------------

# 13. Web Fundamentals & Browser

## 1. Browser components: JS engine, rendering engine, call stack, Web APIs, event loop

The JavaScript engine parses and executes JavaScript and maintains
structures such as the call stack and heap. The rendering engine parses
HTML/CSS and paints the page. Browser APIs provide capabilities such as
timers, network requests, and DOM access. The event loop coordinates
queued tasks and microtasks so asynchronous callbacks can run when the
stack is available.

## 2. How does the browser render a page? (Critical Rendering Path)

The browser parses HTML into the DOM and CSS into the CSSOM, combines
them into a render tree, calculates layout, paints pixels, and
composites layers. JavaScript can block parsing or trigger style/layout
work depending on how it is loaded and used. To improve rendering, I
reduce render-blocking resources, optimize CSS/JS, avoid layout
thrashing, and load non-critical assets later.

## 3. What happens when you type a URL and hit Enter?

The browser parses the URL, checks relevant caches, resolves the domain
through DNS if needed, establishes a connection (TCP and TLS for HTTPS,
with details depending on HTTP version), and sends an HTTP request. The
server or CDN responds; the browser parses the response, fetches
subresources, builds DOM/CSSOM, executes scripts, and renders the page.
Redirects, service workers, caches, and connection reuse can alter the
path.

## 4. `sessionStorage` vs `localStorage` vs cookies

-   `sessionStorage` is scoped to a tab/session and generally cleared
    when that tab session ends.
-   `localStorage` persists across browser restarts until cleared and is
    origin-scoped.
-   Cookies can have expiry, path/domain, `Secure`, `HttpOnly`, and
    `SameSite` attributes and are automatically sent with matching HTTP
    requests. Web storage is accessible to JavaScript, so sensitive
    tokens there are exposed if an XSS vulnerability occurs. `HttpOnly`
    cookies reduce JavaScript access but require CSRF-aware design.

## 5. What is CORS and how does it work?

CORS is a browser-enforced mechanism that lets a server specify which
origins may read its cross-origin responses. The server returns headers
such as `Access-Control-Allow-Origin`. CORS is not authentication and
does not generally prevent a non-browser client from making the request;
it controls whether browser scripts can access the response.

## 6. Preflight request (`OPTIONS`) --- when does it happen?

For requests that are not CORS "simple requests," the browser sends an
`OPTIONS` preflight asking whether the origin, method, and headers are
permitted. The server responds with CORS permission headers; the browser
sends the actual request only if the preflight succeeds. Examples
include many requests using `Authorization`, custom headers, or
non-simple content types.

## 7. HTTP/2 vs HTTP/1.1

HTTP/2 uses a binary framing layer, multiplexes multiple streams over
one connection, compresses headers with HPACK, and allows stream
prioritization mechanisms. HTTP/1.1 commonly uses text-based messages
and may need multiple connections to improve parallelism. HTTP/2 reduces
some overhead, but performance still depends on latency, server
behavior, and resource patterns.

## 8. Multiplexing, header compression, server push

Multiplexing interleaves multiple request/response streams over one
connection. HPACK compresses HTTP/2 headers to reduce repeated header
overhead. HTTP/2 server push was designed to proactively send resources
but has limited practical use and support; it is not a default
optimization I would rely on today. Modern performance work often favors
caching, preload hints, and efficient resource prioritization.

------------------------------------------------------------------------

# 15. DevOps & CI/CD

## 1. How would you add authentication to a REST API?

I would use Spring Security to authenticate requests and enforce
authorization. For a stateless API, a common approach is OAuth 2.0/OIDC
with an identity provider and access tokens; the API validates token
signature, issuer, audience, expiry, and relevant claims. Passwords must
be hashed with a strong adaptive algorithm if the application manages
them. I also use HTTPS, least privilege, validation, and secure secret
storage.

## 2. JWT vs sessions in microservices? Why stateless?

With server-side sessions, the server stores session state and the
client sends a session ID; distributed deployments need shared session
storage or routing strategy. With JWT access tokens, services can
validate signed claims without a central session lookup, improving
distributed validation, but revocation and token-size/claim design
require care. JWTs are not automatically more secure or always stateless
in every architecture. Short-lived access tokens and refresh-token
controls are common.

## 3. How did you use Infrastructure as Code?

A good interview answer, if accurate for your experience: "I use
Infrastructure as Code to define infrastructure in version-controlled
configuration rather than creating resources manually. This makes
environments repeatable, reviewable, and easier to reproduce. I keep
environment-specific values separate, review plans/diffs before applying
changes, and avoid storing secrets in source control." Name Terraform,
CloudFormation, or the actual tool you used, and distinguish hands-on
work from familiarity.

## 4. Biggest risk with IaC? (Drift)

Configuration drift occurs when the real infrastructure differs from the
declared configuration, often because someone changes a resource
manually or another process modifies it. I detect it with plan/drift
checks, restrict manual changes, review state carefully, and bring
changes back into code. State-file protection and locking are also
important for tools such as Terraform.

## 5. Explain your CI/CD pipeline end to end

A typical pipeline starts with a commit or pull request, runs code
checks and unit tests, builds the artifact, runs integration/security
checks, and creates a versioned container image. The image is pushed to
a registry, then deployed to a test/staging environment for validation.
After approval or automated gates, production deployment runs with
health checks and monitoring. I keep artifacts immutable and define
rollback procedures.

## 6. What if a production deployment breaks?

I first assess impact and use metrics/logs to confirm whether the
deployment caused it. If the release is unhealthy, I stop further
rollout and roll back to the last known-good version or shift traffic
back, depending on the deployment strategy. Then I preserve logs,
identify the root cause, verify recovery, and add a regression test or
deployment guard. Database changes need backward-compatible migration
and rollback planning.

## 7. Blue/Green vs Canary deployment

Blue/green maintains two environments: traffic switches from the old
environment to the new one, allowing fast rollback but requiring
duplicate capacity and careful data compatibility. Canary sends a small
percentage of traffic to the new version, then increases it as metrics
remain healthy. Canary reduces exposure but needs reliable routing,
metrics, and automated rollout decisions.

## 8. Which tools do you use for deployment?

Answer with tools actually used. A suitable structure is: "We build and
test through \[CI tool\], package the application as a Docker image,
store it in \[registry\], and deploy to \[ECS/Kubernetes\].
Configuration and secrets are managed through \[actual tools\]. We
monitor with \[CloudWatch/Splunk/etc.\] and use health checks and
rollback procedures." Do not claim hands-on experience with a tool you
have only studied.

## 9. How are Kubernetes deployments done?

I define a Deployment with the container image, replicas, resource
requests/limits, environment configuration, and probes. Kubernetes
schedules Pods, maintains the desired replica count, and updates them
according to the rollout strategy. A Service gives stable network access
to Pods. Secrets/configuration, autoscaling, ingress, and observability
are configured according to the environment. I validate with rollout
status, logs, events, and health metrics.

## 10. Rolling update strategy and readiness probes

A rolling update replaces old Pods gradually while maintaining the
configured availability and surge limits. A readiness probe tells
Kubernetes whether a Pod should receive traffic; a failing readiness
probe removes it from ready endpoints but does not itself restart the
container. A liveness probe can trigger a restart, and a startup probe
gives slow-starting apps more time. Probes should check meaningful
health without being overly sensitive.

------------------------------------------------------------------------

# 16. DSA --- Scenario-Based Questions

## 1. Gaming platform: 1M users. Check whether a username is taken instantly.

I would enforce a unique constraint on a normalized username in the
authoritative database. The UI can call an availability endpoint while
the user types, with debouncing to avoid excessive requests. A cache may
speed up common lookups, but it cannot guarantee uniqueness because two
users can race; the final registration must rely on an atomic database
constraint and handle duplicate-key errors.

## 2. Build cache: max 10K items, evict LRU, all operations O(1)

Use a `HashMap` for key-to-node lookup and a doubly linked list to track
recency. On `get`, move the node to the head. On `put`, update or insert
at the head; if capacity is exceeded, remove the tail and its map entry.
Hash map lookup and linked-list pointer updates are O(1) on average.

## 3. Stock app: thousands of price updates/sec, show top 10 most expensive

I would separate the high-frequency update path from the query path.
Keep the latest price per symbol in a map and maintain a min-heap of
size 10, with lazy invalidation/version checks or an indexed heap to
handle price updates correctly. A naïve heap with stale entries can grow
and return outdated prices, so the implementation must handle updates
and removals carefully. For distributed scale, partition by symbol and
consider a stream-processing aggregation.

## 4. Browser history with back and forward navigation

Use two stacks: a back stack and a forward stack, plus the current page.
On navigation, push the current page to the back stack and clear the
forward stack. On back, push the current page to forward and pop back
into current. On forward, do the reverse. Each navigation operation is
O(1) amortized with stack operations.

## 5. Implement a Queue using Stacks

Use an input stack and an output stack. Enqueue pushes to input. Dequeue
pops from output; if output is empty, transfer all elements from input
to output first. Each element moves at most once between stacks, so
enqueue is O(1) and dequeue is amortized O(1).

``` java
class MyQueue {
    private final Deque<Integer> in = new ArrayDeque<>();
    private final Deque<Integer> out = new ArrayDeque<>();

    void offer(int value) { in.push(value); }

    int poll() {
        if (out.isEmpty()) {
            while (!in.isEmpty()) out.push(in.pop());
        }
        if (out.isEmpty()) throw new NoSuchElementException();
        return out.pop();
    }
}
```

------------------------------------------------------------------------

# 17. Behavioral / Experience-Based

> These should be adapted to your actual work. The examples below are
> structured prompts, not claims that you personally performed every
> action. Use only details you can defend in follow-up questions.

## 1. Tell me about a production issue you debugged

**Answer structure:** "We observed \[customer/business impact\]. I
checked logs, metrics, and traces to narrow it down to \[specific
component or query\]. I reproduced or validated the hypothesis,
identified \[root cause\], and made \[fix\]. I then monitored the
service and added \[test/alert/runbook\] to reduce recurrence." Be
specific about the symptom, your own actions, and measurable outcome; do
not stop at "I checked logs."

## 2. Give an example of performance optimization you did

"First I established a baseline using \[latency/throughput/query
metrics\]. I found the main cost was \[N+1 queries, repeated
computation, unnecessary network calls, oversized payload, etc.\]. I
changed \[specific implementation\], then compared the same metrics
under similar load. The result was \[measured improvement\], with
correctness verified by \[tests/monitoring\]." Use actual numbers only
if you have them.

## 3. How did you reduce API latency in a real project?

"I traced the request end to end and separated application, database,
and downstream latency. I optimized the dominant part rather than
guessing---for example, reducing database round trips, selecting only
required fields, parallelizing independent calls, or caching stable
data. I added regression measurements and verified the effect under
representative traffic." Pick the option that genuinely matches your
experience.

## 4. Describe a scalability challenge you faced

"Traffic or data volume increased and exposed a bottleneck in
\[component\]. I used metrics to determine whether the constraint was
CPU, memory, database connections, queue lag, or downstream capacity. I
addressed it through \[specific fix\], added capacity
limits/backpressure where needed, and monitored the result. I also
explain the trade-off and what I would improve next."

## 5. Tell me about a system migration you led

"I defined the scope and compatibility requirements, checked dependency
and runtime changes, and split the migration into testable steps. I
updated code/configuration, ran unit and integration tests, validated
deployment in a lower environment, and planned rollback. I monitored
errors and performance after release. I distinguish what I personally
owned from work done by the wider team."

## 6. Describe your experience with microservices architecture

"My work has involved services that communicate through
\[REST/events/queues---use what is true\]. I pay attention to service
boundaries, API contracts, timeouts, retries, observability, and data
ownership. Since each service can fail independently, I avoid assuming a
remote call is reliable and design for duplicate events and partial
failure where applicable."

## 7. Explain a project using AWS + Kafka + Spring Boot + React

Use a truthful architecture walkthrough: 1. **Business problem:** what
user or business process the system supports. 2. **React:** the screens
and API interactions you implemented. 3. **Spring Boot:** the business
logic, validation, REST APIs, persistence, and security you owned. 4.
**AWS:** name the services you actually used and explain each role. 5.
**Kafka:** explain the topic, event producer/consumer, partition key,
retry/DLQ approach, and idempotency---only if Kafka was actually part of
that project. 6. **Reliability and outcome:** one real failure mode, how
it was handled, and a measurable result. Do not combine technologies
from separate projects into one story unless that is what really
happened.

## 8. Tell me about owning a product end-to-end (PRD to production)

"I clarified the requirement and success criteria, broke it into
API/data/UI work, discussed design trade-offs, and implemented or
coordinated the pieces. I added tests, reviewed edge cases, worked
through deployment readiness, and monitored production behavior. I
describe the decisions I owned and how I handled changes or feedback."
If you did not own the PRD, say you contributed from requirement
clarification through delivery rather than claiming full ownership.

## 9. Describe a time you disagreed with a technical decision

"I explained the concern using evidence such as failure modes,
complexity, performance, or maintenance cost, and proposed an
alternative with trade-offs. I listened to the other viewpoint and
aligned on a decision based on project constraints. After the decision,
I supported execution and monitored the outcome. If new evidence showed
the decision was wrong, I would raise it constructively." Use a real
example and avoid blaming colleagues.

------------------------------------------------------------------------

# Quick interview reminders

-   Answer in this order: **definition → how it works → when to use it →
    trade-off/example**.
-   Be precise about guarantees: `volatile` is not atomic for compound
    updates; Kafka exactly-once has a defined scope; `@Transactional`
    does not span arbitrary remote calls.
-   State assumptions before system-specific answers, especially for
    database engines, AWS quotas, and framework versions.
-   For experience questions, use **Situation → Task → Action → Result**
    and quantify results only when you can support the numbers.
-   If you have only studied a technology, say "I understand the concept
    and have practiced it" rather than claiming production experience.
