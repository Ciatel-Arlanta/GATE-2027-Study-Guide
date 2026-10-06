# First-Order Logic

> **Paper:** CS · **Priority:** P0 · **Plan topics:** First-order logic
> **Prerequisites:** [Propositional logic](propositional-logic.md) · **Leads to:** [Sets, relations and functions](sets-relations-functions.md) · [Logic and inference for AI](../17-artificial-intelligence/logic-and-inference.md)

## Quick glance

- FOL = propositional logic + **predicates** P(x, y), **quantifiers** ∀ (for all) and ∃ (there exists), over a stated **domain**.
- **Negation flips the quantifier and negates the body:** ¬∀x P ≡ ∃x ¬P; ¬∃x P ≡ ∀x ¬P.
- **∀ distributes over ∧, ∃ distributes over ∨.** ∀ does **not** distribute over ∨; ∃ does **not** distribute over ∧ (only one direction holds).
- **Order matters for mixed quantifiers:** ∃x ∀y P(x,y) → ∀y ∃x P(x,y) is valid; the converse is not.
- **English to FOL:** "All A are B" = ∀x (A(x) → B(x)). "Some A are B" = ∃x (A(x) ∧ B(x)). **∀ pairs with →, ∃ pairs with ∧.**
- A variable is **bound** if inside the scope of a quantifier on it, otherwise **free**. A formula with no free variables is a **sentence** with a definite truth value in each interpretation.
- ∀x P(x) on an **empty domain** is true; ∃x P(x) on an empty domain is false.
- FOL validity is **undecidable** (but semi-decidable); propositional validity is decidable.

## 1. Predicates, domains and quantifiers

**Intuition.** "x is even" is not true or false until x is given a value. A **predicate** P(x) is a function from the domain to {true, false}. Quantifiers convert a predicate into a proposition:

- **∀x P(x)**: P holds for every element of the domain (a big AND over the domain).
- **∃x P(x)**: P holds for at least one element (a big OR over the domain).

**Interpretation.** To give meaning to a formula we fix a **domain** D, assign each predicate symbol a relation on D, each function symbol a function, each constant an element. A sentence is **valid** if true in every interpretation, **satisfiable** if true in some, **unsatisfiable** if true in none.

**Worked example 1.** Domain D = {1, 2, 3}, P(x) = "x is odd".
- ∀x P(x): P(2) is false, so **false**.
- ∃x P(x): P(1) is true, so **true**.
- ∃x ¬P(x) ≡ ¬∀x P(x): P(2) false, so true; consistent with the first line.

### Free and bound variables

In ∀x (P(x, y) → ∃z Q(z, x)):
- x is bound by ∀x, z is bound by ∃z, y is **free**.
- The scope of ∀x is the whole parenthesised body; the scope of ∃z is Q(z, x) only.
- The same letter can be bound in one part and free in another: P(x) ∧ ∀x Q(x): the first x is free, the second bound.
- Renaming a bound variable does not change meaning: ∀x P(x) ≡ ∀y P(y). **Free variables cannot be renamed freely.**

## 2. Negation of quantified statements

| Statement | Negation |
|-----------|----------|
| ∀x P(x) | ∃x ¬P(x) |
| ∃x P(x) | ∀x ¬P(x) |
| ∀x ∃y P(x,y) | ∃x ∀y ¬P(x,y) |
| ∃x ∀y P(x,y) | ∀x ∃y ¬P(x,y) |
| ∀x (P(x) → Q(x)) | ∃x (P(x) ∧ ¬Q(x)) |
| ∃x (P(x) ∧ Q(x)) | ∀x (P(x) → ¬Q(x)) |

**Rule:** push ¬ to the right; every quantifier it crosses flips (∀ ↔ ∃), then negate the body (using ¬(p → q) ≡ p ∧ ¬q and De Morgan).

**Worked example 2.** Negate "every student has a friend who is a topper": ∀s ∃f (Friend(s,f) ∧ Topper(f)).
1. ¬∀s ∃f (...) ≡ ∃s ¬∃f (...).
2. ≡ ∃s ∀f ¬(Friend(s,f) ∧ Topper(f)).
3. ≡ ∃s ∀f (Friend(s,f) → ¬Topper(f)).
In English: **some student has no friend who is a topper** (every friend of theirs is a non-topper).

