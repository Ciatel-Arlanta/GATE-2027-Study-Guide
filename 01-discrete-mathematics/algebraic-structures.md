# Algebraic structures: groups, monoids, and fields

> **Paper:** CS · **Priority:** P1 · **Plan topics:** binary operations, semigroups, monoids, groups, subgroups
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Posets and lattices](posets-and-lattices.md), [Number representation and arithmetic](../09-digital-logic/number-representation-and-arithmetic.md)
> **PYQ weightage:** Medium · 10 Qs since 2015 (5 since 2021) · DA: 0 · Trend: steady · Most asked: Groups & subgroups (Lagrange's theorem), Order of elements & cyclic groups, Binary operations

## Quick glance

- **Exam focus:** Checking properties of binary operations (associativity, identity, inverse), classifying algebraic structures (semigroup, monoid, group, abelian group), subgroups and Lagrange's theorem, orders of elements, cyclic group generators ($\phi(n)$), and ring/field axioms. Usually asked as 1-mark or 2-mark MCQ/MSQ/NAT questions.
- **Hierarchy:**
  - **Groupoid (Magma):** Closed under binary operation $*$.
  - **Semigroup:** Closed + Associative ($(a * b) * c = a * (b * c)$).
  - **Monoid:** Semigroup + Identity element $e$ ($a * e = e * a = a$).
  - **Group:** Monoid + Inverse $a^{-1}$ for every element ($a * a^{-1} = a^{-1} * a = e$).
  - **Abelian group:** Group + Commutative ($a * b = b * a$).
- **Lagrange's Theorem:** If $G$ is a finite group and $H$ is a subgroup of $G$, then $|H|$ divides $|G|$. Consequently, the order of any element $O(a)$ divides $|G|$.
- A group of **prime order $p$** has no non-trivial proper subgroups, is always **cyclic**, and is therefore **abelian**.
- Every cyclic group is abelian, but an abelian group is not necessarily cyclic (e.g., the Klein 4-group $V_4$).
- A finite cyclic group of order $n$ has exactly $\phi(n)$ generators, where $\phi$ is Euler's totient function.
- If $x^2 = e$ for all $x \in G$ (every element is its own inverse), then $G$ is **abelian**.
- The power set under symmetric difference $(\mathcal{P}(S), \Delta)$ forms an abelian group with identity $\emptyset$, and every element is its own inverse.
- A **field** is an algebraic structure $(F, +, \cdot)$ where $(F, +)$ is an abelian group, $(F \setminus \{0\}, \cdot)$ is an abelian group, and multiplication distributes over addition.

## 1. Binary operations and algebraic hierarchies [PYQ: Medium]

A **binary operation** on a non-empty set $S$ is a mapping $*: S \times S \to S$. That is, for all $a, b \in S$, the output $a * b \in S$ (closure is intrinsic to the definition).

### Algebraic axioms

| Axiom | Formal definition | Note / Counterexample |
|---|---|---|
| **Closure** | $\forall a, b \in S: a * b \in S$ | Subtraction on $\mathbb{N}$ is not closed ($2 - 5 = -3 \notin \mathbb{N}$) |
| **Associativity** | $\forall a, b, c \in S: (a * b) * c = a * (b * c)$ | Subtraction is not associative: $(8 - 4) - 2 = 2 \ne 8 - (4 - 2) = 6$ |
| **Commutativity** | $\forall a, b \in S: a * b = b * a$ | Matrix multiplication is not commutative ($AB \ne BA$) |
| **Identity** | $\exists e \in S, \forall a \in S: a * e = e * a = a$ | Identity is unique if it exists |
| **Inverse** | $\forall a \in S, \exists a^{-1} \in S: a * a^{-1} = a^{-1} * a = e$ | Inverses are unique in a monoid |
| **Distributivity** | $a * (b \circ c) = (a * b) \circ (a * c)$ | Operation $*$ distributes over $\circ$ |

### The structural hierarchy

```text
    Algebraic Structure Hierarchy:
    [Set + Closed Operation]             --> Groupoid (Magma)
            + Associativity              --> Semigroup
            + Identity                   --> Monoid
            + Inverses for all elements  --> Group
            + Commutativity              --> Abelian (Commutative) Group
```

**Worked example 1 (custom binary operation).** Let $*$ be defined on $\mathbb{R} \setminus \{1\}$ by:
$$a * b = a + b - ab$$
Determine whether $(\mathbb{R} \setminus \{1\}, *)$ is an abelian group.

1. *Closure:* If $a \ne 1$ and $b \ne 1$, could $a * b = 1$?
   $$a + b - ab = 1 \implies 1 - a - b + ab = 0 \implies (1 - a)(1 - b) = 0$$
   Since $a \ne 1$ and $b \ne 1$, this product is non-zero, so $a * b \ne 1$. Closure holds.
2. *Associativity:*
   $$(a * b) * c = (a + b - ab) * c = (a + b - ab) + c - (a + b - ab)c = a + b + c - ab - ac - bc + abc$$
   By symmetry, $a * (b * c) = a + b + c - ab - bc - ac + abc$. Associativity holds.
3. *Identity:*
   $$a * e = a \implies a + e - ae = a \implies e(1 - a) = 0$$
   Since $a \ne 1$, $1 - a \ne 0 \implies e = 0$. Note $0 \in \mathbb{R} \setminus \{1\}$. Identity is $e = 0$.
4. *Inverse:*
   $$a * b = 0 \implies a + b - ab = 0 \implies b(1 - a) = -a \implies b = \frac{-a}{1 - a} = \frac{a}{a - 1}$$
   Since $a \ne 1$, this $b$ is well-defined and $b \ne 1$ (since $a \ne a - 1$). Every element has an inverse.
5. *Commutativity:* $a * b = a + b - ab = b + a - ba = b * a$.
Conclusion: $(\mathbb{R} \setminus \{1\}, *)$ is an **abelian group**.

## 2. Groups, abelian groups, and cancellation laws [PYQ: High]

A structure $(G, *)$ is a **group** if it satisfies:
1. Closure: $\forall a, b \in G, a * b \in G$.
2. Associativity: $\forall a, b, c \in G, (a * b) * c = a * (b * c)$.
3. Identity: $\exists e \in G, \forall a \in G, a * e = e * a = a$.
4. Inverses: $\forall a \in G, \exists a^{-1} \in G, a * a^{-1} = a^{-1} * a = e$.

If additionally $a * b = b * a$ for all $a, b \in G$, $G$ is an **abelian group**.

### Fundamental properties
- **Cancellation laws:** In any group $G$:
  - Left cancellation: $a * b = a * c \implies b = c$
  - Right cancellation: $b * a = c * a \implies b = c$
- **Reversal rule for inverses:** $(a * b)^{-1} = b^{-1} * a^{-1}$.
- **Uniqueness:** The identity element is unique; the inverse of each element is unique; and $(a^{-1})^{-1} = a$.

### Self-inverse elements and commutativity (GATE 2022)
**Theorem:** If every element in a group $G$ is its own inverse (i.e., $x^2 = e$ for all $x \in G$), then $G$ must be **abelian**.

*Proof:*
Let $a, b \in G$. By assumption, $a = a^{-1}$, $b = b^{-1}$, and $(a * b) = (a * b)^{-1}$.
Using the reversal rule:
$$a * b = (a * b)^{-1} = b^{-1} * a^{-1} = b * a$$
Thus $a * b = b * a$, so $G$ is abelian.

### Symmetric difference group on power set (GATE 2023)
Let $S$ be any set and $\mathcal{P}(S)$ its power set. Define the operation as symmetric difference:
$$A \Delta B = (A \setminus B) \cup (B \setminus A) = (A \cup B) \setminus (A \cap B)$$
Then $(\mathcal{P}(S), \Delta)$ forms an abelian group:
- **Identity:** The empty set $\emptyset$, since $A \Delta \emptyset = A$.
- **Inverse:** Every subset is its own inverse, since $A \Delta A = \emptyset$.
- Therefore, $(\mathcal{P}(S), \Delta)$ is an abelian group where every non-identity element has order 2.

## 3. Subgroups and Lagrange's theorem [PYQ: High]

A non-empty subset $H \subseteq G$ is a **subgroup** of $G$ (written $H \le G$) if $H$ itself forms a group under the operation of $G$.

### Subgroup tests
1. **One-step test:** $H \le G \iff H \ne \emptyset$ and $\forall a, b \in H: a * b^{-1} \in H$.
2. **Two-step test:** $H \le G \iff H \ne \emptyset$, closed under operation ($\forall a, b \in H: ab \in H$), and closed under inverse ($\forall a \in H: a^{-1} \in H$).
3. **Finite subgroup test:** If $H$ is a non-empty **finite** subset of $G$, then $H \le G \iff H$ is closed under the group operation (inverses are guaranteed automatically).

### Cosets and Lagrange's Theorem
For a subgroup $H \le G$ and element $a \in G$:
- **Left coset:** $aH = \{a * h \mid h \in H\}$.
- **Right coset:** $Ha = \{h * a \mid h \in H\}$.

Coset properties:
- The cosets of $H$ form a **partition** of $G$: every element belongs to exactly one left coset.
- Every coset has the exact same size as $H$: $|aH| = |H|$.

**Lagrange's Theorem:**
If $G$ is a finite group and $H$ is a subgroup of $G$, then the order of $H$ divides the order of $G$:
$$|G| = [G : H] \times |H|$$
where $[G : H]$ is the index of $H$ in $G$ (the number of distinct cosets).

**Immediate corollaries (heavily tested in GATE):**
1. The order of any element $a \in G$ divides the order of the group: $O(a) \mid |G|$.
2. For every element $a \in G$: $a^{|G|} = e$.
3. If $|G| = p$ where $p$ is a prime number, then $G$ has only two subgroups: $\{e\}$ and $G$. Furthermore, $G$ is **cyclic** and **abelian**.

**Worked example 2 (GATE 2020).** Let $G$ be a group of 35 elements. Then the largest possible size of a subgroup of $G$ other than $G$ itself is ___________.

1. The order of the group is $|G| = 35$.
2. By Lagrange's theorem, the order of any subgroup $H$ must divide $|G| = 35$.
3. The positive divisors of 35 are $1, 5, 7, 35$.
4. A proper subgroup other than $G$ itself must have order strictly less than 35.
5. The divisors strictly less than 35 are $\{1, 5, 7\}$. The largest among these is **7**.
*Answer:* **7**.

## 4. Order of elements, cyclic groups, and generators [PYQ: High]

### Order of an element
The **order** of an element $a \in G$, denoted $O(a)$, is the smallest positive integer $k$ such that:
$$a^k = \underbrace{a * a * \dots * a}_{k \text{ times}} = e$$
If no such positive integer exists, $a$ has infinite order.

**Crucial rule:** If $a^m = e$, then $O(a)$ divides $m$.

### Cyclic groups
A group $G$ is **cyclic** if there exists an element $g \in G$ such that every element of $G$ can be written as a power of $g$:
$$G = \langle g \rangle = \{g^k \mid k \in \mathbb{Z}\}$$
The element $g$ is called a **generator** of $G$.

### Key properties of cyclic groups
1. **Every cyclic group is abelian** ($g^i * g^j = g^{i+j} = g^j * g^i$).
2. Every subgroup of a cyclic group is **cyclic**.
3. If $G = \langle g \rangle$ is a finite cyclic group of order $n$:
   - The order of $g^k$ is:
     $$O(g^k) = \frac{n}{\gcd(k, n)}$$
   - An element $g^k$ is a generator of $G$ if and only if $\gcd(k, n) = 1$.
   - **Number of generators:** Exactly $\phi(n)$, where $\phi$ is Euler's totient function.
   - For every divisor $d$ of $n$, there exists **exactly one** subgroup of order $d$.

**Worked example 3 (generators and orders in $\mathbb{Z}_{12}$).**
Consider the group $(\mathbb{Z}_{12}, +_{12})$:
1. $|G| = 12$. The group is cyclic with generator $1$.
2. Number of generators = $\phi(12) = 12 \times \left(1 - \frac{1}{2}\right) \times \left(1 - \frac{1}{3}\right) = 12 \times \frac{1}{2} \times \frac{2}{3} = 4$.
3. The generators are the elements coprime to 12: $\{1, 5, 7, 11\}$.
4. Orders of elements:
   - $O(0) = 1$
   - $O(6) = 12 / \gcd(6, 12) = 12 / 6 = 2$
   - $O(4) = O(8) = 12 / \gcd(4, 12) = 3$
   - $O(3) = O(9) = 12 / \gcd(3, 12) = 4$
   - $O(2) = O(10) = 12 / \gcd(2, 12) = 6$
   - $O(1) = O(5) = O(7) = O(11) = 12$

### Self-inverse elements in $\mathbb{Z}_n$ (GATE 2024 Set 2)
In the additive group $(\mathbb{Z}_n, +_n)$, an element $x$ is its own inverse iff:
$$x + x \equiv 0 \pmod n \iff 2x \equiv 0 \pmod n$$
- If $n$ is **odd**: $\gcd(2, n) = 1 \implies x \equiv 0 \pmod n$. Only $x = 0$ is self-inverse (exactly **1** element).
- If $n$ is **even**: $2x$ is a multiple of $n \implies x = 0$ or $x = n/2$ (exactly **2** elements).

## 5. Permutations and symmetric groups [PYQ: Low]

The set of all bijections from a set of $n$ elements to itself under function composition forms the **symmetric group** $S_n$.
- Order of the symmetric group: $|S_n| = n!$.
- $S_n$ is non-abelian for all $n \ge 3$. (Smallest non-abelian group is $S_3$ of order $3! = 6$).
- Every permutation can be written as a product of **disjoint cycles**.
- **Order of a permutation:** The order of $\sigma \in S_n$ is the **least common multiple (lcm)** of the lengths of its disjoint cycles:
  $$O(\sigma) = \text{lcm}(L_1, L_2, \dots, L_k)$$

**Worked example 4 (order of a permutation).**
Find the order of $\sigma = (1\ 2\ 3)(4\ 5)$ in $S_5$:
- The cycles $(1\ 2\ 3)$ and $(4\ 5)$ are disjoint, with lengths 3 and 2.
- $O(\sigma) = \text{lcm}(3, 2) = 6$.

## 6. Rings, integral domains, and fields [PYQ: Low]

### Rings
A system $(R, +, \cdot)$ with two binary operations is a **ring** if:
1. $(R, +)$ is an **abelian group** (identity 0, additive inverse $-a$).
2. $(R, \cdot)$ is a **semigroup** (closed and associative).
3. Multiplication distributes over addition: $a(b + c) = ab + ac$ and $(b + c)a = ba + ca$.

### Types of rings
- **Ring with unity:** Has a multiplicative identity $1$ such that $1 \cdot a = a \cdot 1 = a$ ($1 \ne 0$).
- **Commutative ring:** Multiplication is commutative ($a \cdot b = b \cdot a$).
- **Zero divisor:** A non-zero element $a \in R$ such that $\exists b \ne 0 \in R$ with $a \cdot b = 0$. (E.g., in $\mathbb{Z}_6$, $2 \cdot 3 = 0$).
- **Integral domain:** A commutative ring with unity and **no zero divisors** ($a \cdot b = 0 \implies a = 0 \lor b = 0$).

### Fields
A **field** $(F, +, \cdot)$ is a commutative ring with unity in which **every non-zero element has a multiplicative inverse**:
1. $(F, +)$ is an abelian group with identity 0.
2. $(F \setminus \{0\}, \cdot)$ is an abelian group with identity 1.
3. Distributive laws hold.

### Essential facts for GATE
- Every field is an integral domain (having inverses eliminates zero divisors: if $ab = 0$ and $a \ne 0$, multiply by $a^{-1} \implies b = 0$).
- **Wedderburn's / Finite Integral Domain Theorem:** Every **finite** integral domain is a field.
- $(\mathbb{Z}_n, +_n, \times_n)$ is a field if and only if $n$ is a **prime number**.
- The size of any finite field must be a prime power: $|F| = p^k$ where $p$ is prime.

| Number System | Semigroup? | Monoid? | Group? | Abelian? | Ring? | Integral Domain? | Field? |
|---|---|---|---|---|---|---|---|
| $(\mathbb{N}, +)$ | Yes | Yes (with 0) | No | - | - | - | - |
| $(\mathbb{Z}, +, \cdot)$ | Yes | Yes | Yes (+) | Yes (+) | Yes | Yes | No (2 has no mult. inverse) |
| $(\mathbb{Q}, +, \cdot)$ | Yes | Yes | Yes (+) | Yes (+) | Yes | Yes | Yes |
| $(\mathbb{R}, +, \cdot)$ | Yes | Yes | Yes (+) | Yes (+) | Yes | Yes | Yes |
| $(\mathbb{Z}_p, +_p, \times_p)$ ($p$ prime) | Yes | Yes | Yes (+) | Yes (+) | Yes | Yes | Yes |
| $(\mathbb{Z}_n, +_n, \times_n)$ ($n$ composite) | Yes | Yes | Yes (+) | Yes (+) | Yes | No (has zero divisors) | No |

## Formulas and facts to memorise

| Item | Formula / fact | When to use |
|---|---|---|
| Group axioms | Closure, Associativity, Identity, Inverses | Proving a structure is a group |
| Self-inverse group | $\forall x: x^2 = e \implies G \text{ is abelian}$ | Proving commutativity from involutions |
| Lagrange's Theorem | $\vert H\vert \text{ divides } \vert G\vert \text{ and } O(a) \text{ divides } \vert G\vert$ | Finding possible subgroup and element orders |
| Prime order group | $\vert G\vert = p \implies G \cong \mathbb{Z}_p$ (cyclic & abelian) | Any group of prime order |
| Number of generators | $\phi(n) = n \prod_{p \mid n} (1 - 1/p)$ | Counting generators of cyclic group of order $n$ |
| Order of power $g^k$ | $O(g^k) = \frac{n}{\gcd(k, n)}$ | Computing order of element in cyclic group |
| Self-inverses in $\mathbb{Z}_n$ | 1 element ($x=0$) if $n$ odd; 2 elements ($x=0, n/2$) if $n$ even | GATE NAT questions on self-inverse counts |
| Permutation order | $\text{lcm}(\text{cycle lengths})$ | Finding order of element in $S_n$ |
| Finite field condition | Order must be $p^k$ ($p$ prime); $\mathbb{Z}_n$ field $\iff n$ prime | Testing field existence |

## GATE traps

- **Converse of Lagrange's Theorem is FALSE:** If $d$ divides $|G|$, there is *not necessarily* a subgroup of order $d$ in $G$. (For instance, the alternating group $A_4$ has order 12, but has no subgroup of order 6).
- **Abelian does NOT imply cyclic:** Every cyclic group is abelian, but abelian groups need not be cyclic. The Klein 4-group $V_4 \cong \mathbb{Z}_2 \times \mathbb{Z}_2$ is abelian of order 4, but has no element of order 4.
- **Order $p^2$ groups:** A group of order $p^2$ (where $p$ is prime) is always abelian, but it may not be cyclic (it is isomorphic to either $\mathbb{Z}_{p^2}$ or $\mathbb{Z}_p \times \mathbb{Z}_p$).
- **Reversal of inverse products:** $(ab)^{-1} = b^{-1} a^{-1}$, NOT $a^{-1} b^{-1}$ in non-abelian groups.
- **Zero in multiplicative groups:** $(\mathbb{R}, \cdot)$ is NOT a group because 0 has no multiplicative inverse. The set must exclude zero: $(\mathbb{R} \setminus \{0\}, \cdot)$.
- **Finite integral domain shortcut:** If asked whether a finite commutative ring with unity and no zero divisors is a field, the answer is always YES.

## Connections

- [Sets, relations, and functions](sets-relations-functions.md) — Equivalence relations and partitions directly formalize coset decomposition and quotient groups.
- [Posets and lattices](posets-and-lattices.md) — Subgroup lattices of finite groups form bounded lattices ordered by inclusion.
- [Number representation and arithmetic](../09-digital-logic/number-representation-and-arithmetic.md) — Modular arithmetic in ALU design relies on residue groups $(\mathbb{Z}_n, +_n)$ and $(\mathbb{Z}_2^n, \oplus)$.

## Practice

**Q1 (MCQ, Easy).** On the set of real numbers $\mathbb{R}$, define the binary operation $*$ by $a * b = a + b + 2ab$. The identity element with respect to $*$ is:
(A) 0  
(B) 1  
(C) $-1/2$  
(D) Does not exist  

<details><summary>Answer</summary>

**Answer:** A  
**Solution:**
Let $e$ be the identity element. Then for all $a \in \mathbb{R}$:
$$a * e = a \implies a + e + 2ae = a$$
$$e(1 + 2a) = 0$$
For this equation to hold for *all* $a \in \mathbb{R}$, we must have $e = 0$.
Check on the left: $0 * a = 0 + a + 2(0)(a) = a$.
Thus, the identity element is 0.
</details>

**Q2 (NAT, Easy).** Let $G$ be a group of order 39. The maximum possible order of a proper subgroup of $G$ is ___________.

<details><summary>Answer</summary>

**Answer:** 13  
**Solution:**
By Lagrange's theorem, the order of any subgroup $H$ must divide the order of $G = 39$.
The positive divisors of 39 are 1, 3, 13, and 39.
A proper subgroup must have order strictly less than 39.
The divisors strictly less than 39 are $\{1, 3, 13\}$.
The maximum among these is 13.
</details>

**Q3 (MCQ, Medium).** How many generators does the cyclic group $(\mathbb{Z}_{10}, +_{10})$ have?
(A) 2  
(B) 4  
(C) 5  
(D) 9  

<details><summary>Answer</summary>

**Answer:** B  
**Solution:**
For any finite cyclic group of order $n$, the number of generators is given by Euler's totient function $\phi(n)$.
Here $n = 10 = 2 \times 5$:
$$\phi(10) = 10 \times \left(1 - \frac{1}{2}\right) \times \left(1 - \frac{1}{5}\right) = 10 \times \frac{1}{2} \times \frac{4}{5} = 4$$
The 4 generators are the elements coprime to 10: $\{1, 3, 7, 9\}$.
</details>

**Q4 (MSQ, Medium).** Let $S$ be a non-empty set and $\mathcal{P}(S)$ be its power set. Let $\Delta$ denote the symmetric difference operator defined as $A \Delta B = (A \setminus B) \cup (B \setminus A)$. Which of the following statements is/are TRUE?
(A) $(\mathcal{P}(S), \Delta)$ is an abelian group  
(B) The identity element is $S$  
(C) Every element is its own inverse  
(D) If $|S| = 3$, the order of the group is 8  

<details><summary>Answer</summary>

**Answer:** A, C, D  
**Solution:**
- (A) is TRUE: Symmetric difference is associative, commutative, has an identity ($\emptyset$), and every element has an inverse. Thus it is an abelian group.
- (B) is FALSE: The identity element is $\emptyset$, since $A \Delta \emptyset = A$. ($A \Delta S = S \setminus A = A^c \ne A$).
- (C) is TRUE: $A \Delta A = (A \setminus A) \cup (A \setminus A) = \emptyset \cup \emptyset = \emptyset$, so every subset is its own inverse.
- (D) is TRUE: The elements of the group are the subsets of $S$. If $|S| = 3$, $|\mathcal{P}(S)| = 2^3 = 8$.
</details>

**Q5 (NAT, Medium).** Let $G$ be the group of integers $\{0, 1, 2, \dots, 11\}$ with addition modulo 12 as the group operation. The number of elements in $G$ that are their own inverses is ___________.

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:**
An element $x \in \mathbb{Z}_{12}$ is its own inverse iff:
$$x + x \equiv 0 \pmod{12} \iff 2x \equiv 0 \pmod{12}$$
This requires $2x$ to be a multiple of 12 for $x \in \{0, 1, \dots, 11\}$:
- $x = 0 \implies 2(0) = 0 \equiv 0 \pmod{12}$ ✓
- $x = 6 \implies 2(6) = 12 \equiv 0 \pmod{12}$ ✓
For any other $x$, $2x$ is not a multiple of 12.
Therefore, exactly 2 elements ($0$ and $6$) are their own inverses.
</details>

**Q6 (MSQ, Hard).** Which of the following statements is/are TRUE for any group $G$?
(A) If $x^2 = e$ for all $x \in G$, then $G$ is commutative  
(B) If $G$ has order 4, then $G$ must be commutative  
(C) If every proper subgroup of $G$ is cyclic, then $G$ must be cyclic  
(D) If $G$ is abelian, then every subgroup of $G$ is abelian  

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:**
- (A) is TRUE: $(xy)^2 = e \implies xyxy = e \implies x(xyxy)y = xey \implies x^2 y x y^2 = xy \implies yx = xy$.
- (B) is TRUE: Any group of order $p^2$ (here $2^2 = 4$) is abelian. Up to isomorphism, the only groups of order 4 are $\mathbb{Z}_4$ (cyclic, hence abelian) and the Klein 4-group $V_4 \cong \mathbb{Z}_2 \times \mathbb{Z}_2$ (abelian).
- (C) is FALSE: Consider the non-abelian symmetric group $S_3$ of order 6. Its proper subgroups have orders 1, 2, and 3. Any group of order 1, 2, or 3 is prime order and hence cyclic. Thus every proper subgroup of $S_3$ is cyclic, but $S_3$ itself is not cyclic (not even abelian!).
- (D) is TRUE: If the operation is commutative for all elements in $G$, it is automatically commutative for any subset of elements.
</details>

**Q7 (NAT, Hard).** The maximum possible order of any element in the symmetric group $S_7$ is ___________.

<details><summary>Answer</summary>

**Answer:** 12  
**Solution:**
Any permutation in $S_7$ can be decomposed into disjoint cycles whose lengths partition 7.
The order of a permutation is the least common multiple ($\text{lcm}$) of its cycle lengths.
Consider all partitions of 7 and their corresponding $\text{lcm}$:
- $7 \implies \text{lcm}(7) = 7$
- $6 + 1 \implies \text{lcm}(6, 1) = 6$
- $5 + 2 \implies \text{lcm}(5, 2) = 10$
- $5 + 1 + 1 \implies \text{lcm}(5, 1, 1) = 5$
- $4 + 3 \implies \text{lcm}(4, 3) = 12$
- $4 + 2 + 1 \implies \text{lcm}(4, 2, 1) = 4$
- $3 + 3 + 1 \implies \text{lcm}(3, 3, 1) = 3$
- $3 + 2 + 2 \implies \text{lcm}(3, 2, 2) = 6$
- $3 + 2 + 1 + 1 \implies \text{lcm}(3, 2, 1, 1) = 6$
- $2 + 2 + 2 + 1 \implies \text{lcm}(2, 2, 2, 1) = 2$

The maximum $\text{lcm}$ obtained is 12, achieved by a permutation having disjoint cycles of lengths 4 and 3 (for example, $\sigma = (1\ 2\ 3\ 4)(5\ 6\ 7)$).
Therefore, the maximum possible order of an element in $S_7$ is 12.
</details>
