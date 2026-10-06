# Eigenvalues and Eigenvectors

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Eigenvalues and eigenvectors
> **Prerequisites:** [Matrices and determinants](matrices-and-determinants.md) · [Vector spaces](vector-spaces.md) · **Leads to:** [Orthogonality, projections, SVD](orthogonality-projections-svd.md) · [PCA](../16-machine-learning/dimensionality-reduction-pca.md)

## Quick glance
- **Ax = λx, x ≠ 0.** λ is an eigenvalue, x an eigenvector. Eigenvalues are roots of the **characteristic polynomial** det(A - λI) = 0.
- **Sum of eigenvalues = trace(A); product = det(A).** For 2 × 2: λ² - (trace) λ + det = 0.
- Eigenvalues of A^k: λ^k; of A^-1: 1/λ; of A + cI: λ + c; of p(A): p(λ); of A^T: same as A. **Triangular/diagonal matrix: eigenvalues are the diagonal entries.**
- **Eigenvectors of distinct eigenvalues are independent.** Algebraic multiplicity (AM) = multiplicity as a root; geometric multiplicity (GM) = dim of eigenspace N(A - λI); **1 ≤ GM ≤ AM**.
- **Diagonalisable iff GM = AM for every eigenvalue** (iff n independent eigenvectors). Then A = P D P^-1 and A^k = P D^k P^-1. n distinct eigenvalues is sufficient, not necessary.
- **Cayley–Hamilton:** every matrix satisfies its own characteristic equation. Use it for A^-1 and high powers.
- **Spectral theorem:** a real symmetric matrix has real eigenvalues and an orthonormal eigenbasis: A = Q D Q^T.
- Rank-1 matrix u v^T: eigenvalues u^T v (once) and 0 (n - 1 times).
- **#1 trap:** eigenvalue 0 does not make the eigen-decomposition fail; but a repeated eigenvalue may make the matrix non-diagonalisable (check GM).

## 1. The idea

**Intuition.** A matrix usually rotates and stretches vectors. An eigenvector is a direction that is **only stretched** (or flipped) by the map: A sends x to λx on the same line. Finding such directions turns a complicated map into independent scalings along those directions.

**Definition.** λ is an eigenvalue of the n × n matrix A if Ax = λx has a non-zero solution x. Equivalent statements: (A - λI) is singular; det(A - λI) = 0; N(A - λI) ≠ {0}. The set of all solutions of (A - λI)x = 0 is the **eigenspace** E_λ (a subspace, including 0).

**Procedure.**
1. Form det(A - λI) = 0 (the characteristic equation, degree n).
2. Solve for λ.
3. For each λ solve (A - λI)x = 0 (row reduce; free variables give eigenvectors).

**Characteristic polynomial.** det(λI - A) = λ^n - (trace A) λ^(n-1) + (sum of principal 2 × 2 minors) λ^(n-2) - ... + (-1)^n det A. For 3 × 3: λ³ - tr·λ² + (M11 + M22 + M33) λ - det, where M_ii are the cofactor-minors (the 2 × 2 determinants after deleting row i and column i).

**Worked example 1 — 2 × 2.** A = [[1,2],[3,2]].
1. trace = 3, det = 2 - 6 = -4. Characteristic equation λ² - 3λ - 4 = 0 so (λ - 4)(λ + 1) = 0: **λ = 4, -1**.
2. λ = 4: (A - 4I) = [[-3,2],[3,-2]]; -3x + 2y = 0 gives x = (2, 3).
3. λ = -1: (A + I) = [[2,2],[3,3]]; x + y = 0 gives x = (1, -1).
4. Checks: sum 4 + (-1) = 3 = trace; product -4 = det. A(2,3) = (8,12) = 4(2,3) ✓.