## 3. Nested quantifiers and order

**Intuition.** ∀x ∃y P(x,y): for each x, choose a y that may **depend on x**. ∃y ∀x P(x,y): a **single** y that works for all x. The second is stronger.

**Worked example 3.** Domain = integers, P(x,y) = (x + y = 0).
- ∀x ∃y (x + y = 0): true, take y = −x.
- ∃y ∀x (x + y = 0): false; no single y works for all x.
Same predicate, different quantifier order, different truth value.

**Valid direction.** (∃x ∀y P(x,y)) → (∀y ∃x P(x,y)) is **valid**: if one x₀ works for all y, then for any y we can pick x = x₀. The reverse implication is **not valid** (the integer example above: LHS true, RHS false when swapped).

| Pair | Relationship |
|------|--------------|
| ∀x ∀y P  ≡  ∀y ∀x P | equivalent (same-type quantifiers commute) |
| ∃x ∃y P  ≡  ∃y ∃x P | equivalent |
| ∃x ∀y P → ∀y ∃x P | valid |
| ∀y ∃x P → ∃x ∀y P | **not** valid |

Brute-force check on all binary relations over a 3-element domain: the ∃∀ ⇒ ∀∃ implication holds in every one of the 512 interpretations, and the converse fails in some.

## 4. Distribution of quantifiers over ∧ and ∨

| Law | Holds? |
|-----|--------|
| ∀x (P ∧ Q) ≡ ∀x P ∧ ∀x Q | **Equivalence** |
| ∃x (P ∨ Q) ≡ ∃x P ∨ ∃x Q | **Equivalence** |
| ∀x P ∨ ∀x Q → ∀x (P ∨ Q) | Valid (one direction only) |
| ∀x (P ∨ Q) → ∀x P ∨ ∀x Q | **Invalid** |
| ∃x (P ∧ Q) → ∃x P ∧ ∃x Q | Valid (one direction only) |
| ∃x P ∧ ∃x Q → ∃x (P ∧ Q) | **Invalid** |
| ∀x (P → Q) → (∀x P → ∀x Q) | Valid |
| (∀x P → ∀x Q) → ∀x (P → Q) | **Invalid** |
| ∃x (P → Q) ≡ (∀x P → ∃x Q) | Equivalence |

**Counterexamples (domain = integers or {1,2}).**
- ∀x (P ∨ Q) but not (∀x P ∨ ∀x Q): P(x) = "x is even", Q(x) = "x is odd". Every integer is even or odd, but not all are even and not all are odd.
- ∃x P ∧ ∃x Q but not ∃x (P ∧ Q): same P, Q. There is an even number and an odd number, but no number that is both.
- (∀x P → ∀x Q) true but ∀x (P → Q) false: domain {1,2}, P(1) = T, P(2) = F, Q(1) = F, Q(2) = F. ∀x P is false so the left side is true; but P(1) → Q(1) is false.

All rows were verified by brute force over a 3-element domain with every pair of unary predicates.

**Quantifiers and a variable not in the body.** If x does not occur free in B: ∀x (A(x) ∨ B) ≡ (∀x A(x)) ∨ B and ∃x (A(x) ∧ B) ≡ (∃x A(x)) ∧ B. This lets you pull a quantifier out of one disjunct/conjunct.

## 5. Translating English to FOL (GATE favourite)

**The two patterns.**

| English | FOL | Common wrong form |
|---------|-----|-------------------|
| All A are B | ∀x (A(x) → B(x)) | ∀x (A(x) ∧ B(x)) (says everything is A and B) |
| Some A are B | ∃x (A(x) ∧ B(x)) | ∃x (A(x) → B(x)) (true if anything is not A) |
| No A is B | ∀x (A(x) → ¬B(x)) ≡ ¬∃x (A(x) ∧ B(x)) | |
| Some A are not B | ∃x (A(x) ∧ ¬B(x)) | |
| Not all A are B | ∃x (A(x) ∧ ¬B(x)) | |
| Only A are B | ∀x (B(x) → A(x)) | |

