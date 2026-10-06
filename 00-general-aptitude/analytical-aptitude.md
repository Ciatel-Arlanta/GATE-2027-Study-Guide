# Analytical aptitude: deduction and structured reasoning

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** deduction, induction, analogy, numerical relations, reasoning puzzles
> **Prerequisites:** [Verbal aptitude](verbal-aptitude.md) · **Leads to:** [Propositional logic](../01-discrete-mathematics/propositional-logic.md)

## Quick glance
- Deduction: if premises are true, a valid conclusion must be true.
- Do not infer the converse: “if P then Q” does not imply “if Q then P.”
- For arrangements, convert each sentence into a constraint and draw slots.
- For truth/lie puzzles, test each candidate and reject contradictions.
- In analogies, state the relation in words before checking options.

## 1. Deduction versus induction
Deduction applies a rule to a case: all registered candidates receive an admit card; Mira is registered; therefore Mira receives one. Induction generalises from observations: several sampled metals expanded when heated, so perhaps metals expand when heated. Induction supports a likely rule, not a logically certain conclusion.

**Worked example.** “All analysts know SQL. Some programmers are analysts.” It follows that **some programmers know SQL**: take any member of the nonempty overlap. It does not follow that all programmers know SQL.

## 2. Conditional reasoning
“If P, then Q” is false only when P is true and Q is false. Its contrapositive, “if not Q, then not P,” is equivalent. The converse “if Q then P” and inverse “if not P then not Q” are not equivalent.

Example: If a number is divisible by 4, it is even. Contrapositive: if odd, it is not divisible by 4. But even does not imply divisible by 4 (6 is a counterexample).

## 3. Arrangements and constraints
For five people A–E in a line, “A is before B” means positions satisfy $pos(A)<pos(B)$. “Adjacent” means difference 1. Place the most restrictive relation first; enumerate remaining cases only when necessary.

Example: A, B, C sit in three seats; A is not at an end and B is left of C. A must be in the middle; then B left of C forces B-A-C. A diagram prevents accidentally treating “left of” as “immediately left of.”

## 4. Patterns and analogies
Check common transformations in order: constant difference, constant ratio, alternating subsequences, squares/cubes, digit operations, then a combination. In an analogy, preserve the direction and type of relationship: “seed : tree” is growth, while “tree : seed” reverses it.

## GATE traps
- “Some A are B” does not mean “all A are B” or “some B are all A.”
- One counterexample disproves a universal claim.
- “At least one” includes multiple; “exactly one” does not.
- A puzzle condition may be necessary without being sufficient; check all constraints.

## Connections
- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — truth tables formalise conditionals.
- [First-order logic](../01-discrete-mathematics/first-order-logic.md) — quantifiers distinguish “some” from “all.”
- [Quantitative aptitude](quantitative-aptitude.md) — represent numerical relations with equations.

## Practice
**Q1 (MCQ).** Given $P\to Q$ and $\neg Q$, what follows? (A) P (B) $\neg P$ (C) Q (D) nothing.
<details><summary>Answer</summary> **B.** Modus tollens / contrapositive.</details>

**Q2.** All roses are flowers. Some flowers fade quickly. Must some roses fade quickly?
<details><summary>Answer</summary> **No.** The quickly fading flowers could all be non-roses.</details>

**Q3.** Four tasks W, X, Y, Z: W before X; Y after X; Z before W. Give one valid order.
<details><summary>Answer</summary> **Z, W, X, Y** satisfies all constraints.</details>
