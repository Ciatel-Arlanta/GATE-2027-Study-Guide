# Theory of Computation: Cheat sheet

Chapters: [Regular languages](regular-languages-and-finite-automata.md) · [CFL and PDA](context-free-languages-and-pda.md) · [Pumping lemma](pumping-lemma.md) · [TM and undecidability](turing-machines-and-undecidability.md)

## Hierarchy

| Class | Grammar | Machine | Example in class, not below |
| --- | --- | --- | --- |
| Regular | Type 3 (A → aB \| a) | DFA = NFA = ε-NFA | a*b* |
| DCFL | LR(k) grammars | DPDA (final state) | a^n b^n, w c w^R |
| CFL | Type 2 (A → γ) | NPDA | ww^R, {a^i b^j c^k : i = j or j = k} |
| CSL | Type 1 (\|α\| ≤ \|β\|) | LBA | a^n b^n c^n, ww, a^(n²), a^(2^n) |
| REC | – | TM that always halts | decidable but not CSL |
| RE | Type 0 | TM (any variant) | halting problem, A_TM |

**Regular ⊊ DCFL ⊊ CFL ⊊ CSL ⊊ REC ⊊ RE.**

## Regular expressions

- Precedence: star > concatenation > union. ∅* = ε, ∅·L = ∅, ε·L = L.
- (a+b)* = (a*b*)* = (a*+b*)* = a*(ba*)* = (a*b)*a*. (r+ε)* = r*. r*r* = r*. (ab)*a = a(ba)*.
- Not equal: (a+b)* ≠ a*+b*; (ab)* ≠ a*b*; (a+ab)* ≠ (a+b)*.
- No two consecutive a's: (b+ab)*(a+ε). Even a's: b*(ab*ab*)*. Ends abb: (a+b)*abb.

## Finite automata

| Fact | Value |
| --- | --- |
| DFA | (Q, Σ, δ: Q×Σ→Q, q0, F), δ total |
| NFA → DFA | ≤ 2^n states (construct reachable subsets only) |
| ε-NFA → NFA | same states; use ε-closure |
| Complement | complete DFA, swap F (NFA: determinise first) |
| Product (∩, ∪) | m·n states |
| Reversal | reverse edges, swap start/final |
| RE of length n → ε-NFA | O(n) states (Thompson, ≤ 2n) |
| Minimisation | remove unreachable states, partition refinement; minimal DFA unique |
| Myhill–Nerode | #classes = #states of minimal DFA; infinite classes ⇒ not regular |
| Moore vs Mealy | outputs n+1 vs n for n inputs |
| Arden | R = Q + RP, ε ∉ P ⇒ R = QP* |

**Minimal complete DFA sizes**

| Language | States |
| --- | --- |
| ends with fixed string, length k | k+1 |
| contains fixed substring, length k | k+1 |
| starts with fixed string, length k | k+2 |
| \|w\| ≥ k / at least k a's | k+1 |
| \|w\| = k / at most k a's | k+2 |
| \|w\| mod k = r | k |
| #a mod m and #b mod n | mn |
| k-th symbol from end is 1 | 2^k (NFA: k+1) |
| binary divisible by n (n odd) | n |
| binary divisible by n = 2^k·m (m odd) | m + k (4 → 3, 6 → 4, 8 → 4, 12 → 5) |
| a*b*c* | 4 |

## Closure and decision properties

| | Regular | DCFL | CFL | CSL | REC | RE |
| --- | --- | --- | --- | --- | --- | --- |
| Union | Y | N | Y | Y | Y | Y |
| Intersection | Y | N | N | Y | Y | Y |
| Complement | Y | Y | N | Y | Y | N |
| Concatenation / star | Y | N | Y | Y | Y | Y |
| Reversal | Y | N | Y | Y | Y | Y |
| ∩ with regular | Y | Y | Y | Y | Y | Y |
| Homomorphism | Y | N | Y | N | N | Y |
| Inverse homomorphism | Y | Y | Y | Y | Y | Y |

