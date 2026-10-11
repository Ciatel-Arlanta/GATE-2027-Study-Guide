# Divide and Conquer

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** divide-and-conquer design, recurrences, binary search, merge sort, quicksort
> **Prerequisites:** [Asymptotic analysis](asymptotic-analysis.md), [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) · **Leads to:** [Searching and sorting](searching-and-sorting.md), [Greedy algorithms](greedy-algorithms.md), [Dynamic programming](dynamic-programming.md)
> **PYQ weightage:** High · 39 Qs since 2015 (17 since 2021) · DA: 6 · Trend: steady · Most asked: Recurrence relations, Quicksort partitioning, Binary search variants, Merge sort & inversions

## Quick glance

- **Exam focus:** Recurrence relations (Master Theorem cases, change of variables, substitution), Quicksort partitioning passes, pivot comparisons and recursion depth, Binary search comparison bounds on normal and modified arrays (bitonic, rotated), and Merge sort merge-comparisons and inversion counting. Question types: NAT (comparison counts, recurrence values) and MCQ/MSQ (asymptotic bounds, correctness, algorithmic invariants).
- **Core tri-step:** **Divide** instance into non-overlapping subproblems; **Conquer** subproblems recursively (base case when trivial); **Combine** subproblem solutions into the overall solution.
- **Master Theorem** for $T(n) = aT(n/b) + \Theta(n^k \log^p n)$ ($a \ge 1, b > 1$): Compare $a$ with $b^k$:
  - $a > b^k \implies T(n) = \Theta(n^{\log_b a})$.
  - $a = b^k \implies T(n) = \Theta(n^k \log^{p+1} n)$ (for $p \ge 0$).
  - $a < b^k \implies T(n) = \Theta(n^k \log^p n)$ (for $p \ge 0$ under regularity condition).
- **Binary Search:** Operates on sorted arrays with random access. Height of decision tree is $\lfloor \log_2 n \rfloor$. Max comparisons in successful search: $\lfloor \log_2 n \rfloor + 1$. Unsuccessful search: $\lfloor \log_2 n \rfloor + 1$ (or $\lceil \log_2(n+1) \rceil$).
- **Merge Sort:** $T(n) = 2T(n/2) + \Theta(n) \implies \Theta(n \log n)$ in best, worst, and average cases. Requires $\Theta(n)$ auxiliary space. Stable.
- **2-Way Merge comparisons:** Merging two sorted arrays of lengths $m$ and $n$ requires at most $m + n - 1$ comparisons and at least $\min(m, n)$ comparisons.
- **Inversion Counting:** A pair $(i, j)$ with $i < j$ and $A[i] > A[j]$. Maximum inversions is $n(n-1)/2$ (reverse sorted). Counted in $\Theta(n \log n)$ time by piggybacking on merge sort.
- **Quicksort:** Partition around a pivot. Best/average time $\Theta(n \log n)$, worst case $\Theta(n^2)$ (maximally unbalanced splits). In-place (no array allocation), but auxiliary stack space is $O(\log n)$ average and $O(n)$ worst. Unstable.
- **Simultaneous Min and Max:** An array of $n$ elements requires $\lceil 3n/2 \rceil - 2$ comparisons by comparing elements in pairs first.

## 1. Divide-and-conquer design and recurrence solving [PYQ: High]

Divide-and-conquer decomposes an intractable problem of size $n$ into $a$ smaller, independent subproblems of size $n/b$. Unlike dynamic programming, subproblems here do not overlap.

### Master Theorem (Extended Form)
For recurrences of the form:
$$T(n) = a T(n/b) + f(n), \quad a \ge 1, b > 1$$
where $f(n) = \Theta(n^k \log^p n)$ with $k \ge 0$:

| Condition on $a$ vs $b^k$ | Condition on $p$ | Asymptotic Solution $T(n)$ |
|---|---|---|
| $a > b^k$ ($\log_b a > k$) | Any $p \in \mathbb{R}$ | $\Theta(n^{\log_b a})$ |
| $a = b^k$ ($\log_b a = k$) | $p > -1$ | $\Theta(n^{\log_b a} \log^{p+1} n) = \Theta(n^k \log^{p+1} n)$ |
| $a = b^k$ ($\log_b a = k$) | $p = -1$ | $\Theta(n^{\log_b a} \log \log n)$ |
| $a = b^k$ ($\log_b a = k$) | $p < -1$ | $\Theta(n^{\log_b a})$ |
| $a < b^k$ ($\log_b a < k$) | $p \ge 0$ | $\Theta(n^k \log^p n)$ |
| $a < b^k$ ($\log_b a < k$) | $p < 0$ | $O(n^k)$ |

