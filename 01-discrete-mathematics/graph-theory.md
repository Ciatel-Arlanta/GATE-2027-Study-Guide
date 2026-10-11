# Graph Theory: Connectivity, Matching, and Colouring

> **Paper:** CS · **Priority:** P0 · **Plan topics:** graphs, paths, connectivity, matching, colouring
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Graph traversals](../08-algorithms/graph-traversals.md) · [Networks routing](../15-computer-networks/routing.md)
> **PYQ weightage:** High · 25 Qs since 2015 (16 since 2021) · DA: 0 · Trend: rising · Most asked: Graph Connectivity, Graph Coloring, Degree sequences, Graph Planarity

## Quick glance

- **Exam focus:** Degree sequences (Handshaking lemma, Havel–Hakimi), Connectivity (cut vertices, bridges, components, edge/vertex connectivity), Euler & Hamiltonian graphs, Planarity (Euler's formula $V - E + R = 1 + c$, Kuratowski's theorem), and Graph Colouring ($\chi(G)$, Brook's theorem, independent set bounds). Tested heavily as NAT and MSQ.
- **Handshaking lemma:** $\sum_{v \in V} \deg(v) = 2|E|$. The number of vertices of **odd degree is always even**.
- **Simple graph bounds:** Simple graph on $n$ vertices has at most $\binom{n}{2} = \frac{n(n-1)}{2}$ edges. Minimum edges to guarantee connectivity: $\binom{n-1}{2} + 1$.
- **Eulerian graphs:** Euler circuit (closed trail visiting every edge once) $\iff$ connected and **all vertices have even degree**. Euler trail (open) $\iff$ connected and **exactly 2 vertices have odd degree**.
- **Hamiltonian graphs:** Visits every vertex once. Dirac: $\deg(v) \ge n/2$ for all $v \implies$ Hamiltonian. Ore: $\deg(u) + \deg(v) \ge n$ for all non-adjacent $u, v \implies$ Hamiltonian. Bipartite with parts $V_1, V_2$ has Hamiltonian cycle only if $|V_1| = |V_2|$.
- **Planar graphs:** Euler's formula: $V - E + R = 2$ for connected planar graphs ($V - E + R = 1 + c$ for $c$ components).
  - Simple planar: $E \le 3V - 6$ (for $V \ge 3$). Triangle-free simple planar (e.g. bipartite): $E \le 2V - 4$.
  - Kuratowski: Planar $\iff$ contains no subgraph homeomorphic to $K_5$ or $K_{3,3}$.
- **Chromatic number $\chi(G)$:** Fewest colours for proper vertex colouring. Trees: $\chi = 2$. Even cycle: $\chi = 2$; Odd cycle: $\chi = 3$. Complete graph: $\chi(K_n) = n$. Planar graph: $\chi \le 4$.
- **Matchings and covers:** Maximum matching size $\nu(G)$, Minimum vertex cover size $\beta(G)$, Maximum independent set size $\alpha(G)$.
  - Gallai identity: $\alpha(G) + \beta(G) = |V|$ (for graphs without isolated vertices).
  - Kőnig's theorem: For bipartite graphs, $\nu(G) = \beta(G)$.

## 1. Graph fundamentals, degrees, and Havel–Hakimi [PYQ: Medium]

### Definitions
A **graph** $G = (V, E)$ consists of a set of vertices $V$ and a set of edges $E$.
- **Simple graph:** Undirected, no self-loops, no parallel edges.
- **Degree $\deg(v)$:** Number of edges incident to $v$. In a multigraph, a self-loop contributes 2 to the degree.
- **Handshaking Lemma:**
  $$\sum_{v \in V} \deg(v) = 2|E|$$
  - *Corollary:* Every undirected graph contains an **even number of odd-degree vertices**.
- **Regular graph:** A graph where every vertex has degree $k$ ($k$-regular). $|E| = \frac{n \cdot k}{2}$. (Requires $n \cdot k$ to be even).
- **Complement graph $\overline{G}$:** $V(\overline{G}) = V(G)$, and $(u, v) \in E(\overline{G}) \iff (u, v) \notin E(G)$.
  $$|E(G)| + |E(\overline{G})| = \binom{n}{2} = \frac{n(n - 1)}{2}$$
  - A graph is **self-complementary** ($G \cong \overline{G}$) only if $|E| = \frac{n(n-1)}{4}$, which requires $n \equiv 0 \pmod 4$ or $n \equiv 1 \pmod 4$.

### Havel–Hakimi theorem (Graphic degree sequences)
A sequence of non-negative integers $d_1 \ge d_2 \ge \dots \ge d_n$ is **graphic** (realizable as a simple graph) if and only if:
1. $\sum d_i$ is even (Handshaking condition).
2. The reduced sequence formed by removing the first element $d_1$ and subtracting 1 from the next $d_1$ elements is graphic. (Sort in descending order after each reduction and repeat until all entries are 0).

**Worked example 1 (Degree sequence verification).**
Determine whether the degree sequence $S = (5, 4, 3, 3, 2, 2, 1)$ is graphic. If graphic, find the number of edges in $G$ and in its complement $\overline{G}$.

*Step-by-step solution:*
1. **Sum of degrees:** $5 + 4 + 3 + 3 + 2 + 2 + 1 = 20$, which is even. Handshaking condition is satisfied.
2. **Apply Havel–Hakimi reduction:**
   - Sequence: $(5, 4, 3, 3, 2, 2, 1)$.
   - Remove the head $d_1 = 5$, and subtract 1 from the next 5 elements:
     $$(4 - 1, 3 - 1, 3 - 1, 2 - 1, 2 - 1, 1) = (3, 2, 2, 1, 1, 1)$$
   - Already sorted: $(3, 2, 2, 1, 1, 1)$.
   - Remove head $3$, subtract 1 from next 3 elements:
     $$(2 - 1, 2 - 1, 1 - 1, 1, 1) = (1, 1, 0, 1, 1)$$
   - Sort descending: $(1, 1, 1, 1, 0)$.
   - Remove head $1$, subtract 1 from next 1 element:
     $$(1 - 1, 1, 1, 0) = (0, 1, 1, 0) \implies \text{sort: } (1, 1, 0, 0)$$
   - Remove head $1$, subtract 1 from next 1 element:
     $$(1 - 1, 0, 0) = (0, 0, 0)$$
   - All elements are zero $\implies$ the sequence is **graphic**.
3. **Number of edges in $G$:** $|E(G)| = \frac{\sum d_i}{2} = \frac{20}{2} = 10$.
4. **Number of edges in $\overline{G}$:** Here $n = 7$.
   $$|E(\overline{G})| = \binom{7}{2} - |E(G)| = \frac{7 \times 6}{2} - 10 = 21 - 10 = 11$$

## 2. Paths, cycles, and connectivity [PYQ: High]

### Walks, paths, and components
- **Walk:** Alternating sequence of vertices and edges.
- **Trail:** A walk with no repeated edges.
- **Path:** A walk with no repeated vertices.
- **Cycle:** A closed path of length $\ge 3$ where only the start and end vertices are identical.
- **Connected components $c(G)$:** Maximal connected subgraphs.

### Cuts and connectivity parameters
- **Cut vertex (Articulation point):** A vertex whose removal strictly increases the number of connected components.
  - In a DFS tree: Root is a cut vertex $\iff$ it has $\ge 2$ children. A non-root vertex $u$ is a cut vertex $\iff$ it has a child $v$ with no back-edge to an ancestor of $u$.
- **Bridge (Cut edge):** An edge whose removal strictly increases the number of connected components.
  - An edge $e$ is a bridge $\iff e$ does **not** lie on any cycle.
- **Vertex connectivity $\kappa(G)$:** Minimum vertices to remove to disconnect $G$ or leave a single vertex.
- **Edge connectivity $\lambda(G)$:** Minimum edges to remove to disconnect $G$.
- **Whitney's Inequality:** For any graph $G$:
  $$\kappa(G) \le \lambda(G) \le \delta(G) \le \text{avg deg} \le \Delta(G)$$
  where $\delta(G)$ is the minimum degree and $\Delta(G)$ is the maximum degree.

### Counting spanning trees
- **Cayley's Formula:** A complete graph $K_n$ has exactly $n^{n - 2}$ spanning trees on $n$ labeled vertices.
- Removing an edge from $K_n$: $K_n - e$ has $(n - 2) n^{n - 3}$ spanning trees.
- Complete bipartite graph $K_{m, n}$ has $m^{n - 1} n^{m - 1}$ spanning trees.

**Worked example 2 (Cuts and connectivity parameters).**
Let graph $G$ be formed by joining two distinct copies of $K_4$ at a single common vertex $v$ (total 7 vertices). Compute:
(a) The number of edges, cut vertices, and bridges.
(b) The vertex connectivity $\kappa(G)$ and edge connectivity $\lambda(G)$.

*Step-by-step solution:*
1. **Vertices and edges:**
   - $V = (4 + 4) - 1 = 7$.
   - Edges: each $K_4$ has $\binom{4}{2} = 6$ edges. Since they share only 1 vertex (no shared edge), $|E| = 6 + 6 = 12$.
2. **Cut vertices:**
   - Removing $v$ splits the remaining 6 vertices into two disconnected components (each a $K_3$).
   - Removing any other vertex leaves its $K_3$ connected and the other $K_4$ connected through $v$.
   - Hence $v$ is the **unique cut vertex** (1 cut vertex).
3. **Bridges:**
   - Every edge inside each $K_4$ lies on at least one cycle (in fact, multiple triangles).
   - Removing any single edge leaves the $K_4$ connected.
   - Hence there are **0 bridges**.
4. **Vertex connectivity $\kappa(G)$:** Removing the single vertex $v$ disconnects $G \implies \kappa(G) = 1$.
5. **Edge connectivity $\lambda(G)$:**
   - To disconnect $G$ by removing edges, we must disconnect a vertex in one of the blocks. The minimum degree in $G$ is $\delta(G) = 3$ (for any vertex other than $v$).
   - Removing the 3 edges incident to any non-$v$ vertex isolates it.
   - Removing fewer than 3 edges leaves both $K_4$ blocks connected because $K_4$ has edge connectivity 3.
   - Hence $\lambda(G) = 3$.
   - Whitney's inequality holds: $\kappa(G) = 1 \le \lambda(G) = 3 \le \delta(G) = 3$.

## 3. Eulerian and Hamiltonian graphs [PYQ: High]

| Property | Eulerian Graph | Hamiltonian Graph |
|---|---|---|
| **Definition** | Traverses every **edge** exactly once | Visits every **vertex** exactly once |
| **Characterization** | Exact local degree condition (polynomial time) | NP-complete in general (sufficient conditions only) |
| **Circuit condition** | Connected and **every vertex has even degree** | Dirac: $\delta(G) \ge n/2$; Ore: $\deg(u) + \deg(v) \ge n$ for non-adjacent $u, v$ |
| **Open trail / path** | Connected and **exactly 2 vertices have odd degree** | Path visits all $n$ vertices without returning to start |
| **Bipartite condition** | All vertex degrees even | Possible only if $\vert V_1\vert = \vert V_2\vert$ (cycle alternates sides) |

```text
Eulerian Circuit:          Hamiltonian Cycle:
Visits every EDGE once      Visits every VERTEX once
   (a)---(b)                   (a)---(b)
    |  X  |                     |     |
   (c)---(d)                   (c)---(d)
Degree of all vertices = 3    Traverses: a -> b -> d -> c -> a
No Euler circuit (degrees odd) Hamiltonian cycle exists!
```

### Directed graph Euler conditions
A directed multigraph has an Eulerian circuit if and only if:
1. It is strongly connected (ignoring isolated vertices).
2. For every vertex $v$, $\text{in-degree}(v) = \text{out-degree}(v)$.

**Worked example 3 (Euler and Hamiltonian checks).**
Analyze the complete bipartite graphs $K_{3, 3}$ and $K_{3, 4}$ for Eulerian and Hamiltonian properties.

*Step-by-step solution:*
1. **For $K_{3, 3}$ ($|V_1| = 3, |V_2| = 3$):**
   - Degrees: Every vertex in $V_1$ has degree 3; every vertex in $V_2$ has degree 3.
   - **Eulerian:** All 6 vertices have odd degree (3). An Euler circuit requires all even degrees; an Euler trail requires at most 2 odd degrees. $K_{3, 3}$ has **neither an Euler circuit nor an Euler trail**.
   - **Hamiltonian:** Bipartite with $|V_1| = |V_2| = 3$. We can alternate: $u_1 \to v_1 \to u_2 \to v_2 \to u_3 \to v_3 \to u_1$. Thus $K_{3, 3}$ is **Hamiltonian**.
2. **For $K_{3, 4}$ ($|V_1| = 3, |V_2| = 4$):**
   - Degrees: Vertices in $V_1$ have degree 4 (even); vertices in $V_2$ have degree 3 (odd).
   - There are 4 vertices of odd degree $\implies$ **Not Eulerian**.
   - **Hamiltonian cycle:** Any cycle in a bipartite graph must alternate between $V_1$ and $V_2$, requiring an equal number of vertices from both parts. Since $|V_1| = 3 \neq |V_2| = 4$, $K_{3, 4}$ **has no Hamiltonian cycle**.
   - **Hamiltonian path:** Can alternate starting and ending in $V_2$: $v_1 \to u_1 \to v_2 \to u_2 \to v_3 \to u_3 \to v_4$. Thus $K_{3, 4}$ **contains a Hamiltonian path**.

## 4. Planar graphs and Euler's formula [PYQ: Medium]

### Definition and Euler's formula
A graph is **planar** if it can be drawn in the plane such that no two edges cross. Such a drawing divides the plane into continuous regions called **faces** (or regions $R$), including the unbounded outer face.

**Euler's Formula:**
- For any **connected planar graph**:
  $$V - E + R = 2$$
- For a planar graph with $k$ connected components:
  $$V - E + R = 1 + k$$

### Face degree and edge bounds
Each face is bounded by a cycle of edges. If the boundary length of face $i$ is $\text{deg}(f_i)$, then:
$$\sum_{f} \text{deg}(f) = 2E$$
In a simple connected planar graph:
1. Every face is bounded by at least 3 edges ($\text{deg}(f) \ge 3$):
   $$2E = \sum \text{deg}(f) \ge 3R \implies R \le \frac{2}{3}E$$
   Substitute into Euler's formula: $V - E + \frac{2}{3}E \ge 2 \implies V - \frac{1}{3}E \ge 2 \implies E \le 3V - 6$ (for $V \ge 3$).
2. If the graph is **triangle-free** (girth $\ge 4$, e.g. bipartite planar graphs):
   $$\text{deg}(f) \ge 4 \implies 2E \ge 4R \implies R \le \frac{E}{2} \implies E \le 2V - 4$$
3. *Degree corollary:* Every simple planar graph contains at least one vertex with degree $\deg(v) \le 5$.

### Kuratowski's theorem
A graph $G$ is planar if and only if it does not contain a subgraph homeomorphic to (or a subdivision of) $K_5$ or $K_{3,3}$.
- $K_5$: Non-planar because $V = 5, E = 10 > 3(5) - 6 = 9$.
- $K_{3,3}$: Bipartite, so triangle-free. $V = 6, E = 9 > 2(6) - 4 = 8 \implies$ non-planar.

**Worked example 4 (Euler's formula calculation — GATE 2021).**
In an undirected connected planar graph $G$, there are 8 vertices and 5 faces. Compute the number of edges. Then, find the maximum possible edges in a simple triangle-free planar graph on 10 vertices.

*Step-by-step solution:*
1. **Edges in the given planar graph:**
   - By Euler's formula: $V - E + R = 2$.
   - Substitute $V = 8$ and $R = 5$:
     $$8 - E + 5 = 2 \implies 13 - E = 2 \implies E = 11$$
   - Verification: $E \le 3V - 6 = 3(8) - 6 = 18$. Since $11 \le 18$, this is valid.
2. **Maximum edges in triangle-free planar graph with $V = 10$:**
   - Every face has at least 4 edges $\implies 2E \ge 4R \implies R \le E/2$.
   - By Euler's formula:
     $$V - E + R = 2 \implies 10 - E + \frac{E}{2} \ge 2 \implies 10 - 2 \ge \frac{E}{2} \implies E \le 2(8) = 16$$
   - Using the direct bound $E \le 2V - 4 = 2(10) - 4 = 16$.
   - The maximum number of edges is **16**.

## 5. Graph colouring and chromatic numbers [PYQ: High]

### Vertex colouring
A **proper vertex colouring** assigns colours to vertices such that no two adjacent vertices share the same colour. The **chromatic number** $\chi(G)$ is the minimum number of colours required.

| Graph family | Chromatic number $\chi(G)$ | Note |
|---|---|---|
| Complete graph $K_n$ | $n$ | Every pair adjacent |
| Complete bipartite $K_{m, n}$ | $2$ ($m, n \ge 1$) | Vertices within a part are independent |
| Tree $T$ ($\vert V\vert \ge 2$) | $2$ | Trees are bipartite |
| Cycle $C_n$ | $2$ if $n$ is even; $3$ if $n$ is odd | Odd cycles require 3 colours |
| Wheel graph $W_n$ ($K_1 + C_{n-1}$) | $3$ if $n-1$ is even; $4$ if $n-1$ is odd | Hub takes 1 colour, outer cycle takes 2 or 3 |
| Planar graphs | $\le 4$ | Four Colour Theorem |

### Bounds on $\chi(G)$
1. **Clique bound:** $\chi(G) \ge \omega(G)$, where $\omega(G)$ is the clique number (size of largest complete subgraph).
   - *Trap:* $\chi(G)$ can be strictly larger than $\omega(G)$. Mycielski's construction produces triangle-free graphs ($\omega = 2$) with arbitrarily high chromatic numbers (e.g. Grötzsch graph has $\omega = 2, \chi = 4$).
2. **Independent set bound:** $\chi(G) \ge \left\lceil \frac{|V|}{\alpha(G)} \right\rceil$, where $\alpha(G)$ is the independence number.
   - *Proof:* In any proper colouring, each colour class is an independent set of size $\le \alpha(G)$. If there are $k$ colour classes, $|V| = \sum_{i=1}^k |C_i| \le k \cdot \alpha(G) \implies k \ge |V| / \alpha(G)$.
3. **Degree bounds:**
   - $\chi(G) \le \Delta(G) + 1$ (achievable by greedy colouring).
   - **Brooks' Theorem:** $\chi(G) \le \Delta(G)$ for any connected graph that is neither a complete graph nor an odd cycle.
4. **Edge bound:** A graph with chromatic number $k$ has at least $\binom{k}{2} = \frac{k(k-1)}{2}$ edges (since there must be at least one edge between every pair of colour classes).

**Worked example 5 (Chromatic number analysis — GATE 2024).**
Let $G$ be a graph with $n$ vertices and chromatic number $k = 4$. Prove that $G$ has an independent set of size at least $\lceil n/4 \rceil$, has at least 6 edges, and contains at least one vertex of degree at least 3.

*Step-by-step solution:*
1. **Independent set size:**
   - In any optimal colouring of $G$ with $k = 4$ colours, the vertex set is partitioned into 4 colour classes: $V = C_1 \cup C_2 \cup C_3 \cup C_4$.
   - Each $C_i$ is an independent set. By the Pigeonhole Principle, at least one colour class must contain $\ge \lceil n/4 \rceil$ vertices. Hence $\alpha(G) \ge \lceil n/4 \rceil$.
2. **Minimum edges:**
   - Between any two distinct colour classes $C_i$ and $C_j$, there must be at least 1 edge. If there were no edge between $C_i$ and $C_j$, all vertices in $C_i \cup C_j$ could be assigned the same colour, reducing the chromatic number to 3, a contradiction.
   - There are $\binom{k}{2} = \binom{4}{2} = 6$ pairs of colour classes, so $|E| \ge 6$.
3. **Maximum degree:**
   - If every vertex had degree $\le 2$, then $\Delta(G) \le 2$.
   - By greedy colouring, $\chi(G) \le \Delta(G) + 1 \le 2 + 1 = 3$, contradicting $\chi(G) = 4$.
   - Hence $G$ must contain at least one vertex with degree $\ge 3$.

## 6. Matchings, independent sets, and vertex covers [PYQ: Medium]

### Definitions
- **Matching $M$:** A set of edges with no common endpoints.
  - **Maximal matching:** Cannot add any more edges without violating the matching property.
  - **Maximum matching $\nu(G)$:** A matching with the largest possible number of edges.
  - **Perfect matching:** Matches every vertex in $V$ (requires $|V|$ to be even; $|M| = |V| / 2$).
- **Independent Set $S$:** A set of vertices such that no two are adjacent. Size of largest: $\alpha(G)$.
- **Vertex Cover $C$:** A set of vertices such that every edge in $E$ is incident to at least one vertex in $C$. Size of smallest: $\beta(G)$.
- **Edge Cover $C'$:** A set of edges covering every vertex. Size of smallest: $\beta'(G)$.

### Fundamental relations
1. **Gallai Identities (for graphs with no isolated vertices):**
   $$\alpha(G) + \beta(G) = |V| \quad \text{and} \quad \nu(G) + \beta'(G) = |V|$$
2. **Kőnig's Theorem:** In any **bipartite graph**:
   $$\nu(G) = \beta(G) \quad (\text{Maximum matching size} = \text{Minimum vertex cover size})$$
3. **Hall's Marriage Theorem:** A bipartite graph $G = (X \cup Y, E)$ has a matching that covers $X$ if and only if:
   $$|N(S)| \ge |S| \quad \text{for every subset } S \subseteq X$$
   where $N(S)$ is the set of neighbours of vertices in $S$.
4. **Regular Bipartite Graphs:** Every $k$-regular bipartite graph with $k \ge 1$ has a **perfect matching**.

**Worked example 6 (Matchings in path graphs — GATE 2026).**
Let $P_n$ be a path on $n$ vertices ($n - 1$ edges). Find the recurrence relation for the total number of matchings $m(P_n)$ in $P_n$ (including the empty matching), and compute $m(P_5)$ and $m(P_6)$.

*Step-by-step solution:*
1. Consider the path $P_n = (v_1, v_2, \dots, v_n)$ with edges $e_1, e_2, \dots, e_{n-1}$ where $e_i = (v_i, v_{i+1})$.
2. Focus on the last edge $e_{n-1} = (v_{n-1}, v_n)$:
   - **Case 1 ($e_{n-1} \notin M$):** The edge $e_{n-1}$ is not in the matching. Then the matching is formed using edges of $P_{n-1}$. There are $m(P_{n-1})$ such matchings.
   - **Case 2 ($e_{n-1} \in M$):** The edge $e_{n-1}$ is included in the matching. Then vertex $v_{n-1}$ is matched, so edge $e_{n-2} = (v_{n-2}, v_{n-1})$ cannot be in the matching. The remaining edges must form a matching on $P_{n-2}$. There are $m(P_{n-2})$ such matchings.
3. Therefore, matchings in a path satisfy the **Fibonacci recurrence**:
   $$m(P_n) = m(P_{n-1}) + m(P_{n-2}) \quad \text{for } n \ge 3$$
4. **Base cases:**
   - $P_1$ (1 vertex, 0 edges): only the empty matching $\implies m(P_1) = 1$.
   - $P_2$ (2 vertices, 1 edge $e_1$): $\emptyset, \{e_1\} \implies m(P_2) = 2$.
5. **Compute sequence:**
   - $m(P_3) = m(P_2) + m(P_1) = 2 + 1 = 3$.
   - $m(P_4) = 3 + 2 = 5$.
   - $m(P_5) = 5 + 3 = 8$.
   - $m(P_6) = 8 + 5 = 13$.

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Handshaking lemma | $\sum \deg(v) = 2\vert E\vert$ | Finding edge count from degrees; odd-degree count is even |
| Connectedness guarantee | $\vert E\vert \ge \binom{n-1}{2} + 1$ | Minimum edges to force a graph of $n$ vertices to be connected |
| Spanning trees of $K_n$ | $n^{n-2}$ (Cayley's formula) | Number of labeled spanning trees of a complete graph |
| Spanning trees of $K_n - e$ | $(n - 2) n^{n-3}$ | Spanning trees of complete graph missing one edge |
| Euler's formula | $V - E + R = 2$ | Connected planar graph face/edge/vertex relations |
| Simple planar edge bound | $E \le 3V - 6$ (for $V \ge 3$) | Planarity tests; maximum edges in simple planar graph |
| Triangle-free planar bound | $E \le 2V - 4$ | Bipartite or girth $\ge 4$ planar graph edge bounds |
| Gallai identity | $\alpha(G) + \beta(G) = \vert V\vert$ | Max independent set $\leftrightarrow$ Min vertex cover |
| Kőnig's theorem | $\nu(G) = \beta(G)$ (for bipartite $G$) | Max matching $\leftrightarrow$ Min vertex cover in bipartite graphs |
| Path matchings $m(P_n)$ | $m(P_n) = m(P_{n-1}) + m(P_{n-2})$ | Total matchings in path graph (Fibonacci numbers) |

## GATE traps

- **Havel–Hakimi vs Handshaking:** Handshaking is a **necessary** condition, not sufficient. A sequence like $(3, 3, 3, 1)$ has even sum (8), but Havel–Hakimi reduces it to $(2, 2, 0) \to (1, -1)$ (invalid negative degree $\implies$ non-graphic).
- **Eulerian in disconnected graphs:** All vertex degrees even does **not** imply an Eulerian circuit if the graph has multiple disconnected non-trivial components.
- **Necessary vs sufficient planarity:** $E \le 3V - 6$ is **necessary** for simple planar graphs, but **not sufficient**. For example, $K_{3,3}$ has $V = 6, E = 9 \le 3(6) - 6 = 12$, yet $K_{3,3}$ is non-planar!
- **Matching vs Perfect matching:** A matching does not have to cover all vertices. A matching is perfect only if every vertex is matched ($|M| = |V| / 2$).
- **Chromatic number vs Clique number:** Never assume $\chi(G) = \omega(G)$. Always check whether odd cycles or triangle-free constructions force a higher chromatic number.

## Connections

- [Sets, relations, functions](sets-relations-functions.md) — relations model directed and undirected graphs; equivalence relations partition graphs into connected components.
- [Graph traversals](../08-algorithms/graph-traversals.md) — BFS/DFS algorithms find connected components, cut vertices, bridges, and test bipartiteness ($2$-colourability).
- [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) — Kruskal and Prim algorithms compute minimum-weight spanning trees on connected graphs.
- [Shortest paths](../08-algorithms/shortest-paths.md) — Dijkstra and Bellman–Ford operate on directed graphs.
- [Networks routing](../15-computer-networks/routing.md) — link-state and distance-vector protocols use graph path algorithms across network topologies.

## Practice

**Q1 (NAT).** A simple undirected graph has 10 vertices. Four vertices have degree 4, two have degree 3, and four have degree 2. The number of edges in the graph is ________.
<details><summary>Answer</summary>
**Answer:** 15 · **Solution:**
By the Handshaking Lemma:
$$\sum_{v \in V} \deg(v) = 2|E|$$
Summing the vertex degrees:
$$\sum \deg(v) = 4(4) + 2(3) + 4(2) = 16 + 6 + 8 = 30$$
$$|E| = \frac{30}{2} = 15$$
</details>

**Q2 (MCQ).** What is the chromatic number of the complete bipartite graph $K_{5, 7}$ and the wheel graph $W_7$ (where $W_7$ consists of a hub vertex connected to all vertices of a cycle $C_6$)?
(A) 2 and 3  
(B) 2 and 4  
(C) 5 and 3  
(D) 5 and 4  
<details><summary>Answer</summary>
**Answer:** (A) · **Solution:**
- For any complete bipartite graph $K_{m, n}$ with $m, n \ge 1$, vertices in the same partition have no edges between them, so 2 colours suffice: $\chi(K_{5, 7}) = 2$.
- The wheel graph $W_7$ has 7 vertices: 1 hub vertex connected to an outer cycle $C_6$.
  The outer cycle $C_6$ is an even cycle, requiring $\chi(C_6) = 2$ colours.
  The hub vertex is adjacent to every vertex of $C_6$, so it cannot share either of those 2 colours.
  Thus, the hub requires a 3rd distinct colour: $\chi(W_7) = 2 + 1 = 3$.
</details>

**Q3 (NAT).** A connected simple planar graph has 12 vertices. Every face (including the outer unbounded face) is bounded by at least 4 edges (i.e., the graph is triangle-free). The maximum possible number of edges in this graph is ________.
<details><summary>Answer</summary>
**Answer:** 20 · **Solution:**
Let $V = 12$, $E$ be edges, and $R$ be faces.
Each face has degree $\ge 4$:
$$2E = \sum_{f} \text{deg}(f) \ge 4R \implies R \le \frac{E}{2}$$
By Euler's formula for connected planar graphs:
$$V - E + R = 2 \implies 12 - E + \frac{E}{2} \ge 2 \implies 10 \ge \frac{E}{2} \implies E \le 20$$
Alternatively, for triangle-free planar graphs, $E \le 2V - 4 = 2(12) - 4 = 20$.
</details>

**Q4 (MSQ).** Let $G$ be any simple undirected graph with $n$ vertices and chromatic number $k \ge 2$. Which of the following statements is/are always TRUE?
(A) $G$ contains a complete subgraph with $k$ vertices ($K_k$).  
(B) $G$ contains an independent set of size at least $\lceil n/k \rceil$.  
(C) $G$ contains at least $\binom{k}{2}$ edges.  
(D) $G$ contains at least one vertex of degree at least $k - 1$.  
<details><summary>Answer</summary>
**Answer:** (B), (C), (D) · **Solution:**
- **(A) is FALSE:** Triangle-free graphs can have arbitrarily large chromatic numbers (e.g. the Grötzsch graph has no triangles $\omega = 2$, but $\chi = 4$).
- **(B) is TRUE:** A proper $k$-colouring partitions $V$ into $k$ independent sets. By the Pigeonhole Principle, at least one independent set must have size $\ge \lceil n/k \rceil$.
- **(C) is TRUE:** For each pair of the $k$ colour classes, there must be at least one edge connecting them (otherwise the two classes could be merged, yielding a $(k-1)$-colouring). There are $\binom{k}{2}$ pairs of colour classes, so $|E| \ge \binom{k}{2}$.
- **(D) is TRUE:** If every vertex had degree $\le k - 2$, then by greedy colouring $\chi(G) \le \Delta(G) + 1 \le (k - 2) + 1 = k - 1$, a contradiction. Thus $\Delta(G) \ge k - 1$.
</details>

**Q5 (NAT).** Let $P_6$ be a path on 6 vertices ($v_1 - v_2 - v_3 - v_4 - v_5 - v_6$). The total number of matchings in $P_6$ (including the empty matching) is ________.
<details><summary>Answer</summary>
**Answer:** 13 · **Solution:**
The number of matchings in a path graph $P_n$ satisfies the Fibonacci recurrence:
$$m(P_n) = m(P_{n-1}) + m(P_{n-2})$$
With base cases:
- $m(P_1) = 1$ ($\emptyset$)
- $m(P_2) = 2$ ($\emptyset, \{e_1\}$)
Evaluating successively:
- $m(P_3) = 2 + 1 = 3$
- $m(P_4) = 3 + 2 = 5$
- $m(P_5) = 5 + 3 = 8$
- $m(P_6) = 8 + 5 = 13$
</details>

**Q6 (MSQ).** Which of the following statements is/are TRUE for simple undirected graphs?
(A) Every bipartite graph with an odd number of vertices has no Hamiltonian cycle.  
(B) If every vertex of a connected graph has even degree, the graph contains an Eulerian circuit.  
(C) The complete bipartite graph $K_{m, n}$ contains an Eulerian circuit if and only if $m$ and $n$ are both even, with $m, n \ge 2$.  
(D) If a connected graph $G$ has a Hamiltonian cycle, then $G$ cannot contain a cut vertex.  
<details><summary>Answer</summary>
**Answer:** (A), (B), (C), (D) · **Solution:**
- **(A) is TRUE:** In a bipartite graph, all cycles have even length. A Hamiltonian cycle visits all $n$ vertices, so its length is $n$. If $n$ is odd, no such cycle can exist.
- **(B) is TRUE:** Standard Euler's Theorem for connected graphs.
- **(C) is TRUE:** In $K_{m, n}$, vertices in the first part have degree $n$, and vertices in the second part have degree $m$. All degrees must be even, so both $m$ and $n$ must be even. Connectedness requires $m, n \ge 2$.
- **(D) is TRUE:** If $G$ has a Hamiltonian cycle $C$, removing any vertex $v$ leaves a Hamiltonian path spanning all remaining vertices, which is connected. Thus $G - v$ remains connected, so $v$ cannot be a cut vertex.
</details>

**Q7 (NAT).** The number of spanning trees in a graph obtained by removing exactly one edge from the complete graph $K_5$ on 5 labeled vertices is ________.
<details><summary>Answer</summary>
**Answer:** 75 · **Solution:**
By Cayley's formula, the complete graph $K_n$ has $n^{n-2}$ spanning trees.
For the complete graph missing one edge, $K_n - e$, the number of spanning trees is given by:
$$\tau(K_n - e) = (n - 2) \cdot n^{n - 3}$$
For $n = 5$:
$$\tau(K_5 - e) = (5 - 2) \cdot 5^{5 - 3} = 3 \cdot 5^2 = 3 \cdot 25 = 75$$
(Alternatively, by symmetry, $K_5$ has 10 edges and 125 spanning trees. Each spanning tree has 4 edges, so by double counting, each edge belongs to $\frac{125 \times 4}{10} = 50$ trees. Removing one edge leaves $125 - 50 = 75$ trees).
</details>
