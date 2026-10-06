# Sets, relations, and functions

> **Paper:** CS · **Priority:** P0 · **Plan topics:** sets, relations, equivalence relations, partial orders, functions
> **Prerequisites:** [Propositional logic](propositional-logic.md) · **Leads to:** [Posets and lattices](posets-and-lattices.md) · [Graph theory](graph-theory.md)

## Quick glance
- $A\subseteq B$ means every element of A is in B; equality requires both inclusions.
- $|A\cup B|=|A|+|B|-|A\cap B|$.
- A relation on A is a subset of $A\times A$.
- Equivalence relation = reflexive + symmetric + transitive; it partitions A into equivalence classes.
- Partial order = reflexive + antisymmetric + transitive.
- Function $f:A\to B$ assigns exactly one output in B to every input in A; injective means no collisions, surjective means every B value is hit.

## 1. Sets and counting
The power set $\mathcal P(A)$ contains all subsets; if $|A|=n$, then $|\mathcal P(A)|=2^n$. Example: for $A=\{a,b\}$, $\mathcal P(A)=\{\emptyset,\{a\},\{b\},\{a,b\}\}$.

For three sets, inclusion–exclusion adds singles, subtracts pairwise intersections, then adds the triple intersection. This avoids double-counting elements appearing in multiple sets.

## 2. Relations
For $R\subseteq A\times A$, check properties directly from ordered pairs. On integers, $aRb$ iff $a-b$ is divisible by 3 is reflexive, symmetric, and transitive, so it is an equivalence relation. Its classes are residues modulo 3.

A partial order allows incomparable elements. On subsets of $\{1,2\}$, inclusion orders $\emptyset$ below both singletons, while the two singletons are incomparable.

## 3. Functions
For finite A and B, a function has $|B|^{|A|}$ possibilities. If $|A|=3$ and $|B|=2$, there are $2^3=8$ functions. An injective function from a 3-element set to a 2-element set cannot exist (pigeonhole principle).

Composition $g\circ f$ means apply f first, then g. If $f(x)=2x$ and $g(x)=x+1$, then $(g\circ f)(3)=7$, whereas $(f\circ g)(3)=8$.

## GATE traps
- Antisymmetric does not mean “not symmetric”; it means $aRb$ and $bRa$ force $a=b$.
- Symmetric is about reversing a pair; transitive is about chaining two pairs.
- Injective and surjective are different; bijective means both.
- For a function, every domain element must have exactly one image.

## Connections
- [Posets and lattices](posets-and-lattices.md) — a partial order supplies the structure for bounds and lattices.
- [Graph theory](graph-theory.md) — relations can be drawn as directed edges.
- [Combinatorics](combinatorics.md) — function counts and inclusion–exclusion are counting tools.

## Practice
**Q1.** On integers define $aRb$ iff $a-b$ is even. Is R an equivalence relation?
<details><summary>Answer</summary> **Yes.** Difference 0 is even (reflexive), negation preserves evenness (symmetric), and sums of even differences are even (transitive).</details>

**Q2 (NAT).** How many functions from a 2-element set to a 3-element set?
<details><summary>Answer</summary> $3^2=9$.</details>

**Q3.** Is divisibility on positive integers a total order?
<details><summary>Answer</summary> **No.** 2 and 3 are incomparable: neither divides the other.</details>
