# File organization and indexing

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** file organization (heap, sequential, hash), indexing structures, primary, secondary, and clustered indexes, B and B+ trees (order, search, insert, delete, node fanout, height, block access calculation)
> **Prerequisites:** [Relational model](relational-model-algebra-calculus.md) · [Data structures: trees and BST](../07-data-structures/trees-and-bst.md) · **Leads to:** [Data warehousing and preprocessing](data-warehousing-and-preprocessing.md)
> **PYQ weightage:** High · 22 Qs since 2015 (10 since 2021) · DA: 2 · Trend: steady · Most asked: B/B+ Tree Order & Node Capacity (13), Number of Block Accesses & Disk I/O (6), Primary & Clustered Indexes (3), Secondary Indexes & Hashing (2)

## Quick glance

- **Exam focus:** Numerical calculations of B and B+ tree node order ($p$), maximum and minimum keys/pointers per node, tree height, and total block accesses for equality and range queries; distinguishing dense vs sparse and primary vs clustered vs secondary indexes; asked heavily in GATE CS and DA as NAT and MCQ/MSQ.
- **Index classification:**
  - **Primary Index:** Defined on the **ordering key field** of a sorted data file. Always **sparse** (one entry per data block).
  - **Clustered Index:** Defined on the **ordering non-key field** of a sorted data file. Typically **sparse** (one entry per distinct search-key value).
  - **Secondary Index:** Defined on a **non-ordering field** (candidate key or non-key) of an ordered or unordered data file. **Always dense** (must have pointers to all records/distinct keys).
- **Dense vs Sparse:** An index is **dense** if an entry appears for *every search-key value* in the data file. It is **sparse** if entries appear for only *some* search-key values (typically one per disk block). A sparse index **requires** the data file to be physically sorted on the search key.
- **B-Tree node equation:** Keys and record pointers are stored in **all nodes** (internal and leaf):
  $$p \cdot P + (p - 1) \cdot (K + P_r) \le B_{\text{block}}$$
  where $p$ is order (max child pointers), $P$ is block pointer size, $K$ is key size, $P_r$ is record pointer size.
- **B+ Tree internal node equation:** Stores only keys and tree pointers (no record pointers):
  $$p \cdot P + (p - 1) \cdot K \le B_{\text{block}}$$
- **B+ Tree leaf node equation:** Stores key-record pointer pairs and one next-leaf block pointer:
  $$p_{\text{leaf}} \cdot (K + P_r) + P \le B_{\text{block}}$$
- **Minimum occupancy:** Every non-root node in a B/B+ tree must be at least **half full**:
  - B-tree non-root node: at least $\lceil p / 2 \rceil$ tree pointers, $\lceil p / 2 \rceil - 1$ keys.
  - B+ tree non-root internal node: at least $\lceil p / 2 \rceil$ tree pointers, $\lceil p / 2 \rceil - 1$ keys.
  - B+ tree leaf node: at least $\lceil p_{\text{leaf}} / 2 \rceil$ key-pointer entries.
- **Disk I/O for search:** Traversal of a balanced B+ tree of height $h$ requires $h$ index block reads $+ 1$ data block read (for unclustered data file). Range scan: index traversal to first leaf $+ \text{sequential scan along leaf chain}$.

## 1. File organization and basic index types [PYQ: Medium]

Database records are stored in fixed-size **disk blocks** (pages). The cost of database operations is dominated by the number of **block I/Os (disk accesses)**.

### Basic file organizations

| Organization | Record insertion | Equality search (Block I/Os) | Range search (Block I/Os) | Notes |
|---|---|---|---|---|
| **Heap file (unordered)** | $O(1)$ (append to last block) | $\approx B / 2$ average, $B$ worst-case | $B$ (full table scan) | Good for fast bulk ingest; poor for querying |
| **Sorted file (sequential)** | $O(B)$ (costly shifting or overflow blocks) | $\lceil \log_2 B \rceil$ (binary search) | $\lceil \log_2 B \rceil + \text{blocks matched}$ | Excellent for range queries; expensive updates |
| **Hash file** | $O(1)$ expected | $O(1)$ expected | Full scan (hashing destroys order) | Fast key lookup; useless for `<, >, BETWEEN` |

