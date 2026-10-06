# Mean Value Theorems and Taylor Series

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** Mean value theorem; Taylor series
> **Prerequisites:** [Limits, continuity, differentiability](limits-continuity-differentiability.md) · **Leads to:** [Maxima, minima, optimisation](maxima-minima-optimization.md) · [Integration](integration.md)

## Quick glance
- **Rolle:** f continuous on [a, b], differentiable on (a, b), f(a) = f(b) ⇒ some c ∈ (a, b) with **f'(c) = 0**.
- **Lagrange MVT:** same hypotheses without f(a) = f(b) ⇒ some c with **f'(c) = (f(b) - f(a))/(b - a)** (a tangent parallel to the chord).
- **Cauchy MVT:** f'(c)/g'(c) = (f(b) - f(a))/(g(b) - g(a)) (g' ≠ 0).
- Consequences: f' = 0 on an interval ⇒ f constant; f' > 0 ⇒ strictly increasing; |f'| ≤ M ⇒ |f(x) - f(y)| ≤ M |x - y|.
- **Taylor:** f(x) = Σ f^(n)(a)(x - a)^n / n!. **Maclaurin** = Taylor at a = 0.
- Standard Maclaurin series: e^x = Σ x^n/n!; sin x = x - x³/3! + x⁵/5! - ...; cos x = 1 - x²/2! + x⁴/4! - ...; 1/(1 - x) = Σ x^n; ln(1 + x) = x - x²/2 + x³/3 - ...
- **Lagrange remainder:** R_n(x) = f^(n+1)(c)(x - a)^(n+1)/(n+1)! for some c between a and x. Use it to bound the error.
- #1 trap: hypotheses matter. MVT fails for |x| on [-1, 1] (not differentiable at 0), so f'(c) = 0 has no solution there.

## 1. Rolle's theorem

**Statement.** If (i) f is continuous on [a, b], (ii) differentiable on (a, b), (iii) f(a) = f(b), then there is c ∈ (a, b) with f'(c) = 0.

**Intuition.** A smooth hill-and-valley path that starts and ends at the same height must have a flat point (top or bottom).

**Worked example 1.** f(x) = x² - 4x + 3 on [1, 3]: f(1) = 0 = f(3). f'(x) = 2x - 4 = 0 at **c = 2** ∈ (1, 3) ✓.

**Worked example 2 — hypothesis fails.** f(x) = |x| on [-1, 1]: f(-1) = f(1) = 1 but there is no flat point, because f is not differentiable at 0. Likewise f(x) = x on [0, 1] fails (iii) and has no flat point.

**Use: counting roots.** If f' has k real roots, f has at most k + 1 real roots (between two roots of f lies a root of f'). Example: f(x) = x³ + x - 1 has f'(x) = 3x² + 1 > 0 (no real roots), so f has **exactly one** real root (IVT: f(0) = -1 < 0, f(1) = 1 > 0). Similarly x³ - 3x + k has at most one root in (-1, 1) (f' = 3x² - 3 ≠ 0 inside).

## 2. Lagrange's mean value theorem

**Statement.** If f is continuous on [a, b] and differentiable on (a, b), there is c ∈ (a, b) with

**f'(c) = (f(b) - f(a))/(b - a).**

Rolle is the special case f(a) = f(b). Interpretation: the instantaneous rate equals the average rate at some instant (a car averaging 60 km/h hit exactly 60 km/h at some moment).

**Worked example 3.** f(x) = x³ on [0, 2]: average slope = (8 - 0)/2 = 4. 3c² = 4 gives c = ±2/√3; only **c = 2/√3 ≈ 1.1547** lies in (0, 2).

**Worked example 4.** f(x) = ln x on [1, e]: slope = (1 - 0)/(e - 1), and f'(c) = 1/c, so **c = e - 1 ≈ 1.718** ∈ (1, e) ✓. f(x) = √x on [1, 4]: slope 1/3 and 1/(2√c) = 1/3 gives √c = 3/2, **c = 9/4**.

**Worked example 5 — an inequality.** Show |sin a - sin b| ≤ |a - b|. By MVT sin a - sin b = cos(c)(a - b) and |cos c| ≤ 1. Another: for 0 < a < b, (b - a)/b < ln(b/a) < (b - a)/a (apply MVT to ln x: ln b - ln a = (b - a)/c with a < c < b).

