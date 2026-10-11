# Data warehousing and preprocessing

> **Paper:** DA · **Priority:** P1 · **Plan topics:** data types, transformations, normalization, discretization, sampling, data reduction, warehouse schemas (star, snowflake, fact constellation), OLAP operations, data cubes
> **Prerequisites:** [Relational model](relational-model-algebra-calculus.md) · [Constraints and normalization](constraints-and-normalization.md) · **Leads to:** [ML foundations](../16-machine-learning/ml-foundations.md), [Supervised learning](../16-machine-learning/classification-methods.md)
> **PYQ weightage:** Low · 3 Qs since 2015 (3 since 2021) · DA: 3 · Trend: rising · Most asked: Star vs Snowflake Schemas & Dimensional Modeling, OLAP Operations (Roll-up, Drill-down, Slice, Dice, Pivot), Feature Normalization (Min-Max, Z-Score), Discretization & Sampling

## Quick glance

- **Exam focus:** Dimensional schema design (star vs snowflake schemas, fact vs dimension tables), OLAP cube operations (roll-up vs drill-down, slice vs dice, pivot), feature scaling (min-max and z-score calculations), and discretization/binning; asked specifically in GATE DA as MCQ, MSQ, and NAT.
- **OLTP vs OLAP:** **OLTP** (Online Transaction Processing) manages real-time day-to-day transactions (highly normalized, 3NF/BCNF, updates/deletes, row-oriented). **OLAP** (Online Analytical Processing) manages historical decision support (denormalized, dimensional modeling, read-mostly, complex aggregations).
- **Dimensional Modeling:**
  - **Fact Table:** Contains quantitative numeric measures (e.g., revenue, units sold) and foreign keys referencing dimension tables.
  - **Dimension Tables:** Contain descriptive contextual textual attributes (e.g., customer name, product category, store location) organized into concept hierarchies.
- **Warehouse Schemas:**
  - **Star Schema:** Single central fact table connected directly to completely **denormalized** dimension tables. Minimizes join count for fast queries; introduces redundant dimension data.
  - **Snowflake Schema:** Dimension tables are **normalized** into sub-dimension hierarchies (e.g. Product $\to$ SubCategory $\to$ Category). Eliminates dimension redundancy; increases join cost.
  - **Fact Constellation (Galaxy Schema):** Multiple fact tables sharing shared/conformed dimension tables.
- **Core OLAP Operations on Data Cubes:**
  - **Roll-up (Drill-up):** Climbs up a concept hierarchy (e.g., Day $\to$ Month $\to$ Year) or drops a dimension; aggregates data to coarser granularity.
  - **Drill-down:** Descends down a concept hierarchy (e.g., Year $\to$ Quarter $\to$ Month) or adds a dimension; reveals finer granularity.
  - **Slice:** Fixes a single dimension to a specific value (e.g., `Time = '2025'`), reducing cube dimensionality by 1.
  - **Dice:** Selects a range or condition on **two or more dimensions**, yielding a smaller sub-cube.
  - **Pivot (Rotate):** Reorients the multidimensional presentation axes (e.g., swapping row and column views).
- **Attribute Types:**
  - **Nominal:** Categorical, no intrinsic order (Color, Gender, City).
  - **Ordinal:** Ranked order, but intervals are undefined (Rank, Grade, Satisfaction).
  - **Interval:** Numeric, equal intervals, **arbitrary zero point** (Temperature in °C/°F, Calendar years). Ratios are meaningless.
  - **Ratio:** Numeric, **absolute true zero** (Income, Age, Weight, Distance). Ratios are meaningful.
- **Feature Normalization:**
  - **Min-Max to $[0, 1]$:** $v' = \frac{v - \min}{\max - \min}$. Highly sensitive to outliers.
  - **Z-Score (Standardization):** $v' = \frac{v - \mu}{\sigma}$. Zero mean, unit variance. Robust to moderate outliers.
