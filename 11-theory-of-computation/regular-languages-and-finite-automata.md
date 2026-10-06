# Regular Languages and Finite Automata

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Regular expressions; Finite automata; Regular languages
> **Prerequisites:** [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) · [Propositional logic](../01-discrete-mathematics/propositional-logic.md) · **Leads to:** [Context-free languages and PDA](context-free-languages-and-pda.md) · [Pumping lemma](pumping-lemma.md) · [Lexical analysis](../12-compiler-design/lexical-analysis.md)

## Quick glance

- A **language** is a set of strings over an alphabet Σ. A **regular language** is one accepted by a DFA, equivalently an NFA, ε-NFA, or described by a regular expression.
- DFA = (Q, Σ, δ, q0, F) with δ: Q×Σ → Q (total). NFA has δ: Q×Σ → 2^Q. ε-NFA adds ε-moves. All three accept exactly the regular languages.
- **Subset construction:** an NFA with n states gives a DFA with at most 2^n states (tight in the worst case, e.g. "n-th symbol from the end is 1").
- **Minimal DFA** is unique up to renaming; its number of states = number of Myhill–Nerode classes. Minimise by removing unreachable states then merging indistinguishable ones (partition refinement).
- Quick state counts: ends with a fixed string of length k → k+1; contains a fixed substring of length k → k+1; starts with a fixed string of length k → k+2 (dead state); |w| mod k → k; binary divisible by odd n → n (for n = 2^k·m, m odd: m + k, e.g. 4 → 3); k-th from end → 2^k.
- Closure: regular languages are closed under union, intersection, complement, concatenation, Kleene star, reversal, difference, homomorphism, inverse homomorphism, quotient. All the usual decision problems are decidable.
- Regular = finite memory. **a^n b^n is not regular** (needs unbounded counting); a^n b^m is regular.
- #1 trap: a DFA must have a transition for every (state, symbol); counting minimal states forgets the **dead (trap) state**.

## 1. Alphabets, strings and languages

An **alphabet** Σ is a finite non-empty set of symbols. A **string** is a finite sequence of symbols; |w| is its length; ε is the empty string (|ε| = 0). Σ* is the set of all strings over Σ (including ε), Σ+ = Σ* − {ε}. A **language** is any subset of Σ*.

Operations on languages L1, L2:

| Operation | Definition | Example (L1 = {a, ab}, L2 = {b, ε}) |
| --- | --- | --- |
| Union | L1 ∪ L2 | {a, ab, b, ε} |
| Concatenation | L1·L2 = {xy : x∈L1, y∈L2} | {ab, a, abb, ab} = {a, ab, abb} |
| Kleene star | L* = L⁰ ∪ L¹ ∪ L² ∪ … (L⁰ = {ε}) | {ε, a, ab, aa, aab, aba, abab, …} |
| Plus | L+ = L·L* | L* without ε unless ε ∈ L |
| Complement | Σ* − L | |
| Reversal | L^R = {w^R : w∈L} | {a, ba} |

Facts: **∅* = {ε}**, and ∅·L = ∅ (concatenation with the empty language kills everything), {ε}·L = L. |Σ^n| = |Σ|^n. If |L1| = m and |L2| = n then |L1·L2| ≤ mn (equality fails when different splits give the same string, as above: 4 products, 3 distinct strings).

## 2. Regular expressions

A **regular expression (RE)** over Σ is built from: ∅ (the empty language), ε, each a∈Σ, and if r, s are REs then (r+s) or (r|s) (union), (rs) (concatenation), (r*) (star). Precedence: **star > concatenation > union**.

| RE | Language |
| --- | --- |
| a | {a} |
| a + b | {a, b} |
| ab | {ab} |
| a* | {ε, a, aa, …} |
| (a+b)* | all strings over {a,b} |
| a*b* | zero or more a's followed by zero or more b's |
| (a+b)*abb | strings ending with abb |
| (b + ab)*(a + ε) | strings with no two consecutive a's |
| (aa)* | even number of a's (only a's) |
| b*(ab*ab*)* | even number of a's |

**Identities worth knowing** (each verified by exhaustive checking on all strings up to length 10):

| Identity | Remark |
| --- | --- |
| r + r = r, r + ∅ = r, rε = r, r∅ = ∅ | basic |
| (r*)* = r*, ∅* = ε* = ε | |
| r*r* = r*, r*r = rr* = r+ | |
| (r + ε)* = r* | |
| **(a+b)* = (a*b*)* = (a*+b*)* = a*(ba*)* = (a*b)*a*** | all give Σ* |
| (ab)*a = a(ba)* | shifting rule: r(sr)* = (rs)*r |
| (a+b)*a(a+b)* | contains an a |

