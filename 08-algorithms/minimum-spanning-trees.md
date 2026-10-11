# Minimum Spanning Trees

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Minimum spanning tree, Kruskal's algorithm, Prim's algorithm, cut property, cycle property, MST uniqueness
> **Prerequisites:** [Graph theory](../01-discrete-mathematics/graph-theory.md) · [Greedy algorithms](greedy-algorithms.md) · [Heaps and priority queues](../07-data-structures/heaps.md) · **Leads to:** [Shortest paths](shortest-paths.md) · [Network routing](../15-computer-networks/routing.md)
> **PYQ weightage:** High · 22 Qs since 2015 (9 since 2021) · DA: 0 · Trend: steady · Most asked: Kruskal's algorithm execution, cut and cycle properties, MST uniqueness, edge-weight modifications

## Quick glance

- **Exam focus:** Tracing Kruskal vs Prim on weighted graphs (NAT/MCQ), counting distinct MSTs when edge weights tie (NAT), evaluating edge-weight modifications (MSQ), and finding threshold edge weights for MST inclusion (NAT).
- An **MST** connects all $V$ vertices of a connected, undirected, edge-weighted graph using exactly $V - 1$ edges with minimum total weight $\sum_{e \in T} w(e)$.
- **Distinct weights theorem:** If all edge weights in a connected undirected graph are **distinct**, the graph has a **strictly unique MST**. The converse is false: graphs with duplicate weights can still have a unique MST.
- **Cut property:** For any cut $(S, V \setminus S)$, the lightest edge crossing the cut belongs to some MST. If the lightest crossing edge is strictly unique, it belongs to **every** MST.
- **Cycle property:** In any simple cycle $C$, the edge with strictly maximum weight **cannot belong to any MST**.
- **Kruskal's algorithm:** Sorts edges by weight in ascending order and greedily adds an edge if it connects two disjoint components using **Disjoint-Set Union (DSU)**. Time: $O(E \log E) = O(E \log V)$.
- **Prim's algorithm:** Grows a single connected tree from an arbitrary root vertex, repeatedly adding the cheapest edge crossing the cut between tree and non-tree vertices. Time: $O(V^2)$ with matrix/array; $O(E \log V)$ with binary heap; $O(E + V \log V)$ with Fibonacci heap.
- **Weight shifts:** Adding a constant $c$ to every edge ($w' = w + c$) or multiplying by $c > 0$ or squaring non-negative weights **never changes the set of edges in the MST** (total weight becomes $W + (V - 1)c$ or $cW$).
- Shortest path trees (SPT) and MSTs are fundamentally different: an MST minimizes total edge cost, while an SPT minimizes root-to-vertex distances. Adding $c$ to all edges preserves MST, but **can change the shortest path**.

## 1. Spanning Tree Properties, Cut and Cycle Theorems [PYQ: High]

A spanning tree $T$ of a connected undirected graph $G = (V, E)$ is a subgraph that is a tree and includes every vertex of $V$. If $|V| = n$, every spanning tree contains exactly $n - 1$ edges and no cycles.

### The Cut Property
Let $S \subset V$ be any subset of vertices, defining a cut $(S, V \setminus S)$. An edge crosses the cut if one endpoint is in $S$ and the other in $V \setminus S$.
- **Theorem:** If edge $e = (u, v)$ is a minimum-weight edge crossing cut $(S, V \setminus S)$, then $e$ is safe for some MST of $G$.
- **Uniqueness corollary:** If $e$ is the *strictly unique* lightest edge crossing the cut, $e$ must be present in **every** MST of $G$.

### The Cycle Property
Let $C$ be any simple cycle in $G$.
- **Theorem:** If edge $e$ is the strictly heaviest edge on cycle $C$, then $e$ cannot belong to any MST of $G$.
- **Proof intuition:** If an MST $T$ contained $e$, removing $e$ would split $T$ into two disconnected components $S$ and $V \setminus S$. Cycle $C$ must contain at least one other edge $e'$ crossing this cut. Since $w(e') < w(e)$, swapping $e$ for $e'$ creates a new spanning tree $T' = T \setminus \{e\} \cup \{e'\}$ with strictly smaller total weight, contradicting the optimality of $T$.

### Invariance under Monotonic Transformations
Let $T$ be an MST of graph $G$ with edge weight function $w$.
1. **Uniform addition:** $w'(e) = w(e) + c$ for constant $c \in \mathbb{R}$. Any spanning tree has exactly $V - 1$ edges, so total weight of every spanning tree changes by exactly $(V - 1)c$. The relative order of tree costs is preserved $\implies \mathbf{MST\text{ edges remain identical}}$.
2. **Positive scaling:** $w'(e) = c \cdot w(e)$ with $c > 0 \implies \mathbf{MST\text{ edges remain identical}}$.
3. **Monotonic functions:** If $f(x)$ is strictly monotonically increasing (e.g., $f(x) = x^2$ for $x \ge 0$, or $f(x) = 2^x$), the greedy comparison decisions in Kruskal and Prim are completely unchanged $\implies \mathbf{MST\text{ edges remain identical}}$.

---

## 2. Kruskal's Algorithm and Disjoint-Set Union (DSU) [PYQ: High]

Kruskal's algorithm adopts a global greedy strategy: it repeatedly picks the cheapest available edge anywhere in the graph, provided adding it does not create a cycle.

### Algorithm mechanics
1. Sort all edges $E$ in non-decreasing order of weight: $w(e_1) \le w(e_2) \le \dots \le w(e_m)$.
2. Initialize a Disjoint-Set Union (DSU) structure where each vertex $v \in V$ is in its own singleton set.
3. Initialize empty tree edge set $T = \emptyset$.
4. For each edge $e = (u, v)$ in sorted order:
   - If $\text{Find}(u) \ne \text{Find}(v)$:
     - Add $e$ to $T$.
     - $\text{Union}(u, v)$.
     - If $|T| == |V| - 1$, terminate early.
5. Return $T$.

- **Time complexity:** Sorting takes $O(E \log E) = O(E \log V)$ (since $E \le V^2$, $\log E \le 2 \log V$). DSU operations with path compression and union by rank take $O(E \cdot \alpha(V))$ time, where $\alpha$ is the inverse Ackermann function ($\alpha(V) \le 4$ for all practical inputs). Sorting dominates, so total time is **$O(E \log V)$**.
- **Space complexity:** $O(V)$ auxiliary space for DSU parent and rank arrays.

### Worked example: Kruskal execution trace
Consider the weighted undirected graph with vertices $\{A, B, C, D, E, F\}$ and 9 edges:
$$(B, C): 1, \quad (A, C): 2, \quad (D, E): 2, \quad (E, F): 3, \quad (A, B): 4$$
$$(B, D): 5, \quad (D, F): 6, \quad (C, D): 8, \quad (C, E): 10$$

```text
Sorted edge list:
1. (B, C) wt 1    4. (E, F) wt 3    7. (D, F) wt 6
2. (A, C) wt 2    5. (A, B) wt 4    8. (C, D) wt 8
3. (D, E) wt 2    6. (B, D) wt 5    9. (C, E) wt 10
```

**Step-by-step Kruskal trace:**

| Step | Edge inspected | Weight | Component of endpoints | Action | Disjoint components after step |
|---|---|---|---|---|---|
| 1 | $(B, C)$ | 1 | $\text{Find}(B) \ne \text{Find}(C)$ | **Accept** | $\{B, C\}, \{A\}, \{D\}, \{E\}, \{F\}$ |
| 2 | $(A, C)$ | 2 | $\text{Find}(A) \ne \text{Find}(C)$ | **Accept** | $\{A, B, C\}, \{D\}, \{E\}, \{F\}$ |
| 3 | $(D, E)$ | 2 | $\text{Find}(D) \ne \text{Find}(E)$ | **Accept** | $\{A, B, C\}, \{D, E\}, \{F\}$ |
| 4 | $(E, F)$ | 3 | $\text{Find}(E) \ne \text{Find}(F)$ | **Accept** | $\{A, B, C\}, \{D, E, F\}$ |
| 5 | $(A, B)$ | 4 | $\text{Find}(A) == \text{Find}(B)$ | **REJECT (Cycle $A-C-B-A$)** | $\{A, B, C\}, \{D, E, F\}$ |
| 6 | $(B, D)$ | 5 | $\text{Find}(B) \ne \text{Find}(D)$ | **Accept** | $\{A, B, C, D, E, F\}$ |

We have accepted $|V| - 1 = 6 - 1 = 5$ edges. Algorithm terminates.
- **Edges in MST:** $\{(B, C), (A, C), (D, E), (E, F), (B, D)\}$.
- **Total MST weight:** $1 + 2 + 2 + 3 + 5 = \mathbf{13}$.
- Remaining edges $(D, F)$, $(C, D)$, $(C, E)$ are never needed.

---

## 3. Prim's Algorithm and Priority Queue Implementations [PYQ: High]

Prim's algorithm adopts a local greedy strategy: it starts at an arbitrary root vertex and grows a single tree one edge at a time, always selecting the lightest edge that connects a vertex in the tree to a vertex outside.

### Algorithm mechanics
1. For each $v \in V$, set $\text{key}[v] = \infty$, $\text{parent}[v] = \text{NIL}$, $\text{inMST}[v] = \text{false}$.
2. Choose arbitrary root vertex $r \in V$; set $\text{key}[r] = 0$.
3. Insert all vertices into a priority queue $Q$ keyed by $\text{key}[v]$.
4. While $Q$ is not empty:
   - Extract vertex $u$ with minimum key value from $Q$; set $\text{inMST}[u] = \text{true}$.
   - For each adjacent neighbor $v \in \text{Adj}[u]$:
     - If $\text{inMST}[v] == \text{false}$ and $w(u, v) < \text{key}[v]$:
       - Set $\text{key}[v] = w(u, v)$ and $\text{parent}[v] = u$ (decrease-key operation in $Q$).
5. Return tree formed by edges $\{(\text{parent}[v], v) \mid v \in V \setminus \{r\}\}$.

### Implementation complexities in GATE

| Implementation | Extract-Min cost | Decrease-Key cost | Total time complexity | Best suited for |
|---|---|---|---|---|
| Adjacency matrix + linear array scan | $O(V)$ | $O(1)$ | $O(V^2 + E) = \mathbf{O(V^2)}$ | **Dense graphs** ($E = \Theta(V^2)$) |
| Adjacency list + Binary min-heap | $O(\log V)$ | $O(\log V)$ | $O(V \log V + E \log V) = \mathbf{O(E \log V)}$ | **Sparse graphs** ($E = O(V)$) |
| Adjacency list + Fibonacci heap | $O(\log V)$ amortized | $O(1)$ amortized | $\mathbf{O(E + V \log V)}$ | Theoretical optimum |

### Worked example: Prim execution trace
Using the **same 6-vertex graph** from Section 2, run Prim's algorithm starting at root vertex $A$.

```text
Vertices: A, B, C, D, E, F.
Root: A. Initial keys: key[A]=0, all others = infinity.
```

**Step-by-step trace:**

| Step | Vertex extracted | Key value | Tree edge added | Neighbors relaxed | Key array state $[\text{key}_A, \text{key}_B, \text{key}_C, \text{key}_D, \text{key}_E, \text{key}_F]$ |
|---|---|---|---|---|---|
| 0 | Start at $A$ | 0 | None (root) | $B$: $\min(\infty, 4) = 4$, $C$: $\min(\infty, 2) = 2$ | $[0, 4, 2, \infty, \infty, \infty]$ |
| 1 | Extract $C$ | 2 | $(A, C)$ wt 2 | $B$: $\min(4, 1) = 1$, $D$: $\min(\infty, 8) = 8$, $E$: $\min(\infty, 10) = 10$ | $[0, 1, 2, 8, 10, \infty]$ |
| 2 | Extract $B$ | 1 | $(C, B)$ wt 1 | $D$: $\min(8, 5) = 5$ | $[0, 1, 2, 5, 10, \infty]$ |
| 3 | Extract $D$ | 5 | $(B, D)$ wt 5 | $E$: $\min(10, 2) = 2$, $F$: $\min(\infty, 6) = 6$ | $[0, 1, 2, 5, 2, 6]$ |
| 4 | Extract $E$ | 2 | $(D, E)$ wt 2 | $F$: $\min(6, 3) = 3$ | $[0, 1, 2, 5, 2, 3]$ |
| 5 | Extract $F$ | 3 | $(E, F)$ wt 3 | None unvisited | $[0, 1, 2, 5, 2, 3]$ |

- **Extraction order:** $A \to C \to B \to D \to E \to F$.
- **Edges added to MST:** $(A, C)$ wt 2, $(C, B)$ wt 1, $(B, D)$ wt 5, $(D, E)$ wt 2, $(E, F)$ wt 3.
- **Total MST weight:** $2 + 1 + 5 + 2 + 3 = \mathbf{13}$ (matches Kruskal exactly).

---

## 4. Counting MSTs, Second-Best MST, and Edge Modifications [PYQ: High]

### Counting distinct MSTs with tie-breaking
When multiple edges have identical weights, Kruskal may have choices. Key theorem:
- For any graph $G$, the **number of edges of each weight $w_k$ included in any MST is invariant**.
- If a group of identical-weight edges forms independent cycles or choices, we multiply the number of valid cycle-free choices across weight classes.

### Worked example: Counting distinct MSTs
Consider a 4-cycle $C_4$ on vertices $\{1, 2, 3, 4\}$ with edge weights:
$$w(1, 2) = 2, \quad w(2, 3) = 2, \quad w(3, 4) = 3, \quad w(4, 1) = 3$$
- Kruskal sorts edges: weight 2 edges $\{(1, 2), (2, 3)\}$, weight 3 edges $\{(3, 4), (4, 1)\}$.
- Weight 2 edges: both edges $(1, 2)$ and $(2, 3)$ are selected (they do not form a cycle). Components: $\{1, 2, 3\}$ and $\{4\}$. Total: 2 edges chosen.
- Weight 3 edges: We need 1 more edge to connect vertex 4 to $\{1, 2, 3\}$.
  - Choosing $(3, 4)$ connects $4$ to $3$ (valid tree: edges $(1, 2), (2, 3), (3, 4)$).
  - Choosing $(4, 1)$ connects $4$ to $1$ (valid tree: edges $(1, 2), (2, 3), (4, 1)$).
- Both produce valid spanning trees of weight $2 + 2 + 3 = 7$.
- **Number of distinct MSTs = 2**.

### Constrained edge inclusion (Threshold weight $x$)
A frequent GATE numerical asks: "Given edge $e = (u, v)$ with weight $x$, what is the maximum value of $x$ such that $e$ is guaranteed to be in **every** MST?"
- **Rule:** By the cycle property, $e = (u, v)$ is in an MST iff its weight does not strictly exceed the maximum edge weight on the alternative tree path connecting $u$ and $v$ in the MST formed by the remaining edges.
- Let $M$ be the maximum edge weight on the path between $u$ and $v$ in the MST without $e$.
  - If $x < M$: $e$ is strictly lighter than $M$, so $M$ is eliminated from the cycle and $e$ is in **every** MST.
  - If $x = M$: $e$ can tie with $M$; $e$ is in *some* MST, but not necessarily *every* MST.
  - To be in **every** MST with integer weights, $x \le M - 1$.

### Second-Best MST
The second-best MST $T_2$ differs from the optimal MST $T$ by exactly one edge swap:
$$w(T_2) = \min_{e' \notin T} \left( w(T) + w(e') - \max_{e \in \text{path}(u, v) \text{ in } T} w(e) \right)$$
Adding non-tree edge $e' = (u, v)$ creates a unique cycle in $T$. Removing the heaviest tree edge on that cycle restores a valid spanning tree. Testing all $E - (V - 1)$ non-tree edges takes $O(V^2)$ time.

