# Probability basics

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Counting: permutations and combinations; Probability axioms; Sample spaces and events; Independent events; Mutually exclusive events; Marginal probability; Conditional probability; Joint probability; Bayes theorem
> **Prerequisites:** [Combinatorics](../01-discrete-mathematics/combinatorics.md) · [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) · **Leads to:** [Random variables and moments](random-variables-and-moments.md)

## Quick glance

- **Classical probability:** if all outcomes of a finite sample space $S$ are equally likely, $P(A)=|A|/|S|$. The whole battle is choosing a sample space in which outcomes really are equally likely.
- **Axioms:** $P(A)\ge 0$, $P(S)=1$, and $P(\cup A_i)=\sum P(A_i)$ for pairwise disjoint events.
- **Union:** $P(A\cup B)=P(A)+P(B)-P(A\cap B)$; complement: $P(A^c)=1-P(A)$ (use it for "at least one").
- **Conditional:** $P(A\mid B)=P(A\cap B)/P(B)$; multiplication rule $P(A\cap B)=P(B)\,P(A\mid B)$.
- **Independent:** $P(A\cap B)=P(A)P(B)$. **Mutually exclusive:** $A\cap B=\varnothing$. **Two events with non-zero probability cannot be both** — disjoint events are dependent.
- **Total probability:** $P(A)=\sum_i P(A\mid B_i)P(B_i)$ for a partition $\{B_i\}$.
- **Bayes:** $P(B_j\mid A)=\dfrac{P(A\mid B_j)P(B_j)}{\sum_i P(A\mid B_i)P(B_i)}$.
- **#1 trap:** confusing $P(A\mid B)$ with $P(B\mid A)$ (the disease-test paradox: a 95%-accurate test on a 1%-prevalence disease gives only 16% positive predictive value).

## 1. Counting for probability

Probability on a finite sample space reduces to counting. The toolkit (full treatment in [Combinatorics](../01-discrete-mathematics/combinatorics.md)):

| Situation | Count |
|---|---|
| Arrange $r$ of $n$ distinct items in order | $P(n,r)=\dfrac{n!}{(n-r)!}$ |
| Choose $r$ of $n$ distinct items, order irrelevant | $\binom{n}{r}=\dfrac{n!}{r!(n-r)!}$ |
| $r$ choices from $n$ types with repetition, order matters | $n^r$ |
| $r$ choices from $n$ types with repetition, order irrelevant | $\binom{n+r-1}{r}$ (stars and bars) |
| Arrange $n$ items with repeats $n_1,\dots,n_k$ | $\dfrac{n!}{n_1!\cdots n_k!}$ |
| Circular arrangements of $n$ distinct items | $(n-1)!$ |

**Rule of thumb:** *count the favourable and total outcomes in the same way* — both ordered or both unordered. Mixing the two is the commonest counting error.

**Sample space matters.** Two fair coins: the outcomes $\{HH,HT,TH,TT\}$ are equally likely, so $P(\text{2 heads})=1/4$. The "three outcomes" $\{0,1,2\text{ heads}\}$ are *not* equally likely ($1/4,1/2,1/4$). Likewise for two dice, treat them as distinguishable: $36$ equally likely ordered pairs, not $21$ unordered ones.

**Worked example 1 — two dice.** Roll two fair dice. Find $P(\text{sum}=7)$ and $P(\text{sum}\ge 10)$.

- $|S|=6\times 6=36$.
- Sum 7: $(1,6),(2,5),(3,4),(4,3),(5,2),(6,1)$ → 6 outcomes → $6/36=1/6$.
- Sum $\ge10$: sum 10: $(4,6),(5,5),(6,4)$ = 3; sum 11: $(5,6),(6,5)$ = 2; sum 12: $(6,6)$ = 1 → 6 outcomes → $6/36=1/6$.

**Worked example 2 — drawing without replacement.** Two cards are drawn from a 52-card deck. $P(\text{both aces})$:

- Unordered: favourable $\binom42=6$, total $\binom{52}{2}=1326$ → $6/1326=1/221$.
- Ordered check: $\frac{4}{52}\cdot\frac{3}{51}=\frac{12}{2652}=\frac1{221}$. ✓

