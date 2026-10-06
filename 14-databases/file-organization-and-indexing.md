# File organization and indexing

> **Paper:** CS · **Priority:** P1 · **Plan topics:** records, indexes, B/B+ trees, hashing, primary/secondary and clustered indexes
> **Prerequisites:** [Relational model](relational-model-algebra-calculus.md) · [Data structures: trees](../07-data-structures/trees-and-bst.md)

## Quick glance
- Heap file: fast insertion, scan required without index.
- Ordered/sequential file: range-friendly, costly updates.
- Index maps search key to record/block location; dense has entry per key/record, sparse samples blocks (typically ordered file).
- B+ tree stores records/pointers at leaves; internal nodes guide search; leaves are linked for ranges.
- Hash index is strong for equality, poor for ordered range queries.

## 1. Index choices
Primary index is on ordering key of a sorted file; secondary index is on a non-ordering attribute. Clustered index determines physical row order; unclustered index does not. A table usually has only one physical clustering order.

## 2. B+ tree intuition
Balanced tree keeps all data entries at leaves; every leaf is at same depth. Internal separators route searches. Range query finds first key then scans linked leaves. If fanout is high, height stays small, reducing disk I/Os. In insertion, overflow splits and may propagate upward; deletion may redistribute or merge.

## 3. Hashing
Hash function maps key to bucket. Collisions are resolved by chaining or open addressing. Extendible hashing splits buckets and adjusts a directory as data grows. Hashing expected lookup can be O(1), but worst case degrades with collisions.

## GATE traps
- B-tree and B+ tree store data differently; follow the stated variant.
- Dense/sparse describes entries per search key/block, not whether the index is ordered.
- Hashing does not preserve key order, so range queries are inefficient.
- Count disk I/Os by pages/nodes, not by individual keys.

## Connections
- [Trees and BSTs](../07-data-structures/trees-and-bst.md) — balanced search structure fundamentals.
- [File systems](../13-operating-systems/file-systems-and-disk-scheduling.md) — persistent block allocation.
- [SQL](sql.md) — indexes accelerate predicates and joins.

## Practice
**Q1.** Which is usually better for range queries, B+ tree or hash index?
<details><summary>Answer</summary> B+ tree because keys are ordered and leaves linked.</details>

**Q2.** What is a dense index?
<details><summary>Answer</summary> It has an index entry for every search-key value/record according to the index definition.</details>

**Q3.** Why can hash index not efficiently answer “key between 20 and 40”?
<details><summary>Answer</summary> Hash values do not preserve key order.</details>
