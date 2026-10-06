# Systems of Linear Equations, Gaussian Elimination and LU Decomposition

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Systems of linear equations and solutions; Gaussian elimination; LU decomposition
> **Prerequisites:** [Vector spaces](vector-spaces.md) · [Matrices and determinants](matrices-and-determinants.md) · **Leads to:** [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md) · [Orthogonality, projections, SVD](orthogonality-projections-svd.md)

## Quick glance
- A system is Ax = b with A m × n. Form the **augmented matrix [A | b]** and row-reduce.
- **Consistency (Rouché–Capelli):** consistent iff rank(A) = rank([A|b]). Then: rank = n gives a **unique** solution; rank = r < n gives **infinitely many** with **n - r free parameters**; rank(A) < rank([A|b]) gives **no solution**.
- Homogeneous Ax = 0 is always consistent (x = 0). Non-trivial solutions exist iff rank < n; if m < n they always exist. Square case: iff det A = 0.
- General solution = particular solution + null-space of A: x = x_p + N(A).
- **Gaussian elimination:** forward elimination to echelon form, then back substitution. Cost Θ(n³) (about n³/3 multiplications); back substitution Θ(n²).
- **LU (Doolittle):** A = LU with L unit lower triangular (multipliers below the diagonal), U upper triangular (the echelon form). If row swaps are needed, PA = LU. Solving Ax = b costs two triangular solves: Ly = b, Ux = y, each Θ(n²).
- det A = product of U's diagonal (times (-1)^swaps).
- **#1 trap:** the number of free variables is n - rank (columns minus rank), and "unique solution" needs rank = number of **unknowns**, not number of equations.

## 1. Solution structure of Ax = b

**Intuition.** Each equation is a hyperplane; the solution is where they all meet: a point, a line/plane (infinite family), or nowhere (parallel/inconsistent).

**Column view.** Ax = b asks whether b is a combination of the columns of A, i.e. whether b lies in the column space.

| Condition | Meaning | Solutions |
|---|---|---|
| rank(A) < rank([A \| b]) | b not in column space | **none** (inconsistent: a row 0 = nonzero) |
| rank(A) = rank([A \| b]) = n | all columns pivot | **exactly one** |
| rank(A) = rank([A \| b]) = r < n | n - r free variables | **infinitely many**, an (n - r)-dimensional affine set |

Square n × n: unique solution iff det A ≠ 0. If det A = 0, then either none or infinitely many.

**Homogeneous system Ax = 0.** The solutions form a subspace of dimension n - r. **Trivial solution only iff rank = n.** More unknowns than equations (n > m) forces a non-trivial solution because r ≤ m < n.

**Structure theorem.** If x_p solves Ax = b then every solution is x_p + z, with z in N(A). Two particular solutions differ by a null-space vector.

## 2. Gaussian elimination

**Algorithm.**
1. Take the leftmost column that is not all zero below the current row; if its pivot position is 0, swap with a row below that has a nonzero entry.
2. Eliminate all entries **below** the pivot using R_i ← R_i - (a_ik / a_kk) R_k.
3. Move to the next row/column; repeat. The result is **row echelon form** (staircase).
4. Back substitution from the last nonzero row upwards. (Gauss–Jordan continues to eliminate **above** pivots and scales pivots to 1, giving **reduced** row echelon form, from which solutions are read off.)

**Worked example 1 — unique solution.** Solve x + y + z = 6; 2y + 5z = -4; 2x + 5y - z = 27.

[A | b] = [[1,1,1 | 6],[0,2,5 | -4],[2,5,-1 | 27]]
1. R3 ← R3 - 2R1: [0,3,-3 | 15].
2. R3 ← R3 - (3/2)R2: [0, 0, -3 - 7.5 | 15 + 6] = [0,0,-10.5 | 21].
3. Back substitution: z = 21 / (-10.5) = -2; 2y + 5(-2) = -4, so y = 3; x + 3 - 2 = 6, so x = 5.
4. Solution (5, 3, -2). Check eq 3: 10 + 15 + 2 = 27. rank A = rank [A|b] = 3 = n.

**Worked example 2 — infinitely many.** x + y + z = 6; x + 2y + 3z = 14; 2x + 3y + 4z = 20.
1. [[1,1,1 | 6],[1,2,3 | 14],[2,3,4 | 20]]. R2 - R1 = [0,1,2 | 8]; R3 - 2R1 = [0,1,2 | 8].
2. R3 - R2 = [0,0,0 | 0]. rank A = rank [A|b] = 2 < 3, consistent, **one free variable**.
3. Let z = t. y = 8 - 2t; x = 6 - y - z = 6 - 8 + 2t - t = t - 2.
4. Solution (x,y,z) = (-2, 8, 0) + t(1, -2, 1). The direction (1,-2,1) spans N(A); (-2,8,0) is a particular solution. Check with t = 0: -2 + 8 = 6 ✓; -2 + 16 = 14 ✓; -4 + 24 = 20 ✓.

