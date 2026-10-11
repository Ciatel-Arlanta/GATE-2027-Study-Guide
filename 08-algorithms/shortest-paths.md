# Shortest Paths: Single-Source and All-Pairs

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** Single-source shortest paths, Dijkstra's algorithm, Bellman–Ford algorithm, all-pairs shortest paths, Floyd–Warshall algorithm, DAG shortest paths
> **Prerequisites:** [Graph traversals](graph-traversals.md) · [Dynamic programming](dynamic-programming.md) · [Heaps and priority queues](../07-data-structures/heaps.md) · **Leads to:** [Network routing (OSPF, RIP, BGP)](../15-computer-networks/routing.md)
> **PYQ weightage:** Low · 5 Qs since 2015 (4 since 2021) · DA: 1 · Trend: rising · Most asked: Dijkstra execution trace and heap complexity, Bellman-Ford negative cycle detection, Floyd-Warshall recurrence

## Quick glance

- **Exam focus:** Step-by-step vertex extraction order in Dijkstra (MCQ/NAT), identifying negative-weight cycles with Bellman–Ford (MSQ), and computing intermediate stage values in Floyd–Warshall (NAT).
- **Relaxation primitive:** For edge $(u, v)$ with weight $w(u, v)$, relaxation tests if traveling through $u$ offers a shorter path: `if (d[u] + w(u, v) < d[v]) { d[v] = d[u] + w(u, v); parent[v] = u; }`.
- **Dijkstra's algorithm:** Solves single-source shortest paths with **non-negative edge weights** ($w \ge 0$). Greedily finalizes the vertex with minimum tentative distance. Time: $O(V^2)$ with array, $O(E \log V)$ with binary heap, $O(V \log V + E)$ with Fibonacci heap.
- **Negative edges break Dijkstra:** Dijkstra assumes finalized vertices never need distance reduction; a negative-weight edge can invalidate already finalized vertices, yielding incorrect distances.
- **Bellman–Ford algorithm:** Solves single-source shortest paths with **arbitrary edge weights** (positive and negative). Relaxes all $E$ edges $V - 1$ times in $O(V \cdot E)$ time.
- **Negative cycle detection:** If a distance decreases on the $V$-th relaxation pass, a **negative-weight cycle** reachable from the source exists (shortest path is $-\infty$).
- **DAG shortest paths:** In a Directed Acyclic Graph, topological sort allows shortest paths to be found in **$O(V + E)$ time** by relaxing outgoing edges in topological order (handles negative weights; finds longest path by negating weights).
- **Floyd–Warshall algorithm:** Computes **All-Pairs Shortest Paths (APSP)** in $O(V^3)$ time and $O(V^2)$ space using dynamic programming: $D^{(k)}[i][j] = \min(D^{(k-1)}[i][j], D^{(k-1)}[i][k] + D^{(k-1)}[k][j])$.
- **Outer loop in Floyd–Warshall:** Intermediate vertex $k$ **must** be the outermost loop. A negative diagonal entry $D[i][i] < 0$ after completion indicates a negative-weight cycle.
- **SPT vs. MST:** Shortest Path Tree minimizes distances from root to vertices; MST minimizes sum of edge weights. Adding constant $c > 0$ to every edge preserves the MST, but **can alter the shortest path**.

## 1. Relaxation and Shortest Path Properties [PYQ: Medium]

Given a weighted directed graph $G = (V, E)$ with weight function $w: E \to \mathbb{R}$, the weight of a path $p = \langle v_0, v_1, \dots, v_k \rangle$ is $w(p) = \sum_{i=1}^k w(v_{i-1}, v_i)$. The shortest path distance from $u$ to $v$ is $\delta(u, v) = \min \{w(p) \mid p \text{ is a path from } u \text{ to } v\}$.

### Fundamental properties
1. **Optimal Substructure:** Any subpath of a shortest path is itself a shortest path. If $p_{s \to v} = s \rightsquigarrow u \to v$ is a shortest path, then the prefix subpath $s \rightsquigarrow u$ is a shortest path from $s$ to $u$.
2. **Triangle Inequality:** For all vertices $u, v, x \in V$:
   $$\delta(u, v) \le \delta(u, x) + \delta(x, v)$$