**Worked example 3 — birthday problem.** $n$ people, 365 equally likely birthdays. $P(\text{all distinct})=\dfrac{365\cdot364\cdots(365-n+1)}{365^n}$.
For $n=23$: $\prod_{i=0}^{22}\frac{365-i}{365}\approx 0.4927$, so $P(\text{some match})\approx \mathbf{0.507}$. (For $n=30$ it is $0.706$.) The same calculation gives expected collisions in hashing — see [Hashing](../08-algorithms/hashing.md).

**Worked example 4 — "at least one" via the complement.** Roll a die 4 times; $P(\text{at least one six})=1-(5/6)^4=1-0.4823=\mathbf{0.5177}$.

## 2. Sample spaces, events and the axioms

- **Experiment:** a process with uncertain outcome. **Sample space $S$** (or $\Omega$): the set of all outcomes. **Event:** a subset of $S$. **Elementary event:** one outcome.
- Event operations are set operations: $A\cup B$ (A or B), $A\cap B$ (A and B), $A^c$ (not A), $A\setminus B=A\cap B^c$. De Morgan: $(A\cup B)^c=A^c\cap B^c$.
- **Mutually exclusive (disjoint):** $A\cap B=\varnothing$. **Exhaustive:** $\cup A_i=S$. A **partition** is mutually exclusive *and* exhaustive.

**Kolmogorov axioms.**

1. $P(A)\ge0$ for every event $A$.
2. $P(S)=1$.
3. For pairwise disjoint $A_1,A_2,\dots$: $P(\bigcup_i A_i)=\sum_i P(A_i)$.

**Consequences (all derived from the axioms):**

| Fact | Reason |
|---|---|
| $P(\varnothing)=0$ | $S=S\cup\varnothing\cup\varnothing\cdots$ |
| $P(A^c)=1-P(A)$ | $A$ and $A^c$ partition $S$ |
| $0\le P(A)\le1$ | $P(A^c)\ge0$ |
| $A\subseteq B\Rightarrow P(A)\le P(B)$ | $B=A\cup(B\setminus A)$ |
| $P(A\cup B)=P(A)+P(B)-P(A\cap B)$ | $A\cup B=A\cup(B\setminus A)$, disjoint |
| $P(A\setminus B)=P(A)-P(A\cap B)$ | partition of $A$ |
| Boole/union bound: $P(\cup A_i)\le\sum P(A_i)$ | overlaps are double counted |

**Inclusion–exclusion for three events**

$$P(A\cup B\cup C)=P(A)+P(B)+P(C)-P(A\cap B)-P(A\cap C)-P(B\cap C)+P(A\cap B\cap C)$$

In general: add singles, subtract pairs, add triples, … alternating.

**Worked example 5 — inclusion–exclusion.** A number is chosen uniformly from $1,\dots,100$. $P(\text{divisible by 2, 3 or 5})$?

- $|A_2|=50,\ |A_3|=33,\ |A_5|=20$.
- Pairs: $|A_2\cap A_3|=\lfloor100/6\rfloor=16,\ |A_2\cap A_5|=10,\ |A_3\cap A_5|=\lfloor100/15\rfloor=6$.
- Triple: $\lfloor100/30\rfloor=3$.
- $|A_2\cup A_3\cup A_5|=50+33+20-16-10-6+3=74$. Probability $=\mathbf{0.74}$.

**Worked example 6 — recovering an intersection.** $P(A)=0.5$, $P(B)=0.4$, $P(A\cup B)=0.7$. Then $P(A\cap B)=0.5+0.4-0.7=0.2$. Since $0.5\times0.4=0.2$, $A$ and $B$ are independent.

## 3. Joint, marginal and conditional probability

- **Joint:** $P(A\cap B)$, written $P(A,B)$ — both happen.
- **Marginal:** $P(A)$ on its own; obtained from a joint table by summing the other variable out: $P(A)=P(A\cap B)+P(A\cap B^c)$.
- **Conditional:** $P(A\mid B)=\dfrac{P(A\cap B)}{P(B)}$, defined when $P(B)>0$. Intuition: *shrink the sample space to $B$ and renormalise.*