- **Data Leakage in Preprocessing:** Normalization parameters ($\min, \max, \mu, \sigma$), imputation statistics, and bin boundaries **must be computed strictly on the training set only**, and then applied to validation/test sets without re-estimation!

## 1. Data warehouse architecture and dimensional modeling [PYQ: Low]

A **data warehouse** is a subject-oriented, integrated, time-variant, and non-volatile collection of data organized for managerial decision-making.

```text
+------------------------+------------------------------------+------------------------------------+
| Feature                | OLTP (Operational Database)        | OLAP (Data Warehouse)              |
+------------------------+------------------------------------+------------------------------------+
| Primary Purpose        | Day-to-day transactional execution | Long-term decision support analysis|
| User Base              | Clerks, DBAs, frontline apps       | Business analysts, data scientists |
| Data Scope             | Current, real-time snapshot        | Multi-year historical archive      |
| Schema Design          | Highly normalized (3NF / BCNF)     | Denormalized (Star / Snowflake)    |
| Query Pattern          | Simple, high-throughput read/write | Complex, long-running aggregations |
| Database Operations    | Frequent INSERT, UPDATE, DELETE    | Periodic bulk load (ETL), SELECT   |
| Metric Priority        | Transaction response time & ACID   | Query throughput & analytical speed|
+------------------------+------------------------------------+------------------------------------+
```

### Fact and dimension tables

- **Fact Table:** The central table in dimensional modeling. Contains:
  - **Foreign keys:** Referencing each surrounding dimension table (together forming the composite key of the fact).
  - **Measures (Facts):** Quantitative numeric values representing business events.
    - *Additive measures:* Can be summed across all dimensions (e.g., `sales_amount`, `units_sold`).
    - *Semi-additive measures:* Can be summed across some dimensions but not others (e.g., `account_balance` can be summed across accounts, but not across time).
    - *Non-additive measures:* Cannot be summed across any dimension (e.g., `unit_price`, `profit_margin_percent`, `temperature`).
- **Dimension Table:** Contains descriptive contextual attributes that define the filtering, grouping, and slicing axes of queries (e.g., `Date`, `Product`, `Customer`, `Store`).

### Warehouse schemas

```text
Star Schema:                      Snowflake Schema:
  +------------+                    +------------+
  |  Customer  |                    |  Customer  |
  +-----+------+                    +-----+------+
        |                                 |
+-------+------+  +---------+       +-----+--------+  +-----------+  +------------+
|  Sales Fact  |--| Product |       |  Sales Fact  |--|  Product  |--| SubCategory|--| Category |
+-------+------+  +---------+       +-----+--------+  +-----------+  +------------+  +----------+
        |                                 |
  +-----+------+                    +-----+------+
  |    Time    |                    |    Time    |
  +------------+                    +------------+
(Denormalized dimensions)          (Normalized dimensions)
```

1. **Star Schema:**
   - Central fact table connected directly to radial, single-table dimensions.
   - Dimension tables are **deliberately denormalized** (contain redundant textual columns, e.g., category and subcategory in the same product table).
   - **Trade-off:** Faster query execution (fewer table joins) at the cost of higher storage redundancy.
2. **Snowflake Schema:**
   - Dimension tables are **normalized** into sub-dimension tables, splitting concept hierarchies into separate relational tables.
   - For example: `Product(product_id, name, subcat_id)` joins to `SubCategory(subcat_id, name, cat_id)`, which joins to `Category(cat_id, name)`.
   - **Trade-off:** Minimal data redundancy, easier dimension maintenance, but requires multiple foreign key joins, slowing down analytical OLAP queries.
3. **Fact Constellation (Galaxy Schema):**
   - Multiple fact tables share common dimensions (called **conformed dimensions**).
   - Example: A retail warehouse having both `Sales_Fact` and `Shipping_Fact`, both referencing shared `Time` and `Product` dimensions.