**Worked example 3 — more unknowns than equations.** x1 + 2x2 - x3 + x4 = 3; 2x1 + 4x2 + x3 + 4x4 = 9; x1 + 2x2 + 2x3 + 3x4 = 6.
1. R2 - 2R1 = [0,0,3,2 | 3]; R3 - R1 = [0,0,3,2 | 3]. R3 - R2 = 0.
2. Rank 2, n = 4: two free variables, x2 = s and x4 = t (the non-pivot columns).
3. 3x3 + 2t = 3 so x3 = 1 - (2/3)t. x1 = 3 - 2s + x3 - t = 4 - 2s - (5/3)t.
4. General solution: (4, 0, 1, 0) + s(-2, 1, 0, 0) + t(-5/3, 0, -2/3, 1). Check eq 1: x1 + 2x2 - x3 + x4 = 4 - 2s - 5t/3 + 2s - 1 + 2t/3 + t = 3 ✓.

**Worked example 4 — parameters (classic GATE pattern).** For x + y + z = 6, x + 2y + 3z = 10, x + 2y + λz = μ:
- R2 - R1: y + 2z = 4. R3 - R2: (λ - 3) z = μ - 10.
- **λ ≠ 3:** unique solution (rank 3).
- **λ = 3, μ = 10:** last row 0 = 0, rank 2 = rank of augmented: infinitely many.
- **λ = 3, μ ≠ 10:** last row 0 = nonzero: **no solution**.

**Worked example 5 — another parameter pattern.** x + y + z = 1; x + 2y + 4z = λ; x + 4y + 10z = λ². Subtract: R2 - R1: y + 3z = λ - 1; R3 - R1: 3y + 9z = λ² - 1. R3 - 3R2: 0 = λ² - 3λ + 2 = (λ - 1)(λ - 2). So consistent iff λ = 1 or λ = 2; then rank = 2 and there are infinitely many solutions (one free variable). The coefficient matrix is singular for every λ, so it **never** has a unique solution.

## 3. Cramer's rule and the inverse

**Cramer's rule** (square, det A ≠ 0): x_i = det(A_i) / det(A), where A_i is A with column i replaced by b. Fine for 2 × 2 and 3 × 3; **far too slow** for large n (Θ(n · n!) by cofactors, Θ(n⁴) by elimination).

**Worked example 6.** 2x + y = 5; x + 3y = 10. det A = 5; x = det[[5,1],[10,3]] / 5 = 5/5 = 1; y = det[[2,5],[1,10]] / 5 = 15/5 = 3. Check: 2 + 3 = 5 ✓, 1 + 9 = 10 ✓.

**Inverse by Gauss–Jordan.** Row-reduce [A | I] to [I | A^-1]. If a zero row appears on the left, A is singular.

**Worked example 7.** A = [[2,1,1],[1,3,2],[1,0,0]].
1. [A | I]; swap R1 ↔ R3: rows (1,0,0 | 0,0,1), (1,3,2 | 0,1,0), (2,1,1 | 1,0,0).
2. R2 - R1 = (0,3,2 | 0,1,-1); R3 - 2R1 = (0,1,1 | 1,0,-2).
3. Swap R2 ↔ R3: (0,1,1 | 1,0,-2), (0,3,2 | 0,1,-1). R3 - 3R2 = (0,0,-1 | -3,1,5); scale R3 by -1: (0,0,1 | 3,-1,-5).
4. R2 - R3 = (0,1,0 | -2,1,3).
5. A^-1 = [[0,0,1],[-2,1,3],[3,-1,-5]]. det A = (1)(1)(-1) · (-1)^2 swaps = -1, consistent.

Cost: inversion Θ(n³); solving Ax = b by computing A^-1 then multiplying is wasteful versus elimination or LU.

## 4. LU decomposition

**Idea.** Gaussian elimination turns A into upper triangular U. Recording the multipliers used gives a lower triangular L with A = LU. **Elimination is factorisation.** Factor once (Θ(n³)), then solve for many right-hand sides b at Θ(n²) each.

**Doolittle form:** L has **1s on the diagonal**; U carries the pivots. (**Crout:** U has unit diagonal. **Cholesky:** for symmetric positive definite A = L L^T.)

