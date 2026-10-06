# Vector Spaces, Independence, Basis and Rank

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Vectors and vector spaces; Subspaces; Linear dependence and independence; Rank; Nullity
> **Prerequisites:** [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) · **Leads to:** [Matrices and determinants](matrices-and-determinants.md) · [Linear systems and LU](linear-systems-and-lu.md)

## Quick glance
- A **vector space** over R is a set closed under addition and scalar multiplication (8 axioms). Examples: R^n, matrices of a fixed size, polynomials of degree ≤ d.
- **Subspace test:** non-empty, closed under addition, closed under scalar multiplication. Equivalent: contains the zero vector and closed under linear combinations. **A subspace must contain 0.**
- **Span** = all linear combinations. **Basis** = linearly independent spanning set. **Dimension** = number of vectors in any basis (all bases have the same size).
- Vectors v1..vk are **independent** iff c1 v1 + ... + ck vk = 0 forces all ci = 0, i.e. the matrix [v1 ... vk] has rank k. More than n vectors in R^n are always dependent.
- **Rank** = number of pivots in row echelon form = dim(column space) = dim(row space). **Nullity** = dim(null space) = number of free variables.
- **Rank-nullity:** rank(A) + nullity(A) = n (number of columns).
- Rank facts: rank(A) = rank(A^T); rank(AB) ≤ min(rank A, rank B); rank(A^T A) = rank(A) over R; rank(A + B) ≤ rank A + rank B.
- **#1 trap:** pivot *columns of the original matrix* (not of the echelon form) are the basis of the column space; the nonzero rows *of the echelon form* are the basis of the row space.

## 1. Vectors and vector spaces

**Intuition.** A vector space is any collection of objects you can add together and stretch/shrink, with the usual algebra working. Arrows in the plane, n-tuples of reals, m × n matrices and polynomials all qualify.

**Definition.** A set V with operations + and scalar multiplication (scalars from R) is a vector space if for all u, v, w in V and scalars a, b:

| # | Axiom | # | Axiom |
|---|---|---|---|
| 1 | u + v is in V (closure) | 5 | there is 0 with v + 0 = v |
| 2 | u + v = v + u | 6 | each v has -v with v + (-v) = 0 |
| 3 | (u + v) + w = u + (v + w) | 7 | a v is in V (closure) |
| 4 | a(u + v) = a u + a v | 8 | (a + b) v = a v + b v, (ab) v = a (b v), 1 v = v |

**Standard examples:** R^n (dimension n); M(m × n) (dimension mn); P_d = polynomials of degree ≤ d (dimension d + 1, basis 1, x, ..., x^d); the solutions of a homogeneous linear system Ax = 0.

**Non-examples:** the set of vectors with positive entries (no negatives, no 0); polynomials of degree *exactly* d (the sum of x^2 and -x^2 + 1 has degree 0).

**GATE-level notes.** Vectors are columns by convention; the dot product is u·v = u^T v; length ||v|| = sqrt(v·v). "Vector" in GATE almost always means an element of R^n.

## 2. Subspaces

**Definition.** W ⊆ V is a subspace if W is itself a vector space under the same operations. **Subspace test:** (i) 0 is in W (equivalently W is non-empty), (ii) u, v in W implies u + v in W, (iii) v in W, c scalar implies c v in W.

**Worked example 1 — apply the test to subsets of R^3 / matrices.**

| Set | Subspace? | Reason |
|---|---|---|
| {(x,y,z) : x + y + z = 0} | **Yes** | contains 0; sum of two vectors with coordinate-sum 0 has sum 0; scalar multiple keeps 0. A plane through the origin, dim 2. |
| {(x,y,z) : x + y + z = 1} | No | 0 not in it (also (1,0,0)+(0,1,0) has sum 2). |
| {(x,y) : xy = 0} (the two axes) | No | (1,0) + (0,1) = (1,1), product 1 ≠ 0. Union of subspaces is generally not a subspace. |
| {(x,y) : y ≥ 0} | No | not closed under scalar -1. |
| {(x,y,z) : x = 2z} | Yes | a plane through 0, dim 2. |
| {(x,y,z) : x² = y²} | No | (1,1,0) + (1,-1,0) = (2,0,0) violates x² = y². |
| Symmetric 3 × 3 matrices | Yes | dim 6. |
| Upper triangular n × n matrices | Yes | dim n(n+1)/2. |
| Invertible 2 × 2 matrices | No | 0 matrix is not invertible; I + (-I) = 0. |
| Matrices with trace 0 (n × n) | Yes | dim n² - 1. |
| Matrices with det = 0 (n × n) | No | [[1,0],[0,0]] + [[0,0],[0,1]] = I has det 1. |
| Solutions of Ax = b, b ≠ 0 | No | 0 is not a solution. It is a *translate* (affine set) of a subspace. |

