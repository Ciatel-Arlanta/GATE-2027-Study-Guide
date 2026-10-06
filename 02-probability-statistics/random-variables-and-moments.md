# Random variables and moments

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Random variables; Discrete random variables and PMFs; Continuous random variables and PDFs; Cumulative distribution function; Mean, median and mode; Standard deviation; Covariance; Correlation; Conditional expectation; Conditional variance; Conditional PDF
> **Prerequisites:** [Probability basics](probability-basics.md) · [Integration](../04-calculus-optimization/integration.md) · **Leads to:** [Discrete distributions](discrete-distributions.md) · [Continuous distributions](continuous-distributions.md)

## Quick glance

- A **random variable (RV)** $X$ is a function from outcomes to numbers. **Discrete:** countable values, described by a PMF $p(x)=P(X=x)$. **Continuous:** described by a PDF $f(x)\ge0$ with $\int f=1$; $P(X=x)=0$ for every single $x$.
- **CDF** $F(x)=P(X\le x)$: non-decreasing, right-continuous, $F(-\infty)=0$, $F(\infty)=1$; for continuous $X$, $f=F'$ and $P(a<X\le b)=F(b)-F(a)$.
- **Mean** $E[X]=\sum xp(x)=\int xf(x)dx$; **LOTUS** $E[g(X)]=\sum g(x)p(x)$. **Linearity always holds:** $E[X+Y]=E[X]+E[Y]$, independent or not.
- **Variance** $\mathrm{Var}(X)=E[X^2]-(E[X])^2$; $\mathrm{Var}(aX+b)=a^2\mathrm{Var}(X)$; $\mathrm{Var}(X+Y)=\mathrm{Var}X+\mathrm{Var}Y+2\mathrm{Cov}(X,Y)$.
- **Cov** $=E[XY]-E[X]E[Y]$; **correlation** $\rho=\mathrm{Cov}/(\sigma_X\sigma_Y)\in[-1,1]$. Independent $\Rightarrow$ Cov $=0$, **not conversely**.
- **Indicator trick:** write a count as a sum of 0/1 variables; $E[\text{count}]=\sum P(\text{event}_i)$ — no independence needed.
- **Total expectation:** $E[Y]=E[E[Y\mid X]]$. **Total variance:** $\mathrm{Var}(Y)=E[\mathrm{Var}(Y\mid X)]+\mathrm{Var}(E[Y\mid X])$.
- **#1 trap:** $E[XY]=E[X]E[Y]$ and $\mathrm{Var}(X+Y)=\mathrm{Var}X+\mathrm{Var}Y$ need independence (or zero covariance); linearity of $E$ does not.

## 1. Random variables, PMF and CDF

**Intuition.** An experiment's outcome is often not a number ("HHT"); a random variable attaches a number (number of heads). Formally $X:S\to\mathbb R$.

**Discrete RV.** The **probability mass function** $p(x)=P(X=x)$ satisfies $p(x)\ge0$ and $\sum_xp(x)=1$.

**Worked example 1 — PMF and CDF of the sum of two dice.** $X$ = sum, from the 36 equally likely outcomes:

| $x$ | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| ways | 1 | 2 | 3 | 4 | 5 | 6 | 5 | 4 | 3 | 2 | 1 |
| $p(x)$ | $\frac1{36}$ | $\frac2{36}$ | $\frac3{36}$ | $\frac4{36}$ | $\frac5{36}$ | $\frac6{36}$ | $\frac5{36}$ | $\frac4{36}$ | $\frac3{36}$ | $\frac2{36}$ | $\frac1{36}$ |
| $F(x)$ | $\frac1{36}$ | $\frac3{36}$ | $\frac6{36}$ | $\frac{10}{36}$ | $\frac{15}{36}$ | $\frac{21}{36}$ | $\frac{26}{36}$ | $\frac{30}{36}$ | $\frac{33}{36}$ | $\frac{35}{36}$ | 1 |

Mode $=7$; $F(7)=21/36\ge\frac12$ and $F(6)=15/36<\frac12$, so the median is $7$.

**CDF** $F(x)=P(X\le x)$ — works for both kinds of RV.

