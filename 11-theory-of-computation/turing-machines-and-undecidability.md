# Turing Machines and Undecidability

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Turing machines; Undecidability
> **Prerequisites:** [Context-free languages and PDA](context-free-languages-and-pda.md) · [Pumping lemma](pumping-lemma.md) · [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) · **Leads to:** [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md)

## Quick glance

- **Turing machine (TM):** finite control + an infinite tape + a read/write head that moves left or right. It is the formal model of an algorithm (Church–Turing thesis). A TM can **accept, reject, or loop forever**.
- **Recursive (REC / decidable):** some TM halts on every input and accepts exactly L. **Recursively enumerable (RE / semi-decidable):** some TM accepts exactly L (it may loop on non-members).
- Hierarchy: **Regular ⊊ DCFL ⊊ CFL ⊊ CSL ⊊ REC ⊊ RE ⊊ all languages**. Automata: DFA/NFA, DPDA, NPDA, LBA, TM. Grammars (Chomsky types 3, 2, 1, 0).
- Multi-tape, multi-track, two-way-infinite and **non-deterministic** TMs all accept exactly the RE languages (and decide the REC languages). Nondeterminism adds no power for TMs.
- **Halting problem is undecidable** (diagonal argument), but RE. Its complement is not RE. **L and its complement both RE ⇒ L is REC.**
- **Rice's theorem:** every non-trivial property of the *language* L(M) of a TM M is undecidable (e.g. L(M) = ∅, L(M) finite, L(M) regular, L(M) = Σ*).
- Closure: REC is closed under complement, union, intersection, concatenation, star; RE is closed under union, intersection, concatenation, star but **not complement**.
- Undecidable for CFGs: equivalence, universality (L = Σ*), ambiguity, disjointness (via PCP). Decidable: membership, emptiness, finiteness.
- #1 trap: confusing "decidable" with "RE" and applying Rice's theorem to properties of the *machine* (number of states, steps taken) rather than of its *language*.

## 1. The Turing machine model

**Intuition.** A person with a pencil, an eraser and an endless strip of paper who may only look at one cell at a time and follows a fixed finite rulebook. Unlike a PDA, the tape can be read and rewritten anywhere (not just the top of a stack), which is exactly what lets it copy and compare many things.

**Definition.** M = (Q, Σ, Γ, δ, q0, B, F): states Q; input alphabet Σ; tape alphabet Γ ⊇ Σ ∪ {B} (B = blank, B ∉ Σ); transition δ: Q × Γ → Q × Γ × {L, R} (deterministic); start q0; accept states F. A *configuration* (instantaneous description) is written u q v: tape contents uv with the head on the first symbol of v and the control in state q. The input starts on the leftmost cells, all else blank. M **accepts** w if it reaches an accept state; **rejects** if it halts in a non-accepting state (or has no move); otherwise it **loops**.

L(M) = {w : M accepts w}.

| Term | Meaning |
| --- | --- |
| Recursively enumerable (RE, Turing-recognisable) | L = L(M) for some TM M (may loop on w ∉ L) |
| Recursive (REC, decidable) | L = L(M) for a TM that **halts on all inputs** (a decider) |
| Co-RE | complement is RE |

**Decidable ⊊ RE.** If L is decidable, run the decider. If L is only RE, run M; if w ∈ L it accepts, but if w ∉ L it may run forever, so we can never be sure.

### 1.1 Worked TM: L = {a^n b^n c^n : n ≥ 1}

Idea: repeatedly mark one a (X), one b (Y), one c (Z); when no a remains, verify that no b or c remains.

| State | Reads | Writes | Move | Next | Purpose |
| --- | --- | --- | --- | --- | --- |
| q0 | a | X | R | q1 | mark an a |
| q0 | Y | Y | R | q4 | no a's left, go verify |
| q1 | a, Y | same | R | q1 | skip a's and marked b's |
| q1 | b | Y | R | q2 | mark a b |
| q2 | b, Z | same | R | q2 | skip b's and marked c's |
| q2 | c | Z | L | q3 | mark a c |
| q3 | a, b, Y, Z | same | L | q3 | return left |
| q3 | X | X | R | q0 | next round |
| q4 | Y, Z | same | R | q4 | check only marks remain |
| q4 | B | B | R | q_accept | accept |

