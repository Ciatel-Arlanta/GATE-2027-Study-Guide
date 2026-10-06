# Classification methods: k-NN, naive Bayes, LDA, SVM, decision trees

> **Paper:** DA · **Priority:** P0 · **Plan topics:** k-nearest neighbours; Naive Bayes classifier; Linear discriminant analysis; Support vector machines; Decision trees
> **Prerequisites:** [ML foundations](ml-foundations.md) · [Linear and logistic regression](linear-and-logistic-regression.md) · [Probability basics](../02-probability-statistics/probability-basics.md) · **Leads to:** [Neural networks](neural-networks.md) · [Probabilistic reasoning](../17-artificial-intelligence/probabilistic-reasoning.md)

## Quick glance

- **k-NN**: lazy learner; predict the majority label (or mean for regression) of the $k$ closest training points. $k=1$ gives training error 0. Small $k$ = low bias / high variance. **Scale features first.**
- **Naive Bayes**: $P(c\mid x) \propto P(c)\prod_j P(x_j\mid c)$ (features independent given class). Laplace smoothing: $(n_{jc}+1)/(n_c + V_j)$. Never normalise away the zero-frequency problem.
- **LDA**: Gaussian classes, **shared covariance** $\Rightarrow$ linear boundary. Fisher direction $w \propto S_W^{-1}(\mu_2-\mu_1)$ maximises between-class over within-class scatter. QDA (separate covariances) gives a quadratic boundary.
- **SVM**: maximum-margin hyperplane $w^Tx+b=0$, margin width $2/\|w\|$; only **support vectors** matter. Soft margin: $\min \tfrac12\|w\|^2 + C\sum\xi_i$; large $C$ = narrow margin, low bias. Kernel trick: replace $x_i^Tx_j$ by $K(x_i,x_j)$.
- **Decision tree**: greedy top-down splits maximising information gain $= H(\text{parent}) - \sum \frac{n_v}{n}H(v)$; entropy $H=-\sum p\log_2p$, Gini $=1-\sum p^2$. Deep trees overfit; prune.
- Ensembles: bagging/random forest cut variance; boosting cuts bias.
- #1 trap: information gain favours many-valued attributes (an ID column has gain = $H$(parent)); gain ratio corrects it.

## 1. k-nearest neighbours (k-NN)

**Intuition.** "You are like your neighbours." There is **no training**: store the data; at prediction time find the $k$ stored points closest to the query and vote.

**Distances** between $x, z \in \mathbb R^d$:

| Metric | Formula | Notes |
| --- | --- | --- |
| Euclidean ($L_2$) | $\sqrt{\sum (x_j-z_j)^2}$ | default |
| Manhattan ($L_1$) | $\sum |x_j-z_j|$ | grid distance |
| Minkowski ($L_p$) | $(\sum |x_j-z_j|^p)^{1/p}$ | $p=1,2,\infty$ special cases |
| Cosine distance | $1 - \dfrac{x^Tz}{\|x\|\|z\|}$ | direction only; text data |

**Worked example.** Training set: class A = $(1,1),(2,1),(1,3),(2,3)$; class B = $(4,4),(5,3),(3,5)$. Classify $q=(3,3)$ with Euclidean distance.

| Point | Class | Distance to $q$ |
| --- | --- | --- |
| $(2,3)$ | A | $1$ |
| $(4,4)$ | B | $\sqrt2 = 1.414$ |
| $(1,3)$ | A | $2$ |
| $(5,3)$ | B | $2$ |
| $(3,5)$ | B | $2$ |
| $(2,1)$ | A | $\sqrt5 = 2.236$ |
| $(1,1)$ | A | $\sqrt8 = 2.828$ |