**Why ∀ goes with → and ∃ with ∧.** ∀x (A ∧ B) forces every element of the domain to be an A, which is not what "all A are B" says. ∃x (A → B) is true as soon as one non-A element exists, which is far too weak.

**Worked example 4.** Predicates: Student(x), Course(y), Takes(x,y), Hard(y).
1. "Every student takes some course": ∀x (Student(x) → ∃y (Course(y) ∧ Takes(x,y))).
2. "There is a course that every student takes": ∃y (Course(y) ∧ ∀x (Student(x) → Takes(x,y))).
3. "No student takes a hard course": ∀x ∀y ((Student(x) ∧ Hard(y)) → ¬Takes(x,y)).
4. "Some student takes only hard courses": ∃x (Student(x) ∧ ∀y ((Course(y) ∧ Takes(x,y)) → Hard(y))).

**Worked example 5 (typical GATE option analysis).** "Every person has a mother" with Person(x), Mother(y,x) = y is mother of x:
- ∀x (Person(x) → ∃y Mother(y,x)): correct.
- ∃y ∀x (Person(x) → Mother(y,x)): wrong; says one person is mother of everybody.

**Worked example 6 (counting uniqueness).** "There is exactly one x with P(x)": ∃x (P(x) ∧ ∀y (P(y) → y = x)). "At least two": ∃x ∃y (x ≠ y ∧ P(x) ∧ P(y)). "At most one": ∀x ∀y ((P(x) ∧ P(y)) → x = y).

**Worked example 7 (classic argument).** "All humans are mortal; Socrates is human; therefore Socrates is mortal."
1. ∀x (H(x) → M(x)) (premise); H(s) (premise).
2. Instantiate x = s: H(s) → M(s). Modus ponens gives M(s). Valid.

## 6. Inference rules for quantifiers

| Rule | Form | Note |
|------|------|------|
| Universal instantiation (UI) | ∀x P(x) ⊢ P(c) for any c | |
| Universal generalisation (UG) | P(c) for arbitrary c ⊢ ∀x P(x) | c must be arbitrary, not a specific element |
| Existential instantiation (EI) | ∃x P(x) ⊢ P(c) for a **new** constant c | name must not appear earlier |
| Existential generalisation (EG) | P(c) ⊢ ∃x P(x) | |

**Common fallacy.** From ∃x P(x) and ∃x Q(x), you cannot instantiate both with the same c. From P(c) for one specific c you cannot generalise to ∀x.

## 7. Satisfiability and validity of FOL sentences

- **Valid:** true under every interpretation (e.g. ∀x P(x) → ∃x P(x), valid only if the domain is non-empty, which FOL assumes).
- **Satisfiable:** true under some interpretation, e.g. ∀x ∃y (x < y) is satisfiable (integers) but false for a finite ordered domain: **it has no finite model**.
- **Checking invalidity:** exhibit a small domain (often 2 elements) and predicate assignments making premises true and conclusion false.
- **Decidability:** validity of FOL sentences is **undecidable** (Church–Turing) but **semi-decidable**; satisfiability of a sentence is also undecidable. Restricted fragments (monadic predicates, propositional logic) are decidable. See [Turing machines and undecidability](../11-theory-of-computation/turing-machines-and-undecidability.md).

**Worked example 8.** Is ∀x ∃y R(x,y) → ∃y ∀x R(x,y) valid?
1. Look for a 2-element counterexample: D = {a, b}, R = {(a,b), (b,a)}.
2. LHS: for a take y = b; for b take y = a. True.
3. RHS: y = a fails at x = a (R(a,a) false); y = b fails at x = b. False.
4. **Not valid.** (Same shape as "everybody has a successor" vs "somebody is everyone's successor".)

**Worked example 9.** Is (∃x P(x)) ∧ (∃x Q(x)) → ∃x (P(x) ∧ Q(x)) valid? Counterexample: D = {1,2}, P true only at 1, Q true only at 2. LHS true, RHS false. **Not valid.**

