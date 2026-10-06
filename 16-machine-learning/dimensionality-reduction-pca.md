# Dimensionality reduction and principal component analysis (PCA)

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Dimensionality reduction; Principal component analysis
> **Prerequisites:** [Eigenvalues and eigenvectors](../03-linear-algebra/eigenvalues-and-eigenvectors.md) · [Orthogonality, projections, SVD](../03-linear-algebra/orthogonality-projections-svd.md) · [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) (covariance) · **Leads to:** [CHECKPOINT](CHECKPOINT.md)

## Quick glance

- **Dimensionality reduction**: map $d$-dimensional data to $m<d$ dimensions keeping as much structure as possible. Reasons: compress, visualise, denoise, fight the curse of dimensionality, remove correlated features.
- **PCA steps**: (1) centre the data (subtract the mean; optionally standardise), (2) covariance matrix $S=\frac1{n-1}X_c^TX_c$, (3) eigen-decomposition $Sv=\lambda v$, (4) sort eigenvalues descending, (5) project: $Z=X_cV_m$.
- **Principal components = eigenvectors of the covariance matrix**; eigenvalue $\lambda_i$ = variance of the data along component $i$. Components are orthogonal.
- **Variance explained** by PC $i$: $\lambda_i/\sum_j\lambda_j$; $\sum_j\lambda_j=\operatorname{trace}(S)$.
- **PCA via SVD**: $X_c=U\Sigma V^T$; columns of $V$ = PCs; $\lambda_i=\sigma_i^2/(n-1)$; scores $=U\Sigma$.
- PC1 is the direction of **maximum variance** = the line with **minimum squared reconstruction (perpendicular) error**.
- PCA is **unsupervised** (ignores labels) and linear; LDA is its supervised relative.
- #1 trap: forgetting to centre, or using the wrong divisor ($n$ vs $n-1$) — the eigenvectors and variance ratios are unchanged but the eigenvalues scale.

## 1. Why reduce dimensions?

- **Curse of dimensionality**: with $d$ large, data are sparse, distances concentrate, models need exponentially more samples.
- **Redundancy**: correlated features carry the same information (height in cm and in inches).
- **Visualisation**: 2-D or 3-D plots.
- **Noise reduction** and cheaper storage/training.

Two flavours: **feature selection** (keep a subset of the original columns) and **feature extraction** (build new features as combinations: PCA, LDA, autoencoders). PCA is feature extraction.

## 2. The idea of PCA

**Intuition.** A cloud of points is elongated along some direction. Rotate the axes so the first axis lies along the longest direction (most variance), the second along the longest direction orthogonal to it, and so on. Keep the first few axes; the dropped axes have small spread, so little is lost.

**Derivation sketch.** For centred data and unit vector $v$, the variance of the projections $Xv$ is $v^TSv$. Maximise $v^TSv$ subject to $v^Tv=1$. Lagrangian: $Sv=\lambda v$. So $v$ is an eigenvector and the variance achieved is $\lambda$; the best choice is the eigenvector with the **largest** eigenvalue. The next component maximises variance among directions orthogonal to the first (eigenvectors of a symmetric matrix are orthogonal), and so on. See [eigenvalues](../03-linear-algebra/eigenvalues-and-eigenvectors.md).

**Equivalent view.** Maximising variance of the projection = minimising the mean squared distance between each point and its projection (Pythagoras: total variance is fixed).

## 3. The algorithm

1. **Centre**: $x_c = x-\bar x$ (columnwise mean). If features have different scales/units, also divide by standard deviations (then $S$ is the correlation matrix).
2. **Covariance**: $S=\dfrac1{n-1}X_c^TX_c$ (a $d\times d$ symmetric positive semi-definite matrix). Diagonal = variances, off-diagonal = covariances.
3. **Eigen-decomposition**: $S=V\Lambda V^T$, $\lambda_1\ge\dots\ge\lambda_d\ge0$.
4. **Choose $m$**: smallest $m$ with $\sum_{i\le m}\lambda_i/\sum\lambda_j\ge$ threshold (e.g. 95%), or the elbow of the scree plot, or by cross-validation downstream.
5. **Project**: $Z=X_cV_m$ ($n\times m$). **Reconstruct**: $\hat X=ZV_m^T+\bar x$. Total squared reconstruction error $=(n-1)\sum_{i>m}\lambda_i$.

