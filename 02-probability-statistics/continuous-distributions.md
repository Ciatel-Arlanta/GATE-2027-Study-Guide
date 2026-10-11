# Continuous distributions

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Continuous random variables, probability density function (PDF), cumulative distribution function (CDF), expectation and variance; Uniform distribution; Exponential distribution and memorylessness; Normal and standard normal distribution; t-distribution and Chi-squared distribution; Joint and conditional continuous distributions
> **Prerequisites:** [Random variables and moments](random-variables-and-moments.md) · **Leads to:** [Statistical inference](statistical-inference.md) · [Machine learning foundations](../16-machine-learning/ml-foundations.md)
> **PYQ weightage:** High · 19 Qs since 2015 (12 since 2021) · DA: 8 · Trend: rising · Most asked: Exponential, Uniform, Normal

## Quick glance

- **Exam focus:** Exponential distribution (memorylessness, rate parameter, survival probability) and Uniform distribution (geometric probability in 1D/2D, expected values) appear almost every year in GATE CS and DA; Normal standardisation ($Z$-table lookups, symmetry, linear combinations) is a core NAT topic. Questions are overwhelmingly NAT and 2-mark MCQs.
- **Continuous probability:** For any continuous random variable $X$, $P(X = c) = 0$ for any specific point $c$. Probabilities are areas under the curve: $P(a \le X \le b) = \int_a^b f(x)\,dx$. Thus $P(a < X < b) = P(a \le X \le b)$.
- **PDF conditions:** $f(x) \ge 0$ everywhere, and $\int_{-\infty}^\infty f(x)\,dx = 1$. Note that $f(x)$ can exceed 1 locally; it is a density, not a probability.
- **CDF and relation to PDF:** $F(x) = P(X \le x) = \int_{-\infty}^x f(t)\,dt$. By the Fundamental Theorem of Calculus, $f(x) = \frac{d}{dx}F(x)$ at points of continuity.
- **Uniform $U(a, b)$:** Density $f(x) = \frac{1}{b-a}$ on $[a, b]$; Mean $\mu = \frac{a+b}{2}$; Variance $\sigma^2 = \frac{(b-a)^2}{12}$. Two independent uniforms form geometric areas in the plane.
- **Exponential $\text{Exp}(\lambda)$:** Density $f(x) = \lambda e^{-\lambda x}$ for $x \ge 0$; CDF $F(x) = 1 - e^{-\lambda x}$; Mean $\mu = \frac{1}{\lambda}$; Variance $\sigma^2 = \frac{1}{\lambda^2}$.
- **Memoryless property:** $P(X > s + t \mid X > s) = P(X > t) = e^{-\lambda t}$. The exponential is the *only* continuous memoryless distribution.
- **Normal $N(\mu, \sigma^2)$:** Symmetric bell curve centered at $\mu$ with inflection points at $\mu \pm \sigma$. Standardise via $Z = \frac{X - \mu}{\sigma} \sim N(0, 1)$. Symmetry gives $\Phi(-z) = 1 - \Phi(z)$.
- **Chi-Square $\chi^2(k)$ and Student's $t(k)$:** Sum of $k$ squared independent standard normals is $\chi^2(k)$ (mean $k$, variance $2k$). The ratio $Z / \sqrt{\chi^2(k)/k}$ is $t(k)$ (symmetric, heavier tails than $N(0, 1)$, approaches $N(0, 1)$ as $k \to \infty$).

## 1. Continuous random variables, PDF, CDF, and joint distributions [PYQ: Medium]

A discrete random variable concentrates probability at countable values. In contrast, a **continuous random variable** takes values in an uncountable continuum. The probability of hitting any exact single number is zero: $P(X = c) = 0$.

### 1.1 Probability density and cumulative distribution