**Worked example 1 (Star vs Snowflake schema design and join complexity).**  
A retail store analyzes sales transactions across 3 dimensions: `Time`, `Store`, and `Product`.  
- `Product` hierarchy: `Product_ID -> SubCategory_ID -> Category_ID`.
- `Store` hierarchy: `Store_ID -> City_ID -> State_ID -> Country_ID`.
- `Time` hierarchy: `Date_ID -> Month -> Quarter -> Year`.

*Star Schema implementation:*
- 1 central fact table: `Sales_Fact(Date_ID, Store_ID, Product_ID, units_sold, revenue)`.
- 3 dimension tables: `Dim_Time`, `Dim_Store`, `Dim_Product`.
- Query to find total revenue by Category in 2025:  
  Joins required = `Sales_Fact` $\bowtie$ `Dim_Product` $\bowtie$ `Dim_Time` $\implies$ **2 table joins**.

*Snowflake Schema implementation:*
- `Dim_Product` normalized into 3 tables (`Product`, `SubCategory`, `Category`).
- `Dim_Store` normalized into 4 tables (`Store`, `City`, `State`, `Country`).
- `Dim_Time` normalized into 4 tables (`Date`, `Month`, `Quarter`, `Year`).
- Total tables = 1 fact table $+ 3 + 4 + 4 = 12$ tables.
- Query to find total revenue by Category in 2025:  
  Joins required = `Sales_Fact` $\bowtie$ `Product` $\bowtie$ `SubCategory` $\bowtie$ `Category` $\bowtie$ `Date` $\bowtie$ `Month` $\bowtie$ `Quarter` $\bowtie$ `Year` $\implies$ **7 table joins**.

## 2. OLAP operations and data cubes [PYQ: Medium]

A **data cube** represents multidimensional analytical data organized into dimensional axes and cell measures.
- **Cuboid:** A specific multidimensional view of data at a given abstraction level.
- **Base Cuboid:** The cuboid containing all dimensions at their lowest granularity ($n$-D).
- **Apex Cuboid:** The 0-D cuboid representing the single global aggregate over all dimensions (0-D summary).
- **Lattice of Cuboids:** The complete graph of all possible cuboids.
  - For $n$ dimensions with **no hierarchies**, the lattice contains exactly **$2^n$ cuboids**.
  - If dimension $i$ has a linear concept hierarchy with $L_i$ levels (e.g. Day $\to$ Month $\to$ Year has 3 levels, plus level 0 'all'), the total number of cuboids is:
    $$\text{Total Cuboids} = \prod_{i=1}^n (L_i + 1)$$

```text
Lattice of Cuboids (3 Dimensions: Time [T], Location [L], Product [P]):
                    [ all ] (0-D Apex cuboid)
                  /    |    \
             [ T ]   [ L ]   [ P ] (1-D cuboids)
            /   \   /     \ /   \
         [T,L]    [T,P]    [L,P]   (2-D cuboids)
            \       |       /
             \      |      /
               [ T, L, P ] (3-D Base cuboid)
```

### Core OLAP operations

1. **Roll-up (Drill-up / Aggregation):**
   - Summarizes data by **climbing up** a concept hierarchy or by **dropping** one or more dimensions.
   - Example: Aggregating daily sales data into monthly totals, or removing the `Store` dimension to view global sales by `Product` and `Time`.
2. **Drill-down (Roll-down):**
   - The inverse of roll-up. Uncovers finer detail by **stepping down** a concept hierarchy or by **adding** new dimensions.
   - Example: Expanding yearly sales data into quarterly and monthly breakdowns.
3. **Slice:**
   - Performs a selection on **one specific dimension** of the cube, producing a sub-cube with one fewer dimension ($n \to n-1$).
   - Example: Selecting `Time = '2025'` from a 3-D `(Time, Product, Store)` cube produces a 2-D slice of `(Product, Store)`.
4. **Dice:**
   - Defines a sub-cube by selecting values across **two or more dimensions** simultaneously.
   - Example: Selecting `(Time IN ('Q1', 'Q2')) AND (Location IN ('Delhi', 'Mumbai')) AND (Product = 'Laptop')`.
