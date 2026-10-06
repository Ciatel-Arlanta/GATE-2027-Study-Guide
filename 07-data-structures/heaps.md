# Binary Heaps and Priority Queues

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Binary heaps; Data-structure operations and complexity (priority queues)
> **Prerequisites:** [Trees and BST](trees-and-bst.md) (complete binary trees), [Arrays](arrays-stacks-queues.md) · **Leads to:** [Searching and sorting (heap sort)](../08-algorithms/searching-and-sorting.md), [Shortest paths](../08-algorithms/shortest-paths.md), [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md)

## Quick glance

- A **binary heap** is a **complete binary tree** stored in an array with the **heap property**: max-heap: `parent ≥ children`; min-heap: `parent ≤ children`. It is *not* sorted: only parent-child order is guaranteed.
- **Index formulas, 1-based:** parent `⌊i/2⌋`, children `2i`, `2i+1`. **0-based:** parent `⌊(i-1)/2⌋`, children `2i+1`, `2i+2`.
- Height = `⌊log₂ n⌋`; leaves occupy positions `⌊n/2⌋+1 .. n` (1-based): there are `⌈n/2⌉` leaves and `⌊n/2⌋` internal nodes.
- **Insert:** append at the end, **sift up**: O(log n). **Extract-max/min:** move the last element to the root, **sift down**: O(log n). **Peek max/min:** O(1). **Search:** O(n).
- **Build-heap (Floyd): sift down for `i = ⌊n/2⌋ .. 1` ⇒ O(n)** (not O(n log n)): most nodes are near the leaves and sift only a short distance.
- Building by `n` successive inserts costs O(n log n) worst case.
- **Heap sort:** build-heap + `n - 1` extract-max = O(n log n) time, O(1) extra space, not stable.
- The **minimum of a max-heap** is at a leaf; the **second largest** is a child of the root.
- Distinct max-heaps on keys `1..n`: n = 1..7 → 1, 1, 2, 3, 8, 20, 80.
- **d-ary heap:** height `log_d n`; insert O(log_d n), extract O(d·log_d n).

## 1. Priority queue and the heap

A **priority queue** supports `insert(x)`, `findMax/Min`, `extractMax/Min`, often `decreaseKey` and `delete`. A heap is the standard implementation.

| Implementation | insert | extract-min | find-min | decrease-key |
|---|---|---|---|---|
| Unsorted array/list | O(1) | O(n) | O(n) | O(1) (given position) |
| Sorted array (min at the end) | O(n) | O(1) | O(1) | O(n) |
| Sorted linked list | O(n) | O(1) | O(1) | O(n) |
| **Binary heap** | **O(log n)** | **O(log n)** | **O(1)** | **O(log n)** |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) |
| Fibonacci heap | O(1) amortised | O(log n) amortised | O(1) | O(1) amortised |

Why a **complete** tree stored in an array: no pointers needed, perfect packing, height stays `⌊log₂ n⌋` after each insert/delete (the last position is always the next/previous array cell).

```text
Max-heap on indices 1..10:        array: [16, 14, 10, 8, 7, 9, 3, 2, 4, 1]
             16 (1)
          /        \
      14 (2)       10 (3)
     /     \       /    \
   8 (4)  7 (5)  9 (6)  3 (7)
  /   \    /
 2(8) 4(9) 1(10)
```

**Index relations.** 1-based: node `i`: parent `⌊i/2⌋`, left `2i`, right `2i + 1`. 0-based: parent `⌊(i - 1)/2⌋`, left `2i + 1`, right `2i + 2`. Node `i` is a leaf iff `2i > n` (1-based), i.e. `i > ⌊n/2⌋`.

**Worked example 1 (index arithmetic).** For `n = 11` (1-based): leaves are 6..11 (6 leaves = ⌈11/2⌉); node 5 has children 10 and 11; node 11's parent is 5; node 6 has no children (12 > 11). In 0-based terms the same node as 1-based 11 is index 10, with parent `⌊9/2⌋ = 4` ✓ (1-based 5).

