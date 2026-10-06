# Logic and inference for AI: propositional and predicate logic

> **Paper:** DA · **Priority:** P1 · **Plan topics:** Propositional logic; Predicate logic
> **Prerequisites:** [Propositional logic](../01-discrete-mathematics/propositional-logic.md) · [First-order logic](../01-discrete-mathematics/first-order-logic.md) · [Search](search.md) · **Leads to:** [Probabilistic reasoning](probabilistic-reasoning.md)

> This chapter uses logic as a **knowledge-representation and inference** tool (entailment, resolution, chaining, unification). Syntax, equivalences, normal forms and quantifier rules are taught in full in the discrete-mathematics chapters linked above; here we focus on the algorithms.

## Quick glance

- A **knowledge base** KB is a set of sentences. **Entailment** $KB\models\alpha$: every model of KB is a model of $\alpha$. **Valid** = true in all models; **satisfiable** = true in some model; **unsatisfiable** = true in none.
- **Refutation principle**: $KB\models\alpha$ iff $KB\wedge\neg\alpha$ is unsatisfiable.
- **Model checking** by truth table: $2^n$ rows for $n$ symbols. Propositional SAT is NP-complete.
- **Inference rules**: modus ponens $\dfrac{\alpha\to\beta,\ \alpha}{\beta}$, modus tollens, and-elimination, resolution $\dfrac{\ell\vee A,\ \neg\ell\vee B}{A\vee B}$ (**sound and refutation-complete**).
- **CNF** conversion: eliminate $\leftrightarrow,\to$; push $\neg$ inward (De Morgan); distribute $\vee$ over $\wedge$.
- **Horn clause**: at most one positive literal. Forward chaining (data-driven) and backward chaining (goal-driven) are sound and complete for Horn KBs, in **linear time** for propositional Horn.
- **FOL**: quantifiers $\forall$, $\exists$; **unification** finds the **most general unifier (MGU)**; **Skolemisation** removes $\exists$; resolution works on clauses with unification. Entailment in FOL is **semi-decidable**.
- #1 trap: the connective for $\forall$ is $\to$, for $\exists$ is $\wedge$ ($\forall x(\text{Cat}(x)\to\text{Mammal}(x))$, $\exists x(\text{Cat}(x)\wedge\text{Black}(x))$).

## 1. Propositional logic as a knowledge representation

**Semantics.** A **model** assigns true/false to every symbol. A sentence is true in a model if its truth table says so.

| Notion | Definition | Test |
| --- | --- | --- |
| Valid (tautology) | true in every model | negation unsatisfiable |
| Satisfiable | true in at least one model | SAT |
| Contradiction | true in no model | |
| Entailment $KB\models\alpha$ | models(KB) $\subseteq$ models($\alpha$) | $KB\wedge\neg\alpha$ unsat |
| Equivalence | same models | $\alpha\models\beta$ and $\beta\models\alpha$ |

**Deduction theorem**: $KB\models\alpha$ iff $(\bigwedge KB)\to\alpha$ is valid.

**Worked example (model checking).** $KB=(P\to Q)\wedge(Q\to R)$. Does $KB\models(P\to R)$?
Enumerate the 8 models; those satisfying KB are $(P,Q,R)=(0,0,0),(0,0,1),(0,1,1),(1,1,1)$. In each, $P\to R$ holds ($P=0$ or $R=1$). So **yes**. Does $KB\models R$? $(0,0,0)$ is a model of KB with $R=0$: **no**.

**Counting models.** $(A\vee B)\wedge C$ over 3 symbols: $A\vee B$ holds in 3 of 4 assignments of $(A,B)$, and $C=1$: **3 models** out of 8.

**Soundness and completeness.** An inference procedure is **sound** if it derives only entailed sentences, **complete** if it derives every entailed sentence. Truth-table checking is both, but $O(2^n)$.

## 2. Inference by resolution

**Conjunctive normal form (CNF)**: a conjunction of clauses; a clause is a disjunction of literals.

