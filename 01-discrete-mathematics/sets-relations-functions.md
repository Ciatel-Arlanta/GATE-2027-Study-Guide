# Sets, Relations, and Functions

> **Paper:** CS · **Priority:** P0 · **Plan topics:** sets, relations, equivalence relations, partial orders, functions
> **Prerequisites:** [Propositional logic](propositional-logic.md) · **Leads to:** [Posets and lattices](posets-and-lattices.md) · [Graph theory](graph-theory.md)
> **PYQ weightage:** High · 25 Qs since 2015 (11 since 2021) · DA: 1 · Trend: steady · Most asked: Relations, Functions, Sets & Countability

## Quick glance

- **Exam focus:** Counting relations (reflexive, symmetric, transitive, equivalence) and functions (injective, surjective, bijective), analyzing relation closures/properties, and classifying set countability. Typically tested via 1-mark and 2-mark NATs and MSQs.
- **Power set:** $|\mathcal{P}(A)| = 2^n$ for $|A| = n$. The set of all subsets of a countable set $\mathbb{N}$ is **uncountable** ($2^{\aleph_0} = \mathfrak{c}$).
- **Countable vs Uncountable:** Finite sets, $\mathbb{N}, \mathbb{Z}, \mathbb{Q}$, $\Sigma^*$, and the set of all Turing machines / C programs are **countably infinite**. $\mathbb{R}$, $(0, 1)$, $\mathcal{P}(\mathbb{N})$, and the set of all languages over $\Sigma$ are **uncountable**.
- **Relation on $A$:** A subset $R \subseteq A \times A$. On $|A| = n$, there are $2^{n^2}$ total relations.
- **Reflexive:** $\forall x, (x, x) \in R$ ($2^{n(n-1)}$). **Symmetric:** $(x, y) \in R \implies (y, x) \in R$ ($2^{n(n+1)/2}$).
- **Antisymmetric:** $(x, y) \in R \land (y, x) \in R \implies x = y$ ($2^n \cdot 3^{n(n-1)/2}$). **Not symmetric does not mean antisymmetric.**
- **Equivalence relation:** Reflexive + Symmetric + Transitive. Equivalence classes partition $A$. The number of equivalence relations on $n$ elements is the **Bell number** $B(n)$ ($B(1)=1, B(2)=2, B(3)=5, B(4)=15, B(5)=52$).
- **Partial order:** Reflexive + Antisymmetric + Transitive.
- **Functions $f: A \to B$ ($|A|=m, |B|=n$):** Total $= n^m$. Injective (one-to-one) $= \frac{n!}{(n-m)!}$ (requires $m \le n$). Surjective (onto) $= n! \cdot S(m, n) = \sum_{k=0}^n (-1)^k \binom{n}{k} (n-k)^m$ (requires $m \ge n$). Bijective $= n!$ (requires $m = n$).

## 1. Set algebra, power sets, and countability [PYQ: High]

### Intuition and definitions
A **set** is an unordered collection of distinct elements. For any sets $A, B \subseteq U$:
- **Subset:** $A \subseteq B \iff (\forall x \in A \implies x \in B)$. Equality $A = B \iff A \subseteq B \land B \subseteq A$.
- **Operations:** Union $A \cup B$, Intersection $A \cap B$, Difference $A \setminus B = A \cap B^c$, Symmetric difference $A \oplus B = (A \setminus B) \cup (B \setminus A)$.
- **Power set:** $\mathcal{P}(A) = \{S \mid S \subseteq A\}$. If $|A| = n$, then $|\mathcal{P}(A)| = 2^n$.

### Countability and Cantor's diagonalization
A set $S$ is **countable** if it is finite or has the same cardinality as the natural numbers $\mathbb{N}$ (i.e., there exists a bijection $f: \mathbb{N} \to S$). Otherwise, it is **uncountable**.

| Set | Cardinality | Reason |
|---|---|---|
| Natural numbers $\mathbb{N}$, Integers $\mathbb{Z}$ | Countable ($\aleph_0$) | Identity / alternating enumeration: $0, 1, -1, 2, -2, \dots$ |
| Rational numbers $\mathbb{Q}$ | Countable ($\aleph_0$) | Diagonal traversal of grid $(p, q) \in \mathbb{Z} \times \mathbb{N}$ |
| Finite strings $\Sigma^*$ over finite $\Sigma$ | Countable ($\aleph_0$) | Canonical length-lexicographic order ($\epsilon, 0, 1, 00, \dots$) |
| Valid C programs / Turing Machines | Countable ($\aleph_0$) | Each is a finite string over a finite ASCII alphabet |
| Real numbers $\mathbb{R}$, Interval $(0, 1)$ | Uncountable ($\mathfrak{c} = 2^{\aleph_0}$) | Cantor's diagonal argument (constructed decimal differs from every item) |
| Power set of natural numbers $\mathcal{P}(\mathbb{N})$ | Uncountable ($\mathfrak{c}$) | Cantor's theorem: $\vert A\vert < \vert \mathcal{P}(A)\vert$ for every set $A$ |
| All functions $f: \mathbb{N} \to \{0, 1\}$ | Uncountable ($\mathfrak{c}$) | Equivalent to infinite binary sequences and $\mathcal{P}(\mathbb{N})$ |
| All languages over alphabet $\{0, 1\}$ | Uncountable ($\mathfrak{c}$) | The set of languages is $\mathcal{P}(\Sigma^*) = \mathcal{P}(\mathbb{N})$ |