**Levels.** Level `l` (root = 0) occupies positions `2^l .. 2^(l+1) - 1` (1-based). Height of the heap with n = 10: 3; n = 1000: 9 (positions 512..1000 are on level 9).

## 2. Operations

### 2.1 Sift-up (insert / increase key)

Append the new element at position `n + 1`; while it is larger than its parent (max-heap), swap with the parent.

```c
void insert(int a[], int *n, int x) {          /* max-heap, 1-based */
    int i = ++(*n);
    a[i] = x;
    while (i > 1 && a[i/2] < a[i]) { int t = a[i]; a[i] = a[i/2]; a[i/2] = t; i /= 2; }
}
```
Cost O(height) = O(log n); **best case O(1)** (the new key is not larger than its parent).

**Worked example 2 (min-heap insertions).** Insert `20, 15, 30, 10, 25, 5` into an empty min-heap:

| Insert | Sift-up swaps | Array after |
|---|---|---|
| 20 | none | 20 |
| 15 | parent 20 > 15: swap | 15 20 |
| 30 | parent 15 < 30: none | 15 20 30 |
| 10 | at index 4, parent 20: swap; at index 2, parent 15: swap | 10 15 30 20 |
| 25 | at index 5, parent 15 < 25: none | 10 15 30 20 25 |
| 5 | at index 6, parent index 3 holds 30: swap; at index 3, parent 10: swap | **5 15 10 20 25 30** |

### 2.2 Sift-down (heapify)

`heapify(i)` assumes the subtrees of `i` are heaps and fixes position `i`: swap with the **larger** child (max-heap) while a child is bigger.

```c
void siftDown(int a[], int n, int i) {         /* max-heap, 1-based */
    while (2*i <= n) {
        int c = 2*i;
        if (c + 1 <= n && a[c+1] > a[c]) c++;    /* larger child */
        if (a[i] >= a[c]) break;
        int t = a[i]; a[i] = a[c]; a[c] = t; i = c;
    }
}
```
Per level: 2 comparisons (child vs child, parent vs larger child) and ≤ 1 swap. Cost O(height of the node).

### 2.3 Extract-max (delete root)

Replace the root with the **last** element, shrink the heap by one, sift down from the root.

**Worked example 3.** Max-heap `[16, 14, 10, 8, 7, 9, 3, 2, 4, 1]`, extract-max:
1. Remove 16, move last (1) to the root: `[1, 14, 10, 8, 7, 9, 3, 2, 4]`.
2. Root 1: children 14, 10 → swap with 14: `[14, 1, 10, 8, 7, 9, 3, 2, 4]`.
3. Index 2 (value 1): children 8, 7 → swap with 8: `[14, 8, 10, 1, 7, 9, 3, 2, 4]`.
4. Index 4 (value 1): children 2, 4 → swap with 4: `[14, 8, 10, 4, 7, 9, 3, 2, 1]`. No children left: done.

Returned 16, new array **`[14, 8, 10, 4, 7, 9, 3, 2, 1]`**, 3 swaps (= height). Cost O(log n).

### 2.4 Decrease-key, increase-key, delete

- **Increase-key** in a max-heap (or **decrease-key** in a min-heap): change the value, then **sift up**: O(log n).
- The opposite direction change: **sift down**: O(log n).
- **Delete arbitrary element at position `i`:** replace it with the last element, then sift up or down as needed: O(log n). (You must already know `i`: *finding* an element is O(n).)
- **Merge two heaps:** concatenate and rebuild: O(n); special structures (binomial, Fibonacci, leftist heaps) merge faster.

## 3. Building a heap

