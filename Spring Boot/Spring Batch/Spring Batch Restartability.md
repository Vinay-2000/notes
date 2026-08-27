

## Goal

Understand how Spring Batch behaves when a chunk fails and how the job can be restarted without processing everything from the beginning.

We tested this against an Oracle database using Spring Batch with persistent JDBC metadata.

---

# 1. Why Spring Batch Metadata Matters

Initially, the batch job was running successfully even though we did not see any `BATCH_*` tables.

That was because the basic Spring Batch engine can operate with an in-memory repository.

Conceptually:

```text
Spring Batch
    ↓
JobRepository
    ↓
In-memory state
```

This is enough for the job to run and track execution while the application is alive.

But for true restartability across application restarts, execution metadata needs to be persisted.

We added:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch-jdbc</artifactId>
</dependency>
```

Along with:

```properties
spring.batch.jdbc.initialize-schema=always
```

This created the Spring Batch metadata tables in the Oracle `PAPER_TRADING` schema.

The architecture becomes:

```text
Spring Batch
    ↓
JobRepository
    ↓
JDBC
    ↓
Oracle
    ↓
BATCH_* tables
```

---

# 2. `spring-boot-starter-jdbc` vs `spring-boot-starter-batch-jdbc`

These solve different problems.

## `spring-boot-starter-jdbc`

Used by our application to access the business database:

```text
starter-jdbc
    ↓
DataSource
    ↓
JdbcTemplate
    ↓
index_test
```

This allowed us to do:

```java
jdbcTemplate.update(...)
jdbcTemplate.batchUpdate(...)
```

## `spring-boot-starter-batch-jdbc`

Used by Spring Batch itself to persist Batch metadata:

```text
starter-batch-jdbc
    ↓
Spring Batch JDBC infrastructure
    ↓
JobRepository
    ↓
BATCH_* tables
```

These tables store information needed for execution tracking and restartability.

---

# 3. Important Batch Metadata Tables

The main tables we inspected are:

```text
BATCH_JOB_INSTANCE
BATCH_JOB_EXECUTION
BATCH_STEP_EXECUTION
BATCH_JOB_EXECUTION_CONTEXT
BATCH_STEP_EXECUTION_CONTEXT
```

They represent different levels of information.

---

## `BATCH_JOB_INSTANCE`

Represents a logical instance of a job.

A JobInstance is identified by:

```text
Job name + identifying JobParameters
```

For example:

```text
salaryJob + test=restart1
```

represents one JobInstance.

Conceptually:

```text
JOB_INSTANCE_ID
JOB_NAME
JOB_KEY
```

The `JOB_KEY` is generated from the identifying job parameters.

---

# 4. JobInstance vs JobExecution

This distinction is extremely important.

A single JobInstance can have multiple JobExecutions.

For example:

```text
JobInstance
salaryJob + test=restart1
        │
        ├── JobExecution #20 → FAILED
        │
        └── JobExecution #40 → COMPLETED
```

The second execution is the **restart** of the same JobInstance.

It is NOT a new JobInstance.

---

# 5. What Job Parameters Do

The parameter:

```text
test=restart1
```

does NOT tell Spring Batch:

> "Resume from ID 5001."

Instead, it identifies which JobInstance we are talking about.

So:

```text
salaryJob + test=restart1
```

means:

```text
Find the JobInstance for this job + parameters.
```

The actual restart position comes from the persisted Step execution state / ExecutionContext and the reader's restart information.

---

# 6. `BATCH_STEP_EXECUTION`

This records what happened during a particular Step execution.

We queried:

```sql
SELECT
    STEP_EXECUTION_ID,
    JOB_EXECUTION_ID,
    STEP_NAME,
    STATUS,
    READ_COUNT,
    WRITE_COUNT,
    COMMIT_COUNT,
    ROLLBACK_COUNT
