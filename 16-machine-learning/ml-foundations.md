# ML foundations: problem types, loss, bias-variance, cross-validation, metrics

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Regression vs classification problems; Bias-variance trade-off; Leave-one-out cross-validation; k-fold cross-validation
> **Prerequisites:** [Probability basics](../02-probability-statistics/probability-basics.md) · [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) · **Leads to:** [Linear and logistic regression](linear-and-logistic-regression.md) · [Classification methods](classification-methods.md) · [Clustering](clustering.md)

## Quick glance

- **Supervised** learning: data are pairs $(x_i, y_i)$; learn $f$ with $f(x) \approx y$. **Unsupervised**: only $x_i$; find structure (clusters, low-dimensional directions).
- **Regression**: $y$ is a real number. **Classification**: $y$ is a discrete label.
- Split data into **train** (fit parameters), **validation** (choose hyper-parameters), **test** (touch once, report final error).
- Expected squared error $= \text{bias}^2 + \text{variance} + \text{irreducible noise}$. More complexity: **bias down, variance up**.
- **k-fold CV** trains the model **k** times; **LOOCV** is k = n (n fits): nearly unbiased estimate, high variance, costly.
- Precision $= TP/(TP+FP)$, recall $= TP/(TP+FN)$, F1 = harmonic mean. **Accuracy is misleading on imbalanced classes.**
- Training error always falls with complexity; **test error is U-shaped**.
- #1 trap: **tuning on the test set** (or scaling/selecting features using all data before splitting) leaks information.

## 1. What problem are we solving?

**Intuition.** A learner is handed examples and must generalise to new cases. Whether it is given the right answers decides the family of methods.

| Setting | Data | Goal | Examples in this guide |
| --- | --- | --- | --- |
| Supervised, regression | $(x, y)$, $y \in \mathbb{R}$ | predict a number | linear/ridge regression, regression trees |
| Supervised, classification | $(x, y)$, $y \in \{1..K\}$ | predict a class | logistic regression, k-NN, naive Bayes, LDA, SVM, trees, MLP |
| Unsupervised | $x$ only | find groups or directions | k-means, hierarchical, PCA |

**Regression vs classification is decided by the output type, not by the algorithm.** "Logistic regression" is a classifier despite its name (it regresses a probability, then thresholds). Predicting a house price is regression; predicting "spam / not spam" is classification; predicting a rating 1..5 can be treated either way.

A **model** is a function $f_\theta$ with parameters $\theta$ (weights). **Hyper-parameters** (k in k-NN, $\lambda$ in ridge, tree depth, C in SVM, number of clusters) are set *outside* the fitting procedure, typically by validation.

## 2. Loss functions and empirical risk

A **loss** $\ell(y, \hat{y})$ scores one prediction. The **empirical risk** is the average loss on the training set, and learning (empirical risk minimisation) picks $\theta$ minimising it:

$$\hat{R}(\theta) = \frac{1}{n}\sum_{i=1}^{n} \ell\big(y_i, f_\theta(x_i)\big)$$

| Loss | Formula | Used for | Notes |
| --- | --- | --- | --- |
| Squared | $(y-\hat y)^2$ | regression | smooth, closed form for linear models, sensitive to outliers |
| Absolute | $\lvert y-\hat y\rvert$ | robust regression | minimiser is the **median** (squared: the **mean**) |
| 0-1 | $\mathbb{1}[y \ne \hat y]$ | classification error | not differentiable; not optimised directly |
| Cross-entropy (log loss) | $-[y\ln p + (1-y)\ln(1-p)]$ | probabilistic classification | logistic regression, softmax networks |
| Hinge | $\max(0,\, 1 - y\,f(x))$, $y \in \{\pm1\}$ | SVM | zero once the point is beyond the margin |

**Worked example (which constant minimises the loss?).** Data $y = \{1, 2, 9\}$. Predict one constant $c$.
- Squared loss: $\sum (y_i - c)^2$ minimal at $c = \text{mean} = 4$, value $9 + 4 + 25 = 38$.
- Absolute loss: $\sum |y_i - c|$ minimal at $c = \text{median} = 2$, value $1 + 0 + 7 = 8$ (at $c=4$ it is $3+2+5=10$).

**Squared loss is pulled by the outlier 9; absolute loss is not.**