The probability that $X$ falls within a sub-interval is computed as the integral of its **probability density function (PDF)** $f_X(x)$:
$$P(a \le X \le b) = \int_a^b f_X(x)\,dx$$
A valid PDF must satisfy: (1) $f_X(x) \ge 0$ for all $x \in \mathbb{R}$, and (2) $\int_{-\infty}^\infty f_X(x)\,dx = 1$.

The **cumulative distribution function (CDF)** $F_X(x)$ represents accumulated probability from $-\infty$ up to $x$:
$$F_X(x) = P(X \le x) = \int_{-\infty}^x f_X(t)\,dt, \qquad f_X(x) = \frac{d}{dx} F_X(x)$$
Properties: $0 \le F_X(x) \le 1$, $\lim_{x \to -\infty} F_X(x) = 0$, $\lim_{x \to \infty} F_X(x) = 1$, and $P(a < X \le b) = F_X(b) - F_X(a)$.

### 1.2 Expectation and variance
- **Expectation:** $E[X] = \int_{-\infty}^\infty x f_X(x)\,dx$. LOTUS: $E[g(X)] = \int_{-\infty}^\infty g(x) f_X(x)\,dx$.
- **Variance:** $\text{Var}(X) = E[(X - E[X])^2] = E[X^2] - (E[X])^2$. Standard deviation: $\sigma_X = \sqrt{\text{Var}(X)}$.

### 1.3 Joint and conditional continuous distributions
- **Joint PDF condition:** $f_{X,Y}(x, y) \ge 0$ and $\int_{-\infty}^\infty \int_{-\infty}^\infty f_{X,Y}(x, y)\,dx\,dy = 1$.
- **Marginals:** $f_X(x) = \int_{-\infty}^\infty f_{X,Y}(x, y)\,dy$ and $f_Y(y) = \int_{-\infty}^\infty f_{X,Y}(x, y)\,dx$.
- **Independence:** $f_{X,Y}(x, y) = f_X(x) f_Y(y)$ for all $x, y$.
- **Conditional PDF:** $f_{X \mid Y}(x \mid y) = \frac{f_{X,Y}(x, y)}{f_Y(y)}$ (for $f_Y(y) > 0$).
- **Conditional expectation:** $E[X \mid Y = y] = \int_{-\infty}^\infty x f_{X \mid Y}(x \mid y)\,dx$.

**Worked example 1 (Joint continuous density and conditional expectation).**
Let $f_{X,Y}(x, y) = c(x + 2y)$ for $0 \le x \le 1, 0 \le y \le 1$, and 0 otherwise. Find $c$, $f_Y(y)$, and $E[X \mid Y = 0.5]$.
*Step 1 (Normalisation):* $\int_0^1 \int_0^1 c(x + 2y)\,dx\,dy = c \int_0^1 \left[\frac{x^2}{2} + 2yx\right]_0^1 dy = c \int_0^1 \left(\frac{1}{2} + 2y\right) dy = c \left[\frac{y}{2} + y^2\right]_0^1 = \frac{3}{2}c = 1 \implies c = \frac{2}{3}$.
*Step 2 (Marginal $f_Y$):* $f_Y(y) = \int_0^1 \frac{2}{3}(x + 2y)\,dx = \frac{2}{3}\left[\frac{x^2}{2} + 2yx\right]_0^1 = \frac{2}{3}\left(\frac{1}{2} + 2y\right) = \frac{1 + 4y}{3}$ ($0 \le y \le 1$).
*Step 3 (Conditional expectation):* $f_{X \mid Y}(x \mid 0.5) = \frac{\frac{2}{3}(x + 1)}{\frac{1 + 4(0.5)}{3}} = \frac{2(x + 1)}{3} = \frac{2x + 2}{3}$.
$$E[X \mid Y = 0.5] = \int_0^1 x \left(\frac{2x + 2}{3}\right) dx = \frac{1}{3}\left[\frac{2x^3}{3} + x^2\right]_0^1 = \frac{1}{3}\left(\frac{2}{3} + 1\right) = \mathbf{\frac{5}{9}} \approx 0.556$$