Conditioning on $B$ gives a legitimate probability measure: $P(A^c\mid B)=1-P(A\mid B)$, and so on.

**Multiplication (chain) rule**

$$P(A_1\cap\cdots\cap A_n)=P(A_1)\,P(A_2\mid A_1)\,P(A_3\mid A_1\cap A_2)\cdots P(A_n\mid A_1\cap\cdots\cap A_{n-1})$$

**Worked example 7 — a joint table.** 200 students; Pass/Fail by gender.

| | Pass | Fail | Row total (marginal) |
|---|---|---|---|
| Male | 60 | 40 | 100 |
| Female | 70 | 30 | 100 |
| **Column total** | 130 | 70 | 200 |

- Marginal: $P(\text{Pass})=130/200=0.65$.
- Joint: $P(\text{Female}\cap\text{Pass})=70/200=0.35$.
- Conditional: $P(\text{Pass}\mid\text{Female})=70/100=0.70$; $P(\text{Female}\mid\text{Pass})=70/130=0.5385$ — **not the same number**.
- Independent? $P(\text{Pass})=0.65\ne0.70=P(\text{Pass}\mid\text{Female})$ → dependent.

**Worked example 8 — two children.** A family has two children, each equally likely boy/girl independently. $S=\{BB,BG,GB,GG\}$.

- Given "at least one boy": reduced space $\{BB,BG,GB\}$ → $P(\text{both boys})=1/3$.
- Given "the *elder* is a boy": $\{BB,BG\}$ → $1/2$.

The wording of the condition changes the answer; always write the conditioning event explicitly.

## 4. Independence and mutual exclusivity

**Independent events:** $P(A\cap B)=P(A)P(B)$, equivalently $P(A\mid B)=P(A)$ — learning $B$ tells nothing about $A$. If $A,B$ independent then so are $(A,B^c)$, $(A^c,B)$, $(A^c,B^c)$.

**Mutually exclusive events:** $A\cap B=\varnothing$ — they cannot happen together.

| | Independent | Mutually exclusive |
|---|---|---|
| Defining equation | $P(A\cap B)=P(A)P(B)$ | $P(A\cap B)=0$ |
| $P(A\cup B)$ | $P(A)+P(B)-P(A)P(B)$ | $P(A)+P(B)$ |
| $P(A\mid B)$ | $P(A)$ | $0$ |
| Meaning | no information flow | knowing $B$ rules $A$ out |

**Both at once** requires $P(A)P(B)=0$, i.e. one of them has probability 0. **Hence two events of non-zero probability that are disjoint are dependent.** Example: $A$ = "coin shows H", $B$ = "coin shows T": $P(A\cap B)=0\ne\frac14$.

**Independence of several events.** $A_1,\dots,A_n$ are **mutually independent** if $P(\cap_{i\in I}A_i)=\prod_{i\in I}P(A_i)$ for *every* subset $I$ of size $\ge2$ ($2^n-n-1$ conditions). **Pairwise independence** only checks pairs.

**Worked example 9 — pairwise but not mutually independent.** Toss two fair coins. $A$ = first is H, $B$ = second is H, $C$ = both tosses agree.

- $P(A)=P(B)=P(C)=\tfrac12$ ($C=\{HH,TT\}$).
- $A\cap B=\{HH\}$, $A\cap C=\{HH\}$, $B\cap C=\{HH\}$: each has probability $\tfrac14=\tfrac12\cdot\tfrac12$ → **pairwise independent**.
- $A\cap B\cap C=\{HH\}$ has probability $\tfrac14\ne\tfrac18$ → **not mutually independent**.

**Conditional independence.** $A\perp B\mid C$ means $P(A\cap B\mid C)=P(A\mid C)P(B\mid C)$. Neither implies the other direction: independent events can become dependent given $C$ and vice versa. This is the engine of [naive Bayes](../16-machine-learning/classification-methods.md) and [Bayesian networks](../17-artificial-intelligence/probabilistic-reasoning.md).

