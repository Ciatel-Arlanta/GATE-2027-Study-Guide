# Machine Learning: checkpoint

Twelve mixed GATE-style questions on the whole section, easy to hard. Attempt all before opening any solution. Chapters: [foundations](ml-foundations.md) · [regression](linear-and-logistic-regression.md) · [classification](classification-methods.md) · [neural networks](neural-networks.md) · [clustering](clustering.md) · [PCA](dimensionality-reduction-pca.md).

**Q1 (NAT, foundations).** A dataset has $n=40$ points. How many more model fits does leave-one-out cross-validation need than 5-fold cross-validation?

<details><summary>Answer</summary>

**Answer:** 35. **Solution:** LOOCV fits $n=40$ models, 5-fold fits $5$; difference $35$.

</details>

**Q2 (NAT, regression).** Data $(x,y)$: $(1,3),(2,5),(3,7),(4,10)$. The least-squares prediction at $x=5$?

<details><summary>Answer</summary>

**Answer:** 12.0. **Solution:** $\bar x=2.5$, $\bar y=6.25$, $S_{xx}=5$, $S_{xy}=(-1.5)(-3.25)+(-0.5)(-1.25)+(0.5)(0.75)+(1.5)(3.75)=4.875+0.625+0.375+5.625=11.5$. $\hat\beta_1=2.3$, $\hat\beta_0=6.25-2.3(2.5)=0.5$. At $x=5$: $0.5+11.5=12.0$.

</details>

**Q3 (NAT, ridge).** For the same data fit ridge regression with $\lambda=5$ on the centred slope (intercept unpenalised). Slope?

<details><summary>Answer</summary>

**Answer:** 1.15. **Solution:** slope $=S_{xy}/(S_{xx}+\lambda)=11.5/(5+5)=1.15$ (shrunk from 2.3).

</details>

**Q4 (NAT, logistic).** A logistic regression has $w=(2,-1)$, $b=-1$. For $x=(1,3)$, what is $P(y=1\mid x)$? (3 decimals)

<details><summary>Answer</summary>

**Answer:** 0.119. **Solution:** $z=2-3-1=-2$; $\sigma(-2)=1/(1+e^{2})=0.1192$. Predict class 0.

</details>

**Q5 (MCQ, bias-variance).** Which change most likely reduces **variance** at the cost of higher bias? (a) decrease $k$ in k-NN (b) increase tree depth (c) increase ridge $\lambda$ (d) remove regularisation.

<details><summary>Answer</summary>

**Answer:** (c). **Solution:** larger $\lambda$ shrinks coefficients: simpler model. (a), (b), (d) all increase flexibility and variance.

</details>

**Q6 (NAT, decision trees).** A node has 6 positives and 2 negatives. A split gives child 1 $=(4+,0-)$ and child 2 $=(2+,2-)$. Information gain (3 decimals)?

<details><summary>Answer</summary>

**Answer:** 0.311. **Solution:** $H(\text{parent})=-0.75\log_20.75-0.25\log_20.25=0.8113$. Child entropies $0$ and $1$; weighted $\frac48(0)+\frac48(1)=0.5$. $IG=0.8113-0.5=0.3113$.

</details>

**Q7 (NAT, naive Bayes).** In class 1 there are 4 training examples; a binary feature equals 1 in 3 of them. Laplace-smoothed estimate of $P(x=1\mid\text{class }1)$?

<details><summary>Answer</summary>

**Answer:** 0.667. **Solution:** $(3+1)/(4+2)=2/3$ ($V=2$ values).

</details>

**Q8 (NAT, k-means).** Points $(0,0),(1,0),(0,1),(8,8),(9,8),(8,9)$, $k=2$, initial centroids $(0,0)$ and $(1,0)$. Final SSE?

<details><summary>Answer</summary>

**Answer:** 2.667. **Solution:** Iter 1: $(1,0)$ is nearest to centroid 2 (distance 0); $(0,1)$ and $(0,0)$ go to centroid 1; the three far points go to centroid 2. Centroids: $(0,0.5)$ and $(6.5,6.25)$. Iter 2: $(1,0)$ is now at $1.118$ from $(0,0.5)$ but $8.32$ from $(6.5,6.25)$, so it moves to cluster 1. Centroids: $(\frac13,\frac13)$ and $(\frac{25}3,\frac{25}3)$. Iter 3: no change. SSE per cluster $=\frac29+\frac59+\frac59=\frac{12}9$; total $\frac{24}9=2.667$.

</details>

