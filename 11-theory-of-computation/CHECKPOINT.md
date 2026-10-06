# Theory of Computation: Checkpoint

Attempt without notes, then open the solutions. Questions run easy to hard; several combine chapters.

**Q1 (MCQ).** Which regular expression is equal to (a+b)*?
(A) a*+b* (B) (a*b*)* (C) (ab)* (D) a*b*

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** (a*b*)* contains a and b individually and any interleaving, so it is Σ*. (A) lacks "ab", (C) and (D) are far smaller.

</details>

**Q2 (NAT).** Number of states in the minimal complete DFA over {0,1} accepting binary numbers divisible by 12.

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** 12 = 2²·3. Divisibility by 12 means divisible by 3 (3 remainders) and the last two bits are 00. The minimal DFA has m + k = 3 + 2 = 5 states (not 12), because remainders that behave identically merge. Confirmed by computing Myhill–Nerode classes by brute force.

</details>

**Q3 (MCQ).** Which of the following is NOT necessarily a context-free language, given that L1 and L2 are context-free and R is regular?
(A) L1 ∪ L2 (B) L1 · L2 (C) L1 ∩ R (D) L1 ∩ L2

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:** CFLs are closed under union, concatenation and intersection with regular languages, not under intersection (a^n b^n c^m ∩ a^m b^n c^n = a^n b^n c^n).

</details>

**Q4 (NAT).** A grammar in CNF generates a string of length 10. How many rule applications are in its derivation?

<details><summary>Answer</summary>

**Answer:** 19  
**Solution:** 2n − 1 = 19 (9 binary rules and 10 terminal rules).

</details>

**Q5 (NAT).** Minimal complete DFA states for strings over {a,b} that end with "ab" **and** have even length.

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** Track (length parity, longest suffix match). Reachable distinguishable classes reduce to 4 (verified by computing Myhill–Nerode classes by brute force). A naive product of the 3-state "ends with ab" machine and the 2-state parity machine gives 6, so minimisation matters.

</details>

**Q6 (MSQ).** Which of the following languages are regular?
(A) {a^n b^m : n + m is even} (B) {w ∈ {0,1}* : #0(w) = #1(w)} (C) {w ∈ {0,1}* : the number of occurrences of 01 equals the number of occurrences of 10} (D) {0^n 1^m : n ≤ m}

<details><summary>Answer</summary>

**Answer:** A, C  
**Solution:** (A) a*b* intersected with even length. (C) In any binary string the counts of 01 and 10 differ by at most 1, and they are equal exactly when the first and last symbols are equal (or the string is empty/constant), which a small DFA checks. (B) needs unbounded counting (∩ 0*1* gives 0^n 1^n). (D) fails by pumping on 0^p 1^p with i = 2 (more 0's than 1's).

</details>

**Q7 (MCQ).** Which is true about the language L = {ww^R : w ∈ {a,b}*}?
(A) regular (B) DCFL but not regular (C) CFL but not DCFL (D) not context-free

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** A PDA pushes the first half and pops against the second after guessing the middle; a deterministic PDA cannot know where the middle is. Not regular (a^p b b a^p pumping).

</details>

**Q8 (NAT).** An NFA has 6 states. What is the maximum number of states in the DFA produced by subset construction (counting the empty subset)?

<details><summary>Answer</summary>

**Answer:** 64  
**Solution:** At most 2^6 = 64 subsets, and worst-case 6-state NFAs exist that reach this bound (it is tight; the "k-th symbol from the end" family gives 2^k DFA states from k+1 NFA states).

</details>

**Q9 (MCQ).** In the CYK/derivation setting, the grammar S → SS | a is used. The number of different parse trees for a^5 is
(A) 5 (B) 14 (C) 42 (D) 24

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** t(n) is the Catalan number C(n−1): t(1)=1, t(2)=1, t(3)=2, t(4)=5, t(5)=t1t4+t2t3+t3t2+t4t1 = 5+2+2+5 = 14.

</details>

**Q10 (MSQ).** Which of the following are decidable?
(A) Is L(G) = ∅ for a CFG G? (B) Is a given CFG ambiguous? (C) Is w ∈ L for a given context-sensitive grammar and string w? (D) Does a given TM halt on the empty input?

<details><summary>Answer</summary>

**Answer:** A, C  
**Solution:** (A) generating-symbol test. (C) an LBA has finitely many configurations on a fixed input, so membership is decidable. (B) CFG ambiguity is undecidable (PCP). (D) the halting problem is undecidable.

</details>

**Q11 (MCQ).** Let L be a language such that L is recursively enumerable and its complement is recursively enumerable too. Which of the following cannot be concluded?
(A) L is recursive (B) The complement of L is recursive (C) L is context-sensitive (D) L is accepted by a TM that halts on every input

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Both RE ⇒ REC, so (A), (B), (D) hold. A recursive language need not be context-sensitive (CSL ⊊ REC).

</details>

**Q12 (MCQ).** Which language is NOT context-free?
(A) {a^n b^m c^n d^m : n, m ≥ 1} (B) {a^n b^n c^m : n, m ≥ 0} (C) {a^i b^j c^k : i ≠ j} (D) the complement of {ww^R}

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** (A) has crossing dependencies (a↔c, b↔d); pumping a^p b^p c^p d^p: a window of ≤ p symbols touches at most two adjacent blocks, never both partners. (B) one comparison. (C) compare then branch (DCFL). (D) nondeterministically guess a position where the two halves mismatch, a CFL.

</details>

**Q13 (MCQ).** Which property of a Turing machine M is undecidable by Rice's theorem?
(A) M has exactly 7 states (B) M halts within 1000 steps on input 01 (C) L(M) contains at least two strings (D) The encoding ⟨M⟩ is a palindrome

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** (C) is a non-trivial property of the language L(M) (RE but not decidable). (A) and (D) are syntactic; (B) is decidable by simulation for a bounded number of steps.

</details>

**Q14 (NAT).** To test whether the language of a 9-state DFA is infinite, it suffices to look for an accepted string whose length lies in [9, N]. What is the smallest N guaranteed to work (N = 2p − 1)?

<details><summary>Answer</summary>

**Answer:** 17  
**Solution:** L(M) is infinite iff M accepts some string with p ≤ |w| ≤ 2p − 1. For p = 9 the range is 9 ≤ |w| ≤ 17, so the upper limit is 17.

</details>

**Q15 (MSQ).** Which statements about languages over a one-letter alphabet are true?
(A) Every context-free language over {a} is regular. (B) {a^n : n is prime} is context-free. (C) {a^(n²)} is context-sensitive. (D) {a^(2^n)} is decidable.

<details><summary>Answer</summary>

**Answer:** A, C, D  
**Solution:** (A) true (a classical consequence of the CFL pumping lemma). (B) false: primes are not regular, hence not CFL. (C) true: an LBA can check square length by repeated addition. (D) true: a TM can test whether a length is a power of 2 by repeated halving.

</details>

**Scoring.** ≥ 80% (12+ correct) → move on to [Compiler design](../12-compiler-design/README.md). 60–80% → re-read the chapters for the questions you missed (Q1, Q2, Q5, Q6, Q8: [Regular languages](regular-languages-and-finite-automata.md); Q3, Q4, Q7, Q9, Q10, Q12: [CFL and PDA](context-free-languages-and-pda.md); Q12, Q14, Q15: [Pumping lemma](pumping-lemma.md); Q10, Q11, Q13, Q15: [Turing machines and undecidability](turing-machines-and-undecidability.md)). < 60% → redo all four chapters, starting with the worked examples.
