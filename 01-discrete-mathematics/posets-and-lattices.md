# Posets and lattices

> **Paper:** CS · **Priority:** P1 · **Plan topics:** partial orders, Hasse diagrams, maximal/minimal elements, bounds, lattices
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Boolean algebra](../09-digital-logic/boolean-algebra-and-minimization.md)

## Quick glance
- A poset is a set with a reflexive, antisymmetric, transitive relation.
- In a Hasse diagram, omit reflexive loops and transitive edges; higher means greater.
- Maximal is not necessarily greatest; minimal is not necessarily least.
- A lattice has a unique meet (greatest lower bound) and join (least upper bound) for every pair.
- In a power set ordered by inclusion: meet = intersection, join = union.

## 1. Reading a Hasse diagram
For divisibility on $\{1,2,3,6\}$, the cover relations are $1<2<6$ and $1<3<6$. Draw 1 at bottom, 2 and 3 in the middle, 6 at top. The edge from 1 to 6 is omitted because it is implied by paths.

The greatest element is above every element; a maximal element has no larger element. In a poset with two incomparable tops, each can be maximal while neither is greatest.

## 2. Bounds, meet, and join
For subsets of $\{a,b\}$ under inclusion, the meet of $\{a\}$ and $\{b\}$ is $\emptyset$ (largest subset contained in both); their join is $\{a,b\}$ (smallest set containing both). This is the Boolean lattice $\mathcal P(\{a,b\})$.

To test if a poset is a lattice, examine every pair: there must be exactly one greatest lower bound and one least upper bound. Missing a bound or having two incomparable candidate bounds means it is not a lattice.

## GATE traps
- The highest drawn node may not be greatest if another branch has a separate maximum.
- A least upper bound is itself an upper bound and is below every other upper bound.
- Meet/join terminology is relative to the order, not ordinary numeric min/max in every poset.

## Connections
- [Sets, relations, functions](sets-relations-functions.md) — partial orders are relations with three defining properties.
- [Boolean algebra](../09-digital-logic/boolean-algebra-and-minimization.md) — Boolean lattices model logic and set operations.

## Practice
**Q1.** In a power set ordered by inclusion, find meet and join of $\{a,c\}$ and $\{b,c\}$.
<details><summary>Answer</summary> Meet $=\{c\}$; join $=\{a,b,c\}$.</details>

**Q2.** Can a finite poset have two maximal elements and no greatest element?
<details><summary>Answer</summary> **Yes.** Two incomparable top elements are both maximal; neither is above the other.</details>

**Q3.** What property does a lattice require for each pair?
<details><summary>Answer</summary> A unique greatest lower bound and a unique least upper bound.</details>