| Property | Statement |
|---|---|
| Monotone | $x<y\Rightarrow F(x)\le F(y)$ |
| Limits | $F(-\infty)=0,\ F(+\infty)=1$ |
| Right-continuous | $F(x)=\lim_{h\downarrow0}F(x+h)$ |
| Interval probability | $P(a<X\le b)=F(b)-F(a)$ |
| Discrete jump | $P(X=x)=F(x)-F(x^-)$ (jump height) |
| Continuous | $f(x)=F'(x)$ wherever it exists |

A discrete CDF is a **staircase**; a continuous CDF is a continuous curve; a mixed RV has both.

## 2. Continuous RVs and PDFs

A continuous RV has a **probability density function** $f$ with $f(x)\ge0$, $\int_{-\infty}^{\infty}f=1$ and $P(a\le X\le b)=\int_a^bf(x)\,dx$.

**Density is not probability** ($f(x)$ can exceed 1); $P(X=x)=0$, so $P(a<X<b)=P(a\le X\le b)$ — endpoints do not matter. Compare discrete: $P(X\le 3)\ne P(X<3)$ in general.

**Worked example 2 — normalising constant.** $f(x)=kx(1-x)$ on $[0,1]$, 0 elsewhere.

- $\int_0^1kx(1-x)dx=k(\tfrac12-\tfrac13)=k/6=1\Rightarrow k=6$.
- CDF: $F(x)=6(\tfrac{x^2}2-\tfrac{x^3}3)=3x^2-2x^3$ on $[0,1]$.
- $P(X\le0.5)=3(0.25)-2(0.125)=0.75-0.25=0.5$ (symmetry about $\tfrac12$).
- $E[X]=6\int_0^1x^2(1-x)dx=6(\tfrac13-\tfrac14)=\tfrac12$; $E[X^2]=6\int_0^1x^3(1-x)dx=6(\tfrac14-\tfrac15)=0.3$; $\mathrm{Var}=0.3-0.25=0.05$.

**Worked example 3 — median and mode of $f(x)=2x$ on $[0,1]$.**

- CDF $F(x)=x^2$. Median $m$: $m^2=\tfrac12\Rightarrow m=1/\sqrt2\approx0.7071$.
- Mode: $f$ is increasing, maximum at $x=1$.
- Mean $=\int_0^12x^2dx=\tfrac23$; $E[X^2]=\int_0^12x^3dx=\tfrac12$; $\mathrm{Var}=\tfrac12-\tfrac49=\tfrac1{18}$.
- Order: mean $0.667<$ median $0.707<$ mode $1$ — the typical ordering for a left-skewed density.

## 3. Mean, median, mode and standard deviation

| Measure | Definition | Notes |
|---|---|---|
| **Mean** $\mu=E[X]$ | $\sum xp(x)$ or $\int xf$ | balance point; sensitive to outliers |
| **Median** | $m$ with $F(m)\ge\frac12$ and $P(X\ge m)\ge\frac12$ (continuous: $F(m)=\frac12$) | robust to outliers |
| **Mode** | value maximising PMF / PDF | may be non-unique |
| **Variance** $\sigma^2$ | $E[(X-\mu)^2]=E[X^2]-\mu^2$ | spread |
| **Std. deviation** $\sigma$ | $\sqrt{\mathrm{Var}}$ | same units as $X$ |

**LOTUS (law of the unconscious statistician):** $E[g(X)]=\sum g(x)p(x)$ or $\int g(x)f(x)dx$ — no need to find the distribution of $g(X)$.

**Tail-sum formula:** for $X\in\{0,1,2,\dots\}$, $E[X]=\sum_{k\ge1}P(X\ge k)$; for $X\ge0$ continuous, $E[X]=\int_0^\infty P(X>x)dx$.

**Worked example 4 — dataset statistics.** Data: $2,4,4,4,5,5,7,9$ ($n=8$).

- Mean $=40/8=5$. Sorted already; median $=(4+5)/2=4.5$; mode $=4$ (appears 3 times).
- Squared deviations: $9,1,1,1,0,0,4,16$, sum $=32$.
- **Population** variance $=32/8=4$, $\sigma=2$. **Sample** variance $s^2=32/7=4.571$, $s=2.138$.
- Add 3 to every value: mean $8$, SD still $2$. Multiply by 2: mean $10$, SD $4$ — $\mathrm{Var}(aX+b)=a^2\mathrm{Var}X$.

**Skew rule of thumb:** right-skewed $\Rightarrow$ mean > median > mode; left-skewed reverses it. (Empirical formula for moderately skewed data: mode $\approx3\,$median $-\,2\,$mean.)

**Variance rules**

