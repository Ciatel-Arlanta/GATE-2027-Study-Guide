# Greedy Algorithms

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** greedy choice, activity selection, fractional knapsack, Huffman coding
> **Prerequisites:** [Asymptotic analysis](asymptotic-analysis.md), [Binary Heaps and Priority Queues](../07-data-structures/heaps.md) · **Leads to:** [Minimum spanning trees](minimum-spanning-trees.md), [Dynamic programming](dynamic-programming.md), [Shortest paths](shortest-paths.md)
> **PYQ weightage:** Low · 5 Qs since 2015 (2 since 2021) · DA: 0 · Trend: steady · Most asked: Huffman code trees, Activity selection, Fractional knapsack, Job sequencing

## Quick glance

- **Exam focus:** Huffman tree construction and bit length / weighted path length (NAT), Activity selection compatibility (MCQ/NAT), Fractional knapsack value maximization, and Job sequencing with deadlines to maximize profit. Question types: NAT (total bits in Huffman, maximum profit, max activities) and MCQ/MSQ (correctness of greedy strategies).
- **Greedy paradigm:** Constructs a solution piece by piece, always choosing the currently most favorable option (locally optimal) without ever reconsidering past choices.
- **Two conditions for optimality:**
  1. **Greedy-choice property:** A globally optimal solution can be reached by making a locally optimal choice. Proved via an **exchange argument**.
  2. **Optimal substructure:** An optimal solution to the global problem contains within it optimal solutions to subproblems.
- **Activity Selection:** Sort by finish time ascending. Greedily pick the next activity whose start time $\ge$ finish time of the last chosen activity. Time: $O(n \log n)$ due to sorting.
- **Fractional Knapsack:** Items can be divided. Sort items by value density $r_i = v_i / w_i$ in descending order. Take full items until capacity is nearly exhausted, then take a fractional portion of the next item. Time: $O(n \log n)$. Does **NOT** work for 0/1 Knapsack.
- **Huffman Coding:** Optimal variable-length prefix code. Built bottom-up using a min-heap by repeatedly merging the two smallest frequencies.
  - A Huffman tree for $n$ symbols is a **full binary tree** (every node has 0 or 2 children).
  - Total leaves = $n$, total internal nodes = $n - 1$, total nodes = $2n - 1$.
  - Cost of tree = $\sum f_i \cdot \text{length}(c_i) = \sum \text{all internal node values}$.
- **Job Sequencing with Deadlines:** $n$ jobs with unit execution time, profits, and deadlines. Sort by profit descending. Place each job into the latest available empty slot $t \le d_i$. Time: $O(n^2)$ naive, $O(n \alpha(n))$ with Disjoint Set Union (DSU).
- **Optimal Merge Patterns:** Merge $n$ sorted files of varying sizes in pairs with minimum total record moves. Greedily merge the two smallest files repeatedly (identical to Huffman coding).

## 1. Greedy choice property and exchange arguments [PYQ: Low]

A greedy algorithm makes a sequence of choices, each of which looks best at the moment, with the hope that the overall solution reaches global optimality.

```text
Dynamic Programming: Solves all subproblems, then decides -> Overlapping subproblems
Greedy Algorithm:    Makes the local best choice first, then solves the one remaining subproblem
```

### Proving Correctness: The Exchange Argument
To prove that a greedy choice is safe:
1. Assume an optimal solution $S^*$ exists that differs from the greedy solution $S_{greedy}$.
2. Identify the first choice where $S^*$ diverges from $S_{greedy}$.
3. Replace the non-greedy choice in $S^*$ with the greedy choice without decreasing the objective value (or violating feasibility).
4. Inductively transform $S^*$ into $S_{greedy}$, proving $S_{greedy}$ achieves the optimal value.

### When Greedy Heuristics Fail
A locally optimal choice often yields a suboptimal global solution if the problem lacks the greedy-choice property.

**Worked example 1 (Coin Change Failure):**
Given coin denominations $\{1, 3, 4\}$ and target value $V = 6$:
- **Greedy heuristic (take largest coin $\le$ remaining amount):**
  1. Pick coin $4$. Remaining amount: $6 - 4 = 2$.
  2. Pick coin $1$. Remaining amount: $2 - 1 = 1$.
  3. Pick coin $1$. Remaining amount: $1 - 1 = 0$.
  Total coins used = $3$ (coins: $4, 1, 1$).