*Notation:* $N =$ total records, $R =$ record size (bytes), $B_{\text{block}} =$ block size (bytes), $Bfr = \lfloor B_{\text{block}} / R \rfloor =$ blocking factor, $B = \lceil N / Bfr \rceil =$ number of data blocks.

### Single-level ordered index types

An index is an auxiliary access structure stored on disk containing search keys and pointers.

```text
+-----------------------+-----------------------------+-----------------------------+
| Index Type            | Data File Physical Order?   | Search Key Property         | Dense or Sparse? |
+-----------------------+-----------------------------+-----------------------------+
| Primary Index         | Sorted on Search Key        | Key (unique values)         | Sparse (1 entry per block) |
| Clustered Index       | Sorted on Search Key        | Non-Key (duplicate values)  | Sparse (1 entry per distinct key) |
| Secondary Index (Key) | NOT sorted on Search Key    | Key (unique values)         | Dense (1 entry per record) |
| Secondary Index (Non) | NOT sorted on Search Key    | Non-Key (duplicates)        | Dense (pointers to bucket/list) |
+-----------------------+-----------------------------+-----------------------------+
```

1. **Primary Index:** Defined on the ordering primary key of a sorted file. Each entry contains `<first_key_in_block (anchor key), block_pointer>`.  
   $$\text{Number of entries} = B_{\text{data blocks}}$$
2. **Clustered Index:** Defined on the ordering non-key attribute of a sorted file (e.g., records sorted by `Dept_ID`). Each entry contains `<distinct_search_key, first_block_pointer>`.  
   $$\text{Number of entries} = \text{Number of distinct search key values}$$
3. **Secondary Index:** Defined on a non-ordering attribute. Because the data file is not sorted by this key, binary search on the data file is impossible. An index entry **must exist for every record**.  
   $$\text{Number of entries} = N_{\text{records}} \quad \text{(Dense)}$$

**Worked example 1 (Block accesses for primary and secondary indexes).**  
Given a data file with $N = 100{,}000$ records stored on disk:
- Block size $B_{\text{block}} = 4096$ bytes
- Record size $R = 100$ bytes
- Search key size $K = 16$ bytes
- Block pointer size $P = 6$ bytes
- Record pointer size $P_r = 7$ bytes

*Step 1: Data file blocking factor and blocks.*  
$$Bfr = \left\lfloor \frac{4096}{100} \right\rfloor = 40 \text{ records/block}$$
$$B = \left\lceil \frac{100{,}000}{40} \right\rceil = 2500 \text{ data blocks}$$
- Unindexed heap file equality search: $2500 / 2 = 1250$ average block accesses.
- Unindexed sorted file equality search: $\lceil \log_2 2500 \rceil = 12$ block accesses.

*Step 2: Primary index size and access cost.*  
Primary index is sparse: one entry per data block $\implies 2500$ entries.  
Index entry size $R_i = K + P = 16 + 6 = 22$ bytes.  
Index blocking factor:
$$Bfr_i = \left\lfloor \frac{4096}{22} \right\rfloor = 186 \text{ entries/block}$$
Number of index blocks:
$$b_i = \left\lceil \frac{2500}{186} \right\rceil = 14 \text{ blocks}$$
- Cost to find a record: Binary search on primary index $+ 1$ data block read:
$$\text{Cost} = \lceil \log_2 14 \rceil + 1 = 4 + 1 = 5 \text{ block accesses!}$$

