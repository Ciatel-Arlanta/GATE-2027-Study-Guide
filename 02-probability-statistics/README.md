# Probability and Statistics

> **Paper:** CS+DA · **Priority:** P0/P1

**PYQ weightage:** ~2.6 Qs per CSE paper (2021–26), trend steady; DA: 36 Qs in 2024–26.

Probability quantifies uncertainty; statistics uses samples to reason about a population. First learn events and conditional probability, then random variables and distributions, then inference. DA includes deeper continuous distributions and tests; see [coverage](../COVERAGE.md) for paper tags.

| Order | Chapter | Focus | PYQ weightage |
|---|---|---|---|
| 1 | [Probability basics](probability-basics.md) | Counting, events, conditional probability, Bayes | High · 29 |
| 2 | [Random variables and moments](random-variables-and-moments.md) | PMF/PDF/CDF, expectation, variance | High · 28 |
| 3 | [Discrete distributions](discrete-distributions.md) | Bernoulli, binomial, geometric, Poisson | Low · 5 |
| 4 | [Continuous distributions](continuous-distributions.md) | Uniform, exponential, normal, t, chi-square | High · 19 |
| 5 | [Statistical inference](statistical-inference.md) | Sampling, CLT, intervals, hypothesis tests | Medium · 6 |

## Study priority

- [Probability basics](probability-basics.md) (High · 29) — master combinatorial probability, conditional probability and Bayes' theorem first; the foundation everything else is built on.
- [Random variables and moments](random-variables-and-moments.md) (High · 28) — master expectation via indicator variables, PDF/CDF manipulation, mean/median/mode of grouped data, and covariance and correlation.
- [Continuous distributions](continuous-distributions.md) (High · 19) — master the exponential, uniform and normal cases: memoryless property, transformations, and standardisation to Z.
- [Statistical inference](statistical-inference.md) (Medium · 6) — master the CLT, confidence intervals and the standard hypothesis tests; small in CSE but a large share of the DA paper.
- [Discrete distributions](discrete-distributions.md) (Low · 5) — master the Poisson distribution and Bernoulli/binomial formulas; geometric is the weakest subtopic in the section.

```mermaid
flowchart LR
 C[Counting] --> P[Probability rules]
 P --> R[Random variables]
 R --> D[Distributions]
 D --> I[Inference: sample to population]
```

Connections: [combinatorics](../01-discrete-mathematics/combinatorics.md) counts outcomes; [linear algebra](../03-linear-algebra/README.md) represents covariance and PCA; [ML foundations](../16-machine-learning/ml-foundations.md) uses likelihood and validation.

[Cheatsheet](CHEATSHEET.md) · [Checkpoint](CHECKPOINT.md) · [Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
