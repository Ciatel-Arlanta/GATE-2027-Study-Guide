# Dynamic Programming

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** optimal substructure, overlapping subproblems, memoization, tabulation, classic DP
> **Prerequisites:** [Recurrences](../01-discrete-mathematics/recurrences-and-generating-functions.md), [Divide and conquer](divide-and-conquer.md) · **Leads to:** [Shortest paths](shortest-paths.md), [Greedy algorithms](greedy-algorithms.md)
> **PYQ weightage:** Medium · 6 Qs since 2015 (1 since 2021) · DA: 0 · Trend: falling · Most asked: Matrix chain multiplication, Longest common subsequence (LCS), 0/1 knapsack, Subset sum & DP state order

## Quick glance

- **Exam focus:** Matrix Chain Multiplication (parenthesization and scalar multiplication count), Longest Common Subsequence (table filling, backtracking, counting distinct LCSs), 0/1 Knapsack (state recurrence, space optimization, pseudo-polynomial complexity), and Subset Sum / DP state evaluation order. Question types: NAT (minimum scalar multiplications, LCS length, knapsack maximum value) and MCQ/MSQ (recurrence validity, state dependencies, time/space complexity).
- **Two core prerequisites:**
  1. **Optimal Substructure:** An optimal solution to the problem contains within it optimal solutions to subproblems.
  2. **Overlapping Subproblems:** A recursive algorithm visits the *same* smaller subproblems repeatedly, rather than generating new subproblems at every branch.
- **Approaches:**
  - **Top-Down (Memoization):** Write recursion naturally; cache subproblem solutions in a hash table or array upon return. Overhead: recursion call stack.
  - **Bottom-Up (Tabulation):** Solve smallest subproblems first and iteratively fill a table in topological order of the subproblem DAG. No recursion stack overhead.
- **Complexity derivation:**
  $$\text{Time Complexity} = (\text{Total number of states}) \times (\text{Work done per state})$$
- **Longest Common Subsequence (LCS):** For prefixes $X[1..m]$ and $Y[1..n]$:
  - If $X[i] == Y[j] \implies c[i][j] = 1 + c[i-1][j-1]$.
  - If $X[i] \ne Y[j] \implies c[i][j] = \max(c[i-1][j], c[i][j-1])$.
  - Time: $\Theta(mn)$, Space: $\Theta(mn)$ (can be reduced to $O(\min(m, n))$ for length only).
- **Matrix Chain Multiplication (MCM):** Multiplying $n$ matrices $A_1 \dots A_n$ of dimensions $p_{i-1} \times p_i$:
  - Recurrence: $m[i][j] = \min_{i \le k < j} \{m[i][k] + m[k+1][j] + p_{i-1} p_k p_j\}$.
  - Time: $\Theta(n^3)$, Space: $\Theta(n^2)$. Total parenthesizations: Catalan number $C_{n-1} = \frac{1}{n}\binom{2n-2}{n-1}$.
- **0/1 Knapsack:** $dp[i][w] = \max(dp[i-1][w], v_i + dp[i-1][w - w_i])$.
  - Time: $\Theta(nW)$, Space: $\Theta(nW)$ (reducible to $O(W)$ by traversing backwards).
  - Pseudo-polynomial: Polynomial in numerical value $W$, but exponential in the bit-length of $W$ ($\log_2 W$). It is NP-complete.
- **Subset Sum:** State $dp[i][s] = dp[i-1][s] \lor dp[i-1][s - A[i]]$. Time: $\Theta(n \cdot S)$.

## 1. Principles of dynamic programming and state design [PYQ: Low]

Dynamic Programming (DP) solves optimization problems by combining solutions to subproblems. Unlike Divide-and-Conquer (where subproblems are independent and disjoint), DP applies when subproblems share sub-subproblems.

```text
Divide-and-Conquer (Merge Sort):     Dynamic Programming (Fibonacci / LCS):
             [Root]                                  [F(4)]
            /      \                                 /     \
         [Left]   [Right]                         [F(3)]   [F(2)]
         /    \   /     \                         /    \     |
       [L1]  [L2] [R1]  [R2]                   [F(2)] [F(1)] [F(1)]
   (Disjoint subproblem tree)             (Overlapping subproblem DAG)
```

### Bellman's Principle of Optimality
An optimal policy has the property that whatever the initial state and decision are, the remaining decisions must constitute an optimal policy with regard to the state resulting from the first decision.

