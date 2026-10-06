# Data-Structure Operations: Complexity Reference

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Data-structure operations and complexity
> **Prerequisites:** [Arrays, stacks, queues](arrays-stacks-queues.md), [Linked lists](linked-lists.md), [Trees and BST](trees-and-bst.md), [Heaps](heaps.md), [Graphs](graphs.md) · **Leads to:** [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md), [Hashing](../08-algorithms/hashing.md), [Searching and sorting](../08-algorithms/searching-and-sorting.md)

This page is the one-stop table of "what does operation X cost on structure Y". Every entry is justified in the chapter named in the last column. `n` = number of stored elements, `m` = size of a second structure, `h` = tree height, `α = n/M` = load factor of a hash table with `M` slots, `deg` = vertex degree.

## Quick glance

- **Array:** O(1) access; O(n) search (O(log n) if sorted); O(n) insert/delete in the middle.
- **Linked list:** O(1) at the head; O(n) to search or reach the i-th; delete a given node O(1) only if doubly linked (or via the copy trick).
- **Stack/queue/deque:** push, pop, enqueue, dequeue = O(1); search O(n).
- **BST:** O(h) for search/insert/delete: **O(log n) average, O(n) worst**. **AVL: O(log n) worst.**
- **Binary heap:** find-min O(1), insert/extract/decrease-key O(log n), **build O(n)**, search O(n).
- **Hash table:** search/insert/delete **O(1) average**, **O(n) worst**; no ordered operations.
- **Graph:** matrix O(n²) space, O(1) edge test; list O(n + e) space, O(deg) edge test; BFS/DFS O(n²) vs O(n + e).
- **Rule:** to pick a structure, count the operation mix: frequent min-extraction → heap; frequent lookups → hash table; ordered traversal / range queries → balanced BST; O(1) LIFO/FIFO → stack/queue.

## 1. Master table

Costs are **worst case unless marked (avg)**. "Insert" means insert at a position you must still locate unless stated.

| Structure | Access by index | Search | Insert | Delete | Find min | Merge two |
|---|---|---|---|---|---|---|
| **Unsorted array** | O(1) | O(n) | O(1) at end¹, O(n) elsewhere | O(n) (O(1) swap-with-last if position known) | O(n) | O(n + m) |
| **Sorted array** | O(1) | **O(log n)** (binary search) | O(n) (shifting) | O(n) | **O(1)** | O(n + m) |
| **Singly linked list** (head only) | O(n) | O(n) | **O(1)** at head, O(n) at tail | O(1) head; O(n) general | O(n) | O(n) find tail, then O(1) |
| **Sorted linked list** | O(n) | O(n) | O(n) | O(n) | O(1) | O(n + m) |
| **Doubly linked list** | O(n) | O(n) | O(1) at either end/after a given node | **O(1)** given the node | O(n) | O(1) with tail pointer |
| **Stack** (array/list) | n/a | O(n) | push O(1) | pop O(1) | O(n) (O(1) with auxiliary min-stack) | O(n + m) |
| **Queue / deque** | n/a | O(n) | O(1) | O(1) | O(n) | O(m) |
| **Binary search tree** | O(h) by rank only with size info | O(h): avg O(log n), **worst O(n)** | O(h) | O(h) | O(h) (leftmost) | O(n + m) |
| **AVL tree** | O(log n) | **O(log n)** | O(log n) | O(log n) | O(log n) | O(n + m) |
| **Binary min-heap** | n/a | O(n) | O(log n) (avg O(1) for random keys) | extract-min O(log n); arbitrary O(log n) given index | **O(1)** | O(n + m) rebuild |
| **Hash table** (chaining) | n/a | avg O(1 + α), worst O(n) | avg O(1), worst O(n)² | avg O(1), worst O(n) | O(n + M) | O(n + m) |
| **Hash table** (open addressing) | n/a | avg O(1/(1-α)) probes, worst O(n) | same | same (needs tombstones) | O(M) | O(n + m) |

