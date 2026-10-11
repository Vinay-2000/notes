# 1. Resilience4j

## 1.1 What is Resilience4j?

Resilience4j is a lightweight fault-tolerance library used in Spring
Boot applications to make service-to-service and external API calls more
resilient. It helps prevent cascading failures and supports controlled
recovery when dependencies are slow or unavailable.

### Core modules

  Feature           Purpose
  ----------------- -------------------------------------------------------------
  Circuit Breaker   Stops calls to an unhealthy dependency temporarily
  Retry             Retries operations that fail, often due to transient errors
  Rate Limiter      Limits the number of calls within a time period
  Bulkhead          Limits concurrent executions to isolate resource usage
  Time Limiter      Limits how long an operation may take

## 1.2 Dependencies

For Spring Boot 3, a typical Maven setup is:

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.3.0</version>
</dependency>
```

Use a version compatible with your Spring Boot and Java versions; in a
real project, prefer the version managed by your dependency-management
strategy.

## 1.3 Circuit Breaker

A Circuit Breaker stops repeatedly calling a dependency that is failing.
This can prevent wasted work and cascading failures.

### States

-   **CLOSED:** Calls are allowed; failures are monitored.
-   **OPEN:** Calls are rejected without calling the dependency.
-   **HALF_OPEN:** A limited number of trial calls are allowed to check
    whether the dependency has recovered.

### Configuration --- `application.yml`

``` yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 5
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
```

-   `slidingWindowSize: 5`: evaluate the configured sliding window of
    five calls.
-   `minimumNumberOfCalls: 5`: collect at least five calls before
    evaluating the failure-rate threshold.
-   `failureRateThreshold: 50`: open the circuit when the failure rate
    reaches 50%.
-   `waitDurationInOpenState: 10s`: wait 10 seconds before transitioning
    to half-open.

### Code

``` java
@Service
public class PaymentService {

    @CircuitBreaker(
        name = "paymentService",
        fallbackMethod = "paymentFallback"
    )
    public String processPayment() {
        // Call an external payment API
        return callPaymentApi();
    }

    private String callPaymentApi() {
        // Placeholder for HTTP client call
        throw new RuntimeException("Payment service unavailable");
    }

    public String paymentFallback(Throwable ex) {
        return "Payment service unavailable";
    }
}
```

A fallback should not falsely report that a critical operation such as a
payment succeeded. Preserve the failure status and use
idempotency/reconciliation where needed.

## 1.4 Retry

Retry repeats an operation after a failure. It is most useful for
transient failures, such as temporary network errors. Avoid blindly
retrying non-idempotent operations.

``` yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1s
```

``` java
@Retry(name = "paymentService")
public String callPaymentService() {
    return paymentClient.call();
}
```

`maxAttempts: 3` means up to three attempts total, including the initial
call.

## 1.5 Rate Limiter

A Rate Limiter restricts how many calls are permitted during a
configured period.

``` yaml
resilience4j:
  ratelimiter:
    instances:
      paymentService:
        limitForPeriod: 5
        limitRefreshPeriod: 10s
        timeoutDuration: 0
```

``` java
@RateLimiter(name = "paymentService")
public String processPayment() {
    return paymentClient.call();
}
```

-   `limitForPeriod: 5`: allow five permissions per refresh period.
-   `limitRefreshPeriod: 10s`: refresh the permissions every ten
    seconds.
-   `timeoutDuration: 0`: do not wait for a permission; reject
    immediately if unavailable.

If the limit is exceeded, Resilience4j throws `RequestNotPermitted`.
Handle this appropriately, often by returning HTTP
`429 Too Many Requests` when the limit represents client request
throttling.

**Interview distinction:** Resilience4j's in-memory Rate Limiter is
generally local to each application instance. It is not automatically a
globally shared limit across all instances.

## 1.6 Bulkhead

Bulkhead limits concurrent executions so that a slow dependency cannot
consume all available resources.

``` yaml
resilience4j:
  bulkhead:
    instances:
      paymentService:
        maxConcurrentCalls: 5
        maxWaitDuration: 0