**Worked example 10.** Is ∃x (P(x) → Q(x)) ≡ (∀x P(x) → ∃x Q(x)) true?
- ∃x (¬P(x) ∨ Q(x)) ≡ ∃x ¬P(x) ∨ ∃x Q(x) (∃ distributes over ∨) ≡ ¬∀x P(x) ∨ ∃x Q(x) ≡ (∀x P(x) → ∃x Q(x)). **Yes.** (Brute-force confirmed.)

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|------|--------------|-------------|
| Negate ∀ | ¬∀x P ≡ ∃x ¬P | pushing negation |
| Negate ∃ | ¬∃x P ≡ ∀x ¬P | pushing negation |
| All A are B | ∀x (A → B) | translation |
| Some A are B | ∃x (A ∧ B) | translation |
| ∀ over ∧, ∃ over ∨ | equivalences | simplification |
| ∃∀ → ∀∃ | valid; converse invalid | ordering questions |
| ∀x P ∨ ∀x Q → ∀x (P ∨ Q) | valid; converse invalid | distribution |
| ∃x (P ∧ Q) → ∃x P ∧ ∃x Q | valid; converse invalid | distribution |
| Empty domain | ∀ true, ∃ false | edge cases |
| Valid FOL sentences | undecidable, semi-decidable | theory of computation link |

## GATE traps

- **∀x (A(x) ∧ B(x)) is not "all A are B".** Use → under ∀ and ∧ under ∃.
- **Quantifier order:** ∀x ∃y and ∃y ∀x differ; only ∃∀ ⇒ ∀∃ is valid.
- **∀ over ∨ and ∃ over ∧ do not distribute** both ways; give the "even/odd" counterexample.
- Negating "All A are B" gives "Some A is not B", **not** "No A is B".
- Free variables: a formula with a free variable is not a statement; check scope carefully when the same letter repeats.
- "Only" reverses the arrow: "only A are B" is ∀x (B → A).
- Many options differ just by a missing negation; evaluate on a tiny domain (2 elements) to eliminate options quickly.
- Equality needs explicit handling ("exactly one", "at least two") with ∃x ∃y (x ≠ y ∧ ...).
- A statement being true in one domain does not make it valid; validity needs every interpretation.

## Connections

- [Propositional logic](propositional-logic.md) — ∀ is a (possibly infinite) conjunction and ∃ a disjunction; De Morgan generalises to quantifiers.
- [Sets, relations and functions](sets-relations-functions.md) — properties like reflexive, symmetric, transitive are FOL sentences: ∀x R(x,x), ∀x∀y (R(x,y) → R(y,x)), and so on.
- [Posets and lattices](posets-and-lattices.md) — "greatest element" is ∃x ∀y (y ≤ x); "maximal" is ∃x ∀y (x ≤ y → y = x). Quantifier order is the difference.
- [Algebraic structures](algebraic-structures.md) — group axioms are FOL: ∃e ∀a (e·a = a), ∀a ∃b (a·b = e).
- [Limits, continuity, differentiability](../04-calculus-optimization/limits-continuity-differentiability.md) — the ε–δ definition is nested quantifiers (∀ε ∃δ ∀x); swapping order changes continuity into uniform continuity.
- [Relational model, algebra and calculus](../14-databases/relational-model-algebra-calculus.md) — tuple relational calculus is FOL; safe expressions mirror finite domains.
- [Logic and inference for AI](../17-artificial-intelligence/logic-and-inference.md) — Skolemisation, unification and resolution in predicate logic.
- [Turing machines and undecidability](../11-theory-of-computation/turing-machines-and-undecidability.md) — FOL validity is undecidable.

## Practice

**Q1 (MCQ).** The negation of ∀x (P(x) → Q(x)) is: (a) ∀x (P(x) ∧ ¬Q(x)) (b) ∃x (P(x) ∧ ¬Q(x)) (c) ∃x (¬P(x) → Q(x)) (d) ∀x (¬P(x) → ¬Q(x))