- $k=1$: nearest is $(2,3)$ $\Rightarrow$ **A**.
- $k=3$: $(2,3)$ A, $(4,4)$ B, then a **three-way distance tie** at 2 between A, B, B. If the third neighbour is the A point the vote is 2-1 for A, otherwise 2-1 for B: the answer depends on the tie-break rule (ties in distance are why GATE usually picks $k$ so that none occur).
- $k=5$: the five nearest are $(2,3),(4,4),(1,3),(5,3),(3,5)$ = A, B, A, B, B $\Rightarrow$ **B** (3-2).
- $k=7$ (all points): A has 4, B has 3 $\Rightarrow$ **A** (just the majority class: $k=n$ always predicts the majority class).

The prediction flips with $k$: that is the bias-variance dial.

**Effect of $k$.**

| $k$ | Boundary | Bias | Variance | Training error |
| --- | --- | --- | --- | --- |
| 1 | very jagged | low | high | **0** (each point is its own nearest neighbour; assumes no duplicate points with different labels) |
| large | smooth | high | low | grows; $k=n$ predicts majority class |

Pick $k$ by cross-validation; use odd $k$ for two classes to avoid vote ties.

**Practical issues.**
- **Feature scaling.** A feature in the thousands swamps one in $[0,1]$. Standardise or min-max scale before computing distances.
- **Curse of dimensionality.** In high $d$ all points become nearly equidistant, so "nearest" loses meaning; irrelevant features add noise.
- **Cost.** Training $O(1)$ (store data). Prediction $O(nd)$ per query by brute force: slow for large $n$. Memory $O(nd)$.
- k-NN **regression**: average (optionally distance-weighted) the $k$ neighbour targets. Weighted voting uses $w=1/d^2$.

## 2. Naive Bayes classifier

**Intuition.** Bayes' rule says the best class is the one with the highest posterior. Estimating $P(x_1,\dots,x_d\mid c)$ needs exponentially many parameters. The *naive* assumption — features are **conditionally independent given the class** — turns it into a product of small tables.

$$\hat c = \arg\max_c\; P(c)\prod_{j=1}^d P(x_j\mid c)$$

(The denominator $P(x)$ is the same for all classes, so it is dropped for the argmax and only needed if you want actual probabilities.)

**Parameters.** For $d$ binary features and 2 classes: $1 + 2d$ numbers, instead of $2(2^d-1)+1$ for the full joint.

**Worked example (categorical, Laplace smoothing).** Eight days; features Outlook $\in\{$Sunny, Overcast, Rainy$\}$ (3 values), Temp $\in\{$Hot, Mild$\}$ (2 values), class Play.

| # | Outlook | Temp | Play |
| --- | --- | --- | --- |
| 1 | Sunny | Hot | No |
| 2 | Sunny | Mild | No |
| 3 | Rainy | Mild | Yes |
| 4 | Rainy | Hot | Yes |
| 5 | Sunny | Mild | Yes |
| 6 | Overcast | Hot | Yes |
| 7 | Rainy | Mild | No |
| 8 | Overcast | Mild | Yes |

Counts: Yes = 5 (rows 3,4,5,6,8), No = 3 (rows 1,2,7). Priors $5/8$ and $3/8$.

Query: **Sunny, Mild**, no smoothing.
- Yes: Sunny among Yes = 1 (row 5), Mild among Yes = 3 (rows 3,5,8). Score $=\frac58\cdot\frac15\cdot\frac35 = 0.075$.
- No: Sunny among No = 2, Mild among No = 2 (rows 2,7). Score $=\frac38\cdot\frac23\cdot\frac23 = 0.1667$.
- Normalise: $P(\text{Yes}) = 0.075/0.2417 = 0.310$, $P(\text{No}) = 0.690$. Predict **No**.

Query: **Overcast, Hot**, no smoothing. Overcast never occurs among No, so the No score is $0$. Yes: Overcast $=2/5$, Hot $=2/5$ (rows 4, 6), score $\frac58\cdot\frac25\cdot\frac25 = 0.1$, posterior of Yes $=1.0$. This is over-confident, and a single zero count wipes out an otherwise plausible class: the **zero-frequency problem**.