### Memoization vs Tabulation

| Dimension | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| **Mechanism** | Recursive with lookup table / cache | Iterative loop filling table |
| **State traversal** | On-demand (only reachable states computed) | Exhaustive (computes all table entries) |
| **Call Stack** | Uses $O(\text{recursion depth})$ stack space | $O(1)$ auxiliary stack space |
| **Evaluation Order** | Automatic (governed by recursion tree) | Explicit (must follow DAG topological sort) |

### Topological Evaluation of DP DAG
Every DP subproblem dependence forms a Directed Acyclic Graph (DAG):
- A state $u$ depends on state $v$ if computing $dp[u]$ requires $dp[v]$ (directed edge $v \to u$).
- Table computation must evaluate states in a valid **topological sort** of this DAG.

**Worked example 1 (DP State Evaluation Order):**
Consider the grid recurrence for a table $T[0..n, 0..m]$:
$$T[i, j] = T[i-1, j] + T[i, j-1] + T[i-1, j-1]$$
- Cell $(i, j)$ depends directly on its top neighbor $(i-1, j)$, left neighbor $(i, j-1)$, and diagonal neighbor $(i-1, j-1)$.
- Valid evaluation orders:
  1. Row-major order: outer loop $i = 0 \dots n$, inner loop $j = 0 \dots m$.
  2. Column-major order: outer loop $j = 0 \dots m$, inner loop $i = 0 \dots n$.
  3. Anti-diagonal wavefront: outer loop $k = i + j = 0 \dots n + m$.
- Invalid evaluation order: Reverse row-major ($i = n \dots 0$) because cell $(i, j)$ would attempt to read uncomputed values from earlier rows.

## 2. Longest Common Subsequence and sequence alignment [PYQ: Medium]

A **subsequence** of a string is obtained by deleting zero or more characters without changing the relative order of the remaining characters (e.g., `ACE` is a subsequence of `ABCDE`). A **substring** must be contiguous.

### Recurrence Formulation
Let $X = \langle x_1, x_2, \dots, x_m \rangle$ and $Y = \langle y_1, y_2, \dots, y_n \rangle$.
Let $c[i, j]$ denote the length of an LCS of prefixes $X_i$ and $Y_j$:

$$c[i, j] = \begin{cases} 
0 & \text{if } i = 0 \text{ or } j = 0 \\
c[i-1, j-1] + 1 & \text{if } i, j > 0 \text{ and } x_i = y_j \\
\max(c[i-1, j], c[i, j-1]) & \text{if } i, j > 0 \text{ and } x_i \ne y_j 
\end{cases}$$

- **State space:** $(m + 1) \times (n + 1)$ entries.
- **Time complexity:** $\Theta(mn)$ since each cell does $O(1)$ comparisons.
- **Space complexity:** $\Theta(mn)$ to store the full table for backtracking the actual sequence. If only the *length* is needed, row $i$ depends only on row $i-1$, reducing space to $O(\min(m, n))$.

**Worked example 2 (LCS Table and Multiple Solutions):**
Find the length and all distinct LCSs of $X = \text{"ABCBDAB"}$ ($m = 7$) and $Y = \text{"BDCABA"}$ ($n = 6$).

```text
Table c[i, j]:
      j:   0    1    2    3    4    5    6
           Y:   B    D    C    A    B    A
i: 0  X:   0    0    0    0    0    0    0
1  A       0    0    0    0   1(↖)  1    1(↖)
2  B       0   1(↖)  1    1    1   2(↖)  2
3  C       0    1    1   2(↖)  2    2    2
4  B       0   1(↖)  1    2    2   3(↖)  3
5  D       0    1   2(↖)  2    2    3    3
6  A       0    1    2    2   3(↖)  3   4(↖)
7  B       0   1(↖)  2    2    3   4(↖)  4
```

1. Look at $c[7, 6] = 4 \implies$ maximum LCS length is **4**.
2. Backtracking from $c[7, 6]$ where branches have equal max values reveals three distinct LCS strings:
   - Path 1: `B -> C -> B -> A` $\implies$ **`BCBA`**
   - Path 2: `B -> C -> A -> B` $\implies$ **`BCAB`**
   - Path 3: `B -> D -> A -> B` $\implies$ **`BDAB`**
Total distinct LCS of maximal length = **3**.