**Key fact.** The intersection of subspaces is a subspace; the union is **not** (unless one contains the other). The **sum** U + W = {u + w} is a subspace with
dim(U + W) = dim U + dim W - dim(U ∩ W).

**Worked example 2.** In R^4, U = {x : x1 + x2 + x3 + x4 = 0} (dim 3) and W = {x : x1 = x2} (dim 3). U ∩ W is given by two independent equations, so dim 2. Then dim(U + W) = 3 + 3 - 2 = 4, so U + W = R^4.

## 3. Span, linear independence

**Span.** span{v1, ..., vk} = {c1 v1 + ... + ck vk}. It is the smallest subspace containing the vectors. Whether b lies in the span of the columns of A is the same as whether Ax = b is solvable.

**Linear independence.** v1..vk are independent iff c1 v1 + ... + ck vk = 0 has **only** the trivial solution. Otherwise one vector is a combination of the others (dependent).

**How to test (always the same):** put the vectors as the **columns** of a matrix A and compute the rank. Independent iff rank(A) = k. For n vectors in R^n: independent iff det ≠ 0.

**Quick rules:**
- Any set containing the zero vector is dependent.
- Two vectors are dependent iff one is a multiple of the other.
- More than n vectors in R^n are dependent. (n+1 vectors, only n coordinates.)
- Eigenvectors for distinct eigenvalues are independent.
- A subset of an independent set is independent; a superset of a dependent set is dependent.

**Worked example 3.** Are v1 = (1,2,3), v2 = (2,3,4), v3 = (3,5,7) independent?
1. Notice v1 + v2 = (3,5,7) = v3, so v1 + v2 - v3 = 0: **dependent**.
2. Confirm by rank: columns [v1 v2 v3] = [[1,2,3],[2,3,4],[3,5,7]]. R2 - 2R1 = (0,-1,-2), R3 - 3R1 = (0,-1,-2). R3 - R2 = 0. Two pivots, **rank 2 < 3**.
3. They span a plane (dim 2); a basis of the span is {v1, v2}.

**Worked example 4 — parameter.** For which k are the rows (1,1,1), (1,2,4), (1,k,k²) independent?
This is a Vandermonde matrix with nodes 1, 2, k: det = (2 - 1)(k - 1)(k - 2). So independent iff k ≠ 1 and k ≠ 2. At k = 1 or 2 two rows coincide and rank = 2.

## 4. Basis and dimension

**Definition.** A **basis** of V is a list of vectors that is linearly independent **and** spans V. Every vector then has a **unique** coordinate representation.

**Dimension** = size of a basis (well defined). Consequences, for dim V = n:
- any n independent vectors form a basis;
- any n spanning vectors form a basis;
- any independent set can be extended to a basis; any spanning set can be shrunk to a basis;
- a subspace W of V has dim W ≤ n, with equality iff W = V.

| Space | Dimension | Standard basis |
|---|---|---|
| R^n | n | e1, ..., en |
| m × n matrices | mn | matrices with a single 1 |
| P_d (degree ≤ d) | d + 1 | 1, x, ..., x^d |
| Symmetric n × n | n(n+1)/2 | E_ii and E_ij + E_ji |
| Skew-symmetric n × n | n(n-1)/2 | E_ij - E_ji (i < j) |
| {0} | 0 | empty set |

**Worked example 5 — coordinates.** In R^2, basis B = {(1,1), (1,-1)}. Write (5,1) in this basis: a(1,1) + b(1,-1) = (5,1) gives a + b = 5, a - b = 1, so a = 3, b = 2. Coordinates [3, 2].

## 5. Rank and the four fundamental subspaces

For an m × n matrix A:

| Subspace | Lives in | Definition | Dimension | Basis from |
|---|---|---|---|---|
| Column space C(A) | R^m | span of columns = {Ax} | r | **pivot columns of the original A** |
| Row space C(A^T) | R^n | span of rows | r | nonzero rows of the echelon form |
| Null space N(A) | R^n | {x : Ax = 0} | n - r | one vector per free variable |
| Left null space N(A^T) | R^m | {y : A^T y = 0} | m - r | from echelon of A^T |

Orthogonality: **row space ⊥ null space** (in R^n) and **column space ⊥ left null space** (in R^m); the two pairs are orthogonal complements. Rank r = number of pivots.

**Rank-nullity theorem:** rank(A) + nullity(A) = n.

**Worked example 6 — all four subspaces.** Let

A = [[1,2,1,3],[2,4,0,2],[3,6,1,5],[1,2,-1,-1]] (4 × 4).

Row reduce: R2 - 2R1 = (0,0,-2,-4); R3 - 3R1 = (0,0,-2,-4); R4 - R1 = (0,0,-2,-4). Then R3 - R2 = 0 and R4 - R2 = 0, and scaling R2 by -1/2 gives (0,0,1,2). Reduced echelon form:

```text
[1 2 0 1]
[0 0 1 2]
[0 0 0 0]
[0 0 0 0]
```

- Pivots in columns 1 and 3, so **rank r = 2**; nullity = 4 - 2 = 2.
- **Column space basis** (original columns 1 and 3): (1,2,3,1) and (1,0,1,-1). Dimension 2 in R^4.
- **Row space basis:** (1,2,0,1) and (0,0,1,2).
- **Null space:** free variables x2 = s, x4 = t. Then x3 = -2t, x1 = -2s - t. So x = s(-2,1,0,0) + t(-1,0,-2,1). Check: row 1 on (-1,0,-2,1): -1 + 0 - 2 + 3 = 0.
- **Left null space** (dim 4 - 2 = 2): y with y^T A = 0, e.g. (-1,-1,1,0) (row 3 = row 1 + row 2) and (1,-1,0,1) (row 4 = row 2 - row 1).
- Dimensions: 2 + 2 = 4 = n and 2 + 2 = 4 = m. 

**Computing rank — procedure.** Reduce to row echelon form using row operations (swap, scale, add a multiple of one row to another); count nonzero rows. Row operations do not change rank, row space or null space. They **do** change the column space (but not which columns are pivot columns, nor the dependencies among columns).

**Maximum possible rank** of an m × n matrix is min(m, n). A square matrix has full rank n iff det ≠ 0 iff invertible iff Ax = 0 has only x = 0.

## 6. Rank facts

| Fact | Statement | Note |
|---|---|---|
| Transpose | rank(A) = rank(A^T) | row rank = column rank |
| Product | rank(AB) ≤ min(rank A, rank B) | multiplying cannot raise rank |
| Sylvester | rank(AB) ≥ rank A + rank B - n | A is m × n, B is n × p |
| Sum | rank(A + B) ≤ rank A + rank B | |
| Gram | rank(A^T A) = rank(A A^T) = rank(A) | **real** matrices only |
| Invertible factor | rank(PA) = rank(A) if P invertible | |
| Rank 1 | rank(u v^T) = 1 for nonzero u, v | all rows multiples of v^T |
| Rank 0 | rank(A) = 0 iff A = 0 | |
| Idempotent | rank(A) = trace(A) | see [matrices](matrices-and-determinants.md) |
| Adjugate | rank(adj A) = n, 1, or 0 when rank A = n, n-1, or < n-1 | |

**Worked example 7.** A is 3 × 4 with rank 2, B is 4 × 3 with rank 3. What are the possible ranks of AB (3 × 3)? Upper bound min(2, 3) = 2. Sylvester: ≥ 2 + 3 - 4 = 1. So rank(AB) is 1 or 2. In particular AB is singular (det = 0), since rank < 3.

**Worked example 8.** Nullity of A^T A when A is 5 × 3 with rank 3. A^T A is 3 × 3 with rank 3, so nullity 0 and A^T A is invertible. (This is why the least-squares normal equations work for full-column-rank data; see [projections](orthogonality-projections-svd.md).)

## Linear transformations in one paragraph

A map T: R^n → R^m is linear if T(au + bv) = aT(u) + bT(v); every such map is x → Ax for a unique m × n matrix. **Kernel** = N(A), **image** = C(A), and rank-nullity reads dim(ker) + dim(image) = n. T is one-to-one iff kernel = {0} iff the columns are independent; T is onto iff rank = m.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Subspace test | contains 0, closed under + and scalar | deciding if a set is a subspace |
| dim R^n, M(m×n), P_d | n, mn, d+1 | counting dimension |
| dim(U+W) | dim U + dim W - dim(U∩W) | sum of subspaces |
| Rank-nullity | rank + nullity = #columns | any "find nullity" question |
| Independence test | rank [v1..vk] = k | any independence question |
| Column space basis | pivot columns of **original** A | basis problems |
| Row space basis | nonzero rows of echelon form | basis problems |
| dim N(A), dim N(A^T) | n - r, m - r | four subspaces |
| Product rank | rank(AB) ≤ min; ≥ rA + rB - n | rank bounds |
| Gram | rank(A^T A) = rank(A) | least squares |
| Solutions of Ax = 0 | subspace of dim n - r | counting free parameters |

