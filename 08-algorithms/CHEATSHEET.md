# Algorithms — last-look sheet

| Algorithm | Time | Key condition/property |
|---|---|---|
| Binary search | $O(\log n)$ | sorted random-access sequence |
| Merge sort | $O(n\log n)$ | stable; $O(n)$ extra space |
| Quicksort | expected $O(n\log n)$, worst $O(n^2)$ | pivot split quality |
| Heap sort | $O(n\log n)$ | in-place, not stable |
| BFS / DFS | $O(V+E)$ | adjacency-list graph |
| Dijkstra | $O((V+E)\log V)$ | nonnegative weights |
| Bellman–Ford | $O(VE)$ | negative edges; detects reachable negative cycle |
| Floyd–Warshall | $O(V^3)$ | all-pairs |
| Kruskal | $O(E\log E)$ | sort edges + Union-Find |
| Prim | $O(E\log V)$ | min-priority queue |
| LCS DP | $O(mn)$ | prefix states |

Proof ideas: induction for recursive algorithms; exchange argument for greedy; invariant for loops; cut property for MST. DP checklist: state → recurrence → base → evaluation order → complexity.