*Step 3: Secondary index on candidate key.*  
Secondary index is dense: one entry per data record $\implies 100{,}000$ entries.  
Secondary index entry size: $R_{\text{sec}} = K + P_r = 16 + 7 = 23$ bytes.  
Secondary index blocking factor:
$$Bfr_{\text{sec}} = \left\lfloor \frac{4096}{23} \right\rfloor = 178 \text{ entries/block}$$
Number of secondary index blocks:
$$b_{\text{sec}} = \left\lceil \frac{100{,}000}{178} \right\rceil = 562 \text{ blocks}$$
- Cost to find a record: Binary search on secondary index $+ 1$ data block access:
$$\text{Cost} = \lceil \log_2 562 \rceil + 1 = 10 + 1 = 11 \text{ block accesses}$$

## 2. B-Tree structures and node capacity [PYQ: High]

A **B-tree** of order $p$ is a self-balancing multi-way search tree where all leaf nodes reside at the exact same depth. Unlike a B+ tree, a B-tree stores **search keys and record pointers in both internal nodes and leaf nodes**.

```text
B-Tree Node Layout:
+----+--------+----+--------+----+-----+--------+----+
| P1 | K1, Pr1| P2 | K2, Pr2| P3 | ... | Kk, Prk| Pp |
+----+--------+----+--------+----+-----+--------+----+
Where P_i = tree/child pointer, K_i = search key, Pr_i = record/data pointer
```

### Formal properties of a B-tree of order $p$

1. Every node has at most $p$ tree pointers and at most $p - 1$ search keys.
2. The root node has at least 2 tree pointers (unless the tree consists of only 1 node).
3. Every non-root node has:
   - At least $\lceil p / 2 \rceil$ tree pointers (child pointers).
   - At least $\lceil p / 2 \rceil - 1$ search keys and record pointers.
4. An internal node with $k$ tree pointers contains exactly $k - 1$ search keys.
5. All leaf nodes appear at the same bottom level.

### Calculating B-tree node order

For a block size $B_{\text{block}}$, block pointer size $P$, search key size $K$, and record pointer size $P_r$:
$$p \cdot P + (p - 1) \cdot (K + P_r) \le B_{\text{block}}$$
Solving for order $p$:
$$p \cdot (P + K + P_r) \le B_{\text{block}} + K + P_r \implies p = \left\lfloor \frac{B_{\text{block}} + K + P_r}{P + K + P_r} \right\rfloor$$

**Worked example 2 (B-tree order and key bounds).**  
Given a hard disk with block size $B_{\text{block}} = 1024$ bytes:
- Search key size $K = 10$ bytes
- Record pointer size $P_r = 8$ bytes
- Block pointer size $P = 6$ bytes

*Step 1: Compute maximum order $p$.*  
$$p \cdot 6 + (p - 1) \cdot (10 + 8) \le 1024$$
$$6p + 18p - 18 \le 1024 \implies 24p \le 1042$$
$$p = \left\lfloor \frac{1042}{24} \right\rfloor = 43$$
Verification: $43 \times 6 + 42 \times 18 = 258 + 756 = 1014 \le 1024$ bytes.  
Thus, the maximum order of the B-tree is **$p = 43$**.

*Step 2: Maximum and minimum capacity of non-root nodes.*
- **Maximum keys in a node:** $p - 1 = 43 - 1 = 42$ keys.
- **Maximum child pointers:** $p = 43$ pointers.
- **Minimum child pointers (non-root):** $\lceil 43 / 2 \rceil = 22$ pointers.
- **Minimum keys in a node (non-root):** $\lceil 43 / 2 \rceil - 1 = 22 - 1 = 21$ keys.

## 3. B+ Tree structure, order, height, and block accesses [PYQ: High]

The **B+ Tree** is the standard indexing structure in relational DBMS engines.
- **Internal nodes:** Store only search keys and child tree pointers to guide navigation. No data/record pointers!
- **Leaf nodes:** Store all search keys along with their data/record pointers. All actual data references reside exclusively at the leaves.
- **Leaf linked list:** Leaves are linked from left to right via a sibling pointer ($P_{\text{next}}$), enabling $O(1)$ progression across sequential range scans.