**Worked example 2 — 3 × 3 symmetric with a repeated root.** A = [[2,1,1],[1,2,1],[1,1,2]].
1. tr = 6; principal 2 × 2 minors: (4 - 1) + (4 - 1) + (4 - 1) = 9; det = 2(4 - 1) - 1(2 - 1) + 1(1 - 2) = 6 - 1 - 1 = 4.
2. λ³ - 6λ² + 9λ - 4 = 0. Try λ = 1: 1 - 6 + 9 - 4 = 0 ✓. Divide: (λ - 1)(λ² - 5λ + 4) = (λ - 1)(λ - 1)(λ - 4). **Eigenvalues 1, 1, 4.**
3. λ = 4: A - 4I = [[-2,1,1],[1,-2,1],[1,1,-2]]; rows sum to 0, so x = (1,1,1) and the eigenspace is one-dimensional (rank 2).
4. λ = 1: A - I = all ones matrix, rank 1, so nullity 2: x + y + z = 0, giving two independent eigenvectors (-1,1,0), (-1,0,1). **AM = GM = 2.** A is diagonalisable.
5. Note (1,1,1) is orthogonal to both λ = 1 vectors (spectral theorem). The two λ = 1 vectors are not orthogonal to each other; Gram–Schmidt inside the eigenspace fixes this.

Shortcut: this A = J + I where J is the all-ones matrix. J has eigenvalues 3, 0, 0, so A has 4, 1, 1.

## 2. Properties

| Fact | Statement |
|---|---|
| Trace / determinant | Σ λ_i = tr A; Π λ_i = det A |
| Powers | A^k has eigenvalues λ^k (same eigenvectors) |
| Inverse | A^-1 has 1/λ (A invertible iff no zero eigenvalue) |
| Shift / scale | A + cI → λ + c; cA → cλ |
| Polynomial | p(A) → p(λ) |
| Transpose | A and A^T have the same eigenvalues (different eigenvectors) |
| Similarity | B = P^-1 A P has the same eigenvalues, trace, det, rank |
| Triangular / diagonal | eigenvalues = diagonal entries |
| AB vs BA | same non-zero eigenvalues |
| Block triangular | union of the diagonal blocks' eigenvalues |
| Real matrix | complex eigenvalues come in conjugate pairs |
| Real symmetric | all eigenvalues real; eigenvectors of distinct eigenvalues orthogonal |
| Skew-symmetric real | purely imaginary (or 0) |
| Orthogonal | |λ| = 1 |
| Idempotent / involutory / nilpotent | {0,1} / {±1} / {0} |
| Row sums all s | s is an eigenvalue, eigenvector (1,...,1) |
| Stochastic | eigenvalue 1; all |λ| ≤ 1 |
| Singular | 0 is an eigenvalue; multiplicity of 0 ≥ nullity |
| Gershgorin | every eigenvalue lies in some disc |a_ii - λ| ≤ Σ_{j≠i} |a_ij| |

**Rank-1 matrices.** For M = u v^T: M x = u (v^T x). Any eigenvector with λ ≠ 0 must be a multiple of u: M u = (v^T u) u. So **eigenvalues are v^T u (once) and 0 (n - 1 times)**. Example: u = (1,2,3), v = (4,5,6): M = u v^T has λ = 4 + 10 + 18 = 32, 0, 0. The trace is 4 + 10 + 18 = 32 ✓.

**Eigenvalues from given ones.** If A (3 × 3) has eigenvalues 1, 2, 3 then: det(A²+I) = (1 + 1)(4 + 1)(9 + 1) = 100; tr(A^-1) = 1 + 1/2 + 1/3 = 11/6; det(A) = 6; the eigenvalues of A² - 3A + 2I are λ² - 3λ + 2 = 0, 0, 2.

## 3. Multiplicity and diagonalisability

- **AM(λ)** = multiplicity of λ in the characteristic polynomial.
- **GM(λ)** = dim E_λ = n - rank(A - λI).
- Always 1 ≤ GM ≤ AM. Σ AM = n.
- **A is diagonalisable ⇔ GM(λ) = AM(λ) for every λ ⇔ A has n linearly independent eigenvectors.**
- n distinct eigenvalues ⇒ diagonalisable (all AM = 1).
- A matrix with GM < AM for some λ is **defective**.

