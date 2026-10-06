# Maxima, Minima and Single-Variable Optimisation

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Maxima and minima; Single-variable optimization
> **Prerequisites:** [Limits, continuity, differentiability](limits-continuity-differentiability.md) · [Mean value theorems and Taylor](mean-value-theorems-and-taylor.md) · **Leads to:** [Integration](integration.md) · [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) · [Neural networks](../16-machine-learning/neural-networks.md)

## Quick glance
- **Critical point:** f'(c) = 0 or f'(c) does not exist (c in the domain). Extrema occur only at critical points or endpoints.
- **First derivative test:** f' changes + → - at c: local max; - → +: local min; no sign change: neither.
- **Second derivative test:** f'(c) = 0 and f''(c) > 0: local min; f''(c) < 0: local max; f''(c) = 0: **inconclusive** (use the first-derivative test or higher derivatives).
- **Global extrema on [a, b]:** evaluate f at all critical points inside and at both endpoints; the largest/smallest values win (Extreme Value Theorem guarantees they exist for continuous f).
- **Inflection point:** f'' changes sign (concavity changes). f''(c) = 0 is necessary, not sufficient (x⁴ at 0).
- **Convex:** f'' ≥ 0 (curve above its tangents, chord above the curve). For a convex function **every local minimum is global**; f'(c) = 0 ⇒ global minimum.
- **Gradient descent (1-D):** w ← w - η f'(w). For f(w) = (w - w*)² it converges iff 0 < η < 1.
- #1 trap: f' = 0 does not guarantee an extremum (x³ at 0), and forgetting the **endpoints** when asked for a global extremum on a closed interval.

## 1. Increasing, decreasing, critical points

