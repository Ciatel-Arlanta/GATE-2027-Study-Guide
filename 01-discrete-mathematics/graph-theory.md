# Graph theory: connectivity, matching, and colouring

> **Paper:** CS · **Priority:** P0 · **Plan topics:** graphs, paths, connectivity, matching, colouring
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Graph traversals](../08-algorithms/graph-traversals.md) · [Networks routing](../15-computer-networks/routing.md)

## Quick glance
- Undirected graph with n vertices has at most $n(n-1)/2$ edges if simple.
- Sum of degrees is $2|E|$; odd-degree vertices occur in an even count.
- Connected graph: every vertex pair has a path; a tree is connected and acyclic, with $n-1$ edges.
- Matching uses no vertex more than once; perfect matching covers every vertex.
- Chromatic number $\chi(G)$ is the fewest colours for adjacent vertices to differ.

## 1. Degree, paths, and connectivity
Degree counts incident edges; a self-loop contributes two to degree in an undirected graph. The handshaking lemma follows because each edge touches two endpoints. In a graph with degrees $1,2,2,3$, there are $4$ edges since degree sum 8.

A cut vertex is one whose removal increases the number of connected components. A bridge is an edge whose removal disconnects its component. In a tree, every edge is a bridge and every vertex of degree at least 2 may be an articulation point.

## 2. Trees and spanning trees
A tree on n vertices has exactly $n-1$ edges. Equivalent characterisations include connected and acyclic; unique simple path between every pair; or connected with $n-1$ edges. A spanning tree connects all vertices using a subset of graph edges without cycles.

## 3. Matching and colouring
A matching is a set of pairwise vertex-disjoint edges. In a bipartite graph, a perfect matching pairs every vertex on both sides; Hall's condition says each subset X on one side must have at least $|X|$ neighbours for a matching covering that side.

A complete graph $K_n$ needs n colours; a bipartite graph with at least one edge needs 2. An odd cycle needs 3, while an even cycle needs 2.

## GATE traps
- A graph can have every vertex degree even and still be disconnected.
- “Matching” does not mean every vertex is matched; that is a perfect matching.
- A tree is not just acyclic: disconnected forests are acyclic but not trees.
- Greedy colouring depends on vertex order and need not use the chromatic number.

## Connections
- [Graph traversals](../08-algorithms/graph-traversals.md) — BFS/DFS test reachability and components.
- [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) — construct a minimum-cost spanning tree.
- [Networks routing](../15-computer-networks/routing.md) — model routers and links as a graph.

## Practice
**Q1 (NAT).** A simple undirected graph has 8 vertices, each degree 3. Number of edges?
<details><summary>Answer</summary> Degree sum 24, so $|E|=12$.</details>

**Q2.** How many edges does a tree with 15 vertices have?
<details><summary>Answer</summary> 14.</details>

**Q3.** Chromatic number of a cycle of length 7?
<details><summary>Answer</summary> 3, because odd cycles require three colours.</details>
