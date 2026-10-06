# Graph traversals: BFS, DFS, and ordering

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** BFS, DFS, reachability, connected components, topological sorting, SCCs
> **Prerequisites:** [Graph theory](../01-discrete-mathematics/graph-theory.md) · **Leads to:** [Shortest paths](shortest-paths.md) · [Operating systems](../13-operating-systems/README.md)

## Quick glance
- BFS uses a queue and explores in nondecreasing edge distance; on unweighted graphs it finds shortest paths.
- DFS uses recursion/stack and explores deeply; useful for cycles, finish times, and components.
- With adjacency lists, BFS/DFS are $O(V+E)$.
- Topological order exists iff directed graph is acyclic (DAG).
- SCCs are maximal sets with mutual reachability; Kosaraju uses two DFS passes.

## 1. BFS
Mark start before enqueueing; repeatedly dequeue, then mark/enqueue each unvisited neighbour. Mark-on-enqueue avoids duplicate queue entries. Distances satisfy $d[v]=d[u]+1$ when discovered from u.

Example edges A-B, A-C, B-D, C-D: BFS from A discovers B and C at distance 1, D at distance 2. Parent pointers reconstruct a shortest path.

## 2. DFS and components
DFS marks a vertex, then recursively visits each unvisited neighbour. In an undirected graph, start a DFS from every unvisited vertex to count components. In directed graphs, DFS finish order supports topological sorting and SCC algorithms.

## 3. Topological order and SCC
Kahn's algorithm repeatedly removes a zero-indegree vertex; if fewer than V vertices are removed, a directed cycle exists. SCCs can be found with Kosaraju: DFS to record finish order, reverse all edges, then DFS in decreasing finish order. Each resulting tree is an SCC.

## GATE traps
- BFS gives shortest paths by edge count, not minimum weight when edge weights differ.
- A DFS traversal order can vary with adjacency order; properties should not depend on one tie order.
- Topological order is usually not unique.
- SCC means mutual reachability, not merely weak connectivity.

## Connections
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — graph properties and terminology.
- [Shortest paths](shortest-paths.md) — BFS is the unit-weight shortest path algorithm.
- [Routing](../15-computer-networks/routing.md) — network paths over routers.

## Practice
**Q1.** BFS time using adjacency lists?
<details><summary>Answer</summary> $O(V+E)$.</details>

**Q2.** Can a directed graph with a cycle have a topological ordering?
<details><summary>Answer</summary> No; precedence constraints would form a contradiction.</details>

**Q3.** Which traversal finds shortest paths in an unweighted graph?
<details><summary>Answer</summary> BFS.</details>