**Laplace (add-one) smoothing**: $P(x_j=v\mid c) = \dfrac{n_{v,c}+1}{n_c + V_j}$ with $V_j$ the number of values of feature $j$ (3 for Outlook, 2 for Temp). Then for Overcast, Hot:
- Yes: Overcast $(2+1)/(5+3)=3/8$, Hot $(2+1)/(5+2)=3/7$; score $\frac58\cdot\frac38\cdot\frac37 = 0.1004$.
- No: Overcast $(0+1)/(3+3)=1/6$, Hot $(1+1)/(3+2)=2/5$; score $\frac38\cdot\frac16\cdot\frac25 = 0.025$.
- $P(\text{Yes}) = 0.1004/0.1254 = \mathbf{0.801}$.

**Gaussian naive Bayes** (continuous features): $P(x_j\mid c) = \mathcal N(x_j;\mu_{jc},\sigma_{jc}^2)$ with per-class, per-feature mean and variance estimated from data.

**Linear boundary?** For binary features (Bernoulli NB), and for Gaussian NB with variances equal across classes, $\log\frac{P(1\mid x)}{P(0\mid x)}$ is linear in $x$, so the decision boundary is linear (and has the same form as logistic regression). With class-specific variances Gaussian NB gives a quadratic boundary.

**Facts.** Training = counting, $O(nd)$. Works well even when independence is violated (the argmax is often still right though the probabilities are not). It is a Bayesian network with the class as the single parent of all features: see [probabilistic reasoning](../17-artificial-intelligence/probabilistic-reasoning.md).

## 3. Linear discriminant analysis (LDA)

Two connected ideas share the name.

**(a) Generative view.** Assume $x\mid c\sim\mathcal N(\mu_c,\Sigma)$ with the **same covariance $\Sigma$ for every class**. Then the log posterior ratio is linear in $x$:

$$\delta_c(x) = x^T\Sigma^{-1}\mu_c - \tfrac12\mu_c^T\Sigma^{-1}\mu_c + \log\pi_c,\qquad \hat c = \arg\max_c\delta_c(x)$$

Boundary between two classes: $\delta_1=\delta_2$, a hyperplane. If each class has its own $\Sigma_c$ the quadratic terms no longer cancel: **QDA**, quadratic boundary, more parameters (more variance).

**(b) Fisher's view (projection).** Find the direction $w$ to project onto so that classes are as separated as possible. For two classes with means $\mu_1,\mu_2$:

$$J(w) = \frac{(w^T\mu_2 - w^T\mu_1)^2}{w^TS_Ww},\qquad S_W = \sum_{c}\sum_{x\in c}(x-\mu_c)(x-\mu_c)^T$$

(between-class separation over within-class scatter). Maximiser: $\boxed{w \propto S_W^{-1}(\mu_2-\mu_1)}$. Only the direction matters.

**Worked example (2-D).** Class 1: $(1,2),(2,3),(3,3),(2,1)$; class 2: $(6,5),(7,8),(8,6),(7,5)$.
1. Means: $\mu_1 = (2, 2.25)$, $\mu_2 = (7, 6)$. Difference $\mu_2-\mu_1 = (5, 3.75)$.
2. Scatter of class 1: $\begin{pmatrix}2&1\\1&2.75\end{pmatrix}$; class 2: $\begin{pmatrix}2&1\\1&6\end{pmatrix}$. Sum $S_W = \begin{pmatrix}4&2\\2&8.75\end{pmatrix}$.
3. $\det S_W = 35-4=31$, $S_W^{-1} = \frac1{31}\begin{pmatrix}8.75&-2\\-2&4\end{pmatrix}$.
4. $w \propto S_W^{-1}(5,3.75) = \frac1{31}(43.75-7.5,\;-10+15) = \frac1{31}(36.25,\,5) \propto (1.169,\,0.161)$.
5. Projections $w^Tx$ (using $w=(1.169,0.161)$): class 1 $\approx 1.49, 2.82, 3.99, 2.50$; class 2 $\approx 7.82, 9.48, 10.32, 8.99$. Classes are perfectly separated; threshold = midpoint of projected means $\approx 5.93$.

