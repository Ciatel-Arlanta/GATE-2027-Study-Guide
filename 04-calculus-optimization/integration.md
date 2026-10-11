# Integration: accumulation and area

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** Definite and indefinite integrals, integration by parts, properties of definite integrals, Leibniz integral rule, area bounded by curves, improper integrals
> **Prerequisites:** [Limits, continuity, differentiability](limits-continuity-differentiability.md) · **Leads to:** [Continuous distributions](../02-probability-statistics/continuous-distributions.md) · [Maxima, minima and optimization](maxima-minima-optimization.md)
> **PYQ weightage:** Medium · 9 Qs since 2015 (4 since 2021) · DA: 0 · Trend: steady · Most asked: Definite integrals, Properties of definite integrals, Integration by parts

## Quick glance

- **Exam focus:** Definite integrals (applying King's property $\int_a^b f(x)dx = \int_a^b f(a+b-x)dx$ and odd/even symmetry), Leibniz rule for differentiating integrals, and integration by parts (tabular method) are the highest-yield topics in GATE CS and DA. Questions are predominantly NATs requiring exact numerical evaluation.
- **Antiderivative:** $F(x)$ is an antiderivative of $f(x)$ if $F'(x) = f(x)$. The indefinite integral is $\int f(x)\,dx = F(x) + C$.
- **Fundamental Theorem of Calculus (FTC):**
  - **Part 1:** If $g(x) = \int_a^x f(t)\,dt$, then $g'(x) = f(x)$.
  - **Part 2:** $\int_a^b f(x)\,dx = F(b) - F(a) = [F(x)]_a^b$.
- **Leibniz Integral Rule:** $\frac{d}{dx}\left[\int_{u(x)}^{v(x)} f(t)\,dt\right] = f(v(x)) v'(x) - f(u(x)) u'(x)$. Used heavily to evaluate $0/0$ indeterminate forms with L'Hôpital's rule.
- **King's Property:** $\int_a^b f(x)\,dx = \int_a^b f(a+b-x)\,dx$. When the integrand denominator remains invariant under $x \mapsto a+b-x$, adding the two forms gives $2I = \int_a^b 1\,dx = b - a \implies I = \frac{b - a}{2}$.
- **Symmetry:** For symmetric interval $[-a, a]$:
  $$\int_{-a}^a f(x)\,dx = \begin{cases} 0, & f(-x) = -f(x) \text{ (odd function)} \\ 2\int_0^a f(x)\,dx, & f(-x) = f(x) \text{ (even function)} \end{cases}$$
- **Integration by parts:** $\int u\,dv = uv - \int v\,du$. Choose $u$ following **ILATE** (Inverse trig, Log, Algebraic, Trig, Exponential). Use the **tabular D-I method** for polynomial $\times$ exponential/trig.
- **Area bounded by curves:** $\text{Area} = \int_a^b |f(x) - g(x)|\,dx$. Find curve intersections to determine boundaries and which curve lies above the other.
- **Improper integrals ($p$-test):** $\int_1^\infty \frac{1}{x^p}\,dx$ converges iff $p > 1$; $\int_0^1 \frac{1}{x^p}\,dx$ converges iff $p < 1$.

## 1. Antiderivatives and indefinite integration techniques [PYQ: Low]

An **antiderivative** (or primitive) reverses differentiation. If $F'(x) = f(x)$, then $\int f(x)\,dx = F(x) + C$, where $C$ is an arbitrary constant of integration.

### 1.1 Standard elementary antiderivatives

| Function $f(x)$ | Indefinite integral $\int f(x)\,dx$ | Conditions / Notes |
|---|---|---|
| $x^n$ | $\frac{x^{n+1}}{n+1} + C$ | $n \ne -1$ |
| $\frac{1}{x}$ | $\ln\vert x\vert + C$ | $x \ne 0$ |
| $e^{ax}$ | $\frac{1}{a}e^{ax} + C$ | $a \ne 0$ |
| $a^x$ | $\frac{a^x}{\ln a} + C$ | $a > 0, a \ne 1$ |
| $\sin(ax)$ | $-\frac{1}{a}\cos(ax) + C$ | $a \ne 0$ |
| $\cos(ax)$ | $\frac{1}{a}\sin(ax) + C$ | $a \ne 0$ |
| $\sec^2 x$ | $\tan x + C$ | Standard trigonometric primitive |
| $\frac{1}{1 + x^2}$ | $\tan^{-1} x + C$ | Frequently appears in rational integrals |
| $\frac{1}{\sqrt{1 - x^2}}$ | $\sin^{-1} x + C$ | Inverse sine primitive |

### 1.2 Integration by substitution
Substitution reverses the chain rule. If $u = g(x)$, then $du = g'(x)\,dx$:
$$\int f(g(x)) g'(x)\,dx = \int f(u)\,du$$
For a definite integral, limits must be transformed: $\int_a^b f(g(x))g'(x)\,dx = \int_{g(a)}^{g(b)} f(u)\,du$.

**Worked example 1 (Substitution with transformed limits).**
Evaluate the definite integral $I = \int_0^1 \frac{x}{1 + x^2}\,dx$.
*Solution:*
Let $u = 1 + x^2 \implies du = 2x\,dx \implies x\,dx = \frac{du}{2}$.
Transform the limits:
- When $x = 0$: $u = 1 + 0^2 = 1$.
- When $x = 1$: $u = 1 + 1^2 = 2$.
$$I = \int_1^2 \frac{1}{u}\left(\frac{du}{2}\right) = \frac{1}{2} \int_1^2 \frac{du}{u} = \frac{1}{2}[\ln u]_1^2 = \frac{1}{2}(\ln 2 - \ln 1) = \mathbf{\frac{1}{2}\ln 2} \approx \mathbf{0.3466}$$

---

## 2. Definite integrals, Fundamental Theorem, and Leibniz rule [PYQ: High]

The definite integral $\int_a^b f(x)\,dx$ represents the **signed net area** between the curve $y = f(x)$ and the $x$-axis from $x = a$ to $x = b$ (regions above the axis count as positive, below count as negative).

### 2.1 The Fundamental Theorem of Calculus (FTC)
1. **FTC Part 1 (Accumulation function):** If $f$ is continuous on $[a, b]$ and $g(x) = \int_a^x f(t)\,dt$, then $g(x)$ is differentiable and:
   $$\frac{d}{dx}\left[\int_a^x f(t)\,dt\right] = f(x)$$
2. **FTC Part 2 (Evaluation):** If $F$ is any antiderivative of $f$ on $[a, b]$:
   $$\int_a^b f(x)\,dx = F(b) - F(a)$$

### 2.2 Leibniz Integral Rule (Differentiating under the integral)

When the limits of integration are functions of $x$, differentiation proceeds via the chain rule:
$$\mathbf{\frac{d}{dx}\left[\int_{u(x)}^{v(x)} f(t)\,dt\right] = f(v(x)) \cdot v'(x) - f(u(x)) \cdot u'(x)}$$

This theorem is a favourite GATE tool for solving limits of indeterminate forms $0/0$ involving integral expressions.

**Worked example 2 (GATE-style Leibniz rule with L'Hôpital).**
Evaluate the limit:
$$L = \lim_{x \to 0} \frac{\int_0^x \cos(t^2)\,dt}{x}$$

*Solution:*
As $x \to 0$, the numerator is $\int_0^0 \cos(t^2)\,dt = 0$ and the denominator is $0$, giving an indeterminate form of type $\frac{0}{0}$.
Apply L'Hôpital's rule by differentiating numerator and denominator with respect to $x$:
- Denominator derivative: $\frac{d}{dx}(x) = 1$.
- Numerator derivative (by Leibniz rule with $v(x) = x, u(x) = 0$):
  $$\frac{d}{dx}\left[\int_0^x \cos(t^2)\,dt\right] = \cos(x^2) \cdot \frac{d}{dx}(x) - \cos(0^2) \cdot \frac{d}{dx}(0) = \cos(x^2) \cdot 1 - 0 = \cos(x^2)$$
Substitute back into the limit:
$$L = \lim_{x \to 0} \frac{\cos(x^2)}{1} = \frac{\cos(0)}{1} = \mathbf{1}$$

---

## 3. Properties of definite integrals [PYQ: High]

Mastery of definite integral properties avoids long, tedious computations and directly solves GATE questions.

| Property | Formula | When to apply |
|---|---|---|
| **Limit reversal** | $\int_a^b f(x)\,dx = -\int_b^a f(x)\,dx$ | Switching orientation of bounds |
| **Interval split** | $\int_a^b f(x)\,dx = \int_a^c f(x)\,dx + \int_c^b f(x)\,dx$ | Piecewise and absolute-value integrands $\vert x - c\vert$ |
| **King's Property** | $\mathbf{\int_a^b f(x)\,dx = \int_a^b f(a + b - x)\,dx}$ | Integrands with symmetric denominator structure |
| **King's ($a = 0$)** | $\int_0^a f(x)\,dx = \int_0^a f(a - x)\,dx$ | Standard forms $\sin x \leftrightarrow \cos x$ |
| **Odd function** | $\int_{-a}^a f(x)\,dx = 0$ when $f(-x) = -f(x)$ | Symmetric bounds with odd integrand |
| **Even function** | $\int_{-a}^a f(x)\,dx = 2\int_0^a f(x)\,dx$ when $f(-x) = f(x)$ | Symmetric bounds with even integrand |
| **Queen's Property** | $\int_0^{2a} f(x)\,dx = 2\int_0^a f(x)\,dx$ if $f(2a-x)=f(x)$, else $0$ | Halving the upper bound |
| **Periodicity** | $\int_0^{nT} f(x)\,dx = n\int_0^T f(x)\,dx$ for period $T$ | Periodic functions ($\sin, \cos, \{x\}$) |

### 3.1 The King's Property template in GATE
Whenever an integral takes the form:
$$I = \int_a^b \frac{g(x)}{g(x) + g(a+b-x)}\,dx$$
Applying King's property gives:
$$I = \int_a^b \frac{g(a+b-x)}{g(a+b-x) + g(x)}\,dx$$
Adding the two equations yields:
$$2I = \int_a^b \frac{g(x) + g(a+b-x)}{g(x) + g(a+b-x)}\,dx = \int_a^b 1\,dx = b - a \implies \mathbf{I = \frac{b - a}{2}}$$

**Worked example 3 (King's property and absolute-value splitting).**
1. Evaluate $I_1 = \int_0^{\pi/2} \frac{\sin^3 x}{\sin^3 x + \cos^3 x}\,dx$.
2. Evaluate $I_2 = \int_0^3 |x - 1|\,dx$.

*Solution 1:*
Here $a = 0, b = \pi/2$, so $a + b - x = \frac{\pi}{2} - x$.
Since $\sin(\frac{\pi}{2} - x) = \cos x$ and $\cos(\frac{\pi}{2} - x) = \sin x$:
$$I_1 = \int_0^{\pi/2} \frac{\cos^3 x}{\cos^3 x + \sin^3 x}\,dx$$
Adding the two expressions:
$$2I_1 = \int_0^{\pi/2} \frac{\sin^3 x + \cos^3 x}{\sin^3 x + \cos^3 x}\,dx = \int_0^{\pi/2} 1\,dx = \frac{\pi}{2} \implies \mathbf{I_1 = \frac{\pi}{4}}$$

*Solution 2:*
Split the integral at $x = 1$ where the argument of $|x - 1|$ changes sign:
$$I_2 = \int_0^1 |x - 1|\,dx + \int_1^3 |x - 1|\,dx = \int_0^1 (1 - x)\,dx + \int_1^3 (x - 1)\,dx$$
$$= \left[x - \frac{x^2}{2}\right]_0^1 + \left[\frac{x^2}{2} - x\right]_1^3 = \left(1 - \frac{1}{2}\right) + \left[\left(\frac{9}{2} - 3\right) - \left(\frac{1}{2} - 1\right)\right] = \frac{1}{2} + [1.5 - (-0.5)] = 0.5 + 2 = \mathbf{2.5}$$

---

## 4. Integration by parts and tabular method [PYQ: Medium]

Integration by parts reverses the product rule for differentiation:
$$\mathbf{\int u\,dv = uv - \int v\,du}$$
For definite integrals: $\int_a^b u\,dv = [uv]_a^b - \int_a^b v\,du$.

### 4.1 The ILATE rule
Choose $u$ in priority order of **ILATE**:
1. **I**nverse trigonometric ($\sin^{-1} x, \tan^{-1} x$)
2. **L**ogarithmic ($\ln x, \log x$)
3. **A**lgebraic ($x^n, x^2 + 1$)
4. **T**rigonometric ($\sin x, \cos x$)
5. **E**xponential ($e^x, 2^x$)

The function appearing earlier in ILATE is assigned to $u$ (differentiated); the remaining term is assigned to $dv$ (integrated).

### 4.2 Tabular Integration (D-I Method)
When integrating $P(x) \cdot g(x)$ where $P(x)$ is a polynomial and $g(x)$ is easily integrated repeatedly ($e^{ax}, \sin ax, \cos ax$):
1. **Column D:** Write $P(x)$ and repeatedly differentiate until reaching 0.
2. **Column I:** Write $g(x)$ and repeatedly integrate the same number of times.
3. **Column Sign:** Alternate signs $+ , -, +, -, \dots$.
4. Multiply terms diagonally: $\text{Term}_k = (\text{Sign}_k) \cdot D_k \cdot I_{k+1}$.

```text
 Sign      D (differentiate)         I (integrate)
  +            x^2         \            e^(2x)
                            \
  -            2x            \          e^(2x) / 2
                              \
  +             2               \       e^(2x) / 4
                                 \
  -             0                       e^(2x) / 8
```

**Worked example 4 (Tabular integration).**
Evaluate $I = \int_0^1 x^2 e^{2x}\,dx$.

*Solution:*
Construct the D-I table:
- Row 0: $(+) \; x^2 \quad e^{2x}$
- Row 1: $(-) \; 2x \quad \frac{1}{2}e^{2x}$
- Row 2: $(+) \; 2 \quad \frac{1}{4}e^{2x}$
- Row 3: $(-) \; 0 \quad \frac{1}{8}e^{2x}$

Read diagonally:
$$\int x^2 e^{2x}\,dx = x^2\left(\frac{e^{2x}}{2}\right) - 2x\left(\frac{e^{2x}}{4}\right) + 2\left(\frac{e^{2x}}{8}\right) + C = e^{2x}\left(\frac{x^2}{2} - \frac{x}{2} + \frac{1}{4}\right) + C$$
Now evaluate from $0$ to $1$:
- At $x = 1$: $e^{2(1)}\left(\frac{1}{2} - \frac{1}{2} + \frac{1}{4}\right) = \frac{e^2}{4}$.
- At $x = 0$: $e^0\left(0 - 0 + \frac{1}{4}\right) = \frac{1}{4}$.
$$I = \frac{e^2}{4} - \frac{1}{4} = \mathbf{\frac{e^2 - 1}{4}} \approx \frac{7.3891 - 1}{4} = \mathbf{1.597}$$

---

## 5. Area bounded by curves and improper integrals [PYQ: Medium]

### 5.1 Area enclosed between curves
To find the area bounded between curves $y = f(x)$ and $y = g(x)$ from $x = a$ to $x = b$:
$$\mathbf{\text{Area} = \int_a^b |f(x) - g(x)|\,dx = \int_a^b [y_{\text{top}}(x) - y_{\text{bottom}}(x)]\,dx}$$
Algorithm:
1. Solve $f(x) = g(x)$ to locate all intersection points.
2. Determine which function lies above the other on each sub-interval.
3. Integrate $y_{\text{top}} - y_{\text{bottom}}$ separately over each sub-interval.
*(Alternative along the $y$-axis: $\text{Area} = \int_c^d [x_{\text{right}}(y) - x_{\text{left}}(y)]\,dy$).*

### 5.2 Improper integrals and convergence tests
An integral is **improper** if either:
- The interval is unbounded: $\int_a^\infty f(x)\,dx = \lim_{t \to \infty} \int_a^t f(x)\,dx$ (Type 1).
- The integrand has an infinite discontinuity on $[a, b]$: $\lim_{x \to c} f(x) = \pm\infty$ (Type 2).

The integral **converges** if the limit exists as a finite real number; otherwise it **diverges**.

#### The $p$-integral test (Must memorise)
1. **Unbounded upper limit $[1, \infty)$:**
   $$\int_1^\infty \frac{1}{x^p}\,dx \quad \begin{cases} \mathbf{\text{Converges}}, & \mathbf{p > 1} \quad \left(\text{value } = \frac{1}{p - 1}\right) \\ \mathbf{\text{Diverges}}, & \mathbf{p \le 1} \end{cases}$$
2. **Singularity at zero $(0, 1]$:**
   $$\int_0^1 \frac{1}{x^p}\,dx \quad \begin{cases} \mathbf{\text{Converges}}, & \mathbf{p < 1} \quad \left(\text{value } = \frac{1}{1 - p}\right) \\ \mathbf{\text{Diverges}}, & \mathbf{p \ge 1} \end{cases}$$

### 5.3 Euler's Gamma function
Defined for $z > 0$:
$$\mathbf{\Gamma(z) = \int_0^\infty t^{z-1} e^{-t}\,dt}$$
- For positive integer $n$: $\Gamma(n) = (n - 1)!$.
- Recurrence relation: $\Gamma(z + 1) = z\Gamma(z)$.
- Special value: $\Gamma(1/2) = \sqrt{\pi}$.

**Worked example 5 (Area between curves).**
Find the area of the region enclosed between the parabola $y = x^2$ and the line $y = 2x$.
*Solution:*
1. Find intersection points: $x^2 = 2x \implies x^2 - 2x = 0 \implies x(x - 2) = 0 \implies x = 0$ and $x = 2$.
2. For $x \in [0, 2]$, test midpoint $x = 1$: line $y = 2(1) = 2$ lies above parabola $y = 1^2 = 1$. Thus $y_{\text{top}} = 2x$ and $y_{\text{bottom}} = x^2$.
3. Compute the area:
   $$\text{Area} = \int_0^2 (2x - x^2)\,dx = \left[x^2 - \frac{x^3}{3}\right]_0^2 = \left(2^2 - \frac{2^3}{3}\right) - 0 = 4 - \frac{8}{3} = \mathbf{\frac{4}{3}} \approx \mathbf{1.333}$$

---

## Formulas and facts to memorise

| Concept | Formula | Key condition |
|---|---|---|
| **Leibniz Rule** | $\frac{d}{dx}\int_{u(x)}^{v(x)} f(t)dt = f(v(x))v'(x) - f(u(x))u'(x)$ | Continuous $f$, differentiable $u, v$ |
| **King's Property** | $\int_a^b f(x)dx = \int_a^b f(a+b-x)dx$ | Any integrable $f$ |
| **King's Template** | $\int_a^b \frac{g(x)}{g(x) + g(a+b-x)}dx = \frac{b - a}{2}$ | Direct shortcut |
| **Odd/Even Symmetry** | $\int_{-a}^a f(x)dx = 0$ (odd), $2\int_0^a f(x)dx$ (even) | Symmetric limits $[-a, a]$ |
| **Integration by Parts** | $\int u\,dv = uv - \int v\,du$ | Priority: **ILATE** |
| **Area between curves** | $\int_a^b [y_{\text{top}} - y_{\text{bottom}}]dx$ | Boundary limits from curve intersections |
| **$p$-test ($[1, \infty)$)** | $\int_1^\infty \frac{1}{x^p}dx$ converges $\iff p > 1$ | Diverges for $p \le 1$ |
| **$p$-test ($(0, 1]$)** | $\int_0^1 \frac{1}{x^p}dx$ converges $\iff p < 1$ | Diverges for $p \ge 1$ |
| **Gamma function** | $\int_0^\infty t^{n-1}e^{-t}dt = (n-1)!$ | $\Gamma(1/2) = \sqrt{\pi}$ |

---

## GATE traps

- **Forgetting $v'(x)$ in Leibniz rule:** When differentiating $\int_0^{x^2} \sin(t)\,dt$, the answer is $\sin(x^2) \cdot (2x)$, **not** just $\sin(x^2)$. Always multiply by the derivative of the boundary function.
- **Signs in signed area vs. geometric area:** An integral $\int_a^b f(x)\,dx$ computes signed area (can be negative). A question asking for "geometric area" or "area enclosed" requires $\int_a^b |f(x)|\,dx$, which is strictly non-negative.
- **$p$-test boundary confusion:** Do not mix up the convergence criteria! For infinite interval $[1, \infty)$, $p > 1$ is needed for convergence. For singular interval $(0, 1]$, $p < 1$ is needed. At $p = 1$, both $\int_1^\infty \frac{1}{x}dx$ and $\int_0^1 \frac{1}{x}dx$ diverge logarithmically.
- **ILATE priority violations:** In $\int x e^x dx$, choosing $u = e^x$ and $dv = x dx$ creates $\frac{x^2}{2}e^x$, making the remaining integral harder. Follow ILATE strictly ($x$ is Algebraic, $e^x$ is Exponential, so $u = x$).
- **Discontinuities inside limits of integration:** When computing $\int_{-1}^1 \frac{1}{x^2}\,dx$, writing $[-1/x]_{-1}^1 = -1 - 1 = -2$ is an **epic mistake**. The integrand has an infinite discontinuity at $x = 0$; the improper integral diverges to $+\infty$.

---

## Connections

- [Continuous distributions](../02-probability-statistics/continuous-distributions.md) — CDF, expectations, and normalising constants are computed via definite and improper integrals.
- [Limits, continuity, differentiability](limits-continuity-differentiability.md) — Fundamental Theorem connects derivative to integral; Leibniz rule resolves L'Hôpital limits.
- [Maxima, minima and optimization](maxima-minima-optimization.md) — critical points determine highest/lowest curves and optimal bounded areas.

---

## Practice

**Q1 (NAT).** The value of the definite integral $\int_0^2 (3x^2 - 4x + 1)\,dx$ is _________.

<details><summary>Answer</summary>

**Answer:** 2

**Solution:**
Find the antiderivative:
$$\int (3x^2 - 4x + 1)\,dx = x^3 - 2x^2 + x$$
Evaluate at limits 0 and 2:
$$\left[x^3 - 2x^2 + x\right]_0^2 = (2^3 - 2(2^2) + 2) - 0 = (8 - 8 + 2) - 0 = \mathbf{2}$$

</details>

**Q2 (NAT).** The value of the definite integral $I = \int_0^{\pi/2} \frac{\sqrt{\sin x}}{\sqrt{\sin x} + \sqrt{\cos x}}\,dx$ is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 0.785 (Acceptable range: 0.784 to 0.786, exact value: $\pi/4$)

**Solution:**
Apply King's property: $\int_0^a f(x)\,dx = \int_0^a f(a - x)\,dx$ with $a = \pi/2$:
$$I = \int_0^{\pi/2} \frac{\sqrt{\sin(\frac{\pi}{2} - x)}}{\sqrt{\sin(\frac{\pi}{2} - x)} + \sqrt{\cos(\frac{\pi}{2} - x)}}\,dx = \int_0^{\pi/2} \frac{\sqrt{\cos x}}{\sqrt{\cos x} + \sqrt{\sin x}}\,dx$$
Adding the two equations:
$$2I = \int_0^{\pi/2} \frac{\sqrt{\sin x} + \sqrt{\cos x}}{\sqrt{\sin x} + \sqrt{\cos x}}\,dx = \int_0^{\pi/2} 1\,dx = \frac{\pi}{2} - 0 = \frac{\pi}{2}$$
$$I = \frac{\pi}{4} \approx \frac{3.14159}{4} \approx \mathbf{0.785}$$

</details>

**Q3 (MCQ).** The value of the definite integral $I = \int_{-1}^1 \frac{x^3 + x\cos x}{1 + x^4}\,dx$ is:
(A) $-1$
(B) $0$
(C) $1$
(D) $\pi/2$

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
Check the symmetry of the integrand $f(x) = \frac{x^3 + x\cos x}{1 + x^4}$:
$$f(-x) = \frac{(-x)^3 + (-x)\cos(-x)}{1 + (-x)^4} = \frac{-x^3 - x\cos x}{1 + x^4} = -\frac{x^3 + x\cos x}{1 + x^4} = -f(x)$$
Since $f(x)$ is an **odd function** and the integration limits are symmetric $[-1, 1]$, by Property 4:
$$\int_{-1}^1 f(x)\,dx = \mathbf{0}$$
Option **(B)** is correct.

</details>

**Q4 (NAT).** The value of the definite integral $\int_1^e x \ln x\,dx$ is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 2.097 (Acceptable range: 2.095 to 2.100, exact value: $\frac{e^2 + 1}{4}$)

**Solution:**
Use integration by parts with ILATE:
- Let $u = \ln x \implies du = \frac{1}{x}\,dx$
- Let $dv = x\,dx \implies v = \frac{x^2}{2}$
$$\int x \ln x\,dx = \frac{x^2}{2}\ln x - \int \frac{x^2}{2}\cdot\frac{1}{x}\,dx = \frac{x^2}{2}\ln x - \frac{1}{2}\int x\,dx = \frac{x^2}{2}\ln x - \frac{x^2}{4}$$
Evaluate between $1$ and $e$:
- At $x = e$: $\frac{e^2}{2}\ln(e) - \frac{e^2}{4} = \frac{e^2}{2} - \frac{e^2}{4} = \frac{e^2}{4}$
- At $x = 1$: $\frac{1^2}{2}\ln(1) - \frac{1^2}{4} = 0 - \frac{1}{4} = -\frac{1}{4}$
$$I = \frac{e^2}{4} - \left(-\frac{1}{4}\right) = \frac{e^2 + 1}{4} \approx \frac{7.389056 + 1}{4} = \frac{8.389056}{4} \approx \mathbf{2.097}$$

</details>

**Q5 (NAT).** Let $F(x) = \int_0^{x^2} \sqrt{1 + t^2}\,dt$. The value of the derivative $F'(1)$ is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 2.828 (Acceptable range: 2.820 to 2.835, exact value: $2\sqrt{2}$)

**Solution:**
Apply the Leibniz integral rule with upper limit $v(x) = x^2$ and lower limit $u(x) = 0$:
$$F'(x) = \sqrt{1 + (x^2)^2} \cdot \frac{d}{dx}(x^2) - \sqrt{1 + 0^2} \cdot \frac{d}{dx}(0) = \sqrt{1 + x^4} \cdot (2x) = 2x\sqrt{1 + x^4}$$
Evaluate at $x = 1$:
$$F'(1) = 2(1)\sqrt{1 + 1^4} = 2\sqrt{2} \approx 2 \times 1.4142 = \mathbf{2.828}$$

</details>

**Q6 (MSQ).** Which of the following improper integrals is/are CONVERGENT?
(A) $\int_1^\infty \frac{1}{x^2}\,dx$
(B) $\int_1^\infty \frac{1}{\sqrt{x}}\,dx$
(C) $\int_0^1 \frac{1}{\sqrt{x}}\,dx$
(D) $\int_0^\infty e^{-2x}\,dx$

<details><summary>Answer</summary>

**Answer:** (A), (C), (D)

**Solution:**
- (A) Convergent: By the $p$-test for $[1, \infty)$, $p = 2 > 1$, so it converges (value $= 1/(2-1) = 1$).
- (B) Divergent: By the $p$-test for $[1, \infty)$, $p = 0.5 \le 1$, so it diverges to $\infty$.
- (C) Convergent: For the singularity at zero on $(0, 1]$, the $p$-test states convergence for $p < 1$. Here $p = 0.5 < 1$, so it converges (value $= [2\sqrt{x}]_0^1 = 2$).
- (D) Convergent: $\int_0^\infty e^{-2x}\,dx = \left[-\frac{1}{2}e^{-2x}\right]_0^\infty = 0 - (-1/2) = 0.5 < \infty$.

</details>

**Q7 (NAT).** The area of the region bounded by the parabola $y = x^2$ and the line $y = 4$ is _________ (rounded off to 3 decimal places).

<details><summary>Answer</summary>

**Answer:** 10.667 (Acceptable range: 10.660 to 10.675, exact fraction: $32/3$)

**Solution:**
1. Intersection points: $x^2 = 4 \implies x = \pm 2$.
2. On $[-2, 2]$, the line $y = 4$ is above the parabola $y = x^2$.
3. By symmetry, the enclosed area is:
   $$\text{Area} = \int_{-2}^2 (4 - x^2)\,dx = 2\int_0^2 (4 - x^2)\,dx$$
   $$= 2\left[4x - \frac{x^3}{3}\right]_0^2 = 2\left(4(2) - \frac{2^3}{3}\right) = 2\left(8 - \frac{8}{3}\right) = 2\left(\frac{16}{3}\right) = \mathbf{\frac{32}{3}} \approx \mathbf{10.667}$$

</details>