Any other (state, symbol) pair has no move: reject. Trace on aabbcc (shown at the end of each round):

```text
start:        a a b b c c
after round 1: X a Y b Z c      (head returns to the X, q0 moves right)
after round 2: X X Y Y Z Z
q0 sees Y -> q4 scans Y Y Z Z, then blank -> accept
```

Rejections: aabbc: after round 2 the tape is X X Y Y Z _ ; q2 skips Z, reaches the blank and has no move. aabc: after round 1 it is X a Y Z; round 2 marks the second a, then q1 reads Z (no b left) and is stuck. abcc: after round 1 it is X Y Z c; q0 reads Y, goes to q4, which reads c and is stuck. The machine decides L (halts on all inputs); it takes 8, 23, 46, 77 steps for n = 1, 2, 3, 4 (verified by a simulator), i.e. Θ(n²). The same machine accepts exactly a^n b^n c^n (n ≥ 1) on all strings over {a,b,c} up to length 9. This proves that {a^n b^n c^n}, not CFL, is decidable.

**Simpler TM for a^n b^n:** mark an a as X, move right to the first unmarked b and mark it Y, return to the X, repeat; accept when only X and Y remain.

**TMs as function computers.** A TM can compute functions: for unary addition 1^m 0 1^n → 1^(m+n), scan right to the 0, overwrite it with 1, scan to the blank, step left and erase the last 1. Result: 1^(m+n+1−1) = 1^(m+n).

### 1.2 Variants (all equivalent in power)

| Variant | Effect on power | Cost |
| --- | --- | --- |
| Multi-tape | none | simulation by a single tape costs a quadratic slowdown |
| Two-way infinite tape, multiple tracks, "stay" move | none | constant overhead |
| **Non-deterministic TM** | none (same RE; decider ⇒ decider) | deterministic simulation by BFS over the computation tree, exponential slowdown |
| Enumerator (prints the strings of L) | prints exactly the RE languages | |
| Universal TM | one fixed TM that takes ⟨M, w⟩ and simulates M on w | |
| Linear bounded automaton (LBA) | tape limited to the input length: exactly the CSLs | |
| PDA with two stacks | = TM | |

**Church–Turing thesis.** Every effectively computable function is computable by a TM. It is a thesis (not a theorem) because "effectively computable" is informal; every model proposed (λ-calculus, recursive functions, RAM machines, programs in any language) has turned out equivalent.

**Universal TM and encoding.** Every TM M is encoded as a finite string ⟨M⟩ over {0,1}. A universal TM U on input ⟨M, w⟩ simulates M on w; thus A_TM = {⟨M, w⟩ : M accepts w} is **RE**. The set of all TMs is countable, but the set of all languages is uncountable, so most languages are not RE.

## 2. The Chomsky hierarchy

| Type | Grammar rule form | Machine | Class | Example |
| --- | --- | --- | --- | --- |
| 3 | A → aB \| a (right-linear; or left-linear) | DFA / NFA | Regular | a*b* |
| 2 | A → γ (single variable on the left) | NPDA | Context-free | a^n b^n, ww^R |
| 1 | α → β with \|α\| ≤ \|β\| (S → ε allowed if S not on any right side) | LBA | Context-sensitive (CSL) | a^n b^n c^n, ww, a^(2^n), a^(n²) |
| 0 | α → β, α contains a variable (unrestricted) | TM | RE | A_TM, halting problem |

**Strict inclusions with witnesses.** Regular ⊊ DCFL (a^n b^n); DCFL ⊊ CFL (ww^R); CFL ⊊ CSL (a^n b^n c^n); CSL ⊊ REC (a decidable language that no LBA decides exists, by diagonalisation); REC ⊊ RE (A_TM or the halting problem); RE ⊊ all languages (complement of A_TM).

### 2.1 Closure under operations