Note that $w$ is **not** simply $\mu_2-\mu_1$ (that would ignore the within-class spread); it tilts toward the direction of smaller scatter.

**Comparison.**

| | LDA | QDA | Logistic regression | Naive Bayes |
| --- | --- | --- | --- | --- |
| Type | generative | generative | discriminative | generative |
| Boundary | linear | quadratic | linear | linear or quadratic |
| Assumption | Gaussian, shared $\Sigma$ | Gaussian, class $\Sigma_c$ | none on $x$ | feature independence |
| Best when | small data, Gaussian-ish | more data | assumptions fail | many features, tiny data |

**Dimension limit.** For $K$ classes Fisher LDA gives at most $K-1$ discriminant directions. LDA is **supervised**; PCA is not ([PCA](dimensionality-reduction-pca.md)).

## 4. Support vector machines (SVM)

**Intuition.** Many lines separate two classes; choose the one with the **widest empty street** between the classes. Wide margin = robust to noise = lower variance.

**Hard-margin primal** (labels $y_i\in\{-1,+1\}$):

$$\min_{w,b}\ \tfrac12\|w\|^2\quad\text{s.t.}\quad y_i(w^Tx_i+b)\ge 1\ \ \forall i$$

The margin (street width) is $\dfrac{2}{\|w\|}$, so minimising $\|w\|$ maximises the margin. The constraint is tight (an equality) exactly for the **support vectors**, which lie on the margin lines $w^Tx+b=\pm1$. Remove a non-support vector and the solution does not change. $w=\sum_i\alpha_iy_ix_i$ with $\alpha_i>0$ only for support vectors.

**Worked example.** Negative class: $(0,0),(1,1),(2,0)$; positive class: $(3,3),(4,2),(4,4)$.
1. The closest opposite pairs are negatives $(1,1),(2,0)$ (both on the line $x_1+x_2=2$) and positives $(3,3),(4,2)$ (both on $x_1+x_2=6$). These two parallel lines are the margin lines; the boundary is the middle line $x_1+x_2=4$.
2. Scale so the margin lines give $\mp1$: $w^Tx+b = \tfrac12(x_1+x_2) - 2$. Check: $(1,1)\to-1$, $(2,0)\to-1$, $(3,3)\to+1$, $(4,2)\to+1$ ✓. $(0,0)\to-2$ and $(4,4)\to+2$ satisfy the constraint with room to spare: **not** support vectors.
3. $w=(0.5,0.5)$, $\|w\|=1/\sqrt2$, **margin $=2/\|w\| = 2\sqrt2 = 2.83$** (equal to the distance between the parallel lines: $|6-2|/\sqrt2 = 2.83$ ✓).
4. Support vectors: 4 of 6 points. (Verified numerically with a hard-margin solver.)

**Soft margin.** Allow violations with slack $\xi_i\ge0$:

$$\min_{w,b,\xi}\ \tfrac12\|w\|^2 + C\sum_i\xi_i\quad\text{s.t.}\ y_i(w^Tx_i+b)\ge1-\xi_i$$

Equivalent to **hinge loss** $\max(0,1-y_i f(x_i))$ plus an $L_2$ penalty. $C$ large: violations are costly $\Rightarrow$ narrow margin, low bias, high variance (approaches hard margin). $C$ small: wide margin, more violations, high bias, low variance. Points with $\xi_i>0$ or on the margin are support vectors.

**Kernel trick.** The dual only uses inner products $x_i^Tx_j$. Replace by $K(x_i,x_j)=\phi(x_i)^T\phi(x_j)$ to separate classes linearly in a high-dimensional feature space $\phi$ without computing $\phi$.

| Kernel | $K(x,z)$ | Boundary in input space |
| --- | --- | --- |
| Linear | $x^Tz$ | hyperplane |
| Polynomial | $(x^Tz + c)^p$ | degree-$p$ curve |
| RBF / Gaussian | $\exp(-\gamma\|x-z\|^2)$ | very flexible; infinite-dimensional $\phi$ |

