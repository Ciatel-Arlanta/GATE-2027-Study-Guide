# Orthogonality, Projections, Least Squares and SVD

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** Orthogonal matrices; Projection matrices; Projections; Idempotent matrices; Singular value decomposition
> **Prerequisites:** [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md) · [Vector spaces](vector-spaces.md) · [Matrices and determinants](matrices-and-determinants.md) · **Leads to:** [PCA](../16-machine-learning/dimensionality-reduction-pca.md) · [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md)

## Quick glance
- **Inner product** x·y = x^T y; length ‖x‖ = √(x^T x); orthogonal means x·y = 0. Orthogonal non-zero vectors are independent.
- **Orthogonal matrix Q:** Q^T Q = Q Q^T = I, so Q^-1 = Q^T. It preserves lengths and angles, **det Q = ±1**, all |λ| = 1. det +1: rotation; det -1: reflection.
- **Projection of b onto a:** p = (a^T b / a^T a) a; matrix P = a a^T / (a^T a).
- **Projection onto the column space of A (full column rank):** **P = A (A^T A)^-1 A^T**. P² = P, P^T = P, eigenvalues 0 and 1, **trace P = rank P**.
- **Least squares:** minimise ‖Ax - b‖²; the **normal equations A^T A x̂ = A^T b** give x̂ = (A^T A)^-1 A^T b; A x̂ = P b; residual b - A x̂ ⟂ C(A).
- **Gram–Schmidt:** subtract from each new vector its components along the earlier q's, then normalise. Gives A = QR.
- **SVD:** every m × n matrix A = U Σ V^T with U (m × m), V (n × n) orthogonal and Σ diagonal with σ1 ≥ σ2 ≥ ... ≥ 0. **σ_i = √(eigenvalue of A^T A)**; **rank A = number of non-zero σ_i**.
- **Best rank-k approximation** (Eckart–Young): keep the k largest σ_i; error ‖A - A_k‖_F² = Σ_{i>k} σ_i². This is the mathematics of PCA.
- **#1 trap:** P = A(A^T A)^-1 A^T needs independent columns; for a single vector use a a^T/(a^T a), not a a^T.

## 1. Orthogonality

**Intuition.** Two vectors at right angles share no direction, so each can be treated separately. Orthonormal bases make every computation trivial: coordinates are just dot products.

**Definitions.**
- x·y = Σ x_i y_i = x^T y = ‖x‖‖y‖ cos θ.
- x ⟂ y iff x·y = 0. A set is **orthogonal** if every pair is orthogonal and **orthonormal** if additionally every ‖q_i‖ = 1.
- **Orthogonal complement** of a subspace W: W^⟂ = {x : x·w = 0 for all w ∈ W}. dim W + dim W^⟂ = n.
- From the four subspaces: **C(A^T) ⟂ N(A)** (in R^n) and **C(A) ⟂ N(A^T)** (in R^m). A row of A dotted with a null vector is 0 by definition of Ax = 0.
- Cauchy–Schwarz: |x·y| ≤ ‖x‖‖y‖. Pythagoras: x ⟂ y ⇒ ‖x + y‖² = ‖x‖² + ‖y‖².

**Coordinates in an orthonormal basis.** If q1,...,qn is an orthonormal basis then x = Σ (q_i·x) q_i. No linear system to solve.

**Worked example 1.** q1 = (1,1)/√2, q2 = (1,-1)/√2, x = (3,1). q1·x = 4/√2, q2·x = 2/√2. Check: (4/√2)(1,1)/√2 + (2/√2)(1,-1)/√2 = (2,2) + (1,-1) = (3,1) ✓.

## 2. Orthogonal matrices

**Definition.** A square matrix Q is **orthogonal** if **Q^T Q = I** (columns orthonormal). Then Q^-1 = Q^T and the rows are orthonormal too.

