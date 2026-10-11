# Graph Traversals: BFS, DFS, and Structural Orderings

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** BFS, DFS, reachability, connected components, topological sorting, strongly connected components (SCCs), articulation points and bridges
> **Prerequisites:** [Graph theory](../01-discrete-mathematics/graph-theory.md) · [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) · **Leads to:** [Shortest paths](shortest-paths.md) · [Minimum spanning trees](minimum-spanning-trees.md) · [Operating systems deadlock](../13-operating-systems/deadlocks.md)
> **PYQ weightage:** High · 25 Qs since 2015 (15 since 2021) · DA: 7 · Trend: rising · Most asked: BFS/DFS visit sequences, topological sorting, cycle detection, edge classification

## Quick glance

- **Exam focus:** Valid BFS/DFS vertex visit sequences (NAT/MCQ), counting topological orderings of a DAG (NAT), and identifying DFS edge types (Tree, Back, Forward, Cross) to detect cycles or components.
- **BFS** explores level by level using a **FIFO queue**; on unweighted graphs, the BFS tree gives single-source **shortest paths** in $O(V + E)$ time using adjacency lists.
- **Mark on enqueue:** Always mark a vertex as visited the moment it enters the queue; marking on dequeue leads to redundant vertex insertions and $O(V \cdot E)$ queue bloat.
- **DFS** explores deeply using a **call stack**; records discovery time $d[u]$ and finish time $f[u]$ satisfying the **Parenthesis Theorem**: intervals $[d[u], f[u]]$ are either strictly disjoint or nested.
- **Edge classification (directed):** Tree edge (discovers white vertex), Back edge (points to ancestor / gray vertex $\iff$ **cycle exists**), Forward edge (points to descendant / black vertex with $d[u] < d[v]$), Cross edge (points to cross-branch black vertex with $d[u] > d[v]$).
- **Undirected DFS edges:** Only **Tree edges** and **Back edges** can exist; forward and cross edges are impossible in an undirected DFS tree.
- **Topological sort** exists **if and only if** the directed graph is a DAG; Kahn's algorithm removes indegree-0 vertices, while DFS topological sort outputs vertices in **decreasing order of finish times** $f[u]$.
- A DAG has a **unique topological sort** iff it contains a directed **Hamiltonian path** (every consecutive pair in the ordering has a directed edge).
- **Strongly Connected Components (SCCs):** Kosaraju's algorithm uses two DFS passes ($G$, then reversed graph $G^T$ in decreasing $f[u]$ order) in $O(V + E)$ time.
- **Articulation points (cut vertices):** Root of DFS tree is an articulation point iff it has $\ge 2$ children; non-root $u$ is an articulation point iff it has a child $v$ with $\text{low}[v] \ge \text{disc}[u]$.
- **Bridge (cut edge):** Tree edge $(u, v)$ is a bridge iff $\text{low}[v] > \text{disc}[u]$.

## 1. Breadth-First Search (BFS) and Graph Properties [PYQ: High]

Breadth-First Search radiates outward from a source vertex $s$ level-by-level, visiting all vertices at distance $k$ before any vertex at distance $k+1$.

### Formal specification and complexity
Given graph $G = (V, E)$ represented via adjacency lists:
1. Initialize $d[v] = \infty$, $\text{visited}[v] = \text{false}$ for all $v \in V$. Set $d[s] = 0$, $\text{visited}[s] = \text{true}$, and enqueue $s$ into queue $Q$.
2. While $Q$ is not empty, dequeue $u$. For each neighbor $v \in \text{Adj}[u]$, if $\text{visited}[v] == \text{false}$: set $\text{visited}[v] = \text{true}$, $d[v] = d[u] + 1$, and enqueue $v$.
- **Time complexity:** $O(V + E)$ with adjacency lists; $O(V^2)$ with an adjacency matrix. Space complexity is $O(V)$.

### Key BFS applications in GATE
1. **Shortest paths on unweighted graphs:** $d[v]$ represents the minimum number of edges from $s$ to $v$.
2. **Bipartite testing (2-colorability):** A graph is bipartite iff it contains no odd-length cycles. If an edge connects two vertices of the same BFS level/color, the graph is not bipartite.
3. **Graph diameter in trees:** Run BFS from an arbitrary node $x$ to find the farthest node $y$; run a second BFS from $y$ to find the farthest node $z$. Distance $d(y, z)$ is the tree diameter.