| Rule | Condition |
|---|---|
| $\mathrm{Var}(c)=0$ | constant |
| $\mathrm{Var}(aX+b)=a^2\mathrm{Var}(X)$ | always |
| $\mathrm{Var}(X+Y)=\mathrm{Var}X+\mathrm{Var}Y+2\,\mathrm{Cov}(X,Y)$ | always |
| $\mathrm{Var}(X\pm Y)=\mathrm{Var}X+\mathrm{Var}Y$ | independent (or uncorrelated) |
| $E[aX+bY]=aE[X]+bE[Y]$ | **always** |
| $E[XY]=E[X]E[Y]$ | independent (sufficient, not necessary) |

## 4. Linearity of expectation and indicator variables

**Idea.** To count something, write the count as $N=\sum_iI_i$ where $I_i=1$ if the $i$-th event occurs, else $0$. Then $E[I_i]=P(\text{event}_i)$ and $E[N]=\sum_iP(\text{event}_i)$ — **valid even when the events are dependent.**

**Worked example 5 — fixed points of a random permutation.** A random permutation of $1..n$; $N$ = number of $i$ with $\pi(i)=i$. $I_i=1$ if $\pi(i)=i$; $P(I_i=1)=\frac1n$. $E[N]=n\cdot\frac1n=\mathbf1$ for every $n$. (Check $n=4$: enumerating all 24 permutations the average is exactly $1$.)

**Worked example 6 — empty bins.** $n$ balls thrown independently into $n$ bins uniformly. $J_j=1$ if bin $j$ is empty: $P=(1-\frac1n)^n$. $E[\text{empty bins}]=n(1-\frac1n)^n$. For $n=5$: $5\times0.8^5=1.6384$; $n=10$: $10\times0.9^{10}=3.4868$; as $n\to\infty$, $\to n/e$.

**Worked example 7 — hashing collisions.** $n$ keys hashed uniformly into $m$ slots. Let $C_{ij}=1$ if keys $i<j$ collide: $P=1/m$. Expected colliding pairs $=\binom n2\frac1m$. For $n=10,m=100$: $45/100=0.45$. (Used in [Hashing](../08-algorithms/hashing.md).)

**Worked example 8 — coupon collector.** To collect all $n$ coupon types, the time is $T=T_1+\dots+T_n$ where $T_k$ (waiting for the $k$-th new type, success prob $\frac{n-k+1}n$) is geometric with mean $\frac n{n-k+1}$. $E[T]=n\left(1+\tfrac12+\dots+\tfrac1n\right)=nH_n$. For $n=6$: $6\times2.45=14.7$ rolls to see all faces of a die.

Other uses: expected number of inversions in a random permutation $=\binom n2/2$; expected comparisons in randomised quicksort ([Searching and sorting](../08-algorithms/searching-and-sorting.md)) is computed with indicators $X_{ij}$ = "$i$ and $j$ compared", $P=\frac2{j-i+1}$.

## 5. Joint, marginal and conditional distributions (discrete)

For two RVs the **joint PMF** is $p(x,y)=P(X=x,Y=y)$, summing to 1. **Marginals:** $p_X(x)=\sum_yp(x,y)$, $p_Y(y)=\sum_xp(x,y)$. **Conditional:** $p(y\mid x)=p(x,y)/p_X(x)$. **Independent** iff $p(x,y)=p_X(x)p_Y(y)$ for *all* $(x,y)$.

**Worked example 9 — a joint table.**

| $X\backslash Y$ | 0 | 1 | 2 | $p_X$ |
|---|---|---|---|---|
| 0 | 0.1 | 0.2 | 0.1 | 0.4 |
| 1 | 0.2 | 0.1 | 0.3 | 0.6 |
| $p_Y$ | 0.3 | 0.3 | 0.4 | 1 |