| Property | Reason |
|---|---|
| ‖Qx‖ = ‖x‖ | (Qx)^T(Qx) = x^T Q^T Q x = x^T x |
| (Qx)·(Qy) = x·y (angles preserved) | same |
| det Q = ±1 | det(Q^T Q) = (det Q)² = 1 |
| every eigenvalue has \|λ\| = 1 | ‖Qx‖ = \|λ\|‖x‖ |
| Product of orthogonal matrices is orthogonal; Q^-1 = Q^T is orthogonal | closure (group O(n)) |
| Real eigenvalues can only be +1 or -1 | \|λ\| = 1 and real |

**Geometry in R².** Rotation by θ: R = [[cos θ, -sin θ],[sin θ, cos θ]], det = +1, eigenvalues e^(±iθ). Reflection about the line at angle θ/2: [[cos θ, sin θ],[sin θ, -cos θ]], det = -1, eigenvalues +1, -1. **Householder reflection** H = I - 2 u u^T (‖u‖ = 1) is orthogonal, symmetric, H² = I, eigenvalues -1 once and +1 (n - 1) times.

**Worked example 2.** Q = [[3/5, -4/5],[4/5, 3/5]]. Columns: (3/5)² + (4/5)² = 1; dot = -12/25 + 12/25 = 0. So Q is orthogonal, det = 9/25 + 16/25 = 1, a rotation by θ with cos θ = 3/5. Q (5,0)^T = (3,4): length 5 preserved ✓. Eigenvalues 3/5 ± 4i/5 (modulus 1).

**Worked example 3 — completing a row.** Row 1 of a 2 × 2 orthogonal matrix is (0.6, x). Then 0.36 + x² = 1, x = ±0.8. The other row is a unit vector orthogonal to it, ±(-x, 0.6): 2 choices. So there are 2 × 2 = 4 such matrices (two are rotations, det +1, two are reflections, det -1).

**Orthogonal vs. orthonormal wording.** GATE calls a matrix "orthogonal" only if its columns are orthonormal. A matrix with orthogonal but not unit columns satisfies Q^T Q = diagonal, not I.

## 3. Projection onto a vector

**Intuition.** The shadow of b on the line through a. The error b - p must be perpendicular to a.

**Derivation.** p = x̂ a with a·(b - x̂ a) = 0 ⇒ **x̂ = a^T b / a^T a**. So p = (a^T b / a^T a) a = P b with **P = a a^T / (a^T a)** (a rank-1 matrix).

**Worked example 4.** a = (1,2,2), b = (3,0,0). a^T a = 9, a^T b = 3, x̂ = 1/3, p = (1/3, 2/3, 2/3). Error e = b - p = (8/3, -2/3, -2/3); a·e = 8/3 - 4/3 - 4/3 = 0 ✓. P = (1/9)[[1,2,2],[2,4,4],[2,4,4]]; trace = 9/9 = 1 = rank. Distance from b to the line = ‖e‖ = √(64 + 4 + 4)/3 = √72/3 = 2√2.

## 4. Projection onto a subspace; projection matrices

Let A be m × n with **independent columns**; the subspace is C(A). We seek p = A x̂ with b - A x̂ ⟂ every column: **A^T (b - A x̂) = 0**, i.e. the **normal equations**

**A^T A x̂ = A^T b**, x̂ = (A^T A)^-1 A^T b, p = A (A^T A)^-1 A^T b.

A^T A is invertible exactly when A has independent columns (rank(A^T A) = rank A).

**Projection matrix:** P = A (A^T A)^-1 A^T.

| Property | Why |
|---|---|
| **P² = P** (idempotent) | projecting twice = projecting once |
| **P^T = P** (symmetric) | formula is symmetric |
| Eigenvalues only 0 and 1 | λ² = λ |
| **trace P = rank P = dim of the subspace** | trace = sum of eigenvalues = number of 1's |
| P b ∈ C(A); (I - P) b ∈ N(A^T) | decomposition b = Pb + (I - P)b |
| I - P is the projection onto the orthogonal complement | (I-P)² = I - P, symmetric |
| P x = x for x in C(A) | |
| If Q has orthonormal columns, P = Q Q^T | A^T A = I |