## 5. Total probability and Bayes' theorem

**Law of total probability.** If $B_1,\dots,B_k$ is a partition of $S$ with each $P(B_i)>0$:

$$P(A)=\sum_{i=1}^k P(A\cap B_i)=\sum_{i=1}^k P(A\mid B_i)\,P(B_i)$$

**Bayes' theorem** — reverses the direction of conditioning:

$$P(B_j\mid A)=\frac{P(A\mid B_j)\,P(B_j)}{P(A)}=\frac{P(A\mid B_j)\,P(B_j)}{\sum_i P(A\mid B_i)\,P(B_i)}$$

Vocabulary: $P(B_j)$ **prior**, $P(A\mid B_j)$ **likelihood**, $P(B_j\mid A)$ **posterior**, $P(A)$ **evidence**.

**Worked example 10 — disease test (table method).** Prevalence $1\%$; test sensitivity $P(+\mid D)=0.95$; false-positive rate $P(+\mid D^c)=0.05$. Find $P(D\mid +)$.

Imagine 10 000 people ("natural frequencies"):

| | Test + | Test − | Total |
|---|---|---|---|
| Disease (1%) | $0.95\times100=95$ | 5 | 100 |
| No disease (99%) | $0.05\times9900=495$ | 9405 | 9900 |
| Total | 590 | 9410 | 10 000 |

$P(D\mid+)=95/590=\mathbf{0.161}$. Formula check: $\dfrac{0.95\times0.01}{0.95\times0.01+0.05\times0.99}=\dfrac{0.0095}{0.0095+0.0495}=\dfrac{0.0095}{0.059}=0.1610$ ✓.

Most positives are false positives because the healthy population is so large (low base rate).

**Worked example 11 — three factories (tree diagram).** Factories A, B, C make 50%, 30%, 20% of items with defect rates 1%, 2%, 3%.

```text
            0.50 ── A ── 0.01 ── defective   0.50*0.01 = 0.005
 start ──── 0.30 ── B ── 0.02 ── defective   0.30*0.02 = 0.006
            0.20 ── C ── 0.03 ── defective   0.20*0.03 = 0.006
```

- Total: $P(\text{def})=0.005+0.006+0.006=0.017$.
- Posterior: $P(A\mid\text{def})=0.005/0.017=\mathbf{0.2941}$; $P(B\mid\text{def})=P(C\mid\text{def})=0.006/0.017=0.3529$. The three posteriors sum to 1. ✓

**Worked example 12 — Monty Hall.** You pick door 1; the host (who knows where the car is) opens a goat door among the other two. By total probability over "car location":

- Car behind door 1 (prob $\tfrac13$): switching loses.
- Car behind 2 or 3 (prob $\tfrac23$): the host's reveal forces the car onto the one remaining door, so switching wins.

$P(\text{win by switching})=\mathbf{2/3}$.

**Worked example 13 — a biased-coin Bayes update.** A bag has a fair coin and a two-headed coin, chosen at random. You toss the chosen coin 3 times and see HHH. $P(\text{two-headed}\mid HHH)$:

- Prior $\tfrac12$ each; likelihoods: two-headed $1$, fair $\tfrac18$.
- $\dfrac{\tfrac12\cdot1}{\tfrac12\cdot1+\tfrac12\cdot\tfrac18}=\dfrac{0.5}{0.5625}=\mathbf{8/9}$.

Each extra head multiplies the odds by 2 (odds went from $1:1$ to $8:1$).

**Worked example 14 — derangement (matching) probability.** 4 letters are put into 4 addressed envelopes at random. $P(\text{no letter in the right envelope})=D_4/4!=9/24=\mathbf{3/8}$, since the number of derangements is $D_n=n!\sum_{k=0}^n(-1)^k/k!$ ($D_3=2,\ D_4=9$). As $n\to\infty$ this tends to $1/e\approx0.368$ (see [Combinatorics](../01-discrete-mathematics/combinatorics.md)).

