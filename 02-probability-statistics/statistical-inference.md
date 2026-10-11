# Statistical inference: sampling, intervals, and tests

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** Sampling distributions, Central Limit Theorem (CLT), point estimation, confidence intervals, hypothesis testing, Type I and Type II errors, p-values, z-test, t-test, Chi-squared test
> **Prerequisites:** [Continuous distributions](continuous-distributions.md) · [Random variables and moments](random-variables-and-moments.md) · **Leads to:** [Machine learning foundations](../16-machine-learning/ml-foundations.md)
> **PYQ weightage:** Medium · 6 Qs since 2015 (6 since 2021) · DA: 3 · Trend: rising · Most asked: Hypothesis testing, Confidence intervals, Central Limit Theorem

## Quick glance

- **Exam focus:** Central Limit Theorem (standard error $\sigma/\sqrt{n}$, sample size scaling), Confidence Intervals (critical $z$ vs $t$ values, margin of error), and Hypothesis Testing (Type I/Type II error definitions, power $1-\beta$, $p$-value interpretation) are the core statistical concepts tested in GATE DA. Questions are primarily NAT and conceptual MCQs/MSQs.
- **Statistic vs Parameter:** A **parameter** (e.g., population mean $\mu$, variance $\sigma^2$) is a fixed unknown population constant. A **statistic** (e.g., sample mean $\bar{X}$, sample variance $S^2$) is a random variable computed from sample data.
- **Standard error:** The standard deviation of a statistic's sampling distribution is its **standard error** (SE). For the sample mean, $\text{SE}(\bar{X}) = \frac{\sigma}{\sqrt{n}}$. Quadrupling $n$ cuts the standard error in half.
- **Central Limit Theorem (CLT):** For any i.i.d. sample $X_1, \dots, X_n$ from a distribution with mean $\mu$ and finite variance $\sigma^2$, $\frac{\bar{X} - \mu}{\sigma/\sqrt{n}} \xrightarrow{d} N(0, 1)$ as $n \to \infty$, regardless of the underlying population shape.
- **Confidence Interval (CI):** A $100(1-\alpha)\%$ CI for $\mu$ with known $\sigma$ is $\bar{x} \pm z_{\alpha/2}\frac{\sigma}{\sqrt{n}}$. With unknown $\sigma$ and normal population, use $\bar{x} \pm t_{\alpha/2, n-1}\frac{s}{\sqrt{n}}$.
- **CI interpretation trap:** A 95% CI does *not* mean there is a 0.95 probability that the fixed parameter $\mu$ lies in this specific calculated interval. It means that under hypothetical repeated sampling, 95% of constructed intervals will contain $\mu$.
- **Errors in hypothesis testing:**
  - **Type I error ($\alpha$):** Rejecting $H_0$ when $H_0$ is true (False Positive). Significance level $= \alpha$.
  - **Type II error ($\beta$):** Failing to reject $H_0$ when $H_0$ is false (False Negative).
  - **Power of the test ($1 - \beta$):** Probability of correctly rejecting a false $H_0$. At a fixed $\alpha$, increasing $n$ reduces $\beta$ (raises power).
- **$p$-value:** The probability under $H_0$ of observing a test statistic at least as extreme as the observed value. Reject $H_0$ if $p \le \alpha$. A $p$-value is **not** $P(H_0 \mid \text{data})$.
- **Chi-Square test:** $\chi^2 = \sum \frac{(O_i - E_i)^2}{E_i}$. For goodness-of-fit, $\text{df} = k - 1 - m$; for an $r \times c$ contingency table, $\text{df} = (r - 1)(c - 1)$.

## 1. Sampling distributions and Central Limit Theorem [PYQ: Medium]

A sample is a subset of observations drawn from a larger population. Because different random samples produce different numerical summaries, any sample statistic is itself a random variable with its own probability distribution, called its **sampling distribution**.

### 1.1 Sample mean and sample variance

Let $X_1, X_2, \dots, X_n$ be an independent and identically distributed (i.i.d.) random sample from a population with mean $\mu$ and variance $\sigma^2$.