**Key theorems on countability:**
1. Any subset of a countable set is countable.
2. The union of countably many countable sets is countable.
3. The Cartesian product of finitely many countable sets is countable: $\mathbb{N} \times \mathbb{N}$ is countable.

**Worked example 1 (Countability and power set analysis).**
Consider the following sets:
$S_1$: The set of all polynomial functions $p(x) = a_k x^k + \dots + a_0$ with integer coefficients $a_i \in \mathbb{Z}$.
$S_2$: The set of all subsets of prime numbers.
Classify whether each set is countable or uncountable, and compute the number of non-empty proper subsets of a set $X$ with $|X| = 4$.

*Step-by-step solution:*
1. **For $S_1$:** A polynomial of degree $k$ is determined by a tuple $(a_0, a_1, \dots, a_k) \in \mathbb{Z}^{k+1}$, which is a finite tuple of integers. The set of all polynomials is $\bigcup_{k=0}^\infty \mathbb{Z}^{k+1}$, a countable union of countable sets. Therefore, **$S_1$ is countable**.
2. **For $S_2$:** The set of prime numbers $P = \{2, 3, 5, 7, \dots\}$ is countably infinite ($|P| = \aleph_0$). The set of all subsets of $P$ is its power set $\mathcal{P}(P)$. By Cantor's theorem, $|\mathcal{P}(P)| = 2^{\aleph_0} = \mathfrak{c}$. Therefore, **$S_2$ is uncountable**.
3. **For power set counting:** If $|X| = 4$, total subsets $= 2^4 = 16$. Non-empty means excluding $\emptyset$ (1 subset); proper means excluding $X$ itself (1 subset). Number of non-empty proper subsets $= 16 - 2 = 14$.

## 2. Relations and their fundamental properties [PYQ: High]

### Definition and matrix representation
A **binary relation** $R$ from $A$ to $B$ is a subset $R \subseteq A \times B$. If $A = B$, $R$ is a relation on $A$.
We can represent $R$ on $A = \{x_1, x_2, \dots, x_n\}$ by an $n \times n$ Boolean matrix $M_R$, where:
$$M_R[i, j] = \begin{cases} 1 & \text{if } (x_i, x_j) \in R \\ 0 & \text{if } (x_i, x_j) \notin R \end{cases}$$

### Fundamental properties

1. **Reflexive:** $\forall x \in A, (x, x) \in R$.
   - Matrix: All main diagonal entries $M[i, i] = 1$.
   - **Irreflexive:** $\forall x \in A, (x, x) \notin R$. All diagonal entries $M[i, i] = 0$.
   - *Trap:* Irreflexive is **not** the logical negation of reflexive. A relation can be neither (some diagonal entries 1, some 0).

2. **Symmetric:** $\forall x, y \in A, (x, y) \in R \implies (y, x) \in R$.
   - Matrix: $M_R = M_R^T$ (symmetric across the main diagonal).

3. **Antisymmetric:** $\forall x, y \in A, ((x, y) \in R \land (y, x) \in R) \implies x = y$.
   - In other words: If $x \neq y$, you cannot have both $(x, y) \in R$ and $(y, x) \in R$.
   - Matrix: For $i \neq j$, $M[i, j]$ and $M[j, i]$ cannot both be 1. Diagonal entries can be anything (0 or 1).
   - **Asymmetric:** $\forall x, y \in A, (x, y) \in R \implies (y, x) \notin R$. Equivalent to **irreflexive and antisymmetric**.

4. **Transitive:** $\forall x, y, z \in A, ((x, y) \in R \land (y, z) \in R) \implies (x, z) \in R$.
   - Matrix: $M_R^2 \le M_R$ (Boolean matrix multiplication).

