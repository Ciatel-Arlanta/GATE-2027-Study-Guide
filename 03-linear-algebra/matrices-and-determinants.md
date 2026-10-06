# Matrices, Determinants, Special Matrices and Quadratic Forms

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Matrices and matrix operations; Determinants; Idempotent matrices; Partition matrices and properties; Quadratic forms
> **Prerequisites:** [Vector spaces](vector-spaces.md) · **Leads to:** [Linear systems and LU](linear-systems-and-lu.md) · [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md)

## Quick glance
- Matrix product (AB)_ij = row i of A · column j of B; needs (m × n)(n × p). **AB ≠ BA** in general. (AB)^T = B^T A^T, (AB)^-1 = B^-1 A^-1 (order reverses).
- det: triangular matrix = product of diagonal. **Row swap flips sign; scaling a row by c multiplies det by c; adding a multiple of a row changes nothing.**
- det(AB) = det A det B; det(A^T) = det A; **det(kA) = k^n det A**; det(A^-1) = 1/det A; A invertible iff det ≠ 0.
- A^-1 = adj(A)/det(A); adj(A) = transpose of cofactor matrix; A adj(A) = det(A) I; det(adj A) = det(A)^(n-1).
- Block triangular: det = det(A11) det(A22). General 2 × 2 block: det = det(A) det(D - C A^-1 B) (Schur complement).
- Idempotent A^2 = A: eigenvalues 0 or 1, **rank = trace**. Nilpotent: all eigenvalues 0. Orthogonal: Q^T Q = I, det = ±1.
- Quadratic form x^T A x with A symmetric. **Positive definite iff all eigenvalues > 0 iff all leading principal minors > 0.**
- **#1 trap:** det(A + B) ≠ det A + det B; and (A + B)^2 ≠ A^2 + 2AB + B^2 unless AB = BA.

## 1. Matrix operations

**Intuition.** A matrix is a table that stands for a linear map; the product AB means "apply B first, then A".

**Operations.** Addition and scalar multiplication are entrywise. Transpose swaps rows and columns. Trace(A) = sum of diagonal entries (square A).

**Worked example 1.** A = [[1,2],[3,4]], B = [[0,1],[1,0]].
- AB: row 1 of A · columns of B: (1·0 + 2·1, 1·1 + 2·0) = (2, 1); row 2: (3·0 + 4·1, 3·1 + 4·0) = (4, 3). So AB = [[2,1],[4,3]] (columns of A swapped).
- BA = [[3,4],[1,2]] (rows of A swapped). **AB ≠ BA.**
- trace(AB) = 5 = trace(BA). Always trace(AB) = trace(BA).

| Law | Holds? |
|---|---|
| (A + B)^T = A^T + B^T; (AB)^T = B^T A^T | yes |
| (AB)^-1 = B^-1 A^-1 | yes (both invertible) |
| (A^-1)^T = (A^T)^-1 | yes |
| AB = AC and A ≠ 0 implies B = C | **no** (needs A invertible) |
| AB = 0 implies A = 0 or B = 0 | **no** ([[1,0],[0,0]] [[0,0],[0,1]] = 0) |
| (A + B)^2 = A^2 + 2AB + B^2 | only if AB = BA |
| trace(AB) = trace(BA); trace(ABC) = trace(BCA) | yes (cyclic) |
| A^m A^n = A^(m+n) | yes |

**Powers.** Pattern-spot or diagonalise: [[1,1],[0,1]]^n = [[1,n],[0,1]]. [[2,1],[1,1]]^n = [[F(2n+1), F(2n)],[F(2n), F(2n-1)]]; for n = 10 this is [[10946, 6765],[6765, 4181]].

**Symmetric / skew decomposition.** Every square A = (A + A^T)/2 + (A - A^T)/2: symmetric part + skew part. This is the source of "symmetrise the quadratic form".

## 2. Determinants

**Definition (n = 2, 3).** det [[a,b],[c,d]] = ad - bc. For larger n use **cofactor expansion** along any row/column: det A = Σ_j a_ij C_ij, with C_ij = (-1)^(i+j) M_ij (M_ij is the minor, determinant after deleting row i and column j). Sign pattern:

```text
+ - + -
- + - +
+ - + -
- + - +
```

**Geometric meaning.** |det A| is the volume-scaling factor of the map x → Ax; the sign tells whether orientation is flipped.

**Properties.**