### Longest Increasing Subsequence (LIS) Connection
Given an array $A$ of $n$ numbers, the LIS can be found in two ways:
1. **Via LCS:** Sort $A$ to obtain $A_{\text{sorted}}$ (removing duplicates). Run LCS on $A$ and $A_{\text{sorted}}$. Time: $O(n^2)$, Space: $O(n^2)$.
2. **Via DP directly:** $L[i] = 1 + \max(\{L[j] \mid j < i \text{ and } A[j] < A[i]\} \cup \{0\})$. Time: $O(n^2)$.
3. **Patience Sorting / Binary Search:** Maintain the smallest tail of all increasing subsequences of length $k$. Updates take $O(\log n)$ via binary search $\implies$ Time: $O(n \log n)$.

## 3. Matrix Chain Multiplication [PYQ: Medium]

Given a sequence (chain) of $n$ matrices $\langle A_1, A_2, \dots, A_n \rangle$ where matrix $A_i$ has dimension $p_{i-1} \times p_i$. We wish to compute the product $A_1 A_2 \dots A_n$ using the minimum number of scalar multiplications.

Matrix multiplication is associative: $(A_1 A_2) A_3 = A_1 (A_2 A_3)$, but the cost varies drastically:
- Multiplying $A_{p \times q}$ and $B_{q \times r}$ requires $p \cdot q \cdot r$ scalar multiplications.
- The number of parenthesizations of $n$ matrices is given by the **Catalan number**:
  $$P(n) = C_{n-1} = \frac{1}{n} \binom{2n-2}{n-1} = \Omega\left(\frac{4^n}{n^{3/2}}\right)$$

### Recurrence
Let $m[i, j]$ be the minimum number of scalar multiplications needed to compute $A_i \dots A_j$ ($1 \le i \le j \le n$):

$$m[i, j] = \begin{cases} 
0 & \text{if } i = j \\
\min_{i \le k < j} \left\{ m[i, k] + m[k+1, j] + p_{i-1} p_k p_j \right\} & \text{if } i < j 
\end{cases}$$

- **Table size:** Upper triangular matrix of size $n \times n \implies \Theta(n^2)$ states.
- **Filling order:** By chain length $L = j - i + 1$ from $L = 2$ up to $n$.
- **Work per state:** At state $m[i, j]$, $k$ takes $j - i = L - 1$ possible split values $\implies O(n)$ work per cell.
- **Total Time:** $\sum_{L=2}^n (n - L + 1)(L - 1) = \Theta(n^3)$. Total Space: $\Theta(n^2)$.

**Worked example 3 (MCM Step-by-Step):**
Find the minimum scalar multiplications to multiply 4 matrices:
$A_1(10 \times 30), A_2(30 \times 5), A_3(5 \times 60), A_4(60 \times 10)$.
Dimension array: $p = [10, 30, 5, 60, 10]$ ($n = 4$).

```text
Chain length L = 1:
m[1, 1] = 0, m[2, 2] = 0, m[3, 3] = 0, m[4, 4] = 0

Chain length L = 2:
m[1, 2] = p0 * p1 * p2 = 10 * 30 * 5  = 1500  (split k = 1)
m[2, 3] = p1 * p2 * p3 = 30 * 5 * 60  = 9000  (split k = 2)
m[3, 4] = p2 * p3 * p4 = 5 * 60 * 10  = 3000  (split k = 3)

Chain length L = 3:
m[1, 3]:
  k = 1: m[1, 1] + m[2, 3] + p0*p1*p3 = 0 + 9000 + 10*30*60 = 27000
  k = 2: m[1, 2] + m[3, 3] + p0*p2*p3 = 1500 + 0 + 10*5*60  = 4500
  min = 4500 at k = 2.

m[2, 4]:
  k = 2: m[2, 2] + m[3, 4] + p1*p2*p4 = 0 + 3000 + 30*5*10  = 4500
  k = 3: m[2, 3] + m[4, 4] + p1*p3*p4 = 9000 + 0 + 30*60*10 = 27000
  min = 4500 at k = 2.

Chain length L = 4 (m[1, 4]):
  k = 1: m[1, 1] + m[2, 4] + p0*p1*p4 = 0 + 4500 + 10*30*10 = 7500
  k = 2: m[1, 2] + m[3, 4] + p0*p2*p4 = 1500 + 3000 + 10*5*10 = 5000  <- MINIMUM
  k = 3: m[1, 3] + m[4, 4] + p0*p3*p4 = 4500 + 0 + 10*60*10 = 10500
  min = 5000 at k = 2.
```