> **When Master Theorem Fails:**
> 1. $a$ is not constant (e.g., $T(n) = n T(n/2) + n$).
> 2. $b \le 1$ or subtraction recurrence (e.g., $T(n) = T(n-1) + n \implies \Theta(n^2)$).
> 3. $f(n)$ is not polynomially bounded (e.g., $T(n) = 2T(n/2) + 2^n \implies \Theta(2^n)$ via recursion tree).
> 4. Non-polynomial gap: $T(n) = 2T(n/2) + n/\log n$ has $a = b^1$, but $p = -1$ does not yield $\Theta(n)$ because $f(n)$ is asymptotically smaller than $n^{\log_b a}$ by a logarithmic factor, not a polynomial factor $n^\epsilon$. Here, $T(n) = \Theta(n \log \log n)$.

### Substitution via Change of Variables
When subproblem sizes involve roots or powers, substitute variables to transform the recurrence into standard linear form.

**Worked example 1 (Recurrence with roots):** Solve $T(n) = 2T(\lfloor\sqrt{n}\rfloor) + \log_2 n$ with $T(2) = 1$.
1. Let $n = 2^m \implies m = \log_2 n$.
2. $\sqrt{n} = n^{1/2} = 2^{m/2}$.
3. Rewrite: $T(2^m) = 2 T(2^{m/2}) + m$.
4. Rename $S(m) = T(2^m)$:
   $$S(m) = 2 S(m/2) + m$$
5. Apply Master Theorem to $S(m)$: $a = 2, b = 2, f(m) = m^1 \implies a = b^1$.
   Hence, $S(m) = \Theta(m \log_2 m)$.
6. Substitute back $m = \log_2 n$:
   $$T(n) = \Theta(\log_2 n \cdot \log_2(\log_2 n))$$

## 2. Binary search and logarithmic reduction [PYQ: Medium]

Binary search eliminates half the remaining elements at each step by probing the midpoint of a sorted, indexable array.

```text
Array: [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]   (n = 10)
Search for 23:
Step 1: low=0, high=9, mid=4 -> A[4]=16 < 23 -> low=5
Step 2: low=5, high=9, mid=7 -> A[7]=56 > 23 -> high=6
Step 3: low=5, high=6, mid=5 -> A[5]=23 == 23 -> Found in 3 comparisons!
```

### Exact Comparison Counts
- **Decision Tree:** A binary tree of height $h = \lfloor \log_2 n \rfloor$ with internal nodes representing array elements ($n$) and leaves representing failure intervals ($n+1$).
- **Maximum comparisons (Worst Case):**
  - **Successful search:** $\lfloor \log_2 n \rfloor + 1$.
  - **Unsuccessful search:** $\lfloor \log_2 n \rfloor + 1$ (or $\lceil \log_2(n+1) \rceil$).
- **Average comparisons:** $\approx \log_2 n - 1$.

### Binary Search Variations
1. **Search in Rotated Sorted Array:** (e.g., `[15, 18, 2, 3, 6, 12]`)
   At least one half (`[low..mid]` or `[mid..high]`) is always normally sorted. Check if the target lies within the sorted half; if yes, search that half; else, search the other half. Time: $\Theta(\log n)$.
2. **Search in Bitonic Array:** A bitonic sequence increases monotonically, then decreases monotonically (e.g., `[1, 3, 8, 12, 4, 2]`).
   - Find the peak element (maximum) in $\Theta(\log n)$ comparisons: check whether $A[mid] > A[mid+1]$ and $A[mid] > A[mid-1]$.
   - Binary search ascending on left part: $\Theta(\log n)$.
   - Binary search descending on right part: $\Theta(\log n)$.
   - Total time: $3 \times \Theta(\log n) = \Theta(\log n)$.
3. **Ternary Search:** Splits into 3 parts using two midpoints $m_1, m_2$.
   Recurrence: $T(n) = T(n/3) + 2$ comparisons.
   Total comparisons: $2 \log_3 n = \frac{2}{\log_2 3} \log_2 n \approx 1.26 \log_2 n$.
   > **GATE Fact:** Ternary search does *more* comparisons than binary search in the worst case ($1.26 \log_2 n > \log_2 n$), despite having fewer recursive levels!

