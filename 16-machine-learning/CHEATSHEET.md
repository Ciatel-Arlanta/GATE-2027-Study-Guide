# Machine Learning: cheat sheet

Final-revision page. Chapters: [foundations](ml-foundations.md) · [regression](linear-and-logistic-regression.md) · [classification](classification-methods.md) · [neural nets](neural-networks.md) · [clustering](clustering.md) · [PCA](dimensionality-reduction-pca.md).

## Foundations

| Item | Fact |
| --- | --- |
| Supervised / unsupervised | labels $(x,y)$ / only $x$ (clustering, PCA) |
| Regression / classification | real-valued $y$ / discrete $y$ |
| Splits | train (fit) · validation (tune) · test (touch once) |
| Bias-variance | $\text{Err}=\text{bias}^2+\text{var}+\sigma^2$ |
| Complexity up | bias down, variance up; training error always down; test error U-shaped |
| k-fold / LOOCV | $k$ fits / $n$ fits; LOOCV nearly unbiased, high variance, costly |
| Losses | squared (mean), absolute (median), 0-1, cross-entropy, hinge |
| Metrics | precision $TP/(TP+FP)$; recall $TP/(TP+FN)$; specificity $TN/(TN+FP)$; $F_1=2TP/(2TP+FP+FN)$ |
| Trap | accuracy misleads on imbalanced data; never tune on test; scale inside CV folds |

## Regression

| Model | Formula |
| --- | --- |
| Simple LR | $\hat\beta_1=S_{xy}/S_{xx}$, $\hat\beta_0=\bar y-\hat\beta_1\bar x$; line through $(\bar x,\bar y)$ |
| $R^2$ | $1-SSE/SST$; $=r^2$ for simple LR; never decreases when adding predictors |
| Normal equations | $\hat\beta=(X^TX)^{-1}X^Ty$; $X^Te=0$; hat matrix $P=X(X^TX)^{-1}X^T$ |
| Ridge | $(X^TX+\lambda I)^{-1}X^Ty$; bias up, variance down; invertible for $\lambda>0$; intercept not penalised |
| Lasso | $L_1$ penalty; can set coefficients exactly to 0 |
| Logistic | $P(1\mid x)=\sigma(w^Tx)$; log-odds linear; boundary $w^Tx=0$ linear; gradient $\sum(\hat p-y)x$ |
| Softmax | $e^{z_k}/\sum_je^{z_j}$ |

## Classification

| Method | Key points |
| --- | --- |
| k-NN | no training; predict $O(nd)$; $k=1$ train error 0; small $k$ high variance; scale features; curse of dimensionality |
| Naive Bayes | $\arg\max_cP(c)\prod_jP(x_j\mid c)$; Laplace $(n+1)/(n_c+V)$; training = counting |
| LDA | shared $\Sigma$ $\Rightarrow$ linear; Fisher $w\propto S_W^{-1}(\mu_2-\mu_1)$; $\le K-1$ directions; QDA quadratic |
| SVM | margin $2/\|w\|$; constraints $y_i(w^Tx_i+b)\ge1$; soft margin $\frac12\|w\|^2+C\sum\xi$; large $C$ = narrow margin; only support vectors matter |
| Kernels | linear $x^Tz$; poly $(x^Tz+c)^p$; RBF $e^{-\gamma\|x-z\|^2}$ |
| Entropy / Gini | $-\sum p\log_2p$ / $1-\sum p^2$; 50-50 binary: 1 / 0.5 |
| Information gain | $H(S)-\sum\frac{|S_v|}{|S|}H(S_v)$; gain ratio divides by SplitInfo |
| Ensembles | bagging/random forest: variance down; boosting: bias down |

## Neural networks

| Item | Fact |
| --- | --- |
| Perceptron | $w\leftarrow w+\eta(y-\hat y)x$; converges iff linearly separable; cannot do XOR |
| Layer parameters | $(\text{inputs}+1)\times\text{units}$; e.g. $4\to5\to3\to2$: 51 |
| Forward | $a^{(l)}=\phi(W^{(l)}a^{(l-1)}+b^{(l)})$ |
| Derivatives | $\sigma'=\sigma(1-\sigma)\le0.25$; $\tanh'=1-\tanh^2$; ReLU $'=1[z>0]$ |
| Output delta | softmax/sigmoid + cross-entropy: $\hat y-y$; sigmoid + squared: $(\hat y-t)\hat y(1-\hat y)$ |
| Backprop | hidden $\delta_j=\big(\sum_k\delta_kw_{kj}\big)\phi'(z_j)$; $\partial L/\partial w_{jk}=\delta_j a_k$ |
| Linear activations | any depth = one linear map |
| Universal approximation | 1 hidden layer, enough nonlinear units |
| Vanishing gradients | sigmoid/tanh deep nets; ReLU helps |

## Clustering

| Item | Fact |
| --- | --- |
| k-means | assign to nearest centroid, recompute means; SSE non-increasing; local optimum; $O(nkd)$ per iteration; elbow to choose $k$ |
| k-medoids (PAM) | centre is a data point; any dissimilarity; robust to outliers; $O(k(n-k)^2)$ |
| Agglomerative | start $n$ singletons, $n-1$ merges; $O(n^3)$ naive, $O(n^2)$ memory |
| Divisive | start with one cluster, split recursively |
| Linkage ("multiple linkage") | single = min pair; complete = max pair; average = mean of pairs; Ward = min SSE increase |
| Single linkage | = Kruskal/MST order; chaining |
| Complete linkage | compact clusters; outlier-sensitive |
| Cut dendrogram | clusters $=n-\#\text{merges below cut}$ |

## PCA

| Item | Fact |
| --- | --- |
| Steps | centre $\to$ $S=\frac1{n-1}X_c^TX_c\to$ eigen $\to$ sort $\to$ project $Z=X_cV_m$ |
| Eigenvalue | variance along PC; $\sum\lambda=\text{tr}(S)$ |
| Explained ratio | $\lambda_i/\sum\lambda_j$ |
| $2\times2$ | $\lambda^2-\text{tr}\,\lambda+\det=0$ |
| SVD | $X_c=U\Sigma V^T$; PCs $=V$; $\lambda=\sigma^2/(n-1)$ |
| Max useful PCs | $\min(n-1,d)$ |
| PCA vs LDA | PCA unsupervised (max variance); LDA supervised (max class separation) |
| Standardise | when features have different units |

## Remember

- Ridge $\lambda$ large = more regularisation; SVM $C$ large = *less* regularisation.
- LOOCV: $n$ fits; $k$-fold: $k$ fits.
- Single linkage uses MIN; complete uses MAX.
- Count biases when counting neural network parameters.
- Centre the data before PCA.

[Back to section README](README.md) · [Checkpoint](CHECKPOINT.md)
