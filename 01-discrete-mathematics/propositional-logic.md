# Propositional Logic

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Propositional logic
> **Prerequisites:** none · **Leads to:** [First-order logic](first-order-logic.md) · [Boolean algebra and minimization](../09-digital-logic/boolean-algebra-and-minimization.md)

## Quick glance

- A **proposition** is a sentence that is exactly true or false. Connectives: NOT (¬), AND (∧), OR (∨), XOR (⊕), implication (→), biconditional (↔).
- **p → q is false in exactly one row: p = T, q = F.** Everything else is true (a false hypothesis makes the implication vacuously true).
- Formula with n variables has 2^n truth-table rows and there are 2^(2^n) distinct Boolean functions of n variables.
- **Tautology** = true in every row (valid); **contradiction** = false in every row (unsatisfiable); **contingency** = neither. **Satisfiable** = at least one true row.
- A formula is valid ⇔ its negation is unsatisfiable. Fast validity test: **assume the formula is false and try to force a contradiction.**
- p → q ≡ ¬p ∨ q ≡ ¬q → ¬p (contrapositive). The converse (q → p) and inverse (¬p → ¬q) are **not** equivalent to p → q but are equivalent to each other.
- {NAND} and {NOR} are each functionally complete; {∧, ∨} is **not**; {¬, ∧}, {¬, ∨}, {→, ¬}, {→, F} are.
- Rules of inference: modus ponens, modus tollens, hypothetical syllogism, disjunctive syllogism, resolution. **Affirming the consequent and denying the antecedent are fallacies.**
- DNF = OR of ANDs (read from the true rows); CNF = AND of ORs (read from the false rows).

## 1. Propositions and connectives

**Intuition.** A proposition is a declarative statement with a definite truth value. "2 + 2 = 5" is a proposition (false); "x + 1 = 3" and "close the door" are not (open sentence, command).

Truth table of all the basic connectives (T = 1, F = 0):

| p | q | ¬p | p ∧ q | p ∨ q | p ⊕ q | p → q | p ↔ q | p NAND q | p NOR q |
|---|---|----|-------|-------|-------|-------|-------|----------|---------|
| 1 | 1 | 0 | 1 | 1 | 0 | 1 | 1 | 0 | 0 |
| 1 | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 | 1 | 1 | 0 | 1 | 0 |
| 0 | 0 | 1 | 0 | 0 | 0 | 1 | 1 | 1 | 1 |

**Reading implication.** "p → q" is read "if p then q", "p only if q", "q if p", "q whenever p", "p is sufficient for q", "q is necessary for p". **"p only if q" is p → q, not q → p** (a classic trap).

**Biconditional.** p ↔ q ≡ (p → q) ∧ (q → p) ≡ ¬(p ⊕ q). It is true when p and q have the same truth value. "p if and only if q", "p is necessary and sufficient for q".

**Precedence** (high to low): ¬, ∧, ∨, →, ↔. Implication is right-associative: p → q → r means p → (q → r).

### Related implications

For p → q:

| Name | Form | Equivalent to p → q? |
|------|------|----------------------|
| Converse | q → p | No |
| Inverse | ¬p → ¬q | No |
| Contrapositive | ¬q → ¬p | **Yes** |

The converse and the inverse are contrapositives of each other, hence equivalent to each other.

**Worked example 1.** "If it rains, the match is cancelled." p = rains, q = cancelled.
- Converse: if the match is cancelled, it rains (not implied; maybe the ground is flooded).
- Inverse: if it does not rain, the match is not cancelled.
- Contrapositive: if the match is not cancelled, it did not rain. Logically identical to the original.

## 2. Tautology, contradiction, satisfiability, equivalence

| Class | Definition | Example |
|-------|------------|---------|
| Tautology (valid) | true under every assignment | p ∨ ¬p |
| Contradiction (unsatisfiable) | false under every assignment | p ∧ ¬p |
| Contingency | true under some, false under others | p → q |
| Satisfiable | at least one assignment makes it true | any tautology or contingency |