| Property | Effect |
|---|---|
| Swap two rows (or columns) | det changes sign |
| Multiply one row by c | det multiplied by c |
| Add multiple of a row to another | **unchanged** |
| Two equal rows / a zero row / dependent rows | det = 0 |
| det(A^T) = det(A) | rows and columns behave alike |
| det(AB) = det(A) det(B) | |
| det(kA) = k^n det(A) | n = order of A (every row scaled) |
| det(A^-1) = 1 / det(A) | |
| det(A^m) = det(A)^m | |
| Triangular / diagonal matrix | product of diagonal entries |
| Block triangular | product of diagonal block determinants |
| det = product of eigenvalues | see [eigenvalues](eigenvalues-and-eigenvectors.md) |
| Orthogonal matrix | det = ±1 |
| Skew-symmetric, n odd | det = 0 |
| det(I) = 1 | |

**Worked example 2 — 3 × 3 by cofactors.** M = [[2,1,3],[0,-1,4],[1,2,5]]. Expand along row 1:
det = 2·(-1·5 - 4·2) - 1·(0·5 - 4·1) + 3·(0·2 - (-1)·1) = 2(-13) - 1(-4) + 3(1) = -26 + 4 + 3 = **-19**.

**Worked example 3 — 4 × 4 by row operations.** M = [[1,2,0,3],[2,5,1,7],[0,1,3,2],[1,3,2,6]].
1. R2 ← R2 - 2R1 = (0,1,1,1); R4 ← R4 - R1 = (0,1,2,3). (det unchanged.)
2. Expand along column 1 (only the pivot 1 remains): det = det [[1,1,1],[1,3,2],[1,2,3]].
3. R2 ← R2 - R1 = (0,2,1); R3 ← R3 - R1 = (0,1,2). det = 1·(2·2 - 1·1) = **3**.

**Worked example 4 — scaling.** A is 3 × 3 with det A = 4. Then det(2A) = 2^3 · 4 = 32, det(A^-1) = 1/4, det(A^2) = 16, det(adj A) = 4^2 = 16, det(-A) = -4.

**Vandermonde.** det [[1,a,a²],[1,b,b²],[1,c,c²]] = (b - a)(c - a)(c - b).

## 3. Inverse and adjugate

**Definition.** A is invertible iff det A ≠ 0. The inverse is A^-1 = adj(A) / det(A), where adj(A) is the **transpose of the cofactor matrix**.

**2 × 2 shortcut:** [[a,b],[c,d]]^-1 = 1/(ad - bc) · [[d,-b],[-c,a]].

**Worked example 5.** For M above (det = -19), the cofactor matrix has entries C_ij; C11 = -13, C12 = -(0·5 - 4·1) = 4, C13 = 1, C21 = -(1·5 - 3·2) = 1, C22 = 2·5 - 3·1 = 7, C23 = -(2·2 - 1·1) = -3, C31 = 1·4 - 3·(-1) = 7, C32 = -(2·4 - 3·0) = -8, C33 = 2·(-1) - 0 = -2. Cofactor matrix [[-13,4,1],[1,7,-3],[7,-8,-2]]; transpose gives adj(M) = [[-13,1,7],[4,7,-8],[1,-3,-2]]. So M^-1 = -(1/19) adj(M) = [[13/19,-1/19,-7/19],[-4/19,-7/19,8/19],[-1/19,3/19,2/19]]. (Check M·adj(M) = -19 I in the (1,1) entry: 2(-13) + 1(4) + 3(1) = -19.)

**Adjugate facts (n × n):**
- A adj(A) = adj(A) A = det(A) I
- det(adj A) = det(A)^(n-1); adj(adj A) = det(A)^(n-2) A
- adj(AB) = adj(B) adj(A); adj(kA) = k^(n-1) adj(A)
- rank(adj A) = n if rank A = n; 1 if rank A = n - 1; 0 if rank A ≤ n - 2

**Inverse facts:** (A^-1)^-1 = A; (kA)^-1 = A^-1/k; inverse of a diagonal matrix inverts the diagonal; inverse of an upper triangular matrix is upper triangular (diagonal entries 1/a_ii). Computation by Gauss–Jordan is in [linear systems](linear-systems-and-lu.md).

## 4. Special matrices