5. **Pivot (Rotate):**
   - Rotates the display axes to provide an alternative perspective on the multidimensional data (e.g., swapping rows and columns in a crosstab).

**Worked example 2 (Data cube lattice and operation tracing).**  
A retail data warehouse has 3 dimensions:
- `Time`: Concept hierarchy has 3 levels: `Day -> Month -> Year` ($L_1 = 3$).
- `Location`: Concept hierarchy has 2 levels: `City -> Country` ($L_2 = 2$).
- `Product`: Concept hierarchy has 2 levels: `Item -> Category` ($L_3 = 2$).

*Step 1: Compute total cuboids in the lattice.*  
Including the root `all` level for each dimension, each dimension offers $L_i + 1$ choices of aggregation:
$$\text{Total Cuboids} = (L_1 + 1) \times (L_2 + 1) \times (L_3 + 1) = (3 + 1) \times (2 + 1) \times (2 + 1) = 4 \times 3 \times 3 = 36 \text{ cuboids}$$

*Step 2: Classify operations on this cube.*
- A report currently shows `Total Sales by (Month, City, Item)`.
- If the report is modified to show `Total Sales by (Year, City, Item)`:  
  `Month` $\to$ `Year` ascends the `Time` hierarchy $\implies$ **Roll-up**.
- If the report is modified to show only records where `Year = 2025`:  
  Filtering on a single dimension value $\implies$ **Slice**.
- If the report is modified to filter `Year = 2025 AND Country = 'India'`:  
  Filtering across two dimensions $\implies$ **Dice**.
- If the report swaps rows (`Year`) and columns (`City`):  
  Reorienting axes $\implies$ **Pivot**.

## 3. Data types and handling missing values [PYQ: Low]

Data preprocessing is the foundational first step of machine learning and data science pipelines.

### Measurement scales of attributes

| Scale Type | Description | Mathematical Operations | Central Tendency | Example |
|---|---|---|---|---|
| **Nominal** | Categorical labels with no intrinsic ordering | $=, \neq$ | Mode | Gender, Blood group, Postal code, Color |
| **Ordinal** | Categorical values with a meaningful rank order, but intervals are not quantifiable | $=, \neq, <, >$ | Median, Mode | Academic grade (A, B, C), Customer rating (1 to 5 stars) |
| **Interval** | Numeric values with equal intervals, but **no true/absolute zero** (zero is arbitrary) | $+,-$ (ratios are meaningless) | Mean, Median, Mode | Temperature (°C, °F), Calendar dates (year 2000 vs 1000) |
| **Ratio** | Numeric values with an **absolute true zero point** (zero indicates absence of quantity) | $+,-,\times,\div$ (ratios are meaningful) | Geometric Mean, Harmonic Mean, Mean | Salary, Age, Weight, Distance, Kelvin temperature |

### Missing data mechanisms and handling

- **Missing Completely at Random (MCAR):** Probability of missingness is independent of both observed and unobserved data.
- **Missing at Random (MAR):** Probability of missingness depends on observed data, but not on the missing value itself (e.g., men are less likely to report depression score, but within gender, missingness is random).
- **Missing Not at Random (MNAR):** Probability of missingness depends on the value of the missing attribute itself (e.g., high-income earners refusing to disclose salary).

**Handling Strategies:**
1. **Deletion:**
   - *Listwise deletion (complete-case analysis):* Discards any row containing a missing value. Valid only under MCAR; discards substantial data.
   - *Pairwise deletion:* Uses all available data on an analysis-by-analysis basis.
2. **Imputation:**
   - *Mean / Median imputation:* Replaces missing numeric values with the column mean (if normal) or median (if skewed). Reduces feature variance.
   - *Mode imputation:* Replaces missing categorical values with the most frequent class.
   - *K-Nearest Neighbors (KNN) Imputation:* Replaces missing value with the distance-weighted average of the $k$ most similar complete records.
   - *Regression / MICE (Multivariate Imputation by Chained Equations):* Predicts missing attributes using regression models trained on complete features.