**Key relations.**
- F is valid ⇔ ¬F is unsatisfiable.
- F is unsatisfiable ⇔ ¬F is valid.
- Tautologies are satisfiable; contradictions are not valid. A contingency is both satisfiable and falsifiable.
- **Logical equivalence** A ≡ B means A ↔ B is a tautology (same truth table).
- **Logical consequence** A ⊨ B means every assignment making A true makes B true, i.e. A → B is a tautology.

**Worked example 2 (full truth table).** Is ((p → q) ∧ (q → r)) → (p → r) a tautology? (Hypothetical syllogism.)

| p | q | r | p→q | q→r | (p→q)∧(q→r) | p→r | whole |
|---|---|---|-----|-----|-------------|-----|-------|
| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 1 | 1 | 0 | 1 | 0 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 0 | 1 | 0 | 0 | 1 |
| 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0 | 1 | 0 | 1 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 1 | 1 | 1 | 1 | 1 |
| 0 | 0 | 0 | 1 | 1 | 1 | 1 | 1 |

All eight rows give 1, so it is a **tautology** (verified by brute force).

### Fast validity check: assume the conclusion is false

To prove A → B is a tautology, **try to make it false**: A = T and B = F. If this forces a contradiction in some variable, no falsifying row exists, so the formula is valid. If you find a consistent assignment, that assignment is a counterexample.

**Worked example 3.** Is ((p ∨ q) ∧ (¬p ∨ r)) → (q ∨ r) valid? (This is resolution.)
1. Falsify the conclusion: q = F and r = F.
2. Then (¬p ∨ r) = ¬p, which must be T, so p = F.
3. Then (p ∨ q) = F ∨ F = F, but the premise needs it to be T. Contradiction.
4. No falsifying row exists, so the formula is a **tautology**.

**Worked example 4.** Is (p → q) → (q → p) valid? Falsify: q → p false needs q = T, p = F. Then p → q = F → T = T, so the whole thing is T → F = F. Consistent: counterexample p = F, q = T. **Not valid** (converse error), but satisfiable.

**Worked example 5 (counting satisfying rows).** How many assignments satisfy (p → q) ∧ (q → r) ∧ (r → p)?
- The three implications form a cycle p → q → r → p, so all three variables must be equal.
- Satisfying rows: (1,1,1) and (0,0,0). **Answer: 2.** (Brute-force check confirms 2.)

**Worked example 6.** Count rows making (p → q) → r true.
- It is false when (p → q) = T and r = F. p → q is true in 3 of 4 (p, q) rows; with r = F that is 3 rows of 8.
- True rows = 8 − 3 = **5**. (Verified.)

Compare p → (q → r), which is false only at p = T, q = T, r = F: 7 true rows. Hence **→ is not associative**.

## 3. Equivalence laws

| Law | Statement |
|-----|-----------|
| Identity | p ∧ T ≡ p; p ∨ F ≡ p |
| Domination | p ∨ T ≡ T; p ∧ F ≡ F |
| Idempotent | p ∨ p ≡ p; p ∧ p ≡ p |
| Double negation | ¬¬p ≡ p |
| Commutative | p ∨ q ≡ q ∨ p; p ∧ q ≡ q ∧ p |
| Associative | (p ∨ q) ∨ r ≡ p ∨ (q ∨ r); same for ∧ |
| Distributive | p ∧ (q ∨ r) ≡ (p ∧ q) ∨ (p ∧ r); p ∨ (q ∧ r) ≡ (p ∨ q) ∧ (p ∨ r) |
| De Morgan | ¬(p ∧ q) ≡ ¬p ∨ ¬q; ¬(p ∨ q) ≡ ¬p ∧ ¬q |
| Absorption | p ∨ (p ∧ q) ≡ p; p ∧ (p ∨ q) ≡ p |
| Negation | p ∨ ¬p ≡ T; p ∧ ¬p ≡ F |
| Implication | p → q ≡ ¬p ∨ q ≡ ¬q → ¬p |
| Negated implication | ¬(p → q) ≡ p ∧ ¬q |
| Biconditional | p ↔ q ≡ (p → q) ∧ (q → p) ≡ (p ∧ q) ∨ (¬p ∧ ¬q) |
| Exportation | (p ∧ q) → r ≡ p → (q → r) |
| Disjunction in antecedent | (p ∨ q) → r ≡ (p → r) ∧ (q → r) |
| Conjunction in consequent | p → (q ∧ r) ≡ (p → q) ∧ (p → r) |
| XOR | p ⊕ q ≡ (p ∨ q) ∧ ¬(p ∧ q) ≡ ¬(p ↔ q) |