**Worked example 2 (Bitonic Array Peak):**
Find peak in $A = [3, 9, 18, 20, 17, 5, 1]$ ($n = 7$).
1. $low = 0, high = 6 \implies mid = 3$. $A[3] = 20$.
2. Check neighbors: $A[2] = 18 < 20$ and $A[4] = 17 < 20$.
3. Both neighbors are strictly smaller $\implies$ Peak is $A[3] = 20$. Found in 2 comparisons!

## 3. Merge sort, 2-way merging, and inversion counting [PYQ: Medium]

Merge sort is the canonical divide-and-conquer sorting algorithm. It divides the array into two halves, recursively sorts each half, and merges the sorted sublists.

```text
MergeSort(A, p, r):
    if p < r:
        q = floor((p + r) / 2)
        MergeSort(A, p, q)
        MergeSort(A, q + 1, r)
        Merge(A, p, q, r)
```

Recurrence: $T(n) = 2T(n/2) + \Theta(n) \implies \Theta(n \log n)$ in best, worst, and average cases.

### Comparison Bounds for 2-Way Merge
When merging two sorted arrays $X$ of size $m$ and $Y$ of size $n$:
- **Maximum comparisons:** $m + n - 1$. Occurs when elements alternate strictly between $X$ and $Y$ (e.g., $X = [1, 3, 5]$, $Y = [2, 4, 6] \implies 3 + 3 - 1 = 5$ comparisons).
- **Minimum comparisons:** $\min(m, n)$. Occurs when all elements of one array are strictly smaller than all elements of the other (e.g., $X = [1, 2, 3]$, $Y = [10, 20, 30] \implies 3$ comparisons, after which the remaining array is directly copied).

### Inversion Counting via Merge Sort
An **inversion** in an array $A$ is a pair $(i, j)$ such that $i < j$ and $A[i] > A[j]$.
- Sorted array: 0 inversions.
- Reverse sorted array: $\frac{n(n-1)}{2}$ inversions.
- While merging left subarray $L[p..q]$ and right subarray $R[q+1..r]$:
  If $R[j] < L[i]$, then $R[j]$ is smaller than $L[i]$ and *all remaining elements* in $L$ from index $i$ to $q$.
  Therefore, cross-inversions contributed by $R[j]$ is exactly:
  $$\text{inversions} += (q - i + 1)$$

**Worked example 3 (Full Merge Sort & Inversion Trace):**
Count inversions in $A = [8, 4, 2, 1]$ ($n = 4$).

```text
Split: [8, 4] and [2, 1]
Left half: [8] and [4]
  Merge [8] and [4]: 4 < 8 -> take 4, cross-inversions = (mid - i + 1) = (0 - 0 + 1) = 1.
  Left sorted: [4, 8]. Sub-inversions = 1.

Right half: [2] and [1]
  Merge [2] and [1]: 1 < 2 -> take 1, cross-inversions = 1.
  Right sorted: [1, 2]. Sub-inversions = 1.

Merge [4, 8] and [1, 2]:
  i=0 (L[0]=4), j=0 (R[0]=1):
    R[0]=1 < L[0]=4 -> take 1. Remaining in L: [4, 8] (count = 2 - 0 = 2). Inversions += 2.
  i=0 (L[0]=4), j=1 (R[1]=2):
    R[1]=2 < L[0]=4 -> take 2. Remaining in L: [4, 8] (count = 2 - 0 = 2). Inversions += 2.
  Right array exhausted. Copy remaining [4, 8].

Total inversions = 1 (left) + 1 (right) + 4 (cross) = 6 inversions.
Matches theoretical max: 4 * 3 / 2 = 6.
```

## 4. Quicksort and partitioning algorithms [PYQ: High]

Quicksort selects a **pivot** element and partitions the array such that all elements $\le \text{pivot}$ lie to its left and all elements $\ge \text{pivot}$ lie to its right. It then recurses on the two sides.

### Partition Schemes

1. **Lomuto Partition:**
   Picks last element as pivot. Maintains a single boundary pointer $i$:
   ```c
   int lomuto_partition(int A[], int low, int high) {
       int pivot = A[high], i = low - 1;
       for (int j = low; j < high; j++) {
           if (A[j] <= pivot) {
               i++;
               swap(&A[i], &A[j]);
           }
       }
       swap(&A[i + 1], &A[high]);
       return i + 1;
   }
   ```
   - Comparisons: Exactly $n - 1$ per partition pass.
   - Degrades to $\Theta(n^2)$ when all elements are equal!

