# Database Durability — Oracle Redo & PostgreSQL WAL

## 1. What is Durability?

Durability is the **D in ACID**.

It means:

> Once a database confirms that a transaction has successfully committed, that transaction should survive a crash, restart, or power failure.

Example:

```sql
UPDATE account
SET balance = 900
WHERE id = 1;

COMMIT;
```

If Oracle/PostgreSQL says the commit succeeded and the server immediately crashes, the committed change should not simply disappear.

---

# 2. The problem databases need to solve

A database has data pages/blocks that eventually live in data files.

Writing every changed data page directly to disk before every commit would be expensive.

Conceptually, the slow approach would be:

```text
Transaction
    ↓
Modify data
    ↓
Write data page to disk
    ↓
Wait for disk
    ↓
COMMIT
```

Databases instead use **write-ahead logging**.

---

# 3. Write-Ahead Logging (WAL)

WAL means:

> **The log describing a change must be persisted before the corresponding data change is considered safely committed.**

Conceptually:

```text
Database change
      ↓
Create log record
      ↓
Persist log
      ↓
COMMIT succeeds
      ↓
Data page can be written later
```

The important idea is:

```text
LOG FIRST
DATA LATER
```

This is why it is called **Write-Ahead Logging**.

The log gets written ahead of the actual database data page.

---

# 4. Why does WAL provide durability?

Suppose:

```text
Initial balance = 1000

UPDATE balance → 900
COMMIT
```

The changed database page may still be in memory.

But the log describing the change has already been persisted.

Then:

```text
             💥 DATABASE CRASH
                    ↓
               Read WAL
                    ↓
             Replay changes
                    ↓
              Recover DB
                    ↓
             balance = 900
```

The database doesn't need to have written every modified data page before the crash because it can reconstruct the necessary changes from the log.

---

# 5. PostgreSQL WAL

PostgreSQL explicitly calls its change log **WAL (Write-Ahead Log)**.

Simplified architecture:

```text
                PostgreSQL
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
     Buffer / Cache            WAL
          │                     │
     Changed pages         Change records
          │                     │
          ↓                     ↓
      Data files          WAL on disk
```

A transaction can modify a page in memory while the WAL record for that change is persisted.

The actual data page can be written to the data files later.

If PostgreSQL crashes:

```text
WAL
 ↓
Recovery
 ↓
Replay required changes
 ↓
Consistent database
```

---

# 6. Oracle equivalent: Redo Logging

Oracle doesn't normally call its mechanism "WAL".

Oracle uses **redo logging**.

The conceptual mapping is:

```text
PostgreSQL              Oracle

WAL                     Redo
WAL records             Redo records
WAL replay              Redo/recovery
WAL archiving           Archived redo logs
Streaming replication   Data Guard
```

They are not identical implementations, but they solve the same fundamental problems.

A useful mental model:

> **PostgreSQL WAL ≈ Oracle redo logging**

---

# 7. Oracle durability flow

Suppose:

```sql
UPDATE account
SET balance = 900
WHERE id = 1;

COMMIT;
```

Simplified Oracle flow:

```text
UPDATE
   ↓
Change represented in memory
   ↓
Redo generated
   ↓
Redo written/persisted
   ↓
COMMIT acknowledged
   ↓
Data block can be written later
```

The changed table block does not necessarily need to be written to the datafile at the exact moment of COMMIT.

The redo provides the information needed for recovery.

---

# 8. Oracle memory and storage

A simplified Oracle view:

```text
                  Oracle
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
    Buffer Cache          Redo Buffer
          │                   │
          ↓                   ↓
      Datafiles           Redo Logs
```

### Buffer Cache

Contains database blocks/pages currently being worked on.

### Datafiles

Eventually contain the actual persistent table/index data.

### Redo Logs

Contain information Oracle can use to recover changes.

The important durability relationship is:

```text
Change
  ↓
Redo
  ↓
Durable COMMIT
  ↓
Datafile write can happen later
```

---

# 9. What happens during a crash?

Suppose:

```text
10:00:00
UPDATE balance → 900

10:00:01
COMMIT succeeds

10:00:01.1
💥 Server crashes
```

The data block may not yet have reached the datafile.

But the redo is durable.

When Oracle starts:

```text
Oracle startup
     ↓
Instance recovery
     ↓
Read redo
     ↓
Redo/recovery operations
     ↓
Database becomes consistent
```

The committed change can therefore be recovered.

