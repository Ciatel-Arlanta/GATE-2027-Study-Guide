# Combinatorics: Systematic Counting

> **Paper:** CS · **Priority:** P0 · **Plan topics:** counting, permutations, combinations, pigeonhole principle, inclusion–exclusion
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Probability basics](../02-probability-statistics/probability-basics.md)
> **PYQ weightage:** Medium · 12 Qs since 2015 (6 since 2021) · DA: 0 · Trend: steady · Most asked: Permutations & combinations, Balls in bins, Inclusion–exclusion & derangements

## Quick glance

- **Exam focus:** Multi-way counting, balls-in-bins distributions (Stars and Bars, Stirling numbers, integer partitions), Principle of Inclusion–Exclusion (PIE) with derangements, Pigeonhole principle bounds, and Catalan number applications. Typically tested via 1-mark and 2-mark NATs and MCQs.
- **Product vs Sum rules:** Disjoint cases **add** ($|A \cup B| = |A| + |B|$ for $A \cap B = \emptyset$). Sequential independent choices **multiply** ($|A \times B| = |A| \cdot |B|$).
- **Permutations vs Combinations:** Order matters $\implies P(n, r) = \frac{n!}{(n-r)!}$. Order does not matter $\implies \binom{n}{r} = \frac{n!}{r!(n-r)!}$.
- **Multiset arrangements:** Arranging $n$ objects with duplicates: $\frac{n!}{n_1! n_2! \dots n_k!}$.
- **Circular arrangements:** $n$ distinct objects around a circle $\implies (n - 1)!$. If reflections/reversals are identical (necklaces/keychains) $\implies \frac{(n - 1)!}{2}$.
- **Identical balls into distinct bins (Stars & Bars):** Non-negative integer solutions to $\sum_{i=1}^k x_i = n \implies \binom{n + k - 1}{k - 1}$. Strictly positive ($x_i \ge 1$) $\implies \binom{n - 1}{k - 1}$.
- **Derangements ($D_n$):** No element in its original position. $D_1 = 0, D_2 = 1, D_3 = 2, D_4 = 9, D_5 = 44, D_6 = 265$. Recurrence: $D_n = (n - 1)(D_{n-1} + D_{n-2})$.
- **Pigeonhole Principle (PHP):** Placing $n$ items into $k$ boxes forces at least one box to contain $\ge \lceil n/k \rceil$ items. To guarantee at least $m$ items in some box, you need $k(m - 1) + 1$ items.
- **Catalan numbers:** $C_n = \frac{1}{n+1}\binom{2n}{n}$. Sequence: $1, 1, 2, 5, 14, 42, 132$. Counts distinct BSTs with $n$ keys, balanced parentheses of length $2n$, and Dyck paths.

## 1. Permutations, combinations, and basic counting principles [PYQ: Medium]

### Basic principles
1. **Sum Rule (Addition Principle):** If a task can be done in either of two mutually exclusive ways (one in $n_1$ ways, the other in $n_2$ ways), the total ways are $n_1 + n_2$.
2. **Product Rule (Multiplication Principle):** If a procedure consists of sequential subtasks $T_1, T_2, \dots, T_k$ where $T_i$ can be done in $n_i$ ways regardless of previous choices, the procedure can be performed in $n_1 \times n_2 \times \dots \times n_k$ ways.

### Permutations and combinations
- **Permutation without repetition:** Selecting and ordering $r$ items from $n$ distinct items:
  $$P(n, r) = \frac{n!}{(n - r)!}$$
- **Combination without repetition:** Selecting an unordered subset of $r$ items from $n$ distinct items:
  $$\binom{n}{r} = C(n, r) = \frac{n!}{r!(n - r)!}$$
  - Symmetry: $\binom{n}{r} = \binom{n}{n - r}$.
  - Pascal's identity: $\binom{n}{r} = \binom{n - 1}{r - 1} + \binom{n - 1}{r}$.
  - Vandermonde's identity: $\sum_{k=0}^r \binom{m}{k}\binom{n}{r - k} = \binom{m + n}{r}$.