**Worked example 7.** Simplify ¬(p ∨ (¬p ∧ q)).
1. ¬(p ∨ (¬p ∧ q)) ≡ ¬p ∧ ¬(¬p ∧ q) (De Morgan).
2. ≡ ¬p ∧ (p ∨ ¬q) (De Morgan, double negation).
3. ≡ (¬p ∧ p) ∨ (¬p ∧ ¬q) (distributive) ≡ F ∨ (¬p ∧ ¬q) ≡ **¬p ∧ ¬q**.
Check at p = 0, q = 0: original = ¬(0 ∨ (1 ∧ 0)) = 1; simplified = 1. ✓.

**Worked example 8.** Show p → (q → p) is a tautology. p → (q → p) ≡ ¬p ∨ (¬q ∨ p) ≡ (¬p ∨ p) ∨ ¬q ≡ T ∨ ¬q ≡ **T**.

## 4. Functional completeness

A set of connectives is **functionally complete** if every Boolean function can be built from it.

- Every Boolean function is a DNF (OR of ANDs of literals), so {¬, ∧, ∨} is complete.
- De Morgan removes ∨ or ∧, so {¬, ∧} and {¬, ∨} are complete.
- p → q ≡ ¬p ∨ q, so {¬, →} is complete. {→, F} is complete (¬p ≡ p → F).

**NAND and NOR are universal (single-connective complete sets).**

| Target | Using NAND (↑) | Using NOR (↓) |
|--------|----------------|---------------|
| ¬p | p ↑ p | p ↓ p |
| p ∧ q | (p ↑ q) ↑ (p ↑ q) | (p ↓ p) ↓ (q ↓ q) |
| p ∨ q | (p ↑ p) ↑ (q ↑ q) | (p ↓ q) ↓ (p ↓ q) |

**Not complete.** A set is incomplete if every connective in it preserves some property. {∧, ∨} and even {∧, ∨, →, ↔} all output T when every input is T, so none can build ¬p (which outputs F on T) or the constant F. **{∧, ∨, →, ↔} without ¬ or F is not complete.** GATE usually asks about subsets of {¬, ∧, ∨, →} and about NAND/NOR.

**Number of 2-input Boolean functions:** 2^(2^2) = 16. Of these, 8 are commutative and 8 are associative (brute-force verified). NAND is commutative but **not associative** (verified).

## 5. Normal forms

A **literal** is a variable or its negation. A **minterm** is an AND of literals containing every variable once; a **maxterm** is an OR of literals containing every variable once.

