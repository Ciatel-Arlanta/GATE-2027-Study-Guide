# Linear, ridge and logistic regression

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Simple linear regression; Multiple linear regression; Ridge regression; Logistic regression
> **Prerequisites:** [ML foundations](ml-foundations.md) · [Orthogonality, projections, SVD](../03-linear-algebra/orthogonality-projections-svd.md) · [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) · **Leads to:** [Classification methods](classification-methods.md) · [Neural networks](neural-networks.md) · [PCA](dimensionality-reduction-pca.md)

## Quick glance

- Simple LR: $\hat\beta_1 = S_{xy}/S_{xx}$, $\hat\beta_0 = \bar y - \hat\beta_1 \bar x$. The line passes through $(\bar x, \bar y)$.
- Multiple LR (normal equations): $\hat\beta = (X^TX)^{-1}X^Ty$; $\hat y = X\hat\beta$ is the **orthogonal projection** of $y$ onto the column space of $X$; residuals satisfy $X^Te = 0$.
- $R^2 = 1 - SSE/SST$; adding predictors never lowers $R^2$ (adjusted $R^2$ can fall).
- Ridge: minimise $\|y - X\beta\|^2 + \lambda\|\beta\|^2$; $\hat\beta = (X^TX + \lambda I)^{-1}X^Ty$, always invertible for $\lambda>0$; **shrinks coefficients, raises bias, lowers variance**. Intercept is not penalised.
- Logistic regression: $P(y=1\mid x) = \sigma(w^Tx)$, $\sigma(z)=1/(1+e^{-z})$; **log-odds are linear in $x$**; decision boundary $w^Tx = 0$ is linear.
- Cross-entropy loss gradient: $\sum_i (\hat p_i - y_i)\,x_i$. Same form as squared-error gradient for linear regression.
- #1 trap: logistic regression is a **classifier**, and $\lambda \to\infty$ in ridge sends coefficients (not the intercept) to 0, never exactly 0 for finite $\lambda$ (lasso can hit 0).

## 1. Simple linear regression

**Intuition.** Draw the line that sits closest to all points, measuring closeness by the *vertical* gaps (residuals) squared.

**Model.** $y_i = \beta_0 + \beta_1 x_i + \varepsilon_i$. **Least squares** minimises $SSE = \sum_i (y_i - \beta_0 - \beta_1x_i)^2$. Setting both partial derivatives to zero gives:

$$\hat\beta_1 = \frac{S_{xy}}{S_{xx}} = \frac{\sum (x_i-\bar x)(y_i-\bar y)}{\sum (x_i-\bar x)^2},\qquad \hat\beta_0 = \bar y - \hat\beta_1\bar x$$

Equivalent: $\hat\beta_1 = r\,\dfrac{s_y}{s_x}$ ($r$ = correlation).

**Worked example.** $x = 1,2,3,4,5$; $y = 2,4,5,4,5$.

| $x$ | $y$ | $x-\bar x$ | $y-\bar y$ | $(x-\bar x)(y-\bar y)$ | $(x-\bar x)^2$ |
| --- | --- | --- | --- | --- | --- |
| 1 | 2 | -2 | -2 | 4 | 4 |
| 2 | 4 | -1 | 0 | 0 | 1 |
| 3 | 5 | 0 | 1 | 0 | 0 |
| 4 | 4 | 1 | 0 | 0 | 1 |
| 5 | 5 | 2 | 1 | 2 | 4 |

$\bar x = 3$, $\bar y = 4$, $S_{xy} = 6$, $S_{xx} = 10$.
1. $\hat\beta_1 = 6/10 = 0.6$.
2. $\hat\beta_0 = 4 - 0.6(3) = 2.2$. Line: $\hat y = 2.2 + 0.6x$.
3. Fitted: $2.8, 3.4, 4.0, 4.6, 5.2$. Residuals $e = y-\hat y$: $-0.8, 0.6, 1.0, -0.6, -0.2$ (they sum to $0$).
4. $SSE = 0.64 + 0.36 + 1 + 0.36 + 0.04 = 2.4$. $SST = \sum (y-\bar y)^2 = 4+0+1+0+1 = 6$.
5. $R^2 = 1 - 2.4/6 = \mathbf{0.6}$ (also $r^2$ for simple LR). Prediction at $x=6$: $2.2 + 3.6 = 5.8$.