**Idempotent ≠ projection.** An idempotent matrix (P² = P) is an *oblique* projection; it is an **orthogonal** projection iff additionally P = P^T. Any idempotent has eigenvalues in {0,1}, rank = trace, and I - P also idempotent.

**Worked example 5.** A = [[1,1],[1,2],[1,3]] (columns 1 and t). A^T A = [[3,6],[6,14]], det = 42 - 36 = 6, (A^T A)^-1 = (1/6)[[14,-6],[-6,3]]. Then P = A (A^T A)^-1 A^T = (1/6)[[5,2,-1],[2,2,2],[-1,2,5]]. Checks: trace = (5 + 2 + 5)/6 = 2 = rank ✓; symmetric ✓; P·(1,1,1)^T = (1/6)(6,6,6) = (1,1,1) ✓ (the column is in C(A)). P² = P can be verified entrywise, e.g. row 1 · column 1 of the numerators: 25 + 4 + 1 = 30 = 6 · 5 ✓.

## 5. Least squares

When Ax = b has no exact solution (m equations, n < m unknowns, b ∉ C(A)), choose x̂ minimising ‖b - Ax‖². The minimiser is the projection, so **x̂ solves A^T A x̂ = A^T b**; the minimum residual is ‖b - P b‖ and **the residual is orthogonal to every column of A**.

**Worked example 6 — fit y = c + m t to (1,1), (2,2), (3,2).** A = [[1,1],[1,2],[1,3]], b = (1,2,2).
1. A^T A = [[3,6],[6,14]]; A^T b = (1 + 2 + 2, 1 + 4 + 6) = (5, 11).
2. Solve 3c + 6m = 5, 6c + 14m = 11. Subtract 2 × first: 2m = 1, **m = 1/2**, c = (5 - 3)/3 = **2/3**.
3. Fitted values p = (7/6, 5/3, 13/6). Residual e = b - p = (-1/6, 1/3, -1/6).
4. Orthogonality: sum of e = 0 ✓; Σ t e = -1/6 + 2/3 - 1/2 = 0 ✓.
5. Minimum squared error = ‖e‖² = 1/36 + 1/9 + 1/36 = **1/6**.

**Link to ML.** [Linear regression](../16-machine-learning/linear-and-logistic-regression.md): design matrix X, weights w, the OLS solution is **w = (X^T X)^-1 X^T y**, exactly the normal equations. When X has dependent columns (collinear features) X^T X is singular and ridge regression uses (X^T X + λI)^-1, which is always invertible for λ > 0. Setting the gradient of ‖Xw - y‖² to zero gives 2 X^T (Xw - y) = 0, the same equations.

## 6. Gram–Schmidt and QR

Turn independent a1,...,ak into orthonormal q1,...,qk spanning the same subspaces.
- q1 = a1/‖a1‖.
- v2 = a2 - (q1·a2) q1; q2 = v2/‖v2‖.
- v3 = a3 - (q1·a3) q1 - (q2·a3) q2; q3 = v3/‖v3‖; and so on.

Result: **A = Q R** with Q having orthonormal columns and R upper triangular with positive diagonal (R_ij = q_i·a_j). Then least squares reduces to R x̂ = Q^T b (back-substitution), numerically better than the normal equations.

**Worked example 7.** a1 = (1,1,0), a2 = (1,0,1).
1. q1 = (1,1,0)/√2.
2. q1·a2 = 1/√2, so v2 = a2 - (1/√2) q1 = (1,0,1) - (1/2)(1,1,0) = (1/2, -1/2, 1). ‖v2‖ = √(1/4 + 1/4 + 1) = √(3/2).
3. q2 = (1,-1,2)/√6. Check q1·q2 = (1 - 1)/√12 = 0 ✓.
4. R = [[√2, 1/√2],[0, √(3/2)]]; (QR check: Q R (1,2 entry) = (1/√2)·q1 + √(3/2) q2 = (1/2,1/2,0) + (1/2,-1/2,1) = (1,0,1) = a2 ✓).