2. **Hoare Partition:**
   Picks first element as pivot. Moves two pointers from ends towards each other:
   ```c
   int hoare_partition(int A[], int low, int high) {
       int pivot = A[low], i = low - 1, j = high + 1;
       while (1) {
           do { i++; } while (A[i] < pivot);
           do { j--; } while (A[j] > pivot);
           if (i >= j) return j;
           swap(&A[i], &A[j]);
       }
   }
   ```
   - On average does 3x fewer swaps than Lomuto.
   - Handles duplicate elements gracefully without degrading split balance.

### Complexity Analysis
- **Best Case:** Partitions divide input into two equal halves of size $\approx n/2$:
  $$T(n) = 2T(n/2) + \Theta(n) \implies \Theta(n \log n)$$
- **Worst Case:** One partition has size 0 and the other $n - 1$ (e.g., sorted array with first or last element as pivot):
  $$T(n) = T(n-1) + T(0) + \Theta(n) = T(n-1) + \Theta(n) \implies \Theta(n^2)$$
  Total comparisons in worst case:
  $$\sum_{i=1}^{n-1} i = \frac{n(n-1)}{2}$$
- **Average Case (Randomized Pivot):**
  Expected running time is $\Theta(n \log n)$. Every pair of elements is compared at most once across the entire algorithm execution.
- **Space Complexity (Recursion Call Stack):**
  - Worst case: $O(n)$ stack frames.
  - Tail-call optimization (recurse on smaller partition first, loop on larger): Guarantees at most $O(\log n)$ auxiliary stack depth even in worst case!

**Worked example 4 (Quicksort Partition Pass):**
Run Lomuto partition on $A = [4, 7, 1, 9, 3, 5]$ with pivot = last element ($5$).
- $low = 0, high = 5, pivot = 5, i = -1$.
- $j=0: A[0]=4 \le 5 \implies i=0$, swap $A[0], A[0]$. Array: `[4, 7, 1, 9, 3, 5]`
- $j=1: A[1]=7 > 5 \implies$ no swap.
- $j=2: A[2]=1 \le 5 \implies i=1$, swap $A[1], A[2]$. Array: `[4, 1, 7, 9, 3, 5]`
- $j=3: A[3]=9 > 5 \implies$ no swap.
- $j=4: A[4]=3 \le 5 \implies i=2$, swap $A[2], A[4]$. Array: `[4, 1, 3, 9, 7, 5]`
- Loop ends. Swap $A[i+1]$ with $A[high]$ (swap $A[3]=9$ and $A[5]=5$).
- Resulting array: `[4, 1, 3, 5, 7, 9]`.
- Pivot index returned: $3$. Left subarray `[4, 1, 3]` ($\le 5$), right subarray `[7, 9]` ($> 5$).

## 5. Selection problem and simultaneous extrema [PYQ: Low]

### Simultaneous Minimum and Maximum
Finding min and max independently takes $(n - 1) + (n - 1) = 2n - 2$ comparisons.
Using divide-and-conquer tournament (pairing elements):
1. Pair adjacent elements and compare: $n/2$ comparisons.
2. The larger elements compete for maximum: $n/2 - 1$ comparisons.
3. The smaller elements compete for minimum: $n/2 - 1$ comparisons.
4. Total comparisons for even $n$:
   $$\frac{n}{2} + 2\left(\frac{n}{2} - 1\right) = \frac{3n}{2} - 2$$
   For odd $n$: $3\lfloor n/2 \rfloor$ comparisons.
   General formula: $\lceil \frac{3n}{2} \rceil - 2$.

**Worked example 5:** Find min and max of $A = [14, 3, 9, 25, 7, 18]$ ($n = 6$).
1. Pairwise comparisons:
   - $(14, 3) \implies$ max-candidate 14, min-candidate 3.
   - $(9, 25) \implies$ max-candidate 25, min-candidate 9.
   - $(7, 18) \implies$ max-candidate 18, min-candidate 7.
   (3 comparisons so far).
2. Find max among $\{14, 25, 18\}$: compare 14 and 25 (25), compare 25 and 18 $\implies$ max = 25 (2 comparisons).
3. Find min among $\{3, 9, 7\}$: compare 3 and 9 (3), compare 3 and 7 $\implies$ min = 3 (2 comparisons).
4. Total comparisons = $3 + 2 + 2 = 7$.
   Formula check: $3(6)/2 - 2 = 9 - 2 = 7$ comparisons!