3. **Upper-bound property:** The tentative distance $d[v]$ always satisfies $d[v] \ge \delta(s, v)$. Once $d[v]$ achieves $\delta(s, v)$, it never increases.
4. **Simple path bound:** If $G$ contains no negative-weight cycles reachable from $s$, every shortest path is simple and contains at most **$V - 1$ edges**.

### Shortest paths on Directed Acyclic Graphs (DAGs) in $O(V + E)$
If $G$ is a DAG, we can bypass general priority queues or iterative passes:
1. Topologically sort $V$ in $O(V + E)$ time.
2. Initialize $d[s] = 0$, all other $d[v] = \infty$.
3. For each vertex $u$ in topological order:
   - For each outgoing edge $(u, v) \in E$: relax $(u, v)$.
- **Time complexity:** $O(V + E)$.
- **Key advantage:** Works correctly with **negative edge weights** (since a DAG cannot contain cycles).
- **Longest path on DAG:** Negate all edge weights and run DAG shortest paths, or initialize $d[s]=0, d[v]=-\infty$ and maximize during relaxation. Solves Critical Path Method (CPM) in $O(V + E)$ time (whereas longest path in general graphs is NP-hard).

---

## 2. Dijkstra's Algorithm (Non-negative Weights) [PYQ: High]

Dijkstra's algorithm maintains a set $S$ of vertices whose final shortest-path weights from source $s$ have already been determined. It repeatedly selects the vertex $u \in V \setminus S$ with the minimum tentative distance $d[u]$, adds $u$ to $S$, and relaxes all edges leaving $u$.

### Algorithm specification
1. Initialize $d[v] = \infty$, $\text{parent}[v] = \text{NIL}$ for all $v \in V$. Set $d[s] = 0$.
2. Insert all vertices into priority queue $Q$ with key $d[v]$.
3. While $Q$ is not empty:
   - $u = \text{Extract-Min}(Q)$.
   - For each adjacent neighbor $v \in \text{Adj}[u]$:
     - If $d[u] + w(u, v) < d[v]$:
       - $d[v] = d[u] + w(u, v)$; $\text{parent}[v] = u$; $\text{Decrease-Key}(Q, v, d[v])$.

### Why negative edge weights break Dijkstra
Dijkstra's greedy proof relies on monotonicity: once $u$ is extracted from $Q$, $d[u] = \delta(s, u)$ is finalized and cannot be improved. With non-negative edges, any alternative path from $s$ to $u$ must pass through some unfinalized vertex $y$ with $d[y] \ge d[u]$, and adding non-negative edges cannot reduce the path cost.
With **negative edges**, a path through $y$ could have subsequent negative edges, making its total cost strictly less than $d[u]$ *after* $u$ has already been finalized. Dijkstra will not reconsider $u$, producing an incorrect distance.

```text
Counterexample:
  S -> A (wt 4)
  S -> B (wt 2)
  B -> A (wt -3)

Dijkstra step 1: Extract S (d=0). Tentative d[A]=4, d[B]=2.
Dijkstra step 2: Extract B (d=2). Relax B->A: d[A] becomes min(4, 2 + (-3)) = -1.
If instead S->A had weight 1:
Dijkstra extracts A first (d=1, finalized!). Then extracts B (d=2), but cannot re-finalize A,
missing the true shorter path S -> B -> A of weight -1!
```

### Complexities across data structures

| Priority Queue Structure | Extract-Min cost ($V$ times) | Decrease-Key cost ($E$ times) | Total time complexity | Best used for |
|---|---|---|---|---|
| Unsorted array | $O(V)$ | $O(1)$ | $O(V^2 + E) = \mathbf{O(V^2)}$ | Dense graphs ($E = \Theta(V^2)$) |
| Binary min-heap | $O(\log V)$ | $O(\log V)$ | $O((V + E) \log V) = \mathbf{O(E \log V)}$ | Sparse graphs ($E = O(V)$) |
| Fibonacci heap | $O(\log V)$ amortized | $O(1)$ amortized | $\mathbf{O(V \log V + E)}$ | Theoretical bound |