## GATE traps
- **A subspace must contain 0.** Any set defined by "= nonzero constant" fails; quick elimination in MCQs.
- **Union of subspaces is not a subspace** (e.g. two coordinate axes).
- The set {Ax = b} for b ≠ 0 is not a subspace; its solution set is x_p + N(A).
- The column space basis uses columns of the **original** matrix, not of the reduced form (row ops change the column space).
- "n + 1 vectors in R^n" is dependent; "n vectors" need a determinant/rank check; "fewer than n" can never span R^n.
- Rank(A^T A) = rank(A) fails over complex numbers and says nothing for rank(A A) (e.g. nilpotent A).
- Nullity counts **columns minus rank**, never rows minus rank (that is the left null space).
- Dimension of symmetric vs. skew-symmetric n × n: n(n+1)/2 vs. n(n-1)/2; they sum to n².
- Polynomials of degree **exactly** d, or **monic** polynomials, are not subspaces.

## Connections
- [Systems of linear equations](linear-systems-and-lu.md) — Ax = b is solvable iff b is in C(A); the solution set is x_p + N(A).
- [Determinants](matrices-and-determinants.md) — det ≠ 0 iff columns independent iff rank n.
- [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md) — the eigenspace of λ is N(A - λI); geometric multiplicity = its dimension.
- [Orthogonality, projections, SVD](orthogonality-projections-svd.md) — projection onto C(A); rank = number of nonzero singular values.
- [PCA](../16-machine-learning/dimensionality-reduction-pca.md) — the principal subspace is a low-dimensional subspace chosen by singular vectors.
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) — features as columns; collinear features mean dependent columns (rank-deficient design matrix).
- [Sets, relations and functions](../01-discrete-mathematics/sets-relations-functions.md) — closure and injective/surjective ideas reused for linear maps.
- [Algebraic structures](../01-discrete-mathematics/algebraic-structures.md) — (V, +) is an abelian group; the other axioms add scalars.

## Practice

**Q1 (MCQ).** Which of the following is a subspace of R^3? (A) {(x,y,z): x + y = 1} (B) {(x,y,z): xyz = 0} (C) {(x,y,z): x = y = z} (D) {(x,y,z): z ≥ 0}

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** (A) lacks 0. (B) (1,1,0) + (0,0,1) = (1,1,1) has product 1. (D) not closed under multiplication by -1. (C) is the line spanned by (1,1,1): closed under + and scalars.

</details>

**Q2 (NAT).** The rank of [[1,2,3],[2,3,4],[3,5,7]] is ___.

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** R2 - 2R1 = (0,-1,-2); R3 - 3R1 = (0,-1,-2); these two are equal, so R3' - R2' = 0. Two nonzero rows remain.

</details>

**Q3 (NAT).** A is a 5 × 7 matrix of rank 4. The dimension of the null space of A is ___, and of the null space of A^T is ___.

<details><summary>Answer</summary>

**Answer:** 3 and 1  
**Solution:** N(A) ⊆ R^7: 7 - 4 = 3. N(A^T) ⊆ R^5: 5 - 4 = 1.

</details>

**Q4 (MSQ).** Let v1, v2, v3 be independent in R^5. Which sets are necessarily independent? (A) {v1, v2} (B) {v1 + v2, v2 + v3, v1 + v3} (C) {v1 - v2, v2 - v3, v3 - v1} (D) {v1, v2, v3, 0}

<details><summary>Answer</summary>

**Answer:** (A), (B)  
**Solution:** (A) subset of independent set. (B) coefficient matrix [[1,1,0],[0,1,1],[1,0,1]] has det = 1(1) - 1(0 - 1) + 0 = 2 ≠ 0, so independent. (C) the three sum to 0, dependent. (D) contains 0, dependent.

</details>

**Q5 (NAT).** The dimension of the subspace of 3 × 3 real matrices that are both symmetric and have trace 0 is ___.

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** Symmetric 3 × 3 has dimension 6; the trace-zero condition is one extra independent linear equation on them, giving 5.

</details>

**Q6 (MCQ).** A is 3 × 4 with rank 3, B is 4 × 3 with rank 2. Then rank(AB) can be (A) only 2 (B) 1 or 2 (C) 0, 1 or 2 (D) 3

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Upper bound min(3, 2) = 2. Sylvester lower bound: 3 + 2 - 4 = 1. So 1 or 2 (both achievable). D is impossible since rank(AB) ≤ 2.

</details>

**Q7 (NAT).** For the matrix [[1,1,1],[1,2,4],[1,k,k²]], the sum of all values of k for which the rank is less than 3 is ___.

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** det = (2-1)(k-1)(k-2) (Vandermonde). Zero at k = 1, 2; sum = 3. At those values rank = 2.

</details>

**Q8 (MCQ).** Let U = {x in R^4 : x1 + x2 + x3 + x4 = 0} and W = {x in R^4 : x1 = x2}. dim(U ∩ W) and dim(U + W) are (A) 2 and 4 (B) 3 and 3 (C) 1 and 5 (D) 2 and 3

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** U ∩ W is cut out by two independent equations, so dim 4 - 2 = 2. dim(U + W) = 3 + 3 - 2 = 4. (Dimension 5 is impossible in R^4.)

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
