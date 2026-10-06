# Algorithms checkpoint

**Q1.** Solve $T(n)=2T(n/2)+n$.
<details><summary>Answer</summary> $\Theta(n\log n)$.</details>

**Q2.** Dijkstra on graph with a negative edge: valid?
<details><summary>Answer</summary> Not guaranteed; use Bellman–Ford for negative edges.</details>

**Q3.** BFS shortest path interpretation?
<details><summary>Answer</summary> Minimum number of edges in an unweighted graph.</details>

**Q4.** How many edges does an MST on n vertices contain?
<details><summary>Answer</summary> $n-1$.</details>

**Q5.** Worst-case quicksort recurrence for pivot producing empty and n−1 sides?
<details><summary>Answer</summary> $T(n)=T(n-1)+\Theta(n)=\Theta(n^2)$.</details>

**Q6.** Why does a DP solution need a state definition?
<details><summary>Answer</summary> It identifies the subproblem and all information needed to decide future transitions.</details>

**Q7.** Which edge does Kruskal reject?
<details><summary>Answer</summary> An edge whose endpoints are already connected; it would make a cycle.</details>

**Q8.** All-pairs shortest path algorithm with $O(V^3)$ time?
<details><summary>Answer</summary> Floyd–Warshall.</details>

**Q9.** Merge two sorted lists of lengths 10 and 12: worst comparisons?
<details><summary>Answer</summary> $10+12-1=21$.</details>

≥80%: proceed to systems. Below 60%: revisit recurrence analysis, greedy proof, and graph-algorithm assumptions.
