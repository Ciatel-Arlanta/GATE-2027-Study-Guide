# Pumping Lemma

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Pumping lemma
> **Prerequisites:** [Regular languages and finite automata](regular-languages-and-finite-automata.md) · [Context-free languages and PDA](context-free-languages-and-pda.md) · **Leads to:** [Turing machines and undecidability](turing-machines-and-undecidability.md)

## Quick glance

- **Regular pumping lemma.** If L is regular there is a p ≥ 1 such that every w ∈ L with |w| ≥ p splits as w = xyz with **|xy| ≤ p, |y| ≥ 1**, and **xy^i z ∈ L for all i ≥ 0**. (One pumpable chunk; p = number of DFA states works.)
- **CFL pumping lemma.** If L is context-free there is a p ≥ 1 such that every z ∈ L with |z| ≥ p splits as z = uvwxy with **|vwx| ≤ p, |vx| ≥ 1**, and **uv^i w x^i y ∈ L for all i ≥ 0**. (Two chunks pumped together.)
- Both lemmas are **necessary conditions only**. They can **prove a language is NOT regular/CFL, never that it is**. A language that satisfies the lemma may still be non-regular.
- Proof template: it is a game. The adversary picks p; **you** pick a string w ∈ L with |w| ≥ p (as a function of p); the adversary picks the split; **you** pick an i that leaves L. You win if you have an i for every legal split.
- Good strings: a^p b^p (regular proofs), a^p b^p c^p (CFL), a^p b^p a^p b^p (ww), a^(p²), a^q with q prime ≥ p.
- Useful i values: **i = 0** (pump down) for "greater than" languages like {a^i b^j : i > j}; **i = 2** (pump up) for equality languages; i = p + 1 or q + 1 for primes.
- Fast alternatives: closure properties (intersect with a regular language, then use complement/homomorphism) and Myhill–Nerode.
- #1 trap: **choosing the split yourself**. The split is chosen by the adversary, so your argument must cover every split allowed by |xy| ≤ p (regular) or |vwx| ≤ p (CFL).

## 1. Why pumping works (intuition)

A DFA with p states reading a string of length ≥ p must repeat a state within the first p symbols: it went around a loop. The loop's label is y. Going around the loop again (or skipping it) leads to the same state, so xy^i z is accepted for every i. That is the whole proof: **pigeonhole on states**.

```text
        y (loop, |y| >= 1)
       ┌──────┐
  x    ▼      │        z
 q0 ──▶ q ────┘ ──▶ ... ──▶ accept        |xy| <= p because the first repeat happens within p steps
```

For CFLs the analogue is a parse tree so deep that along some root-to-leaf path a variable A repeats: A ⇒* vAx and A ⇒* w. Repeating the A ⇒* vAx part pumps v and x together. Because the pumped parts v and x are on the two sides of w, **a CFL can pump two places at once but they stay "in sync"**, which is why a single stack comparison (a^n b^n) works but three-way equalities do not.

## 2. Statements

**Pumping lemma for regular languages.** If L is regular, ∃ p ≥ 1 such that ∀ w ∈ L with |w| ≥ p, ∃ x, y, z with w = xyz, (1) |y| ≥ 1, (2) |xy| ≤ p, (3) ∀ i ≥ 0: xy^i z ∈ L.

**Pumping lemma for CFLs.** If L is a CFL, ∃ p ≥ 1 such that ∀ s ∈ L with |s| ≥ p, ∃ u, v, w, x, y with s = uvwxy, (1) |vx| ≥ 1, (2) |vwx| ≤ p, (3) ∀ i ≥ 0: u v^i w x^i y ∈ L.

**Contrapositive (what you actually prove).** L is not regular if: for every p there is w ∈ L, |w| ≥ p, such that for every split satisfying (1) and (2) there is an i with xy^i z ∉ L.