- **DNF** (sum of products): OR of ANDs of literals. **Canonical DNF**: OR of minterms, one per **true** row.
- **CNF** (product of sums): AND of ORs of literals. **Canonical CNF**: AND of maxterms, one per **false** row (the maxterm is the negation of that row's minterm).

**Worked example 9.** Find the canonical DNF and CNF of f = (p → q) → r (the formula counted in Example 6).

| p | q | r | p→q | (p→q)→r |
|---|---|---|-----|---------|
| 0 | 0 | 0 | 1 | 0 |
| 0 | 0 | 1 | 1 | 1 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 | 1 |

True rows: 001, 011, 100, 101, 111 (five). False rows: 000, 010, 110 (three).
- Canonical DNF: (¬p ∧ ¬q ∧ r) ∨ (¬p ∧ q ∧ r) ∨ (p ∧ ¬q ∧ ¬r) ∨ (p ∧ ¬q ∧ r) ∨ (p ∧ q ∧ r).
- Canonical CNF (one maxterm per false row; row 000 gives p ∨ q ∨ r; row 010 gives p ∨ ¬q ∨ r; row 110 gives ¬p ∨ ¬q ∨ r): (p ∨ q ∨ r) ∧ (p ∨ ¬q ∨ r) ∧ (¬p ∨ ¬q ∨ r).
- Simplify: the first two maxterms merge to (p ∨ r); f ≡ ¬(p → q) ∨ r ≡ (p ∧ ¬q) ∨ r ≡ (p ∨ r) ∧ (¬q ∨ r) by distribution. Check against the false rows 000, 010, 110: (p ∨ r) is F only for p = 0, r = 0 (rows 000, 010); (¬q ∨ r) is F for q = 1, r = 0 (rows 010, 110). Together they are false exactly on 000, 010, 110. ✓

**Facts.** Satisfiability of a DNF is easy (any term without a complementary pair works); satisfiability of a CNF is **SAT, NP-complete** (3-SAT also NP-complete, 2-SAT in polynomial time). Validity of a CNF is easy (each clause must contain p and ¬p); validity of a DNF is co-NP-complete. See [NP-completeness context in algorithms](../08-algorithms/README.md).

## 6. Rules of inference and arguments

An argument with premises P₁, …, Pₙ and conclusion C is **valid** iff (P₁ ∧ … ∧ Pₙ) → C is a tautology, i.e. no assignment makes all premises true and C false.

| Rule | Form | Valid? |
|------|------|--------|
| Modus ponens | p, p → q ⊢ q | Valid |
| Modus tollens | ¬q, p → q ⊢ ¬p | Valid |
| Hypothetical syllogism | p → q, q → r ⊢ p → r | Valid |
| Disjunctive syllogism | p ∨ q, ¬p ⊢ q | Valid |
| Addition | p ⊢ p ∨ q | Valid |
| Simplification | p ∧ q ⊢ p | Valid |
| Conjunction | p, q ⊢ p ∧ q | Valid |
| Resolution | p ∨ q, ¬p ∨ r ⊢ q ∨ r | Valid |
| Constructive dilemma | p → q, r → s, p ∨ r ⊢ q ∨ s | Valid |
| **Affirming the consequent** | q, p → q ⊢ p | **Fallacy** |
| **Denying the antecedent** | ¬p, p → q ⊢ ¬q | **Fallacy** |

**Worked example 10.** Premises: (1) p → q, (2) q → r, (3) ¬r ∨ s, (4) p. Conclude s?
- From (1) and (4) by modus ponens: q. From (2): r. From (3) with r true, ¬r is false, so s (disjunctive syllogism). **s follows; valid.**

**Worked example 11 (GATE style).** "Some A are B, all B are C, so some A are C" belongs to FOL syllogisms; see [First-order logic](first-order-logic.md). In pure propositional form, check the argument {p ∨ q, p → r, q → r} ⊢ r:
- Assume r = F. Then p → r forces p = F and q → r forces q = F, so p ∨ q = F, contradicting the first premise. **Valid** (proof by cases).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|------|--------------|-------------|
| Rows in truth table | 2^n | counting assignments |
| Boolean functions of n variables | 2^(2^n) (16 for n = 2, 256 for n = 3) | counting distinct formulas |
| p → q | ¬p ∨ q ≡ ¬q → ¬p | rewriting implication |
| ¬(p → q) | p ∧ ¬q | negating implication |
| p ↔ q | (p ∧ q) ∨ (¬p ∧ ¬q) | simplifying |
| XOR | p ⊕ q ≡ ¬(p ↔ q); associative and commutative | parity problems |
| De Morgan | ¬(p ∧ q) ≡ ¬p ∨ ¬q | pushing negation |
| Valid ⇔ ¬F unsatisfiable | | SAT reasoning |
| Universal connectives | NAND, NOR | gate-level questions |
| Canonical DNF / CNF | true rows / false rows | normal form questions |
| Satisfying rows of a conjunction of implications on a cycle | all variables equal: 2 rows | counting |

## GATE traps

- **"p only if q" means p → q**, not q → p. "p if q" means q → p.
- **The converse is not equivalent to the original.** Only the contrapositive is.
- A false hypothesis makes p → q **true**; "if 2 > 3 then the moon is cheese" is true.
- **→ is not associative** and not commutative; ↔ and ⊕ are both associative and commutative. NAND and NOR are commutative but not associative.
- A satisfiable formula is not necessarily valid; do not confuse "satisfiable" with "tautology".
- Check "equivalent" with a truth table or by checking A ↔ B is a tautology; checking one row proves nothing.
- Count of formulas: the number of **non-equivalent** formulas on n variables is 2^(2^n), not 2^n.
- {∧, ∨} and {∧, ∨, →} are not functionally complete; NAND alone is.
- Affirming the consequent and denying the antecedent are invalid even though they look like modus ponens/tollens.
- In "assume conclusion false" check, you need **all premises true simultaneously**; one consistent assignment is enough for invalidity.

## Connections

- [First-order logic](first-order-logic.md) — adds predicates and quantifiers on top of propositional connectives; all equivalence laws here still apply.
- [Boolean algebra and minimization](../09-digital-logic/boolean-algebra-and-minimization.md) — ∧, ∨, ¬ are AND, OR, NOT gates; minterms and maxterms become K-map cells; canonical DNF/CNF are SOP/POS.
- [Posets and lattices](posets-and-lattices.md) — Boolean algebra is a complemented distributive lattice; propositions up to equivalence form one.
- [Turing machines and undecidability](../11-theory-of-computation/turing-machines-and-undecidability.md) — SAT is the canonical NP-complete problem; validity of FOL is undecidable while propositional validity is decidable.
- [Logic and inference for AI](../17-artificial-intelligence/logic-and-inference.md) — resolution, CNF conversion and entailment checking generalise these ideas.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — truth-table checking is Θ(2^n), which is why SAT is hard.

## Practice

**Q1 (MCQ).** Which is a tautology? (a) (p → q) → (q → p) (b) (p ∧ (p → q)) → q (c) (p ∨ q) → p (d) p → ¬p

<details><summary>Answer</summary>

**Answer:** (b)
**Solution:** (b) is modus ponens: falsify q = F with p = T, then p → q = F so the antecedent is F, no falsifying row. (a) fails at p = F, q = T. (c) fails at p = F, q = T. (d) fails at p = T.

</details>

**Q2 (NAT).** How many rows of the truth table of (p → q) ∧ (q → r) ∧ (r → p) are true?

<details><summary>Answer</summary>

**Answer:** 2
**Solution:** The three implications form a cycle, so p, q, r must all be equal. Only (1,1,1) and (0,0,0) work.

</details>

**Q3 (MSQ).** Which are equivalent to p → q? (a) ¬q → ¬p (b) ¬p ∨ q (c) q → p (d) ¬(p ∧ ¬q)

<details><summary>Answer</summary>

**Answer:** (a), (b), (d)
**Solution:** (a) contrapositive, (b) definition, (d) De Morgan: ¬(p ∧ ¬q) ≡ ¬p ∨ q. (c) is the converse.

</details>

**Q4 (MCQ).** The statement "I will pass only if I study" is: (a) study → pass (b) pass → study (c) pass ↔ study (d) ¬study → pass

<details><summary>Answer</summary>

**Answer:** (b)
**Solution:** "A only if B" is A → B with A = pass, B = study.

</details>

**Q5 (NAT).** Number of true rows for (p → q) → r with three variables.

<details><summary>Answer</summary>

**Answer:** 5
**Solution:** False only when p → q is T and r = F: three (p,q) rows (00, 01, 11) with r = F. So 8 − 3 = 5.

</details>

**Q6 (MCQ).** Which set of connectives is NOT functionally complete? (a) {NAND} (b) {NOR} (c) {→, ¬} (d) {∧, ∨}

<details><summary>Answer</summary>

**Answer:** (d)
**Solution:** ∧ and ∨ map all-true inputs to true, so no formula built from them can compute ¬p (which gives F on input T).

</details>

**Q7 (MCQ).** Premises: p ∨ q, ¬p ∨ r, ¬q ∨ r. What follows? (a) p (b) q (c) r (d) ¬r

<details><summary>Answer</summary>

**Answer:** (c)
**Solution:** Assume r = F. Then ¬p ∨ r forces p = F and ¬q ∨ r forces q = F, so p ∨ q = F, contradiction. Hence r is true. p and q individually need not be true (p = T, q = F, r = T satisfies all premises, and so does the opposite).

</details>

**Q8 (NAT).** How many of the 16 binary Boolean functions (truth tables on two inputs) are commutative?

<details><summary>Answer</summary>

**Answer:** 8
**Solution:** Commutativity requires f(0,1) = f(1,0). Choose f(0,0), f(1,1) and the common value freely: 2³ = 8. (Brute force agrees.)

</details>