**Conversion to CNF** (worked): $(P\leftrightarrow Q)\to R$.
1. Remove $\leftrightarrow$: $((P\to Q)\wedge(Q\to P))\to R$.
2. Remove $\to$: $\neg((\neg P\vee Q)\wedge(\neg Q\vee P))\vee R$.
3. De Morgan: $(\,(P\wedge\neg Q)\vee(Q\wedge\neg P)\,)\vee R$.
4. Distribute: $(P\vee Q\vee R)\wedge(P\vee\neg P\vee R)\wedge(\neg Q\vee Q\vee R)\wedge(\neg Q\vee\neg P\vee R)$. Drop tautological clauses (those containing $\ell\vee\neg\ell$): **$(P\vee Q\vee R)\wedge(\neg P\vee\neg Q\vee R)$** (checked by truth table).

**Resolution rule.** From clauses $C_1=(\ell\vee A)$ and $C_2=(\neg\ell\vee B)$ derive $(A\vee B)$. Resolving $\ell$ with $\neg\ell$ and nothing else left gives the **empty clause** $\square$ = contradiction.

**Resolution refutation algorithm.** To prove $KB\models\alpha$: put $KB\wedge\neg\alpha$ in CNF; repeatedly resolve pairs and add resolvents; stop when $\square$ is derived (entailed) or no new clauses appear (not entailed). **Refutation-complete**: if the set is unsatisfiable, resolution derives $\square$.

**Worked example.** $KB$: $P\to Q$, $Q\to R$, $P$. Prove $R$.
Clauses: (1) $\neg P\vee Q$, (2) $\neg Q\vee R$, (3) $P$, negated goal (4) $\neg R$.
- (1)+(3): $Q$ (5)
- (2)+(5): $R$ (6)
- (4)+(6): $\square$. **Proved.**

**Worked example 2 (harder).** $KB$: $A\vee B$, $\neg A\vee C$, $\neg B\vee C$. Prove $C$ (negate: $\neg C$).
- $A\vee B$ + $\neg A\vee C$: $B\vee C$
- $B\vee C$ + $\neg B\vee C$: $C$
- $C$ + $\neg C$: $\square$. **Proved** (case analysis on $A$ or $B$).

**Complexity.** The number of possible clauses is $\le 3^n$; resolution can take exponential time in the worst case (propositional inference is co-NP-complete).

## 3. Horn clauses and chaining

**Horn clause**: at most one positive literal. **Definite clause**: exactly one. Written as an implication $p_1\wedge\dots\wedge p_k\to q$ (a fact if $k=0$). Horn KBs are what rule-based systems use; entailment of an atom is decided in **linear time**.

**Forward chaining** (data-driven): keep a count of unsatisfied premises per rule and an agenda of derived facts; when a rule's count hits 0, add its head. Stops when the query is derived or the agenda is empty. Sound and complete for definite clauses.

**Worked example.** Rules: (r1) $P\to Q$; (r2) $L\wedge M\to P$; (r3) $B\wedge L\to M$; (r4) $A\wedge P\to L$; (r5) $A\wedge B\to L$. Facts: $A,B$. Query: $Q$.

| Agenda pop | Rule counters updated | New facts |
| --- | --- | --- |
| $A$ | r4: 2$\to$1; r5: 2$\to$1 | — |
| $B$ | r3: 2$\to$1; r5: 1$\to$0 fires | $L$ |
| $L$ | r2: 2$\to$1; r3: 1$\to$0 fires | $M$ |
| $M$ | r2: 1$\to$0 fires | $P$ |
| $P$ | r1: 1$\to$0 fires; r4: 1$\to$0 fires ($L$ already known) | $Q$ |
| $Q$ | goal reached | |

Inferred order: $A,B,L,M,P,Q$. **Query $Q$ is entailed.**

**Backward chaining** (goal-driven): to prove $q$, find rules with head $q$ and recursively prove the premises; avoid loops by tracking goals in progress. On the same KB: $Q\leftarrow P\leftarrow(L\wedge M)$. $L\leftarrow A\wedge P$ (loop: $P$ is already being proved, fail) or $L\leftarrow A\wedge B$ (both facts: success). $M\leftarrow B\wedge L$: $L$ known: success. So $P$ then $Q$ succeed.