```text
Internal Node Layout:
+----+----+----+----+----+-----+----+----+
| P1 | K1 | P2 | K2 | P3 | ... | Kk | Pk |   (No record pointers!)
+----+----+----+----+----+-----+----+----+

Leaf Node Layout:
+---------+---------+-----+---------+-------+
| K1, Pr1 | K2, Pr2 | ... | Kq, Prq | Pnext |   (Contains all data pointers + sibling link)
+---------+---------+-----+---------+-------+
```

### Node capacity equations for B+ Tree

1. **Internal Node Order ($p$):**
   $$p \cdot P + (p - 1) \cdot K \le B_{\text{block}} \implies p = \left\lfloor \frac{B_{\text{block}} + K}{P + K} \right\rfloor$$
2. **Leaf Node Capacity ($p_{\text{leaf}}$):**
   $$p_{\text{leaf}} \cdot (K + P_r) + P \le B_{\text{block}} \implies p_{\text{leaf}} = \left\lfloor \frac{B_{\text{block}} - P}{K + P_r} \right\rfloor$$

### Tree height and block accesses

Let $N$ be the total number of records indexed:
- Number of leaf blocks required:
  - **Minimum leaves (nodes maximally packed):** $L_{\min} = \lceil N / p_{\text{leaf}} \rceil$
  - **Maximum leaves (nodes minimally packed, half-full):** $L_{\max} = \lceil N / \lceil p_{\text{leaf}} / 2 \rceil \rceil$
- **Height of B+ Tree ($h$):** Counting the root as level 1 and leaves as level $h$:
  - Level 1 (root): $\ge 2$ children
  - Level 2: $\ge 2 \cdot \lceil p / 2 \rceil$ children
  - Level $h$ (leaves): $\ge 2 \cdot (\lceil p / 2 \rceil)^{h - 2}$ leaf nodes
  - General formula for tree height:
    $$h \approx \lceil \log_{\text{fanout}} (L) \rceil + 1$$
- **Block accesses for single key search:**
  - Index traversal: $h$ block reads (one per level from root to leaf).
  - Fetching record: $+ 1$ block read (if unclustered data file; 0 if clustered and record is inline in leaf).
  $$\text{Total accesses} = h + 1$$
- **Block accesses for range query ($k_1 \le K \le k_2$ returning $m$ records):**
  - Locate starting key: $h$ block accesses to reach first leaf.
  - Scan leaf blocks: $\lceil m / p_{\text{leaf}} \rceil$ contiguous leaf blocks.
  - Access data records: $m$ random block accesses (worst-case unclustered).

**Worked example 3 (B+ Tree order, height, and query accesses).**  
Given a database with:
- Block size $B_{\text{block}} = 4096$ bytes
- Key size $K = 12$ bytes
- Tree/block pointer size $P = 8$ bytes
- Record pointer size $P_r = 8$ bytes
- Total records $N = 1{,}000{,}000$

*Step 1: Compute internal node order $p$.*  
$$p \cdot 8 + (p - 1) \cdot 12 \le 4096$$
$$8p + 12p - 12 \le 4096 \implies 20p \le 4108$$
$$p = \left\lfloor \frac{4108}{20} \right\rfloor = 205$$
Maximum internal node order: **$p = 205$**. (Fanout up to 205).

*Step 2: Compute leaf node capacity $p_{\text{leaf}}$.*  
$$p_{\text{leaf}} \cdot (12 + 8) + 8 \le 4096$$
$$20 \cdot p_{\text{leaf}} \le 4088 \implies p_{\text{leaf}} = \left\lfloor \frac{4088}{20} \right\rfloor = 204$$
Maximum leaf node capacity: **$p_{\text{leaf}} = 204$** key-pointer pairs.

*Step 3: Compute minimum height (nodes full).*
- Number of leaves: $L_{\min} = \lceil 1{,}000{,}000 / 204 \rceil = 4902$ leaf blocks.
- Level above leaves (Level $h-1$): $\lceil 4902 / 205 \rceil = 24$ internal nodes.
- Level above that (Level $h-2$): $\lceil 24 / 205 \rceil = 1$ node (Root!).
Tree levels: Root (Level 1) $\to$ Internal (Level 2) $\to$ Leaves (Level 3).  
**Minimum height = 3.**

