# Posets and lattices

> **Paper:** CS · **Priority:** P1 · **Plan topics:** partial orders, Hasse diagrams, maximal/minimal elements, bounds, lattices
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Boolean algebra and minimization](../09-digital-logic/boolean-algebra-and-minimization.md)
> **PYQ weightage:** Low · 3 Qs since 2015 (1 since 2021) · DA: 0 · Trend: steady · Most asked: Lattices (distributive/complemented/Boolean), Hasse diagrams & linear extensions

## Quick glance

- **Exam focus:** Recognizing partial orders, constructing/interpreting Hasse diagrams, identifying extremal elements (maximal, minimal, greatest, least), evaluating meets and joins, classifying lattices (distributive, complemented, Boolean), and counting linear extensions (total orders). Usually asked as 1-mark or 2-mark MCQ/MSQ/NAT questions.
- A **partially ordered set (poset)** $(S, \le)$ satisfies three axioms: **reflexive** ($a \le a$), **antisymmetric** ($a \le b \land b \le a \implies a = b$), and **transitive** ($a \le b \land b \le c \implies a \le c$).
- In a **Hasse diagram**, reflexive self-loops and transitive shortcuts are omitted; if $a < b$ with no intermediate element ($b$ covers $a$), $b$ is drawn strictly higher than $a$ with a connecting line.
- **Greatest vs Maximal:** A greatest element $g$ satisfies $\forall x, x \le g$ (unique if it exists). A maximal element $m$ satisfies $\nexists x, m < x$. A finite poset can have multiple maximal elements and NO greatest element.
- A **lattice** is a poset where every pair $\{a, b\}$ has a unique **least upper bound** (LUB / join, $a \vee b$) and a unique **greatest lower bound** (GLB / meet, $a \wedge b$).
- In divisor lattices $(D_n, \mid)$: meet is $\gcd(a, b)$, join is $\text{lcm}(a, b)$, least element is $1$, greatest element is $n$.
- A lattice is **distributive** iff it does not contain the diamond $M_3$ or pentagon $N_5$ as a sublattice.
- A **Boolean lattice** is both distributive and complemented. Every finite Boolean lattice has $2^k$ elements and is isomorphic to a power set $(\mathcal{P}(S), \subseteq)$.
- $D_n$ is a Boolean lattice iff $n$ is **square-free** (product of distinct prime factors).
- A **linear extension** of a poset is a total order that respects all poset relations; counting linear extensions equals counting **topological sorts** of the poset's DAG.

## 1. Partially ordered sets (Posets) and Hasse diagrams [PYQ: Low]

A relation $R$ on a set $S$ is a **partial order** (denoted $\le$) if it satisfies:
1. **Reflexivity:** $a \le a$ for all $a \in S$.
2. **Antisymmetry:** If $a \le b$ and $b \le a$, then $a = b$.
3. **Transitivity:** If $a \le b$ and $b \le c$, then $a \le c$.

The pair $(S, \le)$ is called a **partially ordered set** or **poset**. Elements $a, b \in S$ are **comparable** if $a \le b$ or $b \le a$; otherwise they are **incomparable** (denoted $a \parallel b$).

### Cover relation and Hasse diagram
In a poset $(S, \le)$, element $y$ **covers** $x$ (written $x \prec y$) if $x < y$ and there is no $z \in S$ such that $x < z < y$.

A **Hasse diagram** is a visual representation of a finite poset:
- Elements are drawn as vertices.
- If $x \prec y$, $y$ is placed vertically above $x$ and a line segment connects them.
- Reflexive loops ($x \le x$) are omitted.
- Transitive lines (edges between $x$ and $z$ when $x < y < z$) are omitted.
- Direction is implicitly upward: $a \le b$ iff there is an upward path from $a$ to $b$.

```text
Divisors of 12 (D_12) under divisibility:
           12
         /    \
        4      6
        |    /   \
        2   /     3
         \ /     /
          1 ----'
```

**Worked example 1 (constructing a Hasse diagram).** Let $S = \{1, 2, 3, 4, 6, 9, 12, 18, 36\}$ be $D_{36}$ (divisors of 36) ordered by divisibility ($a \le b \iff a \mid b$).
1. Prime factorisation: $36 = 2^2 \times 3^2$. Total divisors = $(2+1)(2+1) = 9$.
2. Find cover relations ($a \prec b$ means $b/a$ is prime):
   - From 1: $1 \prec 2, 1 \prec 3$.
   - From 2: $2 \prec 4, 2 \prec 6$.
   - From 3: $3 \prec 6, 3 \prec 9$.
   - From 4: $4 \prec 12$.
   - From 6: $6 \prec 12, 6 \prec 18$.
   - From 9: $9 \prec 18$.
   - From 12: $12 \prec 36$.
   - From 18: $18 \prec 36$.