| Type | Definition | Eigenvalue / other facts |
|---|---|---|
| Symmetric | A^T = A | eigenvalues real; eigenvectors for distinct eigenvalues orthogonal; orthogonally diagonalisable |
| Skew-symmetric | A^T = -A | diagonal 0; eigenvalues purely imaginary or 0; det = 0 if n odd; x^T A x = 0 |
| Orthogonal | A^T A = I | |λ| = 1; det = ±1; preserves length and dot products |
| Idempotent | A² = A | eigenvalues only 0, 1; **rank = trace**; I - A also idempotent; singular unless A = I |
| Nilpotent | A^k = 0 for some k | all eigenvalues 0; trace = 0, det = 0; I + A invertible |
| Involutory | A² = I | eigenvalues ±1; A^-1 = A |
| Projection (orthogonal) | A² = A = A^T | eigenvalues 0, 1; see [projections](orthogonality-projections-svd.md) |
| Positive definite | symmetric, x^T A x > 0 for x ≠ 0 | all eigenvalues > 0; invertible; Cholesky exists |
| Diagonal / triangular | | eigenvalues = diagonal entries |
| Singular | det = 0 | has eigenvalue 0; rank < n |
| Stochastic (row) | rows non-negative, sum 1 | eigenvalue 1 (eigenvector all ones) |
| Hermitian / unitary | A* = A / A* A = I | complex analogues of symmetric / orthogonal |

**Worked example 6 — idempotent.** A = [[2,-2,-4],[-1,3,4],[1,-2,-3]]. Compute A²: row 1: (2·2 + (-2)(-1) + (-4)(1), 2(-2) + (-2)(3) + (-4)(-2), 2(-4) + (-2)(4) + (-4)(-3)) = (4 + 2 - 4, -4 - 6 + 8, -8 - 8 + 12) = (2, -2, -4). Matches row 1 of A; similarly the other rows. So A² = A. Trace = 2 + 3 - 3 = 2, so **rank A = 2** and eigenvalues are {1, 1, 0}. Proof sketch of the eigenvalue claim: Ax = λx gives A²x = λ²x = λx, so λ² = λ.

**Worked example 7 — nilpotent / involutory.** N = [[0,1],[0,0]]: N² = 0. Then (I + N)^-1 = I - N. The reflection [[0,1],[1,0]] satisfies A² = I.

## 5. Partitioned (block) matrices

Treat blocks like entries, **provided sizes are compatible**: [[A,B],[C,D]] [[E,F],[G,H]] = [[AE + BG, AF + BH],[CE + DG, CF + DH]]. Order of block products matters (non-commutative). Transpose: [[A,B],[C,D]]^T = [[A^T, C^T],[B^T, D^T]].

**Determinant:**
- Block triangular (B = 0 or C = 0): **det = det(A) det(D)**.
- General, A invertible: det = det(A) · det(D - C A^-1 B). The matrix S = D - C A^-1 B is the **Schur complement** of A. If D is invertible instead: det = det(D) det(A - B D^-1 C).
- If all blocks are n × n and commute (CD = DC): det = det(AD - BC).

**Block inverse:**
- Block diagonal: diag(A, D)^-1 = diag(A^-1, D^-1).
- Block upper triangular: [[A,B],[0,D]]^-1 = [[A^-1, -A^-1 B D^-1],[0, D^-1]].
- General: [[A,B],[C,D]]^-1 = [[A^-1 + A^-1 B S^-1 C A^-1, -A^-1 B S^-1],[-S^-1 C A^-1, S^-1]] with S = D - C A^-1 B.

**Rank:** rank [[A,0],[0,D]] = rank A + rank D; rank [A | B] ≥ max(rank A, rank B) and ≤ rank A + rank B. **Eigenvalues** of a block triangular matrix = union of the eigenvalues of the diagonal blocks.

**Worked example 8 — block triangular.** P = [[1,2,0,0],[3,4,0,0],[0,0,5,6],[0,0,7,8]]. det = det[[1,2],[3,4]] · det[[5,6],[7,8]] = (4 - 6)(40 - 42) = (-2)(-2) = **4**.

**Worked example 9 — Schur complement.** B = [[2,1,0,1],[1,3,1,0],[1,0,4,1],[0,1,1,3]]. Blocks: A = [[2,1],[1,3]], B12 = [[0,1],[1,0]], C = I, D = [[4,1],[1,3]].
1. det A = 6 - 1 = 5; A^-1 = (1/5)[[3,-1],[-1,2]].
2. C A^-1 B12 = A^-1 B12 = (1/5)[[3·0 + (-1)·1, 3·1 + (-1)·0],[(-1)·0 + 2·1, (-1)·1 + 2·0]] = (1/5)[[-1,3],[2,-1]].
3. S = D - (1/5)[[-1,3],[2,-1]] = [[21/5, 2/5],[3/5, 16/5]]; det S = (336 - 6)/25 = 330/25 = 66/5.
4. det B = 5 · 66/5 = **66**. (Direct computation also gives 66.)

## 6. Quadratic forms