## Formulas and facts to memorise

| Item | Formula / fact | When to use |
|---|---|---|
| Classical probability | $P(A)=\lvert A\rvert/\lvert S\rvert$ | equally likely outcomes |
| Complement | $P(A^c)=1-P(A)$ | "at least one", "not" |
| Union (2) | $P(A)+P(B)-P(A\cap B)$ | "or" |
| Union (3) | singles − pairs + triple | three overlapping events |
| Conditional | $P(A\mid B)=P(A\cap B)/P(B)$ | restricted information |
| Multiplication | $P(A\cap B)=P(B)P(A\mid B)$ | sequential draws |
| Independence | $P(A\cap B)=P(A)P(B)$ | test or assume |
| Total probability | $\sum_i P(A\mid B_i)P(B_i)$ | multi-source events |
| Bayes | $\dfrac{P(A\mid B_j)P(B_j)}{\sum_iP(A\mid B_i)P(B_i)}$ | reverse a conditional |
| Odds form | posterior odds = prior odds × likelihood ratio | repeated evidence |
| Derangements | $D_n/n!\to1/e$ | "no match" problems |
| Birthday | $1-\prod_{i<n}\frac{365-i}{365}$; $n=23\to0.507$ | collision probability |

## GATE traps

- **Disjoint ≠ independent.** Disjoint with positive probabilities means *strongly dependent*; $P(A\mid B)=0\ne P(A)$.
- **$P(A\mid B)\ne P(B\mid A)$.** Always identify which event is the condition; Bayes converts between them.
- **Pairwise independence does not imply mutual independence** — check the triple product too (Example 9).
- **Unequal outcomes:** do not treat $\{0,1,2\}$ heads as equally likely; count ordered outcomes.
- **Without replacement** changes probabilities at every draw; independence fails (but draws are still *exchangeable*: the probability that the 2nd card is an ace is $4/52$, same as the first).
- **"At least one"** — use $1-P(\text{none})$, not a sum of overlapping cases.
- **Base-rate neglect** in Bayes questions: the prior may dominate the likelihood.
- **Conditioning wording** ("at least one" vs "the elder"/"a specific one") changes the sample space (Example 8).
- $P(A\cup B)=P(A)+P(B)$ holds only for disjoint events; $P(A\cap B)=P(A)P(B)$ only for independent events.

## Connections

- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — all counting here (permutations, combinations, derangements, stars and bars) is developed there; this chapter only applies it.
- [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) — events are sets; axioms are measure-style rules on sets; De Morgan and inclusion–exclusion carry over.
- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — events behave like propositions ($\cup\leftrightarrow\lor$, $\cap\leftrightarrow\land$, $^c\leftrightarrow\lnot$).
- [Random variables and moments](random-variables-and-moments.md) — turns events into numeric quantities; indicator variables use $P(A)$ directly.
- [Classification methods (ML)](../16-machine-learning/classification-methods.md) — naive Bayes is Bayes' theorem with a conditional-independence assumption.
- [Probabilistic reasoning (AI)](../17-artificial-intelligence/probabilistic-reasoning.md) — Bayesian networks factor a joint distribution by the chain rule plus conditional independence.
- [Hashing](../08-algorithms/hashing.md) — the birthday computation is the collision probability of a hash table.
- [Computer networks: data-link layer](../15-computer-networks/data-link-layer.md) — ALOHA success probabilities use independence of stations.

## Practice

**Q1 (MCQ).** Two fair dice are rolled. The probability that the sum is divisible by 3 is
(A) 1/6 (B) 1/4 (C) 1/3 (D) 5/12

<details><summary>Answer</summary>

**Answer:** (C) 1/3  
**Solution:** Sums 3, 6, 9, 12 occur in $2+5+4+1=12$ of the 36 ordered outcomes. $12/36=1/3$.

</details>

**Q2 (NAT).** An urn has 3 red and 2 blue balls. Two balls are drawn without replacement. Find the probability that both have the same colour (answer to 2 decimals).

<details><summary>Answer</summary>