| | Regular | CFL |
| --- | --- | --- |
| Chunks pumped | 1 (y) | 2 (v and x together) |
| Length bound | \|xy\| ≤ p (the pumpable part is near the start) | \|vwx\| ≤ p (the pumpable window is any window of length ≤ p) |
| Non-empty | \|y\| ≥ 1 | \|vx\| ≥ 1 (v or x may be empty, not both) |
| Quantifier shape | ∃p ∀w ∃split ∀i | ∃p ∀s ∃split ∀i |

## 3. The adversarial-game proof template

1. Assume L is regular (CFL). Let p be the pumping length given by the lemma.
2. **Choose** a specific string w ∈ L that **depends on p** and has |w| ≥ p.
3. Let the lemma split w in **any** way allowed. List the cases the constraints allow (the |xy| ≤ p or |vwx| ≤ p condition usually pins y or vwx inside a single block).
4. In each case **choose i** so that the pumped string is not in L. Usually the count of one symbol changes while another stays fixed.
5. Contradiction, so L is not regular (CFL).

**Choosing w well is the whole skill.** The string must force the constraint |xy| ≤ p (or |vwx| ≤ p) to trap the pumped part inside a region where pumping hurts. A bad w lets the adversary pump somewhere harmless.

**Bad choice example.** For L = {a^i b^j : i > j}, the choice w = a^{2p} b^p ∈ L fails: the adversary picks y = a^k and pumping down gives a^{2p−k} b^p, which still has 2p − k ≥ p a's and may still be in L for small k, so no contradiction is reached. The choice w = a^{p+1} b^p works (pumping down leaves at most p a's). Always check that w ∈ L and that |xy| ≤ p confines y to a region where pumping must hurt.

## 4. Worked proofs: non-regular

### 4.1 L = {a^n b^n : n ≥ 0}

Let p be the pumping length. Take w = a^p b^p ∈ L, |w| = 2p ≥ p. Any split w = xyz with |xy| ≤ p lies entirely inside the a's, so y = a^k with 1 ≤ k ≤ p. Pump up (i = 2): xy²z = a^{p+k} b^p has more a's than b's, so ∉ L. Contradiction. **Not regular.** (Same conclusion by Myhill–Nerode: infinitely many classes.)

### 4.2 L = {w ∈ {a,b}* : #a(w) = #b(w)}

Take w = a^p b^p ∈ L. y = a^k (k ≥ 1) as above; i = 2 gives p + k a's against p b's. ∉ L. Not regular. (Quicker: L ∩ a*b* = {a^n b^n}; if L were regular this intersection would be regular.)

### 4.3 L = {a^i b^j : i > j}