- **Permutations with repetition:** Permutations of $n$ objects where item $i$ appears $n_i$ times ($\sum n_i = n$):
  $$\frac{n!}{n_1! n_2! \dots n_k!}$$
- **Circular permutations:**
  - Distinct orientations: $(n - 1)!$.
  - Flip symmetry (clockwise and counterclockwise indistinguishable): $\frac{(n - 1)!}{2}$.

**Worked example 1 (Constrained letter arrangements).**
Find the number of distinct permutations of the letters in `SUCCESS` such that:
(a) All three `S`'s appear together.
(b) No two `S`'s appear consecutively.

*Step-by-step solution:*
The word `SUCCESS` has 7 letters: 3 `S`, 2 `C`, 1 `U`, 1 `E`.
1. **(a) All three `S`'s together:**
   - Glue the three `S`'s into a single composite entity `[SSS]`.
   - The entities to arrange are: `[SSS]`, `U`, `C`, `C`, `E` (total 5 entities, with 2 `C`'s).
   - Arrangements $= \frac{5!}{2!} = \frac{120}{2} = 60$.
   - Since the three `S`'s are identical, there is only 1 internal arrangement. Total $= 60$.
2. **(b) No two `S`'s appear consecutively (Gap Method):**
   - First arrange the non-`S` letters: `U`, `C`, `C`, `E` (4 letters with two `C`'s).
     $$\text{Ways to arrange non-}S = \frac{4!}{2!} = \frac{24}{2} = 12$$
   - Place the non-`S` letters, creating $4 + 1 = 5$ possible gaps (including ends):
     $$\_ \text{ C } \_ \text{ U } \_ \text{ C } \_ \text{ E } \_$$
   - To ensure no two `S`'s are adjacent, each gap can hold at most one `S`. We choose 3 of these 5 gaps:
     $$\text{Ways to choose gaps} = \binom{5}{3} = 10$$
   - By the product rule: Total ways $= 12 \times 10 = 120$.

## 2. Balls in bins and distributions (The Twelvefold Way) [PYQ: Medium]

Distributing $n$ balls into $k$ bins forms the backbone of GATE counting questions. The count depends on whether balls and bins are distinct or identical:

| Balls ($n$) | Bins ($k$) | Any number of balls per bin | At least 1 ball per bin (No empty bins) | At most 1 ball per bin (Capacity $\le 1$) |
|---|---|---|---|---|
| **Distinct** | **Distinct** | $k^n$ | $k! \cdot S(n, k)$ (Surjections) | $P(k, n) = \frac{k!}{(k - n)!}$ ($n \le k$) |
| **Identical** | **Distinct** | $\binom{n + k - 1}{k - 1}$ (Stars & Bars) | $\binom{n - 1}{k - 1}$ | $\binom{k}{n}$ ($n \le k$) |
| **Distinct** | **Identical** | $\sum_{j=1}^k S(n, j)$ | $S(n, k)$ (Stirling 2nd kind) | $1$ ($n \le k$), else $0$ |
| **Identical** | **Identical** | $p_{\le k}(n)$ (Partitions of $n$ into $\le k$ parts) | $p_k(n)$ (Partitions into exactly $k$ parts) | $1$ ($n \le k$), else $0$ |

### Stars and Bars theorem
The number of non-negative integer solutions to $x_1 + x_2 + \dots + x_k = n$ ($x_i \ge 0$) equals:
$$\binom{n + k - 1}{k - 1} = \binom{n + k - 1}{n}$$
- If each variable has a lower bound $x_i \ge c_i$, set $y_i = x_i - c_i \ge 0$. The equation becomes $\sum y_i = n - \sum c_i$, giving $\binom{(n - \sum c_i) + k - 1}{k - 1}$.
- In particular, for strictly positive solutions ($x_i \ge 1$ for all $i$), the count is $\binom{n - 1}{k - 1}$.

**Worked example 2 (Balls in bins distributions — GATE 2022).**
(a) Find the number of arrangements of six identical balls in three identical bins.
(b) Find the number of non-negative integer solutions to $x_1 + x_2 + x_3 + x_4 = 15$ such that $x_1 \ge 2, x_2 \ge 1, x_3 \ge 0, x_4 \ge 3$.

*Step-by-step solution:*
1. **(a) 6 identical balls into 3 identical bins:**
   - Both balls and bins are identical, so an arrangement is an unordered partition of integer 6 into at most 3 parts ($x_1 \ge x_2 \ge x_3 \ge 0$ with $x_1 + x_2 + x_3 = 6$).
   - We enumerate all partitions systematically by largest part:
     - 1 non-empty bin: $(6, 0, 0)$ $\implies 1$ way.
     - 2 non-empty bins: $(5, 1, 0), (4, 2, 0), (3, 3, 0)$ $\implies 3$ ways.
     - 3 non-empty bins: $(4, 1, 1), (3, 2, 1), (2, 2, 2)$ $\implies 3$ ways.
   - Total arrangements $= 1 + 3 + 3 = 7$.
2. **(b) Bounded integer equation:**
   - Equation: $x_1 + x_2 + x_3 + x_4 = 15$ with $x_1 \ge 2, x_2 \ge 1, x_3 \ge 0, x_4 \ge 3$.
   - Substitute: $y_1 = x_1 - 2 \ge 0$, $y_2 = x_2 - 1 \ge 0$, $y_3 = x_3 \ge 0$, $y_4 = x_4 - 3 \ge 0$.
   - New sum: $y_1 + y_2 + y_3 + y_4 = 15 - (2 + 1 + 0 + 3) = 15 - 6 = 9$.
   - Apply Stars and Bars with $n' = 9$ and $k = 4$:
     $$\binom{n' + k - 1}{k - 1} = \binom{9 + 4 - 1}{4 - 1} = \binom{12}{3} = \frac{12 \times 11 \times 10}{6} = 220$$

## 3. Inclusion–Exclusion and derangements [PYQ: Medium]

### Principle of Inclusion–Exclusion (PIE)
For finite sets $A_1, A_2, \dots, A_n$ inside a universal set $U$:
$$\left| \bigcup_{i=1}^n A_i \right| = \sum_{i} |A_i| - \sum_{i < j} |A_i \cap A_j| + \sum_{i < j < k} |A_i \cap A_j \cap A_k| - \dots + (-1)^{n-1} \left| \bigcap_{i=1}^n A_i \right|$$
Complement counting (elements satisfying none of the properties):
$$\left| \bigcap_{i=1}^n \overline{A_i} \right| = |U| - \left| \bigcup_{i=1}^n A_i \right|$$

### Derangements
A **derangement** of $\{1, 2, \dots, n\}$ is a permutation $\sigma$ such that $\sigma(i) \neq i$ for all $1 \le i \le n$ (no fixed points).
- **Explicit formula:**
  $$D_n = n! \sum_{k=0}^n \frac{(-1)^k}{k!} = n! \left( 1 - \frac{1}{1!} + \frac{1}{2!} - \frac{1}{3!} + \dots + \frac{(-1)^n}{n!} \right) = \left\lfloor \frac{n!}{e} + \frac{1}{2} \right\rfloor$$
- **Recurrence relation:**
  $$D_n = (n - 1)(D_{n-1} + D_{n-2}) \quad \text{for } n \ge 2, \quad D_1 = 0, D_2 = 1$$
- **Values for small $n$:** $D_1 = 0, D_2 = 1, D_3 = 2, D_4 = 9, D_5 = 44, D_6 = 265$.
- **Rencontres numbers (exactly $k$ fixed points):** $\binom{n}{k} D_{n-k}$.

**Worked example 3 (Derangements with identical letters — GATE 2020).**
Find the number of permutations of the characters in `LILAC` such that no character appears in its original position, where the two `L`'s are indistinguishable.

*Step-by-step solution:*
Original positions: Pos 1 = `L`, Pos 2 = `I`, Pos 3 = `L`, Pos 4 = `A`, Pos 5 = `C`.
Total distinct permutations of `LILAC` $= \frac{5!}{2!} = 60$.
Let $A_i$ be the set of permutations where the character at position $i$ matches the original character:
- $A_1$: Pos 1 has `L`. One `L` is placed at pos 1; remaining letters `{L, I, A, C}` are all distinct $\implies |A_1| = 4! = 24$.
- $A_3$: Pos 3 has `L`. Similarly, $|A_3| = 4! = 24$.
- $A_2$: Pos 2 has `I`. Remaining letters `{L, L, A, C}` have two `L`'s $\implies |A_2| = \frac{4!}{2!} = 12$.
- $A_4$: Pos 4 has `A` $\implies |A_4| = \frac{4!}{2!} = 12$; $A_5$: Pos 5 has `C` $\implies |A_5| = \frac{4!}{2!} = 12$.
Sum of singles: $\sum |A_i| = 24 + 24 + 12 + 12 + 12 = 84$.

Pairwise intersections:
- $|A_1 \cap A_3|$: Both `L`'s fixed at Pos 1 and Pos 3. Remaining `{I, A, C}` $\implies 3! = 6$.
- One `L`-position and one other position ($2 \times 3 = 6$ pairs): e.g., Pos 1 has `L`, Pos 2 has `I`. Remaining `{L, A, C}` are distinct $\implies 3! = 6$. Sum $= 6 \times 6 = 36$.
- Two non-`L` positions ($\binom{3}{2} = 3$ pairs): e.g., Pos 2 has `I`, Pos 4 has `A`. Remaining `{L, L, C}` $\implies \frac{3!}{2!} = 3$. Sum $= 3 \times 3 = 9$.
Sum of pairs: $\sum |A_i \cap A_j| = 6 + 36 + 9 = 51$.

Three-way intersections:
- Both `L`'s and one other ($3$ triples): Remaining 2 distinct letters $\implies 2! = 2$. Sum $= 3 \times 2 = 6$.
- One `L` and two others ($2 \times 3 = 6$ triples): Remaining 2 distinct letters $\implies 2! = 2$. Sum $= 6 \times 2 = 12$.
- All three non-`L` positions ($A_2 \cap A_4 \cap A_5$): Remaining `{L, L}` $\implies \frac{2!}{2!} = 1$.
Sum of triples: $6 + 12 + 1 = 19$.

Four-way intersections:
- Quads with both `L`'s ($3$ quads): Remaining 1 letter $\implies 1! = 1$. Sum $= 3$.
- Quads with one `L` ($2$ quads): Remaining 1 letter $\implies 1! = 1$. Sum $= 2$.
Sum of quadruples: $3 + 2 = 5$.

Five-way intersection: $|A_1 \cap \dots \cap A_5| = 1$ (the original word `LILAC`).
By Principle of Inclusion–Exclusion:
$$N = 60 - 84 + 51 - 19 + 5 - 1 = 12$$

## 4. Pigeonhole Principle and generalized bounds [PYQ: Low]

### Basic and strong forms
- **Basic PHP:** If $n$ pigeons are placed into $k$ pigeonholes, and $n > k$, at least one hole contains at least 2 pigeons.
- **Generalized PHP:** If $n$ objects are placed into $k$ boxes, at least one box contains at least:
  $$\left\lceil \frac{n}{k} \right\rceil \text{ objects}$$
  Similarly, at least one box contains at most $\lfloor n/k \rfloor$ objects.
- **Guaranteeing a threshold:** To guarantee that at least one of the $k$ boxes contains at least $m$ objects, the minimum number of objects required is:
  $$n_{\min} = k(m - 1) + 1$$

**Worked example 4 (PHP on subsets and remainders).**
Show that among any 5 integers selected from $\{1, 2, 3, 4, 5, 6, 7, 8\}$, there must exist at least one pair whose sum is 9.

*Step-by-step solution:*
1. Group the 8 integers into 4 disjoint pairs that each sum to 9:
   $$\text{Hole 1: } \{1, 8\}, \quad \text{Hole 2: } \{2, 7\}, \quad \text{Hole 3: } \{3, 6\}, \quad \text{Hole 4: } \{4, 5\}$$
2. Here there are $k = 4$ pigeonholes.
3. We select $n = 5$ numbers (pigeons).
4. By PHP, since $5 > 4$, at least two numbers must come from the same pigeonhole.
5. Since each pigeonhole contains exactly one pair that sums to 9, those two selected numbers must sum to 9.

## 5. Catalan numbers and lattice paths [PYQ: Medium]

The $n$-th **Catalan number** is defined by:
$$C_n = \frac{1}{n + 1} \binom{2n}{n} = \frac{(2n)!}{(n + 1)! \, n!}$$
Recurrence relation:
$$C_0 = 1, \quad C_n = \sum_{i=0}^{n-1} C_i C_{n - 1 - i} \quad \text{for } n \ge 1$$
First values: $C_0 = 1, C_1 = 1, C_2 = 2, C_3 = 5, C_4 = 14, C_5 = 42, C_6 = 132$.

### Standard CS interpretations of $C_n$
1. **Binary Search Trees (BSTs):** Number of distinct BSTs that can be formed using $n$ distinct keys is $C_n$. (With unlabeled nodes: $C_n$ distinct binary tree structures; with $n$ distinct labeled nodes: $n! \cdot C_n$).
2. **Balanced Parentheses:** Number of valid strings consisting of $n$ pairs of `(` and `)` is $C_n$.
3. **Dyck Paths:** Number of paths from $(0, 0)$ to $(n, n)$ in an $n \times n$ grid moving only Right and Up that never go above the diagonal $y = x$.
4. **Matrix Chain Multiplication:** Number of ways to parenthesize the product of $n + 1$ matrices is $C_n$.
5. **Full Binary Trees:** Number of full binary trees with $n + 1$ leaves (and $n$ internal nodes) is $C_n$.
6. **Stack Permutations:** Number of permutations of $\{1, 2, \dots, n\}$ that can be obtained using a single stack is $C_n$.

**Worked example 5 (BST construction and Catalan counting).**
How many distinct binary search trees can be constructed with keys $\{10, 20, 30, 40\}$? Trace the count by root selection.

*Step-by-step solution:*
Here $n = 4$. By the Catalan formula:
$$C_4 = \frac{1}{4 + 1}\binom{8}{4} = \frac{1}{5} \times \frac{8 \times 7 \times 6 \times 5}{24} = \frac{70}{5} = 14$$
Tracing by choice of root key:
- If 10 is root: Left subtree has 0 keys ($C_0 = 1$ BST), right has 3 keys ($C_3 = 5$ BSTs) $\implies 1 \times 5 = 5$.
- If 20 is root: Left has 1 key ($C_1 = 1$), right has 2 keys ($C_2 = 2$) $\implies 1 \times 2 = 2$.
- If 30 is root: Left has 2 keys ($C_2 = 2$), right has 1 key ($C_1 = 1$) $\implies 2 \times 1 = 2$.
- If 40 is root: Left has 3 keys ($C_3 = 5$), right has 0 keys ($C_0 = 1$) $\implies 5 \times 1 = 5$.
Total BSTs $= 5 + 2 + 2 + 5 = 14 = C_4$.

## 6. Modular arithmetic and number theory [PYQ: Low]

### Definitions and congruence
For integers $a, b$ and positive integer $m$:
$$a \equiv b \pmod m \iff m \mid (a - b)$$
- Congruence preserves addition, subtraction, and multiplication:
  $$\text{If } a_1 \equiv b_1 \pmod m \text{ and } a_2 \equiv b_2 \pmod m \implies a_1 \pm a_2 \equiv b_1 \pm b_2 \pmod m, \quad a_1 a_2 \equiv b_1 b_2 \pmod m$$
- **Modular inverse:** $a^{-1} \pmod m$ exists if and only if $\gcd(a, m) = 1$.

### Key theorems
1. **Fermat's Little Theorem (FLT):** If $p$ is prime and $\gcd(a, p) = 1$:
   $$a^{p - 1} \equiv 1 \pmod p \implies a^p \equiv a \pmod p$$
2. **Euler's Totient Function $\phi(n)$:** Count of integers $1 \le k \le n$ coprime to $n$:
   $$\phi(n) = n \prod_{p \mid n} \left(1 - \frac{1}{p}\right)$$
   For prime $p$, $\phi(p) = p - 1$. For prime power $p^k$, $\phi(p^k) = p^k - p^{k-1}$.
3. **Euler's Theorem:** If $\gcd(a, m) = 1$:
   $$a^{\phi(m)} \equiv 1 \pmod m$$

**Worked example 6 (Modular exponentiation via Fermat's Little Theorem).**
Compute the remainder when $7^{2026}$ is divided by 13.

*Step-by-step solution:*
1. The modulus is $p = 13$, which is prime, and $\gcd(7, 13) = 1$.
2. By Fermat's Little Theorem, $7^{13 - 1} = 7^{12} \equiv 1 \pmod{13}$.
3. Divide the exponent 2026 by 12:
   $$2026 = 12 \times 168 + 10$$
4. Therefore:
   $$7^{2026} = (7^{12})^{168} \times 7^{10} \equiv 1^{168} \times 7^{10} \equiv 7^{10} \pmod{13}$$
5. Compute $7^{10} \pmod{13}$ using repeated squaring:
   - $7^2 = 49 \equiv -3 \pmod{13}$ (since $49 = 4 \times 13 - 3$).
   - $7^4 \equiv (-3)^2 = 9 \equiv -4 \pmod{13}$.
   - $7^8 \equiv (-4)^2 = 16 \equiv 3 \pmod{13}$.
   - $7^{10} = 7^8 \times 7^2 \equiv 3 \times (-3) = -9 \equiv 4 \pmod{13}$.
6. The remainder is **4**.

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Permutations without repetition | $P(n, r) = \frac{n!}{(n-r)!}$ | Ordered selections |
| Combinations without repetition | $\binom{n}{r} = \frac{n!}{r!(n-r)!}$ | Unordered selections |
| Permutations with repetition | $\frac{n!}{n_1! n_2! \dots n_k!}$ | Arrangements with identical items |
| Circular permutations | $(n - 1)!$ (or $\frac{(n-1)!}{2}$ for flips) | Arrangements in a closed ring |
| Stars and Bars ($x_i \ge 0$) | $\binom{n + k - 1}{k - 1}$ | Identical balls in distinct bins |
| Stars and Bars ($x_i \ge 1$) | $\binom{n - 1}{k - 1}$ | Non-empty distinct bins |
| Derangements $D_n$ | $(n - 1)(D_{n-1} + D_{n-2}) \approx \lfloor n!/e + 0.5 \rfloor$ | Permutations with no fixed points |
| Pigeonhole threshold | $k(m - 1) + 1$ | Minimum objects to guarantee $\ge m$ in some bin |
| Catalan number $C_n$ | $\frac{1}{n+1}\binom{2n}{n}$ | Unlabeled binary trees, BSTs, balanced brackets |
| Fermat's Little Theorem | $a^{p-1} \equiv 1 \pmod p$ ($\gcd(a, p) = 1$) | Large exponentiation modulo prime $p$ |
| Euler's Totient $\phi(n)$ | $n \prod_{p \mid n} (1 - 1/p)$ | Exponent reduction modulo composite $n$ |

## GATE traps

- **Non-negative vs strictly positive integer solutions:** $\binom{n + k - 1}{k - 1}$ applies to $x_i \ge 0$. If $x_i \ge 1$, use $\binom{n - 1}{k - 1}$. For variable bounds, always substitute $y_i = x_i - c_i \ge 0$ before applying the formula.
- **Identical balls vs distinct balls:** If both balls and bins are identical, Stars & Bars does **not** apply; count integer partitions manually or by table.
- **Circular arrangements:** For distinct directions (e.g. people at a table), the answer is $(n-1)!$. For symmetric directions (e.g. beads on a necklace), divide by 2: $\frac{(n-1)!}{2}$.
- **Catalan with labeled vs unlabeled nodes:** Unlabeled binary trees with $n$ nodes $= C_n$. Binary search trees with $n$ distinct keys $= C_n$. Arbitrary labeled binary trees $= n! \cdot C_n$.
- **Modulo division is invalid:** You cannot divide $a \equiv b \pmod m$ by $c$ unless $\gcd(c, m) = 1$; you must multiply by $c^{-1} \pmod m$.

## Connections

- [Sets, relations, functions](sets-relations-functions.md) — onto function counts are surjection distributions ($k! \cdot S(n, k)$); PIE extends set cardinality unions.
- [Probability basics](../02-probability-statistics/probability-basics.md) — classical probability $P(E) = |E| / |S|$ relies directly on combinatorics.
- [Discrete distributions](../02-probability-statistics/discrete-distributions.md) — binomial distribution probabilities use $\binom{n}{k}$.
- [Trees and BST](../07-data-structures/trees-and-bst.md) — Catalan numbers count distinct BST structures.
- [Dynamic programming](../08-algorithms/dynamic-programming.md) — Matrix Chain Multiplication parenthesization count equals $C_n$.

## Practice

**Q1 (NAT).** The number of non-negative integer solutions to $x_1 + x_2 + x_3 + x_4 = 12$ is ________.
<details><summary>Answer</summary>
**Answer:** 455 · **Solution:**
Using the Stars and Bars theorem with $n = 12$ and $k = 4$:
$$\binom{n + k - 1}{k - 1} = \binom{12 + 4 - 1}{4 - 1} = \binom{15}{3} = \frac{15 \times 14 \times 13}{3 \times 2 \times 1} = 455$$
</details>

**Q2 (MCQ).** In how many ways can 6 people be seated around a circular table such that two particular people are never seated next to each other?
(A) 48 (B) 72 (C) 96 (D) 120
<details><summary>Answer</summary>
**Answer:** (B) · **Solution:**
Total circular permutations of 6 people $= (6 - 1)! = 5! = 120$.
Number of seating arrangements where the two particular people sit together:
Glue them as a single block (which has $2!$ internal orders).
We now arrange 5 entities around the circular table: $(5 - 1)! = 4! = 24$.
Ways together $= 24 \times 2! = 48$.
Ways never together $= 120 - 48 = 72$.
</details>

**Q3 (NAT).** In how many ways can 6 identical coins be distributed among 3 distinct children such that every child receives at least one coin? ________.
<details><summary>Answer</summary>
**Answer:** 10 · **Solution:**
This is the number of strictly positive integer solutions ($x_i \ge 1$) to $x_1 + x_2 + x_3 = 6$.
Using Stars and Bars for positive solutions:
$$\binom{n - 1}{k - 1} = \binom{6 - 1}{3 - 1} = \binom{5}{2} = \frac{5 \times 4}{2} = 10$$
</details>

**Q4 (NAT).** A secret code is a permutation of the digits $\{1, 2, 3, 4, 5\}$. The number of codes in which digit 1 is not in position 1, digit 2 is not in position 2, and digit 3 is not in position 3 (with no restrictions on positions 4 and 5) is ________.
<details><summary>Answer</summary>
**Answer:** 64 · **Solution:**
Total permutations of 5 distinct digits $= 5! = 120$.
Let $A_i$ be the set of permutations where digit $i$ is in position $i$ (for $i \in \{1, 2, 3\}$).
- Single conditions: $|A_i| = 4! = 24$. There are $\binom{3}{1} = 3$ such sets $\implies \sum |A_i| = 3 \times 24 = 72$.
- Pairwise conditions: $|A_i \cap A_j| = 3! = 6$. There are $\binom{3}{2} = 3$ pairs $\implies \sum |A_i \cap A_j| = 3 \times 6 = 18$.
- Three-way condition: $|A_1 \cap A_2 \cap A_3| = 2! = 2$.
By the Principle of Inclusion–Exclusion:
$$N = 120 - 72 + 18 - 2 = 64$$
</details>

**Q5 (MSQ).** Which of the following combinatorial structures on $n$ elements are enumerated by the $n$-th Catalan number $C_n = \frac{1}{n+1}\binom{2n}{n}$?
(A) Number of distinct binary search trees that can be constructed using $n$ distinct keys.  
(B) Number of distinct binary trees that can be formed using $n$ distinct labeled nodes.  
(C) Number of valid sequences of $n$ opening and $n$ closing parentheses.  
(D) Number of full binary trees with $n + 1$ leaves.  
<details><summary>Answer</summary>
**Answer:** (A), (C), (D) · **Solution:**
- (A) is correct: The number of distinct BSTs with $n$ distinct keys is $C_n$.
- (B) is incorrect: With $n$ labeled nodes, the number of binary trees is $n! \cdot C_n$ because keys can be assigned to the unlabeled tree structure in $n!$ ways.
- (C) is correct: Standard Dyck path / balanced parenthesis interpretation of $C_n$.
- (D) is correct: A full binary tree with $n+1$ leaves has $n$ internal nodes; the number of such trees is $C_n$.
</details>

**Q6 (NAT).** The number of integer solutions to $x_1 + x_2 + x_3 = 15$ with $0 \le x_i \le 6$ for each $i \in \{1, 2, 3\}$ is ________.
<details><summary>Answer</summary>
**Answer:** 10 · **Solution:**
Let $U$ be the set of non-negative integer solutions to $x_1 + x_2 + x_3 = 15$:
$$|U| = \binom{15 + 3 - 1}{3 - 1} = \binom{17}{2} = \frac{17 \times 16}{2} = 136$$
Let $A_i$ be the condition that $x_i \ge 7$. Setting $y_i = x_i - 7 \ge 0$, the equation becomes $y_i + x_j + x_k = 15 - 7 = 8$:
$$|A_i| = \binom{8 + 3 - 1}{2} = \binom{10}{2} = 45 \implies \sum_{i=1}^3 |A_i| = 3 \times 45 = 135$$
For two variables $\ge 7$, set $y_i = x_i - 7, y_j = x_j - 7 \ge 0 \implies y_i + y_j + x_k = 15 - 14 = 1$:
$$|A_i \cap A_j| = \binom{1 + 3 - 1}{2} = \binom{3}{2} = 3 \implies \sum |A_i \cap A_j| = \binom{3}{2} \times 3 = 9$$
For three variables $\ge 7$, $x_1 + x_2 + x_3 \ge 21 > 15 \implies 0$.
By Inclusion–Exclusion:
$$\text{Valid solutions} = 136 - 135 + 9 - 0 = 10$$
</details>

**Q7 (NAT).** The remainder when $3^{1000}$ is divided by 35 is ________.
<details><summary>Answer</summary>
**Answer:** 11 · **Solution:**
The modulus is $m = 35 = 5 \times 7$. Note that $\gcd(3, 35) = 1$.
Euler's totient function:
$$\phi(35) = 35 \times \left(1 - \frac{1}{5}\right) \times \left(1 - \frac{1}{7}\right) = 35 \times \frac{4}{5} \times \frac{6}{7} = 24$$
By Euler's Theorem, $3^{24} \equiv 1 \pmod{35}$.
Divide the exponent by 24: $1000 = 24 \times 41 + 16$.
$$3^{1000} \equiv (3^{24})^{41} \times 3^{16} \equiv 3^{16} \pmod{35}$$
Compute $3^{16} \pmod{35}$ by successive squaring:
- $3^2 = 9$
- $3^4 = 81 \equiv 11 \pmod{35}$
- $3^8 \equiv 11^2 = 121 \equiv 16 \pmod{35}$ (since $121 = 3 \times 35 + 16$)
- $3^{16} \equiv 16^2 = 256 \equiv 11 \pmod{35}$ (since $256 = 7 \times 35 + 11$)
The remainder is 11.
</details>