```

``` java
@Bulkhead(name = "paymentService")
public String processPayment() {
    return paymentClient.call();
}
```

-   `maxConcurrentCalls: 5`: at most five concurrent executions.
-   `maxWaitDuration: 0`: do not wait for a slot; reject immediately
    when full.
-   Excess calls can fail with `BulkheadFullException`.

Resilience4j supports a semaphore bulkhead (the default) and a
thread-pool bulkhead. A thread-pool bulkhead uses a separate thread pool
and queue.

## 1.7 Time Limiter

A Time Limiter restricts how long an asynchronous operation is allowed
to run. Its practical use depends on the execution model and integration
being used.

``` yaml
resilience4j:
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 2s
        cancelRunningFuture: true
```

Time Limiter commonly applies to `Future`/`CompletionStage`-based
operations. For synchronous HTTP calls, also configure a connect timeout
and response/read timeout in the HTTP client; a Time Limiter annotation
alone is not a substitute for client-level timeouts.

## 1.8 Combining Resilience4j features

``` java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
@Retry(name = "paymentService")
public String processPayment() {
    return paymentClient.call();
}

public String fallback(Throwable ex) {
    return "Payment service unavailable";
}
```

When using multiple annotations, aspect execution order affects which
failures are retried and which are observed by the Circuit Breaker.
Configure and test the intended behavior rather than assuming a
particular order.

### Quick comparison

  Feature           Controls
  ----------------- ----------------------------------------------------
  Circuit Breaker   Calls based on observed failures and circuit state
  Retry             Number of attempts after failures
  Rate Limiter      Calls per time period
  Bulkhead          Concurrent executions
  Time Limiter      Duration of asynchronous operations

### Interview answer

> I use Resilience4j in Spring Boot microservices to handle failures
> during calls to downstream services. Circuit Breaker stops repeated
> calls to an unhealthy dependency, Retry handles transient failures,
> Rate Limiter controls request volume, Bulkhead isolates concurrent
> executions, and Time Limiter restricts asynchronous operation
> duration. I configure these through `application.yml` and use
> annotations where appropriate. I also configure HTTP client timeouts
> and make sure retries are safe for the operation.

------------------------------------------------------------------------

# 2. Redis

## 2.1 What is Redis?

Redis is an in-memory data store commonly used for caching, counters,
sessions, messaging, and distributed coordination. It reduces latency
and database load for frequently accessed data.

Redis is not automatically a replacement for a relational database.
Durability depends on persistence configuration, and cached data may
expire or be evicted.

## 2.2 Common data structures

  Structure    Typical use
  ------------ --------------------------------
  String       Values, tokens, counters
  Hash         Object-like fields
  List         Ordered collections and queues
  Set          Unique values
  Sorted Set   Rankings and leaderboards
  Stream       Event logs and consumer groups

### Useful commands

``` bash
SET employee:101 Vinay
GET employee:101

SET employee:101 Vinay EX 300
TTL employee:101

HSET employee:101 name Vinay salary 70000
HGET employee:101 name
HGETALL employee:101

INCR page:views
DEL employee:101

SADD active:users 101 102
SISMEMBER active:users 101