**Facts.** Residuals sum to zero (when an intercept is included); residuals are uncorrelated with $x$; the line passes through $(\bar x,\bar y)$. $R^2 = SSR/SST = 1 - SSE/SST$ with $SST = SSR + SSE$. **Regressing $y$ on $x$ and $x$ on $y$ gives different lines** (both pass through the means; slopes multiply to $r^2$).

**Gradient descent version.** With $J = \frac1n\sum(\hat y_i - y_i)^2$: $\partial J/\partial\beta_0 = \frac2n\sum(\hat y_i - y_i)$, $\partial J/\partial\beta_1 = \frac2n\sum(\hat y_i - y_i)x_i$; update $\beta \leftarrow \beta - \eta\nabla J$. For the data above from $\beta = (0,0)$: $\sum(\hat y - y) = -20$, $\sum(\hat y-y)x = -66$. Gradient $= (-8, -26.4)$. With $\eta = 0.01$: $\beta_0 = 0.08$, $\beta_1 = 0.264$. Converges to $(2.2, 0.6)$ because $J$ is convex.

## 2. Multiple linear regression

**Model.** $y = X\beta + \varepsilon$ with design matrix $X$ ($n\times(p+1)$, first column of ones). Minimising $\|y - X\beta\|^2$ gives the **normal equations**

$$X^TX\,\hat\beta = X^Ty \;\Rightarrow\; \hat\beta = (X^TX)^{-1}X^Ty \quad(\text{needs } X \text{ of full column rank}).$$

**Geometric view.** $\hat y = X\hat\beta = Py$ with $P = X(X^TX)^{-1}X^T$ (the **hat matrix**, symmetric and idempotent). It projects $y$ onto $\text{Col}(X)$; the residual $e = y-\hat y$ is orthogonal to every column, so $X^Te=0$. See [projections](../03-linear-algebra/orthogonality-projections-svd.md). If columns are linearly dependent (**multicollinearity**), $X^TX$ is singular or nearly so and estimates blow up in variance: ridge is the cure.

**Worked example.** Four observations, two predictors:

| $x_1$ | $x_2$ | $y$ |
| --- | --- | --- |
| 1 | 2 | 6 |
| 2 | 1 | 5 |
| 3 | 4 | 10 |
| 4 | 3 | 13 |

With $X$ having columns $(1, x_1, x_2)$:
$X^TX = \begin{pmatrix}4&10&10\\10&30&28\\10&28&30\end{pmatrix}$, $X^Ty = (34, 98, 96)^T$.

Solve the system:
- (1) $4b_0 + 10b_1 + 10b_2 = 34$
- (2) $10b_0 + 30b_1 + 28b_2 = 98$
- (3) $10b_0 + 28b_1 + 30b_2 = 96$

(2)−(3): $2b_1 - 2b_2 = 2 \Rightarrow b_1 = b_2 + 1$. Into (1): $4b_0 + 20b_2 + 10 = 34 \Rightarrow b_0 = 6 - 5b_2$. Into (2): $60 - 50b_2 + 30b_2 + 30 + 28b_2 = 98 \Rightarrow 8b_2 = 8 \Rightarrow b_2 = 1$, $b_1 = 2$, $b_0 = 1$.

$\hat y = 1 + 2x_1 + x_2$. Fitted: $5, 6, 11, 12$; residuals $e = (1,-1,-1,1)$. Check $X^Te$: $1-1-1+1=0$; $x_1$: $1-2-3+4=0$; $x_2$: $2-1-4+3=0$ ✓. $SSE = 4$, $\bar y = 8.5$, $SST = 6.25+12.25+2.25+20.25 = 41$, $R^2 = 1-4/41 = 0.902$. Adjusted $R^2 = 1 - \frac{SSE/(n-p-1)}{SST/(n-1)} = 1 - \frac{4/1}{41/3} = 0.707$ (n = 4, p = 2).

