# Dynamic programming

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** optimal substructure, overlapping subproblems, memoization, tabulation, classic DP
> **Prerequisites:** [Recurrences](../01-discrete-mathematics/recurrences-and-generating-functions.md) · **Leads to:** [Shortest paths](shortest-paths.md)

## Quick glance
- DP applies when subproblems overlap and an optimal answer composes from smaller optimal answers.
- Define state precisely, write recurrence, set base cases, choose evaluation order.
- Memoization is top-down caching; tabulation is bottom-up filling.
- LCS: $dp[i][j]$ compares prefixes; time and space $O(mn)$.
- 0/1 knapsack: each item used at most once; time $O(nW)$ (pseudo-polynomial).

## 1. Building a DP
Ask: (1) What subproblem parameter(s) identify a smaller instance? (2) What choices lead to it? (3) What is the base case? (4) Which states depend on which others? Then prove the recurrence accounts for all choices.

## 2. Fibonacci and LCS
Naive Fibonacci repeats work exponentially. Memoization stores each $F_i$ once, reducing time to $O(n)$. For LCS of strings X and Y, if last characters match, $dp[i][j]=1+dp[i-1][j-1]$; otherwise take $\max(dp[i-1][j],dp[i][j-1])$. For `ABC` and `AC`, LCS is `AC`, length 2.

## 3. 0/1 knapsack
Let $dp[i][w]$ be best value using first i items within capacity w. Either skip item i or take it (if it fits): $dp[i][w]=\max(dp[i-1][w],v_i+dp[i-1][w-w_i])$. Use previous row so an item cannot be reused. Capacity W is numeric, making runtime polynomial in W but not in input bit length.

## GATE traps
- State must retain all information needed to make future decisions.
- Loop order can silently turn 0/1 knapsack into unbounded knapsack.
- Memoization avoids repeated work but still uses recursion stack.
- DP complexity is number of states × work per state.

## Connections
- [Greedy algorithms](greedy-algorithms.md) — greedy requires a stronger local-choice property.
- [Shortest paths](shortest-paths.md) — Bellman-Ford and Floyd-Warshall use relaxation/DP structure.
- [Recurrences](../01-discrete-mathematics/recurrences-and-generating-functions.md) — state recurrences describe DP.

## Practice
**Q1.** LCS length of `ABCD` and `ACBD`?
<details><summary>Answer</summary> 3: `ABD` or `ACD`.</details>

**Q2.** If a DP has n×W states and O(1) work per state, time?
<details><summary>Answer</summary> $O(nW)$.</details>

**Q3.** What does memoization store?
<details><summary>Answer</summary> The result of each solved subproblem, so later calls reuse it.</details>