### Worked example: BFS trace and queue states
Consider the undirected graph with vertices $\{1, 2, 3, 4, 5, 6\}$ and edges $E = \{(1, 2), (1, 3), (2, 4), (2, 5), (3, 5), (4, 6), (5, 6)\}$. Start BFS at vertex 1; neighbors visited in ascending order.

| Step | Vertex popped | Neighbors inspected | Queue state (Front $\to$ Rear) | Distances $[d_1..d_6]$ |
|---|---|---|---|---|
| 0 | Initial state | — | `[1]` | $[0, \infty, \infty, \infty, \infty, \infty]$ |
| 1 | Pop 1 | 2 (push), 3 (push) | `[2, 3]` | $[0, 1, 1, \infty, \infty, \infty]$ |
| 2 | Pop 2 | 1 (seen), 4 (push), 5 (push) | `[3, 4, 5]` | $[0, 1, 1, 2, 2, \infty]$ |
| 3 | Pop 3 | 1 (seen), 5 (already marked in step 2) | `[4, 5]` | $[0, 1, 1, 2, 2, \infty]$ |
| 4 | Pop 4 | 2 (seen), 6 (push) | `[5, 6]` | $[0, 1, 1, 2, 2, 3]$ |
| 5 | Pop 5 | 2 (seen), 3 (seen), 6 (already marked) | `[6]` | $[0, 1, 1, 2, 2, 3]$ |
| 6 | Pop 6 | 4 (seen), 5 (seen) | `[]` | $[0, 1, 1, 2, 2, 3]$ |

- **Traversal order:** $1 \to 2 \to 3 \to 4 \to 5 \to 6$.
- **BFS tree edges:** $(1, 2), (1, 3), (2, 4), (2, 5), (4, 6)$. Non-tree edges $(3, 5)$ and $(5, 6)$ connect vertices whose distances differ by $\le 1$.

---

## 2. Depth-First Search (DFS), Timestamps, and Edge Classification [PYQ: High]

DFS dives deep along a branch until it hits a dead end, then backtracks. It tracks discovery time $d[u]$ (color turns Gray) and finish time $f[u]$ (color turns Black).

### Parenthesis Theorem and Edge Classification
For any pair of vertices $u$ and $v$, intervals $[d[u], f[u]]$ and $[d[v], f[v]]$ are either strictly disjoint or nested.
When an edge $(u, v)$ is explored in a directed graph:
- **Tree edge:** $v$ is White (unvisited). Form the DFS spanning forest.
- **Back edge:** $v$ is Gray (ancestor currently on the call stack, $d[v] < d[u] < f[u] < f[v]$). **A directed graph contains a cycle $\iff$ DFS has at least one back edge.**
- **Forward edge:** $v$ is Black, and $v$ is a descendant of $u$ ($d[u] < d[v] < f[v] < f[u]$).
- **Cross edge:** $v$ is Black, but $v$ is not an ancestor or descendant ($d[v] < f[v] < d[u] < f[u]$).
- In an **undirected graph**, every edge is either a **Tree edge** or a **Back edge** (Forward and Cross edges are impossible).

### Worked example: Full DFS trace with timestamps and edge types
Consider directed graph $G = (V, E)$ with vertices $\{A, B, C, D, E, F\}$ and edges:
$$E = \{(A, B), (A, C), (A, E), (B, D), (B, E), (C, B), (C, F), (D, A), (E, F)\}$$
Traverse using DFS starting from $A$, breaking ties alphabetically.

```text
DFS execution step-by-step:
- Time 1: Visit A -> d[A]=1 (Gray). Check A->B: Tree edge (A,B).
- Time 2: Visit B -> d[B]=2 (Gray). Check B->D: Tree edge (B,D).
- Time 3: Visit D -> d[D]=3 (Gray). Check D->A: A is Gray -> Back edge (D,A).
- Time 4: Finish D -> f[D]=4 (Black). Backtrack to B. Check B->E: Tree edge (B,E).
- Time 5: Visit E -> d[E]=5 (Gray). Check E->F: Tree edge (E,F).
- Time 6: Visit F -> d[F]=6 (Gray). No outgoing edges. Time 7: f[F]=7 (Black).
- Time 8: Finish E -> f[E]=8 (Black). Backtrack to B.
- Time 9: Finish B -> f[B]=9 (Black). Backtrack to A. Check A->C: Tree edge (A,C).
- Time 10: Visit C -> d[C]=10 (Gray). Check C->B: f[B]=9 < d[C]=10 -> Cross edge (C,B).
- Check C->F: f[F]=7 < d[C]=10 -> Cross edge (C,F). Time 11: f[C]=11 (Black).
- Backtrack to A. Check A->E: E is descendant of A (d[A]=1 < d[E]=5) -> Forward edge (A,E).
- Time 12: Finish A -> f[A]=12 (Black).
```

