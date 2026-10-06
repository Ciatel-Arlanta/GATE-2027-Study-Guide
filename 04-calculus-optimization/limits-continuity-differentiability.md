# Functions, Limits, Continuity and Differentiability

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Functions of a single variable; Limits; Continuity; Differentiability
> **Prerequisites:** [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) · **Leads to:** [Mean value theorems and Taylor series](mean-value-theorems-and-taylor.md) · [Maxima, minima, optimisation](maxima-minima-optimization.md) · [Integration](integration.md)

## Quick glance
- **Limit** lim_{x→a} f(x) = L exists iff the left limit and right limit both exist and equal L. The value f(a) is irrelevant.
- **Continuous at a** iff lim_{x→a} f(x) = f(a) (three things: f(a) defined, limit exists, they are equal).
- **Differentiable at a** iff lim_{h→0} (f(a + h) - f(a))/h exists (left derivative = right derivative). **Differentiable ⇒ continuous; the converse is false** (|x| at 0).
- Standard limits: sin x/x → 1, (1 - cos x)/x² → 1/2, (e^x - 1)/x → 1, ln(1 + x)/x → 1, (a^x - 1)/x → ln a, (1 + 1/x)^x → e (x → ∞), (1 + x)^(1/x) → e (x → 0).
- **L'Hôpital:** for 0/0 or ∞/∞, lim f/g = lim f'/g' (if the latter exists). Other forms (0·∞, ∞ - ∞, 1^∞, 0^0, ∞^0) must be converted first.
- Exponent forms: for 1^∞, lim f^g = e^{lim g (f - 1)}.
- Discontinuities: **removable** (limit exists ≠ f(a)), **jump** (both one-sided limits exist, differ), **infinite**, **oscillatory**.
- #1 trap: L'Hôpital applied to a non-indeterminate form (e.g. at 1/0 or 2/3) gives a wrong answer; check the form first.

## 1. Functions of a single variable

A function f: D → R assigns one output to each input in the domain D. Vocabulary GATE assumes:

| Term | Meaning |
|---|---|
| Domain / range | allowed inputs / set of outputs |
| One-one (injective) / onto | distinct inputs → distinct outputs / every target value is hit |
| Even / odd | f(-x) = f(x) (y-axis symmetry) / f(-x) = -f(x) (origin symmetry) |
| Periodic | f(x + T) = f(x); sin, cos have T = 2π, tan has π |
| Monotonic | increasing: x1 < x2 ⇒ f(x1) ≤ f(x2); decreasing similarly |
| Composition | (f ∘ g)(x) = f(g(x)) |
| Bounded | \|f(x)\| ≤ M for all x |

Common functions and their domains: polynomial (R); 1/x (x ≠ 0); √x (x ≥ 0); ln x (x > 0); e^x (R, range > 0); |x|; floor ⌊x⌋ (jumps at integers); sin, cos (R, range [-1, 1]).

**Worked example 1 — even/odd.** f(x) = x³ + sin x is odd (sum of odd functions); g(x) = x² cos x is even (even × even); h(x) = x + x² is neither (f(-x) = -x + x²). The product of two odd functions is even; odd × even = odd.

## 2. Limits

**Intuition.** lim_{x→a} f(x) = L says: as x gets close to a (but not equal to a), f(x) gets close to L. It describes the **trend**, not the value at a.

**One-sided limits.** Left limit lim_{x→a⁻}, right limit lim_{x→a⁺}. **The limit exists iff both exist and are equal.**

**Worked example 2.** f(x) = |x|/x: right limit at 0 is 1, left limit is -1, so the limit does not exist. f(x) = ⌊x⌋ at x = 2: left limit 1, right limit 2.

**Limit laws.** If lim f = L, lim g = M: lim (f ± g) = L ± M, lim fg = LM, lim f/g = L/M (M ≠ 0), lim f^n = L^n, and **squeeze theorem**: g ≤ f ≤ h with lim g = lim h = L ⇒ lim f = L.

**Worked example 3 — squeeze.** lim_{x→0} x sin(1/x): since -|x| ≤ x sin(1/x) ≤ |x| and both bounds → 0, the limit is **0**. (But lim_{x→0} sin(1/x) does not exist: it oscillates between -1 and 1.)