- Minimum scalar multiplications = **5000**.
- Optimal parenthesization from split points: $k = 2$ splits $(A_1 A_2)$ and $(A_3 A_4) \implies$ **$((A_1 A_2)(A_3 A_4))$**.

## 4. 0/1 Knapsack and Subset Sum [PYQ: Low]

Given $n$ items, item $i$ having weight $w_i > 0$ and value $v_i > 0$, and knapsack capacity $W$. Each item can be picked at most once ($x_i \in \{0, 1\}$).

### Recurrence
Let $dp[i, w]$ be the maximum value achievable using a subset of the first $i$ items with total weight $\le w$:

$$dp[i, w] = \begin{cases} 
0 & \text{if } i = 0 \text{ or } w = 0 \\
dp[i-1, w] & \text{if } w_i > w \\
\max(dp[i-1, w], v_i + dp[i-1, w - w_i]) & \text{if } w_i \le w 
\end{cases}$$

- **Complexity:** $O(nW)$ time and $O(nW)$ space.
- **Space Optimization (1D Array):** Because $dp[i, w]$ depends only on row $i-1$, we can maintain a single array `dp[0..W]`.
  > **Crucial Rule:** Update `w` **backwards** from $W$ down to $w_i$:
  > ```c
  > for (int i = 0; i < n; i++)
  >     for (int w = W; w >= weight[i]; w--)
  >         dp[w] = max(dp[w], val[i] + dp[w - weight[i]]);
  > ```
  > If `w` is iterated forwards ($w_i \dots W$), an item can be included multiple times, which solves the **Unbounded Knapsack** problem instead!

### Why 0/1 Knapsack is Pseudo-Polynomial
The input size of capacity $W$ in binary representation is $L = \lceil \log_2 W \rceil$ bits.
The runtime $O(n W) = O(n \cdot 2^L)$ is polynomial in the magnitude of $W$, but **exponential** in the length of the input. Thus, 0/1 Knapsack is NP-complete, and this DP is pseudo-polynomial.

**Worked example 4 (0/1 Knapsack Table):**
Capacity $W = 7$, 4 items:
1. Item 1: $(w_1 = 1, v_1 = 1)$
2. Item 2: $(w_2 = 3, v_2 = 4)$
3. Item 3: $(w_3 = 4, v_3 = 5)$
4. Item 4: $(w_4 = 5, v_4 = 7)$

```text
Table dp[i, w]:
 i \ w:  0   1   2   3   4   5   6   7
---------------------------------------
   0     0   0   0   0   0   0   0   0
 1(1,1)  0   1   1   1   1   1   1   1
 2(3,4)  0   1   1   4   5   5   5   5
 3(4,5)  0   1   1   4   5   6   6   9
 4(5,7)  0   1   1   4   5   7   8   9
```

- Maximum value $= dp[4, 7] = \mathbf{9}$.
- Backtracking:
  - $dp[4, 7] = dp[3, 7] = 9 \implies$ Item 4 not needed.
  - $dp[3, 7] = v_3 + dp[2, 7 - 4] = 5 + dp[2, 3] = 5 + 4 = 9 \implies$ **Include Item 3** ($w=4, v=5$).
  - Remaining capacity $= 7 - 4 = 3$.
  - $dp[2, 3] = v_2 + dp[1, 3 - 3] = 4 + dp[1, 0] = 4 + 0 = 4 \implies$ **Include Item 2** ($w=3, v=4$).
  - Remaining capacity $= 3 - 3 = 0$.
Included items: $\{Item\ 2, Item\ 3\}$ with total weight $3 + 4 = 7$ and total value $4 + 5 = 9$.

### Subset Sum Problem
Given a set of positive integers $S = \{a_1, a_2, \dots, a_n\}$ and a target sum $K$. Is there a subset of $S$ that sums to $K$?
- Special case of 0/1 knapsack where each item has $w_i = v_i = a_i$.
- State: $dp[i, s] = dp[i-1, s] \lor dp[i-1, s - a_i]$ (Boolean table of size $(n+1) \times (K+1)$).
- Time: $O(nK)$, Space: $O(K)$.