| | Forward | Backward |
| --- | --- | --- |
| Direction | facts $\to$ conclusions | goal $\to$ facts |
| Work | may derive irrelevant facts | focused on the query |
| Cost | linear in KB size | often much less than linear |
| Example systems | production rules | Prolog |

## 4. Predicate (first-order) logic

**Syntax.** Constants (Socrates), variables ($x,y$), functions ($\text{Mother}(x)$), predicates ($\text{Man}(x)$, $\text{Loves}(x,y)$), connectives, quantifiers $\forall x$, $\exists x$, equality.

**Translation patterns.**

| English | FOL |
| --- | --- |
| All humans are mortal | $\forall x\,(\text{Human}(x)\to\text{Mortal}(x))$ |
| Some human is a student | $\exists x\,(\text{Human}(x)\wedge\text{Student}(x))$ |
| No student likes exams | $\forall x\,(\text{Student}(x)\to\neg\text{Likes}(x,\text{Exams}))$ |
| Everyone loves someone | $\forall x\,\exists y\,\text{Loves}(x,y)$ |
| Someone is loved by everyone | $\exists y\,\forall x\,\text{Loves}(x,y)$ (**stronger**) |

Quantifier order matters: $\forall x\exists y$ does not imply $\exists y\forall x$. Negation: $\neg\forall x\,P\equiv\exists x\,\neg P$; $\neg\exists x\,P\equiv\forall x\,\neg P$. See [first-order logic](../01-discrete-mathematics/first-order-logic.md) for the full treatment.

### 4.1 Unification

**Unifier**: a substitution $\theta$ making two expressions identical. **MGU**: the most general unifier (any other unifier is an instance of it); unique up to renaming. Rules: a variable unifies with any term not containing it (**occurs check**); constants unify only with the same constant; functions unify when function symbols match and arguments unify pairwise.

| Pair | MGU |
| --- | --- |
| $P(x,\,f(y))$ and $P(a,\,f(b))$ | $\{x/a,\ y/b\}$ |
| $\text{Knows}(\text{John},x)$ and $\text{Knows}(y,\text{Mother}(y))$ | $\{y/\text{John},\ x/\text{Mother}(\text{John})\}$ |
| $P(f(x),\,g(y))$ and $P(f(g(z)),\,g(h(a)))$ | $\{x/g(z),\ y/h(a)\}$ |
| $P(x,x)$ and $P(a,b)$ | **fails** ($a\ne b$) |
| $x$ and $f(x)$ | **fails** (occurs check) |
| $\text{Q}(x,\,y)$ and $\text{Q}(y,\,a)$ | $\{x/a,\ y/a\}$ |

*Worked (row 2).* Match arguments left to right: $\text{John}$ vs $y$: bind $y/\text{John}$. Then $x$ vs $\text{Mother}(y)$; apply the existing binding: $\text{Mother}(\text{John})$, bind $x/\text{Mother}(\text{John})$.
*Worked (row 6).* $x$ vs $y$: bind $x/y$; then $y$ vs $a$ (after substitution $y$ vs $a$): bind $y/a$; composing, $x/a$ too.

### 4.2 Converting to clausal form for resolution

1. Eliminate $\leftrightarrow$ and $\to$.
2. Move $\neg$ inward (including quantifier negation rules).
3. **Standardise variables apart** (each quantifier its own variable name).
4. **Skolemise**: replace each $\exists y$ by a **Skolem function** of the universally quantified variables in scope ($\exists y$ under $\forall x$ becomes $f(x)$; with none, a Skolem constant).
5. Drop the $\forall$ prefix (all remaining variables are universal).
6. Distribute to CNF and write each clause separately.

*Example.* $\forall x\,\exists y\,\text{Loves}(x,y)\ \Rightarrow\ \text{Loves}(x,f(x))$. And $\exists x\,\text{Rich}(x)\Rightarrow\text{Rich}(c)$ for a fresh constant $c$.

### 4.3 Resolution in FOL

Resolve $(\ell_1\vee A)$ and $(\neg\ell_2\vee B)$ when $\ell_1,\ell_2$ unify with MGU $\theta$: result $(A\vee B)\theta$.