**Method A (Floyd's bottom-up):** the leaves are already heaps; for `i = ⌊n/2⌋` down to 1, `siftDown(i)`.

**Worked example 4.** `A = [4, 1, 3, 2, 16, 9, 10, 14, 8, 7]` into a max-heap (1-based, n = 10, start at i = 5):

| i | Value | Children | Action | Array after |
|---|---|---|---|---|
| 5 | 16 | 7 (idx 10) | 16 ≥ 7, ok | 4 1 3 2 16 9 10 14 8 7 |
| 4 | 2 | 14, 8 | swap with 14 | 4 1 3 14 16 9 10 2 8 7 |
| 3 | 3 | 9, 10 | swap with 10 | 4 1 10 14 16 9 3 2 8 7 |
| 2 | 1 | 14, 16 | swap with 16 (idx 5); then children of idx 5: 7 → swap with 7 | 4 16 10 14 7 9 3 2 8 1 |
| 1 | 4 | 16, 10 | swap with 16; at idx 2 children 14, 7 → swap with 14; at idx 4 children 2, 8 → swap with 8 | **16 14 10 8 7 9 3 2 4 1** |

Total swaps: 0 + 1 + 1 + 2 + 3 = 7.

**Method B (repeated insert):** inserting the same ten keys one at a time gives a *different* heap: `[16, 14, 10, 8, 7, 3, 9, 1, 4, 2]`. Heaps are not unique, so be careful when a question says "built by insertion" vs "heapify".

**Why Method A is O(n).** A node at height `k` (leaves have height 0) sifts down at most `k` levels. There are at most `⌈n / 2^(k+1)⌉` nodes of height `k`. Total work

`Σ_{k=0}^{⌊log n⌋} ⌈n / 2^(k+1)⌉ · k = O(n · Σ_{k≥0} k / 2^k) = O(n)`, since `Σ_{k≥0} k / 2^k = 2`.

Intuition: half of the nodes are leaves (cost 0), a quarter sift at most 1, an eighth at most 2, ... Method B instead sends half of the nodes (the lower level) up to `log n` levels: Θ(n log n) worst case (e.g. inserting ascending keys into a max-heap; each new key rises to the root). Method A makes at most about `2n` comparisons.

**Worked example 5 (counting nodes by height).** `n = 15` perfect tree: 8 nodes of height 0, 4 of height 1, 2 of height 2, 1 of height 3. Maximum sift-down swaps: `4·1 + 2·2 + 1·3 = 11` < n.

## 4. Heap sort

1. Build a max-heap: O(n).
2. For `i = n` down to 2: swap `a[1]` and `a[i]`, decrease the heap size, `siftDown(1)`. Each of the `n - 1` extractions is O(log n).

Total **Θ(n log n)** in best, average and worst case (the best case for distinct keys is still n log n up to constants); **in-place** (O(1) extra space), **not stable**. After `j` extractions the last `j` array positions hold the `j` largest elements in order.

**Worked example 6.** Heap `[16, 14, 10, 8, 7, 9, 3, 2, 4, 1]`: swap root with last → `[1, 14, 10, 8, 7, 9, 3, 2, 4 | 16]`; sift down → `[14, 8, 10, 4, 7, 9, 3, 2, 1 | 16]`; swap again → `[1, 8, 10, 4, 7, 9, 3, 2 | 14, 16]`, sift down → `[10, 8, 9, 4, 7, 1, 3, 2 | 14 16]`: the sorted suffix grows (`...14 16`). More in [searching and sorting](../08-algorithms/searching-and-sorting.md).

## 5. Uses and facts

- **Largest/smallest k elements:** build a min-heap of size k and scan the rest: O(n log k). **k-th smallest** by building a min-heap (O(n)) and `k` extract-mins (O(k log n)): O(n + k log n).
- **Merging k sorted lists:** keep a min-heap of k heads: O(N log k).
- **Dijkstra / Prim** use a priority queue with `decrease-key` ([shortest paths](../08-algorithms/shortest-paths.md), [MST](../08-algorithms/minimum-spanning-trees.md)): total O((V + E) log V) with a binary heap.
- **Huffman coding** repeatedly extracts the two smallest weights ([greedy algorithms](../08-algorithms/greedy-algorithms.md)).
- **Median maintenance:** a max-heap for the lower half and a min-heap for the upper half.
- **OS:** priority scheduling ([CPU scheduling](../13-operating-systems/cpu-scheduling.md)).

**Where things are in a max-heap of n nodes.**

| Question | Answer |
|---|---|
| Maximum | root, index 1 |
| Second largest | index 2 or 3 |
| Minimum | one of the leaves, positions `⌊n/2⌋+1 .. n` (not necessarily the last) |
| Third largest | among the children of the root's two children and the root's other child: positions 2..7 |
| Is the array sorted? | not necessarily; a sorted-descending array is a valid max-heap, but not conversely |
| Is the inorder of the tree sorted? | no |

**Number of heaps.** The number of distinct max-heaps on `n` distinct keys `1..n`: the root is `n`; choose which `k` keys go to the left subtree (`k` = size of the left subtree in the complete shape): `H(n) = C(n-1, k) · H(k) · H(n-1-k)`. Values: n = 3: 2; n = 4: 3; n = 5: 8; n = 7: 80.

**Worked example 7.** `n = 4`: complete shape has left subtree of size 2, right of size 1. Root = 4; choose 2 of the remaining 3 keys for the left: C(3,2) = 3; `H(2) = 1`, `H(1) = 1`: total 3. Check: positions 2 and 3 are the children of the root and position 4 hangs under position 2. The left pair {pos 2, pos 4} can be {3,1}, {3,2} or {2,1} (larger on top), with the remaining key at position 3: arrays `4 3 2 1`, `4 3 1 2`, `4 2 3 1`. (Three heaps.)

**n = 5 check:** root 5; left subtree has 3 nodes (shape: root with two children), right has 1: `C(4,3)·H(3)·H(1) = 4·2·1 = 8` ✓.

### 5.1 d-ary heaps

Each node has `d` children: 0-based children of `i` are `d·i + 1 .. d·i + d`, parent `⌊(i-1)/d⌋`. Height `log_d n`.

| Operation | Cost |
|---|---|
| insert / decrease-key (sift up) | O(log_d n) |
| extract-min (sift down, compare d children per level) | O(d · log_d n) |

With Dijkstra's `E` decrease-keys and `V` extractions, choosing `d ≈ E/V` gives O(E log_{E/V} V).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Parent / children (1-based) | ⌊i/2⌋; 2i, 2i+1 | index questions |
| Parent / children (0-based) | ⌊(i-1)/2⌋; 2i+1, 2i+2 | code with 0-based arrays |
| Height | ⌊log₂ n⌋ | depth of a heap |
| Leaves | positions ⌊n/2⌋+1..n; count ⌈n/2⌉ | where is the min |
| Insert | O(log n) (O(1) best) | cost |
| Extract | O(log n) | cost |
| Find max | O(1); find arbitrary key O(n) | cost |
| Build-heap | O(n), ≤ ~2n comparisons | build vs insert |
| n inserts | O(n log n) worst | build comparison |
| Heap sort | Θ(n log n), in place, not stable | sorting |
| Max-heaps on n keys | H(n) = C(n-1,k)·H(k)·H(n-1-k) | counting |
| d-ary | height log_d n; extract O(d log_d n) | variants |

## GATE traps

- **Heap ≠ sorted array;** the array of a heap is not in order, and an inorder traversal of the tree is not sorted.
- **Min of a max-heap** is in a leaf, not necessarily the last array element.
- **Build-heap is O(n)**, but **n successive inserts is O(n log n)**; the resulting arrays can differ.
- **Searching a heap** for an arbitrary key is O(n), not O(log n).
- **Sift-down picks the larger child** (max-heap) / smaller child (min-heap); picking the wrong child breaks the heap property.
- **0-based vs 1-based** index formulas: `2i` vs `2i + 1`; read the array convention.
- **Delete arbitrary node** may need sift-up *or* sift-down.
- **The k-th largest** is not at position k; you need k extractions (O(k log n)).
- **Height of a heap** with n nodes is `⌊log₂ n⌋`, not `⌈log₂ n⌉` (for n = 8: height 3).
- **Heap sort is not stable**, and best-case is not O(n) (unlike insertion sort).
- **Duplicates** do not change the structure, but the count of distinct heaps formula assumes distinct keys.

## Connections

- [Trees and BST](trees-and-bst.md) — complete binary trees, height `⌊log₂ n⌋`; a BST orders left-vs-right, a heap orders parent-vs-child.
- [Arrays, stacks, queues](arrays-stacks-queues.md) — priority queue vs FIFO queue; array storage of a tree.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — heap sort; why comparison sorts need Ω(n log n).
- [Shortest paths](../08-algorithms/shortest-paths.md) — Dijkstra with a heap (decrease-key).
- [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) — Prim's algorithm with a priority queue.
- [Greedy algorithms](../08-algorithms/greedy-algorithms.md) — Huffman coding, job scheduling with a heap.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — the O(n) build-heap sum is a classic series bound.
- [CPU scheduling](../13-operating-systems/cpu-scheduling.md) — priority and shortest-job queues.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — counting heaps with binomial coefficients.
- [Graph traversals](../08-algorithms/graph-traversals.md) — best-first search uses a priority queue.
- [Search (AI)](../17-artificial-intelligence/search.md) — the frontier of uniform-cost and A* search is a priority queue.

## Practice

**Q1 (NAT, easy).** In a max-heap stored 1-based in an array of 25 elements, what is the index of the parent of the element at index 17, and how many leaves are there?  (give "parent,leaves")

<details><summary>Answer</summary>

**Answer:** 8,13  
**Solution:** Parent `⌊17/2⌋ = 8`. Leaves: positions `⌊25/2⌋ + 1 = 13` to 25, so 13 leaves (= ⌈25/2⌉).

</details>

**Q2 (NAT).** Height of a binary heap with 1000 elements (single node = height 0)?

<details><summary>Answer</summary>

**Answer:** 9  
**Solution:** `⌊log₂ 1000⌋ = 9` since 512 ≤ 1000 < 1024.

</details>

**Q3 (MCQ).** Build a max-heap from `[4, 1, 3, 2, 16, 9, 10, 14, 8, 7]` using bottom-up heapify. The resulting array is  (A) 16 14 10 8 7 9 3 2 4 1  (B) 16 14 10 8 7 3 9 1 4 2  (C) 16 10 14 7 8 9 3 2 4 1  (D) 16 14 9 8 10 7 3 2 4 1

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** Worked example 4. (B) is the result of inserting the keys one at a time.

</details>

**Q4 (NAT).** How many distinct max-heaps (as arrays) can be formed from the five distinct keys 1..5?

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:** Root 5; shape has 3 nodes on the left and 1 on the right: `C(4,3)·H(3)·H(1) = 4·2·1 = 8`.

</details>

**Q5 (MCQ).** In a max-heap with 15 elements, where can the minimum element be located (1-based indices)?  (A) only index 15  (B) any of indices 8..15  (C) any index  (D) indices 2 or 3

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** The minimum has no smaller child, so it must be a leaf; leaves are indices `⌊15/2⌋ + 1 = 8` through 15.

</details>

**Q6 (NAT).** The keys `4, 1, 3, 2, 16, 9, 10, 14, 8, 7` are inserted one by one (sift-up) into an empty max-heap. What is the value at index 6 (1-based) of the final array?

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** The final array is `[16, 14, 10, 8, 7, 3, 9, 1, 4, 2]` (Method B); index 6 holds 3.

</details>

**Q7 (MSQ).** Which are true? (A) Building a heap from n arbitrary elements takes O(n) time. (B) Finding the minimum of a max-heap takes O(1). (C) Deleting the maximum takes O(log n). (D) Heap sort is stable. (E) The 2nd largest of a max-heap is at index 2 or 3.

<details><summary>Answer</summary>

**Answer:** A, C, E  
**Solution:** (B) the minimum is in some leaf: O(n) to find. (D) heap sort is not stable.

</details>

**Q8 (NAT, harder).** For the build in Worked example 4 (`[4,1,3,2,16,9,10,14,8,7]`, bottom-up), what is the total number of element swaps performed?

<details><summary>Answer</summary>

**Answer:** 7  
**Solution:** i=5: 0; i=4: 1 (2↔14); i=3: 1 (3↔10); i=2: 2 (1↔16, then 1↔7); i=1: 3 (4↔16, 4↔14, 4↔8). Total 7.

</details>