| Vertex | $d[u]$ | $f[u]$ | Interval $[d, f]$ | Outgoing edges classified |
|---|---|---|---|---|
| $A$ | 1 | 12 | $[1, 12]$ | $(A, B)$ Tree, $(A, C)$ Tree, $(A, E)$ Forward |
| $B$ | 2 | 9 | $[2, 9]$ | $(B, D)$ Tree, $(B, E)$ Tree |
| $D$ | 3 | 4 | $[3, 4]$ | $(D, A)$ Back (cycle $A \to B \to D \to A$) |
| $E$ | 5 | 8 | $[5, 8]$ | $(E, F)$ Tree |
| $F$ | 6 | 7 | $[6, 7]$ | — |
| $C$ | 10 | 11 | $[10, 11]$ | $(C, B)$ Cross, $(C, F)$ Cross |

---

## 3. Topological Sorting and DAGs [PYQ: High]

A **topological sort** of a directed graph $G = (V, E)$ is a linear ordering of vertices such that for every directed edge $(u, v) \in E$, $u$ appears before $v$.

### Properties and algorithms
1. **Existence:** A topological ordering exists **if and only if** the graph is a Directed Acyclic Graph (DAG).
2. **Kahn's Algorithm (in-degree queue):** Compute in-degrees. Enqueue all vertices with in-degree 0. While queue not empty, dequeue $u$, output $u$, and decrement neighbors' in-degrees. If neighbor reaches in-degree 0, enqueue it. If output count $< |V|$, a cycle exists. Time: $O(V + E)$.
3. **DFS Finish-Time Algorithm:** Run DFS. Output vertices in **decreasing order of finish times** $f[u]$.
4. **Uniqueness:** A DAG has a unique topological order iff there is an edge $(v_i, v_{i+1})$ between every adjacent pair in the ordering, forming a **directed Hamiltonian path**.

### Worked example: Counting topological orderings
Consider DAG with vertices $\{1, 2, 3, 4, 5\}$ and directed edges:
$$(1, 3), (1, 4), (2, 3), (2, 5), (3, 4), (3, 5), (4, 5)$$
- In-degrees: $\text{in}[1] = 0$, $\text{in}[2] = 0$, $\text{in}[3] = 2$, $\text{in}[4] = 2$, $\text{in}[5] = 3$.
- Vertices 1 and 2 must both precede 3.
- Subgraph $\{3, 4, 5\}$ has edges $3 \to 4, 3 \to 5, 4 \to 5$, fixing their relative order strictly to $3 \prec 4 \prec 5$.
- Vertices 1 and 2 have no edge between them and both precede 3. They can appear in order $(1, 2)$ or $(2, 1)$.
- Hence, exactly **2 valid topological orderings** exist: `1, 2, 3, 4, 5` and `2, 1, 3, 4, 5`.

---

## 4. Connected Components and Strongly Connected Components (SCCs) [PYQ: Medium]

An undirected graph's connected components are found via a single BFS/DFS pass in $O(V + E)$ time. For directed graphs, a **Strongly Connected Component (SCC)** is a maximal sub-graph where every pair $(u, v)$ has paths $u \rightsquigarrow v$ and $v \rightsquigarrow u$.

### Kosaraju-Sharir Two-Pass Algorithm ($O(V + E)$)
1. **Pass 1:** Run DFS on $G$, pushing vertices onto a stack $S$ upon finishing (sorted by ascending finish time).
2. **Transpose:** Construct $G^T$ by reversing all directed edges in $E$.
3. **Pass 2:** Pop vertices from stack $S$. For each unvisited vertex, run DFS on $G^T$. Each resulting DFS tree is an independent SCC.
- **Why it works:** The vertex with highest finish time in Pass 1 belongs to a source SCC in the condensation DAG. In $G^T$, this source becomes a sink, so DFS on $G^T$ cannot bleed into other SCCs.