**Definition.** A quadratic form in x = (x1..xn) is q(x) = x^T A x = Σ a_ij x_i x_j. Only the symmetric part of A matters because x^T A x = x^T ((A + A^T)/2) x. So **always symmetrise first**: the coefficient of x_i x_j (i ≠ j) splits equally, a_ij = a_ji = (coefficient)/2.

**Worked example 10 — writing the matrix.** q = 2x² + 6xy + 5y² gives A = [[2,3],[3,5]]. For a non-symmetric A = [[1,4],[0,3]], x^T A x = x² + 4xy + 3y², with symmetric matrix [[1,2],[2,3]].

**Definiteness** (A symmetric):

| Class | Condition on x^T A x | Eigenvalues | Leading principal minors |
|---|---|---|---|
| Positive definite (PD) | > 0 for all x ≠ 0 | all > 0 | **all > 0** |
| Positive semidefinite (PSD) | ≥ 0 for all x | all ≥ 0 | **all principal minors ≥ 0** (every one, not just leading) |
| Negative definite | < 0 for x ≠ 0 | all < 0 | alternate: D1 < 0, D2 > 0, D3 < 0, ... |
| Indefinite | takes both signs | mixed signs | otherwise |

Leading principal minor D_k = determinant of the top-left k × k block (Sylvester's criterion).

**Worked example 11 — indefinite.** q = x² + 4xy + y², A = [[1,2],[2,1]]. D1 = 1 > 0, D2 = 1 - 4 = -3 < 0, so not PD. Eigenvalues: λ² - 2λ - 3 = 0, so 3 and -1: **indefinite**. Check: q(1,1) = 6 > 0, q(1,-1) = -2 < 0.

**Worked example 12 — positive definite.** A = [[2,-1,0],[-1,2,-1],[0,-1,2]]. D1 = 2, D2 = 4 - 1 = 3, D3 = det A = 2(4 - 1) + 1(-2 - 0) = 6 - 2 = 4. All positive, so PD. (Eigenvalues 2, 2 ± √2, all positive; sum 6 = trace, product 4 = det.)

**Worked example 13 — completing the square.** q = x² + 4xy + 5y² = (x + 2y)² + y². A sum of squares with positive coefficients, so PD. The diagonal form coefficients (1, 1) show the signs; the number of positive/negative squares equals the number of positive/negative eigenvalues (**Sylvester's law of inertia**). By contrast x² + 2xy + y² = (x + y)² is PSD, not PD (zero at (1,-1)).

**Worked example 14 — parameter.** A = [[1,a],[a,4]] is PD iff D1 = 1 > 0 and D2 = 4 - a² > 0, i.e. |a| < 2. At a = ±2 it is PSD (det 0).

**Useful facts:** For real A (any shape) A^T A is symmetric PSD, and PD iff columns of A are independent. A PD matrix has positive diagonal entries. The minimum of q on the unit sphere is λ_min and the maximum is λ_max (**Rayleigh quotient**). A symmetric 2 × 2 is PD iff a > 0 and ac - b² > 0.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| (AB)^T, (AB)^-1 | B^T A^T, B^-1 A^-1 | reversal of order |
| det(kA) | k^n det A | scaling questions |
| det(AB), det(A^-1) | det A det B; 1/det A | |
| A^-1 | adj A / det A | small inverses |
| det(adj A), adj(adj A) | (det A)^(n-1); (det A)^(n-2) A | adjugate questions |
| Block triangular det | det(A) det(D) | partitioned matrices |
| Schur | det A · det(D - C A^-1 B) | general 2 × 2 blocks |
| Idempotent | rank = trace; eigenvalues 0/1 | |
| Quadratic form | x^T A x, A symmetric | symmetrise first |
| PD tests | eigenvalues > 0; leading minors > 0 | definiteness |
| Skew-symmetric odd n | det = 0 | |
| Vandermonde 3 × 3 | (b-a)(c-a)(c-b) | parameter problems |
| 2 × 2 inverse | (1/(ad - bc)) [[d,-b],[-c,a]] | |

## GATE traps
- det(A + B) ≠ det(A) + det(B); det(kA) = **k^n** det(A), not k det(A).
- Adding a multiple of a row leaves det unchanged; **swapping** negates it; **scaling** a row scales it. Forgetting the factor when you scale during elimination is the commonest slip.
- adj(A) is the **transpose** of the cofactor matrix; forgetting the transpose gives the wrong inverse.
- Block formula det(AD - BC) needs the blocks to commute; otherwise use the Schur complement.
- "Positive semidefinite" needs **all** principal minors ≥ 0; leading ones alone are not enough ([[0,0],[0,-1]] has D1 = D2 = 0 but is not PSD).
- Non-symmetric A: definiteness of x^T A x is decided by the symmetrised matrix (A + A^T)/2, not by eigenvalues of A.
- An idempotent matrix is not necessarily symmetric; only symmetric idempotent ones are orthogonal projections.
- If A² = A and A ≠ I then A is singular; if A is nilpotent then I - A is invertible.
- (A + B)^2 expansion and (AB)^k = A^k B^k both need AB = BA.

## Connections
- [Vector spaces](vector-spaces.md) — det ≠ 0 iff rank n iff independent columns.
- [Linear systems and LU](linear-systems-and-lu.md) — row operations give both det and inverse; det(A) = product of U's diagonal (± for swaps).
- [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md) — det = product, trace = sum of eigenvalues; definiteness via eigenvalues.
- [Orthogonality, projections, SVD](orthogonality-projections-svd.md) — orthogonal and projection matrices; A^T A is PSD.
- [Probability: random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — a covariance matrix is symmetric PSD; the multivariate Gaussian needs PD.
- [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) — the Hessian's definiteness classifies critical points (multivariable second-derivative test).
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) — the normal-equation matrix X^T X is PSD (PD for independent features); ridge adds λI to make it PD.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — adjacency and incidence matrices; Kirchhoff's theorem counts spanning trees with a cofactor.

## Practice

**Q1 (NAT).** A is 3 × 3 with det A = -2. The value of det(2 A^T A^-1 A²) is ___.

<details><summary>Answer</summary>

**Answer:** 32  
**Solution:** The scalar 2 contributes 2^3 = 8. det(A^T) = -2, det(A^-1) = -1/2, det(A²) = 4. Product = 8 · (-2) · (-1/2) · 4 = 8 · 1 · 4 = 32.

</details>

**Q2 (NAT).** The determinant of [[1,2,3],[4,5,6],[7,8,10]] is ___.

<details><summary>Answer</summary>

**Answer:** -3  
**Solution:** R2 - 4R1 = (0,-3,-6), R3 - 7R1 = (0,-6,-11). det = 1·((-3)(-11) - (-6)(-6)) = 33 - 36 = -3.

</details>

**Q3 (MCQ).** A is a 4 × 4 matrix with det A = 3. The determinant of adj(A) is (A) 3 (B) 9 (C) 27 (D) 81

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** det(adj A) = det(A)^(n-1) = 3^3 = 27.

</details>

**Q4 (MSQ).** A is a real n × n matrix with A² = A. Which are always true? (A) eigenvalues are 0 or 1 (B) rank A = trace A (C) A is symmetric (D) I - A is idempotent

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)  
**Solution:** (A) λ² = λ. (B) rank equals the number of eigenvalue-1 entries since A is diagonalisable (minimal polynomial divides x² - x which has distinct roots) so trace = rank. (D) (I - A)² = I - 2A + A² = I - A. (C) false: [[1,1],[0,0]] is idempotent but not symmetric.

</details>

**Q5 (NAT).** The quadratic form 2x² + 2y² + 2z² - 2xy - 2yz is positive definite. Its determinant (of the symmetric matrix) is ___.

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** Matrix [[2,-1,0],[-1,2,-1],[0,-1,2]]. det = 2(4 - 1) - (-1)(-2 - 0) + 0 = 6 - 2 = 4. Leading minors 2, 3, 4 are positive, confirming PD.

</details>

**Q6 (MCQ).** For which real a is [[1,a],[a,9]] positive definite? (A) a > 3 (B) |a| < 3 (C) |a| ≤ 3 (D) all real a

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** D1 = 1 > 0 and D2 = 9 - a² > 0 gives |a| < 3. (At |a| = 3 it is only semidefinite.)

</details>

**Q7 (NAT).** Let M = [[A, B],[0, D]] with A = [[1,2],[0,3]], D = [[2,0],[1,4]] and arbitrary 2 × 2 block B. The determinant of M is ___.

<details><summary>Answer</summary>

**Answer:** 24  
**Solution:** Block upper triangular: det = det A · det D = 3 · 8 = 24, independent of B.

</details>

**Q8 (MCQ).** The matrix [[1,2,3],[2,4,6],[3,6,k]] has determinant (A) 0 for every k (B) 0 only for k = 9 (C) k - 9 (D) 3k - 18

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** Row 2 = 2 × Row 1, so the rows are dependent for every k and det = 0. (Rank is 1 when k = 9 and 2 otherwise.)

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