| Operation | Regular | DCFL | CFL | CSL | REC | RE |
| --- | --- | --- | --- | --- | --- | --- |
| Union | Y | N | Y | Y | Y | Y |
| Intersection | Y | N | N | Y | Y | Y |
| Complement | Y | Y | N | **Y** | Y | **N** |
| Concatenation | Y | N | Y | Y | Y | Y |
| Kleene star | Y | N | Y | Y | Y | Y |
| Reversal | Y | N | Y | Y | Y | Y |
| Intersection with regular | Y | Y | Y | Y | Y | Y |
| Difference | Y | N | N | Y | Y | **N** |

(CSLs are closed under complement by the Immerman–Szelepcsényi theorem.) Notes: RE is closed under homomorphism, but REC is not (an erasing homomorphism can turn a decidable language into an undecidable one). RE is not closed under complement or difference L1 − L2. **Theorem: if L and its complement are both RE then L is REC** (run both machines in parallel, accept/reject according to whichever halts first). Consequences: the complement of a REC language is REC; for an RE-but-not-REC language the complement is not RE.

### 2.2 Decision problems across the hierarchy

| Problem | Regular | DCFL | CFL | CSL | REC | RE |
| --- | --- | --- | --- | --- | --- | --- |
| Membership | D | D | D | D | D | **U** |
| Emptiness | D | D | D | U | U | U |
| Finiteness | D | D | D | U | U | U |
| Equivalence L1 = L2 | D | D | U | U | U | U |
| Universality L = Σ* | D | D | U | U | U | U |
| Ambiguity (grammar) | | | U | | | |
| Disjointness | D | U | U | U | U | U |

D = decidable, U = undecidable. For CSL, membership is decidable (the LBA has finitely many configurations). **Memorise the CFL row: membership, emptiness, finiteness decidable; equivalence, universality, ambiguity, disjointness undecidable.**

## 3. The halting problem and diagonalisation

**Intuition.** If a program existed that could decide whether any program halts, we could use it on a program that does the opposite of its own prediction, producing a contradiction. This is Cantor's diagonal argument for programs.

**Theorem.** HALT = {⟨M, w⟩ : M halts on w} is undecidable.

**Proof.** Suppose a decider H exists: H(⟨M, w⟩) = accept if M halts on w, reject otherwise. Build D: on input ⟨M⟩, run H(⟨M, ⟨M⟩⟩); if H says "halts", **loop forever**; if H says "does not halt", **halt**. Run D on its own code ⟨D⟩: D halts on ⟨D⟩ ⇔ H says D does not halt on ⟨D⟩ ⇔ D does not halt on ⟨D⟩. Contradiction. ∎

**Same idea for A_TM.** A_TM = {⟨M, w⟩ : M accepts w} is undecidable and RE (the universal TM recognises it). The diagonal language L_d = {⟨M⟩ : M does not accept ⟨M⟩} is not RE. Its complement L_u = A_TM restricted to ⟨M, ⟨M⟩⟩ is RE but not REC.

| Language | REC | RE | Co-RE |
| --- | --- | --- | --- |
| A_TM, HALT | no | **yes** | no |
| Complement of A_TM, complement of HALT | no | **no** | yes |
| L_d (diagonal) | no | no | yes |
| E_TM = {⟨M⟩ : L(M) = ∅} | no | **no** | yes |
| Complement of E_TM = {⟨M⟩ : L(M) ≠ ∅} | no | **yes** | no |
| EQ_TM, ALL_TM (L = Σ*), FINITE_TM, REGULAR_TM | no | no | no |

Why L(M) ≠ ∅ is RE: dovetail M on all inputs in parallel; if some input is ever accepted, accept. It is not decidable because deciding it would decide A_TM (reduction below).

## 4. Reductions

**Mapping reduction.** A ≤_m B if a computable function f satisfies: w ∈ A ⇔ f(w) ∈ B.

| If A ≤_m B and … | Then |
| --- | --- |
| B is decidable | A is decidable |
| A is undecidable | **B is undecidable** (direction to remember: reduce a known-hard problem to the new one) |
| B is RE | A is RE |
| A is not RE | B is not RE |

### 4.1 Worked reduction: E_TM is undecidable