### Worked example: Dijkstra execution trace
Consider the directed graph with vertices $\{S, A, B, C, D\}$ and edges:
$$(S, A): 4, \quad (S, B): 2, \quad (A, B): 1, \quad (A, C): 5, \quad (B, A): 3, \quad (B, C): 8, \quad (B, D): 10, \quad (C, D): 2$$
Source vertex is $S$.

| Step | Vertex extracted | Distance | Outgoing edges relaxed | Priority queue state $(v: d[v])$ | Finalized set $S$ |
|---|---|---|---|---|---|
| 0 | Start at $S$ | 0 | $(S, A) \to d[A] = 4$; $(S, B) \to d[B] = 2$ | $\{B: 2, A: 4, C: \infty, D: \infty\}$ | $\{S\}$ |
| 1 | Extract $B$ | 2 | $(B, A): 2+3=5 > 4$ (no change); $(B, C): 2+8=10$; $(B, D): 2+10=12$ | $\{A: 4, C: 10, D: 12\}$ | $\{S, B\}$ |
| 2 | Extract $A$ | 4 | $(A, B)$ already finalized; $(A, C): 4+5=9 < 10 \implies d[C] = 9$ | $\{C: 9, D: 12\}$ | $\{S, B, A\}$ |
| 3 | Extract $C$ | 9 | $(C, D): 9+2=11 < 12 \implies d[D] = 11$ | $\{D: 11\}$ | $\{S, B, A, C\}$ |
| 4 | Extract $D$ | 11 | No outgoing edges | $\emptyset$ | $\{S, B, A, C, D\}$ |

- **Extraction order:** $S \to B \to A \to C \to D$.
- **Shortest distances:** $d[S] = 0, \quad d[A] = 4, \quad d[B] = 2, \quad d[C] = 9, \quad d[D] = 11$.
- **Shortest path tree edges:** $(S, A), (S, B), (A, C), (C, D)$.

---

## 3. Bellman–Ford Algorithm (Negative Weights and Cycle Detection) [PYQ: Medium]

The Bellman–Ford algorithm solves the single-source shortest path problem on graphs with arbitrary edge weights and identifies reachable negative-weight cycles.

### Dynamic programming formulation
Let $d^{(k)}[v]$ be the weight of a shortest path from source $s$ to $v$ that uses at most $k$ edges:
$$d^{(k)}[v] = \min \left( d^{(k-1)}[v], \min_{(u, v) \in E} \{d^{(k-1)}[u] + w(u, v)\} \right)$$
Since a simple path contains at most $V - 1$ edges, the algorithm performs $V - 1$ passes of relaxing every edge in $E$.

### Negative-weight cycle detection
After $V - 1$ passes, perform a **$V$-th pass**:
- For each edge $(u, v) \in E$:
  - If $d[u] + w(u, v) < d[v]$:
    - **Report: Negative-weight cycle reachable from source exists.**
- If no distance decreases during the $V$-th pass, all computed distances are strictly optimal.
- **Time complexity:** $O(V \cdot E)$. Space complexity is $O(V)$.

### Worked example: Bellman–Ford trace
Consider vertices $\{S, A, B, C, D\}$ with edges:
$$(S, A): 6, \quad (S, B): 7, \quad (A, C): 5, \quad (B, C): -3, \quad (B, D): 9, \quad (C, A): -2, \quad (C, D): 7$$
Source is $S$. Vertices $V = 5 \implies$ run $V - 1 = 4$ passes.
We track synchronous passes where pass $k$ uses distances from pass $k - 1$:

| Pass | $d[S]$ | $d[A]$ | $d[B]$ | $d[C]$ | $d[D]$ | Edges that relaxed in this pass |
|---|---|---|---|---|---|---|
| Init | 0 | $\infty$ | $\infty$ | $\infty$ | $\infty$ | — |
| **Pass 1** | 0 | 6 | 7 | $\infty$ | $\infty$ | $(S, A) \to 6$; $(S, B) \to 7$ |
| **Pass 2** | 0 | 6 | 7 | **4** | **16** | $(B, C) \to 7 + (-3) = 4$; $(B, D) \to 7 + 9 = 16$ |
| **Pass 3** | 0 | **2** | 7 | 4 | **11** | $(C, A) \to 4 + (-2) = 2$; $(C, D) \to 4 + 7 = 11$ |
| **Pass 4** | 0 | 2 | 7 | 4 | 11 | No values change (converged early) |