**MLE view.** Many losses are negative log-likelihoods. If $y = f(x) + \varepsilon$ with $\varepsilon \sim N(0,\sigma^2)$, maximising likelihood = minimising squared error. If $y \sim \text{Bernoulli}(p(x))$, maximising likelihood = minimising cross-entropy. Naive Bayes and LDA estimate their parameters by MLE too (class frequencies, class means). See [probability basics](../02-probability-statistics/probability-basics.md).

## 3. Overfitting, underfitting, and the three data splits

- **Underfitting**: model too simple; high error on training *and* test data.
- **Overfitting**: model memorises noise; very low training error, high test error.

```text
error
  |\                         test error
  | \                       /
  |  \         ___________/
  |   \_______/   <- sweet spot
  |    \_____________  training error (keeps falling)
  +--------------------------> model complexity
   underfit        overfit
```

| Split | Purpose | Used how often |
| --- | --- | --- |
| Training | fit parameters | many times |
| Validation | choose hyper-parameters / model | many times |
| Test | unbiased final estimate | **once** |

Typical fixes for overfitting: more data, simpler model, regularisation (ridge/lasso, early stopping, dropout), pruning (trees), larger k (k-NN), larger margin (smaller C in SVM). Fixes for underfitting: richer features, more flexible model, less regularisation.

## 4. Bias-variance decomposition

**Intuition.** Imagine drawing many training sets of the same size and training the same model on each. At a fixed input $x_0$ the predictions $\hat f(x_0)$ scatter. **Bias** = how far the *average* prediction is from the truth (systematic error from wrong assumptions). **Variance** = how much predictions *scatter* around their own average (sensitivity to the particular training set).

For $y = f(x) + \varepsilon$, $E[\varepsilon]=0$, $\text{Var}(\varepsilon)=\sigma^2$:

$$E\big[(y - \hat f(x_0))^2\big] = \underbrace{\big(E[\hat f(x_0)] - f(x_0)\big)^2}_{\text{bias}^2} + \underbrace{E\big[(\hat f(x_0) - E[\hat f(x_0)])^2\big]}_{\text{variance}} + \underbrace{\sigma^2}_{\text{irreducible}}$$

**Worked example.** True value $f(x_0) = 5$, noise variance $\sigma^2 = 1$. Five models trained on five different training sets predict $6, 8, 7, 7, 7$.
1. Mean prediction $= 35/5 = 7$. Bias $= 7 - 5 = 2$, so $\text{bias}^2 = 4$.
2. Variance $= \frac{(6-7)^2+(8-7)^2+0+0+0}{5} = 2/5 = 0.4$.
3. Expected squared error $= 4 + 0.4 + 1 = \mathbf{5.4}$.

How the knobs move each term:

| Change | Bias | Variance |
| --- | --- | --- |
| More model complexity (higher polynomial degree, deeper tree, more hidden units) | down | **up** |
| More regularisation (larger ridge $\lambda$, larger tree pruning) | **up** | down |
| Larger k in k-NN | up | down (k = 1: low bias, high variance) |
| Deeper decision tree | down | up |
| More training examples | ~unchanged | down |
| More (irrelevant) features | ~unchanged | up |
| Larger SVM C (narrower margin) | down | up |
| Bagging / averaging many models | ~unchanged | **down** |

**Irreducible noise $\sigma^2$ is unaffected by any model choice.** The total expected error therefore is a U-shape in complexity, with the minimum where the two reducible terms balance.

## 5. Cross-validation

**Why.** One validation split wastes data and is noisy. Cross-validation reuses every point for both training and validation.

**k-fold CV.**
1. Shuffle and split the $n$ points into $k$ equal folds.
2. For each fold $j$: train on the other $k-1$ folds, measure the error on fold $j$.
3. Report the **mean** of the $k$ errors.

- Number of model fits $= k$; each fit uses $n(k-1)/k$ points.
- **Stratified** k-fold keeps the class proportions in every fold (important for imbalanced classes).
- The final model is then usually refit on all $n$ points.

**Worked example.** $n = 100$, 5-fold. Each fold has 20 points; each training set has 80. Validation errors on the five folds: $0.20, 0.30, 0.10, 0.40, 0.00$. CV error $= 1.00/5 = 0.20$. Total fits $= 5$.