*Check:* for $x,z\in\mathbb R^2$, $K=(x^Tz)^2 = (x_1z_1+x_2z_2)^2 = x_1^2z_1^2 + 2x_1x_2z_1z_2 + x_2^2z_2^2 = \phi(x)^T\phi(z)$ with $\phi(x)=(x_1^2,\sqrt2x_1x_2,x_2^2)$. This lets XOR-like data (not linearly separable in 2-D) become separable. Large RBF $\gamma$ = narrow bumps = overfitting.

**Facts.** The training objective is convex (QP), so there is a global optimum. Prediction cost depends on the number of support vectors. Predicted label: $\text{sign}(w^Tx+b)$.

## 5. Decision trees

**Intuition.** Play twenty questions: pick the feature test that best purifies the groups, split, and recurse until the leaves are (nearly) pure.

**Impurity measures** for class proportions $p_k$ in a node:

| Measure | Formula | Pure node | 2 classes 50/50 |
| --- | --- | --- | --- |
| Entropy | $H=-\sum_kp_k\log_2p_k$ | 0 | 1 |
| Gini | $G=1-\sum_kp_k^2$ | 0 | 0.5 |
| Misclassification | $1-\max_kp_k$ | 0 | 0.5 |

**Information gain** of splitting a node $S$ on attribute $A$: $IG = H(S) - \sum_v\frac{|S_v|}{|S|}H(S_v)$. Choose the attribute with the largest gain (ID3).

**Worked example (ID3 root split).** Eight rows:

| # | Outlook | Temp | Play |
| --- | --- | --- | --- |
| 1 | Sunny | Hot | No |
| 2 | Sunny | Hot | No |
| 3 | Overcast | Hot | Yes |
| 4 | Rainy | Mild | Yes |
| 5 | Rainy | Cool | Yes |
| 6 | Rainy | Cool | No |
| 7 | Overcast | Cool | Yes |
| 8 | Sunny | Mild | No |

Root: 4 Yes, 4 No: $H=1$ (Gini $0.5$).

*Split on Outlook:*
- Sunny (3 rows: No, No, No): $H=0$.
- Overcast (2 rows: Yes, Yes): $H=0$.
- Rainy (3 rows: Yes, Yes, No): $H = -\tfrac23\log_2\tfrac23-\tfrac13\log_2\tfrac13 = 0.9183$.
- Weighted: $\frac38(0)+\frac28(0)+\frac38(0.9183)=0.3444$. **$IG = 1-0.3444 = 0.6556$.**

*Split on Temp:*
- Hot (No, No, Yes): $0.9183$; Mild (Yes, No): $1$; Cool (Yes, No, Yes): $0.9183$.
- Weighted: $\frac38(0.9183)+\frac28(1)+\frac38(0.9183)=0.9387$. **$IG=0.0613$.**

Outlook wins. Two of its branches are already pure leaves; only Rainy needs a further split (on Temp: Mild $\to$ Yes, Cool $\to$ mixed, so impure unless more features exist).

**Gain ratio** (C4.5) fixes the bias toward many-valued attributes: $GR = IG/\text{SplitInfo}$, $\text{SplitInfo}=-\sum_v\frac{|S_v|}{|S|}\log_2\frac{|S_v|}{|S|}$. Here Outlook: SplitInfo $=1.561$, $GR=0.420$; Temp: $GR=0.039$. A unique ID column would have $IG=1$ here but SplitInfo $=\log_2 8=3$.

**Continuous features**: sort values, test thresholds at midpoints between consecutive values with different labels.

**Overfitting and pruning.** An unrestricted tree reaches 0 training error (low bias, high variance). Control by **pre-pruning** (max depth, min samples per leaf, min gain) or **post-pruning** (grow fully, then collapse subtrees that do not help on validation data).

**Regression trees.** Leaves predict the mean of targets; choose splits that most reduce the weighted variance (sum of squared error).