**Assumptions (for inference).** Linearity in parameters; independent errors with constant variance (homoscedasticity), mean zero; no perfect multicollinearity; normal errors for $t$-tests/confidence intervals. The *least squares* estimate itself needs only full rank. "Linear" means linear in $\beta$: $y = \beta_0 + \beta_1x + \beta_2x^2$ is still linear regression.

**Parameter count:** $p+1$ coefficients for $p$ features.

## 3. Ridge regression

**Intuition.** With many or correlated features, OLS fits noise and coefficients become huge with opposite signs. Penalise large weights: you accept a little bias for a large drop in variance.

$$\hat\beta^{ridge} = \arg\min_\beta\; \|y - X\beta\|^2 + \lambda\|\beta\|_2^2 = (X^TX + \lambda I)^{-1}X^Ty$$

(Standardise features; do not penalise the intercept, equivalently centre $y$ and $X$ first.)

- $X^TX$ is positive semi-definite, so $X^TX+\lambda I$ has eigenvalues $\ge\lambda>0$: **always invertible** even if columns are dependent or $p>n$.
- $\lambda = 0$: OLS. $\lambda\to\infty$: all coefficients $\to 0$ (prediction $\to \bar y$).
- In the SVD basis, coefficient along direction $j$ is multiplied by $\frac{d_j^2}{d_j^2+\lambda}$: **low-variance directions are shrunk most**.
- Bias$^2$ increases, variance decreases with $\lambda$; training error rises monotonically; test error is U-shaped. Choose $\lambda$ by cross-validation.

**Worked example (SLR with centred data).** From Section 1, $S_{xy}=6$, $S_{xx}=10$. For centred data the ridge slope is $\hat\beta_1 = S_{xy}/(S_{xx}+\lambda)$.

| $\lambda$ | slope | intercept $\bar y - \beta_1\bar x$ |
| --- | --- | --- |
| 0 | 0.6 | 2.2 |
| 5 | $6/15 = 0.4$ | $4 - 1.2 = 2.8$ |
| 10 | $6/20 = 0.3$ | $4 - 0.9 = 3.1$ |

The slope shrinks toward 0 while the line still passes through $(\bar x,\bar y)$.

**Ridge vs lasso.** Lasso penalises $\lambda\|\beta\|_1$: it produces **exact zeros** (feature selection), has no closed form. Ridge keeps all features with small weights. Bayesian reading: ridge = Gaussian prior on weights (MAP), lasso = Laplace prior.

## 4. Logistic regression

**Intuition.** For two classes we want a probability in $[0,1]$. A line can leave that range, so we pass the linear score through the **sigmoid**.

$$P(y=1\mid x) = \sigma(w^Tx) = \frac{1}{1+e^{-w^Tx}},\qquad \ln\frac{P(y=1\mid x)}{P(y=0\mid x)} = w^Tx$$

The **log-odds (logit) are linear in $x$**. Odds $= p/(1-p)$; e.g. $p = 0.75$: odds $=3$, logit $=\ln 3 = 1.0986$. Increasing $x_j$ by 1 multiplies the odds by $e^{w_j}$.

**Decision rule.** Predict 1 if $p > 0.5 \iff w^Tx > 0$. The **boundary $w^Tx=0$ is a hyperplane**, so logistic regression is a **linear classifier**; moving the probability threshold moves the hyperplane parallel to itself.

**Properties of $\sigma$:** $\sigma(0)=0.5$, $\sigma(-z) = 1-\sigma(z)$, $\sigma'(z) = \sigma(z)(1-\sigma(z))\le 0.25$.

**Loss: cross-entropy (negative log-likelihood).** Assuming $y_i \sim \text{Bernoulli}(p_i)$ independently, maximum likelihood gives

$$J(w) = -\sum_i \big[y_i\ln p_i + (1-y_i)\ln(1-p_i)\big],\qquad \nabla_w J = \sum_i (p_i - y_i)\,x_i$$

$J$ is **convex** (single global minimum), but there is no closed form: use gradient descent or Newton's method. **Why not squared error?** With a sigmoid, squared loss is non-convex in $w$, and its gradient contains the factor $p(1-p)$ which vanishes when the model is confidently wrong, so learning stalls. Cross-entropy cancels that factor.