**Worked example 6 — bounding.** If f(0) = 1 and f'(x) ≤ 2 for all x, then f(3) ≤ f(0) + 2 · 3 = 7 (MVT: f(3) - f(0) = 3 f'(c) ≤ 6).

**Consequences.**
- f' ≡ 0 on an interval ⇒ f is constant. Two functions with the same derivative differ by a constant.
- f' > 0 on an interval ⇒ f strictly increasing; f' < 0 ⇒ strictly decreasing.
- A function with bounded derivative is Lipschitz: |f(x) - f(y)| ≤ M |x - y|.
- Darboux: a derivative has the intermediate value property (no jump discontinuities of f').

## 3. Cauchy's mean value theorem

If f, g are continuous on [a, b], differentiable on (a, b) and g' ≠ 0 on (a, b), then for some c ∈ (a, b)

**f'(c)/g'(c) = (f(b) - f(a))/(g(b) - g(a)).**

Taking g(x) = x recovers Lagrange. It is the tool behind L'Hôpital's rule.

**Worked example 7.** f(x) = x², g(x) = x³ on [1, 2]: RHS = (4 - 1)/(8 - 1) = 3/7. LHS = 2c/(3c²) = 2/(3c). So 2/(3c) = 3/7, **c = 14/9 ≈ 1.556** ∈ (1, 2) ✓.

## 4. Taylor and Maclaurin series

**Intuition.** Near a, replace a complicated function by the polynomial that matches its value, slope, curvature, ... at a. More terms give a better approximation closer to a.

**Taylor polynomial of degree n at a:**

P_n(x) = f(a) + f'(a)(x - a) + f''(a)(x - a)²/2! + ... + f^(n)(a)(x - a)^n / n!.

**Lagrange form of the remainder:** f(x) = P_n(x) + R_n(x), **R_n(x) = f^(n+1)(c)(x - a)^(n+1) / (n + 1)!** for some c between a and x. If the remainder → 0 as n → ∞, the Taylor series converges to f.

- **Linear approximation** (n = 1): f(x) ≈ f(a) + f'(a)(x - a). **Quadratic** (n = 2) adds f''(a)(x - a)²/2.
- Maclaurin: a = 0.

### 4.1 Standard Maclaurin series

| Function | Series | Valid for |
|---|---|---|
| e^x | 1 + x + x²/2! + x³/3! + ... | all x |
| sin x | x - x³/3! + x⁵/5! - ... | all x |
| cos x | 1 - x²/2! + x⁴/4! - ... | all x |
| sinh x, cosh x | x + x³/3! + ..., 1 + x²/2! + ... | all x |
| 1/(1 - x) | 1 + x + x² + x³ + ... | \|x\| < 1 |
| 1/(1 + x) | 1 - x + x² - ... | \|x\| < 1 |
| ln(1 + x) | x - x²/2 + x³/3 - ... | -1 < x ≤ 1 |
| (1 + x)^m | 1 + m x + m(m-1)x²/2! + ... | \|x\| < 1 |
| tan⁻¹ x | x - x³/3 + x⁵/5 - ... | \|x\| ≤ 1 |
| tan x | x + x³/3 + 2x⁵/15 + ... | \|x\| < π/2 |
| sin⁻¹ x | x + x³/6 + 3x⁵/40 + ... | \|x\| ≤ 1 |
| √(1 + x) | 1 + x/2 - x²/8 + x³/16 - ... | \|x\| ≤ 1 |

Series can be added, multiplied, composed and differentiated/integrated term by term inside the radius of convergence.

**Worked example 8 — build new series.** (a) e^(-x²) = 1 - x² + x⁴/2 - x⁶/6 + ... (substitute -x² into e^x). (b) 1/(1 + x²) = 1 - x² + x⁴ - ... (c) e^x sin x = (1 + x + x²/2 + ...)(x - x³/6 + ...) = x + x² + x³(1/2 - 1/6) + ... = **x + x² + x³/3 + ...** (d) e^x cos x = 1 + x - x³/3 - x⁴/6 + ...: the x² terms: 1/2 - 1/2 = 0 ✓.

**Worked example 9 — Taylor at a ≠ 0.** f(x) = x³ at a = 1: f(1) = 1, f' = 3x² → 3, f'' = 6x → 6, f''' = 6. So x³ = 1 + 3(x - 1) + 3(x - 1)² + (x - 1)³ (exact, since the degree is 3). For ln x at a = 1: derivatives 0, 1, -1, 2, -6...: ln x = (x - 1) - (x - 1)²/2 + (x - 1)³/3 - ... .

**Worked example 10 — finding derivatives from the series.** If f(x) = e^(2x) then the coefficient of x³ in f's series is 2³/3! = 4/3, so f'''(0) = 3! · 4/3 = **8** (= 2³). In general **f^(n)(0) = n! × (coefficient of x^n)**.

### 4.2 Approximations and error bounds

**Worked example 11.** Approximate e^0.1 with the quadratic: 1 + 0.1 + 0.005 = 1.105; cubic: + 0.1³/6 = 0.000167 → 1.105167 (true 1.105171). Error of the cubic ≤ e^c (0.1)⁴/24 ≤ 1.2 × 4.2 × 10⁻⁶ ≈ 5 × 10⁻⁶ ✓.

**Worked example 12.** sin 0.5 ≈ 0.5 - 0.5³/6 = 0.479167. The next term is the remainder bound: |R| ≤ 0.5⁵/5! = 0.00026 (|sin^(5)| ≤ 1). The true value 0.479426: error 0.000259 ✓. Since the series alternates with decreasing terms, the error is smaller than the first omitted term.

**Worked example 13 — linearisation.** √25.5 ≈ 5 + (0.5)/(2 · 5) = 5.05 (true 5.04975). Also ln(1.1) ≈ 0.1 - 0.005 + 0.000333 = 0.09533 (true 0.09531).

**Worked example 14 — how many terms?** To approximate e = e¹ with error < 0.01 using P_n(1) = Σ_{k≤n} 1/k!, the remainder is ≤ e/(n + 1)! < 3/(n + 1)!. n = 4: 3/120 = 0.025 > 0.01, not guaranteed; n = 5: 3/720 ≈ 0.0042 < 0.01 ✓. (The actual n = 4 error is 0.00995, just under 0.01, but the bound cannot prove it.)

### 4.3 Taylor series for limits

Replacing functions by series is often faster than repeated L'Hôpital:
- (sin x - x)/x³ = (-x³/6 + ...)/x³ → **-1/6**.
- (1 - cos x - x²/2 + x⁴/24 ...) pattern: (cos x - 1 + x²/2)/x⁴ → **1/24**.
- (e^x - 1 - x - x²/2)/x³ → **1/6**.
- (tan x - sin x)/x³ = ((x + x³/3) - (x - x³/6))/x³ → 1/3 + 1/6 = **1/2**.

**Local extremum via Taylor.** If f'(a) = ... = f^(k-1)(a) = 0 and f^(k)(a) ≠ 0: k even ⇒ extremum (max if f^(k) < 0, min if > 0); k odd ⇒ inflection, no extremum. See [Maxima and minima](maxima-minima-optimization.md).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Rolle | f(a) = f(b) ⇒ f'(c) = 0 | root counting, existence of flat point |
| Lagrange MVT | f'(c) = (f(b) - f(a))/(b - a) | find c, inequalities |
| Cauchy MVT | f'(c)/g'(c) = Δf/Δg | ratio problems |
| Taylor | Σ f^(n)(a)(x - a)^n/n! | approximations |
| Lagrange remainder | f^(n+1)(c)(x - a)^(n+1)/(n + 1)! | error bounds |
| Coefficient link | f^(n)(0) = n! · [x^n] | derivatives from series |
| e^x, sin, cos, ln(1 + x), 1/(1 - x), (1 + x)^m | table above | limits, approximations |
| Linearisation | f(a + h) ≈ f(a) + h f'(a) | quick estimates |
| Bounded derivative | \|f(x) - f(y)\| ≤ M \|x - y\| | Lipschitz |

## GATE traps
- Check **all** hypotheses: continuity on the closed interval, differentiability on the open one. |x| on [-1, 1] and 1/x on [-1, 1] fail.
- The c from MVT must lie in the **open interval**; discard roots outside (e.g. c = -2/√3 for x³ on [0, 2]).
- Rolle requires f(a) = f(b); do not force it on a generic MVT question.
- (1 + x)^m and ln(1 + x) converge only for |x| < 1 (or up to 1); do not expand around a point outside the radius.
- The Maclaurin coefficient of x^n is f^(n)(0)/n!, **not** f^(n)(0).
- sin x is odd (only odd powers), cos x even (only even powers). Parity of the series matches parity of the function.
- Taylor remainder uses f^(n+1) at an unknown c; bound it by the maximum of |f^(n+1)| on the interval.

## Connections
- [Limits and L'Hôpital](limits-continuity-differentiability.md) — Cauchy MVT proves L'Hôpital; series give limits quickly.
- [Maxima and minima](maxima-minima-optimization.md) — increasing/decreasing from the sign of f' (MVT); second-order Taylor behind the second-derivative test and gradient-descent analysis.
- [Integration](integration.md) — integrate series term by term; Taylor remainders; the mean value theorem for integrals.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — Taylor expansions give growth approximations, e.g. ln(1 + x) ≈ x, (1 + 1/n)^n ≈ e.
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) and [neural networks](../16-machine-learning/neural-networks.md) — second-order Taylor expansion of the loss motivates gradient descent step sizes and Newton's method.
- [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) — generating functions are formal power series; 1/(1 - x) expansion.
- [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — moment generating function e^(tX) expanded as a series gives the moments.

## Practice

**Q1 (NAT).** For f(x) = x² - 4x + 3 on [1, 3], the value of c in Rolle's theorem is ___.

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** f(1) = f(3) = 0; f'(x) = 2x - 4 = 0 ⇒ x = 2.

</details>

**Q2 (NAT).** For f(x) = x³ on [0, 2], the value c in Lagrange's MVT is ___ (2 decimals).

<details><summary>Answer</summary>

**Answer:** 1.15  
**Solution:** 3c² = (8 - 0)/2 = 4 ⇒ c = 2/√3 ≈ 1.1547 (reject the negative root).

</details>

**Q3 (MCQ).** On which interval does Rolle's theorem apply to f(x) = |x|? (A) [-1, 1] (B) [0, 1] (C) [-1, 0] (D) none of these

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:** On [-1, 1] f is not differentiable at 0; on [0, 1] and [-1, 0], f(a) ≠ f(b). So the hypotheses fail on all three; "none".

</details>

**Q4 (NAT).** The coefficient of x³ in the Maclaurin series of e^x sin x is ___ (as a decimal, 3 d.p.).

<details><summary>Answer</summary>

**Answer:** 0.333  
**Solution:** (1 + x + x²/2 + ...)(x - x³/6) → x³: 1·(-1/6) + 1·(1/2) = 1/3.

</details>

**Q5 (NAT).** lim_{x→0} (tan x - sin x)/x³ = ___.

<details><summary>Answer</summary>

**Answer:** 0.5  
**Solution:** tan x - sin x = (x + x³/3) - (x - x³/6) + O(x⁵) = x³/2.

</details>

**Q6 (MCQ).** If f(0) = 2 and f'(x) ≤ 3 for all x, the largest possible value of f(4) is (A) 8 (B) 12 (C) 14 (D) 6

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** f(4) - f(0) = 4 f'(c) ≤ 12 so f(4) ≤ 14, attained by f(x) = 2 + 3x.

</details>

**Q7 (NAT).** f(x) = e^(2x). f'''(0) = ___.

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:** Differentiating three times multiplies by 2³ = 8; at 0 e⁰ = 1. (Or n! × coefficient: 6 × 8/6.)

</details>

**Q8 (MSQ).** Which statements are true? (A) If f' = 0 on (a, b) then f is constant there. (B) The Maclaurin series of cos x has only even powers. (C) Lagrange's MVT holds for f(x) = 1/x on [-1, 1]. (D) x³ - 3x + 1 has at most one root in (-1, 1).

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)  
**Solution:** (A) consequence of MVT. (B) cos is even. (C) false: 1/x is discontinuous at 0. (D) By Rolle, two roots would force f'(x) = 3x² - 3 = 0 in (-1, 1), but f' < 0 there.

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