**Non-identities (traps):** (a+b)* ≠ a* + b* (the right side has no "ab"); (ab)* ≠ a*b*; (a + ab)* ≠ (a+b)* (no string starting with b other than ε); (a+b)*abb(a+b)* ≠ (a+b)*abb.

**Describing languages in words.** Strategy: find the structure that repeats.

- Strings over {0,1} with at least one 1: 0*1(0+1)*.
- Strings over {a,b} with exactly two a's: b*ab*ab*.
- Strings over {a,b} containing "aa": (a+b)*aa(a+b)*.
- Third symbol from the right is a: (a+b)*a(a+b)(a+b).
- Even length: ((a+b)(a+b))*.

## 3. Deterministic finite automata (DFA)

**Intuition.** A machine with finite memory: it reads the input one symbol at a time, always in exactly one state, and accepts if it ends in a final state. Memory = the state; what the machine must "remember" is what it needs to decide the future.

**Definition.** M = (Q, Σ, δ, q0, F): finite set of states Q, input alphabet Σ, **total** transition function δ: Q×Σ→Q, start state q0, set of final states F ⊆ Q. Extended: δ*(q, ε) = q, δ*(q, wa) = δ(δ*(q,w), a). L(M) = {w : δ*(q0, w) ∈ F}.

**Worked example: strings over {a,b} with an even number of a's and an odd number of b's.** Remember (a-parity, b-parity): 4 states.

```text
          a             b
(E,E) -> (O,E)        (E,O)
(O,E) -> (E,E)        (O,O)
(E,O) -> (O,O)        (E,E)
(O,O) -> (E,O)        (O,E)
start = (E,E),  final = (E,O)
```