3. Levels in Hasse diagram:
   - Level 0: $\{1\}$
   - Level 1: $\{2, 3\}$
   - Level 2: $\{4, 6, 9\}$
   - Level 3: $\{12, 18\}$
   - Level 4: $\{36\}$

## 2. Extremal elements: maximal, minimal, greatest, and least [PYQ: Low]

In any poset $(S, \le)$:

| Concept | Formal definition | Existence & uniqueness |
|---|---|---|
| **Maximal element** | $m \in S$ such that $\nexists x \in S, m < x$ (no element is strictly above $m$) | May have none, one, or multiple |
| **Minimal element** | $m \in S$ such that $\nexists x \in S, x < m$ (no element is strictly below $m$) | May have none, one, or multiple |
| **Greatest element (Maximum / Top / 1)** | $g \in S$ such that $\forall x \in S, x \le g$ | At most one (unique if it exists) |
| **Least element (Minimum / Bottom / 0)** | $l \in S$ such that $\forall x \in S, l \le x$ | At most one (unique if it exists) |

### Key distinctions for GATE
- If a greatest element exists, it is the **unique maximal** element.
- However, having a single maximal element in an infinite poset does not guarantee a greatest element. In a finite poset, if there is a unique maximal element, it IS the greatest element.
- If a poset has two or more maximal elements, it has **no greatest element**.

```text
Poset with NO greatest and NO least element:
       d       e        Maximal: {d, e}
      / \     / \       Greatest: None (d and e are incomparable)
     /   \   /   \
    a      b      c     Minimal: {a, b, c}
                        Least: None
```

**Worked example 2.** In the poset shown above on $\{a, b, c, d, e\}$ where covers are $a \prec d, b \prec d, b \prec e, c \prec e$:
- Minimal elements: $\{a, b, c\}$ (no element below them).
- Maximal elements: $\{d, e\}$ (no element above them).
- Greatest element: None, because neither $d \le e$ nor $e \le d$.
- Least element: None, because no element is $\le$ all others.

## 3. Bounds, meets (GLB), and joins (LUB) [PYQ: Low]

Let $(S, \le)$ be a poset and $A \subseteq S$:
- **Upper bound of $A$:** An element $u \in S$ such that $x \le u$ for all $x \in A$.
- **Lower bound of $A$:** An element $l \in S$ such that $l \le x$ for all $x \in A$.
- **Least Upper Bound (LUB / Supremum / Join, $\vee$):** An upper bound $u^*$ of $A$ such that for any upper bound $u$ of $A$, $u^* \le u$.
- **Greatest Lower Bound (GLB / Infimum / Meet, $\wedge$):** A lower bound $l^*$ of $A$ such that for any lower bound $l$ of $A$, $l \le l^*$.

Uniqueness: If the LUB (or GLB) exists, antisymmetry guarantees it is **unique**.

**Worked example 3 (GLB and LUB in standard posets).**
1. In the power set poset $(\mathcal{P}(U), \subseteq)$:
   - For $A, B \subseteq U$: $\text{GLB}(A, B) = A \cap B$, $\text{LUB}(A, B) = A \cup B$.
2. In the divisibility poset $(\mathbb{Z}^+, \mid)$:
   - For $a, b \in \mathbb{Z}^+$: $\text{GLB}(a, b) = \gcd(a, b)$, $\text{LUB}(a, b) = \text{lcm}(a, b)$.
   - For example, in $D_{36}$: $\text{GLB}(12, 18) = \gcd(12, 18) = 6$; $\text{LUB}(12, 18) = \text{lcm}(12, 18) = 36$.

## 4. Lattices and algebraic properties [PYQ: Low]

A poset $(L, \le)$ is a **lattice** if every pair of elements $a, b \in L$ has both:
- A unique greatest lower bound: $a \wedge b$ (meet)
- A unique least upper bound: $a \vee b$ (join)

