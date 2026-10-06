# Data warehousing and preprocessing

> **Paper:** DA · **Priority:** P1/P2 · **Plan topics:** data types, transformations, normalization, discretization, sampling, compression, warehouse schemas and OLAP
> **Prerequisites:** [Relational model](relational-model-algebra-calculus.md) · [ML foundations](../16-machine-learning/ml-foundations.md)

## Quick glance
- Operational databases support transactions; warehouses integrate historical data for analysis.
- Star schema: central fact table plus denormalised dimensions; snowflake normalises dimensions.
- OLAP: roll-up aggregates, drill-down adds detail, slice fixes one dimension, dice selects subcube, pivot rotates view.
- Feature scaling: min-max to [0,1]; z-score $(x-\mu)/\sigma$.
- Sampling and discretisation simplify data but can lose information; fit transformations on training data only.

## 1. Warehouse and schemas
Fact table stores measurable events (sales amount, units) plus foreign keys to dimensions (time, product, store). Dimension tables describe context and often have hierarchies such as day→month→year. Star schemas favour simple queries; snowflake schemas reduce dimension redundancy.

## 2. OLAP operations
Given sales by product, store, and quarter: roll-up quarter to year aggregates; drill-down reverses it; slice to one year; dice to selected years and product families; pivot swaps row/column dimensions. These operations change analytical view, not source facts.

## 3. Preprocessing
Min-max maps observed min→0, max→1; z-score centres at mean and scales to unit standard deviation. Outliers strongly affect min-max and mean/SD. Discretisation maps continuous values into bins; sampling selects representative rows. Compression may be lossless or lossy.

**Data leakage:** estimate scaling parameters, imputation, feature selection, and bins from training folds only; apply frozen parameters to validation/test data.

## GATE traps
- Database normal forms remove relational redundancy; feature normalisation rescales numeric values.
- Roll-up aggregates; drill-down increases granularity.
- Fact measures are usually numeric; dimensions provide descriptive context.
- Do not let test-set information determine preprocessing statistics.

## Connections
- [Constraints and normalization](constraints-and-normalization.md) — relational decomposition, distinct from numeric feature scaling.
- [ML foundations](../16-machine-learning/ml-foundations.md) — leakage and cross-validation.
- [Probability and statistics](../02-probability-statistics/README.md) — sampling and summary statistics.

## Practice
**Q1.** Which OLAP operation aggregates day data to month?
<details><summary>Answer</summary> Roll-up.</details>

**Q2.** Standardise x=14 when training mean=10 and SD=2.
<details><summary>Answer</summary> z-score = 2.</details>

**Q3.** Why fit scaling parameters inside each training fold?
<details><summary>Answer</summary> To prevent validation/test information leaking into the fitted model.</details>
