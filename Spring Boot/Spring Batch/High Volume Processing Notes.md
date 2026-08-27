## Resume Point

> **Created High Volume Processing using Spring Batch**

The goal of today's experiment was to process a large Oracle table efficiently and understand how Spring Batch can improve throughput for high-volume processing.

Our test table:

```text
index_test
├── id
├── name
└── salary
```

We used **1,000,000 records** and updated every record:

```sql
UPDATE index_test
SET salary = 0
WHERE id = ?
```

---

# 1. Start Simple — Reader + Writer

We first created a normal Spring Batch chunk-oriented job:

```text
Reader → Writer
```

with:

```java
.chunk(1000, transactionManager)
```

So Spring Batch processes:

```text
1000 records
    ↓
write
    ↓
COMMIT
    ↓
next 1000 records
```

### Reader

We initially used:

```text
JdbcCursorItemReader
```

It reads rows sequentially from a database cursor.

### Writer — first implementation

We implemented a normal `ItemWriter` and used `JdbcTemplate.update()` inside a loop:

```java
for (IndexTest item : items) {
    jdbcTemplate.update(
        "UPDATE index_test SET salary = 0 WHERE id = ?",
        item.getId()
    );
}
```

This means every item resulted in an individual JDBC update.

For 100,000 records:

```text
100,000 records
    ↓
100,000 jdbcTemplate.update() calls
```

### Performance

For 100,000 records:

**~3 minutes 31 seconds**

This was very slow.

---

# 2. Database Index

We checked whether `index_test.id` had an index.

It didn't.

Since every update uses:

```sql
WHERE id = ?
```

we created:

```sql
CREATE INDEX idx_index_test_id
ON index_test (id);
```

This made the processing dramatically faster because Oracle could use the index to locate the target row instead of repeatedly searching the table.

---

# 3. JDBC Batch Update

The next improvement was to keep our own `ItemWriter`, but replace individual updates with:

```java
jdbcTemplate.batchUpdate(...)
```

Instead of:

```text
update()
update()
update()
...
```

we now send a group of updates as a JDBC batch.

Conceptually:

```text
Spring Batch chunk
        ↓
1000 items
        ↓
ItemWriter.write()
        ↓
JdbcTemplate.batchUpdate()
        ↓
JDBC batch
        ↓
Oracle
```

For 100,000 records:

**~5.6 seconds**

Huge improvement compared with ~3m 31s.

For 1,000,000 records:

**~33 seconds**

---

# 4. JdbcBatchItemWriter

We also tested Spring Batch's built-in:

```text
JdbcBatchItemWriter
```

It is an `ItemWriter` implementation designed for JDBC batch writing.

Conceptually:

```text
ItemWriter
    ↑
JdbcBatchItemWriter
    ↓
JDBC batch operations
    ↓
Oracle
```

We tested 100,000 records and got approximately:

**~4.8 seconds**

This was very close to our custom:

```text
ItemWriter + JdbcTemplate.batchUpdate()
```

which took:

**~5.6 seconds**

For million records **~33 seconds**

### Important lesson

`JdbcBatchItemWriter` is not magically faster simply because it is Spring Batch.

The important optimization was **JDBC batching**.

Both approaches ultimately use the database's/JDBC driver's ability to execute updates in batches.

---

# 5. Understanding Chunk vs JDBC Batch

These are two different concepts.

### Spring Batch Chunk

```java
.chunk(1000, transactionManager)
```

controls the **transaction boundary**.

Conceptually:

```text
Read 1000
   ↓
Write 1000
   ↓
COMMIT
```

If something goes wrong in the transaction, the chunk can be rolled back.

### JDBC Batch

```text
batchUpdate()
```

controls how multiple SQL operations are sent/executed efficiently through JDBC.

So:

```text
Spring Batch
    ↓
Chunk = 1000
    ↓
ItemWriter
    ↓
JDBC Batch
    ↓
Oracle
```

Chunking and JDBC batching are related but **not the same thing**.

---

# 6. Cursor Reader vs Paging Reader

We then changed the reader from:

```text
JdbcCursorItemReader
```

to:

```text
JdbcPagingItemReader
```

The idea of paging is:

```text
Page 1 → rows 1–1000
Page 2 → rows 1001–2000
Page 3 → rows 2001–3000
...
```

With:

```java
.pageSize(1000)
```

the reader fetches data in pages rather than maintaining one continuous cursor.

### Performance

With:

```text
JdbcPagingItemReader
+
ItemWriter
+
JdbcTemplate.batchUpdate()
+
1 thread
```

1,000,000 records took:

**~38.1 seconds**

This was slower than the previous cursor-based version (~33 seconds).

### Lesson

A cursor reader can be more efficient for simple sequential processing because it can continuously consume a database cursor.

Paging introduces additional page-fetch/query overhead.

However, paging is useful when designing for parallel processing because the work can be divided into independent ranges.

---

# 7. Simple Multi-threaded Step

We then experimented with a multi-threaded Step using:

```text
ThreadPoolTaskExecutor
```

We tested different thread counts.

Results for 1,000,000 records:

```text
1 thread  → ~38.1s
2 threads → ~39.2s
4 threads → ~37.5s
8 threads → ~38.2s
```

### Lesson

Simply adding threads did **not** significantly improve performance.

More threads do not automatically mean more throughput.

The threads still compete for the same downstream resources:

```text
Thread 1 ─┐
Thread 2 ─┤
Thread 3 ─┼──→ Oracle
Thread 4 ─┤
Thread 5 ─┘
```

Eventually the database becomes the bottleneck.


```
                Paging Reader
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Thread 1   Thread 2   Thread 3
          │          │          │
       chunk       chunk      chunk
```

**One caveat:** the exact SQL generated by `JdbcPagingItemReader` depends on the database type and Spring Batch version. Oracle's paging strategy can itself add some overhead.

Also, sharing a reader between multiple threads is not the cleanest architecture for this use case.

---

# 8. Partitioning

This was the approach that gave us a meaningful improvement.

Instead of multiple threads trying to work from the same reader, we divided the data into independent partitions.

For example, with 4 partitions:

```text
1,000,000 records
        ↓
┌───────────────┬───────────────┬───────────────┬───────────────┐
│ Partition 1   │ Partition 2   │ Partition 3   │ Partition 4   │
│ 1 - 250k      │ 250k - 500k   │ 500k - 750k   │ 750k - 1M    │
└───────────────┴───────────────┴───────────────┴───────────────┘
       ↓               ↓               ↓               ↓
    Thread 1        Thread 2        Thread 3        Thread 4
       ↓               ↓               ↓               ↓
    Reader 1        Reader 2        Reader 3        Reader 4
       ↓               ↓               ↓               ↓
    Writer 1        Writer 2        Writer 3        Writer 4
```

Each partition receives its own:

```text
minId
maxId
```

through the Step Execution Context.

The reader is `@StepScope`, so each worker gets its own reader instance configured with its own ID range.

---

# 9. Partitioning Performance

### 4 partitions

```text
~24.7 seconds
```

Compared with:

```text
1 thread + paging → ~38.1 seconds
```

This was a significant improvement.

### 8 partitions

Two runs:

```text
~22.2 seconds
~25.2 seconds
```

So roughly **22–25 seconds**.

### 16 partitions

Two runs:

```text
~25.0 seconds
~25.7 seconds
```

Performance got worse again.

### Conclusion

For our local Oracle-in-Docker setup, **8 partitions was around the sweet spot**.

Increasing concurrency beyond that did not improve throughput.

This demonstrates:

> More workers do not necessarily mean more performance. Eventually the database or system resources become the bottleneck.

---

# 10. Final Architecture

The final high-volume processing design was:

```text
                         Spring Batch Job
                                │
                                ▼
                       Partitioning Step
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
         Partition 1       Partition 2       Partition 3 ...
              │                 │                 │
           Thread 1          Thread 2          Thread 3
              │                 │                 │
        Paging Reader     Paging Reader     Paging Reader
              │                 │                 │
           Chunk 1000        Chunk 1000        Chunk 1000
              │                 │                 │
        ItemWriter          ItemWriter          ItemWriter
              │                 │                 │
       JDBC batchUpdate   JDBC batchUpdate   JDBC batchUpdate
              │                 │                 │
              └─────────────────┼─────────────────┘
                                ↓
                              Oracle
```

The key optimizations were:

1. **Index the column used to locate rows**
2. **Use JDBC batching instead of individual JDBC updates**
3. **Use Spring Batch chunks to control transaction boundaries**
4. **Use paging when designing the reader for parallel partitioned processing**
5. **Use partitioning to give each worker an independent range of data**
6. **Tune concurrency based on actual database/system throughput rather than blindly increasing threads**

---

# 11. Interview Explanation

If asked:

> "How did you implement high-volume processing using Spring Batch?"

A concise answer:

> I implemented a Spring Batch job to process around 1 million Oracle records. Initially, I used a chunk-oriented step with a JDBC reader and a custom `ItemWriter` that performed individual `JdbcTemplate.update()` calls. That was very slow because every record resulted in a separate JDBC operation.
>
> I then added an index on the ID used by the update and changed the writer to use JDBC batch updates, which reduced the processing time dramatically.
>
> For parallel processing, I moved to `JdbcPagingItemReader` and then used Spring Batch partitioning to divide the data into independent ID ranges. Each partition had its own reader and writer and ran concurrently using a thread pool.
>
> I benchmarked different partition counts and found that increasing concurrency eventually stopped improving throughput because the database became the bottleneck. In my local test, around 8 partitions gave the best results.

---

# Performance Summary

```text
1M records

Cursor + batchUpdate
≈ 33s

Paging + batchUpdate, 1 thread
≈ 38s

Simple multi-threaded step
≈ 37–39s

Partitioning, 4 partitions
≈ 24.7s

Partitioning, 8 partitions
≈ 22–25s

Partitioning, 16 partitions
≈ 25–26s
```

## What I should remember

```text
Chunk ≠ JDBC batch ≠ Partition
```

**Chunk** → transaction boundary

**JDBC batch** → efficient execution of many SQL operations

**Partition** → divide a large workload into independent pieces that can be processed concurrently