## 7. Singular value decomposition (SVD)

**Intuition.** Any linear map A: R^n → R^m is a **rotation (V^T), then a stretch along the axes (Σ), then another rotation (U)**. Eigen-decomposition needs a square, diagonalisable matrix; SVD exists for **every** matrix.

**Statement.** A (m × n, rank r) = U Σ V^T with
- V (n × n) orthogonal; its columns v_i (**right singular vectors**) are orthonormal eigenvectors of **A^T A**;
- U (m × m) orthogonal; columns u_i (**left singular vectors**) are orthonormal eigenvectors of **A A^T**;
- Σ (m × n) diagonal with **σ_i = √λ_i(A^T A)** ≥ 0, sorted decreasingly; σ_1,...,σ_r > 0, the rest 0;
- A v_i = σ_i u_i, hence u_i = A v_i / σ_i for σ_i > 0.

A^T A and A A^T have the same non-zero eigenvalues (all σ_i²).

**Recipe.**
1. Form A^T A (n × n symmetric PSD); find eigenvalues λ_i ≥ 0, order them, σ_i = √λ_i.
2. Orthonormal eigenvectors → columns of V.
3. u_i = A v_i/σ_i for the non-zero σ_i; complete U to an orthogonal matrix if needed.

**Worked example 8 — full 2 × 2 SVD.** A = [[3,0],[4,5]].
1. A^T A = [[9 + 16, 0 + 20],[20, 0 + 25]] = [[25,20],[20,25]]. Trace 50, det 625 - 400 = 225: λ² - 50λ + 225 = 0, λ = **45, 5**.
2. σ1 = √45 = 3√5 ≈ 6.708, σ2 = √5 ≈ 2.236.
3. λ = 45: (A^T A - 45I) = [[-20,20],[20,-20]], v1 = (1,1)/√2. λ = 5: [[20,20],[20,20]], v2 = (1,-1)/√2.
4. u1 = A v1/σ1 = (3,9)/√2 / (3√5) = **(1,3)/√10**. u2 = A v2/σ2 = (3,-1)/√2 / √5 = **(3,-1)/√10**. u1·u2 = (3 - 3)/10 = 0 ✓.
5. A = σ1 u1 v1^T + σ2 u2 v2^T. Check entry (2,2): 3√5 · (3/√10)(1/√2) + √5 · (-1/√10)(-1/√2) = 3√5·3/√20 + √5/√20 = (9 + 1)√5/(2√5) = 5 ✓.
6. Checks: σ1σ2 = 15 = |det A| ✓; σ1² + σ2² = 50 = Σ a_ij² = 9 + 16 + 25 ✓ (Frobenius norm).

**Facts.**

| Fact | Statement |
|---|---|
| Rank | rank A = number of non-zero singular values |
| Frobenius norm | ‖A‖_F² = Σ a_ij² = Σ σ_i² |
| Spectral (2-)norm | ‖A‖_2 = σ_1 = max ‖Ax‖/‖x‖ |
| Determinant (square) | \|det A\| = Π σ_i |
| Symmetric PSD A | σ_i = λ_i (SVD = eigen-decomposition) |
| Symmetric A | σ_i = \|λ_i\| |
| Orthogonal Q | all σ_i = 1 |
| Condition number | κ(A) = σ_max/σ_min |
| Pseudo-inverse | A⁺ = V Σ⁺ U^T (invert non-zero σ_i); least squares x̂ = A⁺ b |
| Compact form | A = Σ_{i=1}^{r} σ_i u_i v_i^T (sum of rank-1 matrices) |
| Subspaces | v_1..v_r span C(A^T); v_{r+1}..v_n span N(A); u_1..u_r span C(A); the rest span N(A^T) |