- $E[X]=0.6$; $E[Y]=0\cdot0.3+1\cdot0.3+2\cdot0.4=1.1$.
- $E[XY]=\sum xyp=1\cdot1\cdot0.1+1\cdot2\cdot0.3=0.7$.
- $\mathrm{Cov}=0.7-0.6\times1.1=0.04$.
- $\mathrm{Var}X=0.6-0.36=0.24$; $E[Y^2]=0.3+1.6=1.9$, $\mathrm{Var}Y=1.9-1.21=0.69$.
- $\rho=0.04/\sqrt{0.24\times0.69}=0.04/0.4069=0.0983$.
- Independent? $p(0,0)=0.1\ne0.4\times0.3=0.12$ → no.
- $\mathrm{Var}(X+Y)=0.24+0.69+2(0.04)=1.01$ (direct enumeration of $X+Y$ confirms: $E[(X+Y)^2]=3.9$, $3.9-1.7^2=1.01$).
- Conditionals: $P(Y=1\mid X=1)=0.1/0.6=1/6$; $E[Y\mid X=1]=(0\cdot0.2+1\cdot0.1+2\cdot0.3)/0.6=0.7/0.6=1.1\overline6$; $E[Y\mid X=0]=(0.2+0.2)/0.4=1$.

## 6. Covariance and correlation

$$\mathrm{Cov}(X,Y)=E[(X-\mu_X)(Y-\mu_Y)]=E[XY]-E[X]E[Y],\qquad \rho_{XY}=\frac{\mathrm{Cov}(X,Y)}{\sigma_X\sigma_Y}$$

| Property | Statement |
|---|---|
| Symmetry | $\mathrm{Cov}(X,Y)=\mathrm{Cov}(Y,X)$; $\mathrm{Cov}(X,X)=\mathrm{Var}X$ |
| Bilinear | $\mathrm{Cov}(aX+b,cY+d)=ac\,\mathrm{Cov}(X,Y)$ |
| Additivity | $\mathrm{Cov}(X+Z,Y)=\mathrm{Cov}(X,Y)+\mathrm{Cov}(Z,Y)$ |
| Bound | $-1\le\rho\le1$; $\rho=\pm1$ iff $Y=aX+b$ ($a\gtrless0$) |
| Scale-invariant | $\rho(aX+b,cY+d)=\mathrm{sign}(ac)\rho(X,Y)$ |
| Independence | independent $\Rightarrow\mathrm{Cov}=0$ (**converse false**) |

Correlation measures only **linear** association.

**Worked example 10 — uncorrelated but dependent.** $X$ uniform on $\{-1,0,1\}$ (each $\frac13$), $Y=X^2$.

- $E[X]=0$; $E[XY]=E[X^3]=\frac13(-1+0+1)=0$; so $\mathrm{Cov}(X,Y)=0-0\cdot E[Y]=0$ and $\rho=0$.
- But $Y$ is a function of $X$: $P(Y=0\mid X=0)=1\ne P(Y=0)=\frac13$ → dependent.

Only for **jointly normal** variables does zero covariance imply independence.

The **covariance matrix** $\Sigma_{ij}=\mathrm{Cov}(X_i,X_j)$ is symmetric positive semi-definite; its eigenvectors are the principal components ([PCA](../16-machine-learning/dimensionality-reduction-pca.md); [eigenvalues](../03-linear-algebra/eigenvalues-and-eigenvectors.md)).

## 7. Conditional expectation and variance

$E[Y\mid X=x]=\sum_yy\,p(y\mid x)$ (or $\int yf(y\mid x)dy$). Writing $E[Y\mid X]$ treats it as a random variable, a function of $X$.

$$E[Y]=E\big[E[Y\mid X]\big]\quad\text{(law of total expectation)}$$

$$\mathrm{Var}(Y)=E\big[\mathrm{Var}(Y\mid X)\big]+\mathrm{Var}\big(E[Y\mid X]\big)\quad\text{(law of total variance)}$$

Read the second as "average within-group variance + variance between group means".

**Worked example 11 — die then coins.** Roll a die to get $N\in\{1..6\}$, then toss $N$ fair coins; $X$ = heads.

- $X\mid N\sim\mathrm{Bin}(N,\frac12)$: $E[X\mid N]=N/2$, $\mathrm{Var}(X\mid N)=N/4$.
- $E[X]=E[N]/2=3.5/2=1.75$.
- $\mathrm{Var}(N)=35/12$. $E[\mathrm{Var}(X\mid N)]=E[N]/4=0.875$. $\mathrm{Var}(E[X\mid N])=\mathrm{Var}(N/2)=\frac{35}{12}\cdot\frac14=\frac{35}{48}$.
- $\mathrm{Var}(X)=\frac78+\frac{35}{48}=\frac{42+35}{48}=\frac{77}{48}=1.604$. (A 10⁶-trial simulation gives 1.605.)

