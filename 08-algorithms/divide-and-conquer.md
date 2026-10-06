# Divide and conquer

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** divide-and-conquer design, recurrences, binary search, merge sort, quicksort
> **Prerequisites:** [Asymptotic analysis](asymptotic-analysis.md) · **Leads to:** [Dynamic programming](dynamic-programming.md)

## Quick glance
- Divide: split input; conquer: solve subproblems; combine: merge their results.
- Recurrence captures the work: $T(n)=aT(n/b)+f(n)$.
- Binary search: $T(n)=T(n/2)+\Theta(1)=\Theta(\log n)$.
- Merge sort: $2T(n/2)+\Theta(n)=\Theta(n\log n)$.
- Quicksort: partition; expected $\Theta(n\log n)$ with random pivot, worst $\Theta(n^2)$.

## 1. Design pattern
Use this method when subproblems are smaller instances of the same task. State the base case, number and sizes of recursive calls, and nonrecursive work. Then solve the recurrence and account for stack or temporary storage.

## 2. Merge sort and inversion counting
Split array in half, recursively sort each half, then merge in linear time. The recursion has $\log_2n$ levels with n work per level, giving $\Theta(n\log n)$ time and $\Theta(n)$ auxiliary space. During merge, when a right-half item precedes remaining left items, it contributes one inversion per remaining left item.

Example `[2,4]` and `[1,3]`: take 1 (two left elements are greater: 2,4), then 2, then 3 (4 is greater), then 4; total three cross inversions.

## 3. Quicksort
Partition around pivot so smaller keys precede and larger follow; recurse on each side. Balanced partitions give $\Theta(n\log n)$; repeatedly isolating one item gives $n+(n-1)+\dots+1=\Theta(n^2)$. Random pivot yields expected $\Theta(n\log n)$ regardless of input order, but worst case remains quadratic.

## GATE traps
- Include partition/merge work in recurrence.
- A recurrence result depends on base case and input sizes, not just the recurrence's appearance.
- Quicksort's average/expected time is not its worst-case guarantee.
- Recursion stack space differs from auxiliary array space.

## Connections
- [Recurrences](../01-discrete-mathematics/recurrences-and-generating-functions.md) — same recurrence-solving machinery.
- [Searching and sorting](searching-and-sorting.md) — core applications.
- [Dynamic programming](dynamic-programming.md) — overlapping subproblems change the design choice.

## Practice
**Q1.** Solve $T(n)=2T(n/2)+n$.
<details><summary>Answer</summary> $\Theta(n\log n)$.</details>

**Q2.** Worst-case recurrence for quicksort with maximally unbalanced splits?
<details><summary>Answer</summary> $T(n)=T(n-1)+\Theta(n)=\Theta(n^2)$.</details>

**Q3.** Merge two sorted arrays of lengths 7 and 9: worst comparisons?
<details><summary>Answer</summary> At most $7+9-1=15$.</details>