### Worked example: Kosaraju trace
Let $V = \{1, 2, 3, 4, 5\}$, $E = \{(1, 2), (2, 3), (3, 1), (2, 4), (4, 5), (5, 4)\}$.
- Pass 1 DFS on $G$ from 1: explores cycle $1 \to 2 \to 3 \to 1$, branch $2 \to 4 \leftrightarrow 5$. Finish order (order of finishing): $3, 5, 4, 2, 1$. Pushing each vertex as it finishes gives stack $S$ top-to-bottom: `[1, 2, 4, 5, 3]`.
- Reverse edges for $G^T$: $E^T = \{(2, 1), (3, 2), (1, 3), (4, 2), (5, 4), (4, 5)\}$.
- Pass 2 on $G^T$: Pop 1 $\to$ DFS from 1 visits $\{1, 3, 2\}$. Edge $(4, 2)$ in $G^T$ points inward, so search stays confined. $\implies \mathbf{SCC_1 = \{1, 2, 3\}}$.
- Next unvisited on stack is 4 $\to$ DFS on $G^T$ from 4 visits $\{4, 5\}$. $\implies \mathbf{SCC_2 = \{4, 5\}}$.
- Total SCCs = 2. Condensation DAG has edge $\text{SCC}_1 \to \text{SCC}_2$.

---

## 5. Articulation Points and Bridges (Biconnectivity) [PYQ: Medium]

- An **articulation point (cut vertex)** is a vertex whose removal increases the number of connected components.
- A **bridge (cut edge)** is an edge whose removal increases the number of connected components.

### Tarjan's Low-Link Algorithm
Maintain discovery time $\text{disc}[u]$ and low-link value $\text{low}[u] = \min(\text{disc}[u], \min_{(u,v)\text{ back}}\text{disc}[v], \min_{(u,v)\text{ tree}}\text{low}[v])$.
1. **Root condition:** DFS root is an articulation point **iff** it has $\ge 2$ children in the DFS tree.
2. **Non-root condition:** Vertex $u$ is an articulation point **iff** it has a child $v$ with $\text{low}[v] \ge \text{disc}[u]$.
3. **Bridge condition:** Tree edge $(u, v)$ is a bridge **iff** $\text{low}[v] > \text{disc}[u]$.

### Worked example: Finding articulation points and bridges
Let $V = \{A, B, C, D, E, F\}$, $E = \{(A, B), (B, C), (C, A), (C, D), (D, E), (E, F), (F, D)\}$. Root DFS at $A$.
- DFS tree: $A \to B \to C \to D \to E \to F$. Back edges: $(C, A)$ to ancestor $A$, $(F, D)$ to ancestor $D$.

| Vertex $u$ | $\text{disc}[u]$ | $\text{low}[u]$ | Children in DFS tree | Articulation point? | Reason |
|---|---|---|---|---|---|
| $A$ (root) | 1 | 1 | $\{B\}$ | **No** | Root has only 1 child in DFS tree |
| $B$ | 2 | 1 | $\{C\}$ | **No** | Child $C$ has $\text{low}[C] = 1 < \text{disc}[B] = 2$ |
| $C$ | 3 | 1 | $\{D\}$ | **YES** | Child $D$ has $\text{low}[D] = 4 \ge \text{disc}[C] = 3$ |
| $D$ | 4 | 4 | $\{E\}$ | **YES** | Child $E$ has $\text{low}[E] = 4 \ge \text{disc}[D] = 4$ |
| $E$ | 5 | 4 | $\{F\}$ | **No** | Child $F$ has $\text{low}[F] = 4 < \text{disc}[E] = 5$ |
| $F$ | 6 | 4 | $\emptyset$ | **No** | Leaf node |

- Edge $(C, D)$ has $\text{low}[D] = 4 > \text{disc}[C] = 3 \implies \mathbf{(C, D)}$ **is a bridge**.
- Articulation points: $\{C, D\}$. Bridges: $\{(C, D)\}$.