ZADD leaderboard 100 Vinay
ZREVRANGE leaderboard 0 9 WITHSCORES
```

`TTL` returns remaining seconds. `-1` means the key has no expiry; `-2`
means the key does not exist.

## 2.3 TTL and eviction

**TTL (Time To Live)** defines how long a key should remain before
expiring.

``` bash
SET product:101 Laptop EX 300
```

The key expires after 300 seconds.

When Redis reaches its configured memory limit, `maxmemory-policy`
controls eviction behavior.

  -----------------------------------------------------------------------
  Policy                              Meaning
  ----------------------------------- -----------------------------------
  `noeviction`                        Reject memory-consuming writes
                                      instead of evicting keys

  `allkeys-lru`                       Evict approximately
                                      least-recently-used keys across all
                                      keys

  `volatile-lru`                      Evict approximately
                                      least-recently-used keys with an
                                      expiry

  `allkeys-lfu`                       Evict keys with low access
                                      frequency across all keys

  `volatile-ttl`                      Prefer keys with the shortest
                                      remaining TTL
  -----------------------------------------------------------------------

-   **LRU:** Focuses on recency of access.
-   **LFU:** Focuses on frequency of access.

LRU/LFU are eviction policies; they are not cache interaction patterns.

## 2.4 Caching patterns

### Cache-aside (lazy loading)

1.  Check Redis.
2.  On a hit, return the cached value.
3.  On a miss, query the database.
4.  Store the result in Redis.
5.  Return the result.

Advantages: simple, and only requested data is cached.\
Disadvantages: first request is slower, cached data can become stale,
and concurrent misses can cause a database spike.

### Write-through

The write path updates the database and cache as part of the same
logical operation. This can keep cached data fresh but increases
write-path complexity and latency.

Redis and Oracle do not automatically share one atomic transaction; the
application must coordinate consistency.

### Write-behind (write-back)

The cache is updated first and the database is updated asynchronously.
This can reduce write latency, but introduces data-loss and recovery
risks if pending updates are not handled durably.

### Read-through

The caching layer itself loads missing values from the underlying store.
With cache-aside, the application owns the load-on-miss logic.

  Pattern         Main idea
  --------------- --------------------------------------------
  Cache-aside     Application checks cache and loads on miss
  Read-through    Cache layer loads on miss
  Write-through   Cache and DB updated in the write path
  Write-behind    DB write is deferred asynchronously

## 2.5 Spring Boot Redis setup

Add the Spring Data Redis starter:

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

Example local Redis server:

``` bash
docker run -d --name redis -p 6379:6379 redis:7
```

`application.yml`:

``` yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```

For production, configure authentication, TLS where appropriate, network
restrictions, and secrets management.

## 2.6 `@Cacheable` on a normal service method

`@Cacheable` works on any suitable Spring-managed method, not only
repository methods.

Enable caching:

``` java
@SpringBootApplication
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Service:

``` java
@Service
public class EmployeeService {

    @Cacheable(value = "employees", key = "#id")
    public String getEmployeeName(int id) {
        System.out.println("Executing method...");
        return "Employee-" + id;
    }
}
```

Calling:

``` java
employeeService.getEmployeeName(101);
employeeService.getEmployeeName(101);
employeeService.getEmployeeName(102);
```

Output:

``` text
Executing method...
Executing method...
```

Explanation: - First call for `101`: cache miss, method executes and
result is cached. - Second call for `101`: cache hit, method is
skipped. - Call for `102`: cache miss, method executes because the key
differs.

`value = "employees"` is the cache name; `key = "#id"` uses the argument
as the cache key.

**Important:** This example assumes Redis is configured as the Spring
Cache provider. Without explicit configuration, Spring Boot may use
another cache provider depending on the dependencies and setup.

Spring caching usually works through AOP proxies. A call from one method
to another method in the same class (self-invocation) normally bypasses
the caching advice.

## 2.7 `@CachePut` and `@CacheEvict`

### `@CachePut`

Always executes the method and updates the cache with the returned
value.

``` java
@CachePut(value = "employees", key = "#result.id")
public Employee updateEmployee(Employee employee) {
    return repository.save(employee);
}
```

Use this when the method return value should refresh the cache. This
example assumes the returned `Employee` has an `id` property.

### `@CacheEvict`

Removes a cache entry.

``` java
@CacheEvict(value = "employees", key = "#id")
public void deleteEmployee(Long id) {
    repository.deleteById(id);
}
```

Clear all entries in a cache:

``` java
@CacheEvict(value = "employees", allEntries = true)
public void clearEmployeeCache() {
}
```

  -----------------------------------------------------------------------
  Annotation                          Behavior
  ----------------------------------- -----------------------------------
  `@Cacheable`                        Return cached result on hit;
                                      execute and cache on miss

  `@CachePut`                         Always execute and update cache

  `@CacheEvict`                       Remove cached entries

  `@EnableCaching`                    Enable Spring caching support
  -----------------------------------------------------------------------

For database updates, coordinate eviction or refresh with transaction
completion. A database commit and Redis update are not automatically
atomic together.

## 2.8 `RedisTemplate`

Use `RedisTemplate` for direct Redis operations and data structures.

``` java
@Service
public class EmployeeRedisService {

    private final RedisTemplate<String, Employee> redis;

    public EmployeeRedisService(
            RedisTemplate<String, Employee> redis) {
        this.redis = redis;
    }

    public void save(Employee employee) {
        redis.opsForValue().set(
            "employee:" + employee.getId(),
            employee,
            Duration.ofMinutes(10));
    }

    public Employee get(Long id) {
        return redis.opsForValue().get("employee:" + id);
    }

    public void delete(Long id) {
        redis.delete("employee:" + id);
    }
}
```

This assumes a `RedisTemplate<String, Employee>` bean has been
configured with appropriate serializers.

Common operations: - `opsForValue()` --- string-like values -
`opsForHash()` --- hashes - `opsForList()` --- lists - `opsForSet()` ---
sets - `opsForZSet()` --- sorted sets - `delete()` --- remove keys

Use Spring Cache annotations for straightforward method-level caching.
Use `RedisTemplate` when you need direct control over commands,
counters, data structures, or serialization.

## 2.9 Serialization

Redis stores bytes. Java objects must be serialized before storing and
deserialized when retrieved.

Common choices: - Strings for simple values. - JSON serializers for
objects. - Explicit serializers for other representations.

Prefer predictable, compatible serialization. Be cautious with
unrestricted Java native deserialization, especially with untrusted
data.

## 2.10 Cache invalidation and consistency

Suppose an employee's salary changes in Oracle but Redis still has the
old value.

Common strategies: 1. TTL: expire the value after a defined period. 2.
Invalidation: remove the key after a successful DB update. 3. Refresh:
update the cached value after a successful DB update.

If the DB update succeeds but Redis invalidation fails, stale data may
remain. Options include retrying invalidation, using TTL as a safety
net, publishing reliable invalidation events, or using an outbox pattern
for important changes.

Do not assume that a normal `@CacheEvict` call automatically happens
after the database transaction commits. Configure transaction-aware
behavior or explicitly coordinate eviction with transaction completion
when correctness requires it.

## 2.11 Cache stampede, penetration, and avalanche

-   **Stampede:** Many requests miss the same expired key at once,
    creating a database spike. Mitigate with request coalescing, per-key
    locking, TTL jitter, or refreshing popular keys early.
-   **Penetration:** Requests for nonexistent keys repeatedly hit the
    database. Mitigate with validation, short-lived negative caching, or
    a Bloom filter for suitable large keyspaces.
-   **Avalanche:** Many keys expire around the same time. Mitigate with
    randomized TTL jitter, staggered expiration, and cache warming.

## 2.12 Distributed locking

A basic Redis lock acquisition command:

``` bash
SET lock:job-101 unique-token NX PX 10000
```

-   `NX`: set only if the key does not exist.
-   `PX 10000`: expire after 10 seconds.
-   `unique-token`: identifies the lock owner.

Release the lock only if its stored token matches yours. Otherwise, a
previous owner could accidentally delete a lock acquired by another
process after expiry. Production distributed locking also needs careful
handling of lease expiry, process pauses, and fencing where stale owners
could cause damage.

## 2.13 Persistence, replication, Sentinel, and Cluster

### Persistence