- **5th pass check:** For edge $(C, A)$: $d[C] + w(C, A) = 4 + (-2) = 2 == d[A]$. No edge relaxes $\implies$ **No negative cycle**.
- **Final shortest distances:** $d[S] = 0, \quad d[A] = 2, \quad d[B] = 7, \quad d[C] = 4, \quad d[D] = 11$.

---

## 4. Floyd–Warshall Algorithm (All-Pairs Shortest Paths) [PYQ: Medium]

The Floyd–Warshall algorithm solves the **All-Pairs Shortest Paths (APSP)** problem for directed or undirected graphs with positive or negative edge weights (but no negative cycles) in $O(V^3)$ time.

### Recurrence relation
Number vertices $1, 2, \dots, n$. Let $D^{(k)}[i][j]$ be the shortest path distance from vertex $i$ to vertex $j$ such that all intermediate vertices belong to the subset $\{1, 2, \dots, k\}$.
$$D^{(k)}[i][j] = \min \left( D^{(k-1)}[i][j], \quad D^{(k-1)}[i][k] + D^{(k-1)}[k][j] \right)$$
- **Base case ($k = 0$):** $D^{(0)}[i][j] = w(i, j)$ if $(i, j) \in E$; $D^{(0)}[i][i] = 0$; $\infty$ otherwise.
- **Outer loop variable:** Variable $k$ **must** be the outermost loop:
  ```c
  for (int k = 1; k <= n; k++)
      for (int i = 1; i <= n; i++)
          for (int j = 1; j <= n; j++)
              D[i][j] = min(D[i][j], D[i][k] + D[k][j]);
  ```
- **Negative-cycle detection:** A negative-weight cycle exists **if and only if** any diagonal entry becomes negative: $\exists i: D^{(n)}[i][i] < 0$.
- **Space complexity:** $O(V^2)$ (can be updated in-place without separate matrices).

### Worked example: Full Floyd–Warshall trace
Consider 4 vertices $\{1, 2, 3, 4\}$ with edges:
$$(1, 3): -2, \quad (2, 1): 4, \quad (2, 3): 3, \quad (3, 4): 2, \quad (4, 2): -1$$

```text
D^(0) matrix:
       1     2     3     4
 1 [   0   inf    -2   inf ]
 2 [   4     0     3   inf ]
 3 [ inf   inf     0     2 ]
 4 [ inf    -1   inf     0 ]
```

- **Step $k = 1$ (Intermediate vertex 1):**
  Check paths through 1: $2 \to 1 \to 3$ has cost $D[2][1] + D[1][3] = 4 + (-2) = 2 < D[2][3]=3 \implies D^{(1)}[2][3] = 2$.
  ```text
  D^(1):
   1 [   0   inf    -2   inf ]
   2 [   4     0     2   inf ]
   3 [ inf   inf     0     2 ]
   4 [ inf    -1   inf     0 ]
  ```

- **Step $k = 2$ (Intermediate vertex 2):**
  Check paths through 2:
  $4 \to 2 \to 1$: $D[4][2] + D[2][1] = -1 + 4 = 3 \implies D^{(2)}[4][1] = 3$.
  $4 \to 2 \to 3$: $D[4][2] + D[2][3] = -1 + 2 = 1 \implies D^{(2)}[4][3] = 1$.
  ```text
  D^(2):
   1 [   0   inf    -2   inf ]
   2 [   4     0     2   inf ]
   3 [ inf   inf     0     2 ]
   4 [   3    -1     1     0 ]
  ```

- **Step $k = 3$ (Intermediate vertex 3):**
  Check paths through 3:
  $1 \to 3 \to 4$: $D[1][3] + D[3][4] = -2 + 2 = 0 \implies D^{(3)}[1][4] = 0$.
  $2 \to 3 \to 4$: $D[2][3] + D[3][4] = 2 + 2 = 4 \implies D^{(3)}[2][4] = 4$.
  ```text
  D^(3):
   1 [   0   inf    -2     0 ]
   2 [   4     0     2     4 ]
   3 [ inf   inf     0     2 ]
   4 [   3    -1     1     0 ]
  ```