- **Optimal solution:**
  Pick coin $3$, then coin $3$. Total coins used = $2$ (coins: $3, 3$).
Greedy fails because taking $4$ leaves a subproblem ($2$) that requires two 1-coins, whereas taking two 3-coins leaves no remainder.

## 2. Activity selection and interval scheduling [PYQ: Low]

Given $n$ activities with start times $s_i$ and finish times $f_i$, find the maximum number of mutually compatible activities (no two activities overlap in time).

```text
Time line: 0---1---2---3---4---5---6---7---8---9---10
A1:        [-------]              (f1 = 3) -> PICK
A2:            [-----------]      (f2 = 5) -> Skip (conflicts with A1)
A3:                    [-------]  (f3 = 6) -> PICK
A4:                        [-----------] (f4 = 8) -> Skip (conflicts with A3)
A5:                            [-------] (f5 = 8) -> PICK
```

### The Optimal Greedy Strategy: Earliest Finish Time
- Sort activities by finish times $f_i$ in non-decreasing order: $f_1 \le f_2 \le \dots \le f_n$.
- Select $A_1$ (finishes earliest).
- For each subsequent activity $A_i$, select it if $s_i \ge f_{\text{last\_selected}}$.

> **Why other intuitive rules FAIL:**
> 1. *Earliest Start Time:* Counterexample: $A_1 = [0, 100]$ and $A_2 = [1, 2], A_3 = [3, 4]$. Picking $A_1$ yields 1 activity instead of 2.
> 2. *Shortest Duration:* Counterexample: $A_1 = [0, 4], A_2 = [3, 5], A_3 = [4, 8]$. $A_2$ has duration 2, but picking it blocks both $A_1$ and $A_3$.
> 3. *Fewest Conflicts:* Generally does not yield an optimal polynomial-time guarantee and requires recalculating dynamic degree counts.

**Worked example 2 (Activity Selection Trace):**
Consider 6 activities with $(s_i, f_i)$:
$A_1 = (1, 4), A_2 = (3, 5), A_3 = (0, 6), A_4 = (5, 7), A_5 = (3, 9), A_6 = (5, 9)$.

1. Sort by finish time $f_i$:
   - $A_1: (1, 4)$
   - $A_2: (3, 5)$
   - $A_3: (0, 6)$
   - $A_4: (5, 7)$
   - $A_5: (3, 9)$
   - $A_6: (5, 9)$
2. Select $A_1$: finish time $= 4$.
3. Inspect $A_2$: $s_2 = 3 < 4 \implies$ reject.
4. Inspect $A_3$: $s_3 = 0 < 4 \implies$ reject.
5. Inspect $A_4$: $s_4 = 5 \ge 4 \implies$ **select $A_4$**. New finish time $= 7$.
6. Inspect $A_5$: $s_5 = 3 < 7 \implies$ reject.
7. Inspect $A_6$: $s_6 = 5 < 7 \implies$ reject.
Selected set: $\{A_1, A_4\}$. Maximum compatible activities = **2**.

## 3. Fractional knapsack problem [PYQ: Low]

Given $n$ items, each with weight $w_i > 0$ and value $v_i > 0$, and a knapsack of maximum capacity $W$. Any fraction $x_i \in [0, 1]$ of item $i$ can be placed into the knapsack to contribute $x_i v_i$ value and $x_i w_i$ weight.

### Greedy Strategy
1. Calculate the value density ratio for each item:
   $$r_i = \frac{v_i}{w_i}$$
2. Sort items in descending order of ratio: $r_1 \ge r_2 \ge \dots \ge r_n$.
3. Iterate through sorted items:
   - If $w_i \le W$: take the entire item ($x_i = 1$), deduct $W \leftarrow W - w_i$, add $v_i$ to profit.
   - If $w_i > W$: take fraction $x_i = W / w_i$, add $x_i v_i$ to profit, set $W = 0$, and terminate.

### Why Fractional Knapsack Succeeds while 0/1 Fails
In fractional knapsack, capacity can be completely packed without "slack". In 0/1 knapsack, taking an item of high ratio might leave empty capacity that cannot fit any other item, leading to suboptimal space utilization.

