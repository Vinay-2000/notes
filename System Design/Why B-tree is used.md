# B-Tree Indexes — Why They Are Efficient

## 1. B-tree vs BST

A common question is:

> If a balanced Binary Search Tree (BST) is already O(log N), why do databases use B-trees?

The important answer is:

> **Both can have O(log N) search complexity, but B-trees are designed to minimize expensive disk/storage I/O.**

A normal BST is primarily useful as an in-memory data structure.

A B-tree is designed around **database/storage pages or blocks**.

---

# 2. Binary Search Tree

A BST has at most two children per node:

```text
             50
           /             25      75
        /  \    /        10   30  60   90
```

To find `90`:

```text
50 → 75 → 90
```

A balanced BST containing 1 million values has approximately:

```text
log₂(1,000,000) ≈ 20
```

levels.

In memory, that is perfectly reasonable.

But if each node required a separate disk/storage read:

```text
Disk read
   ↓
Node 1
   ↓
Disk read
   ↓
Node 2
   ↓
Disk read
   ↓
...
   ↓
~20 reads
```

That can be expensive.

---

# 3. B-tree

A B-tree does not restrict each node to two children.

A node can contain **many keys and many child pointers**.

Simplified:

```text
                 [20 | 40 | 60 | 80]
               /     |     |     |                   ↓      ↓     ↓     ↓      ↓
            nodes  nodes nodes nodes  nodes
```

One node might contain hundreds of keys/child pointers depending on:

- Database block/page size
- Key size
- Pointer size
- Database implementation

This gives the tree a very high **branching factor**.

---

# 4. High branching factor = shallow tree

Suppose a simplified B-tree node can point to 100 children.

For 1,000,000 keys:

```text
Level 1:
100 children

Level 2:
100 × 100 = 10,000

Level 3:
100 × 100 × 100 = 1,000,000
```

So a very large number of keys can potentially be reached in only a few levels.

Compare:

```text
Balanced BST:
~20 levels for 1 million values

B-tree:
Potentially ~3 levels with a branching factor of 100
```

The exact numbers vary in real databases, but the principle is what matters.

---

# 5. Database storage works in blocks/pages

Databases don't normally read one individual key from disk.

They read a **block/page** containing many pieces of data.

A B-tree node is designed to fit efficiently into these storage blocks.

Conceptually:

```text
Disk / Storage
      ↓
┌───────────────────────┐
│      Index Block      │
│                       │
│  10  20  30  40  50  │
│  60  70  80  90 ...  │
│                       │
└───────────────────────┘
```

One block read gives the database many keys at once.

This is much more efficient than requiring a separate read for every individual key/node.

---

# 6. BST vs B-tree from an I/O perspective

## BST

Conceptually:

```text
Disk
 ↓
[50]
 ↓
Disk
 ↓
[75]
 ↓
Disk
 ↓
[90]
```

Potentially many storage accesses.

## B-tree

```text
Disk
 ↓
[20 | 40 | 60 | 80 | 100 | ...]
 ↓
Correct child
 ↓
[Many keys]
 ↓
Correct leaf
```

Very few storage accesses.

---

# 7. B-tree search

Suppose we want:

```text
9876543
```

The B-tree might look conceptually like:

```text
                    Root
                     │
                     ↓
          [2M | 5M | 8M]
                     │
                  8M range
                     ↓
          [8.5M | 9M | 9.5M]
                     │
                     ↓
              Leaf node
                     │
                9876543
                     │
                   ROWID
                     │
                     ↓
                Table block
```

Oracle doesn't scan every indexed value.

It navigates the tree by comparing the search key with the keys stored in each node.

---

# 8. Why this is useful for Oracle indexes

When you run:

```sql
SELECT *
FROM employee
WHERE id = 9876543;
```

Oracle can potentially do:

```text
B-tree index
      ↓
Root block
      ↓
Branch block
      ↓
Leaf block
      ↓
ROWID
      ↓
Actual table block
```

Even if the table has hundreds of millions of rows, the number of index levels can remain very small.

---

# 9. B-tree search complexity

A balanced BST:

```text
O(log₂ N)
```

A B-tree:

```text
O(log_B N)
```

where `B` is the branching factor.

The important difference is the base.

For example:

```text
BST:
log₂(1,000,000) ≈ 20
```

With a simplified branching factor of 100:

```text
B-tree:
log₁₀₀(1,000,000) = 3
```

Again, real database indexes have different structures and sizes, but this illustrates why B-trees are so shallow.

---

# 10. B-tree vs BST

| Feature | Balanced BST | B-tree |
|---|---|---|
| Children per node | ~2 | Many |
| Keys per node | Usually 1 | Many |
| Typical use | In-memory structures | Databases/filesystems |
| Height | Relatively larger | Very small |
| Designed around storage blocks | No | Yes |
| Search complexity | O(log N) | O(log N) |
| Storage I/O | Potentially many | Minimized |
| High branching factor | No | Yes |

The important point:

> **B-trees aren't fundamentally faster because they have O(log N) search. A balanced BST also has O(log N) search. B-trees are designed to minimize expensive storage I/O by storing many keys per node and keeping the tree shallow.**

---

# 11. Why databases don't simply use a BST

Imagine a database with:

```text
500 million rows
```

A BST might have many levels.

A B-tree can have a very high branching factor:

```text
                     Root
                       │
           ┌───────────┼───────────┐
           ↓           ↓           ↓
        Branch       Branch      Branch
       /  |  \      / | \      / | \
      ↓   ↓   ↓    ↓  ↓  ↓    ↓  ↓  ↓
    Leaves...
```

Because each node contains many keys, the tree stays shallow.

This means fewer block accesses.

---

# 12. B-tree vs B+ tree

Database indexes are often described as B-trees, but many database implementations use structures closer to **B+ trees**.

A useful simplified distinction:

### B-tree

Keys/data can potentially appear in internal and leaf nodes.

### B+ tree

Internal nodes primarily guide navigation, while the actual row references are stored at the leaf level.

Conceptually:

```text
                Root
                 │
          Internal nodes
                 │
                 ↓
             Leaf nodes
                 │
          Key → ROWID
                 │
                 ↓
             Table row
```

Oracle commonly refers to its standard indexes as **B-tree indexes**, and the important practical mental model is the same: navigate a shallow tree to a leaf entry containing the row locator.

---

# 13. The key mental model

Remember:

```text
BST
→ optimized mainly for in-memory searching

B-tree
→ optimized for storage systems

B-tree
→ many keys per node
→ high branching factor
→ very shallow tree
→ fewer storage block accesses
→ excellent for database indexes
```

The most important sentence:

> **A B-tree is efficient for databases because it packs many keys into each storage block, giving it a high branching factor and keeping the tree shallow, which minimizes expensive disk/storage I/O.**