-   **RDB:** Periodic snapshots.
-   **AOF:** Logs write operations for replay during recovery.

Durability depends on configuration. A cache that can be rebuilt may
tolerate data loss; important persistent state needs a deliberate
recovery plan.

### Replication

Maintains replicas of a primary. It can support redundancy and read
scaling, but replication is generally asynchronous, so recent writes can
potentially be lost during failures.

### Sentinel

Monitors primary/replica deployments and coordinates failover. It does
not provide data sharding.

### Cluster

Redis Cluster distributes keys across multiple primary nodes using
**16,384 hash slots**. It provides sharding, replication, and failover.
Multi-key operations have restrictions when keys map to different slots.

**Sentinel vs Cluster:** Sentinel provides monitoring and failover for a
non-sharded primary/replica setup; Cluster adds sharding.

## 2.14 Pub/Sub vs Streams

-   **Pub/Sub:** Transient messages delivered to active subscribers.
    Disconnected subscribers normally miss messages.
-   **Streams:** Retained event records with IDs, consumer groups, and
    acknowledgments, subject to retention/trimming settings.

Use Pub/Sub for transient broadcasts. Use Streams when you need retained
messages and consumer processing within Redis. Kafka is often a better
fit for large-scale durable event streaming and long-term replay.

## 2.15 Redis vs related technologies

  -----------------------------------------------------------------------
  Comparison                          Difference
  ----------------------------------- -----------------------------------
  Redis vs Oracle                     Redis is commonly used for
                                      low-latency in-memory access;
                                      Oracle provides relational storage
                                      and SQL transactions

  Redis vs local cache                Redis can share data across
                                      application instances; local cache
                                      is process-local

  Redis vs Kafka                      Redis supports caching and multiple
                                      data structures/messaging patterns;
                                      Kafka specializes in durable
                                      distributed event streaming

  Redis vs Memcached                  Both support caching; Redis offers
                                      richer data structures and
                                      persistence/replication options
  -----------------------------------------------------------------------

## 2.16 Rapid-fire interview questions

**Is Redis single-threaded?**\
Command execution has traditionally been primarily single-threaded,
simplifying command-level atomicity. Modern Redis can use additional
threads for I/O and background work, depending on version and
configuration.

**Why is Redis fast?**\
It primarily serves data from memory and uses efficient data structures
and a low-overhead request-processing model.

**What happens if Redis goes down?**\
Cache operations may fail. The application needs timeouts, error
handling, and an explicit fallback policy. Unbounded DB fallback can
overload the database.

**What is cache hit ratio?**

`Hit ratio = hits / (hits + misses) × 100`

A higher hit ratio generally means fewer database lookups, but the
target depends on the workload.

**Can Redis be used for distributed rate limiting?**\
Yes. Shared counters and atomic scripts can coordinate limits across
application instances. Resilience4j's standard in-memory Rate Limiter is
local to an instance.

------------------------------------------------------------------------

# 3. Final revision checklist

## Resilience4j

-   [ ] Explain Circuit Breaker states: CLOSED, OPEN, HALF_OPEN.
-   [ ] Explain Retry and why retries must be safe for the operation.
-   [ ] Distinguish Rate Limiter from Bulkhead.
-   [ ] Know Time Limiter's asynchronous focus and HTTP client timeouts.
-   [ ] Explain fallbacks and annotation interaction.

## Redis

-   [ ] Explain data structures, TTL, and eviction policies.
-   [ ] Compare LRU vs LFU.
-   [ ] Compare cache-aside, read-through, write-through, and
    write-behind.
-   [ ] Write a simple `@Cacheable` service method.
-   [ ] Explain `@CachePut`, `@CacheEvict`, and `RedisTemplate`.
-   [ ] Explain cache invalidation and DB/cache consistency.
-   [ ] Distinguish stampede, penetration, and avalanche.
-   [ ] Explain distributed locking, persistence, replication, Sentinel,
    and Cluster.
