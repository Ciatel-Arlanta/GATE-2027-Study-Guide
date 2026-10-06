# Linear Algebra — Checkpoint

Twelve mixed GATE-style questions, easy to hard. Attempt all without notes, then check. Chapters: [Vector spaces](vector-spaces.md) · [Matrices and determinants](matrices-and-determinants.md) · [Linear systems and LU](linear-systems-and-lu.md) · [Eigenvalues](eigenvalues-and-eigenvectors.md) · [Orthogonality, projections, SVD](orthogonality-projections-svd.md)

**Q1 (MSQ).** Which of the following are subspaces of R³? (A) {(x,y,z): x + y + z = 0} (B) {(x,y,z): x + y + z = 1} (C) {(x,y,z): xyz = 0} (D) {(x,y,z): x = 2z}

<details><summary>Answer</summary>

**Answer:** (A), (D)  
**Solution:** (A), (D) are planes through the origin defined by homogeneous linear equations: closed under + and scalars. (B) lacks the zero vector. (C) is a union of three coordinate planes: (1,1,0) + (0,0,1) = (1,1,1) is not in it, so not closed under addition.

</details>

**Q2 (NAT).** A = [[1,2,3],[2,4,6],[1,0,1]]. rank(A) + nullity(A) + (number of free variables in Ax = 0) = ___.

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** Row 2 = 2 · Row 1 and rows 1, 3 are independent, so rank 2. Nullity = 3 - 2 = 1 (rank-nullity), and free variables = nullity = 1. Sum = 2 + 1 + 1 = 4.

</details>

**Q3 (NAT).** A is 3 × 3 with det A = 5. det(2 · adj(A)) = ___.

<details><summary>Answer</summary>

**Answer:** 200  
**Solution:** det(adj A) = (det A)^(n-1) = 25. det(2 M) = 2³ det M = 8 · 25 = 200.

</details>

**Q4 (MCQ).** The system x + y + z = 6, x + 2y + 3z = 10, x + 2y + kz = m has infinitely many solutions iff (A) k = 3 and m = 10 (B) k = 3 and m ≠ 10 (C) k ≠ 3 (D) m = 10 only

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** R2 - R1: (0,1,2 | 4). R3 - R1: (0,1,k-1 | m-6). Then R3 - R2: (0,0,k-3 | m-10). If k ≠ 3: unique solution. If k = 3 and m = 10: last row is 0 = 0, rank 2 < 3, infinitely many (1 free variable). If k = 3, m ≠ 10: inconsistent.

</details>

**Q5 (NAT).** A 3 × 3 matrix has eigenvalues 1, 2, 4. det(A + I) + trace(A²) = ___.

<details><summary>Answer</summary>

**Answer:** 51  
**Solution:** A + I has eigenvalues 2, 3, 5: det = 30. A² has eigenvalues 1, 4, 16: trace = 21. Total 51.

</details>

**Q6 (MCQ).** [[2,a],[a,2]] is positive definite iff (A) a > 0 (B) |a| < 2 (C) a < 2 (D) |a| ≤ 2

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Leading minors: 2 > 0 and 4 - a² > 0 ⇒ |a| < 2. (Eigenvalues 2 ± a, both positive iff |a| < 2.) At |a| = 2 it is only semi-definite.

</details>

**Q7 (NAT).** For A = [[2,1],[4,5]] with Doolittle A = LU, the value of l21 + u22 is ___. (Also det A = ___ from U.)

<details><summary>Answer</summary>

**Answer:** 5 (and det A = 6)  
**Solution:** l21 = 4/2 = 2. u22 = 5 - 2 · 1 = 3. Sum 5. det A = u11 u22 = 2 · 3 = 6 (= 10 - 4 ✓).

</details>

**Q8 (NAT).** A is a 4 × 2 matrix with independent columns and P = A(A^T A)^-1 A^T. det(P + I) = ___.

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** P is a rank-2 projection in R⁴: eigenvalues 1, 1, 0, 0. P + I has eigenvalues 2, 2, 1, 1: det = 4.

</details>

**Q9 (NAT).** u = (1,2,2)^T, v = (3,4)^T. The only non-zero singular value of the 3 × 2 matrix u v^T is ___.

<details><summary>Answer</summary>

**Answer:** 15  
**Solution:** For a rank-1 matrix u v^T, σ1 = ‖u‖‖v‖ = 3 · 5 = 15. Check: ‖A‖_F² = Σ (u_i v_j)² = ‖u‖²‖v‖² = 225 = σ1².

</details>

**Q10 (NAT).** A = [[2,1],[0,3]]. A³ = a A + b I (Cayley–Hamilton). a + b = ___.

<details><summary>Answer</summary>

**Answer:** -11  
**Solution:** Characteristic polynomial λ² - 5λ + 6, so A² = 5A - 6I. A³ = 5A² - 6A = 5(5A - 6I) - 6A = 19A - 30I. a + b = 19 - 30 = -11. Check: 19 [[2,1],[0,3]] - 30I = [[8,19],[0,27]] = A³ ✓ (diagonal 2³ = 8, 3³ = 27).

</details>

**Q11 (MSQ).** A is a real 3 × 3 matrix with A² = A, trace A = 2. Which are always true? (A) rank A = 2 (B) det(A + I) = 4 (C) A is symmetric (D) det A = 1

<details><summary>Answer</summary>

**Answer:** (A), (B)  
**Solution:** Idempotent: eigenvalues 0/1, trace 2 so eigenvalues 1, 1, 0 and rank = trace = 2 (A). A + I: eigenvalues 2, 2, 1 so det = 4 (B). Idempotent need not be symmetric (e.g. [[1,1,0],[0,0,0],[0,0,1]], trace 2) so (C) is false. det A = 0 (D) false.

</details>

**Q12 (NAT).** A real symmetric 2 × 2 matrix has eigenvalue 1 with eigenvector (1,1) and eigenvalue 3. Its (1,2) entry is ___.

<details><summary>Answer</summary>

**Answer:** -1  
**Solution:** The eigenvector for 3 is orthogonal: (1,-1). A = 1 · (1/2)[[1,1],[1,1]] + 3 · (1/2)[[1,-1],[-1,1]] = [[2,-1],[-1,2]]. Check: trace 4, det 3, A(1,1) = (1,1) ✓. Entry (1,2) = -1.

</details>

**Scoring.** ≥ 80% (10+ correct): move on. 60–80%: redo the questions you missed and the practice sets of those chapters. < 60%: re-read. Mapping — Q1, Q2: [Vector spaces](vector-spaces.md); Q3, Q6, Q11: [Matrices and determinants](matrices-and-determinants.md); Q4, Q7: [Linear systems and LU](linear-systems-and-lu.md); Q5, Q10, Q12: [Eigenvalues](eigenvalues-and-eigenvectors.md); Q8, Q9: [Orthogonality, projections, SVD](orthogonality-projections-svd.md).

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