### 2.1 Standard limits

| Limit | Value |
|---|---|
| lim_{x→0} sin x / x; tan x / x; sin⁻¹x / x; tan⁻¹x / x | 1 |
| lim_{x→0} (1 - cos x)/x² | 1/2 |
| lim_{x→0} (e^x - 1)/x | 1 |
| lim_{x→0} (a^x - 1)/x | ln a |
| lim_{x→0} ln(1 + x)/x | 1 |
| lim_{x→0} (1 + x)^(1/x); lim_{x→∞} (1 + 1/x)^x | e |
| lim_{x→∞} (1 + k/x)^x; lim_{x→0} (1 + kx)^(1/x) | e^k |
| lim_{x→a} (x^n - a^n)/(x - a) | n a^(n-1) |
| lim_{x→0} ((1 + x)^n - 1)/x | n |
| lim_{x→∞} x^n / e^x (any n); ln x / x^p (p > 0) | 0 (exponential beats polynomial beats log) |
| lim_{x→0⁺} x^p ln x (p > 0); x^x | 0; 1 |
| lim_{x→∞} x^(1/x); n^(1/n); a^(1/n) (a > 0) | 1 |
| lim_{x→∞} sin x / x | 0 (bounded over growing) |
| lim_{n→∞} n!/n^n; a^n/n! | 0 |

**Worked example 4.** (a) lim_{x→0} sin 5x / sin 2x = (sin 5x/5x)(2x/sin 2x)(5x/2x) = **5/2**. (b) lim_{x→0} (3^x - 2^x)/x = ln 3 - ln 2 = **ln(3/2)**. (c) lim_{x→2} (x³ - 8)/(x - 2) = 3 · 2² = **12**. (d) lim_{x→∞} (1 + 2/x)^(3x) = ((1 + 2/x)^x)³ → (e²)³ = **e⁶**.

### 2.2 Algebraic techniques for 0/0 and ∞ - ∞

- **Factor and cancel:** (x² - 1)/(x - 1) = x + 1 → 2.
- **Rationalise:** lim_{x→∞} (√(x² + x) - x) = lim x/(√(x² + x) + x) = lim 1/(√(1 + 1/x) + 1) = **1/2**.
- **Divide by the highest power** for x → ∞ rational functions: degree top < bottom → 0; equal → ratio of leading coefficients; top > bottom → ±∞.

**Worked example 5.** lim_{x→∞} (3x² + 5x)/(2x² - 1) = 3/2. lim_{x→-∞} (x + 1)/√(x² + 1): √(x²) = |x| = -x for x < 0, so the limit is **-1** (and +1 as x → +∞).

### 2.3 L'Hôpital's rule

If f/g has the form **0/0 or ∞/∞** at a (a may be ±∞), f and g differentiable near a and g' ≠ 0, then **lim f/g = lim f'/g'** provided the right-hand limit exists. It can be repeated.

**Worked example 6.** (a) lim_{x→0} (e^x - 1 - x)/x²: 0/0 → (e^x - 1)/(2x): 0/0 → e^x/2 → **1/2**. (b) lim_{x→0} (x - sin x)/x³ → (1 - cos x)/(3x²) → sin x/(6x) → **1/6** (so (sin x - x)/x³ → -1/6). (c) lim_{x→∞} ln x / x → (1/x)/1 = **0**.

**Converting other forms** (write as a quotient, or take logarithms):

| Form | Technique | Example |
|---|---|---|
| 0 · ∞ | f g = f/(1/g) | lim_{x→0⁺} x ln x = lim ln x/(1/x) = lim (1/x)/(-1/x²) = lim(-x) = **0** |
| ∞ - ∞ | common denominator / rationalise | lim_{x→0} (1/x - 1/sin x) = lim (sin x - x)/(x sin x) = lim (cos x - 1)/(sin x + x cos x) = 0/0 → lim (-sin x)/(2cos x - x sin x) = **0** |
| 1^∞, 0^0, ∞^0 | y = f^g, then ln y = g ln f, find its limit L, answer e^L | lim_{x→0⁺} x^x: ln y = x ln x → 0, so y → e⁰ = **1** |
| 1^∞ shortcut | lim f^g = e^{lim g (f - 1)} | lim_{x→0} (cos x)^(1/x²) = e^{lim (cos x - 1)/x²} = e^{-1/2} |