---

## 2. Continuous Uniform distribution [PYQ: High]

A continuous random variable $X$ is uniformly distributed over $[a, b]$, written $X \sim U(a, b)$, if density is constant across the interval.

```text
       f(x)
      1/(b-a) |------------------|
              |                  |
        ------+------------------+-------> x
              a                  b
```

### 2.1 PDF, CDF, and moments

| Quantity | Formula | Notes |
|---|---|---|
| **PDF** | $f(x) = \frac{1}{b-a}, \; x \in [a, b]$ | Height is constant, total area $= 1$ |
| **CDF** | $F(x) = \frac{x-a}{b-a}, \; x \in [a, b]$ | Linear ramp from $a$ to $b$ |
| **Mean $E[X]$** | $\mu = \frac{a+b}{2}$ | Center of symmetry |
| **Variance $\text{Var}(X)$** | $\sigma^2 = \frac{(b-a)^2}{12}$ | $\int_a^b \frac{x^2}{b-a}dx - \mu^2 = \frac{(b-a)^2}{12}$ |
| **Median** | $m = \frac{a+b}{2}$ | Equals mean by symmetry |

### 2.2 2D Uniform distribution (Geometric probability)

When two variables $X$ and $Y$ are independent and each uniformly distributed on $[0, T]$, their joint density is constant over the square $[0, T]^2$: $f_{X,Y}(x, y) = \frac{1}{T^2}$. For any event region $E \subseteq [0, T]^2$:
$$P(E) = \frac{\text{Area}(E)}{\text{Total Area}} = \frac{\text{Area}(E)}{T^2}$$

In the **meeting problem**, two arrivals $X, Y \stackrel{\text{iid}}{\sim} U[0, T]$ meet if $|X - Y| \le d$ ($d \le T$). The complement $|X - Y| > d$ corresponds to two corner triangles of area $\frac{1}{2}(T - d)^2$ each, yielding:
$$\mathbf{P(|X - Y| \le d) = 1 - \frac{(T - d)^2}{T^2}}$$

**Worked example 2 (Meeting problem).**
Two students arrive independently at a computer lab between 2:00 PM and 3:00 PM ($T = 60$ min). Each stays for 15 minutes ($d = 15$). Find the probability they meet.
*Solution:* Total sample area $= 60^2 = 3600$. Complement area $= (60 - 15)^2 = 45^2 = 2025$.
$$P(|X - Y| \le 15) = 1 - \frac{45^2}{60^2} = 1 - \left(\frac{3}{4}\right)^2 = 1 - \frac{9}{16} = \mathbf{\frac{7}{16}} = \mathbf{0.4375}$$

---

## 3. Exponential distribution and memorylessness [PYQ: High]

The **exponential distribution** models inter-arrival and waiting times between Poisson events at constant rate $\lambda > 0$.

### 3.1 PDF, CDF, and survival function
- **PDF:** $f(x) = \lambda e^{-\lambda x}$ for $x \ge 0$, and 0 for $x < 0$.
- **CDF:** $F(x) = 1 - e^{-\lambda x}$ for $x \ge 0$.
- **Survival function (Reliability):** $R(x) = P(X > x) = e^{-\lambda x}$.
- **Moments:** Mean $E[X] = \frac{1}{\lambda}$; Variance $\text{Var}(X) = \frac{1}{\lambda^2}$; Standard deviation $\sigma = \frac{1}{\lambda}$.
- **Median:** Solve $1 - e^{-\lambda m} = 0.5 \implies m = \frac{\ln 2}{\lambda} \approx \frac{0.6931}{\lambda}$.

### 3.2 The Memoryless property
A continuous random variable $X$ on $[0, \infty)$ is **memoryless** if:
$$\mathbf{P(X > s + t \mid X > s) = P(X > t) = e^{-\lambda t}} \quad \text{for all } s, t \ge 0$$
*Proof:* $\frac{P(X > s + t)}{P(X > s)} = \frac{e^{-\lambda(s+t)}}{e^{-\lambda s}} = e^{-\lambda t} = P(X > t)$. The component suffers no aging or wear.