**Worked example 3 (Fractional Knapsack Trace):**
Capacity $W = 50$. Items:
- Item 1: $v_1 = 60, w_1 = 10 \implies r_1 = 6.0$
- Item 2: $v_2 = 100, w_2 = 20 \implies r_2 = 5.0$
- Item 3: $v_3 = 120, w_3 = 30 \implies r_3 = 4.0$

Step-by-step:
1. Ratios: $r_1 = 6.0 > r_2 = 5.0 > r_3 = 4.0$.
2. Item 1: $w_1 = 10 \le 50 \implies$ take $x_1 = 1$. Profit $= 60$, remaining capacity $= 50 - 10 = 40$.
3. Item 2: $w_2 = 20 \le 40 \implies$ take $x_2 = 1$. Profit $= 60 + 100 = 160$, remaining capacity $= 40 - 20 = 20$.
4. Item 3: $w_3 = 30 > 20 \implies$ take fraction $x_3 = 20 / 30 = 2/3$.
   Value added $= (2/3) \times 120 = 80$.
   Remaining capacity $= 0$.
Total maximum value $= 60 + 100 + 80 = \mathbf{240}$.

## 4. Huffman coding and optimal prefix codes [PYQ: Medium]

Huffman coding is an optimal greedy algorithm for constructing a variable-length, **prefix-free** binary code. A prefix-free code guarantees that no codeword is a prefix of any other codeword, allowing unambiguous linear decoding without lookahead.

### Structural Properties of Huffman Trees
- **Full Binary Tree:** Every internal node has exactly 2 children (degree 0 or degree 2). There are no degree-1 nodes.
- **Node Counts:** If the alphabet contains $n$ distinct characters:
  $$\text{Leaf nodes} = n, \quad \text{Internal nodes} = n - 1, \quad \text{Total nodes} = 2n - 1$$
- **Cost / Weighted Path Length:**
  $$B(T) = \sum_{i=1}^n f_i \cdot d_T(c_i) = \sum_{v \in \text{Internal Nodes}} \text{weight}(v)$$
  where $f_i$ is character frequency, $d_T(c_i)$ is depth of leaf $c_i$, and $\text{weight}(v)$ is the sum of leaf frequencies in subtree $v$.

### Construction Algorithm using a Min-Heap
```c
/* Pseudocode */
priority_queue Q = build_min_heap(characters);
for (int i = 1; i <= n - 1; i++) {
    node *x = extract_min(Q);
    node *y = extract_min(Q);
    node *z = allocate_node();
    z->left = x;
    z->right = y;
    z->freq = x->freq + y->freq;
    insert(Q, z);
}
return extract_min(Q); /* root of Huffman tree */
```
- Building min-heap: $O(n)$ time.
- $n - 1$ iterations, each doing 2 `extract_min` and 1 `insert`: $(n - 1) \times O(\log n) = O(n \log n)$.
- Total time: $O(n \log n)$. Auxiliary space: $O(n)$.

**Worked example 4 (Huffman Tree Construction):**
Construct Huffman code for alphabet with frequencies:
$A: 5, B: 9, C: 12, D: 13, E: 16, F: 45$ (Total frequency = 100).

```text
Initial min-heap: [A:5, B:9, C:12, D:13, E:16, F:45]

Step 1: Extract A(5), B(9) -> Create internal node N1(14) with children A, B.
        Heap: [C:12, D:13, N1:14, E:16, F:45]

Step 2: Extract C(12), D(13) -> Create internal node N2(25) with children C, D.
        Heap: [N1:14, E:16, N2:25, F:45]

Step 3: Extract N1(14), E(16) -> Create internal node N3(30) with children N1, E.
        Heap: [N2:25, N3:30, F:45]

Step 4: Extract N2(25), N3(30) -> Create internal node N4(55) with children N2, N3.
        Heap: [F:45, N4:55]

Step 5: Extract F(45), N4(55) -> Create root node R(100) with children F, N4.
        Heap: [R:100]
```

```text
               R(100)
              /      \
           (0)        (1)
          F:45       N4(55)
                    /      \
                 (0)        (1)
               N2(25)      N3(30)
              /     \      /    \
            (0)     (1)  (0)    (1)
           C:12    D:13 N1(14)  E:16
                        /    \
                      (0)    (1)
                      A:5    B:9
```