**LOOCV (leave-one-out).** $k = n$: train on $n-1$ points, test on the single left-out point, repeat $n$ times, average.
- **n fits**, each on $n-1$ points. For $n = 100$: 100 fits (vs 10 for 10-fold).
- **Almost unbiased** (training sets are nearly the full data) but **high variance** (the $n$ training sets are almost identical, so the $n$ errors are highly correlated) and **expensive**.
- Deterministic: no random shuffling, so repeating it gives the same answer.
- For linear regression with hat matrix $H = X(X^TX)^{-1}X^T$ there is a shortcut needing one fit: $\text{LOOCV} = \frac{1}{n}\sum_i \left(\frac{e_i}{1-h_{ii}}\right)^2$.

| | k-fold (k = 5 or 10) | LOOCV |
| --- | --- | --- |
| Fits | $k$ | $n$ |
| Bias of error estimate | slightly high | lowest |
| Variance of estimate | lower | higher |
| Cost | moderate | highest |

**Worked example (LOOCV with a trivial model).** Predict by the mean of the training targets, data $y = \{2, 4, 9\}$. Squared error:
- leave out 2: train mean of $\{4,9\} = 6.5$, error $(2-6.5)^2 = 20.25$
- leave out 4: mean of $\{2,9\} = 5.5$, error $2.25$
- leave out 9: mean of $\{2,4\} = 3$, error $36$
- LOOCV $= (20.25 + 2.25 + 36)/3 = 58.5/3 = \mathbf{19.5}$ (3 fits).

## 6. Evaluation metrics for classification

Confusion matrix (positive = class of interest):

| | Predicted + | Predicted - |
| --- | --- | --- |
| **Actual +** | TP | FN |
| **Actual -** | FP | TN |

$$\text{Accuracy}=\frac{TP+TN}{N},\quad \text{Precision}=\frac{TP}{TP+FP},\quad \text{Recall (sensitivity, TPR)}=\frac{TP}{TP+FN},$$
$$\text{Specificity (TNR)}=\frac{TN}{TN+FP},\quad F_1 = \frac{2PR}{P+R}=\frac{2TP}{2TP+FP+FN}$$

**Worked example.** $TP=40, FP=10, FN=20, TN=130$ ($N=200$).
- Accuracy $= 170/200 = 0.85$.
- Precision $= 40/50 = 0.80$. Recall $= 40/60 = 0.667$. Specificity $= 130/140 = 0.929$.
- $F_1 = 2(0.8)(0.667)/(1.467) = 0.727$ (check: $2\cdot 40/(80+10+20) = 80/110 = 0.727$ ✓).

**Why accuracy fails.** 990 negatives, 10 positives; a classifier that always says "negative" has accuracy 99% and recall 0. Use precision, recall, F1 or AUC.

**Precision vs recall trade-off.** Lowering the decision threshold raises recall and (usually) lowers precision. Medical screening wants high recall; spam filtering of important mail wants high precision.

For regression the standard metrics are MSE, RMSE, MAE and $R^2$ (see [linear regression](linear-and-logistic-regression.md)).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Bias-variance | $\text{Err} = \text{bias}^2 + \text{var} + \sigma^2$ | any "what happens when complexity changes" question |
| Empirical risk | $\frac1n\sum \ell(y_i, f(x_i))$ | defining what training minimises |
| Squared / absolute loss minimiser | mean / median | robustness questions |
| k-fold fits | $k$ (each on $n(k-1)/k$ points) | counting cost |
| LOOCV fits | $n$ (each on $n-1$ points) | counting cost |
| Precision, recall | $TP/(TP+FP)$, $TP/(TP+FN)$ | classification metrics |
| $F_1$ | $2TP/(2TP+FP+FN)$ | single-number summary |
| Training error | non-increasing in complexity | spotting false "training error rises" options |

## GATE traps

- **"Training error is a good estimate of test error"** is false; it is optimistically biased and shrinks with complexity.
- **Test error does not keep decreasing with complexity**; it is U-shaped.
- Confusing which direction: **high-degree polynomial = low bias, high variance**; heavy regularisation = high bias, low variance.
- **Variance falls with more training data, bias does not** (for a fixed model family).
- LOOCV is **not** "least variance": it has the *lowest bias* but *higher variance* than 5/10-fold.
- Counting fits: 10-fold on 100 points is **10** fits, LOOCV is **100**, not 99.
- Using the test set (or all data) for scaling, feature selection or hyper-parameter choice before splitting = **data leakage**.
- Precision and recall have **different denominators** (predicted positives vs actual positives); a swap flips the answer.
- Accuracy on an imbalanced set can be high for a useless model; F1 ignores TN.
- Irreducible error cannot be reduced by any model.