### 3.3 Key GATE exponential theorems
1. **Exceeding the mean:** $P(X > E[X]) = P(X > 1/\lambda) = e^{-\lambda(1/\lambda)} = \mathbf{e^{-1} \approx 0.3679}$ (constant for all $\lambda$).
2. **Minimum of independent exponentials:** If $X_i \sim \text{Exp}(\lambda_i)$ are independent, then:
   $$\mathbf{\min(X_1, \dots, X_n) \sim \text{Exp}\left(\sum_{i=1}^n \lambda_i\right)}$$
3. **Race condition:** For independent $X_1 \sim \text{Exp}(\lambda_1)$ and $X_2 \sim \text{Exp}(\lambda_2)$:
   $$\mathbf{P(X_1 < X_2) = \frac{\lambda_1}{\lambda_1 + \lambda_2}}$$
4. **Geometric floor:** The integer part $\lfloor X \rfloor \sim \text{Geometric}(p = 1 - e^{-\lambda})$.

**Worked example 3 (Multi-worker reliability).**
Worker 1 completes jobs with $T_1 \sim \text{Exp}(\lambda_1 = 0.2\text{ min}^{-1})$ and Worker 2 with $T_2 \sim \text{Exp}(\lambda_2 = 0.3\text{ min}^{-1})$ independently.
1. Expected time until the first worker completes: $E[\min(T_1, T_2)] = \frac{1}{0.2 + 0.3} = \frac{1}{0.5} = \mathbf{2.0\text{ min}}$.
2. Probability Worker 1 completes first: $P(T_1 < T_2) = \frac{\lambda_1}{\lambda_1 + \lambda_2} = \frac{0.2}{0.5} = \mathbf{0.40}$.
3. Given Worker 1 has run for 5 min, probability it runs for at least 5 more: $P(T_1 > 10 \mid T_1 > 5) = P(T_1 > 5) = e^{-0.2(5)} = \mathbf{e^{-1} \approx 0.3679}$.

---

## 4. Normal (Gaussian) and Standard Normal distribution [PYQ: Medium]

The **Normal distribution** $N(\mu, \sigma^2)$ has bell-shaped PDF:
$$f(x) = \frac{1}{\sigma \sqrt{2\pi}} \exp\left(-\frac{(x - \mu)^2}{2\sigma^2}\right), \quad x \in \mathbb{R}$$

### 4.1 Key properties and standardisation
1. Symmetric about $\mu$; $\text{Mean} = \text{Median} = \text{Mode} = \mu$. Peak height is $\frac{1}{\sigma\sqrt{2\pi}} \approx \frac{0.3989}{\sigma}$.
2. Inflection points occur at $x = \mu \pm \sigma$.
3. **Standardisation:** Transform $X \sim N(\mu, \sigma^2)$ to standard normal $Z \sim N(0, 1)$ via:
   $$\mathbf{Z = \frac{X - \mu}{\sigma}}, \qquad \Phi(-z) = 1 - \Phi(z)$$
4. **Empirical rule values:**
   - $[\mu \pm \sigma]$: $P(|Z| \le 1) \approx \mathbf{68.27\%}$
   - $[\mu \pm 1.96\sigma]$: $P(|Z| \le 1.96) = \mathbf{95.00\%}$
   - $[\mu \pm 2\sigma]$: $P(|Z| \le 2) \approx \mathbf{95.45\%}$
   - $[\mu \pm 3\sigma]$: $P(|Z| \le 3) \approx \mathbf{99.73\%}$