FROM BATCH_STEP_EXECUTION
ORDER BY STEP_EXECUTION_ID;
```

During our experiment we saw:

```text
STEP_EXECUTION_ID | STATUS      | READ_COUNT | WRITE_COUNT | COMMIT | ROLLBACK
--------------------------------------------------------------------------------
20                | FAILED      | 6000       | 5000        | 5      | 1
40                | COMPLETED   | 4999       | 4999        | 5      | 0
```

This was strong evidence of what happened.

---

# 7. Our Failure Experiment

We used:

```java
if (item.getId() == 5005) {
    throw new RuntimeException("Intentional failure");
}
```

And:

```java
.chunk(1000)
```

The processing looked like:

```text
Chunk 1 → IDs 1–1000       → COMMIT
Chunk 2 → IDs 1001–2000    → COMMIT
Chunk 3 → IDs 2001–3000    → COMMIT
Chunk 4 → IDs 3001–4000    → COMMIT
Chunk 5 → IDs 4001–5000    → COMMIT

Chunk 6 → IDs 5001–6000
             ↓
           ID 5005
             ↓
          exception
             ↓
          ROLLBACK
```

---

# 8. What Rollback Actually Did

The failed chunk contained:

```text
5001–6000
```

Although IDs 5001–5004 were processed before the exception, they were part of the same transaction as ID 5005.

Therefore the entire chunk transaction rolled back.

We checked:

```sql
SELECT id, salary
FROM index_test
WHERE id BETWEEN 4995 AND 5010
ORDER BY id;
```

The result showed:

```text
4995–5000 → salary = 0
5001–5010 → salary = 50000
```

This proves:

```text
Previous chunk → committed
Failed chunk   → rolled back completely
```

This is the transaction boundary created by the Spring Batch chunk.

---

# 9. Why READ_COUNT Was 6000 But WRITE_COUNT Was 5000

The failed execution showed:

```text
READ_COUNT  = 6000
WRITE_COUNT = 5000
COMMIT_COUNT = 5
ROLLBACK_COUNT = 1
```

Why?

Spring Batch had read the sixth chunk:

```text
5001–6000
```

so those records contributed to the read count.

But the sixth chunk never successfully committed.

The previous five chunks had successfully written:

```text
5 × 1000 = 5000
```

Therefore:

```text
READ_COUNT  = 6000
WRITE_COUNT = 5000
```

---

# 10. Restarting the Same Job

After the failure, we removed the intentional exception.

Important:

We kept the same parameter:

```text
test=restart1
```

Then we started the application again.

Spring Batch found:

```text
salaryJob + test=restart1
```

and therefore found the existing JobInstance whose previous execution had failed.

It restarted the failed execution.

The reader started at:

```text
5001
```

rather than:

```text
1
```

This was the actual restartability test.

---

# 11. Why Did It Start at 5001?

This is the key concept.

Spring Batch uses its persistent execution state, including the Step's `ExecutionContext`, to remember restart/checkpoint information.

Conceptually:

```text
BATCH_STEP_EXECUTION
        ↓
Step execution state
        ↓
BATCH_STEP_EXECUTION_CONTEXT
        ↓
Reader restart/checkpoint state
        ↓
Reader resumes
```

The last successfully committed chunk was:

```text
4001–5000
```

The next chunk:

```text
5001–6000
```

failed.

Therefore the restart begins from the beginning of the failed chunk:

```text
5001
```

It does not resume at 5005 because the transaction containing 5001–5004 and 5005 was rolled back.

The restart point is effectively based on the **last successful checkpoint**, not the last item that happened to execute before the exception.

---

# 12. The Restart Execution

The second execution processed the remaining records.

Our metadata showed approximately:

```text
First execution:
READ  = 6000
WRITE = 5000
STATUS = FAILED
ROLLBACK = 1