**Intuition.** The sign of f' says whether the graph climbs or falls. A peak or valley must be flat (f' = 0) or sharp (f' undefined).

| f' on an interval | f is |
|---|---|
| > 0 | strictly increasing |
| < 0 | strictly decreasing |
| = 0 | constant |

**Definitions.** f has a **local maximum** at c if f(c) ≥ f(x) for all x near c; **global (absolute) maximum** if for all x in the domain. Local minima similarly. **Fermat:** if f has a local extremum at an interior point c and f'(c) exists, then f'(c) = 0.

**Worked example 1.** f(x) = x³ - 6x² + 9x + 1. f'(x) = 3x² - 12x + 9 = 3(x - 1)(x - 3). Sign of f': + for x < 1, - on (1, 3), + for x > 3. So f increases, decreases, increases: **local max at x = 1** (f = 5), **local min at x = 3** (f = 1). Second derivative: f'' = 6x - 12: f''(1) = -6 < 0 (max) ✓, f''(3) = 6 > 0 (min) ✓. Inflection at f'' = 0: x = 2.

```text
  f
  5 |   *                          *   (x=4)
    |  /  \                      /
  3 | /    \                   /
    |/      \                /
  1 *        \______________/  (x=3: local min)
    0   1    2    3    4
```

## 2. The tests for local extrema

**First derivative test.** Let c be a critical point. Look at the sign of f' just left and right of c.

| Left of c | Right of c | Conclusion |
|---|---|---|
| + | - | local max |
| - | + | local min |
| same sign | same sign | no extremum |

Works also when f' does not exist at c (e.g. f(x) = |x| has a min at 0; f(x) = x^(2/3) has a min at 0 with a cusp).

**Second derivative test.** If f'(c) = 0: f''(c) > 0 ⇒ local min; f''(c) < 0 ⇒ local max; f''(c) = 0 ⇒ **no conclusion**.

**Worked example 2 — inconclusive.** f(x) = x⁴ - 4x³: f' = 4x²(x - 3) = 0 at x = 0, 3. f'' = 12x² - 24x: f''(3) = 36 > 0 so **min at 3** with f(3) = 81 - 108 = **-27**. f''(0) = 0: inconclusive. First-derivative test: f' = 4x²(x - 3) is negative on both sides of 0, no sign change, so **0 is not an extremum** (a flat inflection-type point). Inflection points where f'' = 12x(x - 2) changes sign: x = 0 and x = 2.

**Higher-order test.** If the first non-zero derivative at c is the k-th: k even ⇒ extremum (min if f^(k)(c) > 0, max if < 0); k odd ⇒ no extremum. Examples: x⁴ at 0 (k = 4, min); x³ (k = 3, none).

**Worked example 3.** f(x) = x e^(-x): f' = (1 - x) e^(-x) = 0 at x = 1; f'' = (x - 2) e^(-x): f''(1) = -e⁻¹ < 0 so **max at x = 1, f(1) = 1/e ≈ 0.368**. As x → ∞, f → 0 so this is the global maximum on x ≥ 0. This is the shape of the Gamma/Erlang-type density and of the Poisson pmf in k.

**Worked example 4 — x² e^(-x).** f' = (2x - x²) e^(-x) = x(2 - x) e^(-x): critical points 0 and 2. Sign: - for x < 0, + on (0, 2), - for x > 2. **Local min at 0 (f = 0), local max at 2 (f = 4/e² ≈ 0.541).**

## 3. Global extrema on a closed interval

**Method (closed interval method).** For continuous f on [a, b]:
1. Find all critical points in (a, b).
2. Evaluate f at those points **and at a and b**.
3. Largest value = global max, smallest = global min.

**Worked example 5.** f(x) = x³ - 6x² + 9x + 1 on [0, 4]: critical points 1, 3. Values: f(0) = 1, f(1) = 5, f(3) = 1, f(4) = 64 - 96 + 36 + 1 = 5. **Global max 5 (at x = 1 and x = 4), global min 1 (at x = 0 and x = 3).** The endpoint x = 4 ties with the local max.

**Worked example 6.** f(x) = x³ - 3x on [-2, 2]: f' = 3x² - 3 = 0 at ±1. f(-2) = -2, f(-1) = 2, f(1) = -2, f(2) = 2. Max 2, min -2.

**Worked example 7.** f(x) = sin x + cos x on [0, π/2]: f' = cos x - sin x = 0 at π/4, f(π/4) = √2 ≈ 1.414; endpoints f(0) = f(π/2) = 1. Max √2, min 1 (at the endpoints). Over all of R the max is √2 and the min is -√2.

**On open or unbounded domains** the global extremum may not exist: check limits at the boundary (as x → ±∞ or at open ends). f(x) = x e^(-x) on R: f → -∞ as x → -∞, so no global minimum.

## 4. Convexity, concavity, inflection

**Definition.** f is **convex** (concave up) on an interval if f(λx + (1 - λ)y) ≤ λ f(x) + (1 - λ) f(y) for all x, y and λ ∈ [0, 1] (chord above the graph). Twice-differentiable: **convex ⇔ f'' ≥ 0**; strictly convex if f'' > 0. **Concave** ⇔ f'' ≤ 0.

**Properties of convex functions.**
- The graph lies above every tangent: f(y) ≥ f(x) + f'(x)(y - x).
- **Every stationary point (f' = 0) is a global minimum**; any local minimum is global. A strictly convex function has at most one minimiser.
- Sum of convex functions is convex; non-negative multiple is convex; max of convex functions is convex; convex of an affine function is convex.
- Jensen: f(E[X]) ≤ E[f(X)] for convex f.

| f(x) | f'' | Nature |
|---|---|---|
| x², e^x, e^(ax), \|x\|, x⁴ | 2, e^x, a² e^(ax), -, 12x² | convex |
| -ln x, x ln x (x > 0) | 1/x², 1/x | convex |
| ln x, √x | -1/x², -1/(4x^(3/2)) | concave |
| x³ | 6x | convex for x > 0, concave for x < 0, inflection at 0 |
| sin x | -sin x | concave on (0, π), convex on (π, 2π) |
| log(1 + e^(-z)) (logistic loss) | σ(z)(1 - σ(z)) > 0 | convex |
| sigmoid σ(z) | σ'(1 - 2σ) | convex for z < 0, concave for z > 0 |

**Inflection point.** A point where concavity changes (f'' changes sign). Necessary: f''(c) = 0 (or does not exist). Example: x³ - 3x² has f'' = 6x - 6, sign change at **x = 1**. But x⁴ has f''(0) = 0 with no sign change: not an inflection.

**Worked example 8.** f(x) = x⁴ - 4x³: f'' = 12x(x - 2): positive for x < 0, negative on (0, 2), positive for x > 2. Inflection points at x = 0 and x = 2; f is convex on (-∞, 0] and [2, ∞), concave on [0, 2].

## 5. Optimisation word problems

**Method.** (1) Name the variable and write the quantity to optimise as a function of **one** variable using the constraint. (2) Fix the domain. (3) Differentiate, solve f' = 0. (4) Confirm max/min with f'' or endpoints. (5) Answer the question asked (value or argument).

**Worked example 9 — open box.** A 12 × 12 sheet; cut squares of side x from the corners and fold up. V(x) = x(12 - 2x)², 0 < x < 6. V' = (12 - 2x)² - 4x(12 - 2x) = (12 - 2x)(12 - 6x) = 0: x = 2 (x = 6 gives V = 0). V'' at 2: V' = 12(6 - x)(2 - x)·... compute V = 4x³ - 48x² + 144x, V' = 12x² - 96x + 144, V'' = 24x - 96 = -48 < 0 at x = 2: max. **V(2) = 2 · 8² = 128.**

**Worked example 10 — closed cylinder.** Volume 1000 fixed. Minimise surface S = 2πr² + 2πrh with h = 1000/(πr²): S(r) = 2πr² + 2000/r. S' = 4πr - 2000/r² = 0 ⇒ r³ = 500/π, **r = (500/π)^(1/3) ≈ 5.42**, and h = 1000/(π r²) = 2r (height equals the diameter). S'' = 4π + 4000/r³ > 0 so it is a min. **S_min ≈ 553.6.**

**Worked example 11 — nearest point.** Point on y² = 4x nearest to (4, 0): d² = (x - 4)² + 4x = x² - 4x + 16. Derivative 2x - 4 = 0: **x = 2**, y = ±2√2, d² = 12, **d = 2√3 ≈ 3.46**. (Minimise d², not d: same minimiser.)

**Worked example 12 — AM–GM style.** x > 0: f(x) = x + 1/x, f' = 1 - 1/x² = 0 at x = 1, f'' = 2/x³ > 0: min value **2**. f(x) = x² + 16/x: f' = 2x - 16/x² = 0 ⇒ x³ = 8, x = 2, f(2) = 4 + 8 = **12** (min; f'' = 2 + 32/x³ > 0).

**Worked example 13 — fencing.** 100 m of fence enclose a rectangle with one side along a wall. Sides x, y with 2x + y = 100, area A = x(100 - 2x). A' = 100 - 4x = 0, x = 25, y = 50, **A = 1250 m²** (A'' = -4 < 0).

## 6. Optimisation in machine learning (1-D intuition)

**Gradient descent.** To minimise a differentiable f, repeat **w ← w - η f'(w)**, η > 0 the learning rate. It moves downhill; at a stationary point f' = 0 it stops.

**Worked example 14.** f(w) = (w - 3)², f'(w) = 2(w - 3). Update: w_new = w - 2η(w - 3), so (w_new - 3) = (1 - 2η)(w - 3): the error is multiplied by (1 - 2η) each step.
- η = 0.25: factor 0.5. From w₀ = 0: w₁ = 1.5, w₂ = 2.25, w₃ = 2.625, ... → 3.
- η = 0.5: factor 0: w₁ = 3 in one step (optimal for a quadratic).
- η = 1: factor -1: 0 → 6 → 0 → 6... oscillates forever.
- η > 1: |factor| > 1, **diverges**. Convergence iff 0 < η < 1 (in general η < 2/L where L = max f'').

**Convex ⇒ GD finds the global minimum** (for small enough η). For non-convex losses (neural networks) GD may stop at a local minimum or a saddle.

**Newton's method (1-D optimisation).** w ← w - f'(w)/f''(w); for a quadratic it lands on the minimum in one step. (Root-finding version: x ← x - f(x)/f'(x).)

**Common ML losses.** Squared loss (w x - y)² is convex quadratic in w; minimiser w = Σ x y / Σ x². Mean of data minimises Σ (w - y_i)²: derivative 2 Σ (w - y_i) = 0 ⇒ **w = mean** (the median minimises Σ |w - y_i|). Logistic loss is convex; the cross-entropy with a sigmoid has no closed-form minimiser, so iterative methods are used. See [regression](../16-machine-learning/linear-and-logistic-regression.md).

**Worked example 15.** Minimise L(w) = Σ_{i=1}^{3} (w - y_i)² for y = (1, 2, 6): L' = 2(3w - 9) = 0 ⇒ w = 3 (mean), L(3) = 4 + 1 + 9 = 14, L'' = 6 > 0.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Critical point | f' = 0 or undefined | start of every problem |
| First-derivative test | sign change of f' | when f'' = 0 or f' undefined |
| Second-derivative test | f'' > 0 min, f'' < 0 max | quick classification |
| Global on [a, b] | critical points + endpoints | closed intervals |
| Convex | f'' ≥ 0; stationary ⇒ global min | optimisation, ML |
| Inflection | f'' changes sign | concavity questions |
| Gradient descent | w ← w - η f'(w); quadratic converges iff 0 < η < 1 for (w - w*)² | ML questions |
| Mean minimises Σ (w - y_i)² | w = mean | least squares |
| x + 1/x | min 2 at x = 1 (x > 0) | AM-GM |
| sin x + cos x | range [-√2, √2] | trig extrema |

## GATE traps
- f'(c) = 0 does **not** imply an extremum (x³ at 0). Check the sign change.
- f''(c) = 0 means the second-derivative test is **silent**, not that c is an inflection.
- On a closed interval always compare **endpoint values**; the global max can be at an endpoint (Example 5).
- Non-differentiable points (corners, cusps) are critical points: |x|, |x - 2|, x^(2/3) have minima there.
- A local max can be below a local min elsewhere; local and global are different.
- Minimise d² instead of d when a square root appears; remember to take the root at the end if distance is asked.
- Check the domain: x + 1/x has no minimum on the whole line; on x > 0 it is 2, on x < 0 the **maximum** is -2.
- Gradient descent step size too large diverges even on a convex function.
- A convex function can have no minimiser (e^x), and a stationary point of a non-convex function may be a saddle.

## Connections
- [Limits, continuity, differentiability](limits-continuity-differentiability.md) — derivative existence defines critical points; Extreme Value Theorem needs continuity.
- [Mean value theorems and Taylor](mean-value-theorems-and-taylor.md) — sign of f' ⇒ monotone (MVT); second-order Taylor proves the second-derivative test and gives GD step bounds.
- [Linear and logistic regression](../16-machine-learning/linear-and-logistic-regression.md) — loss minimisation, normal equations, convex logistic loss, learning rate.
- [Neural networks](../16-machine-learning/neural-networks.md) — gradient descent/backpropagation; vanishing gradients; non-convex loss.
- [Quadratic forms in linear algebra](../03-linear-algebra/matrices-and-determinants.md) — the multivariable second-derivative test is the Hessian's definiteness (positive definite ⇒ minimum).
- [Orthogonality and least squares](../03-linear-algebra/orthogonality-projections-svd.md) — setting the gradient of ‖Ax - b‖² to zero gives the normal equations.
- [Random variables and moments](../02-probability-statistics/random-variables-and-moments.md) — mean minimises squared error, median minimises absolute error; mode = maximum of the pmf/pdf; maximum likelihood.
- [Greedy algorithms](../08-algorithms/greedy-algorithms.md) and [dynamic programming](../08-algorithms/dynamic-programming.md) — discrete optimisation analogues.

## Practice

**Q1 (NAT).** The minimum value of f(x) = x² + 16/x for x > 0 is ___.

<details><summary>Answer</summary>

**Answer:** 12  
**Solution:** f' = 2x - 16/x² = 0 ⇒ x³ = 8 ⇒ x = 2. f'' = 2 + 32/x³ > 0 (min). f(2) = 4 + 8 = 12.

</details>

**Q2 (MCQ).** f(x) = x³ - 6x² + 9x + 1 has a local maximum at x = (A) 0 (B) 1 (C) 2 (D) 3

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** f' = 3(x - 1)(x - 3); f'' = 6x - 12; f''(1) = -6 < 0 ⇒ local max at 1 (value 5). x = 3 is the local min; x = 2 is the inflection point.

</details>

**Q3 (NAT).** The maximum of f(x) = x³ - 3x on [-2, 2] is ___.

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** Critical points ±1. f(-2) = -2, f(-1) = 2, f(1) = -2, f(2) = 2. Max = 2.

</details>

**Q4 (MCQ).** For f(x) = x⁴, at x = 0: (A) local max (B) local min (C) inflection (D) f'(0) ≠ 0

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** f' = 4x³ is negative for x < 0 and positive for x > 0: minimum. f''(0) = 0 so the second-derivative test is inconclusive, and it is not an inflection (f'' = 12x² ≥ 0 does not change sign).

</details>

**Q5 (NAT).** Gradient descent minimises f(w) = (w - 3)² from w₀ = 0 with learning rate η = 0.25. The value of w₂ is ___.

<details><summary>Answer</summary>

**Answer:** 2.25  
**Solution:** w₁ = 0 - 0.25 · 2(0 - 3) = 1.5. w₂ = 1.5 - 0.25 · 2(1.5 - 3) = 1.5 + 0.75 = 2.25.

</details>

**Q6 (MSQ).** Which of the following functions are convex on their domains? (A) e^(3x) (B) ln x (x > 0) (C) x ln x (x > 0) (D) -ln x (x > 0)

<details><summary>Answer</summary>

**Answer:** (A), (C), (D)  
**Solution:** f'': (A) 9e^(3x) > 0. (B) -1/x² < 0 (concave). (C) f' = ln x + 1, f'' = 1/x > 0. (D) 1/x² > 0.

</details>

**Q7 (NAT).** A square sheet of side 12 has equal squares of side x cut from its corners and is folded into an open box. The maximum volume is ___.

<details><summary>Answer</summary>

**Answer:** 128  
**Solution:** V = x(12 - 2x)². V' = (12 - 2x)(12 - 6x) = 0 ⇒ x = 2 (x = 6 gives V = 0). V(2) = 2 · 8 · 8 = 128; V'' = 24x - 96 < 0 at x = 2.

</details>

**Q8 (NAT).** The sum Σ_{i=1}^{3} (w - y_i)², with y = (1, 2, 6), is minimised at w = ___, and its minimum value is ___ (enter the sum of the two numbers).

<details><summary>Answer</summary>

**Answer:** 17  
**Solution:** The minimiser is the mean w = 9/3 = 3. Value: (3 - 1)² + (3 - 2)² + (3 - 6)² = 4 + 1 + 9 = 14. Sum 3 + 14 = 17.

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
