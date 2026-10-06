# Searching and Sorting

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Linear search; Binary search; Selection sort; Bubble sort; Insertion sort; Merge sort; Quicksort (plus heap, counting, radix sort, selection problems)
> **Prerequisites:** [Asymptotic analysis](asymptotic-analysis.md) · [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) · [Heaps](../07-data-structures/heaps.md) · **Leads to:** [Divide and conquer](divide-and-conquer.md) · [Hashing](hashing.md)

## Quick glance

- **Linear search:** $\le n$ comparisons, works on anything. **Binary search:** needs sorted array, $\lfloor\log_2n\rfloor+1$ comparisons worst case.
- **Selection sort:** always $\Theta(n^2)$ comparisons, at most $n-1$ swaps, not stable.
- **Bubble sort** (with early exit): best $n-1$ comparisons on sorted input, $\Theta(n^2)$ worst; **swaps = inversions**.
- **Insertion sort:** best $\Theta(n)$, worst $\Theta(n^2)$; **element shifts = inversions**; best for nearly sorted data.
- **Merge sort:** $\Theta(n\log n)$ always, stable, $O(n)$ extra space; worst-case comparisons $n\lceil\log n\rceil-2^{\lceil\log n\rceil}+1$.
- **Quicksort:** average $\Theta(n\log n)$, worst $\Theta(n^2)$ (already-sorted input with first/last pivot), in place, not stable; randomised pivot gives expected $O(n\log n)$.
- **Heap sort:** $\Theta(n\log n)$ always, in place, not stable; build-heap is $O(n)$.
- **Lower bound:** any comparison sort needs $\Omega(n\log n)$ comparisons ($\log_2 n!$ by the decision tree). **Counting/radix** beat it by not comparing.
- **#1 trap:** stability and in-place-ness differ across algorithms; "best case $O(n)$" holds only for bubble (with flag) and insertion sort.

## 1. Linear search

Scan from the left until the key is found or the list ends. No ordering needed.

```c
int linear(int a[], int n, int x) {
    for (int i = 0; i < n; i++)
        if (a[i] == x) return i;
    return -1;
}
```

| Case | Comparisons |
|---|---|
| Best | 1 (key at index 0) |
| Worst | $n$ (key last or absent) |
| Average, key present uniformly | $(n+1)/2$ |
| Average, absent with prob. $q$ | $(1-q)\frac{n+1}2+qn$ |

Time $\Theta(n)$ worst, space $O(1)$. With a **sentinel** (put $x$ at $a[n]$) the loop drops the bound test.

## 2. Binary search

**Intuition:** guess the middle of a sorted list; one comparison throws away half the candidates. Like finding a word in a dictionary.