Codewords and bit lengths:
- $F: 45 \implies \text{code } 0 \implies \text{length } 1$
- $C: 12 \implies \text{code } 100 \implies \text{length } 3$
- $D: 13 \implies \text{code } 101 \implies \text{length } 3$
- $E: 16 \implies \text{code } 111 \implies \text{length } 3$
- $A: 5 \implies \text{code } 1100 \implies \text{length } 4$
- $B: 9 \implies \text{code } 1101 \implies \text{length } 4$

Total bits to encode message:
$$B(T) = 45(1) + 12(3) + 13(3) + 16(3) + 5(4) + 9(4) = 45 + 36 + 39 + 48 + 20 + 36 = \mathbf{224} \text{ bits}$$
Sum of internal nodes verification:
$$14 + 25 + 30 + 55 + 100 = \mathbf{224} \text{ bits} \quad \checkmark$$
Average codeword length = $224 / 100 = \mathbf{2.24} \text{ bits/character}$.

## 5. Job sequencing with deadlines and optimal merge patterns [PYQ: Low]

### Job Sequencing with Deadlines
Given $n$ jobs where each job $i$ requires 1 unit of time to execute, has an integer deadline $d_i \ge 1$, and yields profit $p_i > 0$ if completed on or before $d_i$. At most one job can run in each unit time slot $[t - 1, t]$ ($t = 1, 2, \dots, \max(d_i)$).

### Greedy Algorithm
1. Sort jobs in descending order of profit: $p_1 \ge p_2 \ge \dots \ge p_n$.
2. For each job $i$, search for the latest empty time slot $t \in [1, d_i]$:
   - If an empty slot $t$ is found, schedule job $i$ in slot $t$.
   - If all slots $1, 2, \dots, d_i$ are occupied, reject job $i$.
3. Time complexity:
   - Naive linear scan for free slots: $O(n^2)$.
   - Optimized with Disjoint Set Union (DSU) where each set root tracks the latest available slot: $O(n \alpha(n))$.

**Worked example 5 (Job Sequencing Trace):**
Consider 5 jobs:
- $J_1$: Profit $= 20$, Deadline $= 2$
- $J_2$: Profit $= 15$, Deadline $= 2$
- $J_3$: Profit $= 10$, Deadline $= 1$
- $J_4$: Profit $= 5$, Deadline $= 3$
- $J_5$: Profit $= 1$, Deadline $= 3$

Slots available: $[0, 1]$ (slot 1), $[1, 2]$ (slot 2), $[2, 3]$ (slot 3).

1. $J_1$ (profit 20, $d = 2$): Slot 2 is empty $\implies$ assign to slot 2.
2. $J_2$ (profit 15, $d = 2$): Slot 2 occupied; slot 1 is empty $\implies$ assign to slot 1.
3. $J_3$ (profit 10, $d = 1$): Slot 1 occupied $\implies$ reject.
4. $J_4$ (profit 5, $d = 3$): Slot 3 is empty $\implies$ assign to slot 3.
5. $J_5$ (profit 1, $d = 3$): Slots 3, 2, 1 all occupied $\implies$ reject.
Selected jobs: $\{J_2, J_1, J_4\}$ with execution schedule $[J_2 \text{ in slot 1}, J_1 \text{ in slot 2}, J_4 \text{ in slot 3}]$.
Total profit $= 15 + 20 + 5 = \mathbf{40}$.

### Optimal Merge Patterns (2-Way Merge Tree)
Given $n$ sorted files with record counts $s_1, s_2, \dots, s_n$, merging two files of sizes $x$ and $y$ costs $x + y$ comparisons/record movements.
To minimize the grand total record movements:
- Always pick the two smallest files and merge them into a single file of size $x + y$.
- Add the combined file back to the collection.
- This merge tree is structurally identical to a Huffman coding tree! Total record movements equals the sum of all internal nodes.

## Formulas and facts to memorise

