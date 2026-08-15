# Database Indexes — Oracle

## 1. What is an Index?

An index is a **separate data structure** that provides an efficient way to find rows in a table.

A useful mental model is:

```text
Indexed key → location of the corresponding table row
```

For a normal Oracle heap-organized table, that location is commonly represented by a **ROWID**.

```sql
CREATE INDEX idx_employee_id
ON employee(id);
```

Conceptually:

```text
Index
  ↓
id = 9876543
  ↓
ROWID
  ↓
Actual row in the table
```

Oracle commonly implements indexes using a **B-tree structure**, rather than a simple key-value map.

---

## 2. Why indexes are useful

Suppose a table has 1 million rows:

```sql
SELECT *
FROM employee
WHERE id = 999999;
```

### Without an index

Oracle may perform:

```text
TABLE ACCESS FULL
        ↓
Read table blocks
        ↓
Check rows
        ↓
Find ID = 999999
```

### With an index

```text
Query
  ↓
Index
  ↓
Find ID = 999999
  ↓
ROWID
  ↓
Fetch actual row
```

The index lets Oracle avoid scanning the entire table when the condition is selective.

---

## 3. B-tree mental model

A B-tree can be visualized as:

```text
                  Root
                   │
            ┌──────┴──────┐
            ↓             ↓
        Branch          Branch
          │               │
          ↓               ↓
       Leaf nodes      Leaf nodes
          │
          ↓
      Key + ROWID
          │
          ↓
     Actual table row
```

Instead of searching every row, Oracle navigates through the tree to find the key.

A B-tree lookup is roughly **O(log N)**, compared with scanning O(N) rows.

The important intuition:

> As the table gets larger, an index can avoid looking at most of the rows.

---

## 4. Selectivity

Indexes are particularly useful for **selective** queries.

If a table has 10 million rows:

```sql
WHERE employee_id = 9876543
```

and that ID is unique:

```text
10 million rows
      ↓
1 matching row
```

The index is very useful.

But:

```sql
WHERE gender = 'Male'
```

if 50% of rows match:

```text
10 million rows
      ↓
5 million matching rows
```

An index may not be better than a full table scan.

> **Selectivity = how small the matching portion of the table is.**

Highly selective predicates generally benefit more from indexes.

---

## 5. Indexes and JOINs

Indexes can make joins much faster, especially with large tables and selective conditions.

Example:

```text
CUSTOMER → 10 million rows
ORDERS   → 100 million rows
```

```sql
SELECT c.name, o.amount
FROM customer c
JOIN orders o
    ON c.id = o.customer_id
WHERE c.id = 12345;
```

With appropriate indexes:

```text
customer.id
     ↓
Find customer
     ↓
customer_id = 12345
     ↓
orders.customer_id index
     ↓
Find matching orders
```

However:

> **An index does not automatically make every JOIN fast.**

Oracle's optimizer can choose among nested loop joins, hash joins, sort merge joins, etc.

---

## 6. The optimizer

Oracle uses a **cost-based optimizer**.

It does not simply follow:

> "If an index exists, use it."

Instead, it estimates the cost of different execution plans using information such as:

- Number of rows
- Number of blocks
- Number of matching rows
- Index selectivity
- Column statistics
- Index statistics
- Clustering
- Join conditions
- Available indexes

Conceptually:

```text
SQL Query
   ↓
Optimizer
   ↓
Compare possible plans
   ├── Full table scan
   ├── Index scan
   ├── Nested loop
   ├── Hash join
   └── Other strategies
   ↓
Choose estimated cheapest plan
```

---

## 7. Optimizer statistics

Statistics tell Oracle what the data looks like.

They can include:

```text
Number of rows
Number of blocks
Number of distinct values
Number of nulls
Minimum value
Maximum value
Data distribution
Index statistics
```

Gather table statistics with:

```sql
BEGIN
    DBMS_STATS.GATHER_TABLE_STATS(
        USER,
        'INDEX_TEST'
    );
END;
/
```

Then inspect:

```sql
SELECT
    NUM_ROWS,
    BLOCKS,
    LAST_ANALYZED
FROM USER_TABLES
WHERE TABLE_NAME = 'INDEX_TEST';
```

---

## 8. Our experiment

We created:

```text
INDEX_TEST
1,000,000 rows
```

with:

```text
id = 1
id = 2
id = 3
...
id = 1,000,000
```

Then queried:

```sql
SELECT *
FROM index_test
WHERE id = 999999;
```

### Before index/table statistics

Oracle estimated:

```text
Rows = 68
```

and used:

```text
TABLE ACCESS FULL
```

It also reported:

```text
dynamic statistics used:
dynamic sampling (level=2)
```

### After creating the index

```sql
CREATE INDEX idx_index_test_id
ON index_test(id);
```

Oracle could use the index and its available statistics. The estimate became:

```text
Rows = 1
```

because every ID was unique.

### After dropping the index

```sql
DROP INDEX idx_index_test_id;
```

the plan returned to:

```text
TABLE ACCESS FULL
```

and the estimate returned to 68 because the index statistics were gone and the table had not yet had persistent statistics.

### After gathering table statistics

```sql
BEGIN
    DBMS_STATS.GATHER_TABLE_STATS(
        USER,
        'INDEX_TEST'
    );
END;
/
```

Oracle still used:

```text
TABLE ACCESS FULL
```

because there was no index.

But the estimated rows became:

```text
Rows = 1
```

This proved an important distinction:

> **Statistics and indexes are separate concepts.**

---

## 9. Index statistics

Inspect index statistics with:

```sql
SELECT
    INDEX_NAME,
    NUM_ROWS,
    DISTINCT_KEYS,
    CLUSTERING_FACTOR,
    LAST_ANALYZED
FROM USER_INDEXES
WHERE INDEX_NAME = 'IDX_INDEX_TEST_ID';
```

For our test:

```text
NUM_ROWS      ≈ 1,000,000
DISTINCT_KEYS ≈ 1,000,000
```

`DISTINCT_KEYS` is particularly useful.

Because every ID was unique:

```text
1,000,000 rows
1,000,000 distinct IDs
```

Oracle can estimate:

```text
WHERE id = 999999
        ↓
approximately 1 row
```

---

## 10. Indexes aren't free

Indexes improve reads but have costs.

An insert:

```sql
INSERT INTO employee ...
```

may require:

```text
Employee table
     +
Employee indexes
```

Updates/deletes involving indexed columns can also require index maintenance.

Therefore:

```text
More indexes
     ↓
Potentially faster reads
     +
More storage
     +
More write work
     +
More maintenance
```

Don't blindly index every column.

Indexes should support actual query patterns.

---

## 11. Full table scan vs index access

### Full table scan

```text
Query
 ↓
Table
 ↓
Read many/all relevant blocks
 ↓
Check rows
```

### Index access

```text
Query
 ↓
Index
 ↓
Find key
 ↓
ROWID
 ↓
Fetch table row
```

But if the query needs a large percentage of the table, a full table scan may be cheaper.

That is why the optimizer makes the decision.

---

## 12. Execution plan vs elapsed time

We measured query times before and after creating an index.

The results varied because of:

- Buffer cache
- Disk/cache state
- SQL Developer overhead
- CPU scheduling
- Database warm-up
- Other system activity

Our measurements were roughly:

```text
Without index:
~130–500 ms

With index:
~115–134 ms
```

The difference wasn't huge because the test table had only 1 million rows and data could be cached.

For database performance analysis, the **execution plan and database statistics** are generally more informative than one wall-clock measurement.

Useful operations to recognize:

```text
TABLE ACCESS FULL
INDEX RANGE SCAN
INDEX UNIQUE SCAN
TABLE ACCESS BY INDEX ROWID
```

---

## 13. Indexes are additional access paths

The best mental model is:

```text
                  Table
                    ↑
                    │
              Actual data
                    │
         ┌──────────┴──────────┐
         │                     │
   Full table scan          Index
                               │
                         Find matching key
                               │
                             ROWID
                               │
                               ↓
                         Actual table row
```

The table still contains the actual data.

The index is a **separate structure designed to locate relevant rows efficiently**.

---

## 14. Key concepts

### Index
A separate data structure that provides an efficient access path to table rows.

### B-tree
A tree-based structure commonly used for Oracle indexes that allows efficient key searches.

### ROWID
A locator Oracle can use to find a row in a heap-organized table.

### Selectivity
How small the matching portion of the table is. Highly selective predicates often benefit more from indexes.

### Statistics
Information about the data that helps Oracle's optimizer estimate costs and choose execution plans.

### Optimizer
The component that evaluates possible execution plans and chooses one based on estimated cost.

### Full table scan
Reading the table's blocks to find matching rows.

### Index range scan
Traversing an index to find a range of matching key values.

### Index unique scan
An efficient lookup through a unique index for a single key.

---

## 15. Complete mental model

```text
                    SQL Query
                        ↓
                    Optimizer
                        ↓
              Uses statistics to estimate
                  different strategies
                        ↓
             ┌──────────┴──────────┐
             ↓                     ↓
      Full Table Scan          Index Access
             │                     │
       Read table blocks       Search B-tree
                                   ↓
                                  ROWID
                                   ↓
                              Fetch rows
```

The key relationship:

```text
INDEX
→ provides an efficient access path

STATISTICS
→ describe the data to the optimizer

OPTIMIZER
→ chooses the execution plan

EXECUTION PLAN
→ shows how Oracle intends to execute the query
```

> **Indexes make fast access possible; statistics help Oracle know when that fast access is actually beneficial.**