---

# 10. WAL/Redo is not the same as a backup

This distinction is important.

### WAL / Redo

Primarily helps with:

- Crash recovery
- Transaction durability
- Replication
- Point-in-time recovery when combined with archived logs/backups

### Backup

Protects against things such as:

- Database/storage destruction
- Accidental deletion
- Corruption
- Disaster recovery

Example:

```text
Database server crashes
        ↓
WAL / Redo
        ↓
Crash recovery
```

But:

```text
Entire database storage is destroyed
        ↓
Restore backup
        ↓
Apply archived WAL / redo
        ↓
Recover to desired point in time
```

So you need both logging and backups for serious disaster recovery.

---

# 11. WAL/Redo and Replication

The log can also be used to replicate changes.

## PostgreSQL

```text
             Primary
                │
               WAL
                │
                ↓
             Replica
                │
            WAL replay
                ↓
          Updated database
```

## Oracle

```text
             Primary
                │
               Redo
                │
                ↓
           Data Guard
                │
           Apply redo
                ↓
             Standby
```

So replication can be thought of as:

```text
Primary
   │
   │ stream changes
   ↓
Replica
   │
   │ replay changes
   ↓
Replica becomes synchronized
```

---

# 12. Synchronous vs Asynchronous replication

The log is also involved in replication timing.

### Asynchronous

```text
Primary
   │
   │ WAL / Redo
   └──────────────→ Replica

Primary can commit without waiting
```

There can be replication lag.

### Synchronous

```text
Primary
   │
   │ WAL / Redo
   ↓
Replica
   │
   │ acknowledgement
   ↓
Primary commits
```

The primary waits for the required acknowledgement, depending on the database/configuration.

This can provide stronger protection but can add latency.

---

# 13. Oracle: Redo → Data Guard

Oracle Data Guard uses redo to maintain standby databases.

```text
                 Primary
               Oracle DB
                   │
                  Redo
                   │
                   ↓
             Data Guard
                   │
            Apply Redo
                   ↓
                Standby
```

With **Active Data Guard**, a physical standby can be open read-only while redo is being applied.

This gives an architecture similar to:

```text
Primary
   │
   ├── writes
   │
   └── redo
         ↓
      Standby
         ↓
       reads
```

---

# 14. Why databases don't immediately write every data page

Suppose there are 10,000 transactions.

Naive approach:

```text
Transaction
    ↓
Write data page
    ↓
Wait for disk
    ↓
COMMIT
```

repeated thousands of times.

That can generate a lot of random I/O.

With logging:

```text
Transaction
    ↓
Generate log
    ↓
Persist log
    ↓
COMMIT
    ↓
Data page written later
```

The database can batch and optimize data-file writes while still maintaining durability.

---

# 15. Important terms to know

### WAL

**Write-Ahead Log**

PostgreSQL's term for its durable change log.

### Redo

Oracle's equivalent concept.

### Datafile

Oracle files containing the database's persistent data.

### Buffer Cache

Memory containing database blocks being accessed/modified.

### Checkpoint

A process that helps ensure dirty database blocks are progressively written to datafiles and reduces the amount of recovery work needed after a crash.

### Archived Redo / Archived WAL

Older log information retained for backup/recovery and point-in-time recovery.

---

# 16. Interview answer: What is WAL?

A strong answer:

> **WAL stands for Write-Ahead Logging. Before a database considers a transaction safely committed, the log describing the change is persisted. The actual data pages can be written later. If the database crashes, it can replay the log during recovery to reconstruct committed changes. WAL can also serve as the basis for replication.**

---

# 17. Interview answer: How does Oracle provide durability?

> **Oracle uses redo logging. When a transaction commits, the necessary redo information is persisted to the online redo logs before the commit is acknowledged. The modified data blocks can be written to the datafiles later. If the database crashes, Oracle uses redo during instance recovery to recover committed changes. Backups and archived redo provide additional protection for media failure and point-in-time recovery.**

---

# 18. The key mental model

Remember these:

```text
Durability
    ↓
Write-Ahead Logging / Redo
    ↓
Persist change log before commit
    ↓
Data pages can be written later
    ↓
Crash
    ↓
Replay log
    ↓
Recover committed changes
```

And:

```text
PostgreSQL → WAL
Oracle     → Redo
```

The most important sentence:

> **The database doesn't need to write the actual data page to disk before every commit; it needs the information required to recover that change to be safely persisted first.**