### Algebraic laws of a lattice
Viewing $(L, \wedge, \vee)$ as an algebraic system with two binary operations:
1. **Idempotence:** $a \wedge a = a$, $a \vee a = a$.
2. **Commutativity:** $a \wedge b = b \wedge a$, $a \vee b = b \vee a$.
3. **Associativity:** $(a \wedge b) \wedge c = a \wedge (b \wedge c)$, $(a \vee b) \vee c = a \vee (b \vee c)$.
4. **Absorption laws:** $a \wedge (a \vee b) = a$, $a \vee (a \wedge b) = a$.

Equivalence of order and algebra: $a \le b \iff a \wedge b = a \iff a \vee b = b$.

### How to test if a Hasse diagram is NOT a lattice
Look for a pair of elements $x, y$ such that:
- **Case 1 (No common upper bound):** $x$ and $y$ have no upper bound at all (e.g., disconnected tops).
- **Case 2 (Incomparable minimal upper bounds):** $x$ and $y$ have two or more common upper bounds $u_1, u_2$ that are incomparable (neither $u_1 \le u_2$ nor $u_2 \le u_1$), meaning no *least* upper bound exists.
- **Case 3 (Incomparable maximal lower bounds):** Dual of Case 2 for lower bounds.

```text
NOT a lattice (elements a, b have two minimal upper bounds c and d):
        e
       / \
      c   d
       \ / \
        a   b
```
In this diagram: common upper bounds of $\{a, b\}$ are $\{c, d, e\}$. The minimal upper bounds are $c$ and $d$. Since $c \parallel d$, there is no least upper bound. Thus it is **not a lattice**.

## 5. Special lattices: distributive, complemented, and Boolean [PYQ: Low]

### 1. Bounded lattice
A lattice $(L, \wedge, \vee)$ is **bounded** if it has a least element $0$ (bottom) and a greatest element $1$ (top):
$$a \wedge 0 = 0, \quad a \vee 0 = a, \quad a \wedge 1 = a, \quad a \vee 1 = 1$$
Every finite lattice is bounded.

### 2. Distributive lattice
A lattice is **distributive** if meet distributes over join, and join distributes over meet:
$$a \wedge (b \vee c) = (a \wedge b) \vee (a \wedge c)$$
$$a \vee (b \wedge c) = (a \vee b) \wedge (a \vee c)$$
(Either identity implies the other).

**Birkhoff's Diamond and Pentagon Theorem:**
A lattice is distributive if and only if it does **not** contain a sublattice isomorphic to:
- **$M_3$ (the diamond lattice):** 5 elements $\{0, a, b, c, 1\}$ where $a, b, c$ are mutually incomparable.
- **$N_5$ (the pentagon lattice):** 5 elements $\{0, a, b, c, 1\}$ with $a < b$ and $c$ incomparable to both.

```text
Diamond M_3 (Non-distributive):      Pentagon N_5 (Non-distributive):
          1                                     1
       /  |  \                                /   \
      a   b   c                              b     |
       \  |  /                               |     c
          0                                  a     |
                                              \   /
                                                0
```

*Proof check for $M_3$:* $a \wedge (b \vee c) = a \wedge 1 = a$. But $(a \wedge b) \vee (a \wedge c) = 0 \vee 0 = 0$. Since $a \ne 0$, distributivity fails!

### 3. Complemented lattice
In a bounded lattice $(L, \wedge, \vee, 0, 1)$, an element $b$ is a **complement** of $a$ if:
$$a \wedge b = 0 \quad \text{and} \quad a \vee b = 1$$
A lattice is **complemented** if every element has at least one complement.
- In general lattices, an element may have 0, 1, or multiple complements (in $M_3$, $a$ has complements $b$ and $c$).
- **Theorem:** In a **distributive** lattice, if an element has a complement, that complement is **unique**.

### 4. Boolean lattice (Boolean algebra)
A lattice is a **Boolean lattice** if it is **both distributive and complemented**.
- Every element $a$ has a **unique** complement $a'$.
- De Morgan's laws hold: $(a \vee b)' = a' \wedge b'$ and $(a \wedge b)' = a' \vee b'$.
- **Cardinality theorem:** Every finite Boolean lattice has $2^k$ elements for some integer $k \ge 0$, and is isomorphic to the power set $(\mathcal{P}(S), \subseteq)$ of a set with $k$ elements.

**Worked example 4 (divisibility lattices $D_n$).**
- Is $D_{30}$ a Boolean lattice?
  $30 = 2 \times 3 \times 5$. The prime factors are distinct (square-free). Divisors: $\{1, 2, 3, 5, 6, 10, 15, 30\}$. Total elements = $2^3 = 8$. Since every prime factor appears to power 1, $D_{30}$ is isomorphic to $\mathcal{P}(\{2, 3, 5\})$. Complement of $x$ is $30/x$. It is **Boolean**.