## 4. Feature transformation and normalization [PYQ: Medium]

Machine learning algorithms that calculate geometric distances (KNN, K-Means, SVM, PCA) or compute gradients (Gradient Descent) are heavily biased toward features with larger numerical ranges. Feature normalization scales attributes onto comparable numeric intervals.

### Normalization methods

1. **Min-Max Normalization:**
   Maps values linearly into a specified range $[new\_\min, new\_\max]$ (typically $[0, 1]$):
   $$v' = \frac{v - \min_A}{\max_A - \min_A} \cdot (new\_\max_A - new\_\min_A) + new\_\min_A$$
   For target range $[0, 1]$:
   $$v' = \frac{v - \min_A}{\max_A - \min_A}$$
   - *Advantage:* Preserves exact linear relationships among data points; bounds data strictly to $[0, 1]$.
   - *Disadvantage:* **Highly sensitive to outliers**. If an extreme outlier exists, it compresses inliers into a tiny sub-interval.
2. **Z-Score Standardization (Zero-Mean Normalization):**
   Centers values around mean $\mu_A$ and scales by standard deviation $\sigma_A$:
   $$v' = \frac{v - \mu_A}{\sigma_A}$$
   where $\mu_A = \frac{1}{N} \sum_{i=1}^N v_i$ and $\sigma_A = \sqrt{\frac{1}{N} \sum_{i=1}^N (v_i - \mu_A)^2}$.
   - Resulting transformed feature has **mean $\mu = 0$** and **standard deviation $\sigma = 1$**.
   - *Advantage:* Robust to moderate outliers; does not compress inliers into an artificial bounded range.
   - *Disadvantage:* Does not produce bounded values within $[0, 1]$.