**Low-rank approximation (Eckart–Young).** A_k = Σ_{i=1}^{k} σ_i u_i v_i^T is the best rank-k approximation in both Frobenius and spectral norm:
‖A - A_k‖_2 = σ_{k+1}, ‖A - A_k‖_F² = σ_{k+1}² + ... + σ_r².

**Worked example 9.** For A above, A_1 = 3√5 · (1,3)^T(1,1)/(√10 √2) = (3√5/√20)[[1,1],[3,3]] = **[[1.5,1.5],[4.5,4.5]]**. A - A_1 = [[1.5,-1.5],[-0.5,0.5]]; its squared Frobenius norm is 2.25 + 2.25 + 0.25 + 0.25 = 5 = σ2² ✓. The retained "energy" is 45/50 = 90%.

**Link to PCA.** Centre the data matrix X (rows = samples). X^T X/(N - 1) is the covariance matrix, so the principal components are the right singular vectors v_i of X and the variance along v_i is **σ_i²/(N - 1)**. Keeping k components = the rank-k approximation. See [PCA](../16-machine-learning/dimensionality-reduction-pca.md). SVD is also used for latent semantic analysis, image compression, and recommender systems (matrix factorisation).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Orthogonal | Q^T Q = I; ‖Qx‖ = ‖x‖; det ±1 | length, determinant questions |
| Projection on a | a a^T/(a^T a) | line projection |
| Projection on C(A) | A(A^T A)^-1 A^T | subspace projection |
| Projection facts | P² = P = P^T; λ ∈ {0,1}; tr P = rank P | rank/trace questions |
| Normal equations | A^T A x̂ = A^T b | least squares |
| Residual | b - A x̂ ⟂ C(A), lies in N(A^T) | checks |
| Gram–Schmidt | v_k = a_k - Σ (q_i·a_k) q_i | orthonormal basis, QR |
| SVD | A = U Σ V^T, σ_i = √λ_i(A^T A) | any matrix |
| Rank | # non-zero σ_i | rank from SVD |
| Frobenius | Σ σ_i² | norm questions |
| Eckart–Young | error σ_{k+1} (spectral); √Σ_{i>k} σ_i² (Frobenius) | low-rank approximation |

## GATE traps
- **Orthogonal matrix** needs orthonormal columns (unit length), not just orthogonal columns.
- det Q = ±1, **not** always +1; eigenvalues have modulus 1 but are not necessarily ±1 (rotations have complex ones).
- Projection matrix onto a line is a a^T/(a^T a); forgetting the denominator when ‖a‖ ≠ 1 breaks P² = P.
- P = A(A^T A)^-1 A^T is **not** A A^T and **not** simplifiable to I unless A is square invertible (then P = I).
- trace P = rank P (an integer); a projection onto a k-dimensional subspace has k ones among its eigenvalues.
- Idempotent does not imply symmetric; only symmetric idempotents are orthogonal projections.
- Singular values are **square roots** of the eigenvalues of A^T A, not the eigenvalues of A (eigenvalues of a non-symmetric A can differ wildly, e.g. A = [[0,1],[0,0]] has λ = 0, 0 but σ = 1, 0).
- Number of non-zero singular values = rank; for an m × n matrix there are min(m, n) singular values in Σ.
- The normal equations need independent columns; otherwise x̂ is not unique (use the pseudo-inverse).
- The least-squares **residual** is orthogonal to the columns; the **fitted vector** lies in the column space.