Take w = a^{p+1} b^p ∈ L. y = a^k with 1 ≤ k ≤ p. **Pump down**, i = 0: xz = a^{p+1−k} b^p. Since k ≥ 1, p + 1 − k ≤ p, so a's ≤ b's, so xz ∉ L. Not regular. (Pumping up would not work: more a's keeps i > j.)

### 4.4 L = {a^(n²) : n ≥ 0} (perfect-square lengths)

Take w = a^{p²}. y = a^k with 1 ≤ k ≤ p. Pump i = 2: length p² + k, and
p² < p² + k ≤ p² + p < p² + 2p + 1 = (p+1)².
A length strictly between consecutive squares is not a square. ∉ L. Not regular. Consecutive perfect squares have gaps 2n + 1 that grow without bound, while the pumping lemma forces a bounded step k ≤ p.

### 4.5 L = {a^q : q prime}

Take a prime q ≥ p + 2 (primes are unbounded), w = a^q. Then y = a^k with 1 ≤ k ≤ p < q. Pump with i = q + 1: the length is q + q·k = **q(k + 1)**, a product of two integers each ≥ 2, so composite, hence ∉ L. Not regular.

### 4.6 L = {ww : w ∈ {a,b}*}

Take w₀ = a^p b a^p b ∈ L (it is ww with w = a^p b). A split with |xy| ≤ p puts y = a^k, 1 ≤ k ≤ p, in the first a-block. Pump up (i = 2): a^{p+k} b a^p b has exactly two b's. If it were uu, each half u would contain one b, so u = a^m b a^r and uu = a^m b a^{r+m} b a^r. Matching a^{p+k} b a^p b gives m = p + k, r = 0 (last block empty) and r + m = p, so m = p, contradicting m = p + k. ∉ L. Not regular.

### 4.7 L = {a^(2^n)}

w = a^{2^p}, y = a^k with 1 ≤ k ≤ p < 2^p. xy²z has length 2^p + k with 2^p < 2^p + k < 2^{p+1}. Between consecutive powers of 2, so not in L. Not regular.

### 4.8 The language {a^i b^j c^k : i = 1 ⇒ j = k} satisfies the lemma but is not regular

(For the "pumping lemma cannot prove regularity" point, §6.) Strings in L: every string a^i b^j c^k with i ≠ 1, plus those with i = 1 and j = k. Take p = 3 and any w ∈ L with |w| ≥ 3.

- i = 0: y = first symbol (x = ε). Pumping never creates an a, so the string stays in L.
- i = 1 (so w = a b^j c^j): y = the leading a. Pumped strings have i = 0 or i ≥ 2 (all in L) or i = 1 (the original).
- i = 2: y = aa (|xy| = 2 ≤ 3). Pumping changes i by even amounts: 0, 2, 4, …, never 1.
- i ≥ 3: y = the first a. Pumping gives i − 1 ≥ 2, or larger, never 1.

So the lemma holds, yet L is **not regular**: L ∩ ab*c* = {a b^j c^j}, which is not regular.

## 5. Worked proofs: non-context-free

### 5.1 L = {a^n b^n c^n : n ≥ 0}

Take s = a^p b^p c^p, |s| = 3p. Any window vwx of length ≤ p cannot contain both an a and a c (a b-block of length p separates them), so vx misses at least one of the three letters. Pump up (i = 2): the letter(s) in vx increase while the missing letter keeps count p. Not all three counts are equal any more. ∉ L. **Not context-free.**

### 5.2 L = {ww}

Because CFLs are closed under intersection with regular languages, consider L' = L ∩ a*b*a*b* = {a^i b^j a^i b^j : i, j ≥ 0}. If L were CFL, so would be L'. Take s = a^p b^p a^p b^p. The four blocks each have length p, so a window of length ≤ p touches **at most two adjacent blocks** and never contains a whole inner block. Pump i = 2 (or 0).

- If the pumped string has left the form a*b*a*b* (v or x straddles a block boundary), it is not in L'.
- Otherwise exactly one or two **adjacent** blocks change length, and at least one does. The partner of block 1 is block 3 and the partner of block 2 is block 4, which are never adjacent, so a changed block always has an unchanged partner. The equalities i = i and j = j break. ∉ L'.

Contradiction. {ww} is **not context-free**. (Contrast: {ww^R} is context-free, and the complement of {ww} is context-free.)

### 5.3 L = {a^i b^j c^k : i < j < k}

Take s = a^p b^{p+1} c^{p+2}. The window cannot contain both a and c.
- vx contains no c and contains a b: pump up, b-count ≥ p + 2 = c-count, so j < k fails.
- vx contains no c and no b (so only a's): pump up, a-count ≥ p + 1 = b-count, i < j fails.
- vx contains a c and (no a): if it also contains a b, pump down: b-count ≤ p = a-count, i < j fails; if only c's, pump down: c-count ≤ p + 1 = b-count, j < k fails.

All cases fail, so L is **not context-free**.

### 5.4 L = {a^(n²)}, {a^p : p prime}, {a^(2^n)}

Unary alphabet: vx = a^k with 1 ≤ k ≤ p, and the pumped length is |s| + (i − 1)k, exactly the arithmetic of §4.4, 4.5, 4.7 with k ≤ p. So these are not CFL. Faster: **a CFL over a one-letter alphabet is regular**, and these are not regular.

### 5.5 L = {a^n b^m c^n d^m}

s = a^p b^p c^p d^p. A window of length ≤ p touches at most two adjacent blocks; pumping changes one or two adjacent blocks. The crossing pairs are (a, c) and (b, d), neither adjacent, so a changed block always keeps its unchanged partner and equality breaks. Not CFL. (Nested pairs a^n b^m c^m d^n **are** CFL; crossing pairs are not.)

## 6. What the lemma cannot do

- **It cannot prove a language is regular (or CFL).** The conditions are necessary, not sufficient. §4.8 gives a non-regular language that obeys the lemma. Similar examples exist for the CFL lemma.
- It cannot be applied by choosing p or the decomposition yourself. If you "find a good split", that proves nothing.
- A finite language always satisfies the lemma (take p larger than the longest word; no string of length ≥ p exists). Hence finite ⇒ regular (and CFL).
- **To show regular, build an automaton/regex or use closure properties.** To show non-regular, use the lemma, Myhill–Nerode, or closure ("if L were regular then L ∩ a*b* would be").

## 7. Closure as a shortcut

| Claim | Argument |
| --- | --- |
| {w : #a = #b} not regular | L ∩ a*b* = {a^n b^n}, not regular |
| {a^n b^n : n ≥ 0}^c not regular | complement closure: if its complement were regular, so is a^n b^n |
| {w : #a = #b = #c} not CFL | ∩ with a*b*c* gives a^n b^n c^n |
| Complements of {ww} and of {a^n b^n c^n} are CFL | the complement of a non-CFL can be CFL, so "complement is CFL" says nothing |
| L ⊆ a* CFL ⇒ regular | unary CFLs are regular |

**Facts about pumping length and DFAs.**
- A DFA with p states accepts an infinite language iff it accepts some string of length in [p, 2p − 1].
- If a DFA with p states accepts any string at all, it accepts one of length < p (shortest accepted string); so emptiness can be checked on strings shorter than p.
- A language is infinite iff it has a string of length ≥ p that can be pumped (for regular and CFL alike).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Regular lemma | w = xyz, \|xy\| ≤ p, \|y\| ≥ 1, xy^i z ∈ L ∀ i ≥ 0 | non-regularity proofs |
| CFL lemma | s = uvwxy, \|vwx\| ≤ p, \|vx\| ≥ 1, uv^i wx^i y ∈ L | non-CFL proofs |
| Pumping length for DFA | p = number of states | length-bound questions |
| Infinite language test | accepts a string with p ≤ length ≤ 2p − 1 | finiteness of DFA language |
| Gap argument | squares: p² < p²+k < (p+1)² ; powers of 2 | unary non-regular proofs |
| Prime argument | pump q+1 times: length q(k+1) | prime-length languages |
| Unary CFL | regular | quick CFL eliminations |

## GATE traps

- **The lemma proves non-regularity only.** "L satisfies the pumping lemma, so L is regular" is always invalid.
- The adversary chooses the split. "Choose y = the first a" is not a proof; you must handle all splits permitted by |xy| ≤ p.
- Mixing up the two lemmas: |xy| ≤ p belongs to the regular lemma, |vwx| ≤ p to the CFL one. The CFL window can be anywhere.
- For regular proofs the pumped part lies in the first p symbols, so put the "fragile" block first: a^p b^p traps y among the a's. Always check your chosen string actually belongs to L.
- Pumping down (i = 0) is needed for ">" and "≥" languages; pumping up fails there.
- Languages like {a^n b^m : n ≠ m} look hard but are CFL (and not regular); a^n b^n with n bounded is regular. Do not apply the lemma to a finite language.
- If the question gives a language and asks "which of the following is NOT context-free", check the standard ones first: three-way equality, copy (ww), crossing dependencies, prime/square/power lengths.
- A language that is not regular is not automatically not-CFL; and a CFL with a non-CFL complement exists.

## Connections

- [Regular languages and finite automata](regular-languages-and-finite-automata.md) — the lemma is the pigeonhole principle on DFA states; Myhill–Nerode is the exact characterisation.
- [Context-free languages and PDA](context-free-languages-and-pda.md) — the CFL version is the pigeonhole principle on variables along a parse-tree path; closure under regular intersection is used constantly.
- [Turing machines and undecidability](turing-machines-and-undecidability.md) — the languages here, e.g. {a^n b^n c^n} and {ww}, are decidable by TMs (CSLs), and they illustrate the strictness of the Chomsky hierarchy.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — pigeonhole principle.
- [Lexical analysis](../12-compiler-design/lexical-analysis.md) — tokens must be regular; balanced constructs are beyond lexers.
- [Parsing](../12-compiler-design/parsing.md) — nested structure needs a stack, so grammars (not regexes) are used for syntax.

## Practice

**Q1 (MCQ).** In the pumping lemma for regular languages, which condition is always required of the split w = xyz?
(A) |x| ≥ 1 (B) |y| ≥ 1 and |xy| ≤ p (C) |yz| ≤ p (D) |xyz| ≤ p

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** The lemma requires a non-empty pumpable y that appears within the first p symbols.

</details>

**Q2 (MSQ).** Which of the following languages can be shown non-regular using the pumping lemma? (A) {a^n b^n} (B) {a^(n²)} (C) {a^n b^m} (D) {ww^R}

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:** (A) pump a's up; (B) the gap argument; (D) take s = a^p b b a^p, y within the a's, pump up. (C) = a*b* is regular.

</details>

**Q3 (MCQ).** To show {a^i b^j : i > j} is not regular with w = a^{p+1} b^p, the pumped string used is
(A) xy²z (B) xz (C) xy³z (D) none, the language is regular

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Since y = a^k, k ≥ 1, removing it leaves a^{p+1−k} b^p with at most p a's, so i ≤ j. Pumping up keeps i > j.

</details>

**Q4 (NAT).** A DFA has 7 states. Its language is infinite iff it accepts a string of length ℓ with 7 ≤ ℓ ≤ N. What is the upper limit N given by the standard result (N = 2p − 1)?

<details><summary>Answer</summary>

**Answer:** 13  
**Solution:** The standard result: L(M) infinite ⇔ M accepts a string with p ≤ |w| ≤ 2p − 1. With p = 7, N = 13. (If a string of length ≥ 2p is accepted, pumping y down repeatedly, removing |y| ≤ p symbols at a time, reaches a length in [p, 2p − 1].)

</details>

**Q5 (MCQ).** Which language is not context-free?
(A) {a^n b^n c^m} (B) {a^n b^m c^m d^n} (C) {a^n b^n c^n} (D) {a^n b^n} ∪ {a^n b^2n}

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** (A) one comparison; (B) nested; (D) union of two CFLs; (C) needs two simultaneous equalities, not CFL by the pumping lemma on a^p b^p c^p.

</details>

**Q6 (MSQ).** Which statements are true?
(A) If L satisfies the regular pumping lemma then L is regular. (B) Every finite language satisfies the pumping lemma. (C) Every CFL over a unary alphabet is regular. (D) {a^p : p prime} is not context-free.

<details><summary>Answer</summary>

**Answer:** B, C, D  
**Solution:** (A) false: the lemma is only necessary (§4.8 counterexample). (B) true: vacuous for p larger than every word. (C) true (Parikh-type result). (D) true, since it is a non-regular unary language.

</details>

**Q7 (NAT).** L = {a^(n²) : n ≥ 0}. Take p = 5, w = a^25, y = a^k with 1 ≤ k ≤ 5, and pump once up (i = 2). For how many values of k is the pumped string still in L?

<details><summary>Answer</summary>

**Answer:** 0  
**Solution:** xy²z has length 25 + k, i.e. 26, 27, 28, 29 or 30. The nearest squares are 25 and 36, so none of these lengths is a square. For every k the pumped string leaves L, which is the contradiction that proves L non-regular.

</details>

**Q8 (MCQ).** Using the pumping lemma for CFLs on s = a^p b^p c^p, why does {a^n b^n c^n} fail to be context-free?
(A) vwx may contain only a's, so pumping adds only a's (B) |vwx| ≤ p prevents vx from touching all three letters (C) uvwxy has too few symbols (D) Both (A) and (B) combine to show some count is unchanged

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:** Because |vwx| ≤ p, vx cannot contain all three letters; whichever letter is absent keeps its count p while at least one other letter changes. The counts are no longer equal, so the pumped string leaves L.

</details>