**Ensembles.**
- **Bagging**: train many trees on bootstrap samples, average/vote $\Rightarrow$ **reduces variance**. **Random forest** adds random feature subsets per split to decorrelate trees.
- **Boosting** (AdaBoost, gradient boosting): train weak learners sequentially, each focusing on previous errors $\Rightarrow$ **reduces bias** (can overfit if too many rounds).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| k-NN prediction | majority vote of $k$ nearest | classification |
| k-NN cost | train $O(1)$, predict $O(nd)$ | complexity MCQ |
| NB rule | $\arg\max_c P(c)\prod_jP(x_j\mid c)$ | all NB problems |
| Laplace | $(n+1)/(n_c+V)$ | zero counts |
| Fisher direction | $w\propto S_W^{-1}(\mu_2-\mu_1)$ | LDA projection |
| LDA directions | $\le K-1$ | counting question |
| SVM margin | $2/\|w\|$ | margin value |
| SVM constraints | $y_i(w^Tx_i+b)\ge1$ | feasibility |
| Soft margin | $\frac12\|w\|^2+C\sum\xi_i$ | role of $C$ |
| Entropy 2-class | $-p\log_2p-(1-p)\log_2(1-p)$ | tree split |
| Information gain | $H(S)-\sum\frac{|S_v|}{|S|}H(S_v)$ | pick split |
| Gini | $1-\sum p_k^2$ | CART |

## GATE traps

- k-NN with $k=1$ has **zero training error** but is not "best"; test error is typically high. $k=n$ predicts the majority class everywhere.
- Forgetting to scale features before k-NN or SVM-RBF.
- Naive Bayes: dividing by $P(x)$ is unnecessary for argmax, but **do** include the prior. Forgetting Laplace $V$ in the denominator (it is the number of *values of that feature*, not classes).
- Naive Bayes outputs are poorly calibrated even when its argmax is right.
- LDA assumes a **shared** covariance; unequal covariances $\Rightarrow$ QDA. LDA is supervised; PCA is not.
- SVM: margin is $2/\|w\|$, not $1/\|w\|$ (that is the distance from the boundary to one margin line). Only support vectors matter; deleting a non-support vector changes nothing.
- Larger $C$ means *less* regularisation (smaller margin, fewer violations), the opposite of ridge's $\lambda$.
- Entropy uses $\log_2$ for bits; a 50/50 two-class node has $H=1$, Gini $=0.5$. Information gain is never negative.
- Information gain prefers attributes with many values (e.g. IDs); gain ratio corrects this.
- A fully grown tree can fit any consistent data, so training accuracy 100% says nothing about generalisation.
- Bagging reduces variance, boosting reduces bias.

## Connections

- [ML foundations](ml-foundations.md) — $k$ in k-NN, $C$ in SVM, tree depth are the bias-variance knobs; choose them by cross-validation.
- [Linear and logistic regression](linear-and-logistic-regression.md) — logistic regression, LDA and Gaussian NB (equal variances) all give linear boundaries; hinge vs cross-entropy loss.
- [Dimensionality reduction and PCA](dimensionality-reduction-pca.md) — LDA is the supervised counterpart of PCA.
- [Neural networks](neural-networks.md) — a perceptron is the ancestor of the SVM; kernels vs learned hidden features.
- [Probabilistic reasoning](../17-artificial-intelligence/probabilistic-reasoning.md) — naive Bayes is a Bayesian network with one parent.
- [Probability basics](../02-probability-statistics/probability-basics.md) — Bayes' rule, conditional independence.
- [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) — Lagrange multipliers give the SVM dual.
- [Heaps, trees and traversals](../07-data-structures/trees-and-bst.md) — a decision tree is a binary/multiway tree whose prediction is a root-to-leaf walk.

## Practice

**Q1 (MCQ).** In k-NN with $k=1$ and no duplicate points, the training error is (a) 0 (b) equal to Bayes error (c) 0.5 (d) depends on dimension.

<details><summary>Answer</summary>