| Algorithm / Concept | Sorting Criterion | Time Complexity | Tree / Structural Property |
|---|---|---|---|
| **Activity Selection** | Finish time $f_i$ ascending | $O(n \log n)$ | Optimal choice: earliest finish leaves maximum remaining time |
| **Fractional Knapsack** | Value density $r_i = v_i / w_i$ descending | $O(n \log n)$ | Fractions allowed; optimal value strictly $\ge$ 0/1 knapsack |
| **Huffman Coding** | Character frequencies ascending (min-heap) | $O(n \log n)$ | Full binary tree: $n$ leaves, $n - 1$ internal nodes, $2n - 1$ total nodes |
| **Huffman Cost** | — | — | $B(T) = \sum f_i \cdot \text{length}(c_i) = \sum \text{internal node frequencies}$ |
| **Fibonacci Huffman** | Frequencies $F_1, F_2, \dots, F_n$ | $O(n \log n)$ | Maximally skewed tree: longest code $= n - 1$, shortest code $= 1$ |
| **Job Sequencing** | Profit $p_i$ descending | $O(n^2)$ or $O(n \alpha(n))$ | Assign to latest available slot $t \le d_i$ |
| **Optimal 2-Way Merge** | File sizes ascending (min-heap) | $O(n \log n)$ | Exact equivalence to Huffman tree; cost $= \sum \text{internal nodes}$ |

## GATE traps

- **0/1 Knapsack cannot be solved greedily:** The greedy ratio strategy does not solve the 0/1 Knapsack problem. 0/1 Knapsack requires dynamic programming ($O(nW)$ pseudo-polynomial time).
- **Non-uniqueness of Huffman Codes:** The actual codewords assigned to characters are not unique (swapping 0 and 1 or breaking frequency ties differently gives different codes). However, the **optimal weighted path length (total encoded bits)** and the set of codeword lengths are invariant.
- **Node count in Huffman Tree:** A Huffman tree for $n$ symbols contains exactly $n - 1$ internal nodes and $2n - 1$ total nodes. It *never* contains a node with a single child.
- **Activity selection sorting key:** Sorting by start time or shortest duration does *not* work. Only sorting by non-decreasing finish time guarantees optimality.
- **Job sequencing deadline semantics:** A job with deadline $d$ can execute in any unit interval $[t - 1, t]$ where $1 \le t \le d$. It does *not* have to execute at exactly time $d$.
- **Sum of internal nodes shortcut:** To compute the total bits for a Huffman tree or total record moves in an optimal merge pattern, simply sum all the internal node values created during the heap merges. Do not re-multiply depth by frequency manually unless checking individual codeword lengths.

## Connections

- [Asymptotic analysis](asymptotic-analysis.md) — complexity analysis of heap operations and sorting steps.
- [Binary Heaps and Priority Queues](../07-data-structures/heaps.md) — priority queues provide the $O(\log n)$ min-extraction for Huffman coding, Prim's, and Dijkstra's algorithms.
- [Minimum spanning trees](minimum-spanning-trees.md) — Kruskal's algorithm (sort edges, add safe edges via DSU) and Prim's algorithm (greedy cut choice) are fundamental greedy algorithms.
- [Shortest paths](shortest-paths.md) — Dijkstra's algorithm uses a greedy priority-queue extraction based on minimum tentative distance.
- [Dynamic programming](dynamic-programming.md) — required when optimal substructure exists but the greedy-choice property fails (e.g., 0/1 knapsack, matrix chain multiplication).

## Practice

**Q1 (MCQ · Easy).** A Huffman code is generated for an alphabet of 12 distinct characters with known frequencies. What is the total number of nodes in the resulting Huffman tree?
- (A) 12
- (B) 23
- (C) 24
- (D) 11

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
A Huffman tree on $n$ distinct characters is a full binary tree with $n$ leaves.
In every full binary tree:
$$\text{Internal nodes} = n - 1$$
$$\text{Total nodes} = n + (n - 1) = 2n - 1$$
For $n = 12$:
$$\text{Total nodes} = 2(12) - 1 = 23$$

</details>

**Q2 (NAT · Medium).** A text file consists of 100 characters drawn from an alphabet of six characters $\{a, b, c, d, e, f\}$ with frequencies:
$a: 10, \ b: 5, \ c: 15, \ d: 20, \ e: 25, \ f: 25$.
What is the minimum number of bits required to encode the entire text file using Huffman coding?