### $k$-th Order Statistic (Quickselect)
Find the $k$-th smallest element in an unsorted array:
- Partition the array around a pivot. Let pivot rank be $r$.
- If $k == r$: return pivot.
- If $k < r$: recurse only on left subarray.
- If $k > r$: recurse on right subarray for $(k - r)$-th element.
- **Average time:** $T(n) = T(n/2) + \Theta(n) \implies \Theta(n)$.
- **Worst-case time:** $\Theta(n^2)$.
- **Deterministic $O(n)$ (Median of Medians):** Divides array into groups of 5, finds median of each group, and uses the median of medians as pivot.
  Recurrence: $T(n) \le T(\lceil n/5 \rceil) + T(7n/10 + 6) + O(n) \implies O(n)$.

## Formulas and facts to memorise

| Algorithm / Concept | Time Complexity (Best / Avg / Worst) | Auxiliary Space | Key Formula / Comparison Count |
|---|---|---|---|
| **Binary Search** | $\Theta(1) \ / \ \Theta(\log n) \ / \ \Theta(\log n)$ | $O(1)$ iterative, $O(\log n)$ recursive | Max comparisons: $\lfloor \log_2 n \rfloor + 1$ |
| **Merge Sort** | $\Theta(n \log n) \ / \ \Theta(n \log n) \ / \ \Theta(n \log n)$ | $\Theta(n)$ | Recurrence: $2T(n/2) + \Theta(n)$; Stable |
| **2-Way Merge ($m, n$)** | $\Theta(\min(m, n)) \ / \ \Theta(m + n) \ / \ \Theta(m + n)$ | $O(m + n)$ | Min comparisons: $\min(m, n)$; Max: $m + n - 1$ |
| **Quicksort** | $\Theta(n \log n) \ / \ \Theta(n \log n) \ / \ \Theta(n^2)$ | $O(\log n)$ avg, $O(n)$ worst | Max comparisons: $n(n-1)/2$; Unstable |
| **Inversion Counting** | $\Theta(n \log n)$ | $\Theta(n)$ | Max inversions: $n(n-1)/2$ |
| **Simultaneous Min-Max** | $\Theta(n)$ | $O(1)$ | Exact comparisons: $\lceil 3n/2 \rceil - 2$ |
| **Quickselect** | $\Theta(n) \ / \ \Theta(n) \ / \ \Theta(n^2)$ | $O(\log n)$ avg, $O(n)$ worst | Average time: $\Theta(n)$ |
| **Master Theorem ($a = b^k, p=0$)** | — | — | $T(n) = \Theta(n^k \log n)$ |

## GATE traps

- **Quicksort is NOT $O(n \log n)$ worst case:** A common mistake is answering $O(n \log n)$ for Quicksort when the question asks for worst-case time without stating "randomized". Unless randomized is explicitly specified, worst case is $\Theta(n^2)$.
- **Master theorem non-polynomial differences:** $T(n) = 2T(n/2) + n/\log n$ cannot be solved by standard Case 1/3 because $n/\log n$ differs from $n^1$ by a logarithmic factor, not $n^\epsilon$. Do not claim Master Theorem Case 1 applies.
- **Ternary search comparison count:** Ternary search does $\log_3 n$ recursive steps, but each step performs 2 comparisons. Total comparisons: $2 \log_3 n \approx 1.26 \log_2 n$, which is strictly *greater* than binary search's $\log_2 n$.
- **Merge sort auxiliary space:** Merge sort on an array cannot easily be made $O(1)$ auxiliary space without compromising time or stability. It requires $\Theta(n)$ auxiliary array space. For linked lists, however, merge sort requires $O(\log n)$ recursion stack space and $O(1)$ node-relinking space.
- **Quicksort stack depth:** In worst case, standard recursive Quicksort takes $O(n)$ stack space. Tail-call elimination on the larger partition reduces this guarantee to $O(\log n)$.
- **Inversion count addition:** When merging, if $R[j] < L[i]$, the inversions added are $(q - i + 1)$ (the number of remaining unmerged elements in the left subarray), not just $1$.

## Connections