| Problem | Regular | DCFL | CFL | CSL | REC/RE |
| --- | --- | --- | --- | --- | --- |
| Membership | D | D | D (CYK O(n³)) | D | REC: D; RE: U |
| Emptiness | D | D | D | U | U |
| Finiteness | D | D | D | U | U |
| Equivalence | D | D | U | U | U |
| Universality (L = Σ*) | D | D | U | U | U |
| Ambiguity | – | – | U | – | – |
| Disjointness | D | U | U | U | U |

## Grammars and PDAs

- Ambiguous: one string, two parse trees (two leftmost derivations). Inherently ambiguous: {a^i b^j c^k : i = j or j = k}. Ambiguity undecidable.
- Simplification order: ε-productions, unit productions, useless symbols (generating first, then reachable).
- CNF (A → BC | a): derivation of length-n string takes **2n − 1** steps. GNF (A → aα): **n** steps.
- Number of parse trees of S → SS | a for a^n: Catalan C(n−1) (a^4: 5).
- PDA: final-state ≡ empty-stack for NPDA; DCFL ⊊ CFL; guess-the-middle ⇒ not DCFL.
- Unary CFL ⇒ regular. CFG ⇄ PDA by leftmost-derivation simulation.

| Language | Class |
| --- | --- |
| a^n b^n, #a = #b, a^n b^m c^n | DCFL |
| ww^R, palindromes, i=j or j=k | CFL, not DCFL |
| a^n b^m c^m d^n (nested) | CFL |
| a^n b^n c^n, ww, a^n b^m c^n d^m (crossing), i<j<k | not CFL |
| complement of ww, of a^n b^n c^n | CFL |
| a^(n²), a^(2^n), a^p (p prime) | not CFL (not regular, unary) |

## Pumping lemmas

| | Regular | CFL |
| --- | --- | --- |
| Split | w = xyz | s = uvwxy |
| Conditions | \|y\| ≥ 1, \|xy\| ≤ p | \|vx\| ≥ 1, \|vwx\| ≤ p |
| Pump | xy^i z ∈ L ∀ i ≥ 0 | uv^i wx^i y ∈ L ∀ i ≥ 0 |
| Standard w | a^p b^p | a^p b^p c^p |

- Only proves non-regularity/non-CFL, never membership. Adversary picks the split.
- Pump down (i = 0) for ">" languages: w = a^{p+1} b^p.
- Squares: p² < p²+k < (p+1)². Primes: pump q+1 times: length q(k+1). Powers of 2: between 2^p and 2^{p+1}.
- DFA with p states: infinite language ⇔ accepts a string with length in [p, 2p−1].

## Turing machines and undecidability

- TM: (Q, Σ, Γ, δ: Q×Γ→Q×Γ×{L,R}, q0, B, F). Multitape ≡ NTM ≡ TM (same languages). Church–Turing thesis. Universal TM simulates ⟨M, w⟩.
- **L and L^c both RE ⇒ L REC.** REC closed under complement; RE not.
- HALT, A_TM: RE, not REC; complements not RE.
- E_TM (L(M) = ∅): not RE; L(M) ≠ ∅: RE, not REC. EQ_TM, ALL_TM, FINITE_TM, REGULAR_TM: neither RE nor co-RE.
- Reduction: A ≤ B, A undecidable ⇒ B undecidable (reduce the known-hard problem to the new one).
- **Rice:** any non-trivial property of L(M) is undecidable. Not applicable to properties of M itself (states, halts within k steps = decidable).
- PCP undecidable; reduces to CFG ambiguity, intersection-emptiness, universality.
- Decidable: DFA/CFG emptiness, CFG membership, CSL membership, DFA equivalence, Presburger arithmetic.

## Classifying a language (procedure)

1. Finite or bounded memory (mod, last k, parity) → regular.
2. One stack comparison → CFL (DCFL if no guess needed).
3. Two comparisons, copy, crossing, n², 2^n, primes → not CFL, but CSL (decidable).
4. About TMs: "exists input/step" → RE, not REC; complement → not RE; "for all inputs" → typically neither.