**Worked example A (exact, 2-D).** Four points $(2,0),(0,2),(3,1),(1,3)$.
1. Mean $=(1.5,1.5)$. Centred: $(0.5,-1.5),(-1.5,0.5),(1.5,-0.5),(-0.5,1.5)$.
2. $S_{11}=\frac{0.25+2.25+2.25+0.25}3=\frac53$; $S_{22}=\frac53$; $S_{12}=\frac{-0.75-0.75-0.75-0.75}3=-1$. $S=\begin{pmatrix}5/3&-1\\-1&5/3\end{pmatrix}$.
3. Eigenvalues: $\det(S-\lambda I)=(\tfrac53-\lambda)^2-1=0\Rightarrow\lambda=\tfrac53\pm1=\tfrac83,\ \tfrac23$.
   - $\lambda_1=8/3$: $(S-\lambda I)v=0\Rightarrow-v_1-v_2=0$: $v_1=\tfrac1{\sqrt2}(1,-1)$.
   - $\lambda_2=2/3$: $v_2=\tfrac1{\sqrt2}(1,1)$. Orthogonal ✓.
4. Variance explained: PC1 $=\frac{8/3}{10/3}=80\%$, PC2 $=20\%$. Trace check: $\frac53+\frac53=\frac{10}3=\frac83+\frac23$ ✓.
5. Scores on PC1 $=x_c\cdot v_1$: $\frac{0.5+1.5}{\sqrt2}=\sqrt2$, $\frac{-1.5-0.5}{\sqrt2}=-\sqrt2$, $\sqrt2$, $-\sqrt2$. Variance $=\frac{4\cdot2}{3}=\frac83$ ✓. Scores on PC2 are $\mp\frac1{\sqrt2}$ with variance $\frac{4\cdot0.5}{3}=\frac23$ ✓.
6. Keeping only PC1 reconstructs each point on the line through the mean with direction $(1,-1)$; residual squared error $=(n-1)\lambda_2=3\cdot\frac23=2$.

**Worked example B (given a covariance matrix).** $S=\begin{pmatrix}4&2\\2&3\end{pmatrix}$.
1. Characteristic equation: $\lambda^2-\operatorname{tr}(S)\lambda+\det S=\lambda^2-7\lambda+8=0$ (trace $=7$, det $=12-4=8$).
2. $\lambda=\dfrac{7\pm\sqrt{49-32}}2=\dfrac{7\pm4.123}2=5.562,\ 1.438$.
3. For $\lambda_1=5.562$: $(4-5.562)v_1+2v_2=0\Rightarrow v_2=0.781v_1$; normalised $v=(0.788,\,0.615)$.
4. Variance explained by PC1: $5.562/7=\mathbf{79.4\%}$.
5. Shortcut checks: sum of eigenvalues = trace $=7$ ✓; product = determinant $=5.562\times1.438=8$ ✓.

**Worked example C (3-D, block structure).** $S=\begin{pmatrix}2&0&0\\0&3&1\\0&1&3\end{pmatrix}$. The block $\begin{pmatrix}3&1\\1&3\end{pmatrix}$ has eigenvalues $4,2$ (vectors $(0,1,1)/\sqrt2$ and $(0,1,-1)/\sqrt2$); the first coordinate is uncorrelated with the rest and has variance $2$. Eigenvalues $4,2,2$ (trace $=8$ ✓). PC1 $=(0,1,1)/\sqrt2$ explains $4/8=50\%$. Repeated eigenvalue $2$: any orthonormal basis of that eigenspace is valid, so PC2/PC3 are not unique.