1. **Sample Mean ($\bar{X}$):**
   $$\bar{X} = \frac{1}{n}\sum_{i=1}^n X_i, \qquad E[\bar{X}] = \mu, \qquad \text{Var}(\bar{X}) = \frac{\sigma^2}{n}, \qquad \text{SE}(\bar{X}) = \frac{\sigma}{\sqrt{n}}$$
   $\bar{X}$ is an **unbiased** estimator of $\mu$ because $E[\bar{X}] = \mu$.
2. **Sample Variance ($S^2$):**
   $$S^2 = \frac{1}{n-1}\sum_{i=1}^n (X_i - \bar{X})^2$$
   The divisor $n-1$ (Bessel's correction) ensures that $E[S^2] = \sigma^2$. Dividing by $n$ produces a downward-biased estimator with expectation $\frac{n-1}{n}\sigma^2$.

### 1.2 The Central Limit Theorem (Lindeberg–Lévy CLT)

Let $X_1, \dots, X_n$ be i.i.d. random variables with $E[X_i] = \mu$ and $\text{Var}(X_i) = \sigma^2 < \infty$. As $n \to \infty$, the standardised sample mean converges in distribution to the standard normal:
$$\mathbf{Z_n = \frac{\bar{X} - \mu}{\sigma / \sqrt{n}} \xrightarrow{d} N(0, 1)}$$
Equivalently, the sample sum $S_n = \sum_{i=1}^n X_i$ is asymptotically normal:
$$S_n \approx N(n\mu, \; n\sigma^2)$$

Key facts for GATE:
- CLT requires **finite variance** (it fails for heavy-tailed distributions like Cauchy).
- The parent population does not need to be symmetric or continuous; CLT works for Bernoulli, Poisson, Uniform, or Exponential variables.
- Practical guideline: $n \ge 30$ is generally sufficient for the normal approximation.

**Worked example 1 (CLT on sample sum).**
A data center receives packets whose sizes $X_i$ are independent and uniformly distributed over $[0, 10]$ KB. A buffer collects a batch of $n = 48$ packets. Using the Central Limit Theorem, approximate the probability that the total batch size exceeds 270 KB. (Given $\Phi(1.5) \approx 0.9332$).

*Solution:*
For each packet $X_i \sim U[0, 10]$:
$$\mu = \frac{0 + 10}{2} = 5\text{ KB}, \qquad \sigma^2 = \frac{(10 - 0)^2}{12} = \frac{100}{12} = \frac{25}{3}\text{ KB}^2$$
For the batch sum $S_{48} = \sum_{i=1}^{48} X_i$:
$$E[S_{48}] = n\mu = 48 \times 5 = 240\text{ KB}$$
$$\text{Var}(S_{48}) = n\sigma^2 = 48 \times \frac{25}{3} = 16 \times 25 = 400 \implies \sigma_{S_{48}} = \sqrt{400} = 20\text{ KB}$$
By CLT, $S_{48} \sim N(240, 400)$. Standardise to find $P(S_{48} > 270)$:
$$P(S_{48} > 270) = P\left(Z > \frac{270 - 240}{20}\right) = P(Z > 1.5) = 1 - \Phi(1.5) \approx 1 - 0.9332 = \mathbf{0.0668}$$

---

## 2. Estimation and Confidence Intervals [PYQ: Medium]

### 2.1 Desirable properties of point estimators
- **Unbiasedness:** $\text{Bias}(\hat{\theta}) = E[\hat{\theta}] - \theta = 0$.
- **Consistency:** $\lim_{n \to \infty} P(|\hat{\theta}_n - \theta| < \epsilon) = 1$ for all $\epsilon > 0$.
- **Efficiency:** Among unbiased estimators, the one with minimal variance is most efficient.

### 2.2 Confidence intervals for the population mean $\mu$

A $(1 - \alpha)$ confidence interval provides an interval $[\hat{\theta}_L, \hat{\theta}_U]$ constructed such that $P(\hat{\theta}_L \le \theta \le \hat{\theta}_U) = 1 - \alpha$.

| Scenario | Population / Assumptions | Formula for $(1-\alpha)$ CI | Critical value |
|---|---|---|---|
| **$\sigma$ known** | Normal or large $n$ ($n \ge 30$) | $\mathbf{\bar{x} \pm z_{\alpha/2} \frac{\sigma}{\sqrt{n}}}$ | $z_{0.025} = 1.96$ for 95% |
| **$\sigma$ unknown** | Normal population, small/any $n$ | $\mathbf{\bar{x} \pm t_{\alpha/2, n-1} \frac{s}{\sqrt{n}}}$ | $t_{\alpha/2, n-1}$ table |
| **Proportion $p$** | Large sample ($n\hat{p} \ge 5, n(1-\hat{p}) \ge 5$) | $\mathbf{\hat{p} \pm z_{\alpha/2} \sqrt{\frac{\hat{p}(1-\hat{p})}{n}}}$ | $z_{\alpha/2}$ from standard normal |

### 2.3 Margin of error and sample size determination
The half-width of the confidence interval is the **margin of error** $E$:
$$E = z_{\alpha/2}\frac{\sigma}{\sqrt{n}}$$
To guarantee a margin of error at most $E$ at confidence level $1 - \alpha$, the required minimum sample size is:
$$\mathbf{n = \left(\frac{z_{\alpha/2} \cdot \sigma}{E}\right)^2}$$
Always round $n$ **up** to the nearest integer.

**Worked example 2 (Confidence interval and sample size).**
A machine learning engineer benchmarks inference latency on a GPU. A sample of $n = 36$ runs yields sample mean $\bar{x} = 82\text{ ms}$ with sample standard deviation $s = 12\text{ ms}$.
1. Construct a 95% confidence interval for the true mean latency $\mu$ (using $z_{0.025} = 1.96$ by large-sample approximation).
2. What minimum sample size $n$ is required to achieve a margin of error $E \le 1.5\text{ ms}$ at 95% confidence?

*Solution:*
1. Standard error $\text{SE} = \frac{s}{\sqrt{n}} = \frac{12}{\sqrt{36}} = \frac{12}{6} = 2.0\text{ ms}$.
   $$\text{CI}_{95\%} = \bar{x} \pm z_{0.025}\cdot \text{SE} = 82 \pm 1.96(2.0) = 82 \pm 3.92 = \mathbf{[78.08\text{ ms}, \; 85.92\text{ ms}]}$$
2. For $E = 1.5$:
   $$n = \left(\frac{1.96 \times 12}{1.5}\right)^2 = \left(\frac{23.52}{1.5}\right)^2 = (15.68)^2 = 245.86$$
   Rounding up gives minimum sample size $\mathbf{n = 246}$.

---

## 3. Hypothesis testing: errors, power, and p-values [PYQ: Medium]

In statistical hypothesis testing, we evaluate competing claims about a population parameter:
- **Null Hypothesis ($H_0$):** Default claim of "no effect", "no difference", or equality (e.g., $H_0: \mu = \mu_0$).
- **Alternative Hypothesis ($H_1$):** Research claim of an effect or difference.
  - Two-tailed: $H_1: \mu \ne \mu_0$.
  - One-tailed (right): $H_1: \mu > \mu_0$; One-tailed (left): $H_1: \mu < \mu_0$.

### 3.1 Type I error, Type II error, and Power

| Decision \ True State | $H_0$ is TRUE | $H_0$ is FALSE ($H_1$ is TRUE) |
|---|---|---|
| **Reject $H_0$** | **Type I error** (prob = $\alpha$, False Positive) | **Correct decision** (prob = $1 - \beta$, Power) |
| **Fail to reject $H_0$** | **Correct decision** (prob = $1 - \alpha$) | **Type II error** (prob = $\beta$, False Negative) |

Mathematical definitions:
- **Significance level ($\alpha$):** $\alpha = P(\text{Reject } H_0 \mid H_0 \text{ is true})$. Set beforehand (typically 0.05 or 0.01).
- **Type II error probability ($\beta$):** $\beta = P(\text{Fail to reject } H_0 \mid H_1 \text{ is true})$. Depends on the specific true value under $H_1$.
- **Statistical Power ($1 - \beta$):** Probability of correctly detecting an effect when $H_0$ is false: $\text{Power} = P(\text{Reject } H_0 \mid H_1 \text{ is true})$.
- **Sample size effect:** For fixed sample size $n$, decreasing $\alpha$ increases $\beta$. Increasing $n$ lowers $\beta$ at the same $\alpha$, or lets you lower both if the cutoff is re-chosen; with the rejection rule fixed in z-terms, $\alpha$ itself does not change.

```text
       Under H0 (mu=20)              Under H1 (mu=23)
             /\                            /\
            /  \                          /  \
           /    \                        /    \
      ----+------+-----+------------+---+------+-----+---> x_bar
         18      20   22 (threshold)    23     25
                       |<- alpha ->|
          |<-   beta   ->|
```

### 3.2 The p-value approach
The **$p$-value** is the probability under the null hypothesis $H_0$ of observing a test statistic at least as extreme as the observed sample statistic:
$$\text{Decision rule: Reject } H_0 \text{ if } p\text{-value} \le \alpha$$
- A small $p$-value ($p \le \alpha$) indicates data that are surprising under $H_0$, leading to rejection.
- A large $p$-value ($p > \alpha$) indicates data compatible with $H_0$; we fail to reject $H_0$ (we do *not* "accept" $H_0$).

**Worked example 3 (Calculating $\alpha$, $\beta$, and power).**
Observations come from $X \sim N(\mu, \sigma^2 = 16)$. A test of $H_0: \mu = 20$ versus $H_1: \mu = 23$ is conducted using a sample of size $n = 16$. The rejection rule is: Reject $H_0$ if $\bar{X} \ge 22$.
1. Find the significance level $\alpha$ (Type I error probability).
2. Find the Type II error probability $\beta$ when the true mean is $\mu = 23$.
3. What is the power of the test?

*Solution:*
Standard error $\sigma_{\bar{X}} = \frac{\sigma}{\sqrt{n}} = \frac{4}{\sqrt{16}} = \frac{4}{4} = 1.0$.
1. Under $H_0$, $\bar{X} \sim N(20, 1^2)$:
   $$\alpha = P(\bar{X} \ge 22 \mid \mu = 20) = P\left(Z \ge \frac{22 - 20}{1}\right) = P(Z \ge 2) = 1 - \Phi(2) = 1 - 0.9772 = \mathbf{0.0228}$$
2. Under $H_1$ ($\mu = 23$), $\bar{X} \sim N(23, 1^2)$:
   $$\beta = P(\bar{X} < 22 \mid \mu = 23) = P\left(Z < \frac{22 - 23}{1}\right) = P(Z < -1) = \Phi(-1) = 1 - \Phi(1) = 1 - 0.8413 = \mathbf{0.1587}$$
3. Power of the test:
   $$\text{Power} = 1 - \beta = 1 - 0.1587 = \mathbf{0.8413}$$

---

## 4. Parametric tests: z-test and t-test [PYQ: Medium]

### 4.1 One-sample tests for the mean

| Test | When to use | Test statistic | Degrees of freedom |
|---|---|---|---|
| **One-sample $z$-test** | $\sigma$ known (or $n \ge 30$) | $\mathbf{z = \frac{\bar{x} - \mu_0}{\sigma / \sqrt{n}}}$ | Standard Normal $N(0, 1)$ |
| **One-sample $t$-test** | $\sigma$ unknown, normal population | $\mathbf{t = \frac{\bar{x} - \mu_0}{s / \sqrt{n}}}$ | $\nu = n - 1$ |

### 4.2 Two-sample and paired t-tests
- **Two independent samples (pooled variance, $\sigma_1^2 = \sigma_2^2$):**
  $$t = \frac{\bar{x}_1 - \bar{x}_2}{s_p \sqrt{\frac{1}{n_1} + \frac{1}{n_2}}}, \quad s_p^2 = \frac{(n_1-1)s_1^2 + (n_2-1)s_2^2}{n_1 + n_2 - 2}, \quad \text{df} = n_1 + n_2 - 2$$
- **Paired $t$-test (dependent samples, before/after measurements):**
  Compute differences $d_i = x_{1i} - x_{2i}$ for each pair ($i = 1, \dots, n$):
  $$\mathbf{t = \frac{\bar{d} - 0}{s_d / \sqrt{n}}}, \quad \text{df} = n - 1$$

**Worked example 4 (One-sample t-test).**
A database cache is claimed to have average access latency $\mu_0 = 20.0\text{ ms}$. A benchmark of $n = 9$ random queries gives $\bar{x} = 22.4\text{ ms}$ with sample standard deviation $s = 3.6\text{ ms}$. Test $H_0: \mu = 20.0$ against $H_1: \mu > 20.0$ at $\alpha = 0.05$. (Given: $t_{0.05, 8} = 1.860$).

*Solution:*
1. State hypotheses: $H_0: \mu = 20.0$ vs $H_1: \mu > 20.0$ (one-tailed right test).
2. Calculate test statistic:
   $$t = \frac{\bar{x} - \mu_0}{s / \sqrt{n}} = \frac{22.4 - 20.0}{3.6 / \sqrt{9}} = \frac{2.4}{3.6 / 3} = \frac{2.4}{1.2} = \mathbf{2.00}$$
3. Rejection region: With $\text{df} = n - 1 = 8$, reject $H_0$ if $t > t_{0.05, 8} = 1.860$.
4. Decision: Since $t = 2.00 > 1.860$, **Reject $H_0$** at the $5\%$ level. There is statistically significant evidence that cache latency exceeds 20 ms.

---

## 5. Chi-Square tests: goodness-of-fit and independence [PYQ: Low]

Pearson's Chi-Square test compares observed sample frequencies ($O_i$) against theoretical frequencies expected under the null hypothesis ($E_i$).

$$\mathbf{\chi^2 = \sum_{i=1}^k \frac{(O_i - E_i)^2}{E_i}}$$
Assumption: Expected count $E_i \ge 5$ in every cell.

### 5.1 Goodness-of-Fit test
Tests whether a categorical sample comes from a specified distribution.
- Null hypothesis: $H_0$: Data follow the specified distribution.
- Degrees of freedom: $\mathbf{\text{df} = k - 1 - m}$, where $k$ is the number of categories and $m$ is the number of population parameters estimated from the sample data.

### 5.2 Test of Independence (Contingency tables)
Tests whether two categorical variables in an $r \times c$ cross-tabulation are independent.
- Expected frequency for cell $(i, j)$:
  $$E_{ij} = \frac{(\text{Row Total } R_i) \times (\text{Column Total } C_j)}{\text{Grand Total } N}$$
- Test statistic: $\chi^2 = \sum_{i=1}^r \sum_{j=1}^c \frac{(O_{ij} - E_{ij})^2}{E_{ij}}$.
- Degrees of freedom: $\mathbf{\text{df} = (r - 1)(c - 1)}$.

**Worked example 5 (Contingency table independence).**
A 200-user sample tests whether user operating system (OS1 vs OS2) is independent of browser preference (B1 vs B2):

| OS \ Browser | B1 | B2 | Row Total |
|---|---|---|---|
| **OS1** | 40 | 60 | 100 |
| **OS2** | 60 | 40 | 100 |
| **Col Total** | 100 | 100 | $N = 200$ |

Compute the $\chi^2$ test statistic and state the degrees of freedom.

*Solution:*
Expected frequencies for all four cells ($i, j \in \{1, 2\}$):
$$E_{ij} = \frac{100 \times 100}{200} = 50$$
Compute $\chi^2$:
$$\chi^2 = \frac{(40 - 50)^2}{50} + \frac{(60 - 50)^2}{50} + \frac{(60 - 50)^2}{50} + \frac{(40 - 50)^2}{50}$$
$$= \frac{(-10)^2 + 10^2 + 10^2 + (-10)^2}{50} = \frac{100 + 100 + 100 + 100}{50} = \frac{400}{50} = \mathbf{8.00}$$
Degrees of freedom: $\text{df} = (2 - 1)(2 - 1) = \mathbf{1}$.

---

## Formulas and facts to memorise

| Concept / Test | Formula | Key condition / Degrees of freedom |
|---|---|---|
| **Standard Error ($\bar{X}$)** | $\text{SE}(\bar{X}) = \frac{\sigma}{\sqrt{n}}$ | Unbiased for any parent distribution with finite $\sigma$ |
| **Sample Variance** | $S^2 = \frac{1}{n-1}\sum (X_i - \bar{X})^2$ | Unbiased: $E[S^2] = \sigma^2$ (Bessel's correction) |
| **CI for mean ($\sigma$ known)** | $\bar{x} \pm z_{\alpha/2}\frac{\sigma}{\sqrt{n}}$ | $z_{0.025} = 1.96$ for 95% |
| **CI for mean ($\sigma$ unknown)** | $\bar{x} \pm t_{\alpha/2, n-1}\frac{s}{\sqrt{n}}$ | Normal population or $n \ge 30$; $\text{df} = n - 1$ |
| **Sample size formula** | $n = \left(\frac{z_{\alpha/2}\sigma}{E}\right)^2$ | Round up to nearest integer |
| **One-sample z-statistic** | $z = \frac{\bar{x} - \mu_0}{\sigma / \sqrt{n}}$ | Standard normal under $H_0$ |
| **One-sample t-statistic** | $t = \frac{\bar{x} - \mu_0}{s / \sqrt{n}}$ | $\text{df} = n - 1$ under $H_0$ |
| **Paired t-statistic** | $t = \frac{\bar{d}}{s_d / \sqrt{n}}$ | $d_i = x_{1i} - x_{2i}$; $\text{df} = n - 1$ |
| **Chi-Square test statistic** | $\chi^2 = \sum \frac{(O_i - E_i)^2}{E_i}$ | $E_i \ge 5$; GoF: $\text{df} = k - 1 - m$; Table: $\text{df} = (r-1)(c-1)$ |

---

## GATE traps

- **Misinterpreting confidence intervals:** The statement "there is a 95% probability that the population mean $\mu$ lies in $[48.04, 51.96]$" is **strictly false**. $\mu$ is a fixed constant, not a random variable. The 95% refers to the long-run coverage rate of the *interval procedure* across repeated samples.
- **Misinterpreting the $p$-value:** A $p$-value of 0.03 does **not** mean $P(H_0 \mid \text{data}) = 0.03$. It means that if $H_0$ were true, the probability of getting data as extreme as observed is 0.03.
- **Failing to reject $\ne$ accepting:** Failing to reject $H_0$ means there is insufficient evidence to disprove $H_0$; it does *not* prove that $H_0$ is true.
- **Divisor in standard error:** Remember that standard error divides by $\sqrt{n}$, not $n$. Doubling sample size shrinks SE by $\sqrt{2} \approx 1.414$, not by 2.
- **One-tailed vs. Two-tailed critical values:** For 95% confidence in a two-tailed test, tail area is $\alpha/2 = 0.025$ ($z = 1.96$). For a one-tailed test at $\alpha = 0.05$, critical value is $z = 1.645$.
- **Degrees of freedom in contingency tables:** Do not use $n - 1$ for contingency tables; use $(r - 1)(c - 1)$.

---

## Connections

- [Continuous distributions](continuous-distributions.md) — normal, Student's $t$, and Chi-square laws provide critical values and sampling distributions.
- [Probability basics](probability-basics.md) — conditional probability clarifies false positives, false discovery rates, and Bayes rule.
- [Machine learning foundations](../16-machine-learning/ml-foundations.md) — cross-validation and hypothesis testing validate model performance differences.

---

## Practice

**Q1 (NAT).** A sample of size $n = 100$ is drawn from a population with standard deviation $\sigma = 15$. The standard error of the sample mean is _________.

<details><summary>Answer</summary>

**Answer:** 1.5

**Solution:**
$$\text{SE}(\bar{X}) = \frac{\sigma}{\sqrt{n}} = \frac{15}{\sqrt{100}} = \frac{15}{10} = \mathbf{1.5}$$

</details>

**Q2 (MCQ).** A 95% confidence interval for the population mean latency of a web server is calculated as $[42\text{ ms}, 48\text{ ms}]$ from a random sample of 64 requests. Which of the following statements is mathematically CORRECT?
(A) There is a 95% probability that the true mean latency lies between 42 ms and 48 ms.
(B) 95% of all individual web server requests have latency between 42 ms and 48 ms.
(C) If the experiment is repeated many times, 95% of the resulting confidence intervals will contain the true population mean.
(D) The probability that the next sample mean falls in $[42\text{ ms}, 48\text{ ms}]$ is 0.95.

<details><summary>Answer</summary>

**Answer:** (C)

**Solution:**
In frequentist inference, the population parameter $\mu$ is a fixed constant, so it does not possess a probability distribution. The confidence coefficient 95% describes the long-term reliability of the estimation procedure: across repeated sampling under the same conditions, 95% of the calculated intervals will cover $\mu$. Options (A), (B), and (D) are classic statistical fallacies.

</details>

**Q3 (NAT).** A researcher wants to estimate a population mean with 95% confidence ($z_{0.025} = 1.96$). The population standard deviation is known to be $\sigma = 10$. The minimum sample size required to ensure that the margin of error does not exceed 2 is _________.

<details><summary>Answer</summary>

**Answer:** 97

**Solution:**
Margin of error $E = z_{\alpha/2}\frac{\sigma}{\sqrt{n}} \le 2$.
$$n \ge \left(\frac{z_{\alpha/2}\sigma}{E}\right)^2 = \left(\frac{1.96 \times 10}{2}\right)^2 = (9.8)^2 = 96.04$$
Since the sample size must be an integer, rounding up gives **97**.

</details>

**Q4 (MSQ).** In statistical hypothesis testing, which of the following statements is/are TRUE?
(A) Type I error occurs when a true null hypothesis is rejected.
(B) Increasing the sample size $n$ while holding significance level $\alpha$ constant increases the power of the test.
(C) The power of a hypothesis test is defined as $1 - \alpha$.
(D) If the $p$-value is less than the chosen significance level $\alpha$, the null hypothesis is rejected.

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)

**Solution:**
- (A) True: Type I error is rejecting $H_0$ when $H_0$ is true.
- (B) True: Increasing $n$ shrinks the standard error, making the test more sensitive to deviations from $H_0$, thus decreasing $\beta$ and increasing power $1 - \beta$.
- (C) False: Power is $1 - \beta$ (where $\beta$ is Type II error probability), not $1 - \alpha$.
- (D) True: By definition, $p \le \alpha$ constitutes sufficient evidence to reject $H_0$.

</details>

**Q5 (NAT).** A machine component is designed to have a target tensile strength of $\mu_0 = 100\text{ MPa}$. A quality inspector measures a random sample of $n = 64$ components and finds sample mean tensile strength $\bar{x} = 104\text{ MPa}$. The population standard deviation is known to be $\sigma = 16\text{ MPa}$. The value of the $z$-test statistic for testing $H_0: \mu = 100$ against $H_1: \mu \ne 100$ is _________.

<details><summary>Answer</summary>

**Answer:** 2 (or 2.0)

**Solution:**
$$z = \frac{\bar{x} - \mu_0}{\sigma / \sqrt{n}} = \frac{104 - 100}{16 / \sqrt{64}} = \frac{4}{16 / 8} = \frac{4}{2} = \mathbf{2.0}$$

</details>

**Q6 (MCQ).** A researcher tests $H_0: \mu = 50$ against $H_1: \mu \ne 50$ at significance level $\alpha = 0.05$ and obtains a $p$-value of 0.02. What should the researcher conclude?
(A) Accept $H_0$ because 0.02 is very small.
(B) Reject $H_0$ because $p \le \alpha$.
(C) Conclude that there is a 2% chance that $H_0$ is true.
(D) Conclude that there is a 98% chance that $H_1$ is true.

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
Because the $p$-value ($0.02$) is less than the significance level $\alpha$ ($0.05$), the observed test statistic is sufficiently improbable under $H_0$ to justify rejecting $H_0$. Options (C) and (D) falsely interpret the $p$-value as a posterior probability of the hypothesis.

</details>

**Q7 (NAT).** A 6-sided die is rolled 120 times to test whether it is fair. The observed face counts for faces 1 through 6 are 15, 25, 18, 22, 16, and 24, respectively. The value of Pearson's Chi-Square goodness-of-fit test statistic $\chi^2$ is _________.

<details><summary>Answer</summary>

**Answer:** 4.5

**Solution:**
Under the null hypothesis of a fair die, each face is equally likely ($p_i = 1/6$).
The expected frequency for each face is:
$$E_i = 120 \times \frac{1}{6} = 20 \quad \text{for all } i = 1, \dots, 6$$
Calculate the Chi-Square statistic:
$$\chi^2 = \sum_{i=1}^6 \frac{(O_i - E_i)^2}{E_i}$$
$$= \frac{(15 - 20)^2 + (25 - 20)^2 + (18 - 20)^2 + (22 - 20)^2 + (16 - 20)^2 + (24 - 20)^2}{20}$$
$$= \frac{(-5)^2 + 5^2 + (-2)^2 + 2^2 + (-4)^2 + 4^2}{20} = \frac{25 + 25 + 4 + 4 + 16 + 16}{20} = \frac{90}{20} = \mathbf{4.5}$$

</details>