3. **Decimal Scaling Normalization:**
   Normalizes by shifting the decimal point of values:
   $$v' = \frac{v}{10^j}$$
   where $j$ is the smallest integer such that $\max(\lvert v' \rvert) < 1$.

### The Data Leakage trap in preprocessing

> [!WARNING]
> When evaluating models using train-test splits or $K$-fold cross-validation:
> 1. Compute summary statistics ($\min, \max, \mu, \sigma$, imputation values) **exclusively on the training split**.
> 2. Apply these frozen training statistics to transform the validation and test sets.
> 3. **Never fit scalers on the full dataset before splitting!** Doing so leaks future distribution information into the training phase, invalidating test error estimates.

**Worked example 3 (Calculating Min-Max, Z-score, and Decimal scaling).**  
Given a dataset of feature $A$: $\{20, 30, 50, 70, 80\}$.
Compute the normalized value for $v = 30$ under:
1. Min-Max normalization to $[0, 1]$.
2. Z-Score standardization.
3. Decimal scaling.

*Step 1: Min-Max normalization.*
- $\min_A = 20$, $\max_A = 80$.
$$v' = \frac{30 - 20}{80 - 20} = \frac{10}{60} = \frac{1}{6} \approx 0.1667$$

*Step 2: Z-Score standardization.*
- Mean $\mu$:
$$\mu = \frac{20 + 30 + 50 + 70 + 80}{5} = \frac{250}{5} = 50$$
- Variance $\sigma^2$:
$$\sigma^2 = \frac{(20-50)^2 + (30-50)^2 + (50-50)^2 + (70-50)^2 + (80-50)^2}{5}$$
$$\sigma^2 = \frac{(-30)^2 + (-20)^2 + 0^2 + 20^2 + 30^2}{5} = \frac{900 + 400 + 0 + 400 + 900}{5} = \frac{2600}{5} = 520$$
$$\sigma = \sqrt{520} \approx 22.8035$$
- For $v = 30$:
$$v' = \frac{30 - 50}{\sqrt{520}} = \frac{-20}{22.8035} \approx -0.877$$

*Step 3: Decimal scaling.*
- $\max(\lvert A \rvert) = 80$. Smallest integer $j$ such that $80 / 10^j < 1$ is $j = 2$ ($80 / 100 = 0.8 < 1$).
- For $v = 30$:
$$v' = \frac{30}{10^2} = \frac{30}{100} = 0.3$$

## 5. Discretization, sampling, and data reduction [PYQ: Low]

### Discretization and binning

Discretization transforms continuous numeric attributes into discrete nominal or ordinal intervals (bins).
- **Equal-width binning (Distance-based):**
  Divides the range of attribute $A$ into $k$ intervals of equal size:
  $$\text{Width } w = \frac{\max_A - \min_A}{k}$$
  - Bin intervals: $[\min, \min + w], (\min + w, \min + 2w], \dots, (\min + (k-1)w, \max]$.
  - *Limitation:* If data contains skewed outliers, some bins become heavily populated while others remain empty.
- **Equal-frequency binning (Quantile-based):**
  Divides the sorted data into $k$ intervals such that each bin contains approximately the same number of data points ($N / k$).
  - Overcomes skewed distributions and spreads data evenly across bins.
- **Bin smoothing techniques:**
  - *Smoothing by bin means:* Replace each value in a bin with the mean of that bin.
  - *Smoothing by bin medians:* Replace each value with the bin median.
  - *Smoothing by bin boundaries:* Replace each value with the closer of the minimum or maximum boundary of that bin.

### Sampling methods

Sampling extracts a representative subset $S$ of size $s$ from a large population dataset $D$ of size $N$.
1. **Simple Random Sampling without Replacement (SRSWOR):** Every tuple has an equal probability of selection ($1/N$ for the first draw). Once selected, it cannot be chosen again.
2. **Simple Random Sampling with Replacement (SRSWR):** A tuple is returned to the population after being selected; can appear multiple times in the sample.
3. **Stratified Sampling:** The population is partitioned into mutually disjoint subgroups called **strata** (e.g. by age group or class label). Random samples are drawn independently from each stratum proportional to its population share:
   $$s_i = s \cdot \frac{N_i}{N}$$
   Guarantees minority classes are represented adequately in classification models.
4. **Reservoir Sampling:** Algorithm to draw $k$ items uniformly at random from a data stream of unknown, potentially infinite length $N$. Stores the first $k$ items; for each subsequent item $i > k$, replaces a random item in the reservoir with probability $k / i$.

**Worked example 4 (Equal-width vs Equal-frequency binning).**  
Given 12 sorted data points:  
$$A = [4, 8, 9, 15, 21, 21, 24, 25, 26, 28, 29, 34]$$  
Partition into $k = 3$ bins.

*Method 1: Equal-width binning ($k = 3$).*
- $\min = 4, \max = 34 \implies \text{Width } w = \frac{34 - 4}{3} = \frac{30}{3} = 10$.
- Bin intervals:
  - Bin 1: $[4, 14] \implies \{4, 8, 9\}$ (3 items)
  - Bin 2: $(14, 24] \implies \{15, 21, 21, 24\}$ (4 items)
  - Bin 3: $(24, 34] \implies \{25, 26, 28, 29, 34\}$ (5 items)

*Method 2: Equal-frequency binning ($k = 3$).*
- Frequency per bin: $N / k = 12 / 3 = 4$ items per bin.
  - Bin 1: $\{4, 8, 9, 15\}$ (4 items)
  - Bin 2: $\{21, 21, 24, 25\}$ (4 items)
  - Bin 3: $\{26, 28, 29, 34\}$ (4 items)

*Smoothing Bin 1 of equal-frequency by bin means:*
$$\text{Mean} = \frac{4 + 8 + 9 + 15}{4} = \frac{36}{4} = 9 \implies \text{Smoothed Bin 1} = \{9, 9, 9, 9\}$$

## Formulas and facts to memorise

| Item | Formula / Definition | When to use |
|---|---|---|
| Min-Max Normalization | $v' = \frac{v - \min}{\max - \min} \cdot (new\_\max - new\_\min) + new\_\min$ | Scaling numeric features to $[0, 1]$ or custom range |
| Z-Score Standardization | $v' = \frac{v - \mu}{\sigma}$ | Standardizing features to zero mean, unit variance |
| Decimal Scaling | $v' = \frac{v}{10^j}$ where $\max(\lvert v' \rvert) < 1$ | Fast scaling without computing variance |
| Equal-width bin width | $w = \frac{\max - \min}{k}$ | Partitioning data into $k$ equal-interval bins |
| Equal-frequency count | $\frac{N}{k}$ items per bin | Partitioning data into $k$ equal-occupancy bins |
| Cuboid count (no hierarchy) | $2^n$ | Total cuboids for $n$ dimensions without hierarchies |
| Cuboid count (with hierarchy) | $\prod_{i=1}^n (L_i + 1)$ | Total cuboids for $n$ dimensions with $L_i$ hierarchy levels |
| Stratified sample size | $s_i = s \cdot \frac{N_i}{N}$ | Drawing proportional samples across strata |