**Worked example 7.** lim_{x→∞} ((x + 3)/(x + 1))^x: this is 1^∞. Exponent g (f - 1) = x · 2/(x + 1) → 2. Limit = **e²**.

**Worked example 8 — when L'Hôpital does not apply.** lim_{x→0} (x + 1)/(x + 2) = 1/2 is plain substitution; differentiating (giving 1/1 = 1) would be wrong. And lim_{x→∞} (x + sin x)/x: ratio of derivatives (1 + cos x)/1 has no limit, but the original limit is 1 (divide by x). **L'Hôpital's conclusion holds only when the derivative limit exists; if it does not, the rule is silent and you must try another method.**

**Taylor-series method (often faster).** Replace each function by its series: sin x = x - x³/6 + ..., cos x = 1 - x²/2 + ..., e^x = 1 + x + x²/2 + ... . Example: (tan x - x)/x³ with tan x = x + x³/3 + ... gives **1/3**. See [Taylor series](mean-value-theorems-and-taylor.md).

## 3. Continuity

**Definition.** f is **continuous at a** iff (1) f(a) is defined, (2) lim_{x→a} f(x) exists, (3) lim f(x) = f(a). Continuous on an interval means continuous at every point (one-sided at the endpoints).

**Continuous functions:** polynomials, e^x, sin, cos, |x| everywhere; rational functions where the denominator ≠ 0; ln x for x > 0; √x for x ≥ 0. Sums, products, quotients (denominator ≠ 0) and compositions of continuous functions are continuous.

### 3.1 Types of discontinuity

| Type | Behaviour | Example at x = 0 or given point |
|---|---|---|
| Removable | limit exists, but ≠ f(a) or f(a) undefined | (x² - 1)/(x - 1) at 1 (limit 2); sin x / x at 0 |
| Jump | left and right limits exist but differ | ⌊x⌋ at integers; sign(x) at 0 |
| Infinite | a one-sided limit is ±∞ | 1/x, 1/x² at 0; tan x at π/2 |
| Oscillatory | no limit because of oscillation | sin(1/x) at 0 |

Jump, removable: **discontinuity of the first kind**; infinite, oscillatory: second kind.

**Worked example 9 — finding a constant.** f(x) = sin 3x / x for x ≠ 0 and f(0) = k. lim_{x→0} sin 3x/x = 3, so f is continuous at 0 iff **k = 3**.

**Worked example 10 — piecewise.** f(x) = x² for x < 1, a at x = 1, 2 - x for x > 1: left limit 1, right limit 1, so f is continuous at 1 iff a = 1. Another: f(x) = ax + 3 for x ≤ 2, x² - 1 for x > 2: continuity at 2 needs 2a + 3 = 3, so a = 0.

### 3.2 Theorems on continuous functions on [a, b]

- **Intermediate Value Theorem (IVT):** f takes every value between f(a) and f(b). In particular, **if f(a) f(b) < 0 there is a root in (a, b)**. Used to show an equation has a solution (bisection method).
- **Extreme Value Theorem:** f attains a maximum and a minimum on [a, b] (closed and bounded needed: 1/x on (0, 1] has no maximum).
- A continuous function on [a, b] is bounded.

**Worked example 11.** x³ - x - 1 = 0 has a root in (1, 2): f(1) = -1 < 0, f(2) = 5 > 0, f continuous.

## 4. Differentiability

**Definition.** f'(a) = lim_{h→0} (f(a + h) - f(a))/h = lim_{x→a} (f(x) - f(a))/(x - a). Geometrically the slope of the tangent at a. Differentiable at a ⇔ the left derivative f'(a⁻) and right derivative f'(a⁺) exist and are equal.

**Theorem.** **Differentiable at a ⇒ continuous at a.** Proof: f(a + h) - f(a) = h · (f(a + h) - f(a))/h → 0 · f'(a) = 0. **Converse is false.**

**Where continuous functions fail to be differentiable:** corners (|x| at 0, slopes -1 and +1), cusps (x^(2/3) at 0), vertical tangents (x^(1/3) at 0, derivative infinite), discontinuities.