## Connections
- [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md) — SVD uses the spectral theorem on A^T A; projections have eigenvalues 0/1; orthogonal matrices have |λ| = 1.
- [Vector spaces](vector-spaces.md) — the four fundamental subspaces are orthogonal pairs; projection splits b into C(A) and N(A^T) parts.
- [Linear systems and LU](linear-systems-and-lu.md) — inconsistent systems: least squares gives the best approximate solution.
- [Matrices and determinants](matrices-and-determinants.md) — idempotent matrices and positive semi-definite A^T A.
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) — OLS = normal equations; the hat matrix H = X(X^T X)^-1 X^T is a projection (trace = number of parameters).
- [PCA](../16-machine-learning/dimensionality-reduction-pca.md) — principal components via SVD/eigen-decomposition of the covariance matrix; variance explained = σ_i²/Σ σ_j².
- [Classification methods](../16-machine-learning/classification-methods.md) — LDA and SVM use projections onto a direction and margins (distance = |w·x + b|/‖w‖).
- [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) — least squares is minimising a convex quadratic: gradient zero gives the normal equations.
- [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — the correlation/regression line is a projection in the space of random variables.

## Practice

**Q1 (NAT).** A is 5 × 3 with independent columns and P is the projection matrix onto C(A). trace(P) = ___.

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** trace of a projection = rank = dim C(A) = 3 (eigenvalues: three 1's and two 0's).

</details>

**Q2 (MCQ).** Q is a real orthogonal matrix and x = (3,4)^T. The value of ‖Qx‖ is (A) 1 (B) 5 (C) 7 (D) depends on Q

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Orthogonal matrices preserve length: ‖Qx‖ = ‖x‖ = √(9 + 16) = 5.

</details>

**Q3 (NAT).** The projection of b = (3,0,0) onto the line spanned by a = (1,2,2) has second component ___ (answer as a fraction/decimal).

<details><summary>Answer</summary>

**Answer:** 2/3 ≈ 0.667  
**Solution:** x̂ = a·b/a·a = 3/9 = 1/3; p = (1/3)(1,2,2) = (1/3, 2/3, 2/3).

</details>

**Q4 (MSQ).** P is a real symmetric matrix with P² = P. Which are true? (A) Eigenvalues are 0 or 1 (B) rank P = trace P (C) I - P is also a projection (D) det P = 1 always

<details><summary>Answer</summary>

**Answer:** (A), (B), (C)  
**Solution:** λ² = λ gives 0/1; trace = number of 1's = rank; (I - P)² = I - 2P + P = I - P and it is symmetric. (D) false: P = 0 or any proper projection is singular (det 0); det = 1 only for P = I.

</details>

**Q5 (NAT).** The least-squares line y = c + m t is fitted to (1,1), (2,2), (3,2). The slope m is ___.

<details><summary>Answer</summary>

**Answer:** 0.5  
**Solution:** Normal equations 3c + 6m = 5 and 6c + 14m = 11. Eliminating c: 2m = 1, so m = 1/2 (c = 2/3).

</details>

**Q6 (MCQ).** A is 5 × 4 with singular values 3, 2, 0, 0. Dimension of the null space of A^T is (A) 2 (B) 3 (C) 4 (D) 5

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** rank = 2 (non-zero σ). A^T is 4 × 5, so N(A^T) ⊂ R^5 has dimension m - r = 5 - 2 = 3. (N(A) has dimension 4 - 2 = 2.)

</details>

**Q7 (NAT).** A has singular values 5, 3, 1. The Frobenius-norm error (not squared) of the best rank-1 approximation is ___ (to 2 decimals).

<details><summary>Answer</summary>

**Answer:** 3.16  
**Solution:** error² = σ2² + σ3² = 9 + 1 = 10, error = √10 ≈ 3.16. (The spectral-norm error would be σ2 = 3.)

</details>

**Q8 (NAT).** For A = [[3,0],[4,5]], σ1² + σ2² = ___ and σ1 σ2 = ___ (enter the sum of the two numbers).

<details><summary>Answer</summary>

**Answer:** 65  
**Solution:** σ1² + σ2² = ‖A‖_F² = 9 + 0 + 16 + 25 = 50; σ1σ2 = |det A| = 15. Sum = 65. (Direct: eigenvalues of A^T A are 45 and 5.)

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