---

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Number of edges in MST | Exactly $V - 1$ edges | Spanning tree sanity check |
| Kruskal's time complexity | $O(E \log V)$ (DSU + sorting) | Sparse graphs, edge-centric implementation |
| Prim's time (Adjacency matrix) | $O(V^2)$ | Dense graphs ($E = \Theta(V^2)$) |
| Prim's time (Binary heap) | $O(E \log V)$ | Sparse graphs ($E = O(V)$) |
| Prim's time (Fibonacci heap) | $O(E + V \log V)$ | Dense/intermediate theoretical bound |
| MST uniqueness criterion | All edge weights distinct $\implies$ strictly unique MST | Checking uniqueness of MST |
| Cut property | Lightest crossing edge is in some MST | Proving greedy correctness, edge inclusion |
| Cycle property | Strictly heaviest edge in any cycle is in NO MST | Cycle elimination, second-best MST |
| Weight shift $w' = w + c$ | MST edges unchanged; total cost $= W + (V - 1)c$ | Evaluating additive transformations |
| Positive scaling $w' = c \cdot w$ | MST edges unchanged; total cost $= c \cdot W$ | Evaluating multiplicative scaling |
| Monotonic function $w' = f(w)$ | MST edges unchanged ($f$ strictly increasing) | Squaring or exponentiating edge weights |
| Minimum Spanning Forest | $V - k$ edges for $k$ connected components | Disconnected graphs |