## 5. Edit distance and shortest paths connections [PYQ: Low]

### Edit Distance (Levenshtein Distance)
Given two strings $X[1..m]$ and $Y[1..n]$, find the minimum number of edit operations (Insert, Delete, Replace) to convert $X$ to $Y$:

$$E[i, j] = \begin{cases} 
j & \text{if } i = 0 \text{ (insert all)} \\
i & \text{if } j = 0 \text{ (delete all)} \\
E[i-1, j-1] & \text{if } X[i] == Y[j] \\
1 + \min\begin{pmatrix} E[i, j-1] & (\text{Insert}) \\ E[i-1, j] & (\text{Delete}) \\ E[i-1, j-1] & (\text{Replace}) \end{pmatrix} & \text{if } X[i] \ne Y[j]
\end{cases}$$

Time: $\Theta(mn)$, Space: $\Theta(mn)$ (or $O(\min(m, n))$ space).

### Dynamic Programming in Graph Algorithms
Classic graph shortest path algorithms are direct formulations of DP:
1. **Floyd-Warshall (All-Pairs Shortest Paths):**
   State $D^{(k)}[i, j]$ is shortest path from $i$ to $j$ using intermediate vertices only from $\{1, 2, \dots, k\}$:
   $$D^{(k)}[i, j] = \min\left( D^{(k-1)}[i, j], \ D^{(k-1)}[i, k] + D^{(k-1)}[k, j] \right)$$
   Time: $\Theta(V^3)$, Space: $\Theta(V^2)$.
2. **Bellman-Ford (Single-Source Shortest Paths):**
   State $d^{(k)}[v]$ is shortest path from source $s$ to $v$ using at most $k$ edges:
   $$d^{(k)}[v] = \min_{u \in V} \left( d^{(k-1)}[v], \ d^{(k-1)}[u] + w(u, v) \right)$$
   Time: $O(V \cdot E)$.

## Formulas and facts to memorise

| DP Problem | State Definition | Recurrence / Core Relation | Time | Space |
|---|---|---|---|---|
| **LCS** | $c[i, j]$ = length of LCS of $X_{1..i}, Y_{1..j}$ | If match: $1 + c[i-1, j-1]$; else $\max(c[i-1, j], c[i, j-1])$ | $\Theta(mn)$ | $\Theta(mn)$ (or $O(\min(m,n))$) |
| **Matrix Chain** | $m[i, j]$ = min multiplications for $A_{i..j}$ | $\min_{i \le k < j} \{m[i, k] + m[k+1, j] + p_{i-1} p_k p_j\}$ | $\Theta(n^3)$ | $\Theta(n^2)$ |
| **Catalan $C_n$** | Number of parenthesizations | $C_n = \frac{1}{n+1}\binom{2n}{n}$; For $n$ matrices: $C_{n-1}$ | — | — |
| **0/1 Knapsack** | $dp[i, w]$ = max value with items $1..i$, cap $w$ | $\max(dp[i-1, w], v_i + dp[i-1, w - w_i])$ | $\Theta(nW)$ | $\Theta(nW)$ (or $O(W)$) |
| **Subset Sum** | $dp[i, s]$ = can form sum $s$ with $1..i$ | $dp[i-1, s] \lor dp[i-1, s - A[i]]$ | $\Theta(nS)$ | $O(S)$ |
| **Edit Distance** | $E[i, j]$ = min edits for $X_{1..i} \to Y_{1..j}$ | $\min(\text{insert}, \text{delete}, \text{replace}) + 1$ | $\Theta(mn)$ | $\Theta(mn)$ |
| **Floyd-Warshall** | $D^{(k)}[i, j]$ = APSP via vertices $\{1..k\}$ | $\min(D^{(k-1)}[i, j], D^{(k-1)}[i, k] + D^{(k-1)}[k, j])$ | $\Theta(V^3)$ | $\Theta(V^2)$ |

## GATE traps