Check with the joint table of Example 9: $E[Y]=P(X=0)E[Y\mid0]+P(X=1)E[Y\mid1]=0.4(1)+0.6(1.1\overline6)=0.4+0.7=1.1$ ✓.

## 8. Joint PDF, marginal PDF and conditional PDF

For continuous $(X,Y)$: joint density $f(x,y)\ge0$, $\iint f=1$. **Marginal:** $f_X(x)=\int f(x,y)dy$. **Conditional PDF:**

$$f_{Y\mid X}(y\mid x)=\frac{f(x,y)}{f_X(x)},\quad f_X(x)>0.$$

Independent iff $f(x,y)=f_X(x)f_Y(y)$ (the support must be a rectangle too!).

**Worked example 12 — $f(x,y)=x+y$ on the unit square.**

- Check: $\int_0^1\int_0^1(x+y)dydx=\int_0^1(x+\tfrac12)dx=\tfrac12+\tfrac12=1$ ✓.
- Marginals: $f_X(x)=x+\tfrac12$, $f_Y(y)=y+\tfrac12$.
- Conditional: $f_{Y\mid X}(y\mid x)=\dfrac{x+y}{x+\frac12}$, $0<y<1$.
- $E[Y\mid X=x]=\dfrac{\int_0^1y(x+y)dy}{x+\frac12}=\dfrac{x/2+1/3}{x+1/2}=\dfrac{3x+2}{3(2x+1)}$. At $x=0.5$: $\frac{3.5}{6}=0.5833$.
- $E[X]=E[Y]=\int_0^1x(x+\tfrac12)dx=\tfrac13+\tfrac14=\tfrac7{12}$ (also $\int E[Y\mid x]f_X\,dx=\int(x/2+1/3)dx=\tfrac7{12}$ ✓).
- $E[XY]=\iint xy(x+y)=\tfrac13$; $\mathrm{Cov}=\tfrac13-\tfrac{49}{144}=-\tfrac1{144}$.
- $E[X^2]=\int_0^1x^2(x+\frac12)dx=\frac14+\frac16=\frac5{12}=\frac{60}{144}$, so $\mathrm{Var}X=\frac{60}{144}-\frac{49}{144}=\frac{11}{144}$.
- $\rho=-\frac1{144}\big/\frac{11}{144}=-\frac1{11}$.
- Not independent: $x+y\ne(x+\frac12)(y+\frac12)$.

**Worked example 13 — triangle support.** $f(x,y)=2$ on $0<y<x<1$.

- $f_X(x)=\int_0^x2\,dy=2x$. $f_{Y\mid X}(y\mid x)=2/2x=1/x$ on $(0,x)$: **uniform on $(0,x)$**. So $E[Y\mid X]=X/2$, $\mathrm{Var}(Y\mid X)=X^2/12$.
- $E[X]=\int2x^2=\frac23$, $E[X^2]=\int2x^3=\frac12$, $\mathrm{Var}X=\frac1{18}$.
- $E[Y]=E[X]/2=\frac13$; $E[Y^2]=E[X^2/3]=\frac16$ (since $E[Y^2\mid X]=\mathrm{Var}+\text{mean}^2=\frac{X^2}{12}+\frac{X^2}4=\frac{X^2}3$); $\mathrm{Var}Y=\frac16-\frac19=\frac1{18}$.
- $E[XY]=E[X\cdot X/2]=\frac14$; $\mathrm{Cov}=\frac14-\frac23\cdot\frac13=\frac1{36}$; $\rho=\frac{1/36}{1/18}=\frac12$. (Simulation of max/min of two uniforms: $\rho\approx0.500$.)
- Dependent: support is a triangle, not a rectangle.

## 9. Transformations of random variables

**Linear:** $Y=aX+b$: $E[Y]=aE[X]+b$, $\mathrm{Var}Y=a^2\mathrm{Var}X$; for continuous, $f_Y(y)=\frac1{|a|}f_X\!\big(\frac{y-b}a\big)$.

**Monotone $g$:** $f_Y(y)=f_X(g^{-1}(y))\left|\dfrac{d\,g^{-1}(y)}{dy}\right|$. Safer general method: find $F_Y(y)=P(g(X)\le y)$ in terms of $F_X$, then differentiate.

**Worked example 14 — $Y=X^2$, $X\sim U(0,1)$.** $F_Y(y)=P(X\le\sqrt y)=\sqrt y$ on $[0,1]$; $f_Y(y)=\frac1{2\sqrt y}$; $E[Y]=\int_0^1y\frac1{2\sqrt y}dy=\frac13=E[X^2]$ (LOTUS agrees).

