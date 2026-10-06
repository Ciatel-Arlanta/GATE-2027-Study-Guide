# Brief: 02-probability-statistics (CS+DA; CS ~2-4 marks, DA ~15-20 marks; DA tests inference heavily)

Plan topics: Counting: permutations and combinations; Probability axioms; Sample spaces and events; Independent events; Mutually exclusive events; Marginal probability; Conditional probability; Joint probability; Bayes theorem; Conditional expectation; Conditional variance; Mean, median and mode; Standard deviation; Correlation; Covariance; Random variables; Discrete random variables and PMFs; Bernoulli distribution; Binomial distribution; Uniform distribution (discrete); Continuous random variables and PDFs; Uniform distribution (continuous); Exponential distribution; Poisson distribution; Normal distribution; Standard normal distribution; t-distribution; Chi-squared distribution; Cumulative distribution function; Conditional PDF; Central limit theorem; Confidence intervals; z-test; t-test; Chi-squared test.

Files: README.md, probability-basics.md, random-variables-and-moments.md, discrete-distributions.md, continuous-distributions.md, statistical-inference.md, CHEATSHEET.md, CHECKPOINT.md.

Must cover (verify numbers with Python, e.g. simulation or math.erf):
- Counting for probability (link ../01-discrete-mathematics/combinatorics.md for depth); axioms and consequences; inclusion-exclusion for probability.
- Independence vs mutual exclusivity (disjoint events with non-zero probability are dependent); pairwise vs mutual independence; total probability; Bayes via tree diagrams and the table method (disease-test style).
- Joint/marginal tables for discrete RVs; linearity of expectation with indicator variables (expected fixed points, expected empty bins, hashing collisions).
- Variance properties (Var(aX+b), Var(X+Y) with covariance); covariance and correlation (independence implies zero covariance, not conversely, with example).
- Law of total expectation and law of total variance; conditional PDF from a joint PDF; median/mode of distributions; CDF properties; basic transformations of RVs.
- Memorylessness of geometric and exponential; Poisson as a binomial limit; Poisson process link to exponential; geometric distribution (often needed though not listed).
- Normal standardisation with a mini z-table (Phi at 1, 1.645, 1.96, 2, 2.33, 2.576, 3) and the 68-95-99.7 rule; sums of independent normals.
- Chi-squared as a sum of squared standard normals (mean, variance); t-distribution (when used, degrees of freedom, heavier tails, tends to normal).
- Sample mean/variance (n-1 divisor and why); CLT statement and use; confidence intervals (z and t; correct interpretation).
- Hypothesis testing: H0/H1, test statistic, p-value, significance level, type I/II errors, one- vs two-tailed, critical values; one-sample z-test; one- and two-sample t-tests; chi-squared goodness-of-fit and independence (contingency table, expected counts, df = (r-1)(c-1)).
- A master distribution table: PMF/PDF, support, mean, variance, typical use.

Connections: 16-machine-learning (naive Bayes, LDA, Gaussian assumptions, MLE), 17-artificial-intelligence/probabilistic-reasoning.md, 08-algorithms/hashing.md (expected probes), 08-algorithms/searching-and-sorting.md (randomised quicksort expectation), 15-computer-networks (ALOHA probability), 03-linear-algebra (covariance matrix -> PCA), 04-calculus-optimization/integration.md (PDF integrals).