<details><summary>Answer</summary>

**Answer:** (b)
**Solution:** ¬∀x (P → Q) ≡ ∃x ¬(P → Q) ≡ ∃x (P ∧ ¬Q).

</details>

**Q2 (MCQ).** "Some birds cannot fly" with B(x) = bird, F(x) = flies: (a) ∃x (B(x) ∧ ¬F(x)) (b) ∃x (B(x) → ¬F(x)) (c) ∀x (B(x) → ¬F(x)) (d) ¬∀x (B(x) ∧ F(x))

<details><summary>Answer</summary>

**Answer:** (a)
**Solution:** "Some" uses ∧. (b) is true whenever a non-bird exists. (c) says no bird flies. (d) says not everything is a flying bird, much weaker.

</details>

**Q3 (MSQ).** Which are valid? (a) ∀x (P ∧ Q) → ∀x P ∧ ∀x Q (b) ∀x (P ∨ Q) → ∀x P ∨ ∀x Q (c) ∃x (P ∧ Q) → ∃x P ∧ ∃x Q (d) ∃x P ∧ ∃x Q → ∃x (P ∧ Q)

<details><summary>Answer</summary>

**Answer:** (a), (c)
**Solution:** (a) and (c) are the valid directions. (b): P = even, Q = odd over integers. (d): same P, Q: both exist but none is both.

</details>

**Q4 (MCQ).** Over the integers, which is true? (a) ∃y ∀x (x + y = x) (b) ∀x ∃y (x · y = 1) (c) ∃x ∀y (y < x) (d) ∀x ∃y (y < x) is false

<details><summary>Answer</summary>

**Answer:** (a)
**Solution:** (a) y = 0 works for every x. (b) fails at x = 0 (and for x = 2 no integer y). (c) no greatest integer. (d) is false as a claim because ∀x ∃y (y < x) is true (take y = x − 1), so the option "is false" is itself wrong.

</details>

**Q5 (MCQ).** Which sentence says "there are exactly two elements with P"? (a) ∃x ∃y (x ≠ y ∧ P(x) ∧ P(y)) (b) ∃x ∃y (x ≠ y ∧ P(x) ∧ P(y) ∧ ∀z (P(z) → (z = x ∨ z = y))) (c) ∃x ∃y (P(x) ∧ P(y)) (d) ∀x ∀y (P(x) ∧ P(y) → x ≠ y)

<details><summary>Answer</summary>

**Answer:** (b)
**Solution:** (a) is "at least two"; (c) is satisfied by x = y, so only "at least one". (d) is false whenever some element satisfies P (take x = y). (b) adds the bound on all other elements.

</details>

**Q6 (NAT).** Over a domain with 2 elements and a single binary predicate R, how many interpretations (relations R) make ∀x ∃y R(x,y) true?

<details><summary>Answer</summary>

**Answer:** 9
**Solution:** A relation on {a,b} has 4 possible pairs, 16 relations. Each element needs at least one outgoing pair: each row (2 pairs) must be non-empty: 3 choices per row (not 00), so 3 × 3 = 9.

</details>

**Q7 (MCQ).** Which implication is **not** valid? (a) ∃x ∀y R(x,y) → ∀y ∃x R(x,y) (b) ∀x ∀y R → ∀y ∀x R (c) ∀x R(x,x) → ∀x ∃y R(x,y) (d) ∀x ∃y R(x,y) → ∃y ∀x R(x,y)

<details><summary>Answer</summary>

**Answer:** (d)
**Solution:** (a) valid, (b) same-type quantifiers commute, (c) take y = x. (d) fails on R = {(a,b),(b,a)} over {a,b}.

</details>

**Q8 (MCQ).** "No student failed" negated: (a) Every student failed (b) Some student failed (c) Some student passed (d) No student passed

<details><summary>Answer</summary>

**Answer:** (b)
**Solution:** "No student failed" = ¬∃x (S(x) ∧ F(x)); its negation is ∃x (S(x) ∧ F(x)), some student failed. (a) is far too strong.

</details>