**Worked example 3 — defective.** A = [[2,1],[0,2]]. λ = 2 with AM 2. A - 2I = [[0,1],[0,0]] has rank 1 so GM = 1. Only one independent eigenvector (1, 0): **not diagonalisable**. Likewise B = [[2,0,0],[1,2,0],[0,0,3]]: λ = 2 has AM 2, B - 2I has rank 2 so GM = 1, defective. By contrast the identity I has AM = GM = n.

**Diagonalisation.** If P has the eigenvectors as columns and D = diag(λ_i) then **A = P D P^-1** and **A^k = P D^k P^-1**.

**Worked example 4 — A^n.** For A = [[1,2],[3,2]] (λ = 4, -1; vectors (2,3), (1,-1)). P = [[2,1],[3,-1]] has det -5; P^-1 = (1/5)[[1,1],[3,-2]]. Then A^n = P diag(4^n, (-1)^n) P^-1. Check n = 2 directly: A² = [[7,6],[9,10]]. Via P D² P^-1: P D² = [[2·16, 1·1],[3·16, -1·1]] = [[32,1],[48,-1]]; times (1/5)[[1,1],[3,-2]] gives (1/5)[[32 + 3, 32 - 2],[48 - 3, 48 + 2]] = (1/5)[[35,30],[45,50]] = [[7,6],[9,10]] ✓.

Uses: closed forms for recurrences (Fibonacci via [[1,1],[1,0]]), Markov chain steady states, differential equations x' = Ax.

## 4. Cayley–Hamilton theorem

**Statement.** If p(λ) = det(λI - A) then **p(A) = 0**.

**Worked example 5 — inverse.** A = [[1,2],[3,4]]: trace 5, det -2, so p(λ) = λ² - 5λ - 2. Hence A² - 5A - 2I = 0, so A(A - 5I) = 2I and **A^-1 = (A - 5I)/2** = (1/2)[[-4,2],[3,-1]] = [[-2,1],[3/2,-1/2]] ✓ (check with the 2 × 2 formula: (1/-2)[[4,-2],[-3,1]]).

**Worked example 6 — high power.** Reduce powers using A² = 5A + 2I:
- A³ = 5A² + 2A = 5(5A + 2I) + 2A = 27A + 10I
- A⁴ = 27A² + 10A = 27(5A + 2I) + 10A = 145A + 54I
- A⁵ = 145A² + 54A = 145(5A + 2I) + 54A = 779A + 290I
- So A⁵ = 779 [[1,2],[3,4]] + 290 I = [[1069, 1558],[2337, 3406]].

**Worked example 7 — expression.** For A = [[1,2],[3,2]], A² = 3A + 4I, so A² - 3A + I = 4I + I = 5I. Every eigenvalue of this matrix is 5 (matches λ² - 3λ + 1 at λ = 4 and -1: both 5).

## 5. Symmetric matrices: the spectral theorem

For real **symmetric** A:
- all eigenvalues are real;
- eigenvectors for different eigenvalues are **orthogonal**;
- A is always diagonalisable with an **orthonormal** eigenbasis: **A = Q D Q^T** with Q orthogonal (Q^-1 = Q^T);
- AM = GM always (no defective symmetric matrices);
- A = Σ λ_i q_i q_i^T (sum of rank-1 projections).

**Proof idea for orthogonality.** If Ax = λx, Ay = μy, then λ(x·y) = (Ax)·y = x·(Ay) = μ(x·y), so (λ - μ)(x·y) = 0.

**Worked example 8.** [[2,1,1],[1,2,1],[1,1,2]]: unit eigenvectors q1 = (1,1,1)/√3 (λ = 4), q2 = (1,-1,0)/√2 and q3 = (1,1,-2)/√6 (λ = 1; the latter is the λ = 1 vector orthogonal to q2). Then Q = [q1 q2 q3] is orthogonal and A = Q diag(4,1,1) Q^T.

**Definiteness link.** Symmetric A is positive definite iff all eigenvalues > 0 (see [quadratic forms](matrices-and-determinants.md)). **Rayleigh quotient:** λ_min ≤ x^T A x / x^T x ≤ λ_max.

