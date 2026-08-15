# Database Scaling: Read Replicas, Sharding & Consistent Hashing

> Notes focused on concepts that apply across common databases, with Oracle as the main reference point.

## 1. Why do databases need scaling?

A single database can eventually become a bottleneck because of:

- Too many reads
- Too many writes
- CPU/memory pressure
- Storage limits
- Connection limits
- Very large datasets
- High availability requirements

There are two major scaling ideas to understand:

```text
Replication → make copies of data
Sharding    → split data across databases
```

They solve different problems and can be combined.

---

# 2. Read Replicas / Replication

Replication means maintaining copies of database data on other database instances.

```text
                Primary
              /         \
             ↓           ↓
        Replica 1    Replica 2
          READ          READ
```

The primary usually handles writes:

```text
INSERT
UPDATE
DELETE
   ↓
Primary
```

Reads can be distributed:

```text
SELECT
  ↓
Replica 1 / Replica 2
```

## Why use replication?

### Read scaling

If the application has far more reads than writes:

```text
100,000 reads/sec
10,000 writes/sec
```

Instead of forcing the primary to handle everything, reads can be distributed across replicas.

### High availability

A replica/standby can potentially take over if the primary fails, depending on the architecture.

### Disaster recovery

A geographically separate replica can provide another copy of the data.

---

# 3. Is replication synchronous or asynchronous?

It depends on the database and configuration.

## Asynchronous replication

The primary doesn't wait for the replica to receive/apply the change before committing.

```text
Primary
   │
   │ change
   ↓
commit

        ─────────→ Replica
```

There can be replication lag:

```text
Primary  → transaction 105
Replica  → transaction 103
```

This is common when maximizing performance.

## Synchronous replication

The primary waits for the required acknowledgement from another database before committing, depending on the configuration.

```text
Primary
   │
   │ redo/change
   ↓
Replica
   │
   │ acknowledgement
   ↓
Primary commits
```

This provides stronger protection against losing recently committed data, but can introduce latency and dependency on network/replica availability.

---

# 4. How common databases implement replication

The concept is the same, but the implementation and terminology differ.

| Database | Common replication technology/concept |
|---|---|
| Oracle | Data Guard / Active Data Guard |
| PostgreSQL | Streaming Replication |
| MySQL | MySQL Replication |
| SQL Server | Always On Availability Groups, replication and other mechanisms |
| MongoDB | Replica Sets |

## Oracle example

Oracle uses **Data Guard** for standby databases.

```text
                  Primary
                Oracle DB
                    │
                Redo data
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      Standby 1           Standby 2
```

With **Active Data Guard**, a physical standby can be open read-only while redo is being applied.

This is conceptually similar to a read replica, although the implementation is Oracle-specific.

Oracle supports both synchronous and asynchronous redo transport depending on configuration/protection mode.

---

# 5. Replication is NOT Sharding

This distinction is extremely important.

## Replication

Make copies:

```text
Primary
 ├── Replica 1
 └── Replica 2
```

Each replica contains the same logical dataset.

Main goals:

- Read scaling
- High availability
- Disaster recovery

## Sharding

Split the dataset:

```text
             Database
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
    Shard 1  Shard 2  Shard 3
```

Each shard contains only part of the data.

Main goals:

- Scale storage
- Scale write throughput
- Scale CPU/I/O
- Prevent one database from becoming a bottleneck

---

# 6. Sharding example

Suppose we have 1 billion users.

Instead of storing all users in one database:

```text
DB
└── 1 billion users
```

we can distribute them:

```text
Shard 1 → users 1 - 300M
Shard 2 → users 300M - 600M
Shard 3 → users 600M - 1B
```

The application needs to know which shard owns a user's data.

This creates the **routing problem**.

---

# 7. Where does the routing happen?

There is no single universal answer.

The routing can be implemented in:

### A. Application

```text
Spring Boot
    ↓
Shard Router
    ↓
DB-1 / DB-2 / DB-3
```

The application calculates the destination shard.

### B. Dedicated routing/proxy layer