---

## GATE traps

- **Distinct weights converse trap:** Distinct edge weights imply a unique MST, but a unique MST does **not** imply distinct edge weights. (Example: a star graph where all edges have weight 1 has only one spanning tree, hence a unique MST despite non-distinct weights).
- **Shortest path vs. MST:** The shortest path between two vertices $u$ and $v$ does **not** necessarily use edges from the MST. Conversely, the tree formed by Dijkstra's algorithm (Shortest Path Tree) does **not** necessarily minimize total edge weight.
- **Additive weight shift on shortest paths:** Adding constant $c > 0$ to every edge preserves the MST, but it **can change the shortest path** between vertices because paths with more edges are penalized more by the addition.
- **Negative edge weights in MST:** Prim and Kruskal work correctly with **negative edge weights**, provided the graph is undirected. Negative weights do *not* break MST algorithms (unlike Dijkstra's shortest path algorithm).
- **Cycle tie-breaking:** The cycle property states that the *strictly heaviest* edge is excluded. If two or more edges tie for maximum weight in a cycle, one or more of them may still appear in an MST.

---

## Connections

- [Graph theory](../01-discrete-mathematics/graph-theory.md) — Cayley's formula ($n^{n-2}$ spanning trees for $K_n$), tree definitions, and cuts.
- [Greedy algorithms](greedy-algorithms.md) — matroid theory and cut/cycle greedy choice property.
- [Heaps and priority queues](../07-data-structures/heaps.md) — binary heaps and decrease-key operations underlying Prim's algorithm.
- [Shortest paths](shortest-paths.md) — contrasting total tree weight minimization (MST) with source-to-vertex path minimization (SPT).

---

## Practice

**Q1. (NAT, Easy)** A connected undirected graph $G$ has 15 vertices and 25 edges. The number of edges in any minimum spanning tree of $G$ is _______.

<details><summary>Answer</summary>

**Answer:** 14  
**Solution:**  
Any spanning tree of a connected graph with $V$ vertices contains exactly $V - 1$ edges. Here $V = 15$, so any spanning tree (including the MST) has $15 - 1 = 14$ edges.
</details>

---

**Q2. (MCQ, Easy)** For a dense connected undirected graph with $V$ vertices and $E = \Theta(V^2)$ edges, which implementation of Prim's algorithm achieves the best asymptotic time complexity?
(A) Prim using an adjacency list and binary heap: $O(V^2 \log V)$  
(B) Prim using an adjacency matrix and linear array search: $O(V^2)$  
(C) Kruskal using a binary heap: $O(V^2 \log V)$  
(D) Prim using a Fibonacci heap: $O(V^3)$  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
When $E = \Theta(V^2)$, the binary heap implementation takes $O(E \log V) = O(V^2 \log V)$ time. Prim's algorithm using an adjacency matrix and linear scanning to find the minimum key takes $O(V^2 + E) = O(V^2)$ time, which is faster by a factor of $\log V$ and asymptotically optimal for dense graphs.
</details>

---

**Q3. (NAT, Medium)** Consider the following undirected graph with vertices $\{A, B, C, D, E\}$ and edge weights:
$$(A, B): 3, \quad (A, C): 5, \quad (B, C): 2, \quad (B, D): 7, \quad (C, D): 4, \quad (C, E): 6, \quad (D, E): 1$$
The total weight of the Minimum Spanning Tree of this graph is _______.

<details><summary>Answer</summary>

**Answer:** 10  
**Solution:**  
Sort all edges in ascending order of weight:
1. $(D, E): 1$ $\to$ Add $(D, E)$. Components: $\{D, E\}, \{A\}, \{B\}, \{C\}$.
2. $(B, C): 2$ $\to$ Add $(B, C)$. Components: $\{D, E\}, \{B, C\}, \{A\}$.
3. $(A, B): 3$ $\to$ Add $(A, B)$. Components: $\{D, E\}, \{A, B, C\}$.
4. $(C, D): 4$ $\to$ Add $(C, D)$. Merges $\{A, B, C\}$ and $\{D, E\}$. All 5 vertices connected.
Total edges added $= 4 = V - 1$.
Total MST weight $= 1 + 2 + 3 + 4 = 10$.
</details>

---

**Q4. (NAT, Medium)** Consider a weighted undirected graph $G = (V, E)$ containing a cycle with edges $e_1, e_2, e_3, e_4$ having weights $4, 7, 9, x$ respectively. All other edges in $G$ have distinct weights greater than 20. The maximum integer value of $x$ such that edge $e_4$ is guaranteed to belong to **every** Minimum Spanning Tree of $G$ is _______.

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:**  
On the cycle formed by $e_1, e_2, e_3, e_4$, the weights are $4, 7, 9, x$.
By the cycle property, the strictly heaviest edge in any simple cycle cannot belong to any MST.
The maximum weight among the other three cycle edges is 9 (edge $e_3$).
- If $x < 9$, $e_3$ (weight 9) is the strictly heaviest edge on the cycle and must be excluded, forcing $e_4$ to be included in every MST.
- If $x = 9$, both $e_3$ and $e_4$ tie for the maximum weight on the cycle; one can be excluded while the other is included, so $e_4$ is not guaranteed to be in *every* MST.
- For integer weights, the maximum value of $x$ ensuring strict inclusion is $9 - 1 = 8$.
</details>

---

**Q5. (MSQ, Medium)** Let $G$ be a connected weighted undirected graph with $|V| \ge 3$. Which of the following statements are **TRUE**?
(A) If all edge weights in $G$ are distinct, then $G$ has a unique MST.  
(B) If the lightest edge $e$ in $G$ is unique, $e$ must be present in every MST of $G$.  
(C) If every edge weight is replaced by its square, the edges belonging to the MST remain unchanged.  
(D) If an edge $e$ belongs to some MST of $G$, then $e$ must be the lightest edge crossing some cut in $G$.  

<details><summary>Answer</summary>

**Answer:** (A), (B), (C), (D)  
**Solution:**  
- **(A) True:** Distinct edge weights guarantee a unique MST.
- **(B) True:** Consider the cut separating one endpoint of the lightest edge from the rest of the graph; $e$ is the strictly lightest edge crossing this cut, so by the cut property it belongs to every MST.
- **(C) True:** The function $f(w) = w^2$ is strictly increasing for non-negative weights, preserving the relative order of all edge comparisons.
- **(D) True:** By the cut theorem, an edge belongs to an MST if and only if it is a minimum-weight edge crossing some cut (specifically, the cut formed by removing $e$ from the MST).
</details>

---

**Q6. (NAT, Hard)** Consider a connected undirected graph $G$ with 7 vertices consisting of two squares (4-cycles) that share a single common vertex $C$: square 1 has vertices $\{A, B, C, D\}$ and square 2 has vertices $\{C, E, F, G\}$. All 4 edges in square 1 have weight 2. All 4 edges in square 2 have weight 5. The number of distinct Minimum Spanning Trees of $G$ is _______.

<details><summary>Answer</summary>

**Answer:** 16  
**Solution:**  
The graph has $|V| = 7$ vertices and 8 edges. An MST must contain $7 - 1 = 6$ edges.
- Square 1 has 4 edges of weight 2 forming a cycle. To span the 4 vertices $\{A, B, C, D\}$ without forming a cycle, we must pick any 3 edges out of the 4 edges. Number of ways $= \binom{4}{3} = 4$.
- Square 2 has 4 edges of weight 5 forming a cycle. To span the vertices $\{C, E, F, G\}$ without forming a cycle, we must pick any 3 edges out of the 4 edges. Number of ways $= \binom{4}{3} = 4$.
- The two squares share only vertex $C$, so the choice of spanning tree in square 1 is completely independent of the choice in square 2.
- Total distinct MSTs $= 4 \times 4 = 16$. Total weight of each MST is $3(2) + 3(5) = 6 + 15 = 21$.
</details>

---

**Q7. (MCQ, Hard)** Let $T$ be an MST of a connected undirected graph $G = (V, E)$ with distinct edge weights. Let $e = (u, v)$ be an edge that does not belong to $T$. Let $P$ be the path between $u$ and $v$ in $T$. Suppose we increase the weight of every edge on path $P$ by 10 and keep all other edge weights unchanged. Which of the following is **TRUE**?
(A) $T$ is guaranteed to remain the MST of the modified graph.  
(B) $T$ cannot be the MST of the modified graph.  
(C) The new MST may contain $e$, but cannot contain any edge from $P$.  
(D) The new MST may contain $e$, and may still contain some edges from $P$.  

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:**  
Originally, $e \notin T \implies w(e) > w(e')$ for every edge $e' \in P$ by the cycle property.
When we increase the weight of every edge in $P$ by 10, some edges in $P$ might now have weight greater than $w(e)$.
By swapping $e$ for the newly heaviest edge on $P$, we could obtain a tree with lower total weight than $T$.
However, $P$ contains multiple edges; the new tree only replaces one edge of $P$ with $e$, so other edges of $P$ remain in the new spanning tree. Hence, the new MST may contain $e$ and will still contain some edges from $P$.
</details>