### 4.2 Linear combination of independent normals
If $X_1 \sim N(\mu_1, \sigma_1^2)$ and $X_2 \sim N(\mu_2, \sigma_2^2)$ are independent:
$$\mathbf{aX_1 + bX_2 \sim N\left(a\mu_1 + b\mu_2, \; a^2\sigma_1^2 + b^2\sigma_2^2\right)}$$
Variances always add: $\text{Var}(X_1 - X_2) = \sigma_1^2 + \sigma_2^2$.

**Worked example 4 (Linear combinations and standardisation).**
$X \sim N(50, 16)$ and $Y \sim N(10, 9)$ are independent. Let $W = 2X - 3Y$.
1. $\mu_W = 2(50) - 3(10) = 70$.
2. $\sigma_W^2 = 2^2(16) + (-3)^2(9) = 64 + 81 = 145 \implies \sigma_W = \sqrt{145} \approx 12.04$.
3. $P(W \le 70) = P\left(Z \le \frac{70 - 70}{\sqrt{145}}\right) = \Phi(0) = \mathbf{0.50}$.

---

## 5. Student's t and Chi-Square distributions [PYQ: Low]

### 5.1 Chi-Square distribution $\chi^2(k)$
If $Z_1, \dots, Z_k \stackrel{\text{iid}}{\sim} N(0, 1)$, then $Q = \sum_{i=1}^k Z_i^2 \sim \chi^2(k)$.
- Support: $[0, \infty)$; Mean $= k$; Variance $= 2k$.
- Additive: $\chi^2(k_1) + \chi^2(k_2) = \chi^2(k_1 + k_2)$ for independent variables.
- Sample variance relation: For a normal sample of size $n$, $\mathbf{\frac{(n-1)S^2}{\sigma^2} \sim \chi^2(n-1)}$.

### 5.2 Student's t-distribution $t(\nu)$
Let $Z \sim N(0, 1)$ and $V \sim \chi^2(\nu)$ be independent. Then:
$$T = \frac{Z}{\sqrt{V / \nu}} \sim t(\nu)$$
- Symmetric about 0 with heavier tails than $N(0, 1)$; Mean $= 0$ ($\nu > 1$); Variance $= \frac{\nu}{\nu - 2}$ ($\nu > 2$).
- As $\nu \to \infty$, $t(\nu) \xrightarrow{d} N(0, 1)$.
- Sample mean statistic with unknown $\sigma$: $\mathbf{\frac{\bar{X} - \mu}{S / \sqrt{n}} \sim t(n - 1)}$.

**Worked example 5 (Sample variance properties).**
A sample of size $n = 16$ is drawn from $N(\mu, \sigma^2 = 25)$. Find $E[S^2]$ and $\text{Var}(S^2)$.
*Solution:* $W = \frac{15 S^2}{25} \sim \chi^2(15) \implies E[W] = 15, \text{Var}(W) = 2(15) = 30$.
- $E[S^2] = \frac{25}{15} E[W] = \mathbf{25} = \sigma^2$ (unbiased).
- $\text{Var}(S^2) = \left(\frac{25}{15}\right)^2 \text{Var}(W) = \frac{25}{9} \times 30 = \frac{250}{3} \approx \mathbf{83.33}$.

---

## Formulas and facts to memorise