- [Asymptotic analysis](asymptotic-analysis.md) — growth rates and Big-O, Big-$\Omega$, Big-$\Theta$ classifications used in all divide-and-conquer recurrences.
- [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) — mathematical foundations for solving characteristic roots and non-homogeneous recurrences.
- [Searching and sorting](searching-and-sorting.md) — comprehensive comparative analysis across all sorting paradigms (insertion, heap, quick, merge, counting).
- [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) — function call stacks used in recursive divide-and-conquer execution.
- [Dynamic programming](dynamic-programming.md) — used when subproblems overlap instead of branching independently.

## Practice

**Q1 (MCQ · Easy).** The running time of an algorithm is represented by the recurrence $T(n) = 4T(n/2) + n^2$. The asymptotic complexity of $T(n)$ is:
- (A) $\Theta(n^2)$
- (B) $\Theta(n^2 \log n)$
- (C) $\Theta(n^3)$
- (D) $\Theta(n \log n)$

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
Using the Master Theorem:
$a = 4, b = 2, f(n) = n^2 = \Theta(n^k \log^p n)$ where $k = 2, p = 0$.
Calculate $b^k = 2^2 = 4$.
Since $a = b^k = 4$ and $p = 0 > -1$, Case 2 of the Master Theorem applies:
$$T(n) = \Theta(n^{\log_b a} \log^{p+1} n) = \Theta(n^{\log_2 4} \log^1 n) = \Theta(n^2 \log n)$$

</details>

**Q2 (NAT · Easy).** What is the maximum number of key comparisons needed to merge two sorted arrays of sizes 12 and 15 into a single sorted array?

<details><summary>Answer</summary>

**Answer:** 26

**Solution:**
To merge two sorted arrays of lengths $m$ and $n$, the maximum number of comparisons occurs when elements strictly alternate until one element remains:
$$\text{Max comparisons} = m + n - 1$$
Given $m = 12$ and $n = 15$:
$$\text{Max comparisons} = 12 + 15 - 1 = 26$$

</details>

**Q3 (MCQ · Medium).** Consider the array `A = [26, 19, 37, 12, 54, 8, 41]`. If the first element (26) is chosen as the pivot and a single partition step of Quicksort is executed using Hoare's original partitioning algorithm, what is the resulting array permutation before recursing?
- (A) `[8, 19, 12, 26, 54, 37, 41]`
- (B) `[8, 19, 12, 37, 54, 26, 41]`
- (C) `[12, 19, 8, 26, 54, 37, 41]`
- (D) `[8, 19, 26, 12, 54, 37, 41]`

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
Hoare's partitioning algorithm with pivot = 26:
- Initial pointers: $i = -1, j = 7$.
- Pass 1:
  - Increment $i$ until $A[i] \ge 26 \implies i = 0$ ($A[0] = 26$).
  - Decrement $j$ until $A[j] \le 26 \implies j = 5$ ($A[5] = 8$).
  - Since $i < j$, swap $A[0]$ and $A[5]$:
    Array becomes `[8, 19, 37, 12, 54, 26, 41]`.
- Pass 2:
  - Increment $i$ until $A[i] \ge 26 \implies i = 2$ ($A[2] = 37$).
  - Decrement $j$ until $A[j] \le 26 \implies j = 3$ ($A[3] = 12$).
  - Since $i < j$, swap $A[2]$ and $A[3]$:
    Array becomes `[8, 19, 12, 37, 54, 26, 41]`.
- Pass 3:
  - Increment $i$ until $A[i] \ge 26 \implies i = 3$ ($A[3] = 37$).
  - Decrement $j$ until $A[j] \le 26 \implies j = 2$ ($A[2] = 12$).
  - Now $i \ge j$, partition halts and returns index $j = 2$.
- The array left in memory after the pass is `[8, 19, 12, 37, 54, 26, 41]`, matching (B). Hoare's scheme does **not** place the pivot in its final sorted position: the pivot 26 stays at index 5, and the returned split index is $j = 2$.

</details>

**Q4 (NAT · Medium).** Find the total number of inversions in the array $A = [5, 3, 2, 4, 1]$.

<details><summary>Answer</summary>

**Answer:** 8

**Solution:**
An inversion is a pair $(i, j)$ with $i < j$ and $A[i] > A[j]$:
- For $A[0] = 5$: $(5, 3), (5, 2), (5, 4), (5, 1) \implies 4$ inversions.
- For $A[1] = 3$: $(3, 2), (3, 1) \implies 2$ inversions.
- For $A[2] = 2$: $(2, 1) \implies 1$ inversion.
- For $A[3] = 4$: $(4, 1) \implies 1$ inversion.
- For $A[4] = 1$: 0 inversions.