*Step 4: Compute maximum height (nodes half-full).*
- Minimum keys per leaf: $\lceil 204 / 2 \rceil = 102$.
- Number of leaves: $L_{\max} = \lceil 1{,}000{,}000 / 102 \rceil = 9804$ leaf blocks.
- Minimum internal fanout: $\lceil 205 / 2 \rceil = 103$.
- Level above leaves: $\lceil 9804 / 103 \rceil = 96$ nodes.
- Root (level 1): holds 96 child pointers (valid since root can hold up to 205 pointers).  
**Maximum height = 3.** (Even when half-full, the tree fits in 3 levels!).

*Step 5: Disk accesses.*
- Search single key: $h + 1 = 3 + 1 = 4$ disk accesses.
- Range query retrieving 1000 contiguous records:
  - Traversal to first leaf: 3 accesses.
  - Leaf scan: $\lceil 1000 / 204 \rceil = 5$ leaf block accesses.
  - Data records (unclustered): up to 1000 data block accesses.
  - Total accesses = $3 + 5 + 1000 = 1008$ block accesses.

## 4. B/B+ Tree insertions and deletions [PYQ: Medium]

### B+ Tree insertion algorithm

1. Search for the appropriate leaf node $L$ where key $K$ belongs.
2. If $L$ has space ($< p_{\text{leaf}}$ keys): insert $(K, P_r)$ in sorted order.
3. If $L$ overflows ($p_{\text{leaf}} + 1$ keys):
   - **Split leaf $L$** into two leaves $L$ and $L'$.
   - The first $\lfloor (p_{\text{leaf}} + 1) / 2 \rfloor$ entries stay in $L$; the remaining move to $L'$. **Convention varies:** Elmasri–Navathe keep $\lceil (p_{\text{leaf}} + 1) / 2 \rceil$ in $L$ (the bigger half on the left). Both give valid B+ trees; when a GATE question depends on it, it states the split rule, so follow the question.
   - **Copy up:** The smallest key of $L'$ is copied up to the parent node as a separator (it remains in $L'$!).
4. If the parent node overflows ($> p - 1$ keys):
   - **Split internal node:** The middle key is **pushed up** (not copied!) to its parent.
   - If the root splits, a new root is created with 2 children; tree height increases by 1.

```text
Leaf Split:      Key is COPIED up (retains copy in leaf)
Internal Split:  Key is PUSHED up (removed from internal node)
```

**Worked example 4 (B+ Tree insertion trace).**  
Construct a B+ tree with order $p = 3$ (internal node: max 3 pointers, max 2 keys) and leaf capacity $p_{\text{leaf}} = 2$ (leaf: max 2 keys).  
Insert keys: `10, 20, 30, 40`.

1. Insert 10, 20:  
   Leaf: `[10, 20]`
2. Insert 30 (Leaf overflows: `[10, 20, 30]`):
   - Split leaf into `[10]` and `[20, 30]`.
   - Smallest key of right leaf (20) is **copied up** to become the root.
   ```text
           [ 20 ]
          /      \
      [ 10 ] -> [ 20, 30 ]
   ```
3. Insert 40:
   - Target leaf `[20, 30]` overflows: `[20, 30, 40]`.
   - Split leaf into `[20]` and `[30, 40]`.
   - Smallest key of right leaf (30) is copied up to root.
   - Root had `[20]`; now receives 30: `[20, 30]`.
   ```text
             [ 20, 30 ]
            /    |     \
      [ 10 ] -> [ 20 ] -> [ 30, 40 ]
   ```
   Height remains 2. All 4 keys exist in the linked leaf chain `[10] -> [20] -> [30, 40]`.

## 5. Hash-based indexing [PYQ: Medium]