**Answer:** 0.40  
**Solution:** Favourable $=\binom32+\binom22=3+1=4$; total $=\binom52=10$. $4/10=0.40$. Check by sequential: $\frac35\cdot\frac24+\frac25\cdot\frac14=0.3+0.1=0.4$.

</details>

**Q3 (MCQ).** Events $A,B$ with $P(A)=0.5$, $P(B)=0.4$, $P(A\cup B)=0.7$. Which is true?
(A) $A,B$ mutually exclusive (B) $A,B$ independent (C) $P(A\mid B)=0.4$ (D) $P(B\mid A)=0.5$

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** $P(A\cap B)=0.5+0.4-0.7=0.2\ne0$, so not exclusive. $P(A)P(B)=0.2=P(A\cap B)$ → independent. $P(A\mid B)=0.2/0.4=0.5$ (not 0.4); $P(B\mid A)=0.2/0.5=0.4$ (not 0.5).

</details>

**Q4 (MSQ).** Let $A,B$ be events with $P(A)>0$ and $P(B)>0$. Select all statements that are TRUE.
(A) If $A,B$ are mutually exclusive, they are independent.
(B) If $A,B$ are independent, then $A$ and $B^c$ are independent.
(C) If $A,B$ are mutually exclusive, $P(A\cup B)=P(A)+P(B)$.
(D) If $A,B$ are independent, they cannot be mutually exclusive.

<details><summary>Answer</summary>

**Answer:** (B), (C), (D)  
**Solution:** (A) false: disjoint with positive probabilities gives $P(A\cap B)=0\ne P(A)P(B)>0$. (B) true: $P(A\cap B^c)=P(A)-P(A)P(B)=P(A)P(B^c)$. (C) true by axiom 3. (D) true: independence with positive probabilities gives $P(A\cap B)>0$.

</details>

**Q5 (NAT).** One of two coins is chosen at random: a fair coin or a two-headed coin. It is tossed three times and shows three heads. Probability that it is the two-headed coin (as a fraction $p/q$; give $p+q$).

<details><summary>Answer</summary>

**Answer:** 17 (probability $8/9$)  
**Solution:** $\dfrac{\frac12\cdot1}{\frac12\cdot1+\frac12\cdot\frac18}=\dfrac{1}{1+\frac18}=\dfrac89$. $p+q=8+9=17$.

</details>

**Q6 (MCQ).** A test detects a condition with sensitivity 90% and false-positive rate 10%. The prevalence is 5%. If a person tests positive, the probability of having the condition is closest to
(A) 0.90 (B) 0.32 (C) 0.05 (D) 0.50

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** $\dfrac{0.9\times0.05}{0.9\times0.05+0.1\times0.95}=\dfrac{0.045}{0.045+0.095}=\dfrac{0.045}{0.14}=0.321$.

</details>

**Q7 (NAT).** Four letters are placed at random into four addressed envelopes, one per envelope. Probability that no letter is in the correct envelope (answer as decimal).

<details><summary>Answer</summary>

**Answer:** 0.375  
**Solution:** $D_4=24(1-1+\frac12-\frac16+\frac1{24})=24\cdot\frac{9}{24}=9$. $9/24=0.375$.

</details>

**Q8 (MCQ).** Three events $A,B,C$ satisfy $P(A\cap B)=P(A)P(B)$, $P(B\cap C)=P(B)P(C)$, $P(A\cap C)=P(A)P(C)$. Which conclusion is guaranteed?
(A) $A,B,C$ are mutually independent
(B) $P(A\cap B\cap C)=P(A)P(B)P(C)$
(C) $A,B,C$ are pairwise independent
(D) $A,B,C$ are mutually exclusive

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** The three given equations are exactly the definition of pairwise independence. The triple-product condition is extra and may fail (Example 9: two coins, $A$ = first H, $B$ = second H, $C$ = same; triple intersection has probability $1/4\ne1/8$). So (A),(B) are not guaranteed.

</details>

---
[Probability & Statistics README](README.md) · [Cheat sheet](CHEATSHEET.md) · [Checkpoint](CHECKPOINT.md)