```text
Application
     ↓
Sharding / Routing Layer
     ↓
┌────┼────┐
↓    ↓    ↓
DB1  DB2  DB3
```

This is useful because the application doesn't need to know the physical database topology.

### C. Database/platform itself

Some distributed databases provide sharding and routing as built-in functionality.

Examples include MongoDB's sharded architecture and Oracle's distributed/sharding capabilities.

---

# 8. How does the router know where data belongs?

You need a **shard key**.

For example:

```text
user_id
```

Then you can use a strategy such as:

```text
user_id
   ↓
hash(user_id)
   ↓
shard
```

This is called **hash-based sharding**.

---

# 9. Hash-based sharding vs consistent hashing

These are related but NOT identical concepts.

## Simple hash/modulo sharding

For example:

```text
shard = hash(user_id) % N
```

With 3 shards:

```text
hash(user_id) % 3
```

You get:

```text
User 101 → Shard 2
User 102 → Shard 1
User 103 → Shard 0
```

### Problem when adding a shard

Suppose we add a fourth shard:

```text
hash(user_id) % 4
```

Now many users that previously mapped to one shard will map somewhere else.

Potentially a huge amount of data needs to move.

---

# 10. Consistent Hashing

Consistent hashing is a technique designed to minimize data movement when nodes are added or removed.

Instead of:

```text
hash(key) % N
```

we imagine a hash ring.

```text
                 Shard A
                    ●
              ┌───────────┐
           ●                 ●
       Shard D             Shard B
              └───────────┘
                    ●
                 Shard C
```

Both the data keys and the shards are mapped onto the same hash space.

For example:

```text
user_id
   ↓
hash(user_id)
   ↓
position on hash ring
   ↓
next shard
```

So:

```text
User 12345
    ↓
hash(12345)
    ↓
Hash Ring
    ↓
Shard B
```

---

# 11. Why consistent hashing helps

Suppose we have:

```text
Shard A
Shard B
Shard C
```

and add:

```text
Shard D
```

With simple modulo hashing, many keys may change destination.

With consistent hashing, only keys in the affected portion of the ring generally need to move.

```text
Before:

A ───── B ───── C

After:

A ─── D ─── B ───── C
      ↑
  affected area
```

This is the major advantage.

> Consistent hashing minimizes the amount of data that needs to be remapped when nodes are added or removed.

---

# 12. Where does the hash ring actually live?

This is a very common interview follow-up.

The hash ring is part of the **routing/sharding layer**.

It does NOT have to be inside the database.

For a custom architecture:

```text
Spring Boot
     ↓
Shard Router
     ↓
Consistent Hash Ring
     ↓
┌────┼────┐
↓    ↓    ↓
DB1  DB2  DB3
```

The routing layer maintains information such as:

```text
Shard 1 → database connection
Shard 2 → database connection
Shard 3 → database connection
```

and the hash ring maps keys to those shards.

The topology/configuration may be stored in:

- Application configuration
- Service discovery
- A metadata/configuration store
- A dedicated routing service
- A database/platform's own topology system

The exact implementation depends on the architecture.

---

# 13. Real-world example: YouTube / Vitess

A very useful real-world example is **YouTube's use of Vitess**.

Vitess is a database infrastructure layer originally developed at YouTube and now widely used for scaling MySQL.

The important architecture is:

```text
Application
     ↓
   VTGate
     ↓
Sharding / routing
     ↓
┌────┼────┐
↓    ↓    ↓
MySQL MySQL MySQL
Shard  Shard  Shard
```

The application does not need to directly choose:

```text
"Connect to MySQL shard 7"
```

Instead it sends the query to **VTGate**.

VTGate determines which shard(s) need to receive the query.

---

# 14. How Vitess knows where data belongs

Vitess uses concepts such as:

- VSchema
- Vindexes
- Keyspaces
- Shards

A simplified flow:

```text
Query:

SELECT *
FROM users
WHERE user_id = 12345;

              ↓

           VTGate
              ↓
         VSchema/Vindex
              ↓
         Hash user_id
              ↓
        Keyspace ID
              ↓
        Shard / Keyrange
              ↓
          MySQL shard
```