Reduce A_TM to the complement of E_TM. Given ⟨M, w⟩, build M' that ignores its own input x, runs M on w, and accepts if M accepts w. Then L(M') = Σ* if M accepts w, and ∅ otherwise. So ⟨M, w⟩ ∈ A_TM ⇔ L(M') ≠ ∅. A decider for emptiness would decide A_TM; none exists. Hence E_TM is undecidable. The same M' also shows ALL_TM (L = Σ*), "L(M) is regular", "L(M) is finite" are undecidable. Rice's theorem (§5) packages this gadget for every non-trivial property of L(M).

### 4.2 Other classic undecidable problems

- **Post Correspondence Problem (PCP):** given two lists of strings (u1…uk), (v1…vk), is there a non-empty sequence i1…im with u_{i1}…u_{im} = v_{i1}…v_{im}? Undecidable (RE). Example: dominoes (a, ab), (b, ca), (ca, a), (abc, c): the sequence 1, 2, 3, 1, 4 gives a·b·ca·a·abc = abcaaabc on top and ab·ca·a·ab·c = abcaaabc at the bottom (both read abcaaabc). PCP reduces to CFG ambiguity, CFG intersection emptiness, and CFG universality.
- **CFG problems:** ambiguity, L(G1) = L(G2), L(G) = Σ*, L(G1) ∩ L(G2) = ∅, "is L(G) regular", "is L(G) a DCFL" are all undecidable.
- **Program-level:** does a program halt on all inputs; do two programs compute the same function; is a given line of code dead (reachability); does the program output a given string. All undecidable (Rice-like).
- **Decidable by contrast:** halting for DFAs/PDAs/LBAs; emptiness and equivalence of DFAs; word problem for CSL grammars; Presburger arithmetic; tiling a finite region.

## 5. Rice's theorem

**Statement.** Let P be a property of Turing-recognisable languages (a set of RE languages) that is **non-trivial**: some TM's language has P and some TM's language does not. Then {⟨M⟩ : L(M) has P} is undecidable.

**How to apply (checklist).**
1. Is the property about the **language L(M)** (what M accepts), not about M's code, states, or running time? If it is about the machine, Rice does not apply.
2. Is the property **non-trivial**? Properties that hold for all (or no) RE languages are decidable (trivially).
3. If both yes, the problem is undecidable. (Whether it is also RE depends on the property: "L(M) ≠ ∅" is RE; "L(M) = ∅" is not.)

| Question | Rice applies? | Answer |
| --- | --- | --- |
| L(M) = ∅ | yes | undecidable |
| L(M) is finite / infinite | yes | undecidable |
| L(M) is regular / context-free | yes | undecidable |
| L(M) = Σ* | yes | undecidable |
| L(M) contains the string "abc" | yes | undecidable |
| \|L(M)\| ≥ 5 | yes | undecidable (RE) |
| L(M1) = L(M2) | yes (two machines) | undecidable |
| L(M) is a RE language | trivial (always true) | decidable |
| M has at least 5 states | no (machine property) | decidable |
| M halts within 100 steps on input w | no | decidable (simulate 100 steps) |
| M halts on the empty input | no (halting, a behaviour on one input) | undecidable by reduction from HALT |

The distinction between the last few rows: Rice's theorem is about *semantic* properties of the accepted language. A syntactic property (number of states, number of transitions) is decidable by inspecting the finite description; a bounded-step behaviour is decidable by simulation; **unbounded-step behaviour** (halting, reaching a state) is usually undecidable but needs its own reduction, not Rice.

## 6. Classifying languages: a decision procedure

For "which of these languages is regular / CFL / REC / RE / not RE" questions:

1. **Finite language** (or only bounded counts)? Regular.
2. **Only bounded memory** (mod counts, last k symbols, parity, starts/ends/contains)? Regular. Check with Myhill–Nerode or by building a small DFA.
3. **One unbounded comparison** done with a stack (a^n b^n, a^n b^m c^n, nested pairs a^n b^m c^m d^n, palindromes, #a = #b)? CFL. If a guess of the middle is needed (ww^R, "i = j or j = k") then CFL but not DCFL.
4. **Two or more simultaneous comparisons or copying** (a^n b^n c^n, ww, a^n b^m c^n d^m, a^(n²), a^(2^n), a^p prime)? Not CFL; but decidable, so **CSL ⊆ REC**.
5. **Questions about TMs/programs** (halting, acceptance, language of a TM)? Look at the quantifier: "∃ input / ∃ step where something happens" (halts, L ≠ ∅, accepts some string) is **RE but not REC**; the complement ("never halts", L = ∅) is **co-RE, not RE**; "for all inputs" statements (L = Σ*, equivalence) are typically neither RE nor co-RE.
6. **A property with a bounded search** (halts in ≤ k steps; some string of length ≤ k accepted) is decidable.

**Worked classification.** (a) {⟨M⟩ : M halts on ε}: RE, not REC. (b) its complement: not RE. (c) {a^n b^n c^n d^n}: CSL, decidable, not CFL. (d) {a^n b^m : n ≠ m}: DCFL. (e) {⟨G⟩ : G is a CFG with L(G) = ∅}: decidable (generating-symbol algorithm). (f) {⟨G⟩ : G ambiguous}: undecidable, RE (guess a string and two trees). (g) {⟨G⟩ : L(G) = Σ*}: undecidable, not RE (its complement is RE: search for a string not derivable). (h) {⟨M⟩ : L(M) has exactly one string}: undecidable by Rice; it is neither RE nor co-RE (confirming one string is accepted is RE-like, confirming no second string is accepted is co-RE-like, and the property needs both).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Hierarchy | Regular ⊊ DCFL ⊊ CFL ⊊ CSL ⊊ REC ⊊ RE | containment questions |
| TM variants | multitape ≡ NTM ≡ TM | equivalence questions |
| Co-RE theorem | L and L^c RE ⇒ L REC | closure/decidability |
| RE closure | ∪ ∩ · * reverse; not complement | closure table |
| REC closure | all of the above and complement | closure table |
| Halting | undecidable, RE; complement not RE | classification |
| Rice | non-trivial semantic properties of L(M) undecidable | property questions |
| Reduction direction | A ≤ B and A undecidable ⇒ B undecidable | reduction questions |
| CFG decidable | membership, emptiness, finiteness | decision-property table |
| CFG undecidable | equivalence, universality, ambiguity, disjointness | decision-property table |
| LBA | membership decidable; emptiness undecidable | CSL questions |

## GATE traps

- **Decidable vs RE:** halting is RE (accept when it halts) but not decidable. Its complement is not RE. A language is REC iff both it and its complement are RE.
- **Reduction direction:** reducing a problem *to* a known-undecidable problem proves nothing. To show X undecidable, reduce a *known undecidable problem to X* (A_TM ≤ X).
- **Rice's theorem** covers properties of L(M), not of M. "M has 10 states" and "M halts within 100 steps" are decidable. "L(M) = ∅" is undecidable but its complement is RE: do not confuse "undecidable" with "not RE".
- **Nondeterministic TM** ≡ deterministic TM in power; contrast with NFA (same) and NPDA (more powerful than DPDA).
- **Membership for CSL is decidable; for type 0 it is not.** Emptiness for CSL is undecidable, unlike for CFLs.
- **Context-free grammar problems:** equivalence and ambiguity are undecidable even though membership and emptiness are decidable. Equivalence of DPDAs is decidable, regular languages all decidable.
- **A language not CFL is not automatically undecidable:** a^n b^n c^n and ww are decidable (CSL).
- **A TM can loop.** "TM accepts L" says nothing about halting on non-members; only a **decider** halts always.
- A single tape TM cannot do things impossible for multi-tape ones; only the running time changes (quadratic).

## Connections

- [Context-free languages and PDA](context-free-languages-and-pda.md) — PDAs are TMs restricted to a stack; CFG ambiguity/equivalence undecidability comes from PCP.
- [Regular languages and finite automata](regular-languages-and-finite-automata.md) — all decision problems decidable at the bottom of the hierarchy; undecidability starts at CFL (equivalence) and CSL (emptiness).
- [Pumping lemma](pumping-lemma.md) — separates the lower levels of the hierarchy.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — TM step counts define time complexity; polynomial-time classes (P, NP) sit inside REC.
- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — satisfiability is decidable (truth tables) while validity of first-order logic is only RE (see [First-order logic](../01-discrete-mathematics/first-order-logic.md)).
- [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) — countable vs uncountable sets underlie the existence of non-RE languages.
- [Parsing](../12-compiler-design/parsing.md) — programmers want unambiguous grammars, but ambiguity is undecidable in general; parser generators detect conflicts conservatively.
- [Optimization and data-flow analysis](../12-compiler-design/optimization-and-dataflow.md) — exact program analyses (dead code, constant propagation) are undecidable by Rice-style arguments, so compilers use conservative approximations.

## Practice

**Q1 (MCQ).** Which of the following machines accepts exactly the recursively enumerable languages?
(A) DFA (B) Deterministic PDA (C) Linear bounded automaton (D) Non-deterministic Turing machine

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:** DFA ↔ regular, DPDA ↔ DCFL, LBA ↔ CSL. TMs of all variants, including non-deterministic ones, accept exactly the RE languages.

</details>

**Q2 (MCQ).** Which statement is true?
(A) Every RE language is decidable. (B) The complement of every RE language is RE. (C) If L and its complement are RE then L is decidable. (D) The halting problem is decidable.

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Run both semi-deciders in parallel; exactly one will halt, so we can decide L. (A), (B) fail for the halting problem; (D) false.

</details>

**Q3 (MSQ).** Which of the following are decidable?
(A) Given a CFG G, is L(G) empty? (B) Given CFGs G1, G2, is L(G1) = L(G2)? (C) Given a DFA M, is L(M) infinite? (D) Given a TM M, is L(M) empty?

<details><summary>Answer</summary>

**Answer:** A, C  
**Solution:** (A) test whether S is a generating symbol. (C) look for a reachable, co-reachable cycle (or a string of length in [n, 2n−1]). (B) CFG equivalence is undecidable. (D) E_TM is undecidable (Rice).

</details>

**Q4 (MCQ).** Let L be a language such that both L and its complement are recursively enumerable. Which is certainly true?
(A) L is regular (B) L is context-free (C) L is recursive (D) L is context-sensitive

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Both RE ⇒ REC. It need not be CSL or context-free (any decidable language has this property).

</details>

**Q5 (MCQ).** Which of the following is undecidable by Rice's theorem?
(A) Does the TM M have more than 10 states? (B) Does M halt within 50 steps on input w? (C) Is L(M) a regular language? (D) Is the encoding ⟨M⟩ longer than 100 symbols?

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** "L(M) is regular" is a non-trivial property of the language. (A) and (D) are syntactic, (B) is bounded simulation: all decidable.

</details>

**Q6 (MSQ).** Which of the following languages are not recursively enumerable?
(A) {⟨M, w⟩ : M halts on w} (B) {⟨M⟩ : L(M) = ∅} (C) {⟨M⟩ : L(M) ≠ ∅} (D) the complement of A_TM

<details><summary>Answer</summary>

**Answer:** B, D  
**Solution:** (A) HALT is RE (simulate and accept if M halts). (C) nonempty: dovetailing finds an accepted string, RE. (B) is the complement of (C), and (C) is not decidable, so (B) is not RE (otherwise (C) would be decidable). (D) complement of A_TM: not RE, because A_TM is RE but not decidable.

</details>

**Q7 (MCQ).** To prove that problem X is undecidable by reduction, which direction is correct?
(A) reduce X to the halting problem (B) reduce the halting problem to X (C) reduce X to a decidable problem (D) reduce a decidable problem to X

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** If the halting problem reduces to X and X were decidable, the halting problem would be decidable. (A) would prove nothing about X's hardness.

</details>

**Q8 (NAT).** How many of the following languages are not context-free? {a^n b^n c^n}, {ww}, {ww^R}, {a^n b^m c^n}, {a^(2^n)}, {a^n b^n c^m}, {a^n b^m c^m d^n}

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** Not CFL: a^n b^n c^n, ww, a^(2^n) (non-regular unary). The others are CFL: ww^R (guess the middle), a^n b^m c^n, a^n b^n c^m, a^n b^m c^m d^n (nested). All seven are decidable (CSL or lower).

</details>