```c
int binary(int a[], int n, int x) {
    int lo = 0, hi = n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;      // avoids overflow of lo+hi
        if (a[mid] == x) return mid;
        else if (a[mid] < x) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

**Worked example 1.** Search $x=23$ in `a = [2,5,8,12,16,23,38,56,72,91]` ($n=10$, indices 0..9).

| Step | lo | hi | mid | a[mid] | Action |
|---|---|---|---|---|---|
| 1 | 0 | 9 | 4 | 16 | $16<23$: lo=5 |
| 2 | 5 | 9 | 7 | 56 | $56>23$: hi=6 |
| 3 | 5 | 6 | 5 | 23 | found |

Three comparisons. **Recurrence:** $T(n)=T(n/2)+1\Rightarrow\Theta(\log n)$. The maximum number of loop iterations is $\lfloor\log_2n\rfloor+1$ (for $n=15$: 4; for $n=10$: 4).

For a full tree with $n=2^k-1$ elements, successful search average is $\frac{(k-1)2^k+1}{n}$; for $n=15$: $49/15\approx3.27$. Unsuccessful search takes $\lfloor\log_2 n\rfloor+1$ or $+0$ iterations depending on where it falls off.

**Off-by-one traps.** (1) `hi = n` with `while (lo < hi)` needs `hi = mid` not `mid-1`. (2) `while (lo < hi)` with `hi = n-1` misses the case of one element. (3) With `mid = (lo+hi)/2` and `lo = mid` (no +1) the loop never ends when `hi = lo+1`. (4) `mid` uses integer division, so for even sizes it is the lower middle, which matters when you count comparisons or asked "which element is probed first".

**Binary search on a linked list** is not $O(\log n)$: reaching the middle costs $O(n)$. On an unsorted array it is simply wrong.

## 3. Elementary sorts

An **inversion** is a pair $i<j$ with $a[i]>a[j]$. A sorted array has $0$ inversions; a reversed array of $n$ distinct items has the maximum, $\binom n2=\frac{n(n-1)}2$. The average over random permutations is $\frac{n(n-1)}4$.

A sort is **stable** if equal keys keep their input order. **In-place** means $O(1)$ (or $O(\log n)$) extra space. **Adaptive** means it runs faster on nearly sorted input.

### 3.1 Selection sort

**Idea:** find the minimum of the unsorted suffix and swap it to the front.

```c
for (i = 0; i < n-1; i++) {
    m = i;
    for (j = i+1; j < n; j++) if (a[j] < a[m]) m = j;
    swap(&a[i], &a[m]);
}
```

**Worked example 2.** Sort `[64,25,12,22,11]`.

| After pass | Array |
|---|---|
| 1 (min 11 swapped to index 0) | 11 25 12 22 64 |
| 2 (min 12) | 11 12 25 22 64 |
| 3 (min 22) | 11 12 22 25 64 |
| 4 (min 25, already placed) | 11 12 22 25 64 |

Comparisons: always $(n-1)+(n-2)+\dots+1=\frac{n(n-1)}2=10$ here. Swaps $\le n-1$. **Not stable:** `[5a,5b,2]`: swapping 2 into front sends `5a` behind `5b`. It is the sort with the fewest writes.

### 3.2 Bubble sort

**Idea:** repeatedly compare neighbours and swap if out of order; after pass $k$ the $k$ largest are in place.

```c
for (i = 0; i < n-1; i++) {
    swapped = 0;
    for (j = 0; j < n-1-i; j++)
        if (a[j] > a[j+1]) { swap(&a[j], &a[j+1]); swapped = 1; }
    if (!swapped) break;           // early exit
}
```

**Worked example 3.** `[5,1,4,2,8]` ($n=5$).

| Pass | Compares and swaps | Array after |
|---|---|---|
| 1 | (5,1)s (5,4)s (5,2)s (5,8) | 1 4 2 5 8 |
| 2 | (1,4) (4,2)s (4,5) | 1 2 4 5 8 |
| 3 | (1,2)(2,4) no swaps, stop | 1 2 4 5 8 |

Swaps: $3+1=4$ = number of inversions of `[5,1,4,2,8]`: pairs (5,1),(5,4),(5,2),(4,2) $=4$. ✓. **Each adjacent swap removes exactly one inversion**, so bubble sort does exactly (#inversions) swaps.

Comparisons with early exit: best case (sorted) $n-1$; worst $\frac{n(n-1)}2$. Without the flag it is always $\frac{n(n-1)}2$. Stable, in place, adaptive (with flag).

### 3.3 Insertion sort

**Idea:** like sorting a hand of cards: take the next card, slide it left past larger cards.

```c
for (i = 1; i < n; i++) {
    key = a[i]; j = i - 1;
    while (j >= 0 && a[j] > key) { a[j+1] = a[j]; j--; }
    a[j+1] = key;
}
```

**Worked example 4.** `[5,2,4,6,1,3]`.

| i | key | Array after insertion | Shifts |
|---|---|---|---|
| 1 | 2 | 2 5 4 6 1 3 | 1 |
| 2 | 4 | 2 4 5 6 1 3 | 1 |
| 3 | 6 | 2 4 5 6 1 3 | 0 |
| 4 | 1 | 1 2 4 5 6 3 | 4 |
| 5 | 3 | 1 2 3 4 5 6 | 3 |

Total shifts $=1+1+0+4+3=9$ = inversions of the input. **Number of shifts (inner-loop body executions) equals the inversion count; comparisons = inversions + up to $n-1$.** Best case (sorted) $n-1$ comparisons, $\Theta(n)$; worst (reversed) $\frac{n(n-1)}2$. Stable, in place, adaptive, online. Using binary search to find the slot cuts comparisons to $O(n\log n)$ but shifts remain $O(n^2)$.

If every element is at most $k$ places from its final position, insertion sort runs in $O(nk)$.

## 4. Merge sort

**Idea:** split in half, sort each half recursively, **merge** two sorted halves with two pointers.

```c
void msort(int a[], int l, int r) {
    if (l >= r) return;
    int m = (l + r) / 2;
    msort(a, l, m); msort(a, m+1, r);
    merge(a, l, m, r);              // uses a temporary array of size r-l+1
}
```

**Merging two sorted lists of sizes $m$ and $n$:**

| Case | Comparisons | When |
|---|---|---|
| Worst | $m+n-1$ | elements interleave so neither list empties until the end |
| Best | $\min(m,n)$ | all of one list is smaller than the other |

**Worked example 5.** Merge `[1,4,7]` and `[2,3,9]`: compare 1/2→1, 4/2→2, 4/3→3, 4/9→4, 7/9→7; the left list is empty, copy 9. Comparisons $=5=m+n-1$. Result `[1,2,3,4,7,9]`.

**Recurrence:** $T(n)=2T(n/2)+\Theta(n)\Rightarrow\Theta(n\log n)$ in every case. Exact comparison counts for $n=2^k$:

- **Worst:** $n\log_2n-n+1$. For $n=8$: $24-8+1=17$.
- **Best:** $\frac n2\log_2n$. For $n=8$: $12$.

(Both confirmed by running merge sort on all $8!$ permutations: minimum 12, maximum 17.) For general $n$, worst $=n\lceil\log_2 n\rceil-2^{\lceil\log_2n\rceil}+1$.

Space $O(n)$ for the buffer plus $O(\log n)$ stack. **Stable** if the merge takes from the left list on ties. Good for linked lists (no buffer needed) and external sorting. Bottom-up merge sort avoids recursion.

## 5. Quicksort

**Idea:** pick a pivot, **partition** so smaller elements go left and larger go right (the pivot lands in its final position), then recurse on both sides.

### 5.1 Lomuto partition (pivot = last element)

```c
int partition(int a[], int lo, int hi) {
    int p = a[hi], i = lo - 1;
    for (int j = lo; j < hi; j++)
        if (a[j] <= p) { i++; swap(&a[i], &a[j]); }
    swap(&a[i+1], &a[hi]);
    return i + 1;
}
```

**Worked example 6.** `[2,8,7,1,3,5,6,4]`, pivot $p=4$, $i=-1$.

| j | a[j] | $\le4$? | i | Array |
|---|---|---|---|---|
| 0 | 2 | yes | 0 | 2 8 7 1 3 5 6 4 |
| 1 | 8 | no | 0 | same |
| 2 | 7 | no | 0 | same |
| 3 | 1 | yes | 1 | 2 1 7 8 3 5 6 4 |
| 4 | 3 | yes | 2 | 2 1 3 8 7 5 6 4 |
| 5 | 5 | no | 2 | same |
| 6 | 6 | no | 2 | same |

Final swap $a[3]\leftrightarrow a[7]$: `[2,1,3,4,7,5,6,8]`, pivot index 3. Always $n-1$ comparisons per partition of $n$ elements.

### 5.2 Hoare partition (pivot = first element)

```c
int hoare(int a[], int lo, int hi) {
    int p = a[lo], i = lo - 1, j = hi + 1;
    while (1) {
        do i++; while (a[i] < p);
        do j--; while (a[j] > p);
        if (i >= j) return j;
        swap(&a[i], &a[j]);
    }
}   // recurse on (lo, j) and (j+1, hi)
```

**Worked example 7.** `[5,3,8,4,2,7,1,10]`, $p=5$.

| Step | i stops at | j stops at | Action | Array |
|---|---|---|---|---|
| 1 | 0 (5) | 6 (1) | swap | 1 3 8 4 2 7 5 10 |
| 2 | 2 (8) | 4 (2) | swap | 1 3 2 4 8 7 5 10 |
| 3 | 4 (8) | 3 (4) | $i\ge j$: return 3 | 1 3 2 4 / 8 7 5 10 |

The pivot value is **not** necessarily at index $j$; the array is split into $[0..3]$ and $[4..7]$ with everything left $\le5\le$ everything right. Hoare does about 3x fewer swaps than Lomuto and handles duplicates better (Lomuto degrades to $O(n^2)$ on all-equal keys).

### 5.3 Analysis

| Case | Recurrence | Result |
|---|---|---|
| Best/balanced | $T=2T(n/2)+n$ | $\Theta(n\log n)$ |
| Worst (pivot is min or max) | $T=T(n-1)+n$ | $\Theta(n^2)$, $\frac{n(n-1)}2$ comparisons |
| Constant split $9{:}1$ | $T=T(9n/10)+T(n/10)+n$ | $\Theta(n\log n)$ |
| Average | $\approx1.39\,n\log_2n$ comparisons | $\Theta(n\log n)$ |

**Worst case occurs on already-sorted (or reverse-sorted) input** with first/last/Lomuto pivot; for $n=10$ sorted, Lomuto does $9+8+\dots+1=45$ comparisons (confirmed by running). Using the **median** as pivot guarantees $\Theta(n\log n)$.

**Randomised quicksort** picks a uniformly random pivot, so no input is bad: **expected** time $O(n\log n)$ for every input (the worst case still exists but has probability $\to0$). Recursion depth: $\Theta(\log n)$ expected, $\Theta(n)$ worst; recursing on the smaller side first bounds stack at $O(\log n)$. Not stable; in place.

## 6. Heap sort

Uses a **max-heap** in an array (children of $i$ at $2i+1,2i+2$, 0-indexed). See [Heaps](../07-data-structures/heaps.md).

1. **Build-heap:** sift down from index $\lfloor n/2\rfloor-1$ to 0. Cost $O(n)$ (not $n\log n$), since $\sum_h\frac n{2^{h+1}}\cdot h=O(n)$.
2. Repeat $n-1$ times: swap root with the last heap element, shrink the heap, sift down the root. $O(\log n)$ each.

**Worked example 8.** Build a max-heap from `[4,10,3,5,1,2,8]`. Start at index 2 (value 3): children 2,8 → swap with 8: `[4,10,8,5,1,2,3]`. Index 1 (10): children 5,1 → fine. Index 0 (4): children 10,8 → swap with 10 → `[10,4,8,5,1,2,3]`; continue sifting 4 at index 1: children 5,1 → swap with 5 → `[10,5,8,4,1,2,3]`. Done (verified by code).

Total $\Theta(n\log n)$ in every case, space $O(1)$, not stable, not adaptive.

## 7. Non-comparison sorts

### 7.1 Counting sort (keys are integers in $0..k$)

1. Count occurrences $C[v]$. 2. Prefix sums so $C[v]$ = number of keys $\le v$. 3. Scan input **right to left**, place $x$ at position $C[x]-1$ and decrement (this makes it stable).

**Worked example 9.** `A=[2,5,3,0,2,3,0,3]`, $k=5$.
- Counts $C=[2,0,2,3,0,1]$; prefix: $C=[2,2,4,7,7,8]$.
- Right-to-left placement yields `B=[0,0,2,2,3,3,3,5]`.

Time $\Theta(n+k)$, space $\Theta(n+k)$, **stable**, not in place. Good when $k=O(n)$.

### 7.2 Radix sort (LSD)

Sort by the least significant digit first, then the next, using a **stable** sort (counting) per digit. For $d$ digits in base $b$: $\Theta(d(n+b))$.

**Worked example 10.** `[170,45,75,90,802,24,2,66]`:
- by units: 170 90 802 2 24 45 75 66
- by tens: 802 2 24 45 66 170 75 90
- by hundreds: 2 24 45 66 75 90 170 802. ✓

Sorting $n$ numbers in range $0..n^c-1$: choose base $n$, $d=c$ digits, $\Theta(cn)=\Theta(n)$. **Bucket sort:** uniform keys in $[0,1)$ distributed to $n$ buckets, expected $\Theta(n)$.

## 8. Lower bound for comparison sorting

A comparison sort on $n$ distinct items is a **binary decision tree**: each internal node is a comparison, each leaf an output permutation. It needs $\ge n!$ leaves. A binary tree of height $h$ has $\le2^h$ leaves, so
$$2^h\ge n!\ \Rightarrow\ h\ge\log_2(n!)=\Theta(n\log n).$$

**Worked example 11.** Minimum comparisons to sort 5 elements in the worst case: $\lceil\log_25!\rceil=\lceil\log_2120\rceil=\lceil6.9\rceil=7$. For 4 elements: $\lceil\log_224\rceil=5$. For 10 elements: $\lceil21.79\rceil=22$. These are lower bounds (achievable for 4 and 5: Ford–Johnson uses 5 and 7).

Average-case lower bound is also $\Omega(n\log n)$.

## 9. Selection problems

### 9.1 k-th smallest: quickselect

Partition around a pivot; if the pivot's rank is $k$ done, else recurse into **one** side only. Expected time $T(n)=T(n/2)+O(n)=O(n)$; worst $O(n^2)$. **Median of medians:** split into groups of 5, take the median of the group medians as pivot; guarantees a $30/70$ split, $T(n)=T(n/5)+T(7n/10)+O(n)=O(n)$ worst case. (Groups of 3 would give $T(n/3)+T(2n/3)$, which is $n\log n$, so group size 5 matters.)

### 9.2 Minimum and maximum together

Compare elements in pairs: the smaller of each pair goes to the min-candidates, the larger to max-candidates. Comparisons: $1$ per pair $+\,2$ per pair $=3$ per pair, minus saving for the first pair:
$$\left\lceil\frac{3n}2\right\rceil-2.$$
For $n=8$: $10$ (verified by code); for $n=7$: $9$; versus $2n-2=14$ for the naive method.

### 9.3 Second largest

Knockout tournament: $n-1$ comparisons to find the max; the second largest must have lost directly to the max, so it is among the $\lceil\log_2n\rceil$ players the winner beat; $\lceil\log_2 n\rceil-1$ more comparisons. Total
$$n+\lceil\log_2n\rceil-2.$$
For $n=8$: $9$.

## 10. Master comparison table

| Algorithm | Best | Average | Worst | Space | Stable | In place | Adaptive | Swaps/writes |
|---|---|---|---|---|---|---|---|---|
| Selection | $n^2$ | $n^2$ | $n^2$ | $O(1)$ | No | Yes | No | $\le n-1$ swaps |
| Bubble (flag) | $n$ | $n^2$ | $n^2$ | $O(1)$ | Yes | Yes | Yes | = inversions |
| Insertion | $n$ | $n^2$ | $n^2$ | $O(1)$ | Yes | Yes | Yes | = inversions (shifts) |
| Merge | $n\log n$ | $n\log n$ | $n\log n$ | $O(n)$ | Yes | No | No | $n\log n$ writes |
| Quick | $n\log n$ | $n\log n$ | $n^2$ | $O(\log n)$ avg / $O(n)$ worst | No | Yes | No | $O(n\log n)$ swaps |
| Heap | $n\log n$ | $n\log n$ | $n\log n$ | $O(1)$ | No | Yes | No | $n\log n$ |
| Counting | $n+k$ | $n+k$ | $n+k$ | $O(n+k)$ | Yes | No | – | – |
| Radix | $d(n+k)$ | $d(n+k)$ | $d(n+k)$ | $O(n+k)$ | Yes | No | – | – |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Max inversions | $n(n-1)/2$ (reversed) | bubble/insertion worst case |
| Binary search worst | $\lfloor\log_2n\rfloor+1$ iterations | counting comparisons |
| Merge two lists | worst $m+n-1$, best $\min(m,n)$ | merge-sort counts |
| Merge sort worst | $n\log_2n-n+1$ ($n=2^k$) | exact comparison counts |
| Quicksort worst | $n(n-1)/2$ comparisons | sorted input |
| Decision-tree bound | $\lceil\log_2n!\rceil$ | min comparisons to sort |
| Min and max | $\lceil3n/2\rceil-2$ | selection problems |
| Second largest | $n+\lceil\log_2n\rceil-2$ | tournament |
| Build-heap | $\Theta(n)$ | heap sort analysis |
| Counting sort | $\Theta(n+k)$, stable | integer keys |

## GATE traps

- **Insertion/bubble swaps = inversions.** For `[3,2,1]` it is 3 inversions, so bubble does 3 swaps, insertion 3 shifts.
- **Bubble sort best case is $O(n)$ only with the early-exit flag.** Plain nested loops are always $\Theta(n^2)$ comparisons.
- **Selection sort is not stable** (long-range swaps); insertion, bubble, merge (careful merge), counting, radix are stable. Quick and heap are not.
- **Quicksort on sorted input is the worst case** with first/last pivot; on random input it is fine. With a median-of-three pivot sorted input becomes good.
- **Merge sort's best case is not $O(n)$:** it still splits and merges, $\Theta(n\log n)$ (only the merge comparisons drop to $\frac n2\log n$).
- **Binary search comparisons:** distinguish "iterations" ($\lfloor\log_2n\rfloor+1$) from "three-way comparisons". A question saying "number of comparisons of the form $a[mid]$ vs $x$" counts $2$ per iteration if `==` and `<` are tested separately.
- **Build-heap is $O(n)$**, but $n$ insertions one by one is $O(n\log n)$.
- **Counting sort's $O(n+k)$ is not $O(n)$ if $k\gg n$** (e.g., 32-bit keys).
- **Decision tree lower bound** applies to *comparison* sorts only; it does not forbid radix sort.
- **Randomised quicksort worst case is still $O(n^2)$;** only the *expected* time is $O(n\log n)$ for every input.

## Connections

- [Asymptotic analysis](asymptotic-analysis.md) — recurrences for merge sort and quicksort come from the Master theorem.
- [Heaps](../07-data-structures/heaps.md) — heap sort is a direct application of the array heap; also priority queues.
- [Divide and conquer](divide-and-conquer.md) — merge sort, quicksort, binary search, counting inversions via merge.
- [Hashing](hashing.md) — an alternative to sorted search: expected $O(1)$ lookup.
- [Trees and BST](../07-data-structures/trees-and-bst.md) — inorder traversal sorts a BST; building a BST of $n$ keys mirrors quicksort's recursion.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — $n!$ leaves, inversions and permutation counting.
- [Database file organization and indexing](../14-databases/file-organization-and-indexing.md) — external merge sort for large relations.
- [Probability basics](../02-probability-statistics/probability-basics.md) — expected inversions and randomised pivot analysis use expectation of indicator variables.

## Practice

**Q1 (NAT).** How many comparisons does binary search make (counting one per loop iteration) in the worst case on a sorted array of 1000 elements?

<details><summary>Answer</summary>

**Answer:** 10  
**Solution:** $\lfloor\log_2 1000\rfloor+1=9+1=10$.

</details>

**Q2 (NAT).** The array `[3,1,4,2]` is sorted by bubble sort. How many swaps are performed?

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** Inversions: (3,1),(3,2),(4,2) $=3$. Pass 1: (3,1)s→1 3 4 2; (3,4); (4,2)s→1 3 2 4. Pass 2: (1,3); (3,2)s→1 2 3 4. Total 3 swaps.

</details>

**Q3 (MCQ).** Which sort has $\Theta(n)$ best case, is stable and in place? (A) Selection (B) Insertion (C) Merge (D) Quick

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Insertion sort on sorted input does $n-1$ comparisons and no shifts. Selection is always $n^2$; merge needs $O(n)$ space; quick is unstable.

</details>

**Q4 (NAT).** Number of comparisons in the worst case when merging two sorted lists of 8 and 12 elements?

<details><summary>Answer</summary>

**Answer:** 19  
**Solution:** $m+n-1=8+12-1=19$.

</details>

**Q5 (MCQ).** Quicksort with the last element as pivot is run on `[1,2,3,4,5,6]`. Number of element comparisons?

<details><summary>Answer</summary>

**Answer:** 15  
**Solution:** Pivot 6 splits into 5 and 0 elements: $5+4+3+2+1=15=\frac{6\cdot5}2$.

</details>

**Q6 (NAT).** Minimum number of comparisons in the worst case needed to find both the maximum and minimum of 100 distinct numbers.

<details><summary>Answer</summary>

**Answer:** 148  
**Solution:** $\lceil3\cdot100/2\rceil-2=150-2=148$.

</details>

**Q7 (MSQ).** Which are correct? (A) Any comparison-based sort needs $\Omega(n\log n)$ comparisons in the worst case. (B) Heap sort is stable. (C) Counting sort can sort 1,000,000 integers in $[0,999]$ in $O(n)$ time. (D) Randomised quicksort has worst-case time $O(n\log n)$.

<details><summary>Answer</summary>

**Answer:** A, C  
**Solution:** (A) decision-tree argument. (B) false. (C) $k=1000=O(n)$, so $\Theta(n+k)=\Theta(n)$. (D) false, only expected.

</details>

**Q8 (NAT).** Using the knockout-tournament method, how many comparisons are needed in the worst case to find the largest and second largest among 64 numbers?

<details><summary>Answer</summary>

**Answer:** 68  
**Solution:** $n+\lceil\log_2n\rceil-2=64+6-2=68$: 63 to find the max, then 5 among the 6 players the champion beat.

</details>