The application can therefore remain relatively unaware of the physical shard topology.

---

# 15. What happens when Vitess needs to reshard?

Suppose we have:

```text
Before:

Shard A
Shard B
```

and want:

```text
After:

Shard A
Shard B
Shard C
Shard D
```

A sharding system such as Vitess can perform resharding by migrating/splitting data and then changing the serving topology.

Conceptually:

```text
Old shard
    │
    ├────→ New shard
    │
    └────→ New shard
```

Then routing changes:

```text
Before:

users → Shard A

After:

some users → Shard A
some users → Shard C
some users → Shard D
```

The application doesn't need to hard-code every physical database location.

---

# 16. Another real-world example: Slack

Slack has used sharded MySQL architectures.

A simplified model is:

```text
Slack application
       ↓
workspace_id
       ↓
Shard metadata
       ↓
Shard number
       ↓
MySQL shard
```

For example:

```text
workspace 123
     ↓
metadata lookup
     ↓
Shard 17
     ↓
MySQL
```

This demonstrates an important point:

> The routing layer doesn't have to be a separate proxy.

It can be part of the application's database infrastructure.

---

# 17. Etsy and Vitess

Etsy is another useful example of large-scale MySQL sharding.

Their database infrastructure has used a large number of shards, and they have been moving their sharding/routing infrastructure toward Vitess.

The architecture is conceptually:

```text
Etsy application
       ↓
Vitess
       ↓
Vindexes / routing
       ↓
MySQL shards
```

This is another example of putting the complexity of shard routing below the application.

---

# 18. Sharding + Replication

These concepts can be combined.

For a large system:

```text
                         Application
                              │
                       Sharding layer
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Shard 1             Shard 2             Shard 3
          │                   │                   │
       Primary              Primary              Primary
        /   \                /   \                /   \
       ↓     ↓              ↓     ↓              ↓     ↓
    Replica Replica      Replica Replica      Replica Replica
```

Now:

- Sharding distributes the dataset.
- Replication creates copies of each shard.
- Routing determines which shard receives a request.
- Replicas can handle read traffic or provide failover.

This is a common large-scale architecture.

---

# 19. Oracle in all of this

Since Oracle is a commonly used database in enterprise systems, it is useful to map the concepts to Oracle.

### Replication

Oracle:

```text
Primary
   │
Data Guard
   │
Standby
```

Active Data Guard can provide a read-only standby while redo is being applied.

### Sharding

Oracle supports sharding / globally distributed database capabilities.

Conceptually:

```text
                 Logical Database
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Shard 1       Shard 2      Shard 3
       Oracle DB     Oracle DB    Oracle DB
```

Oracle provides its own sharding infrastructure and routing mechanisms, so you don't necessarily implement a custom consistent-hashing layer yourself.

---

# 20. Important interview distinction

Don't say:

> "Oracle uses consistent hashing."

unless you know the exact implementation being discussed.

A better statement is:

> "We can use hash-based sharding to distribute data across database instances. If we build our own routing layer, we could use consistent hashing to minimize data movement when shards are added or removed. Some database platforms provide their own sharding and routing mechanisms, so the application may not need to implement this itself."

---

# 21. A strong system-design answer

If asked:

**"How would you scale the database for a huge application?"**

A good progression is:

```text
Start with one DB
       ↓
Optimize queries
       ↓
Indexes
       ↓
Connection pooling
       ↓
Caching
       ↓
Read replicas
       ↓
Partitioning
       ↓
Sharding
       ↓
Replication of shards
```

And explain the roles:

```text
Read replicas
→ Scale reads

Sharding
→ Scale storage and write capacity

Consistent hashing
→ A possible routing strategy that reduces
  remapping/data movement when shard topology changes

Routing layer
→ Determines which database shard handles a request

Caching
→ Reduce database traffic altogether
```

## The key mental model

Remember these four sentences:

> **Replication = make copies.**

> **Sharding = split the data.**

> **Routing = decide where a request goes.**

> **Consistent hashing = one possible routing/distribution technique that minimizes remapping when nodes change.**

That's the core of the whole topic.