## 4. PCA via SVD

For centred data $X_c$ ($n\times d$): $X_c=U\Sigma V^T$ (thin SVD). Then

$$S=\frac1{n-1}X_c^TX_c=V\frac{\Sigma^2}{n-1}V^T$$

so **right singular vectors $V$ = principal directions** and $\lambda_i=\sigma_i^2/(n-1)$. Principal-component scores $=X_cV=U\Sigma$. Using SVD avoids forming $X_c^TX_c$ (better numerically). In example A: singular values of $X_c$ are $\sqrt8=2.828$ and $\sqrt2=1.414$; $\sigma^2/3=2.667,\ 0.667$ ✓. See [SVD](../03-linear-algebra/orthogonality-projections-svd.md).

## 5. Properties and cautions

| Fact | Consequence |
| --- | --- |
| Components are orthonormal, uncorrelated scores | covariance of $Z$ is diagonal $=\text{diag}(\lambda_i)$ |
| Number of PCs $\le\min(n-1,d)$ with nonzero variance | rank of centred data |
| Keeping all $d$ components | lossless rotation; 100% variance |
| Scale-dependent | standardise features with different units, otherwise the large-variance feature dominates |
| Unsupervised | the max-variance direction need not separate classes |
| Linear | cannot unroll curved manifolds (kernel PCA etc. can) |
| Sign of a PC is arbitrary | $v$ and $-v$ are both valid |

**PCA vs LDA.** PCA maximises total variance and ignores labels; LDA maximises class separation (between-class over within-class scatter) and needs labels. For two elongated parallel classes side by side, PC1 follows the elongation (useless for classifying) while the LDA direction is across the gap. See [classification methods](classification-methods.md).

**Choosing $m$.** Example: eigenvalues $(6,3,0.6,0.3,0.1)$: total $10$; cumulative ratios $0.6,0.9,0.96,0.99,1.0$. To keep $\ge95\%$ take $m=3$.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Covariance | $S=\frac1{n-1}X_c^TX_c$ | step 2 |
| PCs | eigenvectors of $S$, largest $\lambda$ first | all |
| Variance along PC $i$ | $\lambda_i$ | explained variance |
| Explained ratio | $\lambda_i/\sum\lambda_j$, $\sum\lambda_j=\text{tr}(S)$ | choose $m$ |
| $2\times2$ eigenvalues | $\lambda^2-\text{tr}\lambda+\det=0$ | hand calculation |
| SVD link | $\lambda_i=\sigma_i^2/(n-1)$ | SVD questions |
| Projection | $Z=X_cV_m$ | reduced data |
| Reconstruction error | $(n-1)\sum_{i>m}\lambda_i$ | error questions |
| Max PCs with variance | $\min(n-1,d)$ | rank question |

## GATE traps

- **Centre first.** Without centring the first "component" mostly points at the mean.
- Eigenvalue of the covariance matrix = **variance** along the component, not standard deviation. Singular values give std-like quantities ($\sigma_i/\sqrt{n-1}$).
- Variance explained uses eigenvalues of $S$ (or squared singular values), not the eigenvector entries.
- $n$ vs $n-1$ divisor changes eigenvalues by a constant factor, not the eigenvectors or the ratios; use whatever the question states.
- PCA directions need not discriminate classes; do not confuse with LDA.
- Standardising changes the PCs (covariance matrix becomes the correlation matrix).
- Principal components are orthogonal, the original features need not be.
- Reducing to $m$ dimensions never *increases* total variance retained as $m$ grows: it is monotone non-decreasing and equals 100% at $m=d$.
- Repeated eigenvalues mean the corresponding PCs are not unique.

## Connections