---

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| BFS time complexity | $O(V + E)$ (lists), $O(V^2)$ (matrix) | Shortest paths in unweighted graphs |
| DFS time complexity | $O(V + E)$ (lists), $O(V^2)$ (matrix) | Cycle detection, topological sort, components |
| Directed cycle criterion | Directed cycle $\iff$ Back edge in DFS | Cycle verification in directed graphs |
| Undirected cycle criterion | Non-tree edge exists $\iff$ Back edge $\iff$ Cycle | Cycle verification in undirected graphs |
| Bipartite graph criterion | Bipartite $\iff$ No odd-length cycle | 2-colorability verification |
| Topological sort condition | Graph is a DAG $\iff$ Topo sort exists | Task scheduling, dependency resolution |
| Unique topological sort | Unique $\iff$ Directed Hamiltonian path exists | Checking if ordering is completely determined |
| Kosaraju SCC time | 2 DFS passes $= O(V + E)$ | Computing strongly connected components |
| Articulation point (root) | $\ge 2$ children in DFS tree | Testing starting DFS vertex |
| Articulation point (non-root) | Child $v$ satisfies $\text{low}[v] \ge \text{disc}[u]$ | Cut vertex testing for internal nodes |
| Bridge condition | Tree edge $(u, v)$ satisfies $\text{low}[v] > \text{disc}[u]$ | Critical link identification |

---

## GATE traps

- **Mark on dequeue bug:** Enqueueing a neighbor without marking it visited allows multiple parallel paths to push the same node repeatedly, causing queue size to explode to $O(E)$ and runtime to degrade. Always **mark on enqueue**.
- **Topological sort uniqueness:** Having in-degree 0 at only one vertex initially does *not* guarantee a unique topological sort; ties can appear at any later stage. A unique topological sort exists **if and only if** there is an edge $(v_i, v_{i+1})$ between every consecutive pair in the ordering (Hamiltonian path).
- **Undirected DFS edges:** Never classify an undirected DFS edge as "forward" or "cross". In undirected graphs, any non-tree edge reaches an already active ancestor and is strictly a **back edge**.
- **Weak vs. Strong connectivity:** A directed graph can be weakly connected (connected when edge directions are ignored) without having any SCC with $> 1$ vertex. SCC requires **mutual reachability**.
- **Root articulation point trap:** The condition $\text{low}[v] \ge \text{disc}[r]$ holds vacuously for the root, but the root is an articulation point **only if** it has $\ge 2$ subtrees in the DFS forest.

---

## Connections

- [Graph theory](../01-discrete-mathematics/graph-theory.md) — degree sum formula, bipartite graph characterizations, and tree properties.
- [Shortest paths](shortest-paths.md) — BFS is Dijkstra's algorithm specialized to unit-weight edges.
- [Minimum spanning trees](minimum-spanning-trees.md) — cut property and cycle property parallel DFS edge classifications.
- [Operating systems deadlock](../13-operating-systems/deadlocks.md) — Resource Allocation Graph (RAG) cycle detection maps directly to DFS back edges.

---

## Practice

**Q1. (MCQ, Easy)** Let $G$ be an undirected connected graph with $n$ vertices and $m$ edges. Running BFS using an adjacency matrix takes time:
(A) $O(n + m)$  
(B) $O(n^2)$  
(C) $O(m \log n)$  
(D) $O(n \cdot m)$  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
For each of the $n$ dequeued vertices, the algorithm scans an entire row of length $n$ in the adjacency matrix to identify adjacent neighbors, regardless of the number of edges. Total time $= \sum_{u=1}^n O(n) = O(n^2)$. Adjacency lists achieve $O(n + m)$.
</details>

---

**Q2. (MCQ, Easy)** In a depth-first traversal of a directed graph, vertex $u$ is discovered at timestamp 3 and finishes at timestamp 10. Vertex $v$ is discovered at timestamp 4 and finishes at timestamp 7. The directed edge $(u, v)$ is a:
(A) Tree edge or Forward edge  
(B) Back edge  
(C) Cross edge  
(D) Tree edge or Cross edge  

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:**  
The interval for $v$, $[4, 7]$, is strictly nested within the interval for $u$, $[3, 10]$. By the Parenthesis Theorem, $v$ is a descendant of $u$ in the DFS forest. A directed edge $(u, v)$ from an ancestor to a descendant is either a **Tree edge** (if $v$ was discovered directly from $u$) or a **Forward edge** (if $v$ was discovered through another path from $u$).
</details>

---

**Q3. (NAT, Medium)** Consider the following directed acyclic graph (DAG) with vertices $\{A, B, C, D, E\}$:
$$E = \{(A, B), (A, C), (B, D), (C, D), (D, E)\}$$
The number of distinct topological orderings of this graph is _______.

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:**  
1. $A$ has in-degree 0 and must come first.
2. $E$ has out-degree 0 and depends on $D$, which depends on $B$ and $C$.
3. Path $A \to B \to D \to E$ forces relative order $A \prec B \prec D \prec E$.
4. Path $A \to C \to D \to E$ forces relative order $A \prec C \prec D \prec E$.
5. Between $B$ and $C$, there is no directed edge. Both can be placed in either order between $A$ and $D$.
Valid orderings: $A, B, C, D, E$ and $A, C, B, D, E$. Total count $= 2$.
</details>