If the data are **linearly separable**, MLE has no finite solution (weights grow without bound); regularisation fixes this.

**Worked example (prediction, loss and one update).** $w = (w_0,w_1,w_2) = (-5, 1.5, 1)$ and $x = (1, 2, 1)$ (leading 1 for bias), true label $y=1$.
1. $z = -5 + 1.5(2) + 1(1) = -1$.
2. $p = \sigma(-1) = 1/(1+e) = 0.2689$. Predicted class 0 (wrong).
3. Loss $= -\ln 0.2689 = 1.313$.
4. Gradient $= (p-y)x = (-0.7311)(1,2,1) = (-0.7311, -1.4621, -0.7311)$.
5. Learning rate 0.5: $w \leftarrow w - 0.5\cdot$grad $= (-4.6345,\ 2.2311,\ 1.3655)$.
6. New $z = -4.6345 + 4.4621 + 1.3655 = 1.1932$, $p = 0.7673$, loss $= 0.265$. Now classified correctly.

**Multiclass: softmax.** $P(y=k\mid x) = e^{z_k}/\sum_j e^{z_j}$ with $z_k = w_k^Tx$. For $z = (2,1,0)$: $e^z = (7.389, 2.718, 1)$, sum $= 11.107$, probabilities $(0.665, 0.245, 0.090)$. Loss is cross-entropy $-\ln p_{\text{true class}}$. Two classes reduce to the sigmoid. Parameters: $K(p+1)$ (one redundant set).

| | Linear regression | Logistic regression |
| --- | --- | --- |
| Output | real number | probability of class 1 |
| Loss | squared error (MLE with Gaussian noise) | cross-entropy (MLE with Bernoulli) |
| Closed form | yes, normal equations | no, iterative |
| Boundary / fit | line, plane | linear boundary $w^Tx=0$ |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Simple LR slope, intercept | $S_{xy}/S_{xx}$, $\bar y-\hat\beta_1\bar x$ | hand fit on 4-6 points |
| Normal equations | $X^TX\hat\beta = X^Ty$ | multiple LR |
| Hat matrix | $P = X(X^TX)^{-1}X^T$, $P^2=P=P^T$ | projection / leverage questions |
| $R^2$ | $1-SSE/SST$; adj $=1-\frac{SSE/(n-p-1)}{SST/(n-1)}$ | fit quality |
| Ridge | $(X^TX+\lambda I)^{-1}X^Ty$ | multicollinearity, variance reduction |
| Centred ridge slope (1 feature) | $S_{xy}/(S_{xx}+\lambda)$ | numeric ridge |
| Sigmoid / derivative | $1/(1+e^{-z})$; $\sigma(1-\sigma)$ | logistic computations |
| Logit | $\ln\frac{p}{1-p}=w^Tx$ | odds interpretation |
| Cross-entropy gradient | $\sum(p_i-y_i)x_i$ | gradient descent step |
| Softmax | $e^{z_k}/\sum e^{z_j}$ | multiclass |

## GATE traps

- **$R^2$ never decreases when you add a predictor**, even a useless one; compare models with adjusted $R^2$ or CV error.
- **Ridge never sets coefficients exactly to zero** (lasso does); the intercept is not penalised.
- **Ridge always has a unique solution** for $\lambda>0$, even when $X^TX$ is singular.
- Ridge **increases bias, decreases variance**; training error goes up with $\lambda$.
- "Logistic regression is linear in $x$" is true **for the log-odds**, not for the probability.
- The logistic decision boundary is linear; a quadratic boundary needs quadratic features.
- Residuals sum to zero only if the model includes an intercept.
- Normal equations need $X^TX$ invertible: fails when $p+1>n$ or columns are collinear.
- Cross-entropy, not squared error, is the standard logistic loss (convexity, no vanishing gradient).
- Gradient of cross-entropy w.r.t. weights is $(p - y)x$, **not** $(p-y)p(1-p)x$ (that is for squared loss).

## Connections

