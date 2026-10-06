# Statistical inference: sampling, intervals, and tests

> **Paper:** DA (advanced) · **Priority:** P1 · **Plan topics:** sampling distributions, CLT, confidence intervals, z-test, t-test, chi-square test
> **Prerequisites:** [Continuous distributions](continuous-distributions.md) · **Leads to:** [ML foundations](../16-machine-learning/ml-foundations.md)

## Quick glance
- A statistic is computed from a sample; its sampling distribution describes how it varies across repeated samples.
- CLT: for sufficiently large n under usual independence/finite-variance conditions, sample mean is approximately normal with mean $\mu$ and standard error $\sigma/\sqrt n$.
- Confidence level describes long-run coverage of the interval procedure, not probability that a fixed parameter moves.
- Hypothesis test: state $H_0,H_1$, choose statistic and significance $\alpha$, then compare p-value with $\alpha$.
- p-value is probability (under $H_0$) of data at least as extreme; it is not $P(H_0\mid data)$.

## 1. Standard error and CLT
For independent observations with population SD $\sigma$, sample mean has standard error $\sigma/\sqrt n$. If $\sigma=12$ and $n=36$, standard error is 2. Increasing sample size by factor 4 halves standard error.

## 2. Confidence intervals
When population SD is known, a 95% normal interval for a mean is $\bar x\pm1.96\sigma/\sqrt n$. If $\bar x=50,\sigma=10,n=100$, interval is $50\pm1.96=[48.04,51.96]$. When sigma is unknown and data are normal, replace 1.96 with the relevant t critical value and sigma with sample SD.

## 3. Hypothesis testing
For two-sided z-test, $H_0:\mu=\mu_0$, $H_1:\mu\ne\mu_0$. Compute $z=(\bar x-\mu_0)/(\sigma/\sqrt n)$. At 5% significance, reject when $|z|>1.96$ (large-sample normal test). Failing to reject is not proof that $H_0$ is true.

Type I error: reject true null, probability controlled by $\alpha$. Type II error: fail to reject false null; power is $1-\beta$. Chi-square tests compare observed and expected counts or test association in contingency tables.

## GATE traps
- Statistical significance does not imply practical importance.
- A 95% interval procedure covers the true parameter in 95% of repeated samples under its assumptions.
- Choose one- vs two-sided alternative before looking at data.
- A small p-value is evidence against $H_0$, not the probability that $H_0$ is false.
- Check independence, sampling design, and distribution assumptions.

## Connections
- [Continuous distributions](continuous-distributions.md) — critical values come from normal/t/chi-square laws.
- [Probability basics](probability-basics.md) — conditional probability supports Bayesian interpretation.
- [ML foundations](../16-machine-learning/ml-foundations.md) — train/validation samples estimate generalisation.

## Practice
**Q1 (NAT).** If $\sigma=8,n=64$, find standard error of sample mean.
<details><summary>Answer</summary> $8/\sqrt{64}=1$.</details>

**Q2.** A test reports p=0.03 at $\alpha=0.05$. Decision?
<details><summary>Answer</summary> Reject $H_0$ at the 5% level, assuming test conditions hold.</details>

**Q3.** Does p=0.03 mean there is a 3% probability that the null is true?
<details><summary>Answer</summary> No. It is a tail probability for the data under the null model.</details>