### Closures of relations
The closure of $R$ with respect to a property $P$ is the smallest relation containing $R$ that satisfies $P$:
- **Reflexive closure:** $R \cup \Delta$, where $\Delta = \{(x, x) \mid x \in A\}$.
- **Symmetric closure:** $R \cup R^{-1}$, where $R^{-1} = \{(y, x) \mid (x, y) \in R\}$.
- **Transitive closure:** $R^+ = \bigcup_{k=1}^n R^k$ (computed via Warshall's algorithm in $O(n^3)$ time).

**Worked example 2 (Property verification and closures).**
Let $A = \{1, 2, 3\}$. Given relation $R = \{(1, 2), (2, 3), (1, 3), (3, 1)\}$.
Determine which properties $R$ satisfies, and compute its transitive closure $R^+$.

*Step-by-step solution:*
1. **Reflexive:** Requires $(1, 1), (2, 2), (3, 3) \in R$. None are present $\implies$ Not reflexive. In fact, since no diagonal pair is present, $R$ is **irreflexive**.
2. **Symmetric:** $(1, 2) \in R$ but $(2, 1) \notin R \implies$ **Not symmetric**.
3. **Antisymmetric:** Notice $(1, 3) \in R$ and $(3, 1) \in R$, but $1 \neq 3$. Both directions exist between distinct elements $\implies$ **Not antisymmetric**.
4. **Transitive:** $(2, 3) \in R$ and $(3, 1) \in R$, which requires $(2, 1) \in R$. But $(2, 1) \notin R \implies$ **Not transitive**.
5. **Transitive closure $R^+$:**
   - $R^1 = \{(1, 2), (2, 3), (1, 3), (3, 1)\}$.
   - Two-step paths $R^2$:
     - $(1, 2)$ and $(2, 3) \implies (1, 3)$ (already in $R$).
     - $(2, 3)$ and $(3, 1) \implies (2, 1)$.
     - $(3, 1)$ and $(1, 2) \implies (3, 2)$.
     - $(3, 1)$ and $(1, 3) \implies (3, 3)$.
     - $(1, 3)$ and $(3, 1) \implies (1, 1)$.
   - Three-step paths $R^3$:
     - $(2, 1)$ and $(1, 3) \implies (2, 3)$; $(2, 1)$ and $(1, 2) \implies (2, 2)$.
     - $(3, 2)$ and $(2, 3) \implies (3, 3)$; $(3, 2)$ and $(2, 1) \implies (3, 1)$.
   - All pairs $(x, y) \in A \times A$ are generated! Thus $R^+ = A \times A$, containing all $3^2 = 9$ ordered pairs.

## 3. Counting relations on a finite set [PYQ: High]

Let $|A| = n$. The Cartesian product $A \times A$ has $n^2$ pairs, split into:
- $n$ diagonal pairs $(x_i, x_i)$.
- $n^2 - n = n(n - 1)$ off-diagonal pairs, which group into $\binom{n}{2} = \frac{n(n-1)}{2}$ symmetric pairs $\{(x_i, x_j), (x_j, x_i)\}$.

```text
n x n relation matrix:
       x1     x2     x3   ...   xn
x1   [ d1   | p12  | p13  | ... | p1n ]    n diagonal elements:
x2   [ p21  | d2   | p23  | ... | p2n ]    d_i in {(xi, xi)}
x3   [ p31  | p32  | d3   | ... | p3n ]    n(n-1)/2 symmetric pairs:
..   [ ...  | ...  | ...  | ... | ... ]    { (xi, xj), (xj, xi) }
xn   [ pn1  | pn2  | pn3  | ... | dn  ]
```

| Type of relation | Diagonal choices | Off-diagonal pair choices | Total count formula |
|---|---|---|---|
| **Total relations** | $2$ choices each ($2^n$) | $4$ choices each ($4^{n(n-1)/2}$) | $2^{n^2}$ |
| **Reflexive** | Must be 1 ($1^n = 1$) | Any ($2^{n(n-1)}$) | $2^{n(n-1)}$ |
| **Irreflexive** | Must be 0 ($1^n = 1$) | Any ($2^{n(n-1)}$) | $2^{n(n-1)}$ |
| **Symmetric** | Any ($2^n$) | Both in or both out: 2 choices | $2^n \cdot 2^{n(n-1)/2} = 2^{n(n+1)/2}$ |
| **Reflexive & Symmetric** | Must be 1 ($1$) | 2 choices per pair | $2^{n(n-1)/2}$ |
| **Irreflexive & Symmetric** | Must be 0 ($1$) | 2 choices per pair | $2^{n(n-1)/2}$ |
| **Antisymmetric** | Any ($2^n$) | 3 choices: neither, $(i,j)$, or $(j,i)$ | $2^n \cdot 3^{n(n-1)/2}$ |
| **Reflexive & Antisymmetric** | Must be 1 ($1$) | 3 choices per pair | $3^{n(n-1)/2}$ |
| **Asymmetric (Irref & Antisym)** | Must be 0 ($1$) | 3 choices per pair | $3^{n(n-1)/2}$ |
| **Symmetric & Antisymmetric** | Any ($2^n$) | Cannot pick both $\implies$ only neither (1 choice) | $2^n$ |
| **Equivalence relations** | Partitions of $A$ | Partitions of $A$ | Bell number $B(n)$ |

**Worked example 3 (Numerical relation counting on small sets).**
Let $A = \{1, 2, 3\}$ ($n = 3$). Compute the count of relations that are:
(a) Symmetric, (b) Antisymmetric, (c) Both reflexive and antisymmetric, (d) Neither reflexive nor irreflexive.

*Step-by-step solution:*
1. Here $n = 3$, $n(n-1)/2 = 3(2)/2 = 3$.
2. **(a) Symmetric:** $2^{n(n+1)/2} = 2^{3(4)/2} = 2^6 = 64$.
3. **(b) Antisymmetric:** $2^n \cdot 3^{n(n-1)/2} = 2^3 \cdot 3^3 = 8 \cdot 27 = 216$.
4. **(c) Reflexive & Antisymmetric:** $3^{n(n-1)/2} = 3^3 = 27$.
5. **(d) Neither reflexive nor irreflexive:**
   - Reflexive requires all 3 diagonal elements to be 1 ($1$ choice of diagonal).
   - Irreflexive requires all 3 diagonal elements to be 0 ($1$ choice of diagonal).
   - Total diagonal configurations $= 2^3 = 8$.
   - Diagonals with at least one 1 and at least one 0 $= 8 - 1 - 1 = 6$ choices.
   - For each diagonal choice, the $n(n-1) = 6$ off-diagonal elements have no restriction ($2^6 = 64$ choices).
   - Total $= 6 \times 2^6 = 6 \times 64 = 384$.

## 4. Equivalence relations, equivalence classes, and partitions [PYQ: High]

### Equivalence relation and partition theorem
A relation $R$ on $A$ is an **equivalence relation** if and only if it is:
1. **Reflexive:** $(a, a) \in R$ for all $a \in A$.
2. **Symmetric:** $(a, b) \in R \implies (b, a) \in R$.
3. **Transitive:** $(a, b) \in R \land (b, c) \in R \implies (a, c) \in R$.

For any $a \in A$, the **equivalence class** of $a$ is:
$$[a] = \{x \in A \mid (x, a) \in R\}$$

**Fundamental Theorem of Equivalence Relations:**
- Every equivalence relation $R$ on $A$ partitions $A$ into mutually disjoint, non-empty equivalence classes whose union is $A$.
- Conversely, every partition $\{A_1, A_2, \dots, A_k\}$ of $A$ induces an equivalence relation: $(a, b) \in R \iff a, b$ belong to the same block $A_i$.

### Bell numbers and Stirling numbers
The number of distinct partitions of an $n$-element set into $k$ non-empty blocks is the **Stirling number of the second kind** $S(n, k)$:
$$S(n, k) = k \cdot S(n-1, k) + S(n-1, k-1), \quad S(n, 1) = S(n, n) = 1$$

The total number of equivalence relations on an $n$-element set is the **Bell number** $B(n)$:
$$B(n) = \sum_{k=1}^n S(n, k), \quad B(n+1) = \sum_{k=0}^n \binom{n}{k} B(k)$$

| $n$ | $S(n, 1), \dots, S(n, n)$ | Bell number $B(n)$ |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 1, 1 | 2 |
| 3 | 1, 3, 1 | 5 |
| 4 | 1, 7, 6, 1 | 15 |
| 5 | 1, 15, 25, 10, 1 | 52 |

### Circular relations (GATE 2021)
A relation $R$ is defined to be **circular** if:
$$\forall a, b, c \in A, \quad ((a, b) \in R \land (b, c) \in R) \implies (c, a) \in R$$

- **Theorem:** If $R$ is **reflexive and circular**, then $R$ is an **equivalence relation**.
  - *Proof of symmetry:* Let $(a, b) \in R$. Since $R$ is reflexive, $(b, b) \in R$. Applying circularity to $(a, b)$ and $(b, b)$ gives $(b, a) \in R$. Hence $R$ is symmetric.
  - *Proof of transitivity:* Let $(a, b) \in R$ and $(b, c) \in R$. Circularity gives $(c, a) \in R$. Since $R$ is symmetric, $(c, a) \in R \implies (a, c) \in R$. Hence $R$ is transitive.
  - Reflexive + symmetric + transitive = equivalence relation.

**Worked example 4 (Equivalence classes and modulo relation).**
Let $A = \mathbb{Z}$ and $n = 4$. Define $aRb \iff a \equiv b \pmod 4$ (i.e., $4 \mid (a - b)$).
Prove $R$ is an equivalence relation, find all distinct equivalence classes, and find the number of equivalence relations on $S = \{1, 2, 3, 4\}$ having exactly 2 equivalence classes.

*Step-by-step solution:*
1. **Reflexive:** $a - a = 0 = 4 \times 0 \implies a \equiv a \pmod 4$ for all $a \in \mathbb{Z}$.
2. **Symmetric:** $a - b = 4k \implies b - a = 4(-k) \implies b \equiv a \pmod 4$.
3. **Transitive:** If $a - b = 4k$ and $b - c = 4m$, then $a - c = (a - b) + (b - c) = 4(k + m)$, so $a \equiv c \pmod 4$.
4. **Equivalence classes:**
   $[0] = 4\mathbb{Z}$, $[1] = 4\mathbb{Z} + 1$, $[2] = 4\mathbb{Z} + 2$, $[3] = 4\mathbb{Z} + 3$.
   There are exactly 4 distinct equivalence classes.
5. **Equivalence relations on 4 elements with 2 classes:** This equals $S(4, 2)$, the number of ways to partition a 4-element set into 2 non-empty subsets:
   $$S(4, 2) = \frac{1}{2!} \sum_{j=0}^2 (-1)^{2-j} \binom{2}{j} j^4 = \frac{1}{2} (2^4 - 2 \cdot 1^4) = \frac{16 - 2}{2} = 7$$
   Alternatively, size splits are $1+3$ ($\binom{4}{1} = 4$ ways) and $2+2$ ($\frac{1}{2}\binom{4}{2} = 3$ ways), giving $4 + 3 = 7$.

## 5. Partial orders and Poset foundations [PYQ: Medium]

### Definitions
A relation $R$ on set $P$ is a **partial order** (and $(P, R)$ is a partially ordered set or **poset**) if it is:
1. **Reflexive:** $\forall x \in P, (x, x) \in R$.
2. **Antisymmetric:** $\forall x, y \in P, ((x, y) \in R \land (y, x) \in R) \implies x = y$.
3. **Transitive:** $\forall x, y, z \in P, ((x, y) \in R \land (y, z) \in R) \implies (x, z) \in R$.

- **Comparable elements:** $x, y \in P$ are comparable if $x \le y$ or $y \le x$.
- **Total order (linear order):** A partial order where every pair of elements is comparable ($\forall x, y, x \le y \lor y \le x$).
- **Strict partial order:** Irreflexive + Transitive (e.g. strict inequality $<$ or strict subset $\subset$).

**Worked example 5 (Partial order on function sets — GATE 2025).**
Let $\mathcal{F}$ be the set of all functions from $\{1, 2\}$ to $\{1, 2, 3\}$.
Define relation $\preceq$ on $\mathcal{F}$ by:
$$f \preceq g \iff f(1) \le g(1) \land f(2) \le g(2)$$
Show that $(\mathcal{F}, \preceq)$ is a partial order, and determine whether it is a total order.

*Step-by-step solution:*
1. **Reflexivity:** For any $f \in \mathcal{F}$, $f(1) \le f(1)$ and $f(2) \le f(2)$ holds in the standard $\le$ on integers. Thus $f \preceq f$.
2. **Antisymmetry:** If $f \preceq g$ and $g \preceq f$:
   - $f(1) \le g(1)$ and $g(1) \le f(1) \implies f(1) = g(1)$.
   - $f(2) \le g(2)$ and $g(2) \le f(2) \implies f(2) = g(2)$.
   Since $f(x) = g(x)$ for all inputs $x \in \{1, 2\}$, $f = g$. Hence $\preceq$ is antisymmetric.
3. **Transitivity:** If $f \preceq g$ and $g \preceq h$, then $f(1) \le g(1) \le h(1) \implies f(1) \le h(1)$, and $f(2) \le g(2) \le h(2) \implies f(2) \le h(2)$. Thus $f \preceq h$.
4. **Is it a total order?**
   Consider $f_1 = \{(1, 1), (2, 3)\}$ and $f_2 = \{(1, 2), (2, 1)\}$.
   $f_1(1) = 1 < f_2(1) = 2$, but $f_1(2) = 3 > f_2(2) = 1$.
   Neither $f_1 \preceq f_2$ nor $f_2 \preceq f_1$ holds. They are **incomparable**.
   Hence $(\mathcal{F}, \preceq)$ is a partial order, but **not a total order**.

## 6. Functions, compositions, and counting mappings [PYQ: High]

### Definitions
A **function** $f: A \to B$ associates with each element $a \in A$ exactly one element $f(a) \in B$.
Let $|A| = m$ (domain size) and $|B| = n$ (codomain size).

```text
Domain A (|A|=m)         Codomain B (|B|=n)
    a1 ---------\
                 +-----> b1
    a2 ---------/
    a3 ----------------> b2
                         b3  (unmapped in non-surjective)
Every a in A must have exactly 1 outgoing arrow.
Injective: no two arrows land on the same b.
Surjective: every b in B has at least 1 incoming arrow.
```

- **Injective (One-to-One):** $f(a_1) = f(a_2) \implies a_1 = a_2$.
  - Pigeonhole constraint: requires $m \le n$. If $m > n$, no injective function exists.
- **Surjective (Onto):** $\forall b \in B, \exists a \in A \text{ such that } f(a) = b$ (range = codomain).
  - Constraint: requires $m \ge n$. If $m < n$, no surjective function exists.
- **Bijective (One-to-One Correspondence):** Injective and Surjective.
  - Requires $m = n$. Invertible: $f^{-1}: B \to A$ exists.

### Counting mappings ($|A| = m, |B| = n$)

1. **Total functions:** $n^m$. (Each of the $m$ elements in $A$ has $n$ independent choices in $B$).
2. **Injective functions ($m \le n$):**
   $$P(n, m) = n(n - 1)(n - 2)\cdots(n - m + 1) = \frac{n!}{(n - m)!}$$
3. **Surjective functions ($m \ge n$):**
   Using Principle of Inclusion–Exclusion (PIE):
   $$\text{Onto}(m, n) = \sum_{k=0}^n (-1)^k \binom{n}{k} (n - k)^m = n! \cdot S(m, n)$$
4. **Bijective functions ($m = n$):** $n!$.
5. **Strictly increasing functions** $f: \{1, \dots, m\} \to \{1, \dots, n\}$ ($m \le n$): $\binom{n}{m}$.
   (Once we choose $m$ distinct values from $n$, there is exactly 1 way to assign them in increasing order).
6. **Non-decreasing functions** $f: \{1, \dots, m\} \to \{1, \dots, n\}$: $\binom{n + m - 1}{m}$.

### Composition and inverses
For $f: A \to B$ and $g: B \to C$, the composite function is $(g \circ f): A \to C$ defined by $(g \circ f)(x) = g(f(x))$.
- **Associative:** $h \circ (g \circ f) = (h \circ g) \circ f$.
- **Not commutative:** $g \circ f \neq f \circ g$ in general.
- **Injectivity inheritance:**
  $$\text{If } g \circ f \text{ is injective} \implies f \text{ is injective} \quad (\text{no claim about } g)$$
- **Surjectivity inheritance:**
  $$\text{If } g \circ f \text{ is surjective} \implies g \text{ is surjective} \quad (\text{no claim about } f)$$
- **Inverse of composite:** $(g \circ f)^{-1} = f^{-1} \circ g^{-1}$.

**Worked example 6 (Onto function calculation with full steps).**
Let $A = \{1, 2, 3, 4, 5\}$ and $B = \{x, y, z\}$.
Compute: (a) Total functions, (b) Injective functions, (c) Surjective functions from $A$ to $B$.

*Step-by-step solution:*
Here $|A| = m = 5$, $|B| = n = 3$.
1. **(a) Total functions:** $n^m = 3^5 = 243$.
2. **(b) Injective functions:** Since $m = 5 > n = 3$, by the Pigeonhole Principle, at least two domain elements must collide. Injective functions $= 0$.
3. **(c) Surjective functions:**
   - Using PIE:
     $$\text{Onto}(5, 3) = \binom{3}{0} 3^5 - \binom{3}{1} 2^5 + \binom{3}{2} 1^5 - \binom{3}{3} 0^5$$
     - $\binom{3}{0} 3^5 = 1 \times 243 = 243$.
     - $\binom{3}{1} 2^5 = 3 \times 32 = 96$ (functions missing at least 1 element of $B$).
     - $\binom{3}{2} 1^5 = 3 \times 1 = 3$ (functions missing at least 2 elements of $B$).
     - $\binom{3}{3} 0^5 = 0$.
   - $\text{Onto}(5, 3) = 243 - 96 + 3 = 150$.
   - *Verification via Stirling numbers:*
     Partitions of 5 into 3 parts: size splits can be $3+1+1$ ($\binom{5}{3} \cdot 1 = 10$ ways) or $2+2+1$ ($\frac{1}{2!} \binom{5}{2} \binom{3}{2} = \frac{10 \times 3}{2} = 15$ ways).
     $S(5, 3) = 10 + 15 = 25$.
     $\text{Onto}(5, 3) = 3! \cdot S(5, 3) = 6 \times 25 = 150$. Both methods agree.

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Total relations on $\vert A\vert = n$ | $2^{n^2}$ | Number of binary relations on $n$ elements |
| Reflexive / Irreflexive relations | $2^{n(n-1)}$ | Fixed diagonal ($1$s or $0$s), off-diagonals free |
| Symmetric relations | $2^{n(n+1)/2}$ | Diagonal free, off-diagonal symmetric pairs chosen together |
| Antisymmetric relations | $2^n \cdot 3^{n(n-1)/2}$ | Diagonal free ($2^n$), 3 choices per off-diagonal pair |
| Reflexive & Symmetric | $2^{n(n-1)/2}$ | Diagonal fixed to 1, off-diagonal symmetric pairs 2 choices |
| Reflexive & Antisymmetric | $3^{n(n-1)/2}$ | Diagonal fixed to 1, off-diagonal pairs 3 choices |
| Asymmetric relations | $3^{n(n-1)/2}$ | Irreflexive + antisymmetric |
| Equivalence relations | Bell number $B(n)$ ($1, 2, 5, 15, 52$) | Number of partitions of an $n$-element set |
| Total functions $A \to B$ | $\vert B\vert ^{\vert A\vert } = n^m$ | Mappings from domain $m$ to codomain $n$ |
| Injective functions | $\frac{n!}{(n-m)!}$ for $m \le n$, else $0$ | One-to-one mapping counts |
| Surjective functions | $n! \cdot S(m, n) = \sum_{k=0}^n (-1)^k \binom{n}{k} (n-k)^m$ | Onto mapping counts ($m \ge n$) |
| Strictly increasing functions | $\binom{n}{m}$ | Order-preserving strict mappings from $\{1..m\} \to \{1..n\}$ |
| Composition injectivity | $g \circ f$ injective $\implies f$ injective | MSQ deduction on function pipelines |
| Composition surjectivity | $g \circ f$ surjective $\implies g$ surjective | MSQ deduction on function pipelines |

## GATE traps

- **Antisymmetric vs Not Symmetric:** "Antisymmetric" does **not** mean "not symmetric". A relation can be both (e.g., identity relation $\{(x, x)\}$), neither, symmetric only, or antisymmetric only.
- **Empty relation on $A \neq \emptyset$:** The empty relation $\emptyset$ is **irreflexive, symmetric, antisymmetric, and transitive**, but **not reflexive** (and thus not an equivalence relation).
- **Universal relation $A \times A$:** Reflexive, symmetric, transitive $\implies$ equivalence relation. But **not antisymmetric** (unless $|A| \le 1$).
- **Onto function count when $m < n$:** Always 0. Do not blindly apply Stirling numbers without checking domain vs codomain size.
- **Composition order:** $(g \circ f)(x) = g(f(x))$ means $f$ operates **first**, and $g$ operates **second**.
- **Injective / Surjective transfer:**
  - $g \circ f$ injective guarantees $f$ is injective, but $g$ **might not** be injective.
  - $g \circ f$ surjective guarantees $g$ is surjective, but $f$ **might not** be surjective.
- **Countable union trap:** A countable union of countable sets is countable. However, an **uncountable** union of countable sets (e.g. $\bigcup_{x \in \mathbb{R}} \{x\} = \mathbb{R}$) is uncountable!

## Connections

- [Posets and lattices](posets-and-lattices.md) — partial orders provide the foundational ordering relation for bounds, Hasse diagrams, and lattices.
- [Combinatorics](combinatorics.md) — function counts and equivalence relation counts connect directly with Stirling numbers, Bell numbers, and the Twelvefold Way.
- [Graph theory](graph-theory.md) — any binary relation is a directed graph; symmetric relations correspond to undirected graphs; equivalence relations are disjoint complete subgraphs.
- [Propositional logic](propositional-logic.md) — relation properties are quantified first-order statements over pairs.
- [Theory of computation](../11-theory-of-computation/regular-languages-and-finite-automata.md) — equivalence relations with finite index underpin the Myhill–Nerode theorem and DFA state minimization.

## Practice

**Q1 (MCQ).** Let $A = \{1, 2, 3, 4\}$. The number of binary relations on $A$ that are both reflexive and symmetric is:
(A) 16 (B) 64 (C) 256 (D) 1024  
<details><summary>Answer</summary>

**Answer:** (B) · **Solution:**
For a relation to be reflexive, all $n = 4$ diagonal pairs $(i, i)$ must be present (1 choice for each).
For symmetry, for each of the $\binom{n}{2} = \binom{4}{2} = 6$ unordered pairs $\{i, j\}$ ($i \neq j$), both $(i, j)$ and $(j, i)$ must either both be in the relation or both be absent (2 choices per pair).
$$\text{Count} = 1^4 \times 2^{\binom{4}{2}} = 2^6 = 64$$
</details>

**Q2 (NAT).** Let $A = \{1, 2, 3, 4, 5\}$ and $B = \{1, 2, 3\}$. The number of surjective (onto) functions from $A$ to $B$ is ________.
<details><summary>Answer</summary>

**Answer:** 150 · **Solution:**
Here $m = |A| = 5$ and $n = |B| = 3$. By the Principle of Inclusion–Exclusion:
$$\text{Onto}(5, 3) = \sum_{k=0}^3 (-1)^k \binom{3}{k} (3 - k)^5$$
- $k = 0$: $\binom{3}{0} 3^5 = 1 \times 243 = 243$
- $k = 1$: $-\binom{3}{1} 2^5 = -3 \times 32 = -96$
- $k = 2$: $+\binom{3}{2} 1^5 = +3 \times 1 = +3$
- $k = 3$: $-\binom{3}{3} 0^5 = 0$
$$\text{Onto}(5, 3) = 243 - 96 + 3 = 150$$
</details>

**Q3 (MSQ).** A binary relation $R$ on a set $A$ is said to be *circular* if for all $a, b, c \in A$, $(a, b) \in R$ and $(b, c) \in R$ together imply $(c, a) \in R$. Which of the following statements is/are TRUE?
(A) If $R$ is reflexive and circular, then $R$ is an equivalence relation.  
(B) If $R$ is circular and symmetric, then $R$ is an equivalence relation.  
(C) If $R$ is transitive and symmetric, then $R$ is circular.  
(D) If $R$ is an equivalence relation, then $R$ is circular.  
<details><summary>Answer</summary>

**Answer:** (A), (C), (D) · **Solution:**
- **(A) is TRUE:**
  - Let $(a, b) \in R$. Since $R$ is reflexive, $(b, b) \in R$. By circularity on $(a, b)$ and $(b, b)$, we get $(b, a) \in R$. So $R$ is symmetric.
  - Let $(a, b) \in R$ and $(b, c) \in R$. By circularity, $(c, a) \in R$. Since $R$ is symmetric, $(a, c) \in R$. So $R$ is transitive.
  - Reflexive + symmetric + transitive $\implies$ equivalence relation.
- **(B) is FALSE:** Let $A = \{1, 2\}$ and $R = \emptyset$. The empty relation is vacuously circular and symmetric, but not reflexive on $A$, so not an equivalence relation.
- **(C) is TRUE:** If $(a, b) \in R$ and $(b, c) \in R$, by transitivity $(a, c) \in R$. By symmetry, $(a, c) \in R \implies (c, a) \in R$. Thus $R$ is circular.
- **(D) is TRUE:** An equivalence relation is symmetric and transitive, hence by (C) it is circular.
</details>

**Q4 (NAT).** Let $A = \{1, 2, 3, 4\}$. The number of equivalence relations on $A$ in which the elements $1$ and $2$ belong to the same equivalence class is ________.
<details><summary>Answer</summary>

**Answer:** 5 · **Solution:**
An equivalence relation corresponds to a partition of $A$.
We are given that $1$ and $2$ must be in the same partition block. We can treat the pair $\{1, 2\}$ as a single coalesced entity $X$.
The problem reduces to finding the number of partitions of the 3-element set $\{X, 3, 4\}$.
The number of partitions of a 3-element set is the Bell number $B(3)$:
$$B(3) = 5$$
The 5 partitions are: 1 block: $\{\{1, 2, 3, 4\}\}$; 2 blocks: $\{\{1, 2, 3\}, \{4\}\}$, $\{\{1, 2, 4\}, \{3\}\}$, $\{\{1, 2\}, \{3, 4\}\}$; 3 blocks: $\{\{1, 2\}, \{3\}, \{4\}\}$.
</details>

**Q5 (MCQ).** Which of the following sets is UNCOUNTABLE?
(A) The set of all valid C source programs that compile cleanly under `gcc`.  
(B) The set of all rational numbers $\mathbb{Q}$.  
(C) The set of all functions from the natural numbers $\mathbb{N}$ to $\{0, 1\}$.  
(D) The set of all finite subsets of natural numbers.  
<details><summary>Answer</summary>

**Answer:** (C) · **Solution:**
- (A) Every C program is a finite string over a finite ASCII character set $\Sigma$. The set of all finite strings $\Sigma^*$ is countably infinite. Valid programs form a subset of $\Sigma^*$, so the set is countable.
- (B) $\mathbb{Q}$ is countably infinite (can be enumerated via diagonalization on $\mathbb{Z} \times \mathbb{N}$).
- (C) A function $f: \mathbb{N} \to \{0, 1\}$ corresponds to an infinite binary sequence $(f(0), f(1), f(2), \dots)$. The set of all such sequences has cardinality $2^{\aleph_0} = \mathfrak{c}$, which is uncountable by Cantor's diagonal argument.
- (D) The set of all finite subsets of $\mathbb{N}$ is $\bigcup_{k=0}^\infty \{S \subset \mathbb{N} \mid |S| = k\}$, a countable union of countable sets, hence countable.
</details>

**Q6 (MSQ).** Let $f: A \to B$ and $g: B \to C$ be functions. Which of the following statements is/are always TRUE?
(A) If $g \circ f$ is injective, then $f$ must be injective.  
(B) If $g \circ f$ is injective, then $g$ must be injective.  
(C) If $g \circ f$ is surjective, then $g$ must be surjective.  
(D) If $g \circ f$ is surjective, then $f$ must be surjective.  
<details><summary>Answer</summary>

**Answer:** (A), (C) · **Solution:**
- **(A) is TRUE:** Suppose $f(x_1) = f(x_2)$. Then $g(f(x_1)) = g(f(x_2)) \implies (g \circ f)(x_1) = (g \circ f)(x_2)$. Since $g \circ f$ is injective, $x_1 = x_2$. Thus $f$ is injective.
- **(B) is FALSE:** Counterexample: Let $A = \{1\}$, $B = \{1, 2\}$, $C = \{1\}$. Let $f(1) = 1$, and $g(1) = 1, g(2) = 1$. Then $(g \circ f)(1) = 1$, which is injective on $A$, but $g$ is not injective since $g(1) = g(2) = 1$.
- **(C) is TRUE:** Let $c \in C$. Since $g \circ f$ is surjective, there exists $a \in A$ such that $(g \circ f)(a) = c \implies g(f(a)) = c$. Let $b = f(a) \in B$; then $g(b) = c$. Thus every element in $C$ has a pre-image in $B$, so $g$ is surjective.
- **(D) is FALSE:** In the same counterexample above: $g \circ f$ maps $\{1\} \to \{1\}$, which is surjective. But $f$ maps $\{1\} \to \{1, 2\}$, leaving $2$ uncovered, so $f$ is not surjective.
</details>

**Q7 (NAT).** Let $A = \{1, 2, 3, 4, 5, 6\}$. The number of bijective functions $f: A \to A$ such that $f(i) \neq i$ for all $i \in A$ and $f(f(i)) = i$ for all $i \in A$ is ________.
<details><summary>Answer</summary>

**Answer:** 15 · **Solution:**
The condition $f(f(i)) = i$ means $f$ is an involution ($f = f^{-1}$). In cycle notation, every cycle in $f$ has length either 1 or 2.
The condition $f(i) \neq i$ for all $i$ means there are **no fixed points** (no 1-cycles).
Therefore, every element of $A$ must belong to a 2-cycle. Since $|A| = 6$, $f$ must consist of exactly three disjoint 2-cycles:
$$(a_1, a_2)(a_3, a_4)(a_5, a_6)$$
The number of ways to partition a set of 6 distinct elements into 3 unordered pairs of size 2 is:
$$\frac{\binom{6}{2} \times \binom{4}{2} \times \binom{2}{2}}{3!} = \frac{15 \times 6 \times 1}{6} = 15$$
</details>