¹ Amortised O(1) for a dynamic (doubling) array; O(n) if the array is full and must be copied.
² Insertion into a chained table is O(1) if duplicates are not checked, O(chain length) if they are.

### Graph representations (n vertices, e edges)

| Representation | Space | Edge `(u,v)`? | Neighbours of `v` | Add edge | Delete edge | BFS/DFS |
|---|---|---|---|---|---|---|
| Adjacency matrix | Θ(n²) | **O(1)** | Θ(n) | O(1) | O(1) | O(n²) |
| Adjacency list | Θ(n + e) | O(deg u) | **O(deg v)** | O(1) | O(deg) | **O(n + e)** |
| Edge list | Θ(e) | O(e) | O(e) | O(1) | O(e) | O(n·e) naive |
| Incidence matrix | Θ(n·e) | O(e) | O(e) | O(n) | O(n) | impractical |

### Space

| Structure | Space | Note |
|---|---|---|
| Array | n·w | dynamic array: up to 2n·w |
| Singly / doubly list | n(w + 1 pointer) / n(w + 2 pointers) | allocator overhead extra |
| BST / AVL | n(w + 2 pointers) (+ height field for AVL) | |
| Heap | n·w | no pointers (complete tree in an array) |
| Hash table | M slots + n entries | `M ≈ n/α` |

## 2. Reading the table: the reasoning behind each row

- **Arrays** give O(1) access because the address is computed (`base + i·w`); the price is shifting for insertion/deletion. Binary search needs sortedness **and** random access.
- **Linked lists** trade access for cheap relinking: no shifting, but no jumping. Binary search on a list is **not** O(log n) (reaching the middle costs O(n)).
- **Stack/queue** restrict access to one or two ends so that those operations are O(1) in either an array (with `top` / circular `front, rear`) or a list.
- **BST** costs one comparison per level: cost = height. Balanced ⇒ `⌊log₂ n⌋ ≤ h`; skewed ⇒ `h = n - 1` ([trees](trees-and-bst.md)).
- **Heap** orders only parent versus child, which is exactly enough for O(1) min and O(log n) updates; it cannot search quickly because siblings and cousins are unordered.
- **Hash** trades order for speed: with a good function, expected chain length is α; the worst case is all keys in one slot ([hashing](../08-algorithms/hashing.md)).

## 3. Amortised analysis and bulk construction

**Dynamic array (append with doubling).** When the array is full, allocate double the capacity and copy.

**Worked example 1.** Append 16 elements to an empty array of capacity 1 (double when full). Copies occur when inserting the 2nd (1 copy), 3rd (2), 5th (4), 9th (8) elements: `1 + 2 + 4 + 8 = 15` copies. Total work = 16 writes + 15 copies = **31 operations**, i.e. `< 2` per append: **amortised O(1)**. Growing by a **constant** `c` instead of doubling copies `c + 2c + ... ≈ n²/(2c)`: amortised O(n).

**Stack with `multipop`, queue from two stacks:** each element is pushed and popped O(1) times overall, so any sequence of `m` operations costs O(m) ([stacks and queues](arrays-stacks-queues.md)).

**Building from `n` elements**

| Structure | Build time | Note |
|---|---|---|
| Heap from an array | **O(n)** | bottom-up heapify |
| Heap by `n` inserts | O(n log n) | |
| Sorted array | O(n log n) | comparison sort lower bound |
| BST by `n` random inserts | O(n log n) average | worst **O(n²)** for sorted input |
| BST/AVL from a sorted array | O(n) | pick the middle recursively |
| AVL by `n` inserts | O(n log n) | |
| Hash table | O(n) average | |
| Adjacency list from an edge list | O(n + e) | |
| Adjacency matrix | O(n² + e) | initialise `n²` cells |