**Existence without pivoting:** A has a unique Doolittle LU iff all **leading principal minors** (top-left k × k determinants, k < n) are nonzero. If a pivot would be zero (or tiny), swap rows: **PA = LU** with a permutation P.

**Doolittle formulas:** row i of U: u_ij = a_ij - Σ_{k<i} l_ik u_kj (j ≥ i); column j of L: l_ij = (a_ij - Σ_{k<j} l_ik u_kj) / u_jj (i > j).

**Worked example 8 — factor and solve.** A = [[2,1,1],[4,-6,0],[-2,7,2]].
1. Multipliers for column 1: l21 = 4/2 = 2; l31 = -2/2 = -1.
2. R2 - 2R1 = (0,-8,-2); R3 + R1 = (0,8,3).
3. Multiplier for column 2: l32 = 8/(-8) = -1. R3 + R2 = (0, 0, 3 - 2) = (0,0,1).

```text
    [ 1  0  0]        [2  1  1]
L = [ 2  1  0]    U = [0 -8 -2]
    [-1 -1  1]        [0  0  1]
```
Check row 3 of LU: (-1)(2,1,1) + (-1)(0,-8,-2) + (0,0,1) = (-2,-1,-1) + (0,8,2) + (0,0,1) = (-2,7,2) ✓.

Solve Ax = b with b = (5, -2, 9):
- Forward (Ly = b): y1 = 5; 2·5 + y2 = -2, so y2 = -12; -5 + 12 + y3 = 9, so y3 = 2.
- Backward (Ux = y): x3 = 2; -8x2 - 2·2 = -12 gives x2 = 1; 2x1 + 1 + 2 = 5 gives x1 = 1.
- x = (1, 1, 2). Check: 2 + 1 + 2 = 5, 4 - 6 + 0 = -2, -2 + 7 + 4 = 9 ✓.
- det A = 2 · (-8) · 1 = **-16** (L has det 1).

**Worked example 9 — pivoting needed.** A = [[0,1,2],[1,1,1],[2,1,3]]. a11 = 0, so plain LU fails (the leading 1 × 1 minor is zero).
1. Swap R1 ↔ R2: PA = [[1,1,1],[0,1,2],[2,1,3]], P = [[0,1,0],[1,0,0],[0,0,1]].
2. l21 = 0, l31 = 2: R3 - 2R1 = (0,-1,1). l32 = -1/1 = -1: R3 + R2 = (0,0,3).
3. L = [[1,0,0],[0,1,0],[2,-1,1]], U = [[1,1,1],[0,1,2],[0,0,3]]. Check LU row 3: 2(1,1,1) - (0,1,2) + (0,0,3) = (2,1,3) ✓.
4. det A = (-1)^1 · (1 · 1 · 3) = -3.

**Partial pivoting** picks the largest |entry| in the column as pivot to control rounding error. It is about numerical stability, not about exact arithmetic.

| Method | Cost (flops) | Use |
|---|---|---|
| Gaussian elimination to solve Ax = b | ~ (2/3) n³ (n³/3 multiplications) | one right-hand side |
| LU factorisation | ~ (2/3) n³ | many b's |
| Each extra solve with L, U | ~ 2n² | |
| Back/forward substitution | Θ(n²) | triangular systems |
| Inverse by Gauss–Jordan | ~ 2n³ | rarely needed |
| Cholesky (SPD) | ~ n³/3 | half of LU |
| Cramer's rule | exponential via cofactors | small cases only |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Consistent | rank A = rank [A \| b] | any solvability question |
| Unique | rank = n (number of unknowns) | counting |
| Free parameters | n - rank | infinite solutions |
| Homogeneous nontrivial | rank < n; always if m < n; det = 0 if square | |
| General solution | x_p + null space | |
| Cramer | x_i = det A_i / det A | tiny systems |
| LU | A = LU; PA = LU with pivoting; Ly = b, Ux = y | factorisation |
| Doolittle / Crout | L unit diagonal / U unit diagonal | |
| det via LU | ± product of diag(U) | |
| LU existence | all leading principal minors nonzero | no-pivot case |
| Costs | factor Θ(n³); solve Θ(n²) | complexity |
| Inverse | [A \| I] → [I \| A^-1] | |