**Worked example 12 — |x| at 0.** f'(0⁺) = lim_{h→0⁺} h/h = 1; f'(0⁻) = lim_{h→0⁻} (-h)/h = -1. Not equal: not differentiable at 0 although continuous.

**Worked example 13 — x sin(1/x) vs x² sin(1/x).** With f(0) = 0:
- f(x) = x sin(1/x): (f(h) - f(0))/h = sin(1/h), which oscillates: **continuous at 0, not differentiable**.
- g(x) = x² sin(1/x): (g(h) - 0)/h = h sin(1/h) → 0 by squeeze, so **g'(0) = 0**. For x ≠ 0, g'(x) = 2x sin(1/x) - cos(1/x), which has no limit as x → 0, so **g' is not continuous at 0**: differentiable does not imply continuously differentiable.

**Worked example 14 — making a piecewise function differentiable.** f(x) = x² for x ≤ 1, ax + b for x > 1. Continuity at 1: a + b = 1. Differentiability: left slope 2x = 2, right slope a, so **a = 2, b = -1**.

**Worked example 15.** f(x) = x|x|: for x > 0 it is x², for x < 0 it is -x². f'(0) = lim h|h|/h = lim |h| = 0. So x|x| is differentiable at 0 (f' = 2|x|), but f'' does not exist at 0. And |x|³ has f'(0) = 0, f''(0) = 0 but f''' does not exist at 0.

### 4.1 Derivative rules and standard derivatives

| Rule | Formula |
|---|---|
| Sum, constant | (cf ± g)' = cf' ± g' |
| Product | (fg)' = f'g + fg' |
| Quotient | (f/g)' = (f'g - fg')/g² |
| Chain | (f(g(x)))' = f'(g(x)) g'(x) |
| Inverse | (f⁻¹)'(y) = 1/f'(x) where y = f(x) |
| Implicit | differentiate both sides; solve for dy/dx |
| Log differentiation | y = f^g: ln y = g ln f, then differentiate |
| Parametric | dy/dx = (dy/dt)/(dx/dt) |

| f(x) | f'(x) | f(x) | f'(x) |
|---|---|---|---|
| x^n | n x^(n-1) | sin x | cos x |
| e^x | e^x | cos x | -sin x |
| a^x | a^x ln a | tan x | sec² x |
| ln x | 1/x | cot x | -csc² x |
| log_a x | 1/(x ln a) | sec x | sec x tan x |
| 1/x | -1/x² | sin⁻¹ x | 1/√(1 - x²) |
| √x | 1/(2√x) | tan⁻¹ x | 1/(1 + x²) |
| x^x | x^x (1 + ln x) | sinh x, cosh x | cosh x, sinh x |
| \|x\| | sign(x), x ≠ 0 | ln \|x\| | 1/x |

**Worked example 16.** y = x^x: ln y = x ln x, y'/y = ln x + 1, so y' = x^x (1 + ln x). y = x² e^(-x): y' = 2x e^(-x) - x² e^(-x) = x(2 - x) e^(-x). y = sin(x²): y' = 2x cos(x²). y = tan⁻¹(1/x): y' = (1/(1 + 1/x²))(-1/x²) = -1/(x² + 1).

**Higher derivatives.** (e^(ax))^(n) = a^n e^(ax); (sin x)^(n) = sin(x + nπ/2); (x^m)^(n) = m!/(m - n)! x^(m-n); Leibniz for products (fg)^(n) = Σ C(n,k) f^(k) g^(n-k).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Limit exists | left = right | piecewise functions |
| Continuity | lim = f(a) | find constants |
| Differentiable ⇒ continuous | not conversely | true/false statements |
| sin x / x, (e^x - 1)/x, ln(1 + x)/x | 1 | any 0/0 |
| (1 - cos x)/x² | 1/2 | 0/0 |
| (a^x - 1)/x | ln a | 0/0 |
| 1^∞ | e^{lim g (f - 1)} | exponent limits |
| L'Hôpital | f/g → f'/g' only for 0/0, ∞/∞ | indeterminate forms |
| Growth | ln x ≪ x^p ≪ a^x ≪ x! ≪ x^x | limits at ∞ |
| IVT | f(a) f(b) < 0 ⇒ root | existence of roots |
| Chain/product/quotient | standard | differentiation |