- **Index off-by-one in Matrix Chain Multiplication:** For $n$ matrices, the dimension array $p$ has length $n + 1$. The scalar cost term for split $k$ is $p_{i-1} p_k p_j$, **not** $p_i p_k p_j$.
- **Catalan index for matrix chains:** The number of ways to parenthesize $n$ matrices is $C_{n-1}$, not $C_n$. For 4 matrices, it is $C_3 = 5$, not $C_4 = 14$. For 5 matrices, it is $C_4 = 14$.
- **0/1 Knapsack vs Unbounded Knapsack loop order:** When using a 1D DP array, looping $w$ backwards ($W \to w_i$) prevents an item from being used repeatedly (0/1 Knapsack). Looping $w$ forwards ($w_i \to W$) allows multiple copies of the same item (Unbounded Knapsack).
- **LCS vs Longest Common Substring:** A substring must be continuous. If $X[i] \ne Y[j]$ in the substring problem, $dp[i, j] = 0$ (reset), whereas in LCS, $dp[i, j] = \max(dp[i-1, j], dp[i, j-1])$.
- **Pseudo-polynomial misconception:** $O(nW)$ is *not* a polynomial-time algorithm in the formal complexity sense. It is pseudo-polynomial because $W$ is a numeric value whose representation requires only $\log_2 W$ bits.
- **Topological order in table filling:** In MCM, states must be filled in order of increasing chain length $L = j - i + 1$, not simple row-major order, because $m[i, j]$ depends on $m[k+1, j]$ which lies below it in the table.

## Connections

- [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) — Catalan numbers and state recurrences.
- [Divide and conquer](divide-and-conquer.md) — contrast between disjoint subproblems (D&C) and overlapping subproblems (DP).
- [Greedy algorithms](greedy-algorithms.md) — greedy commits to a single choice; DP evaluates all choices and caches optimal subproblems.
- [Shortest paths](shortest-paths.md) — Floyd-Warshall and Bellman-Ford algorithms are direct applications of the principle of optimality.

## Practice

**Q1 (NAT · Easy).** The number of distinct ways to fully parenthesize the matrix product $A_1 \times A_2 \times A_3 \times A_4 \times A_5$ is ________.

<details><summary>Answer</summary>

**Answer:** 14

**Solution:**
The number of distinct parenthesizations of the product of $n$ matrices is given by the $(n-1)$-th Catalan number:
$$P(n) = C_{n-1} = \frac{1}{n} \binom{2n-2}{n-1}$$
For $n = 5$ matrices:
$$P(5) = C_4 = \frac{1}{5} \binom{8}{4} = \frac{1}{5} \times \frac{8 \times 7 \times 6 \times 5}{4 \times 3 \times 2 \times 1} = \frac{70}{5} = 14$$

</details>

**Q2 (NAT · Medium).** The length of the Longest Common Subsequence of the strings $X = \text{"AGGTAB"}$ and $Y = \text{"GXTXAYB"}$ is ________.

<details><summary>Answer</summary>

**Answer:** 4

**Solution:**
Trace the common matching characters preserving left-to-right order:
- $X = \text{A } \mathbf{G} \text{ } \mathbf{T} \text{ } \mathbf{A} \text{ } \mathbf{B}$
- $Y = \mathbf{G} \text{ X } \mathbf{T} \text{ X } \mathbf{A} \text{ Y } \mathbf{B}$
The common subsequence is $\mathbf{G T A B}$ of length 4.
No common subsequence of length 5 exists because the remaining characters $\{A, G\}$ in $X$ and $\{X, Y\}$ in $Y$ do not align.

</details>

**Q3 (NAT · Medium).** Three matrices $A_1, A_2, A_3$ have dimensions $10 \times 100$, $100 \times 5$, and $5 \times 50$ respectively. The minimum number of scalar multiplications required to compute $A_1 \times A_2 \times A_3$ is ________.

<details><summary>Answer</summary>

**Answer:** 7500

**Solution:**
Dimension array: $p = [10, 100, 5, 50]$ with $n = 3$.
There are two possible parenthesizations:
1. $(A_1 A_2) A_3$:
   - Multiply $A_1 (10 \times 100)$ and $A_2 (100 \times 5)$: $10 \times 100 \times 5 = 5000$. Result has dimension $10 \times 5$.
   - Multiply $(A_1 A_2) (10 \times 5)$ and $A_3 (5 \times 50)$: $10 \times 5 \times 50 = 2500$.
   - Total multiplications $= 5000 + 2500 = 7500$.
2. $A_1 (A_2 A_3)$:
   - Multiply $A_2 (100 \times 5)$ and $A_3 (5 \times 50)$: $100 \times 5 \times 50 = 25000$. Result has dimension $100 \times 50$.
   - Multiply $A_1 (10 \times 100)$ and $(A_2 A_3) (100 \times 50)$: $10 \times 100 \times 50 = 50000$.
   - Total multiplications $= 25000 + 50000 = 75000$.