**Worked example.** KB: (a) everyone loves someone; (b) anyone who is loved is happy. Prove: someone is happy.
- KB: $\forall x\exists y\,\text{Loves}(x,y)$; $\forall x\forall y\,(\text{Loves}(x,y)\to\text{Happy}(y))$. Goal: $\exists z\,\text{Happy}(z)$, negated: $\forall z\,\neg\text{Happy}(z)$.
- Clauses: (1) $\text{Loves}(x,f(x))$; (2) $\neg\text{Loves}(x,y)\vee\text{Happy}(y)$; (3) $\neg\text{Happy}(z)$.
- (2)+(3) with $\theta=\{z/y\}$: (4) $\neg\text{Loves}(x,y)$.
- (4)+(1) with $\theta=\{x/x',\ y/f(x')\}$ (variables standardised apart): $\square$. **Proved.**

**Classic.** $\forall x\,(\text{Man}(x)\to\text{Mortal}(x))$, $\text{Man}(\text{Socrates})$; prove $\text{Mortal}(\text{Socrates})$. Clauses $\neg\text{Man}(x)\vee\text{Mortal}(x)$, $\text{Man}(S)$, $\neg\text{Mortal}(S)$. Resolve the first with the third: $\theta=\{x/S\}$, giving $\neg\text{Man}(S)$; with the second: $\square$.