- Is $D_{12}$ a Boolean lattice?
  $12 = 2^2 \times 3$ (not square-free; 4 divides 12). Total elements = 6 (not a power of 2!). Consider element 2: for a complement $x$, $\gcd(2, x) = 1 \implies x \in \{1, 3\}$. But $\text{lcm}(2, 1) = 2 \ne 12$, and $\text{lcm}(2, 3) = 6 \ne 12$. Element 2 has **no complement**. Hence $D_{12}$ is distributive but **not complemented**, so **not Boolean**.

## 6. Chains, antichains, and linear extensions (total orders) [PYQ: Low]

- A subset $C \subseteq S$ is a **chain** if every pair of elements in $C$ is comparable ($C$ is totally ordered).
- A subset $A \subseteq S$ is an **antichain** if every pair of distinct elements in $A$ is incomparable.
- **Dilworth's Theorem:** In any finite poset, the minimum number of disjoint chains needed to cover the poset equals the size of the largest antichain (width of the poset).

### Linear extensions (Topological sorts)
A **total order (linear order)** on $S$ is a partial order where every pair of elements is comparable.

A **linear extension** of a poset $(S, \le_P)$ is a total order $(S, \le_T)$ that is compatible with $P$:
$$\forall x, y \in S, \quad x \le_P y \implies x \le_T y$$

Viewing the Hasse diagram as a Directed Acyclic Graph (DAG) with edges oriented upward ($x \to y$ if $x \prec y$), a linear extension is precisely a **valid topological sort** of that DAG.

**Worked example 5 (GATE 2024 Set 2 style counting linear extensions).** Let $S = \{1, 2, 3, 4, 5\}$ and let $P$ be the partial order defined by cover relations:
$$1 < 3, \quad 2 < 3, \quad 3 < 4, \quad 3 < 5$$
Find the number of total orders on $S$ that contain $P$.

1. *Analyze the precedence constraints:*
   - Both 1 and 2 must appear before 3.
   - 3 must appear before both 4 and 5.
2. *Determine valid positions:*
   - Elements $\{1, 2\}$ must occupy the first two positions in some order. Since they are incomparable, there are $2! = 2$ orderings: $(1, 2)$ or $(2, 1)$.
   - Element $3$ must occupy the third position (1 choice).
   - Elements $\{4, 5\}$ must occupy the remaining two positions. Since they are incomparable, there are $2! = 2$ orderings: $(4, 5)$ or $(5, 4)$.
3. *Total valid linear extensions:*
   $$2! \times 1 \times 2! = 2 \times 1 \times 2 = 4$$
   The 4 total orders are:
   1. $1 < 2 < 3 < 4 < 5$
   2. $1 < 2 < 3 < 5 < 4$
   3. $2 < 1 < 3 < 4 < 5$
   4. $2 < 1 < 3 < 5 < 4$
   *Answer:* **4**.

## Formulas and facts to memorise

| Item | Formula / condition | When to use |
|---|---|---|
| Partial order | Reflexive + Antisymmetric + Transitive | Checking whether a relation is a poset |
| Lattice definition | $\forall a, b: \exists unique\ (a \wedge b) \text{ and } (a \vee b)$ | Checking if a poset is a lattice |
| Divisor lattice operations | Meet $= \gcd(a, b)$, Join $= \text{lcm}(a, b)$ | Solving $D_n$ lattice questions |
| Non-distributive sublattices | Contains $M_3$ (diamond) or $N_5$ (pentagon) | Birkhoff's test for distributivity |
| Unique complement condition | Distributive lattice + complemented element | Proving complement uniqueness |
| Boolean lattice size | $\vert B\vert = 2^k$ for integer $k \ge 0$ | Eliminating candidate Boolean lattices |
| Divisor Boolean condition | $n = p_1 p_2 \dots p_k$ (square-free) | Checking if $D_n$ is a Boolean algebra |
| Dilworth's theorem | Min chain partition $=$ Max antichain size | Finding minimum chain covers |
| Linear extensions | Number of valid topological sorts of poset DAG | Counting total orders extending a poset |

## GATE traps