| Distribution | PDF / Definition | Support | Mean $E[X]$ | Variance $\text{Var}(X)$ | Key properties |
|---|---|---|---|---|---|
| **Uniform $U(a, b)$** | $f(x) = \frac{1}{b-a}$ | $[a, b]$ | $\frac{a+b}{2}$ | $\frac{(b-a)^2}{12}$ | CDF linear; 2D meeting probability $1 - (T-d)^2/T^2$ |
| **Exponential $\text{Exp}(\lambda)$** | $f(x) = \lambda e^{-\lambda x}$ | $[0, \infty)$ | $\frac{1}{\lambda}$ | $\frac{1}{\lambda^2}$ | Memoryless; $P(X > E[X]) = e^{-1}$; $\min \sim \text{Exp}(\sum \lambda_i)$ |
| **Normal $N(\mu, \sigma^2)$** | $\frac{1}{\sigma\sqrt{2\pi}}e^{-(x-\mu)^2/(2\sigma^2)}$ | $(-\infty, \infty)$ | $\mu$ | $\sigma^2$ | Symmetric; $Z = \frac{X-\mu}{\sigma}$; $\Phi(-z) = 1-\Phi(z)$; 68-95-99.7 rule |
| **Chi-Square $\chi^2(k)$** | $\sum_{i=1}^k Z_i^2, \; Z_i \stackrel{\text{iid}}{\sim} N(0, 1)$ | $[0, \infty)$ | $k$ | $2k$ | Additive; $(n-1)S^2/\sigma^2 \sim \chi^2(n-1)$ |
| **Student's $t(\nu)$** | $\frac{Z}{\sqrt{Q/\nu}}, \; Q \sim \chi^2(\nu)$ | $(-\infty, \infty)$ | 0 ($\nu > 1$) | $\frac{\nu}{\nu-2}$ ($\nu > 2$) | Symmetric, heavier tails than $Z$; converges to $N(0, 1)$ as $\nu \to \infty$ |

---

## GATE traps

- **Rate vs. Scale parameter:** In GATE, check whether exponential distribution uses **rate** $\lambda$ ($f(x) = \lambda e^{-\lambda x}$, mean $1/\lambda$) or **scale** $\beta$ ($f(x) = \frac{1}{\beta}e^{-x/\beta}$, mean $\beta$).
- **PDF values can exceed 1:** A density $f(x)$ is not a probability; $f(x) > 1$ is valid as long as total area $\int f(x)\,dx = 1$.
- **Strict vs. Non-strict inequalities:** For any continuous random variable, $P(X < a) = P(X \le a)$ because $P(X = a) = 0$.
- **Variance addition:** For independent variables, $\text{Var}(X - Y) = \text{Var}(X) + \text{Var}(Y)$, **never** $\text{Var}(X) - \text{Var}(Y)$. Standard deviations never add directly: $\sigma_{X+Y} = \sqrt{\sigma_X^2 + \sigma_Y^2} \ne \sigma_X + \sigma_Y$.
- **Memoryless misapplication:** Memorylessness applies strictly to *remaining* time: $P(X > s + t \mid X > s) = P(X > t)$. Do not write $P(X < s + t \mid X > s) = P(X < t)$; the correct complement is $1 - P(X > t) = 1 - e^{-\lambda t}$.
- **Degrees of freedom in $S^2$:** Dividing by $n-1$ yields $(n-1)S^2/\sigma^2 \sim \chi^2(n-1)$, with $n-1$ degrees of freedom, not $n$.

---

## Connections

- [Random variables and moments](random-variables-and-moments.md) — foundational expectations, LOTUS, and moment generating functions.
- [Statistical inference](statistical-inference.md) — normal, Student's $t$, and Chi-square form the basis of confidence intervals and hypothesis testing.
- [Integration](../04-calculus-optimization/integration.md) — continuous probabilities and expectations are evaluated via definite and improper integrals.
- [Machine learning foundations](../16-machine-learning/ml-foundations.md) — Gaussian Naive Bayes and PCA rely directly on Gaussian density mathematics.

---

## Practice

**Q1 (NAT).** A continuous random variable $X$ is uniformly distributed over $[2, 10]$. The conditional probability $P(X > 4 \mid X < 8)$ is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 0.667 (Acceptable range: 0.665 to 0.669, exact fraction: $2/3$)

**Solution:**
$$P(X > 4 \mid X < 8) = \frac{P(4 < X < 8)}{P(2 \le X < 8)} = \frac{\frac{8 - 4}{10 - 2}}{\frac{8 - 2}{10 - 2}} = \frac{4/8}{6/8} = \frac{4}{6} = \frac{2}{3} \approx \mathbf{0.667}$$

</details>