**Q9 (NAT, hierarchical).** Distance matrix for $p_1..p_4$: $d_{12}=2$, $d_{13}=6$, $d_{14}=10$, $d_{23}=5$, $d_{24}=9$, $d_{34}=4$. Difference between the final merge heights of complete and single linkage?

<details><summary>Answer</summary>

**Answer:** 5. **Solution:** both first merge $(p_1,p_2)$ at 2, then $(p_3,p_4)$ at 4 (cross distances to $\{p_1,p_2\}$ are $\ge5$). Final merge: single $=\min(6,10,5,9)=5$; complete $=\max=10$. Difference $5$. (Average would be $7.5$.)

</details>

**Q10 (MSQ, PCA).** Data $(1,1),(2,2),(3,3)$. Which are true? (a) The covariance matrix is $\begin{pmatrix}1&1\\1&1\end{pmatrix}$. (b) PC1 explains 100% of the variance. (c) PC1 is along $(1,1)/\sqrt2$. (d) The second eigenvalue is 2.

<details><summary>Answer</summary>

**Answer:** (a), (b), (c). **Solution:** centred points $(-1,-1),(0,0),(1,1)$; $S=\frac12\begin{pmatrix}2&2\\2&2\end{pmatrix}=\begin{pmatrix}1&1\\1&1\end{pmatrix}$. Eigenvalues $2$ (along $(1,1)/\sqrt2$) and $0$. So PC1 explains $2/2=100\%$; (d) is false (the second eigenvalue is 0).

</details>

**Q11 (NAT, neural networks + linear algebra).** A network $3\to4\to2$ uses identity activations. (i) How many parameters does it have? (ii) How many independent numbers describe the equivalent single linear map $y=Ax+c$? Give (i)+(ii).

<details><summary>Answer</summary>

**Answer:** 34. **Solution:** (i) $(3+1)4+(4+1)2=16+10=26$. (ii) $A$ is $2\times3$ (6 numbers) and $c$ has 2: $8$. Total $26+8=34$. The collapse $A=W_2W_1$ shows depth adds parameters but no expressive power.

</details>

**Q12 (NAT, SVM).** A hard-margin linear SVM is trained on 1-D points $x=-2,-1$ (class $-1$) and $x=1,3$ (class $+1$). (i) Margin width? (ii) How many support vectors? Answer (i)+(ii).

<details><summary>Answer</summary>

**Answer:** 4. **Solution:** Closest opposite points are $-1$ and $1$, boundary at $0$. $w=1,b=0$ gives $w x+b=\mp1$ at $\mp1$. Margin $=2/\|w\|=2$. Support vectors: $-1$ and $1$ (the points $-2$ and $3$ satisfy the constraints strictly, e.g. $3\to3>1$): 2. Sum $=2+2=4$.

</details>

**Q13 (NAT, cross-validation).** A model predicts the mean of the training targets. For $y=(2,4,9)$, compute the LOOCV mean squared error.

<details><summary>Answer</summary>

**Answer:** 19.5. **Solution:** hold out 2: predict mean(4,9)$=6.5$, error$^2=20.25$; hold out 4: predict $5.5$, $2.25$; hold out 9: predict $3$, $36$. Mean $=(20.25+2.25+36)/3=19.5$.

</details>

**Q14 (MCQ, mixed).** Which pair of methods both produce a **linear decision boundary** under their standard assumptions? (a) QDA and RBF-SVM (b) LDA and logistic regression (c) 1-NN and LDA (d) decision tree and QDA.

<details><summary>Answer</summary>

**Answer:** (b). **Solution:** LDA (shared covariance) and logistic regression are linear; QDA, RBF-SVM and 1-NN have curved/piecewise boundaries; trees have axis-aligned (piecewise) boundaries.

</details>

## Scoring

Count correct answers out of 14.

- **$\ge 80\%$ (12+)**: move on to [17 Artificial Intelligence](../17-artificial-intelligence/README.md).
- **60-80%**: revise the chapters of the questions you missed, then retry them.
- **$<60\%$**: re-read the chapters below.

| Questions | Chapter to re-read |
| --- | --- |
| Q1, Q5, Q13 | [ML foundations](ml-foundations.md) |
| Q2, Q3, Q4 | [Linear and logistic regression](linear-and-logistic-regression.md) |
| Q6, Q7, Q12, Q14 | [Classification methods](classification-methods.md) |
| Q11 | [Neural networks](neural-networks.md) |
| Q8, Q9 | [Clustering](clustering.md) |
| Q10 | [Dimensionality reduction and PCA](dimensionality-reduction-pca.md) |

[Back to section README](README.md) · [Cheat sheet](CHEATSHEET.md)