## GATE traps

- **Relational Normalization vs Feature Normalization:** Relational normal forms (1NF, 2NF, 3NF, BCNF) eliminate relational update anomalies by decomposing schemas; feature normalization (min-max, z-score) rescales continuous numeric feature values for ML algorithms. Do not confuse them in written questions!
- **Roll-up vs Drill-down direction:** Roll-up *climbs up* concept hierarchies (increases aggregation, decreases detail); drill-down *steps down* concept hierarchies (decreases aggregation, increases detail).
- **Slice vs Dice:** Slice fixes **exactly one** dimension to a single value; Dice filters across **two or more** dimensions simultaneously.
- **Interval vs Ratio zero point:** 0°C does not mean "no heat" (it is an arbitrary freezing point on an interval scale). 0 kg means "no mass" (absolute zero on a ratio scale). Ratios are valid *only* on ratio scales!
- **Data leakage in Min-Max:** Computing $\min$ and $\max$ on the test set (or combined train+test set) causes data leakage. Normalization must fit *only* on training data.
- **Star vs Snowflake joins:** Star schema has *fewer joins* than snowflake schema because dimension tables in star schemas are denormalized into single tables.

## Connections

- [Constraints and normalization](constraints-and-normalization.md) — Relational schema normalization decomposes operational databases into 3NF/BCNF; data warehouses intentionally reverse this into denormalized dimensional star schemas.
- [ML foundations](../16-machine-learning/ml-foundations.md) — Feature scaling and data leakage prevention are mandatory prerequisites for training machine learning estimators.
- [Probability and statistics](../02-probability-statistics/README.md) — Summary statistics (mean, variance, standard deviation) and sampling theories directly underpin z-score and stratified sampling.

## Practice

**Q1 (MCQ).** In a multidimensional data warehouse, which OLAP operation transforms the view of sales data from `Sales by City and Quarter` to `Sales by Country and Quarter`?  
(A) Drill-down  
(B) Roll-up  
(C) Slice  
(D) Pivot  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
Moving from `City` to `Country` ascends the geographic concept hierarchy (`City -> State -> Country`).  
Ascending a concept hierarchy to view data at a coarser, more aggregated granularity is a **Roll-up** operation.  
Option (B) is correct.
</details>

**Q2 (NAT).** Given a numeric attribute $A$ with training values $\{10, 20, 30, 40, 50\}$. What is the min-max normalized value of $x = 35$ scaled to the range $[0, 1]$?  
<details><summary>Answer</summary>

**Answer:** 0.625  
**Solution:**  
From the given training set:
- $\min_A = 10$
- $\max_A = 50$
Apply the min-max formula to $[0, 1]$:
$$v' = \frac{x - \min_A}{\max_A - \min_A} = \frac{35 - 10}{50 - 10} = \frac{25}{40} = \frac{5}{8} = 0.625$$
The normalized value is 0.625.
</details>

