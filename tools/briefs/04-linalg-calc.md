# Brief: 03-linear-algebra + 04-calculus-optimization

Verify determinants, eigenvalues, ranks, SVDs, limits and integrals with Python (numpy/sympy if available, else plain Python).

## 1) 03-linear-algebra/ (CS+DA; CS ~2-3 marks, DA ~8-12 marks)
Plan topics: Vectors and vector spaces; Subspaces; Linear dependence and independence; Matrices and matrix operations; Projection matrices; Orthogonal matrices; Idempotent matrices; Partition matrices and properties; Quadratic forms; Systems of linear equations and solutions; Gaussian elimination; Eigenvalues and eigenvectors; Determinants; Rank; Nullity; Projections; LU decomposition; Singular value decomposition.
Files: README.md, vector-spaces.md, matrices-and-determinants.md, linear-systems-and-lu.md, eigenvalues-and-eigenvectors.md, orthogonality-projections-svd.md, CHEATSHEET.md, CHECKPOINT.md.
Must cover:
- Vector-space axioms and the subspace test (examples and non-examples); span, basis, dimension; the four fundamental subspaces; rank via row echelon form; rank-nullity; rank facts (rank(AB) <= min, rank(A^T A) = rank(A)).
- Determinant properties (row-operation effects, det(AB), det(kA) = k^n det(A), triangular and block-triangular); inverse and adjugate.
- Special-matrix table (symmetric, skew-symmetric, orthogonal, idempotent, nilpotent, involutory, projection, positive definite) with eigenvalue facts for each.
- Partitioned multiplication, block inverse, Schur-complement determinant formula.
- Quadratic forms x^T A x: symmetrisation; definiteness via eigenvalues and leading principal minors; completing squares.
- Consistency of Ax = b via rank(A) vs rank([A|b]) (unique / infinite / none) with parametric solutions; Gaussian elimination step by step; Doolittle LU fully worked and when pivoting is needed.
- Eigenvalues: characteristic polynomial; trace = sum, det = product; eigenvalues of A^k, A^-1, A + cI, triangular matrices; Cayley-Hamilton (for A^-1, A^n); algebraic vs geometric multiplicity; diagonalisability; spectral theorem for symmetric matrices; 2x2/3x3 shortcuts; rank-1 matrices uv^T have eigenvalues u^T v and zeros.
- Orthogonal matrices (Q^T Q = I, length-preserving, det = +-1, rotation/reflection).
- Projection onto a vector and onto a column space, P = A(A^T A)^-1 A^T; P^2 = P, P^T = P, eigenvalues 0/1; least-squares normal equations (link ML linear regression); Gram-Schmidt briefly.
- SVD A = U Sigma V^T: singular values = sqrt of eigenvalues of A^T A; a fully worked 2x2 SVD; rank = number of non-zero singular values; low-rank approximation; link to PCA.

## 2) 04-calculus-optimization/ (CS+DA; ~2-5 marks)
Plan topics: Functions of a single variable; Limits; Continuity; Differentiability; Taylor series; Maxima and minima; Mean value theorem; Integration; Single-variable optimization.
Files: README.md, limits-continuity-differentiability.md, mean-value-theorems-and-taylor.md, maxima-minima-optimization.md, integration.md, CHEATSHEET.md, CHECKPOINT.md.
Must cover:
- Standard-limits table (sin x/x, (1+1/x)^x, (a^x - 1)/x, ...); L'Hopital for 0/0 and inf/inf and converting 0*inf, 1^inf, inf-inf, 0^0; left/right limits.
- Continuity and types of discontinuity; differentiability (|x| at 0, x sin(1/x)-style examples); differentiable implies continuous but not conversely; derivative-rules table.
- Rolle, Lagrange MVT (finding c), Cauchy MVT; Taylor/Maclaurin with a standard-expansions table and approximations.
- Increasing/decreasing, critical points, first and second derivative tests, global extrema on closed intervals (check endpoints), inflection points, convexity (link ML optimisation: gradient descent on a convex 1-D function such as squared loss); single-variable optimisation word problems.
- Integration: standard-integrals table, substitution, by parts (LIATE), definite-integral properties (King's property, odd/even functions), improper integrals and convergence, area under curves, integrals used in probability (x e^(-lambda x), the Gaussian integral), Leibniz rule briefly.

Connections: 02-probability-statistics (PDFs, expectations as integrals), 16-machine-learning (gradient descent, logistic loss, convexity), 03-linear-algebra <-> 16-machine-learning/dimensionality-reduction-pca.md and linear-and-logistic-regression.md, 08-algorithms/asymptotic-analysis.md (limits for comparing growth; integrals bounding sums, e.g. harmonic ~ ln n).
