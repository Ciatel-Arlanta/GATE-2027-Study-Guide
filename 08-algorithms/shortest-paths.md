# Shortest paths

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** BFS shortest paths, Dijkstra, Bellman–Ford, Floyd–Warshall
> **Prerequisites:** [Graph traversals](graph-traversals.md) · [Dynamic programming](dynamic-programming.md) · **Leads to:** [Network routing](../15-computer-networks/routing.md)

## Quick glance
- Unweighted graph: BFS gives shortest paths by number of edges.
- Dijkstra: nonnegative edge weights; repeatedly finalises nearest unsettled vertex.
- Bellman–Ford: allows negative edges; V−1 relaxation passes; detects reachable negative cycle.
- Floyd–Warshall: all-pairs, $O(V^3)$; update through intermediate k.
- Relax edge $(u,v)$ when $d[u]+w(u,v)<d[v]$.

## 1. Relaxation and Dijkstra
Initialise source distance 0, others infinity. Relaxing an edge improves a tentative path. Dijkstra chooses the smallest tentative distance and finalises it; nonnegative weights ensure no later route can improve it. A negative edge breaks this reasoning.

Example S→A weight 4, S→B weight 1, B→A weight 2: direct tentative d(A)=4; after finalising B, relaxation improves d(A)=3.

## 2. Bellman–Ford
Relax all edges V−1 times because a shortest simple path has at most V−1 edges. If any edge can still relax afterward, a reachable negative-weight cycle exists. Runtime $O(VE)$.

## 3. Floyd–Warshall
For each intermediate vertex k, update $D[i][j]=\min(D[i][j],D[i][k]+D[k][j])$. Initialise diagonal 0 and direct edge weights; absent edges infinity. Negative cycles are indicated by a negative diagonal after completion.

## GATE traps
- Dijkstra is not valid with negative edge weights.
- Negative edge is not itself a negative cycle.
- Bellman–Ford detects only cycles reachable from the chosen source unless augmented with a super-source.
- Floyd–Warshall loop order must keep k as the outer stage.

## Connections
- [Graph traversals](graph-traversals.md) — BFS is shortest path for equal edge costs.
- [Dynamic programming](dynamic-programming.md) — Floyd–Warshall builds paths through permitted intermediate vertices.
- [Routing](../15-computer-networks/routing.md) — routing protocols calculate network paths.

## Practice
**Q1.** Which algorithm handles negative edges and detects negative cycles?
<details><summary>Answer</summary> Bellman–Ford.</details>

**Q2.** Complexity of Floyd–Warshall?
<details><summary>Answer</summary> $O(V^3)$ time and $O(V^2)$ space.</details>

**Q3.** Dijkstra's key assumption about weights?
<details><summary>Answer</summary> All edge weights must be nonnegative.</details>