**Q3 (MCQ).** Consider the temperature in degrees Celsius ($^\circ\text{C}$) and student ID numbers. Under Stevens' typology of measurement scales, what are the attribute types of temperature and student ID, respectively?  
(A) Ratio and Nominal  
(B) Interval and Nominal  
(C) Interval and Ordinal  
(D) Ratio and Ordinal  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
- Temperature in $^\circ\text{C}$ has equal intervals, but $0^\circ\text{C}$ is an arbitrary reference point (not the absence of heat), so it is an **Interval** scale.
- Student ID numbers are unique identifying labels with no mathematical ordering or arithmetic meaning, so they are **Nominal** attributes.  
Thus, option (B) is correct.
</details>

**Q4 (MSQ).** Which of the following statements is/are TRUE regarding dimensional modeling and data warehousing?  
(A) In a star schema, dimension tables are normalized to 3NF.  
(B) A snowflake schema reduces data redundancy compared to a star schema by normalizing dimension tables.  
(C) A slice operation performs a selection on one dimension of a data cube.  
(D) Fact tables primarily contain descriptive textual attributes.  
<details><summary>Answer</summary>

**Answer:** (B), (C)  
**Solution:**  
- (A) is FALSE: star schemas keep dimension tables denormalized for faster query joins.
- (B) is TRUE: snowflake schemas normalize dimension tables into sub-dimensions, reducing redundancy.
- (C) is TRUE: a slice operation fixes one dimension to a single value.
- (D) is FALSE: fact tables contain quantitative numerical measures and foreign keys; descriptive textual attributes reside in dimension tables.  
Thus, (B) and (C) are TRUE.
</details>

**Q5 (NAT).** A data warehouse has 4 dimensions without concept hierarchies. How many total cuboids exist in its lattice of cuboids?  
<details><summary>Answer</summary>

**Answer:** 16  
**Solution:**  
For $n$ dimensions with no concept hierarchies, each dimension can either be present or absent (aggregated to 'all') in a cuboid.  
$$\text{Total Cuboids} = 2^n = 2^4 = 16$$
These include one 0-D apex cuboid, four 1-D cuboids, six 2-D cuboids, four 3-D cuboids, and one 4-D base cuboid ($1 + 4 + 6 + 4 + 1 = 16$).
</details>

**Q6 (MCQ).** You are training a machine learning model using 5-fold cross-validation. Which of the following procedures correctly avoids data leakage when standardizing features with z-score normalization?  
(A) Standardize the entire dataset using its overall mean and standard deviation, then partition into folds.  
(B) For each fold, compute the mean and standard deviation on the 4 training folds, and use them to standardize both the 4 training folds and the 1 validation fold.  
(C) Compute the mean and standard deviation independently on each fold (training folds on training data, validation fold on validation data).  
(D) Standardize only the validation fold; leave the training folds unstandardized.  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
To prevent information from the validation fold leaking into the model:
- Mean $\mu$ and standard deviation $\sigma$ must be computed **strictly on the training folds only**.
- The validation fold must be transformed using the frozen training statistics $\mu_{\text{train}}$ and $\sigma_{\text{train}}$.  
Option (A) leaks validation distribution; option (C) uses validation statistics during validation evaluation.  
Thus, (B) is the correct procedure.
</details>

**Q7 (NAT).** A dataset of 10 values has mean $\mu = 40$ and standard deviation $\sigma = 5$. If an observation has value $x = 52$, its z-score standardized value is _______.  
<details><summary>Answer</summary>

**Answer:** 2.4  
**Solution:**  
Using the z-score formula:
$$z = \frac{x - \mu}{\sigma} = \frac{52 - 40}{5} = \frac{12}{5} = 2.4$$
The standardized value is 2.4.
</details>