Hashing provides $O(1)$ expected lookup by computing bucket index $h(k) = k \bmod M$.

### Static vs dynamic hashing

- **Static Hashing:** Fixed number of buckets $M$. When a bucket overflows, overflow chaining is used. Over time, long chains degrade search time to $O(N)$.
- **Extendible Hashing (Dynamic):** Accommodates dynamic growth using an in-memory directory of pointers to buckets on disk.
  - **Global depth ($d$):** Number of bits of the hash value used by the directory ($2^d$ directory entries).
  - **Local depth ($d'$):** Number of bits used by an individual bucket ($d' \le d$).
  - A bucket with local depth $d'$ is pointed to by $2^{d - d'}$ directory entries.

### Bucket overflow in extendible hashing

When bucket $B$ overflows on insertion:
1. **Case 1: $d' < d$ (Local depth strictly less than global depth):**
   - Split bucket $B$ into two buckets $B$ and $B_{\text{new}}$.
   - Rehash records between $B$ and $B_{\text{new}}$ using the $(d' + 1)$-th bit.
   - Increment local depth: $d' = d' + 1$.
   - Update directory pointers. **The directory size does NOT double!**
2. **Case 2: $d' = d$ (Local depth equals global depth):**
   - **Double the directory:** increment global depth $d = d + 1$ ($2^{d+1}$ entries).
   - Split bucket $B$, set both local depths to $d' + 1$.
   - Rehash records and adjust directory pointers.

**Worked example 5 (Extendible hashing split trace).**  
Suppose directory has global depth $d = 2$ (4 entries: `00, 01, 10, 11`).  
Bucket $A$ has local depth $d' = 1$ and is pointed to by entries `00` and `10` (shares last bit `0`).  
Bucket $B$ has $d' = 2$ pointed to by `01`.  
Bucket $C$ has $d' = 2$ pointed to by `11`.

- If Bucket $A$ overflows:  
  Since $d' = 1 < d = 2$, bucket $A$ splits into $A_0$ (handled by `00`) and $A_1$ (handled by `10`).  
  Local depths of $A_0$ and $A_1$ become 2.  
  **Global depth remains 2! Directory does not double.**
- If Bucket $B$ now overflows:  
  Since $d' = 2 == d = 2$, the directory **must double**!  
  Global depth becomes $d = 3$ (8 directory entries). Bucket $B$ splits with local depth 3.

## Formulas and facts to memorise

| Item | Formula / Value | When to use |
|---|---|---|
| B-Tree node order $p$ | $p \cdot P + (p - 1)(K + P_r) \le B_{\text{block}}$ | Maximum order of B-tree node |
| B+ Tree internal order $p$ | $p \cdot P + (p - 1)K \le B_{\text{block}}$ | Maximum order of B+ tree internal node |
| B+ Tree leaf capacity $p_{\text{leaf}}$ | $p_{\text{leaf}}(K + P_r) + P \le B_{\text{block}}$ | Maximum key-pointer capacity of B+ tree leaf |
| Min keys in B-Tree (non-root) | $\lceil p / 2 \rceil - 1$ | Minimum key occupancy constraint |
| Min keys in B+ leaf | $\lceil p_{\text{leaf}} / 2 \rceil$ | Half-full leaf occupancy constraint |
| Primary index entries | Equal to number of data blocks $B_{\text{data}}$ | Calculating primary index size |
| Secondary index entries | Equal to number of data records $N$ | Calculating secondary index size |
| Search I/O (B+ Tree) | $h$ (index reads) $+ 1$ (data read if unclustered) | Calculating disk accesses for key search |
| Directory size (Extendible Hash) | $2^d$ entries (where $d =$ global depth) | Extendible hashing memory requirements |

## GATE traps

- **B-Tree vs B+ Tree internal node formula:** B-tree internal nodes store *record pointers*; B+ tree internal nodes do *NOT*. Forgetting to include $P_r$ in B-tree or mistakenly including $P_r$ in B+ tree internal node calculation changes the computed order!
- **Order definition convention:** Some authors define order as max pointers ($p$), others as max keys ($m$). In standard GATE questions, order refers to the **maximum number of child pointers** a node can hold. Always check if the question states "each node can hold at most $k$ keys" or "order $p$".
- **Sparse index requires sorted data:** A sparse index cannot be constructed on an unordered data file because binary search on block boundaries is invalid without ordering.
- **Copy-up vs Push-up in B+ trees:** On leaf split, the separator key is **copied up** (remains in the leaf). On internal node split, the separator key is **pushed up** (deleted from the node).
- **Secondary index is always dense:** Even if defined on a candidate key, a secondary index must have an entry for *every record* because the underlying records are not stored in search-key order.

## Connections

- [Trees and BST](../07-data-structures/trees-and-bst.md) — Multi-way balanced search trees extend binary search trees to minimize disk I/O latency for secondary storage.
- [Relational model](relational-model-algebra-calculus.md) — Indexes provide the underlying access paths that relational query optimizers select to evaluate selection and join predicates.
- [Operating systems: File systems](../13-operating-systems/file-systems-and-disk-scheduling.md) — Disk block size and I/O access models in OS form the hardware foundation for database indexing.

## Practice

**Q1 (NAT).** A B+ tree index is to be built on a relation with search key length 15 bytes. Disk block size is 512 bytes, child block pointer is 6 bytes, and data record pointer is 9 bytes. The maximum order of an internal node in this B+ tree is _______.  
<details><summary>Answer</summary>

**Answer:** 25  
**Solution:**  
In a B+ tree internal node, only keys and child pointers are stored (no record pointers).  
Equation for internal node:
$$p \cdot P + (p - 1) \cdot K \le B_{\text{block}}$$
Substitute given values:
$$p \cdot 6 + (p - 1) \cdot 15 \le 512$$
$$6p + 15p - 15 \le 512 \implies 21p \le 527$$
$$p = \left\lfloor \frac{527}{21} \right\rfloor = 25$$
Check: $25 \times 6 + 24 \times 15 = 150 + 360 = 510 \le 512$ bytes.  
Thus, the maximum order is 25.
</details>

**Q2 (NAT).** For the same specifications as Q1 (block size 512 bytes, key 15 bytes, block pointer 6 bytes, record pointer 9 bytes), the maximum number of key-record pointer pairs that can fit into a leaf node is _______.  
<details><summary>Answer</summary>

**Answer:** 21  
**Solution:**  
A leaf node stores pairs of $(K + P_r)$ plus one block pointer $P$ to the next leaf:
$$p_{\text{leaf}} \cdot (K + P_r) + P \le B_{\text{block}}$$
Substitute values:
$$p_{\text{leaf}} \cdot (15 + 9) + 6 \le 512$$
$$24 \cdot p_{\text{leaf}} \le 512 - 6 = 506$$
$$p_{\text{leaf}} = \left\lfloor \frac{506}{24} \right\rfloor = 21$$
Check: $21 \times 24 + 6 = 504 + 6 = 510 \le 512$ bytes.  
Thus, maximum leaf capacity is 21.
</details>

**Q3 (MCQ).** Consider a B-tree of order 5 (maximum 5 child pointers per node). What is the minimum number of keys in any non-root internal node?  
(A) 1  
(B) 2  
(C) 3  
(D) 4  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
For a B-tree of order $p$, any non-root internal node must have at least $\lceil p / 2 \rceil$ child pointers.  
With $p = 5$:
$$\text{Minimum child pointers} = \lceil 5 / 2 \rceil = 3$$
The number of keys in an internal node with $k$ child pointers is $k - 1$.  
$$\text{Minimum keys} = 3 - 1 = 2$$
Therefore, option (B) is correct.
</details>

**Q4 (MSQ).** Which of the following statements is/are TRUE regarding database index structures?  
(A) A secondary index on a candidate key must be dense.  
(B) A clustered index can be created on an unordered data file.  
(C) In a B+ tree, the depth of all leaf nodes from the root is strictly equal.  
(D) Hashing is more efficient than B+ tree indexing for range queries.  
<details><summary>Answer</summary>

**Answer:** (A), (C)  
**Solution:**  
- (A) is TRUE: because the data file is not physically ordered on the secondary key, every key value must have an entry in the index (dense).
- (B) is FALSE: a clustered index requires the underlying data file to be ordered on the clustering attribute.
- (C) is TRUE: B+ trees are perfectly balanced; all leaves reside at the same bottom depth.
- (D) is FALSE: hash functions do not preserve ordering, making range queries require a full scan, whereas B+ trees support range scans via linked leaves.
</details>

**Q5 (NAT).** A database table has 30,000 records stored in a heap file. Block size is 1024 bytes and record size is 100 bytes. A secondary B+ tree index is constructed on a key attribute. The B+ tree leaf nodes can hold 20 key-pointer pairs, and internal nodes have order 10. The height of the B+ tree (with root as level 1) is _______.  
<details><summary>Answer</summary>

**Answer:** 5  
**Solution:**  
Total records $N = 30{,}000$.  
Leaf capacity $p_{\text{leaf}} = 20$.  
Number of leaf nodes required = $\lceil 30{,}000 / 20 \rceil = 1500$ leaf nodes.  
Internal nodes have maximum fanout $p = 10$.  
- Level $h$ (Leaves): 1500 nodes.
- Level $h-1$: $\lceil 1500 / 10 \rceil = 150$ internal nodes.
- Level $h-2$: $\lceil 150 / 10 \rceil = 15$ internal nodes.
- Level $h-3$: $\lceil 15 / 10 \rceil = 2$ internal nodes.
- Level $h-4$ (Root): $\lceil 2 / 10 \rceil = 1$ node (the root!).  
Counting levels from root (Level 1) to leaves:
Level 1 (Root) $\to$ Level 2 (2 nodes) $\to$ Level 3 (15 nodes) $\to$ Level 4 (150 nodes) $\to$ Level 5 (1500 leaves).  
Height of the B+ tree is 5.
</details>

**Q6 (MCQ).** In an extendible hashing scheme, the directory has global depth 3. A bucket with local depth 2 overflows upon an insertion. What will happen to the global depth and directory size?  
(A) Global depth becomes 4; directory size doubles.  
(B) Global depth remains 3; directory size remains unchanged.  
(C) Global depth becomes 2; directory size is halved.  
(D) Global depth becomes 4; directory size remains unchanged.  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
When a bucket overflows in extendible hashing:
- If local depth $d' < d$ (global depth), the bucket splits, its local depth increments by 1 ($d' = 2 + 1 = 3$), and directory pointers are redistributed.
- The global depth does **not** change, and the directory size ($2^d = 2^3 = 8$) remains unchanged.
Directory doubling only occurs when an overflowing bucket already has $d' = d$.  
Thus, option (B) is correct.
</details>

**Q7 (NAT).** In a B+ tree where each node occupies one disk block of size 2048 bytes, the key size is 16 bytes and child pointer size is 8 bytes. What is the maximum number of keys that can be stored in an internal node?  
<details><summary>Answer</summary>

**Answer:** 85  
**Solution:**  
Let $p$ be the maximum number of child pointers (order) of an internal node:
$$p \cdot P + (p - 1) \cdot K \le B_{\text{block}}$$
$$p \cdot 8 + (p - 1) \cdot 16 \le 2048$$
$$8p + 16p - 16 \le 2048 \implies 24p \le 2064$$
$$p = \left\lfloor \frac{2064}{24} \right\rfloor = 86$$
Check: $86 \times 8 + 85 \times 16 = 688 + 1360 = 2048 \le 2048$ bytes (fits exactly!).  
The maximum number of keys in an internal node is $p - 1$:
$$\text{Max keys} = 86 - 1 = 85$$
</details>