Check "aab": (E,E) -a-> (O,E) -a-> (E,E) -b-> (E,O): accepted (2 a's, 1 b). Check "ab": (E,E)→(O,E)→(O,O): rejected (one a). **A product of two independent counters needs m×n states** (here 2×2 = 4, all distinguishable).

**Acceptance examples.**

| Language over {a,b} | Minimal DFA idea | States |
| --- | --- | --- |
| ends with ab | remember longest suffix that is a prefix of "ab" | 3 |
| contains aba | longest matched prefix of "aba", then absorbing | 4 |
| no two consecutive a's | last symbol was a? + dead state | 3 |
| starts with ab | q0 → q1 → q2(accept, loops), plus dead | 4 |

## 4. NFA and ε-NFA

**Intuition.** An NFA may be in several states at once (or, equivalently, may guess). It accepts if **some** computation path ends in a final state. ε-moves let it change state without reading input.

**Definition.** NFA: δ: Q×Σ → 2^Q. ε-NFA: δ: Q×(Σ∪{ε}) → 2^Q. **ε-closure(q)** = set of states reachable from q using only ε-moves (including q).

**Power.** Nondeterminism adds no recognising power for finite automata (it does for PDAs; it is unknown/irrelevant for TMs). It can reduce the number of states exponentially.

### 4.1 Subset construction (NFA → DFA), fully worked

NFA N for (a+b)*abb: states 0,1,2,3; start 0; final {3}.

```text
δ(0,a) = {0,1}   δ(0,b) = {0}
δ(1,a) = ∅       δ(1,b) = {2}
δ(2,a) = ∅       δ(2,b) = {3}
δ(3,*) = ∅
```

Algorithm: start with {start}; for each new subset S and each symbol x compute ∪ δ(q,x) for q∈S; a subset is final if it contains a final state of N.

| Subset | on a | on b | Final? |
| --- | --- | --- | --- |
| {0} (A) | {0,1} (B) | {0} (A) | no |
| {0,1} (B) | {0,1} (B) | {0,2} (C) | no |
| {0,2} (C) | {0,1} (B) | {0,3} (D) | no |
| {0,3} (D) | {0,1} (B) | {0} (A) | **yes** |

Only 4 of the 16 possible subsets are reachable, so the DFA has 4 states (and it is minimal, since the four states remember the matched prefix length 0, 1, 2, 3 of "abb"). Always construct **only reachable subsets**.

### 4.2 ε-NFA → NFA/DFA, fully worked

ε-NFA for a*b*c*: states q0,q1,q2; q0 -a→ q0; q0 -ε→ q1; q1 -b→ q1; q1 -ε→ q2; q2 -c→ q2; final state {q2} (q0 and q1 reach it by ε-moves).

ε-closures: ε-cl(q0) = {q0,q1,q2}, ε-cl(q1) = {q1,q2}, ε-cl(q2) = {q2}. Start of DFA = ε-cl(q0) = {q0,q1,q2}. For symbol x: take δ over all states in the subset, then ε-close.

| Subset | on a | on b | on c | Final? |
| --- | --- | --- | --- | --- |
| {q0,q1,q2} (S) | ε-cl{q0} = S | ε-cl{q1} = {q1,q2} (T) | ε-cl{q2} = {q2} (U) | yes (has q2) |
| {q1,q2} (T) | ∅ (dead) | T | U | yes |
| {q2} (U) | ∅ | ∅ | U | yes |
| ∅ (dead) | ∅ | ∅ | ∅ | no |

Four states including the dead one. This matches the intuitive minimal DFA of a*b*c* (phase a, phase b, phase c, dead).

### 4.3 Key relations

| Conversion | State blow-up |
| --- | --- |
| ε-NFA → NFA | none (same states) |
| NFA → DFA | up to 2^n (tight) |
| DFA → NFA | none (a DFA is an NFA) |
| RE of length n → ε-NFA (Thompson) | O(n) states |
| DFA → RE | can be exponential in size |
| DFA complement | swap final/non-final (DFA must be complete); same states |
| NFA complement | determinise first, then swap |

**Thompson's construction (brief).** Build an ε-NFA by induction on the RE with a single start and single final state per fragment: symbol a: s -a→ f. Union r+s: new start with ε to both starts, both finals ε to new final. Concatenation: ε from final of r to start of s. Star: new start/final with ε-loop back from final to start of r and ε-bypass start→final. At most 2 states per symbol/operator, so ≤ 2|r| states. The same fragments appear in lexer generators ([Lexical analysis](../12-compiler-design/lexical-analysis.md)).

## 5. DFA minimisation

**Idea.** Two states p, q are **distinguishable** if some string w leads one to a final state and the other to a non-final state. Indistinguishable states can be merged. **Steps:** (1) delete states unreachable from q0; (2) partition into {final}, {non-final}; (3) repeatedly split a block if two of its states go to different blocks on some symbol; (4) stop when stable; each block becomes one state. This is the same as the table-filling algorithm.

### Worked example (8 states, one unreachable)

Alphabet {0,1}, start A, final {C}.

| State | on 0 | on 1 |
| --- | --- | --- |
| A | B | F |
| B | G | C |
| C | A | C |
| D | C | G |
| E | H | F |
| F | C | G |
| G | G | E |
| H | G | C |

**Step 1: reachability.** From A: B, F; from B: G, C; from F: C, G; from G: E; from C: A; from E: H, F; from H: G, C. Reachable = {A,B,C,E,F,G,H}. **D is unreachable** (nothing goes to D): delete it.

**Step 2: initial partition.** P0 = { {C} , {A,B,E,F,G,H} }.

**Step 3: refine.** Describe each state by (block of δ(·,0), block of δ(·,1)); N = the big non-final block, C = {C}.

P0 → signatures: A:(N,N), B:(N,C), E:(N,N), F:(C,N), G:(N,N), H:(N,C).
P1 = { {C}, {A,E,G}, {B,H}, {F} }.

Signatures w.r.t. P1: A: 0→B∈{B,H}, 1→F∈{F}; E: 0→H∈{B,H}, 1→F; G: 0→G∈{A,E,G}, 1→E∈{A,E,G}. So G differs from A and E. B: 0→G∈{A,E,G}, 1→C; H: 0→G, 1→C: same.
P2 = { {C}, {A,E}, {B,H}, {F}, {G} }.

Signatures w.r.t. P2: A:(B,F)→({B,H},{F}); E:(H,F)→({B,H},{F}): same. B:(G,C), H:(G,C): same. **Stable.**

**Result: 5 states** {A,E}, {B,H}, {C}, {F}, {G}; start {A,E}, final {C}. Transitions: {A,E}: 0→{B,H}, 1→{F}; {B,H}: 0→{G}, 1→{C}; {C}: 0→{A,E}, 1→{C}; {F}: 0→{C}, 1→{G}; {G}: 0→{G}, 1→{A,E}. (Verified by program.)

### Counting states of the minimal DFA (frequent GATE theme)

All counts below were verified by computing Myhill–Nerode classes by brute force.

| Language | Minimal DFA states (complete) |
| --- | --- |
| Binary numbers divisible by n (leading zeros allowed) | n for odd n (remainders 0..n−1; δ(r,b) = (2r+b) mod n); for n = 2^k·m with m odd: **m + k** (n=4 → 3, 6 → 4, 8 → 4, 10 → 6, 12 → 5), since for even n some remainders merge |
| \|w\| ≡ r (mod k) | k |
| #a ≡ r (mod m) and #b ≡ s (mod n) | m·n |
| Ends with a fixed string of length k (e.g. ab) | k+1 |
| Contains a fixed substring of length k (e.g. aba) | k+1 |
| Starts with a fixed string of length k | k+2 |
| At least k a's | k+1 |
| At most k a's | k+2 |
| \|w\| ≥ k | k+1 |
| \|w\| = k exactly | k+2 |
| k-th symbol from the end is 1 (Σ={0,1}) | **2^k** (NFA needs only k+1) |
| a* b* c* | 4 |

For "binary divisible by 3": states r0, r1, r2; δ(r,x) = (2r+x) mod 3: r0 -0→ r0, r0 -1→ r1, r1 -0→ r2, r1 -1→ r0, r2 -0→ r1, r2 -1→ r2. Start and final r0. Test 110 = 6: r0→r1→r0→r0 accepted.

## 6. Regular expression ⇄ finite automaton

### 6.1 RE → NFA
Use Thompson (above) or build directly by intuition. Example: (a+b)*abb → the NFA of §4.1.

### 6.2 DFA → RE by state elimination, worked

DFA for "even number of a's": q0 (start, final) -a→ q1, q0 -b→ q0, q1 -a→ q0, q1 -b→ q1.

Method: add a new start S and new final F with ε-edges (S→q0, q0→F). Eliminate states one by one: to remove state r, for every pair (p, s) with p→r and r→s add a direct edge labelled R(p,r)·R(r,r)*·R(r,s), unioned with any existing p→s edge.

1. Eliminate q1: the only path through q1 is q0 -a→ q1 -(b loop)→ q1 -a→ q0, giving a new loop on q0: a b* a. Now q0 has loops b and ab*a, i.e. loop (b + ab*a).
2. Graph: S -ε→ q0, q0 loop (b + ab*a), q0 -ε→ F.
3. Eliminate q0: S -ε (b+ab*a)* ε→ F.

**RE = (b + ab*a)\***. Verified by comparing with parity of a's on all strings up to length 9.

### 6.3 Arden's theorem
If R = Q + RP and ε ∉ P, then R = QP*. To solve a DFA, write one equation per state (state = union of (predecessor · symbol)), plus ε for the start, then eliminate. Useful when equations are small.

## 7. Mealy and Moore machines (brief)

Finite automata **with output**, no final states.

| | Moore | Mealy |
| --- | --- | --- |
| Output depends on | state only | state and current input |
| Output length for input of length n | n+1 (includes initial state's output) | n |
| Number of states | possibly more | possibly fewer |

Example (Mealy, 1's complement of a binary string): one state q, transitions 0/1 and 1/0. Input 0110 → output 1001. A Moore machine for the same job needs extra states to hold the output. Every Moore machine converts to an equivalent Mealy machine with the same state count, and a Mealy machine converts to Moore with at most (states × |output alphabet|) states. These machines are the ancestors of the sequential circuits in [Sequential circuits](../09-digital-logic/sequential-circuits.md).

## 8. Myhill–Nerode theorem

Define x ≡_L y iff for all z, (xz ∈ L ⟺ yz ∈ L). **L is regular ⟺ ≡_L has finitely many equivalence classes, and the number of classes equals the number of states of the minimal DFA.**

Use it two ways:

- **Count states:** each class is "a distinct situation the machine must remember" (e.g. for "ends with ab": classes of strings ending in ab, ending in a, others).
- **Prove non-regular:** for L = {a^n b^n}, the strings a, aa, aaa, … are pairwise inequivalent (a^i b^i ∈ L but a^j b^i ∉ L for i≠j), giving infinitely many classes. Intuition: a finite machine cannot count unboundedly.

## 9. Regular languages: closure and decision properties

**Closure.** Regular languages are closed under:

| Operation | Closed? | How |
| --- | --- | --- |
| Union, intersection | yes | product automaton |
| Complement | yes | complete DFA, swap F |
| Difference L1 − L2 | yes | L1 ∩ complement(L2) |
| Concatenation, Kleene star, plus | yes | via NFA with ε-moves |
| Reversal | yes | reverse arrows, swap start/final |
| Homomorphism, inverse homomorphism | yes | substitute symbols / simulate |
| Prefix, suffix, substring sets; quotient L1/L2 | yes | modify start/final states |
| Infinite union | **no** | {a^n b^n} = ∪ {a^k b^k} each finite (regular) |
| Subset of a regular language | **no** | {a^n b^n} ⊂ a*b* |

**Decision properties** (all decidable for regular languages given a DFA/NFA/RE):

| Question | Method |
| --- | --- |
| Membership w ∈ L | simulate in O(\|w\|) on a DFA |
| Emptiness | is any final state reachable? |
| Finiteness | is there a cycle on a path from start to a final state (a "useful" cycle)? |
| Equivalence L1 = L2 | minimise both and compare, or check L1 Δ L2 = ∅ |
| Universality L = Σ* | complement is empty |
| Subset L1 ⊆ L2 | L1 ∩ complement(L2) = ∅ |

**Regular or not? Quick test.**

| Language | Regular? | Reason |
| --- | --- | --- |
| a^n b^m | yes | a*b* |
| a^n b^n | no | counting |
| a^n b^n, n ≤ 1000 | yes | finite |
| ww, w^R w (w ∈ {a,b}*) | no | must remember w |
| strings with equal number of a's and b's | no | counting |
| strings where the number of a's is divisible by 3 | yes | mod counter |
| a^(n²), a^(2^n), a^p (p prime) | no | gaps grow ([Pumping lemma](pumping-lemma.md)) |
| a^n b^m with n+m even | yes | parity only |
| strings of the form x c x^R over {a,b,c} | no | match |
| binary strings whose value is a multiple of 7 | yes | 7 states |
| {w : w has the same number of occurrences of "ab" and "ba"} | yes | difference is always 0 or ±1: depends only on first and last symbol |

Heuristic: **if the machine must compare two unbounded quantities, it is not regular; if it only tracks a bounded amount (parity, last k symbols, a bounded count), it is.** Do not trust a language that "looks like" it needs counting; check (the last row is regular).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Subset construction | ≤ 2^n DFA states from n-state NFA | NFA→DFA size questions |
| Minimal DFA | unique, = number of Myhill–Nerode classes | counting states |
| Product automaton | \|Q1\|·\|Q2\| states | intersection/union |
| Complement | swap F in a **complete** DFA | complement questions |
| Reversal | reverse edges, swap start and finals (NFA) | reversal; minimal DFA of L^R can be exponentially larger |
| Binary divisible by n | n states if n odd; m + k if n = 2^k·m | classic counts |
| Ends with length-k string | k+1 states | classic counts |
| k-th from end | 2^k DFA, k+1 NFA | blow-up questions |
| Moore vs Mealy | output length n+1 vs n | machine questions |
| (a+b)* | = (a*b*)* = (a*+b*)* = a*(ba*)* = (a*b)*a* | RE equivalence |
| Arden | R = Q + RP ⟹ R = QP* | solving state equations |

## GATE traps

- **Forgetting the dead state.** "Starts with ab" needs 4 states, not 3; "at most k a's" needs k+2. Whether a question counts it depends on "complete DFA" (default in GATE for minimal DFA is complete unless it says otherwise; read the options to see which count appears).
- **Complementing an NFA by swapping final states** is wrong; determinise first. Complementing an incomplete DFA by swapping also fails: add the dead state first.
- (a+b)* vs a*+b*, (ab)* vs a*b*: do not "distribute" the star.
- Minimal DFA of the reversal can be exponentially larger; the minimal **NFA** is not unique.
- "Is the language regular?" Many languages that look like they need counting are regular (a^n b^m, parity conditions, finite sets). A subset of a regular language need not be regular; a superset need not either.
- Epsilon: a string accepted by an ε-NFA may use ε-moves at the end after the last symbol; closure must be applied at the start and after every symbol.
- Moore output has length n+1 for an n-symbol input; Mealy n.
- Finiteness test: a cycle only matters if it lies between the start and some final state (otherwise the cycle is irrelevant).

## Connections

- [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) — languages are sets; Myhill–Nerode is an equivalence relation with finite index.
- [Lexical analysis](../12-compiler-design/lexical-analysis.md) — token patterns are regular expressions compiled to DFAs.
- [Sequential circuits](../09-digital-logic/sequential-circuits.md) — Mealy/Moore machines are exactly state machines in hardware.
- [Context-free languages and PDA](context-free-languages-and-pda.md) — adds a stack; strictly more powerful.
- [Pumping lemma](pumping-lemma.md) — the standard proof technique for non-regularity.
- [Turing machines and undecidability](turing-machines-and-undecidability.md) — decision properties here are decidable; they fail for bigger classes.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — emptiness/finiteness are reachability and cycle questions on the transition graph.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — a DFA scanning input is the model behind linear-time pattern matching.

## Practice

**Q1 (MCQ).** Which regular expression generates strings over {a,b} with no two consecutive a's?
(A) (b+ab)* (B) (b+ab)*(a+ε) (C) (a+b)*bb (D) b*(ab*)*

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Write the string as blocks each a b or ab; the string may end with a lone a. (A) misses strings ending in a such as "ba". (C) forces ending in bb. (D) = (a+b)*. Verified by brute force over all strings up to length 10: the RE (b+ab)*(a+ε) matches exactly the strings with no "aa".

</details>

**Q2 (NAT).** Number of states in the minimal complete DFA over {0,1} accepting binary strings that are divisible by 5.

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** States are remainders 0..4; δ(r,b) = (2r+b) mod 5. All five are distinguishable (e.g., by suffix strings that reach remainder 0 from each). Brute-force Myhill–Nerode count gives 5.

</details>

**Q3 (NAT).** Minimal number of states in a complete DFA for "the third symbol from the end is 1" over {0,1}.

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:** The DFA must remember the last 3 symbols: 2³ = 8 classes, all distinguishable (two different 3-bit windows differ at some position; a suitable suffix of length ≤ 2 shifts that position to the third-from-end slot). The NFA needs only 4 states. Verified by computing the classes by program.

</details>

**Q4 (MSQ).** Which of the following are regular? (A) {a^n b^m : n+m is even} (B) {a^n b^n : n ≤ 100} (C) {w ∈ {a,b}* : #a(w) = #b(w)} (D) {a^n : n is a multiple of 3 or 5}

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:** (A) only parity of n+m matters: a*b* intersected with even length. (B) finite. (C) needs unbounded counting, not regular (a^n b^n = C ∩ a*b*; regular ∩ regular would be regular). (D) union of (aaa)* and (aaaaa)*.

</details>

**Q5 (NAT).** Apply subset construction to the NFA with states {p,q,r}, start p, final {r}, δ(p,0)={p,q}, δ(p,1)={p}, δ(q,0)=∅, δ(q,1)={r}, δ(r,·)=∅. How many DFA states are reachable?

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** {p} -0→ {p,q}, {p} -1→ {p}. {p,q} -0→ {p,q}, {p,q} -1→ {p,r}. {p,r} -0→ {p,q}, {p,r} -1→ {p}. Reachable subsets: {p}, {p,q}, {p,r}. Only {p,r} is final. The language is "ends with 01" (p -0→ q -1→ r), whose minimal DFA has k+1 = 3 states, consistent.

</details>

**Q6 (MCQ).** Let L1, L2 be regular. Which of the following is NOT necessarily regular? (A) L1 ∩ L2 (B) L1 − L2 (C) a subset of L1 (D) L1^R

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Regular languages are closed under intersection, difference and reversal. Any language is a subset of Σ*, which is regular, so subsets are not guaranteed regular.

</details>

**Q7 (NAT).** A DFA over {a,b} has states q0..q3 with δ(qi,a) = q((i+1) mod 4), δ(qi,b) = qi; start q0, final {q0, q2}. How many states does the minimal DFA have?

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** The machine counts a's mod 4 and accepts when the count is 0 or 2 mod 4, i.e. when #a is even. P0 = {q0,q2} | {q1,q3}. Under a, q0→q1 and q2→q3 (same block), and q1→q2, q3→q0 (same block); b is a self-loop. The partition is stable, so q0≡q2 and q1≡q3: 2 states.

</details>

**Q8 (MCQ).** The minimal DFA for L has n states. How many states does a minimal DFA for the complement of L have?
(A) n (B) n−1 (C) n+1 (D) 2n

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** Complement swaps final/non-final in the complete DFA; distinguishability is unchanged, so the state count stays n (assuming the minimal DFA is the complete one).

</details>