- [Orthogonality, projections, SVD](../03-linear-algebra/orthogonality-projections-svd.md) — OLS fit is a projection; ridge in the SVD basis.
- [Linear systems](../03-linear-algebra/linear-systems-and-lu.md) — solving the normal equations by elimination.
- [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) — setting gradients to zero, convexity, gradient descent.
- [ML foundations](ml-foundations.md) — squared loss/cross-entropy as MLE; ridge as the bias-variance knob.
- [Neural networks](neural-networks.md) — a single sigmoid neuron *is* logistic regression; softmax output layers.
- [Classification methods](classification-methods.md) — LDA gives the same linear form with different estimation; SVM uses hinge instead of log loss.
- [Dimensionality reduction (PCA)](dimensionality-reduction-pca.md) — principal-component regression; ridge shrinks small-variance PCs.
- [Probability: Gaussian and Bernoulli](../02-probability-statistics/discrete-distributions.md) — the likelihoods behind the two losses.

## Practice

**Q1 (NAT).** Points $(1,3),(2,5),(3,7)$ are fitted by least squares. What is the predicted $y$ at $x=10$?

<details><summary>Answer</summary>

**Answer:** 21  
**Solution:** The points are exactly on $y = 2x+1$ (slope 2, intercept 1), so SSE = 0 and the fit is that line. At $x=10$: 21.

</details>

**Q2 (NAT).** For $x = 1,2,3,4,5$ and $y = 2,4,5,4,5$, find the least-squares slope.

<details><summary>Answer</summary>

**Answer:** 0.6  
**Solution:** $\bar x=3,\bar y=4$; $S_{xy} = 4+0+0+0+2 = 6$; $S_{xx} = 10$; slope $=0.6$.

</details>

**Q3 (MCQ).** Increasing the ridge parameter $\lambda$ from 0 towards large values (a) raises variance, lowers bias (b) lowers variance, raises bias (c) lowers both (d) has no effect on either.

<details><summary>Answer</summary>

**Answer:** (b)  
**Solution:** Shrinkage restricts the model: more bias, less sensitivity to training sample (less variance).

</details>

**Q4 (NAT).** Using the Section 1 data, find the ridge slope (centred data, intercept unpenalised) with $\lambda = 10$.

<details><summary>Answer</summary>

**Answer:** 0.3  
**Solution:** $S_{xy}/(S_{xx}+\lambda) = 6/(10+10) = 0.3$.

</details>

**Q5 (NAT).** A logistic model has $w_0 = -3$, $w_1 = 1$. For $x = 3$ the probability of class 1 is? Then for which $x$ is it exactly 0.5?

<details><summary>Answer</summary>

**Answer:** $p=0.5$ at $x=3$; boundary at $x = 3$.  
**Solution:** $z = -3 + 1\cdot 3 = 0$, $\sigma(0) = 0.5$. Boundary is $w_0+w_1x=0\Rightarrow x=3$.

</details>

**Q6 (MSQ).** Which are true for logistic regression? (A) log-odds are linear in features (B) the cross-entropy loss is convex in $w$ (C) it has a closed-form solution via normal equations (D) the decision boundary is a hyperplane

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:** MLE needs iterative optimisation, so (C) is false.

</details>

**Q7 (NAT).** Logistic model with $w = (0, 2)$ (bias, slope) sees point $x = 1$ (feature vector $(1,1)$), label $y = 0$. Compute the cross-entropy loss, to 3 decimals.

<details><summary>Answer</summary>

**Answer:** 2.127  
**Solution:** $z = 2$, $p = \sigma(2) = 1/(1+e^{-2}) = 0.8808$. Since $y=0$, loss $= -\ln(1-p) = -\ln(0.1192) = 2.127$.

</details>

**Q8 (MSQ).** In multiple linear regression with an intercept, which hold for the OLS residual vector $e$? (A) $\sum e_i = 0$ (B) $e \perp$ every column of $X$ (C) $\hat y \perp e$ (D) $\|e\|$ is zero whenever $n > p+1$

<details><summary>Answer</summary>

**Answer:** A, B, C  
**Solution:** The first column of $X$ is all ones, so $X^Te=0$ gives (A); (B) is $X^Te=0$; $\hat y = X\hat\beta$ is a combination of columns so (C). (D) is false: residuals vanish only if $y\in\text{Col}(X)$.

</details>
