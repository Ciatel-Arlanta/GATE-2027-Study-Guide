# Minimum spanning trees

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** minimum spanning tree, Prim's algorithm, Kruskal's algorithm, cut property
> **Prerequisites:** [Graph theory](../01-discrete-mathematics/graph-theory.md) · [Greedy algorithms](greedy-algorithms.md)

## Quick glance
- A spanning tree of connected undirected graph connects all V vertices with V−1 edges.
- MST minimises total edge weight; it need not be unique.
- Kruskal sorts edges and adds one if it joins different components (Union-Find).
- Prim grows one tree by repeatedly adding cheapest edge crossing its cut.
- Cut property: a lightest edge crossing a cut is safe for some MST.
- Complexity commonly $O(E\log E)$ for Kruskal and $O(E\log V)$ with heap Prim.

## 1. Kruskal
Sort edges by weight, scan in order, accept an edge only if endpoints are in different components. Union-Find detects cycles efficiently. Stop after V−1 edges. Example weights 1,2,3,4 on a four-cycle: choose 1,2,3; skip 4 because it closes a cycle.

## 2. Prim
Start at any vertex and maintain a key for each outside vertex: the cheapest edge connecting it to the current tree. Repeatedly add the minimum key and update neighbours. In a disconnected graph this finds an MST only for the component; a minimum spanning forest covers all components.

## GATE traps
- MST is defined for connected, undirected weighted graphs; directed analogue is different.
- A locally cheap edge that closes a cycle is skipped by Kruskal.
- Multiple equal-weight edges can produce multiple MSTs with the same total weight.
- Negative edge weights are allowed for MSTs.

## Connections
- [Greedy algorithms](greedy-algorithms.md) — cut property proves safe greedy choices.
- [Heaps](../07-data-structures/heaps.md) — Prim selects minimum keys.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — spanning trees and connectivity.

## Practice
**Q1.** How many edges in a spanning tree of 12 vertices?
<details><summary>Answer</summary> 11.</details>

**Q2.** Kruskal considers an edge whose endpoints are already in one component. What happens?
<details><summary>Answer</summary> Skip it; adding it would create a cycle.</details>

**Q3.** Must an MST be unique?
<details><summary>Answer</summary> No; ties can produce different trees of equal weight.</details>