**Properties.** FOL resolution is sound and **refutation-complete** (Robinson; Herbrand's theorem). Entailment in FOL is **semi-decidable**: a proof is found if the sentence is entailed, but the search may run forever if it is not.

## 5. Comparing the logics

| | Propositional | First-order |
| --- | --- | --- |
| Objects | none (facts only) | objects, relations, functions |
| Expressiveness | weak; needs a copy per object | compact with quantifiers |
| Inference | decidable (SAT, NP-complete) | semi-decidable |
| Complete procedure | resolution, DPLL, truth tables | resolution with unification |

**DPLL** (brief): backtracking SAT solver using unit propagation and pure-literal elimination; the base of modern SAT solvers.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Entailment test | $KB\wedge\neg\alpha$ unsat | resolution proofs |
| Models in truth table | $2^n$ | counting |
| Resolution rule | $\ell\vee A,\ \neg\ell\vee B\vdash A\vee B$ | proofs |
| Horn clause | $\le1$ positive literal | choose chaining |
| Forward/backward chaining | linear time for propositional Horn | complexity MCQ |
| $\forall$ with $\to$, $\exists$ with $\wedge$ | translation | FOL formalisation |
| Skolemisation | $\exists y$ in scope of $\forall x\ \to\ f(x)$ | clausal form |
| MGU | most general substitution making terms equal | unification |
| Occurs check | $x$ cannot unify with $f(x)$ | failure cases |

## GATE traps

- Using $\wedge$ with $\forall$ ($\forall x\,(\text{Cat}(x)\wedge\text{Mammal}(x))$ says everything is a cat) and $\to$ with $\exists$ (trivially true for any non-cat).
- Quantifier order: $\forall x\exists y$ differs from $\exists y\forall x$.
- Resolution proves by contradiction: **negate the goal** and add it. Forgetting to negate is the most common slip.
- Resolve **one** complementary pair at a time; resolving two pairs at once gives a wrong (often tautological) clause.
- Skolem functions must depend on all enclosing universal variables; a Skolem constant is wrong if the $\exists$ is under a $\forall$.
- Unification: apply substitutions consistently, check the occurs check, and standardise apart variables of different clauses.
- Forward chaining derives everything derivable; backward chaining explores only goal-relevant rules. Both are complete only for Horn/definite KBs; general KBs need resolution.
- Entailment $\ne$ implication in the object language; but by the deduction theorem $KB\models\alpha$ iff $KB\to\alpha$ is valid.
- A valid sentence is entailed by every KB; an unsatisfiable KB entails everything.

## Connections

- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — syntax, normal forms, equivalences, satisfiability used throughout this chapter.
- [First-order logic](../01-discrete-mathematics/first-order-logic.md) — quantifier equivalences and translation practice.
- [Search](search.md) — inference is search over proofs; backward chaining is depth-first, resolution can be driven by uniform-cost/BFS strategies.
- [Probabilistic reasoning](probabilistic-reasoning.md) — probabilities extend entailment: logic is the special case with only 0/1 probabilities.
- [Boolean algebra](../09-digital-logic/boolean-algebra-and-minimization.md) — CNF/DNF and satisfiability are Boolean-function questions.
- [Turing machines and undecidability](../11-theory-of-computation/turing-machines-and-undecidability.md) — FOL validity is undecidable (semi-decidable), propositional is decidable.
- [Grammar and parsing](../12-compiler-design/parsing.md) — unification-like matching also appears in type inference and Prolog.

## Practice

**Q1 (NAT).** How many models (out of the total) satisfy $(P\to Q)$ over symbols $P,Q,R$?

<details><summary>Answer</summary>

**Answer:** 6. **Solution:** $P\to Q$ is false only when $P=1,Q=0$ (2 assignments of $R$): $8-2=6$.

</details>

**Q2 (MCQ).** $KB=\{A\to B,\ B\to C\}$. Which sentence is entailed? (a) $C$ (b) $A\to C$ (c) $A$ (d) $\neg A$.

<details><summary>Answer</summary>

**Answer:** (b). **Solution:** hypothetical syllogism: $A\to B,\ B\to C\models A\to C$. The model $(A,B,C)=(0,0,0)$ satisfies KB and falsifies $C$, so (a) fails; $(0,0,0)$ falsifies $A$ so (c) fails; $(1,1,1)$ satisfies KB and falsifies $\neg A$, so (d) fails.

</details>

**Q3 (MCQ).** Which is a Horn clause? (a) $\neg P\vee\neg Q\vee R$ (b) $P\vee Q$ (c) $P\vee Q\vee\neg R$ (d) $\neg P\vee Q\vee R$.

<details><summary>Answer</summary>

**Answer:** (a). **Solution:** exactly one positive literal ($R$), i.e. $P\wedge Q\to R$. The others have 2 positive literals.

</details>

**Q4 (NAT).** Resolve $(A\vee\neg B\vee C)$ and $(\neg C\vee D)$. How many literals does the resolvent have?

<details><summary>Answer</summary>

**Answer:** 3. **Solution:** resolve on $C$: $(A\vee\neg B\vee D)$.

</details>

**Q5 (MCQ).** MGU of $P(x,\,g(y))$ and $P(f(z),\,g(a))$ is (a) $\{x/f(z),\,y/a\}$ (b) $\{x/z,\,y/a\}$ (c) fails (d) $\{z/x,\,y/g(a)\}$.

<details><summary>Answer</summary>

**Answer:** (a). **Solution:** $x$ vs $f(z)$: $x/f(z)$; $g(y)$ vs $g(a)$: $y/a$.

</details>

**Q6 (MCQ).** "Every student likes some teacher" in FOL:

<details><summary>Answer</summary>

**Answer:** $\forall x\,(\text{Student}(x)\to\exists y\,(\text{Teacher}(y)\wedge\text{Likes}(x,y)))$. **Solution:** $\forall$ takes $\to$; the inner $\exists$ takes $\wedge$; the teacher $y$ may depend on $x$ (so $\exists$ is inside).

</details>

**Q7 (NAT).** Forward chaining with facts $A$, rules $A\to B$, $B\to C$, $B\wedge C\to D$, $E\to F$. How many distinct atoms are inferred (including the given fact)?

<details><summary>Answer</summary>

**Answer:** 4. **Solution:** $A,B,C,D$; $F$ needs $E$ which is never derived.

</details>

**Q8 (MSQ).** Which statements are true? (a) Resolution is refutation-complete for propositional logic. (b) Propositional entailment is decidable. (c) FOL entailment is decidable. (d) Forward chaining is complete for definite-clause KBs.

<details><summary>Answer</summary>

**Answer:** (a), (b), (d). **Solution:** FOL entailment is only semi-decidable.

</details>