Total inversions = $4 + 2 + 1 + 1 = 8$.

</details>

**Q5 (NAT · Medium).** The minimum number of comparisons needed to find both the maximum and minimum elements in an array of 100 distinct numbers using the optimal tournament (pairwise) algorithm is ________.

<details><summary>Answer</summary>

**Answer:** 148

**Solution:**
For $n = 100$ (even):
$$\text{Comparisons} = \frac{3n}{2} - 2$$
Substitute $n = 100$:
$$\text{Comparisons} = \frac{3(100)}{2} - 2 = 150 - 2 = 148$$

Step breakdown:
1. Pairwise comparisons: $100 / 2 = 50$ comparisons to form 50 winners and 50 losers.
2. Find maximum among 50 winners: $50 - 1 = 49$ comparisons.
3. Find minimum among 50 losers: $50 - 1 = 49$ comparisons.
Total comparisons = $50 + 49 + 49 = 148$.

</details>

**Q6 (MSQ · Hard).** Which of the following statements is/are **CORRECT** regarding Quicksort and recursive partitioning?
- (A) The best-case time complexity of Quicksort is $\Theta(n \log n)$ when the partition always splits the array into ratios of $1:9$.
- (B) The auxiliary stack space of Quicksort can always be limited to $O(\log n)$ in the worst case by recursing on the smaller subarray first and converting the call on the larger subarray into a loop.
- (C) Randomized Quicksort completely eliminates the $\Theta(n^2)$ worst-case running time on all inputs.
- (D) Quicksort is an in-place sorting algorithm because it does not allocate an auxiliary array of size $\Theta(n)$.

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)

**Solution:**
- **(A) is Correct:** If splits are consistently $1:9$, the recurrence is $T(n) = T(n/10) + T(9n/10) + \Theta(n)$. The depth of the recursion tree is at most $\log_{10/9} n = O(\log n)$ and the work per level is at most $O(n)$, giving $\Theta(n \log n)$. Any constant-ratio split yields $\Theta(n \log n)$.
- **(B) is Correct:** Tail-call optimization on the larger subarray ensures that each pushed stack frame corresponds to a problem of size at most $n/2$. Thus, the maximum stack depth is bounded by $\lfloor \log_2 n \rfloor = O(\log n)$.
- **(C) is Incorrect:** Randomized Quicksort has an **expected** time of $\Theta(n \log n)$. However, an adversary or extraordinarily unlucky coin flips could still pick the extreme element every time, meaning the worst-case remains $\Theta(n^2)$.
- **(D) is Correct:** An algorithm is in-place if it requires $O(1)$ additional storage outside the recursive call stack (i.e., no auxiliary data array proportional to $n$).

</details>

**Q7 (NAT · Hard).** Consider the recurrence relation:
$$T(n) = T(\lfloor\sqrt{n}\rfloor) + 1, \quad T(2) = 1$$
The value of $T(2^{64})$ is ________.

<details><summary>Answer</summary>

**Answer:** 7

**Solution:**
Trace the argument $n$ down to the base case $n = 2$:
1. $n_0 = 2^{64} \implies T(2^{64}) = T(2^{32}) + 1$
2. $n_1 = 2^{32} \implies T(2^{32}) = T(2^{16}) + 1$
3. $n_2 = 2^{16} \implies T(2^{16}) = T(2^8) + 1$
4. $n_3 = 2^8 \implies T(2^8) = T(2^4) + 1$
5. $n_4 = 2^4 \implies T(2^4) = T(2^2) + 1$
6. $n_5 = 2^2 \implies T(2^2) = T(2^1) + 1$
7. Base case: $T(2^1) = T(2) = 1$.

Now unwind the steps:
- $T(2^2) = 1 + 1 = 2$
- $T(2^4) = 2 + 1 = 3$
- $T(2^8) = 3 + 1 = 4$
- $T(2^{16}) = 4 + 1 = 5$
- $T(2^{32}) = 5 + 1 = 6$
- $T(2^{64}) = 6 + 1 = 7$

General formula: $T(2^m) = \lfloor \log_2 m \rfloor + 1$. For $m = 64$, $\log_2 64 = 6 \implies 6 + 1 = 7$.

</details>