**Worked example 15 — generating an exponential.** $U\sim U(0,1)$, $Y=-\ln(U)/\lambda$: $F_Y(y)=P(U\ge e^{-\lambda y})=1-e^{-\lambda y}$, i.e. $Y\sim\mathrm{Exp}(\lambda)$ ([Continuous distributions](continuous-distributions.md)).

## Formulas and facts to memorise

| Item | Formula / fact | When to use |
|---|---|---|
| PMF / PDF validity | $\sum p=1$ / $\int f=1$ | find constant $k$ |
| CDF to PDF | $f=F'$; $P(a<X\le b)=F(b)-F(a)$ | continuous probabilities |
| Expectation | $\sum xp$, $\int xf$; LOTUS $E[g(X)]$ | any moment |
| Variance | $E[X^2]-(E[X])^2$ | computation shortcut |
| Affine | $E[aX+b]=aE[X]+b$; $\mathrm{Var}=a^2\mathrm{Var}X$ | rescaling |
| Sum | $\mathrm{Var}(X+Y)=\mathrm{Var}X+\mathrm{Var}Y+2\mathrm{Cov}$ | dependent sums |
| Cov / corr | $E[XY]-E[X]E[Y]$; $\rho=\mathrm{Cov}/\sigma_X\sigma_Y$ | association |
| Indicator | $E[N]=\sum P(\text{event}_i)$ | counting expectations |
| Fixed points | $E=1$ | random permutations |
| Empty bins | $n(1-1/n)^n$ | balls into bins |
| Total expectation | $E[Y]=E[E[Y\mid X]]$ | mixtures |
| Total variance | $E[\mathrm{Var}(Y\mid X)]+\mathrm{Var}(E[Y\mid X])$ | mixtures |
| Conditional PDF | $f(x,y)/f_X(x)$ | joint densities |
| Tail sum | $E[X]=\sum_{k\ge1}P(X\ge k)$ | nonneg. integer $X$ |

## GATE traps

- **Zero covariance ≠ independence** ($Y=X^2$ example). Independence ⇒ zero covariance only.
- **$\mathrm{Var}(X-Y)=\mathrm{Var}X+\mathrm{Var}Y$** (independent): variances add, never subtract.
- **$\mathrm{Var}(aX)=a^2\mathrm{Var}X$**, not $a\,\mathrm{Var}X$; **$\mathrm{Var}(X+b)$ ignores $b$**.
- **Sample vs population variance:** check whether the question says "population" (divide by $n$) or "sample" (divide by $n-1$).
- **$E[1/X]\ne1/E[X]$** and $E[X^2]\ne(E[X])^2$ — the gap is the variance. $E[g(X)]\ne g(E[X])$ unless $g$ is linear.
- **Median of a discrete RV / even-sized dataset** is the average of the two middle values for data, but defined via the CDF for distributions.
- **Density can exceed 1**; only the integral is a probability. A PDF with $\int f\ne1$ is invalid — check the constant.
- **Joint density independence needs a rectangular support** as well as factorisation.
- **Correlation is unit-free and symmetric**; $\rho=0$ for $Y=X^2$ with symmetric $X$ even though perfectly dependent.
- **Law of total variance has two terms** — forgetting $\mathrm{Var}(E[Y\mid X])$ is a classic wrong-option.

## Connections

- [Probability basics](probability-basics.md) — an indicator variable is the event $A$ turned into a number; conditional PMF/PDF are Bayes-style conditionals.
- [Discrete distributions](discrete-distributions.md) / [Continuous distributions](continuous-distributions.md) — named PMFs/PDFs whose means and variances come from the integrals and sums here.
- [Statistical inference](statistical-inference.md) — sample mean/variance are RVs; their moments feed the CLT and tests.
- [Integration](../04-calculus-optimization/integration.md) — every PDF probability, mean and variance is a definite integral; double integrals for joint PDFs.
- [Eigenvalues and eigenvectors](../03-linear-algebra/eigenvalues-and-eigenvectors.md) and [PCA](../16-machine-learning/dimensionality-reduction-pca.md) — the covariance matrix and its eigen-decomposition.
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) — $E[Y\mid X]$ is exactly what regression estimates; $R^2=\rho^2$ for simple regression.
- [Hashing](../08-algorithms/hashing.md) and [Searching and sorting](../08-algorithms/searching-and-sorting.md) — expected probes and randomised quicksort via indicator linearity.
- [Probabilistic reasoning (AI)](../17-artificial-intelligence/probabilistic-reasoning.md) — joint/marginal/conditional tables and marginalisation (variable elimination).

