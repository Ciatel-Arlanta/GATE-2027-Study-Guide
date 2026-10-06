# Continuous distributions

> **Paper:** CS+DA · **Priority:** P0/P1 · **Plan topics:** uniform, exponential, normal, standard normal, t, chi-squared distributions
> **Prerequisites:** [Random variables and moments](random-variables-and-moments.md) · **Leads to:** [Statistical inference](statistical-inference.md) · [Probabilistic reasoning](../17-artificial-intelligence/probabilistic-reasoning.md)

## Quick glance
- A PDF is nonnegative and integrates to 1; probability is area, so $P(X=x)=0$ for continuous X.
- Uniform $[a,b]$: mean $(a+b)/2$, variance $(b-a)^2/12$.
- Exponential(rate $\lambda$): $f(x)=\lambda e^{-\lambda x}$ for $x\ge0$, mean $1/\lambda$, memoryless.
- Normal $N(\mu,\sigma^2)$: standardise with $Z=(X-\mu)/\sigma$.
- t distribution is symmetric with heavier tails; degrees of freedom increase toward normal.
- Chi-square with k degrees of freedom is nonnegative, mean k, variance 2k.

## 1. Uniform and exponential
For $X\sim U[2,8]$, density is $1/6$; $P(3\le X\le5)$ is interval width times density: $2/6=1/3$. The mean is 5 and variance is $36/12=3$.

For exponential waiting time with rate $\lambda=0.5$ per minute, $P(X>4)=e^{-0.5\cdot4}=e^{-2}$. Memorylessness means $P(X>s+t\mid X>s)=P(X>t)$: waiting longer so far does not change the residual waiting-time distribution.

## 2. Normal distribution
If $X\sim N(100,15^2)$, then $P(X\le115)=P(Z\le1)\approx0.8413$. A z-table may give lower-tail, upper-tail, or area from zero; identify which convention it uses. Symmetry gives $P(Z<-z)=P(Z>z)$.

## 3. t and chi-square
For normal data with unknown population variance, a sample-mean statistic uses t with $n-1$ degrees of freedom. As degrees of freedom grow, t approaches standard normal. The chi-square distribution arises from sums of squared standard normals and is used for variance and categorical goodness-of-fit procedures.

## GATE traps
- The parameter $\sigma^2$ is variance; $\sigma$ is standard deviation.
- A PDF height can exceed 1; only total area must equal 1.
- For continuous X, $P(X<a)=P(X\le a)$.
- Exponential parameterisation varies: rate $\lambda$ means mean $1/\lambda$.
- Standardising subtracts the mean and divides by standard deviation.

## Connections
- [Random variables and moments](random-variables-and-moments.md) — integrate density to get probabilities and moments.
- [Statistical inference](statistical-inference.md) — normal, t, and chi-square underpin confidence intervals/tests.
- [Probabilistic reasoning](../17-artificial-intelligence/probabilistic-reasoning.md) — distributions encode uncertain evidence.

## Practice
**Q1 (NAT).** $X\sim U[0,10]$. Find $P(2<X<7)$.
<details><summary>Answer</summary> $5/10=0.5$.</details>

**Q2.** $X\sim Exp(2)$ with rate 2. Find $P(X>1)$.
<details><summary>Answer</summary> $e^{-2}$.</details>

**Q3.** $X\sim N(50,4^2)$. Standardise x=58.
<details><summary>Answer</summary> $z=(58-50)/4=2$.</details>