- **Confusing maximal with greatest:** A maximal element simply has nobody above it. A greatest element must be above *every* element in the set. A poset can have 5 maximal elements and 0 greatest elements.
- **Assuming all lattices have 0 and 1:** Every *finite* lattice has a least element 0 and greatest element 1. Infinite lattices may not (e.g., $(\mathbb{Z}, \le)$ is a lattice with meet $=\min$ and join $=\max$, but has neither 0 nor 1).
- **Non-uniqueness of complements:** In non-distributive lattices (like $M_3$), an element can have multiple distinct complements. Only distributivity guarantees uniqueness of complements.
- **Divisibility lattice misconception:** $D_n$ is *always* a distributive lattice for any positive integer $n$ (because $\gcd$ and $\text{lcm}$ distribute over each other). However, $D_n$ is *complemented* (and thus Boolean) if and only if $n$ is square-free.
- **Linear extensions order count:** Incomparable branches can be interleaved. If elements $x, y$ are incomparable to independent branch $z$, use multinomial combinations to interleave them validly.

## Connections

- [Sets, relations, and functions](sets-relations-functions.md) — Partial orders are antisymmetric transitive relations; equivalence relations partition sets.
- [Boolean algebra and minimization](../09-digital-logic/boolean-algebra-and-minimization.md) — Boolean lattices provide the formal algebraic foundation for logic circuits and switching functions.
- [Graph traversals](../08-algorithms/graph-traversals.md) — Finding linear extensions of a poset is algorithmically identical to topological sorting on directed acyclic graphs.

## Practice

**Q1 (MCQ, Easy).** Consider the divisibility poset $(D_{12}, \mid)$ where $D_{12} = \{1, 2, 3, 4, 6, 12\}$. Which of the following statements is TRUE?
(A) $D_{12}$ is not a lattice  
(B) $D_{12}$ is a lattice, but not distributive  
(C) $D_{12}$ is a distributive lattice, but not complemented  
(D) $D_{12}$ is a Boolean algebra  

<details><summary>Answer</summary>

**Answer:** C  
**Solution:**
For any positive integer $n$, the divisibility poset $D_n$ is always a distributive lattice because the operations $\gcd$ (meet) and $\text{lcm}$ (join) satisfy the distributive laws.
However, for $D_{12}$, $12 = 2^2 \times 3$, which is not square-free. Element 2 has no complement: for $x$ to be a complement of 2, we need $\gcd(2, x) = 1$ (so $x \in \{1, 3\}$), but $\text{lcm}(2, 1) = 2 \ne 12$ and $\text{lcm}(2, 3) = 6 \ne 12$. Because element 2 has no complement, $D_{12}$ is not complemented. Thus, it is distributive but not complemented.
</details>

**Q2 (MCQ, Easy).** In a bounded lattice, if an element has more than one complement, then the lattice CANNOT be:
(A) Modular  
(B) Complete  
(C) Distributive  
(D) Finite  

<details><summary>Answer</summary>

**Answer:** C  
**Solution:**
By theorem, in a distributive lattice, if an element has a complement, that complement is strictly unique. If any element has two or more distinct complements, the lattice cannot be distributive.
</details>

**Q3 (NAT, Medium).** Let $S = \{1, 2, 3, 4, 5, 6\}$ and let $P$ be a poset on $S$ whose Hasse diagram consists of two disjoint chains: $1 < 2 < 3$ and $4 < 5 < 6$. The number of linear extensions (total orders) of $P$ is ___________.

<details><summary>Answer</summary>

**Answer:** 20  
**Solution:**
A total order of $S$ has 6 positions. To respect the chain $1 < 2 < 3$, elements 1, 2, 3 must appear in that relative order among the 6 positions.
The number of ways to choose 3 positions out of 6 for the first chain is:
$$\binom{6}{3} = \frac{6 \times 5 \times 4}{3 \times 2 \times 1} = 20$$
Once these 3 positions are chosen, elements 1, 2, 3 can be placed in only 1 way (in increasing order). The remaining 3 positions must then be occupied by 4, 5, 6 in only 1 way (increasing order $4 < 5 < 6$).
Therefore, the number of linear extensions is $\binom{6}{3} = 20$.
</details>