**Answer:** (a). **Solution:** each training point's nearest neighbour is itself, which carries its own label.

</details>

**Q2 (NAT).** A node has 8 positive and 8 negative examples. A split yields child A with (6+, 2-) and child B with (2+, 6-). Information gain? (3 decimals)

<details><summary>Answer</summary>

**Answer:** 0.189. **Solution:** $H(\text{parent})=1$. Each child: $H=-0.75\log_20.75-0.25\log_20.25 = 0.3113+0.5=0.8113$. Weighted $=0.8113$. $IG=1-0.8113=0.1887$.

</details>

**Q3 (NAT).** Naive Bayes with priors $P(+)=0.6$, $P(-)=0.4$; for a test point $P(x_1\mid+)=0.5,P(x_2\mid+)=0.2$, $P(x_1\mid-)=0.3,P(x_2\mid-)=0.9$. Find $P(+\mid x)$.

<details><summary>Answer</summary>

**Answer:** 0.3571. **Solution:** score$_+=0.6\cdot0.5\cdot0.2=0.06$; score$_-=0.4\cdot0.3\cdot0.9=0.108$. $P(+\mid x)=0.06/0.168=0.357$. Predict $-$.

</details>

**Q4 (MSQ).** Which are true for a hard-margin linear SVM on separable data? (a) The margin is $2/\|w\|$. (b) Removing a non-support-vector changes the solution. (c) The solution is unique. (d) All support vectors satisfy $y_i(w^Tx_i+b)=1$.

<details><summary>Answer</summary>

**Answer:** (a), (c), (d). **Solution:** (b) is false: only support vectors determine $w,b$. The problem is strictly convex in $w$, so the max-margin hyperplane is unique.

</details>

**Q5 (MCQ).** The SVM kernel $K(x,z)=(x^Tz+1)^2$ on $x\in\mathbb R^2$ corresponds to a feature space of dimension: (a) 3 (b) 5 (c) 6 (d) infinite.

<details><summary>Answer</summary>

**Answer:** (c). **Solution:** monomials of degree $\le2$ in 2 variables: $1,x_1,x_2,x_1^2,x_1x_2,x_2^2$ = 6.

</details>

**Q6 (MCQ).** Which change typically increases the variance of an SVM? (a) smaller $C$ (b) larger $C$ (c) smaller RBF $\gamma$ (d) more training data.

<details><summary>Answer</summary>

**Answer:** (b). **Solution:** large $C$ penalises slack heavily, so the boundary bends to fit individual points (narrow margin). Smaller $\gamma$ and more data reduce variance.

</details>

**Q7 (NAT).** Using the 7-point k-NN training set from section 1 (A: $(1,1),(2,1),(1,3),(2,3)$; B: $(4,4),(5,3),(3,5)$), classify $q=(2,2)$ with $k=3$ using Manhattan distance. How many of the 3 neighbours are class A?

<details><summary>Answer</summary>

**Answer:** 3. **Solution:** Manhattan distances from $(2,2)$: $(1,1)\to2$, $(2,1)\to1$, $(1,3)\to2$, $(2,3)\to1$, $(4,4)\to4$, $(5,3)\to4$, $(3,5)\to4$. Three smallest: $1,1,2$ (with the other 2 tied): all A, since the B points are at 4. Predict A.

</details>

**Q8 (MCQ).** Two classes have means $(0,0)$ and $(4,0)$ with shared within-class covariance $\Sigma=\begin{pmatrix}1&0\\0&4\end{pmatrix}$ and equal priors. The LDA boundary is: (a) $x_1=2$ (b) $x_2=2$ (c) $x_1+x_2=2$ (d) $x_1=4$.

<details><summary>Answer</summary>

**Answer:** (a). **Solution:** $w=\Sigma^{-1}(\mu_2-\mu_1)=(4,0)$; the means differ only along $x_1$, so $w\propto(1,0)$. Equal priors put the threshold at the midpoint of the means: $x_1=2$.

</details>