Restart execution:
READ  = 4999
WRITE = 4999
STATUS = COMPLETED
ROLLBACK = 0
```

The restart therefore processed the remaining work rather than starting the entire 10,000-row job again.

---

# 13. Why Same Parameters Matter

Suppose we run:

```text
salaryJob + test=restart1
```

and it fails.

Then:

```text
salaryJob + test=restart1
```

again means:

```text
Same JobInstance
        ↓
Previous execution FAILED
        ↓
Restart allowed
```

But if we change the parameter:

```text
salaryJob + test=restart2
```

then:

```text
NEW JobInstance
        ↓
Starts as a new job
```

The parameter identifies the JobInstance; it does not itself contain the restart position.

---

# 14. What Happens After Successful Completion?

After the restart completed:

```text
salaryJob + test=restart1
        ↓
JobInstance
        ↓
COMPLETED
```

If we try to start:

```text
salaryJob + test=restart1
```

again, Spring Batch rejects it with a `JobInstanceAlreadyCompleteException`.

The reason is:

```text
Same JobInstance
        ↓
Previous execution = COMPLETED
        ↓
No restart required
        ↓
Reject
```

To run the job again as a new instance, provide different identifying parameters.

---

# 15. The Three Cases to Remember

```text
FAILED JobInstance
+ SAME parameters
        ↓
RESTART allowed
```

```text
COMPLETED JobInstance
+ SAME parameters
        ↓
JobInstanceAlreadyCompleteException
```

```text
DIFFERENT parameters
        ↓
NEW JobInstance
        ↓
Starts from beginning
```

---

# 16. The Complete Mental Model

```text
                    Job
                     │
                     ▼
              Job + Parameters
                     │
                     ▼
                JobInstance
                     │
              ┌──────┴──────┐
              ▼             ▼
        JobExecution 1  JobExecution 2
           FAILED         COMPLETED
              │
              ▼
        StepExecution
              │
              ▼
        ExecutionContext
              │
              ▼
      Reader checkpoint
              │
              ▼
       Restart from the
    last successful checkpoint
```

---

# 17. Rollback vs Restartability

These are separate concepts.

## Rollback

Handled by the database transaction:

```text
Chunk
  ↓
SQL operations
  ↓
Exception
  ↓
ROLLBACK
```

It answers:

> What happens to the current chunk?

---

## Restartability

Handled through Spring Batch execution metadata:

```text
JobRepository
    ↓
BATCH_* tables
    ↓
ExecutionContext
    ↓
Remember previous progress
    ↓
Restart
```

It answers:

> After the application/job fails, where should processing continue?

So:

```text
ROLLBACK
= undo the failed transaction

RESTARTABILITY
= remember successful progress and continue later
```

---

# 18. Interview Answer

If asked:

> How does Spring Batch support restartability?

A good answer:

> Spring Batch persists job and step execution metadata through the JobRepository. For a JDBC-backed repository, this metadata is stored in the BATCH_* tables. During chunk processing, successful progress is checkpointed and the reader's restart state can be stored in the Step ExecutionContext. If a chunk fails, its transaction is rolled back. When the same failed JobInstance is restarted with the same identifying JobParameters, Spring Batch uses the persisted execution state and reader restart information to resume from the last successful checkpoint rather than starting from the beginning.

---

# 19. Most Important Takeaways

```text
JobInstance
    =
Job name + identifying JobParameters
```

```text
JobExecution
    =
one execution/attempt of a JobInstance
```

```text
StepExecution
    =
execution details for a Step
```

```text
ExecutionContext
    =
persisted state used for restart/checkpoint information
```

```text
Chunk
    =
transaction boundary
```

And the complete flow:

```text
Read
 ↓
Process
 ↓
Write
 ↓
Commit
 ↓
Checkpoint
 ↓
Next chunk

Failure
 ↓
Rollback current chunk
 ↓
Persist FAILED execution
 ↓
Application stops

Restart same JobInstance
 ↓
Load persisted execution state
 ↓
Reader resumes from last successful checkpoint
 ↓
Continue processing
```