**Q4 (MSQ, Medium).** Which of the following posets is/are lattices?
(A) $(\mathbb{Z}^+, \le)$ with ordinary less-than-or-equal-to  
(B) $(\mathcal{P}(\{a, b, c\}), \subseteq)$ ordered by set inclusion  
(C) Poset with elements $\{a, b, c, d\}$ and covers $a \prec c, a \prec d, b \prec c, b \prec d$  
(D) $(D_{24}, \mid)$ ordered by divisibility  

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:**
- (A) is a lattice: for any $x, y \in \mathbb{Z}^+$, $x \wedge y = \min(x, y)$ and $x \vee y = \max(x, y)$, both of which always exist and are unique.
- (B) is a lattice: for any subsets $X, Y$, $X \wedge Y = X \cap Y$ and $X \vee Y = X \cup Y$.
- (C) is NOT a lattice: for the pair $\{a, b\}$, common upper bounds are $\{c, d\}$. Since $c$ and $d$ are incomparable, neither is less than the other; hence there is no *least* upper bound.
- (D) is a lattice: for any $a, b \in D_{24}$, $a \wedge b = \gcd(a, b)$ and $a \vee b = \text{lcm}(a, b)$, both of which divide 24 and are in $D_{24}$.
</details>

**Q5 (NAT, Medium).** Consider the divisibility poset $D_{210}$. The number of elements in $D_{210}$ that have a unique complement is ___________.

<details><summary>Answer</summary>

**Answer:** 16  
**Solution:**
Prime factorisation: $210 = 2 \times 3 \times 5 \times 7$.
All prime factors are distinct, so 210 is square-free.
Therefore, $D_{210}$ is a Boolean algebra, isomorphic to $\mathcal{P}(\{2, 3, 5, 7\})$.
The number of elements in $D_{210}$ is $2^4 = 16$.
In any Boolean algebra, the lattice is distributive and complemented, so *every* element has a unique complement (specifically, the complement of $x$ is $210 / x$).
Thus, all 16 elements have a unique complement.
</details>

**Q6 (MCQ, Hard).** What is the minimum number of edges that must be removed from the diamond lattice $M_3$ to make it a distributive lattice while preserving all 5 elements as a lattice?
(A) 0  
(B) 1  
(C) 2  
(D) It is impossible without removing elements  

<details><summary>Answer</summary>

**Answer:** D  
**Solution:**
$M_3$ has 5 elements $\{0, a, b, c, 1\}$ with 6 cover edges: $0 \prec a, 0 \prec b, 0 \prec c$ and $a \prec 1, b \prec 1, c \prec 1$.
- Removing a bottom edge $0 \prec x$ kills the meet of $\{0, x\}$: the only candidate lower bound is $0$, but $0 \le x$ no longer holds, so no GLB exists.
- Removing a top edge $x \prec 1$ kills the join of $\{x, 1\}$: the only candidate upper bound is $1$, but $x \le 1$ no longer holds, so no LUB exists.
Every edge of $M_3$ is either some element's only bottom edge or some element's only top edge, so removing even one edge leaves a poset that is not a lattice at all; no number of edge removals can work.
A distributive lattice on a subset of these elements does exist: removing any one of the middle elements $a, b, c$ leaves $\{0, \text{two middle elements}, 1\}$, the 4-element Boolean diamond, which is distributive. But that requires deleting an element, which the question forbids. Hence the correct option is **(D)**.
</details>

**Q7 (NAT, Hard).** Let $P$ be a poset on $\{1, 2, 3, 4, 5, 6, 7\}$ defined by the relations:
$$1 < 2, \quad 1 < 3, \quad 2 < 4, \quad 3 < 4, \quad 4 < 5, \quad 4 < 6, \quad 5 < 7, \quad 6 < 7$$
The number of linear extensions (total orders) of $P$ is ___________.

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:**
Trace the precedence layers of the DAG:
1. $1$ is the unique minimal element and must be at position 1 (1 choice).
2. Elements $\{2, 3\}$ are both above $1$ and below $4$. They are incomparable to each other. They must occupy positions 2 and 3. Number of permutations $= 2! = 2$ ($(2, 3)$ or $(3, 2)$).
3. Element $4$ is above both 2 and 3, and below 5 and 6. It must occupy position 4 (1 choice).
4. Elements $\{5, 6\}$ are both above $4$ and below $7$. They are incomparable to each other. They must occupy positions 5 and 6. Number of permutations $= 2! = 2$ ($(5, 6)$ or $(6, 5)$).
5. Element $7$ is above both 5 and 6, and must occupy position 7 (1 choice).

By the multiplication rule:
$$\text{Total linear extensions} = 1 \times 2! \times 1 \times 2! \times 1 = 2 \times 2 = 4$$
</details>