The minimum number of scalar multiplications is $\min(7500, 75000) = 7500$.

</details>

**Q4 (MCQ · Medium).** When optimizing the 0/1 Knapsack dynamic programming table from 2D space $O(nW)$ to 1D space $O(W)$, why must the capacity loop run in reverse order (from $W$ down to $w_i$)?
- (A) To improve CPU cache spatial locality.
- (B) To ensure each item is considered at most once by reading values only from the previous item's state.
- (C) To handle negative item values correctly.
- (D) Because topological sorting of the knapsack DAG requires decreasing capacity order.

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
In the 2D recurrence, $dp[i, w]$ depends on $dp[i-1, w - w_i]$ from the *previous* row.
When compressed to 1D array `dp[w]`:
- If we update backwards ($w = W \dots w_i$), the value `dp[w - w_i]` still holds the result from the $(i-1)$-th iteration (not yet overwritten by item $i$).
- If we update forwards ($w = w_i \dots W$), `dp[w - w_i]` would have already been updated with item $i$, allowing item $i$ to be added multiple times (which solves Unbounded Knapsack).

</details>

**Q5 (NAT · Medium).** Given a set of integers $S = \{2, 3, 7, 8, 10\}$ and target sum $K = 11$. What is the minimum number of elements from $S$ required to form a subset whose sum is exactly $K$?

<details><summary>Answer</summary>

**Answer:** 2

**Solution:**
We search for subsets summing to 11:
- Subsets of size 1: none equals 11.
- Subsets of size 2:
  - $3 + 8 = 11$
No single element can form 11 ($10 < 11$). The subset $\{3, 8\}$ has size 2 and sums to 11. Thus, the minimum number of elements is 2.

</details>

**Q6 (MSQ · Hard).** Which of the following statements is/are **CORRECT** regarding dynamic programming?
- (A) Top-down DP with memoization always explores the same or fewer subproblems than bottom-up tabulation.
- (B) The 0/1 knapsack problem can be solved in strictly polynomial time $O(nW)$ where $n$ is the number of items and $W$ is the maximum capacity.
- (C) The Floyd-Warshall algorithm can compute all-pairs shortest paths on graphs with negative edge weights as long as there are no negative weight cycles.
- (D) If a problem exhibits overlapping subproblems, greedy algorithms are guaranteed to find the globally optimal solution.

<details><summary>Answer</summary>

**Answer:** (A), (C)

**Solution:**
- **(A) is Correct:** Top-down memoization visits only the states that are reachable from the original root call. Bottom-up tabulation blindly fills all entries in the table, potentially computing unreachable subproblems.
- **(B) is Incorrect:** $O(nW)$ is pseudo-polynomial, not polynomial. The input size of $W$ is $\log_2 W$ bits, making $O(nW)$ exponential in the input size.
- **(C) is Correct:** Floyd-Warshall correctly finds all-pairs shortest paths with negative edges provided no negative cycles exist (it detects negative cycles if $D[i, i] < 0$ for any $i$).
- **(D) is Incorrect:** Overlapping subproblems is a property of dynamic programming. Greedy algorithms require the greedy-choice property; problems like 0/1 Knapsack and LCS have overlapping subproblems but cannot be solved greedily.

</details>

**Q7 (NAT · Hard).** Consider the strings $X = \text{"aba"}$ and $Y = \text{"bab"}$. The number of distinct longest common subsequences between $X$ and $Y$ is ________.

<details><summary>Answer</summary>

**Answer:** 2

**Solution:**
Find the LCS length of $X = \text{"aba"}$ and $Y = \text{"bab"}$:
- Subsequences of $X$: `{"", "a", "b", "ab", "aa", "ba", "aba"}`
- Subsequences of $Y$: `{"", "b", "a", "ba", "bb", "ab", "bab"}`

Intersection of subsequences:
- Length 1: `{"a", "b"}`
- Length 2: `{"ab", "ba"}`
- Length 3: `{"aba"}` is not in $Y$, `{"bab"}` is not in $X$.

The maximum LCS length is 2.
The distinct common subsequences of maximum length 2 are `"ab"` and `"ba"`.
Hence, there are exactly 2 distinct longest common subsequences.

</details>