## Connections

- [Probability basics](../02-probability-statistics/probability-basics.md) — Bayes' rule underlies naive Bayes and conditional probabilities used in precision/recall.
- [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — bias, variance and expectation are the same moments studied there.
- [Linear and logistic regression](linear-and-logistic-regression.md) — first models to apply loss, MLE and the bias-variance picture (ridge).
- [Classification methods](classification-methods.md) — k, tree depth and C are the knobs from the bias-variance table.
- [Neural networks](neural-networks.md) — the most flexible models; overfit easily; early stopping.
- [Clustering](clustering.md) — unsupervised counterpart; no labels, so no CV error.
- [Statistical inference](../02-probability-statistics/statistical-inference.md) — confidence intervals and tests quantify the same estimation uncertainty.

## Practice

**Q1 (MCQ).** Which statement about a model that has zero training error but high test error is correct?
(A) high bias (B) high variance (C) high irreducible noise (D) underfitting

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Memorising the training set (zero training error) with poor generalisation is overfitting, i.e. low bias and high variance.

</details>

**Q2 (NAT).** A dataset of 120 points is used for 6-fold cross-validation. How many points are in each training set, and how many models are fitted in total? Give the number of points.

<details><summary>Answer</summary>

**Answer:** 100 points per training set (6 fits)  
**Solution:** Each fold has $120/6 = 20$ points; training uses the other 5 folds: $5 \times 20 = 100$. Fits $= k = 6$.

</details>

**Q3 (MCQ).** Increasing k in k-NN from 1 to 15 typically
(A) decreases bias and increases variance (B) increases bias and decreases variance (C) decreases both (D) leaves both unchanged

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** k = 1 follows every noisy point (low bias, high variance). Averaging over 15 neighbours smooths the boundary: more bias, less variance.

</details>

**Q4 (NAT).** A classifier gives $TP = 30$, $FP = 20$, $FN = 10$, $TN = 40$. Compute the $F_1$ score to 2 decimals.

<details><summary>Answer</summary>

**Answer:** 0.67  
**Solution:** $F_1 = 2TP/(2TP+FP+FN) = 60/(60+20+10) = 60/90 = 0.667$. (Precision $30/50 = 0.6$, recall $30/40 = 0.75$; harmonic mean $= 2(0.6)(0.75)/1.35 = 0.667$ ✓.)

</details>

**Q5 (MSQ).** Which of the following reduce the **variance** of a flexible model? (A) more training data (B) stronger regularisation (C) averaging many models trained on bootstrap samples (D) adding many noisy features

<details><summary>Answer</summary>

**Answer:** A, B, C  
**Solution:** More data, regularisation and bagging all lower variance. Extra noisy features increase it.

</details>

**Q6 (NAT).** Targets are $y = \{1, 3, 8\}$ and the model always predicts the mean of the training fold. Compute the LOOCV mean squared error.

<details><summary>Answer</summary>

**Answer:** 19.5  
**Solution:**
- Leave out 1: mean of $\{3,8\} = 5.5$; error $(1-5.5)^2 = 20.25$.
- Leave out 3: mean of $\{1,8\} = 4.5$; error $(3-4.5)^2 = 2.25$.
- Leave out 8: mean of $\{1,3\} = 2$; error $(8-2)^2 = 36$.
- Average $= (20.25+2.25+36)/3 = 58.5/3 = 19.5$.

</details>

**Q7 (MSQ).** Regarding LOOCV vs 10-fold CV on $n = 500$ points, which are true? (A) LOOCV needs 500 fits (B) LOOCV estimate is nearly unbiased (C) LOOCV has strictly lower variance than 10-fold (D) 10-fold trains on 450 points per fit

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:** LOOCV: $n=500$ fits on 499 points, nearly unbiased. Its variance is typically *higher* because the training sets overlap almost completely, so (C) is false. 10-fold: $500 \cdot 9/10 = 450$ ✓.

</details>

**Q8 (NAT).** Expected squared error at a point is 7.0. The noise variance is 1.5 and the variance of the predictor is 2.0. What is the squared bias?

<details><summary>Answer</summary>

**Answer:** 3.5  
**Solution:** $7.0 = \text{bias}^2 + 2.0 + 1.5 \Rightarrow \text{bias}^2 = 3.5$.

</details>
