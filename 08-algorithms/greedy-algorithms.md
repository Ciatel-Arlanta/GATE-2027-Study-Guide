# Greedy algorithms

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** greedy choice, activity selection, fractional knapsack, Huffman coding
> **Prerequisites:** [Asymptotic analysis](asymptotic-analysis.md) · **Leads to:** [Minimum spanning trees](minimum-spanning-trees.md)

## Quick glance
- A greedy algorithm commits to a locally best choice and does not revise it.
- Correctness needs a proof: greedy-choice property plus optimal substructure/exchange argument.
- Activity selection: sort by finish time; repeatedly choose earliest-finishing compatible activity.
- Fractional knapsack: take highest value/weight first; this fails for 0/1 knapsack.
- Prim and Kruskal are greedy MST algorithms; cut property justifies safe edges.

## 1. Activity selection
For intervals, choose the compatible activity that finishes earliest; it leaves the largest remaining time. Example activities (start,finish): (1,3),(2,5),(4,6),(6,8). Choose (1,3), then (4,6), then (6,8): three activities.

## 2. Knapsack distinction
Fractional: capacity 10; items (value,weight) (60,10),(100,20),(120,30). Ratios 6,5,4; take first and half second: value 110. In 0/1 knapsack, fractions are forbidden; ratio greedy can fail because a valuable combination of lower-ratio items may fit better.

## 3. Huffman coding
Repeatedly combine the two least frequent symbols into a binary tree node. For frequencies a:5, b:9, c:12, d:13, e:16, f:45, combine 5+9=14, 12+13=25, 14+16=30, 25+30=55, 45+55=100. Frequent symbols end up nearer the root, minimising weighted code length.

## GATE traps
- A plausible local rule is not a proof of global optimality.
- Earliest start is not the correct activity-selection rule; earliest finish is.
- Fractional and 0/1 knapsack are different problems.
- Huffman ties may yield multiple optimal code trees and lengths.

## Connections
- [Minimum spanning trees](minimum-spanning-trees.md) — greedy cut property.
- [Dynamic programming](dynamic-programming.md) — use when greedy choice cannot be justified.
- [Data structures: heaps](../07-data-structures/heaps.md) — efficiently select the next minimum.

## Practice
**Q1.** What sorting key is used by the classic activity-selection greedy algorithm?
<details><summary>Answer</summary> Increasing finish time.</details>

**Q2.** Is value/weight greedy guaranteed optimal for 0/1 knapsack?
<details><summary>Answer</summary> No; the guarantee holds for fractional knapsack, not 0/1.</details>

**Q3.** Huffman construction combines which two symbols at each step?
<details><summary>Answer</summary> The two currently least frequent nodes.</details>