## GATE traps
- **Limit ≠ value.** f(a) can be anything or undefined; lim depends only on nearby points.
- Applying L'Hôpital when the form is not 0/0 or ∞/∞, or when the derivative quotient has no limit.
- 1^∞ is **not** 1: (1 + 1/n)^n → e. Check the base tends to exactly 1.
- 0 · ∞, ∞ - ∞ must be rewritten as a quotient first.
- |x| is continuous but not differentiable at 0; differentiable needs both one-sided derivatives equal (also equal values for continuity).
- x sin(1/x) is continuous at 0 (with f(0) = 0) but not differentiable; x² sin(1/x) is differentiable at 0 but its derivative is discontinuous.
- sin(1/x) as x → 0 has **no limit**; sin x / x as x → ∞ has limit 0.
- √(x²) = |x|, important for limits at -∞.
- For a piecewise function, differentiability needs continuity **and** matching slopes.
- IVT needs continuity on the closed interval; EVT needs a closed bounded interval.

## Connections
- [Mean value theorems and Taylor series](mean-value-theorems-and-taylor.md) — Rolle/Lagrange need continuity on [a, b] and differentiability on (a, b); Taylor expansions give quick limits.
- [Maxima and minima](maxima-minima-optimization.md) — derivative zero/non-existence marks critical points.
- [Integration](integration.md) — continuity guarantees integrability; improper integrals are limits.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — limit of f(n)/g(n) decides Θ/o/ω; L'Hôpital compares growth rates.
- [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) — convergence of series, e as a limit.
- [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — CDF continuity, PDF as derivative of CDF.
- [Neural networks](../16-machine-learning/neural-networks.md) — differentiable activations (sigmoid, tanh); ReLU is continuous but not differentiable at 0.

## Practice

**Q1 (NAT).** lim_{x→0} sin(5x)/sin(2x) = ___.

<details><summary>Answer</summary>

**Answer:** 2.5  
**Solution:** sin 5x/sin 2x = (sin 5x/5x)(2x/sin 2x)(5/2) → 1 · 1 · 5/2.

</details>

**Q2 (MCQ).** lim_{x→∞} (1 + 2/x)^(3x) = (A) e² (B) e³ (C) e⁶ (D) 1

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** (1 + 2/x)^x → e², cube it: e⁶.

</details>

**Q3 (NAT).** f(x) = (sin 3x)/x for x ≠ 0, f(0) = k, is continuous at 0. k = ___.

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** lim sin 3x / x = 3 · lim sin 3x/(3x) = 3.

</details>

**Q4 (NAT).** lim_{x→0} (e^x - 1 - x)/x² = ___.

<details><summary>Answer</summary>

**Answer:** 0.5  
**Solution:** Series e^x = 1 + x + x²/2 + ... gives (x²/2)/x² = 1/2. (Or L'Hôpital twice.)

</details>

**Q5 (MCQ).** Which function is continuous at 0 but not differentiable at 0? (A) x² (B) |x| (C) x|x| (D) sin x

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** |x| has one-sided derivatives -1 and 1. x|x| has derivative 0 at 0 (lim h|h|/h = 0). x² and sin x are differentiable.

</details>

**Q6 (NAT).** f(x) = x² for x ≤ 1, ax + b for x > 1 is differentiable at 1. a + b = ___.

<details><summary>Answer</summary>

**Answer:** 1  
**Solution:** Slope: a = 2. Continuity: a + b = 1² = 1, so b = -1 and a + b = 1.

</details>

**Q7 (NAT).** lim_{x→0} (cos x)^(1/x²) = e^p. p = ___.

<details><summary>Answer</summary>

**Answer:** -0.5  
**Solution:** 1^∞ form: p = lim (cos x - 1)/x² = -1/2.

</details>

**Q8 (MSQ).** Which statements are true? (A) A differentiable function is continuous. (B) lim_{x→0} x sin(1/x) exists. (C) lim_{x→0} sin(1/x) exists. (D) lim_{x→∞} (√(x² + x) - x) = 1/2.

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)  
**Solution:** (A) standard theorem. (B) squeeze gives 0. (C) false: oscillates in [-1, 1]. (D) multiply by the conjugate: x/(√(x² + x) + x) → 1/(1 + 1) = 1/2.

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