**Worked example 2 (total comparisons).** Insert `n = 10` keys in **increasing** order into an empty BST: key `k` (0-based) is compared with all `k` earlier keys (a chain), so total comparisons `= 0 + 1 + ... + 9 = 45 = n(n-1)/2`. An AVL tree on the same input ends with height 3 and uses 25 comparisons in total.

**Worked example 3 (choosing).** A job needs `n` inserts and then `n` extract-min operations.

| Structure | Insert phase | Extract phase | Total |
|---|---|---|---|
| Unsorted array | O(n) | O(n²) | O(n²) |
| Sorted array | O(n²) | O(n) | O(n²) |
| BST (average) / AVL | O(n log n) | O(n log n) | O(n log n) |
| **Binary heap** | O(n log n) (or O(n) build) | O(n log n) | **O(n log n)**, small constants, no pointers |

If extractions are replaced by searches, a hash table gives O(n) expected; for range or sorted listing queries a balanced BST is required.

**Worked example 4 (hashing load).** `n = 1000` keys, `M = 400` chained slots: `α = 2.5`; average chain length 2.5; expected comparisons ≈ `1 + α` ≈ 3.5 for an unsuccessful search, `1 + α/2` ≈ 2.25 for a successful one. For open addressing with `α = 0.5`: ≈ `1/(1 - α) = 2` probes (unsuccessful), `(1/α) ln(1/(1-α)) ≈ 1.39` probes (successful). Open addressing requires `α < 1`.

## 4. Other quick results

| Topic | Result |
|---|---|
| Comparison sort lower bound | Ω(n log n) comparisons (decision tree with n! leaves has height ≥ log₂ n!) |
| Heap sort / merge sort | Θ(n log n); heap in place, merge needs O(n) extra |
| Binary search | ⌊log₂ n⌋ + 1 comparisons worst case |
| BST traversal | O(n) time, O(h) stack |
| Height of AVL / complete tree | ≤ 1.44 log₂ n / ⌊log₂ n⌋ |
| Number of BSTs on n keys | Catalan C(2n,n)/(n+1) |
| Hash: expected chain | α = n/M |
| Min nodes AVL height h | N(h) = N(h-1)+N(h-2)+1 |
| Queue by two stacks | O(1) amortised per op |
| Dijkstra | O(n²) array; O((n+e) log n) heap |
| Linked list cycle detection | O(n) time, O(1) space |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Array access | O(1); search O(n) or O(log n) sorted | baseline |
| Insert/delete in array | O(n) shifts | comparison with lists |
| List insert at head | O(1) | stacks |
| Delete last of singly list | O(n) even with tail | trap |
| BST | O(h): log n avg, n worst | search/insert/delete |
| AVL | O(log n) worst | guaranteed bounds |
| Heap | min O(1), insert/extract O(log n), build O(n) | priority queue |
| Hash | O(1) avg, O(n) worst | dictionary |
| Open addressing probes | 1/(1-α) unsuccessful | analysis |
| Doubling array | amortised O(1) append | dynamic arrays |
| Graph | matrix O(n²) space / list O(n+e) | representation |

## GATE traps

- **"Average" vs "worst" for BST and hash tables:** BST worst is O(n) and hash worst is O(n); only AVL/balanced guarantees O(log n).
- **Binary search needs random access:** O(n) on linked lists.
- **Heap search is O(n)**; finding the minimum of a *max-heap* is also O(n).
- **Build-heap O(n)** vs sequential inserts O(n log n).
- **Deleting the last node** of a singly linked list is O(n) even with a tail pointer; doubly linked gives O(1).
- **Insert in a sorted array** is O(n) (shifting), although the position is found in O(log n).
- **Sorted linked list insert** is O(n) (finding the position), not O(1).
- **Matrix BFS is O(n²)** even for sparse graphs.
- **Amortised ≠ average-case ≠ worst-case:** amortised bounds are worst-case guarantees over a *sequence* of operations.
- **Hash tables do not support min or range queries efficiently**.
- **Deleting from an open-addressing table** needs tombstones (marking), or later searches break.