**Q2 (MCQ).** The lifetime of a network switch is exponentially distributed with failure rate $\lambda = 0.2\text{ year}^{-1}$. If a switch has operated without failure for 5 years, the probability it operates for at least another 10 years is:
(A) $e^{-1}$
(B) $e^{-2}$
(C) $e^{-3}$
(D) $1 - e^{-2}$

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
By the memoryless property of the exponential distribution:
$$P(T > 5 + 10 \mid T > 5) = P(T > 10) = e^{-\lambda \times 10} = e^{-0.2 \times 10} = \mathbf{e^{-2}}$$
Option **(B)** is correct.

</details>

**Q3 (NAT).** Let $f(x, y) = c x y$ for $0 < x < 1, 0 < y < 1$, and 0 otherwise. The probability $P(X + Y < 1)$ is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 0.167 (Acceptable range: 0.165 to 0.169, exact fraction: $1/6$)

**Solution:**
1. $\int_0^1 \int_0^1 c x y\,dx\,dy = c (1/2)(1/2) = c/4 = 1 \implies c = 4$.
2. $P(X + Y < 1) = \int_0^1 4x \left[\int_0^{1-x} y\,dy\right] dx = \int_0^1 2x(1-x)^2\,dx = 2\int_0^1 (x - 2x^2 + x^3)\,dx = 2\left(\frac{1}{2} - \frac{2}{3} + \frac{1}{4}\right) = 2\left(\frac{1}{12}\right) = \mathbf{\frac{1}{6}} \approx \mathbf{0.167}$.

</details>

**Q4 (MSQ).** Let $X_1 \sim N(50, 25)$ and $X_2 \sim N(30, 16)$ be independent. Define $W = X_1 - X_2$. Which statement(s) is/are TRUE?
(A) The expected value $E[W] = 20$.
(B) The standard deviation of $W$ is 9.
(C) $P(W > 20) = 0.5$.
(D) The variance $\text{Var}(W) = 41$.

<details><summary>Answer</summary>

**Answer:** (A), (C), (D)

**Solution:**
- (A) True: $E[W] = 50 - 30 = 20$.
- (D) True: $\text{Var}(W) = 25 + (-1)^2(16) = 41$.
- (B) False: Standard deviation is $\sqrt{41} \approx 6.403 \ne 9$.
- (C) True: $W \sim N(20, 41)$ is symmetric about 20, so $P(W > 20) = 0.5$.

</details>

**Q5 (NAT).** Two parallel independent workers execute tasks with times $T_1 \sim \text{Exp}(0.1\text{ min}^{-1})$ and $T_2 \sim \text{Exp}(0.15\text{ min}^{-1})$. The probability the first task finishes within 4 minutes is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 0.632 (Acceptable range: 0.630 to 0.635)

**Solution:**
$M = \min(T_1, T_2) \sim \text{Exp}(0.1 + 0.15) = \text{Exp}(0.25)$.
$$P(M \le 4) = 1 - e^{-0.25 \times 4} = 1 - e^{-1} \approx 1 - 0.367879 = \mathbf{0.632}$$

</details>

**Q6 (MCQ).** If $Z \sim N(0, 1)$, what is the median of $Y = \max(Z, 0)$?
(A) $-0.5$
(B) $0$
(C) $0.5$
(D) $1.0$

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
$P(Z \le 0) = 0.5$. When $Z \le 0$, $Y = \max(Z, 0) = 0$. Thus $P(Y \le 0) = 0.5$, meaning the median is **0**. Option **(B)** is correct.

</details>

**Q7 (NAT).** Let $Q \sim \chi^2(12)$. The value of $E[Q^2]$ is _________.

<details><summary>Answer</summary>

**Answer:** 168

**Solution:**
For $\chi^2(k)$, $E[Q] = k = 12$ and $\text{Var}(Q) = 2k = 24$.
$$E[Q^2] = \text{Var}(Q) + (E[Q])^2 = 24 + 12^2 = 24 + 144 = \mathbf{168}$$

</details>