<details><summary>Answer</summary>

**Answer:** 245

**Solution:**
Use a min-heap to merge the two smallest frequencies at each step:
1. Merge $b(5)$ and $a(10) \implies$ internal node $N_1 = 15$.
   Remaining frequencies: $\{c: 15, N_1: 15, d: 20, e: 25, f: 25\}$.
2. Merge $c(15)$ and $N_1(15) \implies$ internal node $N_2 = 30$.
   Remaining frequencies: $\{d: 20, e: 25, f: 25, N_2: 30\}$.
3. Merge $d(20)$ and $e(25) \implies$ internal node $N_3 = 45$.
   Remaining frequencies: $\{f: 25, N_2: 30, N_3: 45\}$.
4. Merge $f(25)$ and $N_2(30) \implies$ internal node $N_4 = 55$.
   Remaining frequencies: $\{N_3: 45, N_4: 55\}$.
5. Merge $N_3(45)$ and $N_4(55) \implies$ root node $R = 100$.

Total encoded bits equals the sum of all internal nodes created:
$$\text{Total bits} = N_1 + N_2 + N_3 + N_4 + R = 15 + 30 + 45 + 55 + 100 = 245 \text{ bits}$$

</details>

**Q3 (NAT · Medium).** Consider an instance of the fractional knapsack problem with capacity $W = 60$ and the following four items:
- Item 1: value $= 280$, weight $= 40$
- Item 2: value $= 100$, weight $= 10$
- Item 3: value $= 120$, weight $= 20$
- Item 4: value $= 120$, weight $= 24$

The maximum total value obtainable is ________.

<details><summary>Answer</summary>

**Answer:** 440

**Solution:**
Calculate value-to-weight ratio for each item:
- Item 1: $280 / 40 = 7.0$
- Item 2: $100 / 10 = 10.0$
- Item 3: $120 / 20 = 6.0$
- Item 4: $120 / 24 = 5.0$

Sorted order by ratio: Item 2 (10.0), Item 1 (7.0), Item 3 (6.0), Item 4 (5.0).
Greedy filling:
1. Take all of Item 2: weight used $= 10$, value gained $= 100$. Remaining capacity $= 60 - 10 = 50$.
2. Take all of Item 1: weight used $= 40$, value gained $= 280$. Remaining capacity $= 50 - 40 = 10$.
3. Take fraction of Item 3: available capacity is 10, item weight is 20 $\implies$ fraction $= 10 / 20 = 0.5$.
   Value gained $= 0.5 \times 120 = 60$.
   Remaining capacity $= 0$.

Total maximum value $= 100 + 280 + 60 = 440$.

</details>

**Q4 (MCQ · Medium).** Consider the following set of activities with start and finish times:
$A_1(1, 3), A_2(2, 5), A_3(4, 7), A_4(1, 8), A_5(5, 9), A_6(8, 10)$.
What is the maximum size of a subset of mutually compatible activities?
- (A) 2
- (B) 3
- (C) 4
- (D) 5

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
Activities sorted by finish time:
1. $A_1: (1, 3) \implies$ select $A_1$. Finish time $= 3$.
2. $A_2: (2, 5) \implies s_2 = 2 < 3 \implies$ reject.
3. $A_3: (4, 7) \implies s_3 = 4 \ge 3 \implies$ select $A_3$. Finish time $= 7$.
4. $A_4: (1, 8) \implies s_4 = 1 < 7 \implies$ reject.
5. $A_5: (5, 9) \implies s_5 = 5 < 7 \implies$ reject.
6. $A_6: (8, 10) \implies s_6 = 8 \ge 7 \implies$ select $A_6$. Finish time $= 10$.

Selected compatible subset: $\{A_1, A_3, A_6\}$. Maximum size is 3.

</details>

**Q5 (NAT · Medium).** Seven jobs are given with profits and deadlines as follows:
$J_1(35, 3), J_2(30, 4), J_3(25, 4), J_4(20, 2), J_5(15, 3), J_6(12, 1), J_7(5, 2)$.
Each job takes 1 unit of time to execute. The maximum total profit that can be earned is ________.

<details><summary>Answer</summary>

**Answer:** 110