## Connections

- [Arrays, stacks, queues](arrays-stacks-queues.md), [Linked lists](linked-lists.md), [Trees and BST](trees-and-bst.md), [Heaps](heaps.md), [Graphs](graphs.md) — the chapters that derive each entry.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — notation and recurrences behind the bounds.
- [Hashing](../08-algorithms/hashing.md) — collision resolution, load factor, expected probes.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — sorted-array search, sorting costs, lower bound.
- [Graph traversals](../08-algorithms/graph-traversals.md), [Shortest paths](../08-algorithms/shortest-paths.md), [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) — algorithm costs depend on the representation and the priority queue.
- [Dynamic programming](../08-algorithms/dynamic-programming.md) — table lookups assume O(1) array access.
- [File organization and indexing](../14-databases/file-organization-and-indexing.md) — B-trees and hashing on disk; costs counted in block accesses.
- [Memory hierarchy and cache](../10-computer-organization/memory-hierarchy-and-cache.md) — arrays beat lists in practice due to locality.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — counting BSTs, decision-tree leaves `n!`.

## Practice

**Q1 (MCQ, easy).** Which structure gives O(1) *worst-case* access to the i-th element?  (A) singly linked list  (B) BST  (C) array  (D) hash table

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Array address arithmetic is a constant-time computation. A hash table has no notion of index order.

</details>

**Q2 (NAT).** Elements 1, 2, ..., 10 are inserted in this order into an initially empty BST. What is the total number of key comparisons made by all insertions?

<details><summary>Answer</summary>

**Answer:** 45  
**Solution:** The tree is a right chain; inserting the k-th element (k = 1..10) compares it with the k - 1 existing keys: `0 + 1 + ... + 9 = 45`.

</details>

**Q3 (NAT).** Elements are appended to a dynamic array that starts with capacity 1 and doubles when full. How many element copies occur in total when 32 elements are appended?

<details><summary>Answer</summary>

**Answer:** 31  
**Solution:** Resizes happen when inserting the 2nd, 3rd, 5th, 9th, 17th elements, copying 1, 2, 4, 8, 16 elements: `1 + 2 + 4 + 8 + 16 = 31` copies, which is under n = 32: amortised O(1).

</details>

**Q4 (MCQ).** The best data structure for a workload of many `insert` and `extract-min` operations is  (A) sorted array  (B) unsorted linked list  (C) binary min-heap  (D) hash table

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Heap: O(log n) for both. Sorted array: O(n) insert; unsorted list: O(n) extract-min; a hash table cannot find the minimum.

</details>

**Q5 (MSQ).** Which operations are O(log n) in the *worst case* for an AVL tree and O(n) in the worst case for a plain BST? (A) search  (B) insert  (C) delete  (D) find minimum  (E) inorder traversal

<details><summary>Answer</summary>

**Answer:** A, B, C, D  
**Solution:** These all cost O(height): O(log n) for AVL, O(n) for a skewed BST. Inorder traversal is Θ(n) in both.

</details>

**Q6 (NAT).** A chained hash table has 500 slots and stores 1250 keys. What is the expected number of keys in a chain (load factor)?

<details><summary>Answer</summary>

**Answer:** 2.5  
**Solution:** `α = n / M = 1250 / 500 = 2.5`.

</details>

**Q7 (MCQ).** In a singly linked list with both `head` and `tail` pointers, which operation is still O(n)?  (A) insert at head  (B) insert at tail  (C) delete the head  (D) delete the tail

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:** To delete the tail you must update the predecessor's `next`, and finding the predecessor requires a walk from the head.

</details>

**Q8 (NAT, harder).** The maximum possible height (edges) of an AVL tree with 12 nodes is?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** Minimum nodes: N(3) = 7, N(4) = 12, N(5) = 20. With 12 nodes height 4 is attainable (N(4) = 12 ≤ 12) but height 5 needs ≥ 20 nodes. Maximum height = 4.

</details>