## 6. Complex eigenvalues and other cases

- A rotation by θ, [[cos θ, -sin θ],[sin θ, cos θ]], has eigenvalues e^(±iθ). For θ = 90°, [[0,-1],[1,0]]: λ² + 1 = 0 so λ = ±i. A real matrix of odd order always has at least one real eigenvalue.
- A 2 × 2 real matrix has complex eigenvalues iff (trace)² < 4 det.
- A projection has eigenvalues 0 and 1; a reflection has +1 and -1.
- **Power iteration** (idea): repeatedly multiplying a vector by A converges in direction to the eigenvector of the largest |λ| (the dominant eigenvector); this is how PageRank is computed.

**Worked example 9 — Markov chain.** P = [[0.5,0.5],[0.2,0.8]] (rows sum to 1). tr = 1.3, det = 0.4 - 0.1 = 0.3 so λ² - 1.3λ + 0.3 = 0: λ = 1 and 0.3. Steady state: π P = π, πᵀ is the eigenvector of Pᵀ for λ = 1: π1 = 0.2 π... solve 0.5π1 + 0.2π2 = π1 so 0.2π2 = 0.5π1; π = (2/7, 5/7). Convergence rate is governed by the second eigenvalue 0.3.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Eigen equation | det(A - λI) = 0; (A - λI)x = 0 | find λ, x |
| Sum / product | Σλ = tr A; Πλ = det A | shortcuts, checks |
| 2 × 2 | λ² - tr·λ + det = 0 | quick |
| 3 × 3 | λ³ - tr λ² + (Σ principal minors) λ - det = 0 | |
| f(A) | eigenvalues f(λ) | A^k, A^-1, A + cI |
| Rank-1 uv^T | eigenvalues u^T v, 0, ..., 0 | |
| GM | n - rank(A - λI) | diagonalisability |
| Diagonalisable iff | GM = AM for all λ | |
| A^k | P D^k P^-1 | powers |
| Cayley–Hamilton | p(A) = 0 | A^-1, A^n |
| Spectral theorem | symmetric: A = Q D Q^T | |
| Triangular | diagonal entries | |
| Orthogonal | |λ| = 1 | |

## GATE traps
- **Eigenvector must be non-zero**; the zero vector solves every equation.
- Sum of eigenvalues is the trace **with multiplicity**; a repeated root counts twice.
- A^-1 exists iff 0 is not an eigenvalue; the eigenvalues of A^-1 are the **reciprocals**.
- A and A^T share eigenvalues but **not eigenvectors**; AB and BA share non-zero eigenvalues.
- Repeated eigenvalue ≠ not diagonalisable. Check GM = n - rank(A - λI) (identity is diagonalisable, [[2,1],[0,2]] is not).
- Triangular matrices give eigenvalues instantly, but only **diagonal** gives eigenvectors instantly.
- Eigenvalues of A + B are **not** λ_A + λ_B (only when they share eigenvectors/commute).
- Similar matrices share eigenvalues, but matrices with the same eigenvalues need not be similar.
- Cayley–Hamilton: substitute the matrix and write the **constant term times I**.
- For a rank-1 matrix, count: n - 1 zero eigenvalues, one eigenvalue equal to the trace.