- [Eigenvalues and eigenvectors](../03-linear-algebra/eigenvalues-and-eigenvectors.md) — PCA is the eigen-decomposition of a symmetric PSD matrix (spectral theorem).
- [Orthogonality, projections, SVD](../03-linear-algebra/orthogonality-projections-svd.md) — projection onto the span of the top $m$ vectors; SVD gives PCA directly; the truncated SVD is the best rank-$m$ approximation.
- [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — covariance matrix, variance of a linear combination $v^TSv$.
- [Linear regression](linear-and-logistic-regression.md) — OLS minimises *vertical* residuals, PCA minimises *perpendicular* distances; principal-component regression; ridge shrinks along small-variance PCs most.
- [Classification methods](classification-methods.md) — LDA is supervised dimensionality reduction; k-NN suffers the curse that PCA mitigates.
- [Clustering](clustering.md) — cluster in the reduced space.
- [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) — Lagrange multipliers derive the eigenproblem.

## Practice

**Q1 (NAT).** A covariance matrix has eigenvalues $6,\ 3,\ 1$. Fraction of variance explained by the first two components? (as a decimal)

<details><summary>Answer</summary>

**Answer:** 0.9. **Solution:** $(6+3)/10=0.9$.

</details>

**Q2 (NAT).** $S=\begin{pmatrix}2&1\\1&2\end{pmatrix}$. Largest eigenvalue? Direction of PC1?

<details><summary>Answer</summary>

**Answer:** $\lambda_1=3$, PC1 $=(1,1)/\sqrt2$. **Solution:** $(2-\lambda)^2=1\Rightarrow\lambda=3,1$; for $\lambda=3$: $-v_1+v_2=0$.

</details>

**Q3 (MCQ).** PCA on centred data with $n=5$ points in $\mathbb R^{10}$. The maximum number of components with nonzero variance is (a) 10 (b) 5 (c) 4 (d) 1.

<details><summary>Answer</summary>

**Answer:** (c). **Solution:** centred rows sum to zero, so rank $\le n-1=4$.

</details>

**Q4 (NAT).** Centred data matrix has singular values $6$ and $2$, with $n=5$ samples. Variance along PC1?

<details><summary>Answer</summary>

**Answer:** 9. **Solution:** $\lambda_1=\sigma_1^2/(n-1)=36/4=9$.

</details>

**Q5 (MSQ).** Which statements about PCA are true? (a) Principal components are eigenvectors of the covariance matrix. (b) PCA uses class labels. (c) The scores on different PCs are uncorrelated. (d) The sum of the eigenvalues equals the trace of the covariance matrix.

<details><summary>Answer</summary>

**Answer:** (a), (c), (d). **Solution:** (b) false: PCA is unsupervised. $\text{Cov}(Z)=V^TSV=\Lambda$ is diagonal.

</details>

**Q6 (NAT).** Example A's four points $(2,0),(0,2),(3,1),(1,3)$: what is the coordinate of the first point on PC1 (use $v_1=(1,-1)/\sqrt2$) after centring? (3 decimals)

<details><summary>Answer</summary>

**Answer:** 1.414. **Solution:** centred $(0.5,-1.5)$; dot with $(1,-1)/\sqrt2$ $=2/\sqrt2=1.414$.

</details>

**Q7 (NAT).** $S=\begin{pmatrix}4&2\\2&3\end{pmatrix}$; what is the total squared reconstruction error per sample (as variance) if only PC1 is kept?

<details><summary>Answer</summary>

**Answer:** 1.438. **Solution:** discarded variance $=\lambda_2=7-5.562=1.438$ (per $n-1$ normalisation).

</details>

**Q8 (MCQ).** Two features: height in metres (variance $0.01$) and weight in grams (variance $10^6$), uncorrelated. Unstandardised PCA's PC1 is approximately (a) height (b) weight (c) a 45° mixture (d) undefined.

<details><summary>Answer</summary>

**Answer:** (b). **Solution:** PC1 aligns with the axis of largest variance; scaling dominates, which is why we standardise first.

</details>