## Practice

**Q1 (NAT).** $P(X=1)=0.2$, $P(X=2)=0.3$, $P(X=3)=0.5$. Find $\mathrm{Var}(X)$.

<details><summary>Answer</summary>

**Answer:** 0.61  
**Solution:** $E[X]=0.2+0.6+1.5=2.3$; $E[X^2]=0.2+1.2+4.5=5.9$; $\mathrm{Var}=5.9-5.29=0.61$.

</details>

**Q2 (NAT).** $\mathrm{Var}(X)=4$, $\mathrm{Var}(Y)=9$, $\mathrm{Cov}(X,Y)=-1$. Find $\mathrm{Var}(3X-2Y+5)$.

<details><summary>Answer</summary>

**Answer:** 84  
**Solution:** $9\cdot4+4\cdot9+2(3)(-2)(-1)=36+36+12=84$. The constant 5 does not matter.

</details>

**Q3 (MSQ).** Select all TRUE statements.
(A) If $\mathrm{Cov}(X,Y)=0$ then $X,Y$ are independent.
(B) If $X,Y$ are independent then $E[XY]=E[X]E[Y]$.
(C) $E[X+Y]=E[X]+E[Y]$ for any $X,Y$ with finite means.
(D) $\rho(X,Y)=\rho(2X+1,\,-3Y)$.

<details><summary>Answer</summary>

**Answer:** (B), (C)  
**Solution:** (A) false ($Y=X^2$, $X$ symmetric). (B) true. (C) true by linearity. (D) false: $\rho(aX+b,cY+d)=\mathrm{sign}(ac)\rho$; $a=2,c=-3$ flips the sign, so $\rho(2X+1,-3Y)=-\rho(X,Y)$ (equal only if $\rho=0$).

</details>

**Q4 (NAT).** A random permutation of $\{1,\dots,10\}$ is generated. Expected number of fixed points?

<details><summary>Answer</summary>

**Answer:** 1  
**Solution:** $\sum_{i=1}^{10}P(\pi(i)=i)=10\cdot\frac1{10}=1$.

</details>

**Q5 (NAT).** Five balls are thrown independently and uniformly into five bins. Expected number of empty bins (4 decimals).

<details><summary>Answer</summary>

**Answer:** 1.6384  
**Solution:** Each bin is empty with prob $(4/5)^5=0.32768$; times 5 bins $=1.6384$.

</details>

**Q6 (NAT).** $f(x)=kx(1-x)$ on $[0,1]$. Find $k$ and $\mathrm{Var}(X)$; give $k\times$ (Var $\times100$).

<details><summary>Answer</summary>

**Answer:** 30  
**Solution:** $k=6$; $E[X]=\frac12$, $E[X^2]=0.3$, $\mathrm{Var}=0.05$. $6\times5=30$.

</details>

**Q7 (NAT).** A die is rolled to get $N$; then $N$ fair coins are tossed; $X$ = number of heads. Find $\mathrm{Var}(X)$ (3 decimals).

<details><summary>Answer</summary>

**Answer:** 1.604  
**Solution:** Total variance: $E[N/4]+\mathrm{Var}(N/2)=\frac{3.5}{4}+\frac{35/12}{4}=0.875+0.7292=1.6042=\frac{77}{48}$.

</details>

**Q8 (NAT).** $(X,Y)$ has joint PDF $2$ on $0<y<x<1$. Find $\rho(X,Y)$.

<details><summary>Answer</summary>

**Answer:** 0.5  
**Solution:** $f_X=2x$, $Y\mid X\sim U(0,X)$. $E[X]=\frac23$, $\mathrm{Var}X=\frac1{18}$; $E[Y]=\frac13$, $\mathrm{Var}Y=\frac1{18}$; $E[XY]=E[X^2]/2=\frac14$; $\mathrm{Cov}=\frac14-\frac29=\frac1{36}$; $\rho=\frac{1/36}{1/18}=\frac12$.

</details>

---
[Probability & Statistics README](README.md) · [Cheat sheet](CHEATSHEET.md) · [Checkpoint](CHECKPOINT.md)