## Connections
- [Vector spaces](vector-spaces.md) — eigenspace = null space of A - λI; GM = nullity; rank deficiency means eigenvalue 0.
- [Matrices and determinants](matrices-and-determinants.md) — det and trace as product and sum; definiteness; special-matrix eigenvalue facts.
- [Orthogonality, projections, SVD](orthogonality-projections-svd.md) — singular values are square roots of the eigenvalues of A^T A; projections have eigenvalues 0/1.
- [PCA](../16-machine-learning/dimensionality-reduction-pca.md) — principal components are eigenvectors of the covariance matrix; eigenvalues are variances.
- [Probability: random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — covariance matrix is symmetric PSD (non-negative eigenvalues).
- [Probabilistic reasoning](../17-artificial-intelligence/probabilistic-reasoning.md) — Markov chains and steady states via eigenvalue 1.
- [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) — a recurrence as a matrix power; roots of the characteristic equation are eigenvalues.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — adjacency and Laplacian spectra (the number of zero Laplacian eigenvalues counts components).
- [Clustering](../16-machine-learning/clustering.md) — spectral clustering uses Laplacian eigenvectors.

## Practice

**Q1 (NAT).** The eigenvalues of [[4,2],[1,3]] are λ1 and λ2. The value of λ1 λ2 + λ1 + λ2 is ___.

<details><summary>Answer</summary>

**Answer:** 19  
**Solution:** det = 12 - 2 = 10, trace = 7. Sum = 7, product = 10, so 10 + 7 = 17. Recheck: eigenvalues are roots of λ² - 7λ + 10, i.e. 2 and 5; 2·5 + 2 + 5 = **17**. (The answer is 17.)

</details>

**Q2 (MCQ).** Matrix A has eigenvalues 1, 2, 3. det(A² + A) equals (A) 36 (B) 72 (C) 144 (D) 24

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Eigenvalues of A² + A are λ² + λ: 2, 6, 12. Product = 144... recompute: 1 + 1 = 2; 4 + 2 = 6; 9 + 3 = 12; 2 · 6 · 12 = 144. So the answer is **(C) 144**.

</details>

**Q3 (NAT).** u = (1,2,3)^T and v = (4,5,6)^T. The non-zero eigenvalue of u v^T is ___.

<details><summary>Answer</summary>

**Answer:** 32  
**Solution:** Rank 1 with eigenvalues v^T u = 4 + 10 + 18 = 32 and two zeros (trace = 32).

</details>

**Q4 (MCQ).** Which of the following matrices is **not** diagonalisable over R? (A) [[2,0],[0,2]] (B) [[2,1],[0,2]] (C) [[1,2],[2,1]] (D) [[3,1],[0,2]]

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** (A) already diagonal. (C) symmetric. (D) distinct eigenvalues 3, 2. (B) has λ = 2 with AM 2 but A - 2I = [[0,1],[0,0]] has rank 1, so GM = 1 < 2.

</details>

**Q5 (NAT).** For A = [[1,2],[3,4]], the (1,1) entry of A^-1 computed through Cayley–Hamilton, A^-1 = (A - 5I)/2, is ___.

<details><summary>Answer</summary>

**Answer:** -2  
**Solution:** (A - 5I) = [[-4,2],[3,-1]]; divided by 2 the (1,1) entry is -2. Check by direct inverse: (1/(4 - 6))·4 = -2.

</details>

**Q6 (MSQ).** A is a real symmetric 3 × 3 matrix with eigenvalues 1, 1, 5. Which are true? (A) A is diagonalisable (B) There exist two orthogonal eigenvectors for λ = 1 (C) A is positive definite (D) det(A) = 5

<details><summary>Answer</summary>

**Answer:** (A), (B), (C), (D)  
**Solution:** Symmetric means orthonormal eigenbasis, so (A), (B). All eigenvalues > 0 gives PD (C). det = 1 · 1 · 5 = 5 (D).

</details>

**Q7 (NAT).** A 3 × 3 matrix has trace 6, det 6 and the eigenvalue 1. The other two eigenvalues' sum of squares is ___.

<details><summary>Answer</summary>

**Answer:** 13  
**Solution:** The other two eigenvalues a, b satisfy a + b = 5 and ab = 6, so {a,b} = {2,3} and a² + b² = 25 - 12 = 13.

</details>

**Q8 (MCQ).** A = [[3,1,1],[1,3,1],[1,1,3]]. The largest eigenvalue and the dimension of its eigenspace are (A) 5 and 1 (B) 5 and 2 (C) 3 and 3 (D) 2 and 2

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** A = 2I + J (J = all ones, eigenvalues 3, 0, 0), so eigenvalues 5, 2, 2. For λ = 5, A - 5I has rank 2, so the eigenspace has dimension 1, spanned by (1,1,1).

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