- **Step $k = 4$ (Intermediate vertex 4):**
  Check paths through 4:
  $1 \to 4 \to 2$: $D[1][4] + D[4][2] = 0 + (-1) = -1 \implies D^{(4)}[1][2] = -1$.
  $3 \to 4 \to 1$: $D[3][4] + D[4][1] = 2 + 3 = 5 \implies D^{(4)}[3][1] = 5$.
  $3 \to 4 \to 2$: $D[3][4] + D[4][2] = 2 + (-1) = 1 \implies D^{(4)}[3][2] = 1$.
  ```text
  D^(4) (Final):
   1 [   0    -1    -2     0 ]
   2 [   4     0     2     4 ]
   3 [   5     1     0     2 ]
   4 [   3    -1     1     0 ]
  ```
All diagonal entries $D[i][i] == 0 \implies$ No negative cycles exist.

---

## Formulas and facts to memorise

| Algorithm | Type | Weights allowed? | Negative cycles? | Time complexity | Space complexity |
|---|---|---|---|---|---|
| **BFS** | Single-Source | Unweighted / Unit weights only | No | $O(V + E)$ | $O(V)$ |
| **DAG Shortest Path** | Single-Source | Arbitrary (positive and negative) | Acyclic by definition | $O(V + E)$ | $O(V)$ |
| **Dijkstra (Array)** | Single-Source | Non-negative ($w \ge 0$) | Fails | $O(V^2)$ | $O(V)$ |
| **Dijkstra (Heap)** | Single-Source | Non-negative ($w \ge 0$) | Fails | $O(E \log V)$ | $O(V)$ |
| **Dijkstra (Fibonacci)** | Single-Source | Non-negative ($w \ge 0$) | Fails | $O(V \log V + E)$ | $O(V)$ |
| **Bellman–Ford** | Single-Source | Arbitrary ($w \in \mathbb{R}$) | Detects reachable | $O(V \cdot E)$ | $O(V)$ |
| **Floyd–Warshall** | All-Pairs | Arbitrary ($w \in \mathbb{R}$) | Detects ($D[i][i] < 0$) | $O(V^3)$ | $O(V^2)$ |
| **Transitive Closure** | All-Pairs Reachability | Unweighted directed | N/A | $O(V^3)$ bitwise | $O(V^2)$ |

---

## GATE traps

- **Dijkstra with negative edges:** Never apply Dijkstra if the graph has even a single negative edge. Even if there are no negative cycles, Dijkstra can output incorrect shortest path distances.
- **Negative edge $\ne$ Negative cycle:** A negative edge does *not* imply an undefined shortest path. Shortest path is $-\infty$ **only if** a cycle exists whose net sum of edge weights is strictly negative, and that cycle is on a path from source to destination.
- **Floyd–Warshall loop ordering:** The outer loop must be the intermediate vertex index $k$. If you write $i$ or $j$ as the outer loop, the algorithm computes only a subset of path combinations, giving wrong answers.
- **Bellman–Ford disconnected negative cycles:** Bellman–Ford detects negative cycles reachable from the chosen source $s$. To detect arbitrary negative cycles anywhere in the graph, connect a virtual super-source $S_0$ with directed 0-weight edges to all vertices.
- **SPT vs. MST weight shift:** Adding a positive constant $c$ to every edge in graph $G$ leaves the Minimum Spanning Tree invariant, but it **can change the shortest path**. (A path with 3 edges gets penalized by $3c$, while a path with 1 edge gets penalized by only $1c$).

---

## Connections

- [Graph traversals](graph-traversals.md) — BFS is Dijkstra's algorithm specialized to unit-weight edges; topological sort enables $O(V + E)$ DAG shortest paths.
- [Dynamic programming](dynamic-programming.md) — Floyd–Warshall and Bellman–Ford are canonical examples of dynamic programming over graph structures.
- [Heaps and priority queues](../07-data-structures/heaps.md) — binary min-heap operation underlies the $O(E \log V)$ implementation of Dijkstra.
- [Network routing](../15-computer-networks/routing.md) — OSPF uses Dijkstra's link-state algorithm; RIP uses the Bellman–Ford distance-vector algorithm.