**Solution:**
Max deadline is 4, so there are 4 time slots: 1, 2, 3, 4.
Sort jobs by decreasing profit:
1. $J_1$ (profit 35, deadline 3): Put in latest slot $\le 3 \implies$ slot 3.
2. $J_2$ (profit 30, deadline 4): Put in latest slot $\le 4 \implies$ slot 4.
3. $J_3$ (profit 25, deadline 4): Slots 4 and 3 are full; put in slot 2.
4. $J_4$ (profit 20, deadline 2): Slot 2 is full; put in slot 1.
5. $J_5$ (profit 15, deadline 3): Slots 3, 2, 1 are all full $\implies$ reject.
6. $J_6$ (profit 12, deadline 1): Slot 1 is full $\implies$ reject.
7. $J_7$ (profit 5, deadline 2): Slots 2 and 1 are full $\implies$ reject.

Scheduled jobs: $\{J_4, J_3, J_1, J_2\}$.
Total profit $= 20 + 25 + 35 + 30 = 110$.

</details>

**Q6 (NAT · Hard).** An alphabet of 8 characters has frequencies proportional to the first 8 Fibonacci numbers: $1, 1, 2, 3, 5, 8, 13, 21$. What is the length of the longest codeword in the optimal Huffman code for this alphabet?

<details><summary>Answer</summary>

**Answer:** 7

**Solution:**
Because $\sum_{i=1}^k F_i = F_{k+2} - 1 < F_{k+2}$, the sum of the two smallest frequencies at each stage is always $\le$ the next Fibonacci number:
1. Combine $1$ and $1 \implies 2$. Remaining: $2, 2, 3, 5, 8, 13, 21$.
2. Combine $2$ and $2 \implies 4$. Remaining: $3, 4, 5, 8, 13, 21$.
3. Combine $3$ and $4 \implies 7$. Remaining: $5, 7, 8, 13, 21$.
4. Combine $5$ and $7 \implies 12$. Remaining: $8, 12, 13, 21$.
5. Combine $8$ and $12 \implies 20$. Remaining: $13, 20, 21$.
6. Combine $13$ and $20 \implies 33$. Remaining: $21, 33$.
7. Combine $21$ and $33 \implies 54$ (root).

At each step, one leaf node merges with the cumulative subtree, resulting in a strictly linear, maximally skewed binary tree.
In a skewed tree of $n$ leaves:
- The longest codeword belongs to the first two characters ($F_1, F_2$) and has length $n - 1$.
- For $n = 8$, length of the longest codeword $= 8 - 1 = 7$.

</details>

**Q7 (MSQ · Hard).** Which of the following statements is/are **CORRECT** regarding greedy algorithms?
- (A) In the activity selection problem, choosing activities in non-decreasing order of duration always produces an optimal solution.
- (B) If a problem possesses optimal substructure, it can always be solved optimally by a greedy algorithm.
- (C) In Huffman coding, changing the binary bit assignment (0 for left, 1 for right to 1 for left, 0 for right) does not change the average codeword length.
- (D) The fractional knapsack problem can be solved in $O(n)$ worst-case time using a deterministic linear-time selection algorithm (median-of-medians) without fully sorting the items.

<details><summary>Answer</summary>

**Answer:** (C), (D)

**Solution:**
- **(A) is Incorrect:** Counterexample: $A_1 = [0, 4]$, $A_2 = [3, 5]$, $A_3 = [4, 8]$. $A_2$ has the shortest duration (2), but picking it allows only 1 activity, whereas picking $A_1$ and $A_3$ yields 2 activities.
- **(B) is Incorrect:** Optimal substructure is necessary for both dynamic programming and greedy algorithms, but greedy also requires the **greedy-choice property**. For example, 0/1 knapsack and Longest Common Subsequence have optimal substructure but cannot be solved by greedy algorithms.
- **(C) is Correct:** Inverting bits simply flips each 0 to 1 and each 1 to 0; the length of each codeword and thus the weighted path length remains identical.
- **(D) is Correct:** Using the median-of-medians algorithm, one can find the median ratio in $O(n)$ time, partition items into ratios greater and smaller than the median, and greedily decide whether the entire top half fits into the knapsack. This yields the recurrence $T(n) = T(n/2) + O(n) = O(n)$ time.

</details>
