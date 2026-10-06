# Linear Algebra — Cheat Sheet

One page for final revision. Details: [Vector spaces](vector-spaces.md) · [Matrices and determinants](matrices-and-determinants.md) · [Linear systems and LU](linear-systems-and-lu.md) · [Eigenvalues](eigenvalues-and-eigenvectors.md) · [Orthogonality, projections, SVD](orthogonality-projections-svd.md)

## Vector spaces and rank
| Item | Fact |
|---|---|
| Subspace test | contains 0, closed under + and scalar multiplication |
| Span / basis / dim | all combinations / independent spanning set / number of basis vectors |
| Independent iff | rank [v1..vk] = k; more than n vectors in R^n are dependent |
| Rank | #pivots = dim C(A) = dim row space; rank A = rank A^T |
| Rank-nullity | rank + nullity = #columns n |
| Four subspaces (A is m × n, rank r) | C(A): r in R^m · N(A^T): m - r · C(A^T): r in R^n · N(A): n - r |
| Bases | column space: pivot columns of **original** A; row space: non-zero rows of echelon form |
| Rank inequalities | rank(AB) ≤ min; rank(A + B) ≤ rank A + rank B; rank(A^T A) = rank A |
| dim(U + W) | dim U + dim W - dim(U ∩ W) |
| dims | R^n: n · M(m × n): mn · polynomials deg ≤ d: d + 1 |

## Matrices and determinants
| Item | Fact |
|---|---|
| Product | (m × n)(n × p); AB ≠ BA; (AB)^T = B^T A^T; (AB)^-1 = B^-1 A^-1 |
| Row operations on det | swap: × -1 · scale row by c: × c · add multiple of a row: unchanged |
| det rules | det(AB) = det A det B · det A^T = det A · det(kA) = k^n det A · det A^-1 = 1/det A |
| Triangular / block triangular | product of diagonal / det(A11) det(A22) |
| Schur | det [[A,B],[C,D]] = det A · det(D - C A^-1 B) |
| Inverse | A^-1 = adj A / det A; det(adj A) = (det A)^(n-1); adj(adj A) = (det A)^(n-2) A |
| 2 × 2 inverse | (1/(ad - bc)) [[d,-b],[-c,a]] |
| Skew-symmetric, odd n | det = 0 |
| Vandermonde 3 × 3 | (b - a)(c - a)(c - b) |
| det(A + B) | ≠ det A + det B |

## Special matrices (eigenvalue facts)
| Type | Defining property | Eigenvalues / facts |
|---|---|---|
| Symmetric | A^T = A | real; orthonormal eigenvectors; A = QDQ^T |
| Skew-symmetric | A^T = -A | purely imaginary or 0; odd order is singular |
| Orthogonal | Q^T Q = I | \|λ\| = 1; det ±1; preserves length |
| Idempotent | A² = A | 0 or 1; rank = trace |
| Projection (orthogonal) | P² = P = P^T | 0/1; tr = rank; P = A(A^T A)^-1 A^T |
| Involutory | A² = I | ±1 |
| Nilpotent | A^k = 0 | all 0; det = 0 |
| Positive definite | x^T A x > 0 | all λ > 0; all leading minors > 0 |
| Triangular / diagonal | | diagonal entries |
| Stochastic (rows sum 1) | | 1 is an eigenvalue; \|λ\| ≤ 1 |
| Rank-1 uv^T | | u^T v once, 0 (n - 1) times |

## Quadratic forms
- x^T A x with A symmetric (symmetrise: (A + A^T)/2). PD: all λ > 0 ⇔ all leading principal minors > 0. PSD: all λ ≥ 0 (all principal minors ≥ 0). Indefinite: mixed signs.
- 2 × 2 [[a,b],[b,c]]: PD iff a > 0 and ac - b² > 0.

## Linear systems
| Case (A is m × n) | Condition |
|---|---|
| Consistent | rank A = rank [A \| b] |
| Unique | rank = n |
| Infinitely many | rank < n (n - rank free parameters) |
| None | rank A < rank [A \| b] |
| Homogeneous non-trivial | rank < n; always if m < n; det = 0 if square |
| General solution | x_p + N(A) |
| Cramer | x_i = det A_i / det A |
| Inverse by elimination | [A \| I] → [I \| A^-1] |

## Gaussian elimination and LU
- Forward elimination Θ(n³) (≈ n³/3 mults), back substitution Θ(n²). Partial pivoting: swap for the largest pivot.
- A = LU (Doolittle: L unit lower, multipliers l_ij = a_ij/u_jj; Crout: U unit upper). With swaps PA = LU. Exists without pivoting iff all leading principal minors ≠ 0.
- Solve: Ly = b (forward), Ux = y (back), each Θ(n²). det A = ± Π u_ii.

## Eigenvalues
| Item | Fact |
|---|---|
| Definition | Ax = λx, x ≠ 0; det(A - λI) = 0 |
| Σλ, Πλ | trace, determinant |
| 2 × 2 | λ² - (tr) λ + det = 0 |
| 3 × 3 | λ³ - tr λ² + (sum of principal 2 × 2 minors) λ - det = 0 |
| f(A) | eigenvalues f(λ): A^k → λ^k, A^-1 → 1/λ, A + cI → λ + c |
| A vs A^T, AB vs BA | same eigenvalues / same non-zero eigenvalues |
| AM, GM | 1 ≤ GM ≤ AM; GM = n - rank(A - λI) |
| Diagonalisable | iff GM = AM for all λ (n distinct λ is sufficient); A^k = P D^k P^-1 |
| Cayley–Hamilton | p(A) = 0; use for A^-1 and A^n |
| Real 2 × 2 complex λ | iff (tr)² < 4 det |
| Gershgorin | λ in a disc centre a_ii radius Σ_{j≠i} \|a_ij\| |

## Orthogonality, projections, least squares
| Item | Fact |
|---|---|
| Projection on vector | p = (a·b / a·a) a; P = a a^T / a^T a |
| Projection on C(A) | P = A(A^T A)^-1 A^T; P² = P = P^T; tr P = rank P |
| Normal equations | A^T A x̂ = A^T b; residual ⟂ C(A) |
| Gram–Schmidt | v_k = a_k - Σ (q_i · a_k) q_i → A = QR |
| Orthogonal Q | Q^-1 = Q^T; ‖Qx‖ = ‖x‖; det ±1 |
| Orthogonal subspaces | C(A^T) ⟂ N(A); C(A) ⟂ N(A^T) |

## SVD
| Item | Fact |
|---|---|
| Decomposition | A = U Σ V^T; U, V orthogonal; Σ diagonal ≥ 0 |
| Singular values | σ_i = √λ_i(A^T A) (= √λ_i(A A^T) for non-zero) |
| u_i | A v_i / σ_i |
| Rank | # non-zero σ_i |
| Norms | ‖A‖_F² = Σ σ_i² · ‖A‖_2 = σ_1 · \|det A\| = Π σ_i |
| Rank-k approximation | A_k = Σ_{i ≤ k} σ_i u_i v_i^T; errors σ_{k+1} (spectral), √Σ_{i>k} σ_i² (Frobenius) |
| Symmetric A | σ_i = \|λ_i\| |
| PCA | variance along v_i = σ_i²/(N - 1) for centred data |

## Remember
- rank = number of pivots = non-zero singular values; for idempotent/projection, rank = trace.
- Eigenvalue 0 ⇔ singular ⇔ det 0 ⇔ rank < n.
- Free variables = columns - rank.
- Repeated eigenvalue: check GM before declaring diagonalisable.
- Symmetric ⇒ always diagonalisable with orthogonal eigenvectors.

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