## GATE traps
- **Rank comparison uses the augmented matrix.** If only A is reduced, a hidden row [0 0 0 | c], c ≠ 0 is missed.
- The number of free variables is n - r (columns - rank). Equations - rank is the number of *redundant equations*.
- A square system with det A = 0 does **not** necessarily have infinitely many solutions; it may have none (parameter questions: check the augmented rank).
- A homogeneous system never has "no solution".
- **Unique solution iff rank = n**, independent of m: an overdetermined (m > n) system can still be consistent with a unique solution.
- Multipliers go into L with their **sign as used** (R_i - l R_k records +l). Recording -l is a common slip.
- LU with unit-diagonal L is Doolittle; textbooks that put the unit diagonal in U are Crout. Read the question's convention.
- LU without pivoting can fail even for invertible A (e.g. [[0,1],[1,0]]); it exists iff the leading principal minors are nonzero.
- Gaussian elimination is Θ(n³); back substitution alone is Θ(n²). Solving after factorisation is Θ(n²).

## Connections
- [Vector spaces](vector-spaces.md) — rank, nullity and the four subspaces decide the number of solutions.
- [Matrices and determinants](matrices-and-determinants.md) — Cramer's rule, adjugate inverse, and det from U.
- [Eigenvalues and eigenvectors](eigenvalues-and-eigenvectors.md) — eigenvectors are non-trivial solutions of (A - λI)x = 0.
- [Orthogonality, projections, SVD](orthogonality-projections-svd.md) — an inconsistent overdetermined system is solved in the least-squares sense by A^T A x = A^T b.
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) — the normal equations are a linear system; solved by Cholesky/QR in practice.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — counting the cubic cost of elimination with nested loops.
- [Graphs (data structures)](../07-data-structures/graphs.md) — Gaussian elimination as fill-in on sparse graph matrices; PageRank as a linear system.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — systems over finite fields (GF(2)) appear in coding and parity problems.

## Practice

**Q1 (MCQ).** The system x + y = 2, 2x + 2y = 4 has (A) no solution (B) exactly one solution (C) infinitely many solutions (D) exactly two solutions

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Equation 2 = 2 × equation 1, so rank A = rank [A|b] = 1 < 2: one free variable, infinitely many solutions.

</details>

**Q2 (NAT).** For x + y + z = 6, x + 2y + 3z = 10, x + 2y + λz = μ, the system has no solution when λ = 3 and μ ≠ ___.

<details><summary>Answer</summary>

**Answer:** 10  
**Solution:** R3 - R2 gives (λ - 3) z = μ - 10. With λ = 3 this reads 0 = μ - 10; inconsistent iff μ ≠ 10.

</details>

**Q3 (NAT).** A homogeneous system has 5 equations in 7 unknowns and the coefficient matrix has rank 4. The dimension of the solution space is ___.

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** n - r = 7 - 4 = 3.

</details>

**Q4 (MSQ).** For the system Ax = b with A of size 4 × 3, which statements are correct? (A) If rank A = 3 and the system is consistent the solution is unique. (B) If rank A = 2 and the system is consistent there are infinitely many solutions. (C) The system is always consistent if rank A = 3. (D) A homogeneous version always has only the trivial solution.

<details><summary>Answer</summary>

**Answer:** (A), (B)  
**Solution:** (A) rank = n = 3. (B) 3 - 2 = 1 free variable. (C) false: b may be outside the 3-dimensional column space of R^4. (D) false when rank < 3.

</details>

**Q5 (NAT).** In the LU (Doolittle) factorisation of A = [[2,1,1],[4,-6,0],[-2,7,2]], the entry l32 of L is ___.

<details><summary>Answer</summary>

**Answer:** -1  
**Solution:** After step 1, R3 becomes (0,8,3) and R2 becomes (0,-8,-2); l32 = 8 / (-8) = -1.

</details>

**Q6 (NAT).** Using the same factorisation, the determinant of A is ___.

<details><summary>Answer</summary>

**Answer:** -16  
**Solution:** det L = 1, det U = 2 · (-8) · 1 = -16.

</details>

**Q7 (MCQ).** x + y + z = 1, x + 2y + 4z = λ, x + 4y + 10z = λ². The system is consistent for (A) all λ (B) λ = 1 or 2 only (C) λ = 0 only (D) no λ

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Eliminating gives 0 = λ² - 3λ + 2 = (λ - 1)(λ - 2) in the last row, with the first two rows independent (rank 2).

</details>

**Q8 (NAT).** The number of multiplications/divisions in Gaussian elimination to triangularise an n × n system grows as c·n³. The constant c is ___ (answer as a decimal to two places).

<details><summary>Answer</summary>

**Answer:** 0.33  
**Solution:** Step k eliminates (n - k) rows, each requiring about (n - k) multiplications: Σ (n - k)² ≈ n³/3.

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