---

## Practice

**Q1. (NAT, Easy)** Consider a directed graph $G$ with vertices $\{S, A, B, C\}$ and edge weights:
$$(S, A): 1, \quad (S, B): 4, \quad (A, B): 2, \quad (A, C): 6, \quad (B, C): 3$$
The length of the shortest path from $S$ to $C$ using Dijkstra's algorithm is _______.

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:**  
1. Start at $S$: $d[S] = 0, d[A] = \infty, d[B] = \infty, d[C] = \infty$.
2. Extract $S$ (0): Relax $(S, A) \to d[A] = 1$; $(S, B) \to d[B] = 4$.
3. Extract $A$ (1): Relax $(A, B) \to d[B] = \min(4, 1 + 2) = 3$; $(A, C) \to d[C] = 1 + 6 = 7$.
4. Extract $B$ (3): Relax $(B, C) \to d[C] = \min(7, 3 + 3) = 6$.
5. Extract $C$ (6): Final distance $d[C] = 6$ along path $S \to A \to B \to C$.
</details>

---

**Q2. (MCQ, Easy)** What is the tightest worst-case time complexity of finding the single-source shortest path in a Directed Acyclic Graph (DAG) with $V$ vertices, $E$ edges, and arbitrary (positive or negative) edge weights?
(A) $O(V \cdot E)$  
(B) $O(E \log V)$  
(C) $O(V + E)$  
(D) $O(V^2)$  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**  
In a DAG, vertices can be topologically sorted in $O(V + E)$ time using DFS or Kahn's algorithm. Relaxing outgoing edges of each vertex in topological order guarantees that when vertex $u$ is processed, its shortest path distance is already finalized. Total time is $O(V + E)$, and this algorithm correctly handles negative edge weights.
</details>

---

**Q3. (MCQ, Medium)** In Dijkstra's algorithm executed on a directed weighted graph with non-negative edge weights starting from source $s$, in what order are vertices finalized?
(A) In order of increasing number of edges from $s$  
(B) In non-decreasing order of their true shortest path distances from $s$  
(C) In reverse topological order  
(D) In alphabetical order of vertex labels  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
Dijkstra's algorithm greedily extracts and finalizes the unvisited vertex with the minimum tentative distance. Because all edge weights are non-negative, this tentative distance is guaranteed to be the exact shortest distance $\delta(s, u)$, ensuring vertices are finalized in monotonically non-decreasing order of their true shortest distances from $s$.
</details>

---

**Q4. (NAT, Medium)** Consider the Bellman–Ford algorithm running on a graph with vertices $\{1, 2, 3, 4\}$ and source vertex 1. Directed edge weights are:
$$(1, 2): 3, \quad (1, 3): 2, \quad (2, 4): 1, \quad (3, 2): -2, \quad (3, 4): 4$$
Assuming synchronous relaxation passes where each pass uses distances from the previous pass, the tentative distance to vertex 4 after exactly 2 passes, $d^{(2)}[4]$, is _______.

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:**  
- **Initial:** $d[1] = 0, d[2] = \infty, d[3] = \infty, d[4] = \infty$.
- **Pass 1:** Only outgoing edges from source 1 can relax:
  $d^{(1)}[2] = 0 + 3 = 3$.
  $d^{(1)}[3] = 0 + 2 = 2$.
  $d^{(1)}[4] = \infty$.
- **Pass 2:** Uses distances from pass 1:
  $d^{(2)}[2] = \min(3, d^{(1)}[3] + (-2)) = \min(3, 2 - 2) = 0$.
  $d^{(2)}[4] = \min(\infty, d^{(1)}[2] + 1, d^{(1)}[3] + 4) = \min(\infty, 3 + 1, 2 + 4) = \min(4, 6) = 4$.
Thus, $d^{(2)}[4] = 4$. (Note: in Pass 3, $d^{(3)}[4]$ improves to $0 + 1 = 1$ via path $1 \to 3 \to 2 \to 4$).
</details>