---

**Q4. (MCQ, Medium)** Let $G = (V, E)$ be a directed graph. Which of the following statements is **FALSE**?
(A) If $G$ contains a directed cycle, any DFS on $G$ will find at least one back edge.  
(B) If $G$ is a DAG, Kahn's algorithm will remove all $|V|$ vertices.  
(C) In an undirected graph, cross edges can exist in a DFS forest if the graph has multiple connected components.  
(D) The condensation graph (graph formed by contracting each SCC into a single supernode) is always a DAG.  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**  
In any DFS of an undirected graph, **cross edges and forward edges can never exist**, even across multiple components or branches. If an edge exists between two vertices in an undirected graph, one will be discovered while the other is still active, making it a tree edge or a back edge. Statements A, B, and D are all standard true theorems.
</details>

---

**Q5. (MSQ, Medium)** Which of the following statements are **TRUE** regarding graph traversals?
(A) In a connected undirected graph, BFS finds the shortest path between any two vertices in terms of number of edges.  
(B) A graph is bipartite if and only if BFS traversal produces no cross edges between vertices at the same level.  
(C) In a DAG, the vertex with the minimum finish time in DFS must have out-degree 0.  
(D) If an undirected graph has an articulation point, it must also have a bridge.  

<details><summary>Answer</summary>

**Answer:** (A), (B), (C)  
**Solution:**  
- **(A) True:** BFS explores in non-decreasing order of edge distance from the source.
- **(B) True:** Vertices at the same level in BFS have the same partition color. An edge between two vertices at the same level creates an odd-length cycle, violating bipartiteness.
- **(C) True:** The vertex that finishes first cannot have any unvisited outgoing edges; all reachable nodes have already finished or it has out-degree 0 in the DAG.
- **(D) False:** Counterexample: Two triangles sharing a single common vertex (bowtie graph). The shared vertex is an articulation point, but every edge lies on a cycle, so there are **no bridges**.
</details>

---

**Q6. (NAT, Hard)** Let $G$ be an undirected graph on 8 vertices consisting of two cycles of length 5 that share exactly one edge $(u, v)$ (total 8 vertices and 9 edges). The number of articulation points in $G$ is _______.

<details><summary>Answer</summary>

**Answer:** 0  
**Solution:**  
Let the shared edge be between $u$ and $v$. The two cycles share vertices $u$ and $v$ and the edge between them.
Cycle 1: $u - a_1 - a_2 - a_3 - v - u$ (length 5).
Cycle 2: $u - b_1 - b_2 - b_3 - v - u$ (length 5).
Total vertices: $u, v, a_1, a_2, a_3, b_1, b_2, b_3 \implies 8$ vertices.
- If we remove $u$: $v$ is still connected to all $a_i$ via $v-a_3-a_2-a_1$ and to all $b_i$ via $v-b_3-b_2-b_1$. The remaining graph is connected.
- If we remove $v$: by symmetry, $u$ connects all remaining vertices.
- If we remove any $a_i$ or $b_j$: the remaining vertices in that cycle remain connected via $u$ and $v$.
Since the removal of no single vertex disconnects the graph, the number of articulation points is **0** (the graph is 2-connected / biconnected).
</details>

---

**Q7. (MCQ, Hard)** Consider a directed graph $G = (V, E)$. Let $T$ be the DFS tree produced by a search from vertex $s$, and suppose $T$ spans all vertices in $V$. If $G$ has no back edges, what can be asserted about the topological ordering of $G$?
(A) No topological ordering exists because $s$ cannot reach itself.  
(B) Exactly one unique topological ordering exists.  
(C) At least one topological ordering exists, but it may not be unique.  
(D) $G$ must be a directed cycle.  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**  
Because $G$ has no back edges, $G$ is a Directed Acyclic Graph (DAG), ensuring that at least one topological ordering exists.
However, having a DFS spanning tree does not imply a Hamiltonian path. For example, if $s$ has tree edges to $A$ and $B$, but no edge exists between $A$ and $B$, both $(s, A, B)$ and $(s, B, A)$ could be valid topological orderings. Hence, the topological order may not be unique.
</details>
