# Integration: accumulation and area

> **Paper:** CS · **Priority:** P2 · **Plan topics:** definite and indefinite integration
> **Prerequisites:** [Limits, continuity, differentiability](limits-continuity-differentiability.md) · **Leads to:** [Probability distributions](../02-probability-statistics/continuous-distributions.md)

## Quick glance
- An antiderivative F satisfies $F'=f$; $\int f(x)dx=F(x)+C$.
- Fundamental theorem: $\int_a^b f(x)dx=F(b)-F(a)$.
- Definite integral is signed area; regions below x-axis contribute negatively.
- Substitution reverses chain rule; integration by parts reverses product rule.
- For continuous probability density, total integral over support is 1.

## 1. Basic antiderivatives
$\int x^n dx=x^{n+1}/(n+1)+C$ for $n\ne-1$; $\int 1/x dx=\ln|x|+C$; $\int e^x dx=e^x+C$; $\int \cos x dx=\sin x+C$.

Example: $\int_0^2 3x^2dx=[x^3]_0^2=8$. The indefinite integral is $x^3+C$; the constant cancels in a definite integral.

## 2. Substitution and parts
Substitution: if an integrand contains a function and its derivative, set $u=g(x)$. Example $\int_0^1 2x e^{x^2}dx$: let $u=x^2$, giving $\int_0^1e^u du=e-1$.

Integration by parts: $\int u,dv=uv-\int v,du$. Example $\int xe^x dx=xe^x-e^x+C=e^x(x-1)+C$.

## 3. Area and symmetry
If f is odd, $\int_{-a}^{a}f(x)dx=0$; if even, it is $2\int_0^a f(x)dx$. For $f(x)=x^2$, $\int_{-1}^1x^2dx=2/3$.

## GATE traps
- Include +C only for indefinite integrals.
- Keep bounds and signs correct when reversing limits: $\int_a^b=-\int_b^a$.
- A geometric area below the axis is positive, but the definite integral is negative.
- Check by differentiating an antiderivative.

## Connections
- [Continuous distributions](../02-probability-statistics/continuous-distributions.md) — integrate a PDF to get probabilities.
- [Maxima and minima](maxima-minima-optimization.md) — derivatives find extrema; integrals measure accumulated change.
- [Limits and derivatives](limits-continuity-differentiability.md) — integration reverses differentiation.

## Practice
**Q1 (NAT).** Evaluate $\int_1^3 2x dx$.
<details><summary>Answer</summary> $[x^2]_1^3=8$.</details>

**Q2.** Evaluate $\int_0^1 1 dx$.
<details><summary>Answer</summary> 1.</details>

**Q3.** If f is odd, what is $\int_{-2}^{2}f(x)dx$?
<details><summary>Answer</summary> 0, by symmetry.</details>
