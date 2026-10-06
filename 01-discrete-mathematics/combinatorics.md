# Combinatorics: systematic counting

> **Paper:** CS · **Priority:** P0 · **Plan topics:** counting, permutations, combinations, pigeonhole principle, inclusion–exclusion
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Probability basics](../02-probability-statistics/probability-basics.md)

## Quick glance
- Product rule: successive independent choices multiply; sum rule: disjoint alternatives add.
- Permutations order matters: $P(n,r)=n!/(n-r)!$.
- Combinations order does not: $\binom nr=n!/[r!(n-r)!]$.
- Repeated objects: $n!/(n_1!n_2!\dots)$ arrangements.
- Pigeonhole: placing n objects into k boxes forces some box to contain at least $\lceil n/k\rceil$ objects.
- Inclusion–exclusion for two sets: $|A\cup B|=|A|+|B|-|A\cap B|$.

## 1. Product and sum rules
Three shirts and two trousers give $3\cdot2=6$ outfits. If you can travel by 4 bus routes or 3 train routes and choices are distinct alternatives, there are $4+3=7$ route choices.

## 2. Permutations and combinations
Choose and assign president and secretary from 5 people: $5\cdot4=20$. Choose an unordered two-person committee: $\binom52=10$. The factor of 2 difference is the two role assignments per pair.

For the word `LEVEL`, five letters include two Ls and two Es, giving $5!/(2!2!)=30$ distinct arrangements.

## 3. Inclusion–exclusion and pigeonhole
In a class of 30, 18 study C, 16 study Python, and 8 study both. At least one: $18+16-8=26$; neither: 4.

Among 13 people, at least two share a birth month because 12 months are boxes: $\lceil13/12\rceil=2$.

## GATE traps
- If repetition is allowed, choices may not decrease from one position to the next.
- Circular arrangements of n distinct people: $(n-1)!$ when rotations are considered identical.
- “At least one” often uses complement counting.
- State what distinguishes outcomes before deciding whether order matters.

## Connections
- [Probability basics](../02-probability-statistics/probability-basics.md) — equiprobable probability is a ratio of counts.
- [Quantitative aptitude](../00-general-aptitude/quantitative-aptitude.md) — same counting tools appear in GA.
- [Discrete distributions](../02-probability-statistics/discrete-distributions.md) — binomial coefficients count success patterns.

## Practice
**Q1 (NAT).** Number of 4-digit strings using digits 0–9 with repetition allowed?
<details><summary>Answer</summary> $10^4=10000$; strings may begin with zero.</details>

**Q2 (NAT).** Choose 3 from 8.
<details><summary>Answer</summary> $\binom83=56$.</details>

**Q3.** In 50 people, 32 know A, 27 know B, and 15 know both. How many know neither?
<details><summary>Answer</summary> Union $=32+27-15=44$; neither $=50-44=6$.</details>