---

**Q5. (NAT, Medium)** In the Floyd–Warshall algorithm on a graph with vertices $\{1, 2, 3\}$, the initial distance matrix $D^{(0)}$ is:
$$D^{(0)} = \begin{bmatrix} 0 & 4 & 11 \\ 6 & 0 & 2 \\ 3 & \infty & 0 \end{bmatrix}$$
The value of $D^{(2)}[1][3]$ (the shortest distance from 1 to 3 with permitted intermediate vertices $\{1, 2\}$) is _______.

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:**  
- For $k = 1$:
  $D^{(1)}[1][3] = \min(D^{(0)}[1][3], D^{(0)}[1][1] + D^{(0)}[1][3]) = \min(11, 0 + 11) = 11$.
  $D^{(1)}[1][2] = D^{(0)}[1][2] = 4$, $D^{(1)}[2][3] = D^{(0)}[2][3] = 2$.
- For $k = 2$:
  $$D^{(2)}[1][3] = \min \left( D^{(1)}[1][3], D^{(1)}[1][2] + D^{(1)}[2][3] \right) = \min(11, 4 + 2) = \min(11, 6) = 6$$
The path is $1 \to 2 \to 3$ with cost $4 + 2 = 6$.
</details>

---

**Q6. (MSQ, Medium)** Which of the following statements are **TRUE** regarding shortest path algorithms?
(A) If all edge weights in a graph are positive, adding a constant $c > 0$ to every edge weight may change the shortest path between a pair of vertices.  
(B) If a graph has negative edge weights but no negative cycles, Bellman–Ford is guaranteed to find shortest path distances from the source.  
(C) In Floyd–Warshall, if $D[i][i] < 0$ for some vertex $i$ after the algorithm completes, the graph contains a negative-weight cycle.  
(D) Dijkstra's algorithm is guaranteed to run correctly if all negative-weight edges are directed towards the source vertex.  

<details><summary>Answer</summary>

**Answer:** (A), (B), (C)  
**Solution:**  
- **(A) True:** Consider two paths from $u$ to $v$: path $P_1$ with 1 edge of weight 10, and path $P_2$ with 3 edges of weights 2, 2, 2 (cost 6). Initially $P_2$ is shorter ($6 < 10$). If $c = 5$ is added to all edges: $w'(P_1) = 10 + 5 = 15$, while $w'(P_2) = 6 + 3(5) = 21$. Now $P_1$ is shorter ($15 < 21$).
- **(B) True:** Bellman–Ford solves the single-source shortest path problem for arbitrary edge weights as long as no negative cycle is reachable.
- **(C) True:** A negative diagonal entry $D[i][i] < 0$ indicates a cycle from $i$ to $i$ with strictly negative total cost.
- **(D) False:** Any negative edge can violate the greedy non-decreasing assumption of Dijkstra regardless of orientation.
</details>

---

**Q7. (MCQ, Hard)** Let $G = (V, E)$ be a directed graph with edge weights $w: E \to \mathbb{R}$. We want to find the shortest path from $s$ to $t$. A student proposes modifying all edge weights by setting $w'(u, v) = w(u, v) + |w_{min}|$, where $w_{min}$ is the most negative edge weight in $G$, so that all edge weights become non-negative, and then running Dijkstra's algorithm. This proposal is:
(A) Correct, because all edge weights become non-negative and Dijkstra runs in $O(E \log V)$  
(B) Incorrect, because paths with more edges are penalized more heavily, which can favor paths with fewer edges over the true shortest path  
(C) Correct, because adding a constant preserves the relative order of all path weights  
(D) Incorrect, because Dijkstra cannot handle graphs where the minimum weight is 0  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
Adding a constant $C = |w_{min}|$ to every edge increases the weight of a path $p$ with $k$ edges by $k \cdot C$. A path with more edges receives a larger total increase than a path with fewer edges. Consequently, Dijkstra on the shifted graph will bias towards paths with fewer edges rather than the path with the minimum original weight. (Johnson's algorithm uses vertex potentials $h(u)$, not uniform constants, to reweight edges correctly).
</details>
