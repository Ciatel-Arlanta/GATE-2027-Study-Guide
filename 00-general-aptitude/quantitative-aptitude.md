# Quantitative Aptitude: Arithmetic, Algebra, Geometry, and Data Interpretation

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** ratios, percentages, powers and logarithms, counting, series, mensuration, data interpretation, elementary statistics
> **Prerequisites:** none · **Leads to:** [Probability basics](../02-probability-statistics/probability-basics.md), [Combinatorics](../01-discrete-mathematics/combinatorics.md)
> **PYQ weightage:** High · 104 Qs since 2015 (60 since 2021) · DA: 18 · Trend: rising · Most asked: Probability & Counting, Data Interpretation (Bar/Pie/Tables), Ratios & Percentages, Geometry & Mensuration

## Quick glance

- **Exam focus:** Data interpretation (bar/pie/tables) and counting/probability appear in virtually every GATE paper (CS and DA), usually as 2-mark MCQs or NATs; commercial arithmetic (percentages, ratios) and algebra/geometry supply steady 1-mark questions.
- **Percentage multiplier:** A change of $+p\%$ corresponds to multiplying by $(1 + p/100)$; a decrease of $-p\%$ multiplies by $(1 - p/100)$. Successive changes $a\%$ and $b\%$ yield a net multiplier $(1 + a/100)(1 + b/100)$ or net change $(a + b + \frac{ab}{100})\%$.
- **Ratios and totals:** For ratio $a:b:c$ with total $T$, write parts as $ax, bx, cx$ where $x = \frac{T}{a+b+c}$.
- **Average speed for equal distances:** Not arithmetic mean: $v_{\text{avg}} = \frac{2v_1 v_2}{v_1 + v_2}$ (harmonic mean). For equal times: $v_{\text{avg}} = \frac{v_1 + v_2}{2}$.
- **Time and work:** If A does work in $d_A$ days, rate is $r_A = 1/d_A$ work/day. Combined rate $r = \sum r_i$; total time $= 1/r$. Emptying pipes contribute negative rates.
- **Logarithms:** $\log_b(xy) = \log_b x + \log_b y$, $\log_b(x/y) = \log_b x - \log_b y$, $\log_b(x^k) = k\log_b x$, and change of base $\log_b a = \frac{\log_c a}{\log_c b}$. Valid only for positive arguments and base $> 0, \ne 1$.
- **Counting:** Ordered arrangements of $r$ from $n$ distinct objects: $P(n,r) = \frac{n!}{(n-r)!}$. Unordered selections: $\binom{n}{r} = \frac{n!}{r!(n-r)!}$.
- **Geometry similarity:** When linear dimensions scale by factor $k$, perimeter scales by $k$, area scales by $k^2$, and volume scales by $k^3$.
- **Pie chart angles:** $100\% \equiv 360^\circ$; hence $1\% = 3.6^\circ$ and $1^\circ = \frac{5}{18}\%$.
- **#1 trap:** Applying a percentage increase followed by the same percentage decrease never restores the baseline: $(1 + x)(1 - x) = 1 - x^2 < 1$.

## 1. Ratios, Percentages, and Commercial Arithmetic [PYQ: High]

A **ratio** expresses the relative magnitude of quantities in identical units. If two quantities are in ratio $a:b$, express them as $a x$ and $b x$. An **alligation** or weighted mixture of two components with values $c_1$ and $c_2$ giving average $c_{\text{mix}}$ satisfies:

$$\frac{q_1}{q_2} = \frac{c_2 - c_{\text{mix}}}{c_{\text{mix}} - c_1}$$

**Percentage change** is strictly relative to the **initial (base) value**:

$$\text{Percentage change} = \frac{\text{New} - \text{Base}}{\text{Base}} \times 100\%$$

When price increases by $P\%$, consumption must decrease by $\left(\frac{P}{100 + P} \times 100\right)\%$ to keep expenditure fixed.

**Commercial arithmetic relationships:**
- **Cost Price ($CP$)**, **Selling Price ($SP$)**, **Marked Price ($MP$)**.
- $\text{Profit} = SP - CP$; $\text{Profit } \% = \frac{SP - CP}{CP} \times 100\%$.
- $\text{Discount} = MP - SP$; $\text{Discount } \% = \frac{MP - SP}{MP} \times 100\%$.
- **Simple Interest ($SI$):** $SI = \frac{P \cdot R \cdot T}{100}$, with total amount $A = P + SI$.
- **Compound Interest ($CI$):** Compounded $n$ times per year for $t$ years: $A = P\left(1 + \frac{R}{100n}\right)^{nt}$.

**Worked example 1 (Mark-up, discount, and profit).** A retailer marks goods 40% above cost price and offers two successive discounts of 10% and 20% on the marked price. If the final selling price is Rs. 1008, find (i) the cost price, and (ii) the net profit percentage.

*Step 1: Set up the multipliers.*
Let cost price be $C$.
$$\text{Marked Price } MP = C \times (1 + 0.40) = 1.40 C$$
$$\text{Selling Price } SP = MP \times (1 - 0.10) \times (1 - 0.20) = 1.40 C \times 0.90 \times 0.80$$

*Step 2: Calculate effective multiplier.*
$$0.90 \times 0.80 = 0.72$$
$$SP = 1.40 \times 0.72 \times C = 1.008 C$$

*Step 3: Solve for cost price.*
$$1.008 C = 1008 \implies C = \frac{1008}{1.008} = 1000\text{ Rs.}$$

*Step 4: Net profit percentage.*
$$\text{Net profit } \% = \frac{1008 - 1000}{1000} \times 100\% = \frac{8}{1000} \times 100\% = 0.8\%$$

## 2. Rates, Time, Speed, Distance, and Work [PYQ: Medium]

The fundamental rate equation is $\text{Quantity} = \text{Rate} \times \text{Time}$.

### 2.1 Time, Speed, and Distance
- **Conversion factor:** $1\text{ km/h} = \frac{5}{18}\text{ m/s}$; $1\text{ m/s} = \frac{18}{5}\text{ km/h}$.
- **Relative speed:**
  - Objects moving in **opposite directions** at speeds $u$ and $v$: relative speed $= u + v$.
  - Objects moving in the **same direction** at speeds $u > v$: relative speed $= u - v$.
- **Trains:**
  - Time to pass a point pole/man of negligible width: $t = \frac{L_{\text{train}}}{u}$.
  - Time to pass a platform/bridge of length $L_p$: $t = \frac{L_{\text{train}} + L_p}{u}$.
- **Average speed:** Always $\frac{\text{Total Distance}}{\text{Total Time}}$. If distance $D$ is covered at $v_1$ and return distance $D$ at $v_2$, total time $t = \frac{D}{v_1} + \frac{D}{v_2} = D\frac{v_1 + v_2}{v_1 v_2}$. Hence $v_{\text{avg}} = \frac{2D}{t} = \frac{2v_1 v_2}{v_1 + v_2}$.

### 2.2 Time and Work
If an entity completes a task in $T$ units of time, its working rate is $\frac{1}{T}$ task per unit time.
- If $A$ takes $t_A$ days and $B$ takes $t_B$ days, together their rate is $\frac{1}{t_A} + \frac{1}{t_B}$, completing the work in $\frac{t_A t_B}{t_A + t_B}$ days.
- **Pipes and cisterns:** Inlet pipes contribute positive rates ($+1/t_i$); drain pipes contribute negative rates ($-1/t_d$).

**Worked example 2 (Pipes and cisterns with drainage).** Inlet pipe A fills a reservoir in 12 hours, inlet pipe B fills it in 15 hours, and drain pipe C empties a full reservoir in 20 hours. All three pipes are opened simultaneously into an empty reservoir. How many hours will it take to fill completely?

*Step 1: Determine hourly rates.*
- Rate of A: $r_A = +1/12$ reservoir/hour.
- Rate of B: $r_B = +1/15$ reservoir/hour.
- Rate of C: $r_C = -1/20$ reservoir/hour.

*Step 2: Sum the rates using common denominator.*
$$\text{Net rate } r = \frac{1}{12} + \frac{1}{15} - \frac{1}{20}$$
LCM of $12, 15, 20$ is 60:
$$r = \frac{5}{60} + \frac{4}{60} - \frac{3}{60} = \frac{5 + 4 - 3}{60} = \frac{6}{60} = \frac{1}{10}\text{ reservoir/hour}$$

*Step 3: Compute time to fill.*
$$\text{Time } T = \frac{1}{r} = \frac{1}{1/10} = 10\text{ hours}$$

## 3. Number Properties, Exponents, Logarithms, and Progressions [PYQ: High]

### 3.1 Integers and Modular Arithmetic
- **Fundamental Theorem of Arithmetic:** Every integer $n > 1$ factors uniquely as $n = p_1^{a_1} p_2^{a_2} \cdots p_k^{a_k}$.
- **Divisors count:** $\tau(n) = (a_1 + 1)(a_2 + 1)\cdots(a_k + 1)$.
- **Divisibility checks:** By 3 and 9: sum of digits; by 4: last 2 digits; by 8: last 3 digits; by 11: alternating sum of digits $\equiv 0 \pmod{11}$.
- **Cyclicity of unit digits:** Powers of integers modulo 10 repeat with period at most 4:
  - 2: $2, 4, 8, 6$ (cycle 4)
  - 3: $3, 9, 7, 1$ (cycle 4)
  - 7: $7, 9, 3, 1$ (cycle 4)
  - 8: $8, 4, 2, 6$ (cycle 4)
  - 4: $4, 6$ (cycle 2); 9: $9, 1$ (cycle 2); 0, 1, 5, 6: cycle 1.

### 3.2 Progressions
- **Arithmetic Progression (AP):** $n$-th term $a_n = a + (n-1)d$; Sum $S_n = \frac{n}{2}[2a + (n-1)d] = \frac{n}{2}(a + a_n)$.
- **Geometric Progression (GP):** $n$-th term $a_n = a \cdot r^{n-1}$; Sum $S_n = \frac{a(1 - r^n)}{1 - r}$ ($r \ne 1$).
  - For infinite GP with $|r| < 1$: $S_\infty = \frac{a}{1 - r}$.

**Worked example 3 (Logarithmic equations).** Find the positive value of $x$ satisfying:
$$\log_2 x + \log_4 x + \log_{16} x = 21$$

*Step 1: Change all logarithms to common base 2.*
Using $\log_{b^k} x = \frac{1}{k}\log_b x$:
$$\log_4 x = \log_{2^2} x = \frac{1}{2}\log_2 x$$
$$\log_{16} x = \log_{2^4} x = \frac{1}{4}\log_2 x$$

*Step 2: Factor out $\log_2 x$.*
$$\log_2 x \left(1 + \frac{1}{2} + \frac{1}{4}\right) = 21$$
$$\log_2 x \left(\frac{4 + 2 + 1}{4}\right) = 21 \implies \log_2 x \cdot \frac{7}{4} = 21$$

*Step 3: Solve for $\log_2 x$ and $x$.*
$$\log_2 x = 21 \times \frac{4}{7} = 3 \times 4 = 12$$
$$x = 2^{12} = 4096$$

## 4. Counting Principles, Permutations, Combinations, and Elementary Probability [PYQ: High]

### 4.1 Counting Principles
- **Multiplication Rule:** If step 1 has $m$ outcomes and step 2 has $n$ outcomes, joint procedure has $m \cdot n$ outcomes.
- **Permutations:** Arranging $r$ objects out of $n$ distinct objects: $P(n,r) = \frac{n!}{(n-r)!}$.
- **Combinations:** Selecting an unordered subset of $r$ objects out of $n$: $\binom{n}{r} = \frac{n!}{r!(n-r)!}$.
- **Arrangements with repeated items:** Total permutations of $n$ items where item $i$ repeats $k_i$ times: $\frac{n!}{k_1! k_2! \cdots k_m!}$.
- **Circular arrangements:** Distinct cyclic arrangements of $n$ distinct items: $(n-1)!$.

### 4.2 Elementary Probability
- Classical probability on sample space $S$: $P(E) = \frac{|E|}{|S|}$ for equally likely outcomes.
- **Complement rule:** $P(E) = 1 - P(E^c)$. When calculating "at least one", always evaluate $1 - P(\text{none})$.
- **Addition rule:** $P(A \cup B) = P(A) + P(B) - P(A \cap B)$. For mutually exclusive events ($A \cap B = \emptyset$), $P(A \cup B) = P(A) + P(B)$.
- **Independent events:** $P(A \cap B) = P(A) \cdot P(B)$.

**Worked example 4 (Constrained selection and probability).** An urn contains 5 red balls, 4 blue balls, and 3 yellow balls (total 12 balls). Three balls are drawn at random without replacement.
1. Find the probability that all 3 balls have different colours.
2. Find the probability that at least 2 balls are red.

*Step 1: Total possible outcomes.*
$$|S| = \binom{12}{3} = \frac{12 \times 11 \times 10}{3 \times 2 \times 1} = 220$$

*Step 2: Favourable outcomes for distinct colours.*
Pick 1 red, 1 blue, and 1 yellow:
$$|E_1| = \binom{5}{1} \times \binom{4}{1} \times \binom{3}{1} = 5 \times 4 \times 3 = 60$$
$$P(\text{all different}) = \frac{60}{220} = \frac{3}{11}$$

*Step 3: Favourable outcomes for at least 2 red.*
Partition into mutually exclusive cases:
- **Case A (Exactly 2 red, 1 non-red):** Non-red balls $= 4 + 3 = 7$.
  $$\binom{5}{2} \times \binom{7}{1} = 10 \times 7 = 70$$
- **Case B (Exactly 3 red):**
  $$\binom{5}{3} = 10$$
Total favourable $= 70 + 10 = 80$.
$$P(\ge 2\text{ red}) = \frac{80}{220} = \frac{4}{11}$$

## 5. Geometry, Mensuration, and Coordinate Geometry [PYQ: High]

### 5.1 2D Geometry & Mensuration
- **Triangles:**
  - Area $= \frac{1}{2} b h = \sqrt{s(s-a)(s-b)(s-c)}$ where semi-perimeter $s = \frac{a+b+c}{2}$ (Heron's formula).
  - Inradius $r = \frac{\Delta}{s}$; for right-angled triangle with legs $a, b$ and hypotenuse $c$, $r = \frac{a + b - c}{2}$.
  - Circumradius $R = \frac{abc}{4\Delta}$; for right-angled triangle, $R = \frac{c}{2}$.
- **Circles:** Area $= \pi r^2$, circumference $= 2\pi r$. Sector area $= \frac{\theta}{360^\circ}\pi r^2$; arc length $= \frac{\theta}{360^\circ}2\pi r$.

### 5.2 3D Mensuration
| Solid | Curved / Lateral Surface Area | Total Surface Area | Volume |
|---|---|---|---|
| Cube (side $a$) | $4a^2$ | $6a^2$ | $a^3$ |
| Cuboid ($l, w, h$) | $2h(l + w)$ | $2(lw + wh + hl)$ | $l \cdot w \cdot h$ |
| Cylinder (radius $r$, height $h$) | $2\pi r h$ | $2\pi r(r + h)$ | $\pi r^2 h$ |
| Cone (radius $r$, height $h$, slant $l=\sqrt{r^2+h^2}$) | $\pi r l$ | $\pi r(r + l)$ | $\frac{1}{3}\pi r^2 h$ |
| Sphere (radius $r$) | $4\pi r^2$ | $4\pi r^2$ | $\frac{4}{3}\pi r^3$ |
| Hemisphere (radius $r$) | $2\pi r^2$ | $3\pi r^2$ | $\frac{2}{3}\pi r^3$ |

### 5.3 Coordinate Geometry
- Distance between $(x_1, y_1)$ and $(x_2, y_2)$: $d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$.
- Slope of line joining points: $m = \frac{y_2 - y_1}{x_2 - x_1}$. Two lines with slopes $m_1, m_2$ are perpendicular iff $m_1 m_2 = -1$.
- Area of triangle with vertices $(x_1,y_1), (x_2,y_2), (x_3,y_3)$: $\frac{1}{2}|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)|$.

**Worked example 5 (Mensuration volume conservation).** A solid metallic cylinder of radius 6 cm and height 12 cm is melted down and recast into identical solid spheres, each of radius 3 cm. Assuming no loss of metal, how many complete spheres are formed?

*Step 1: Compute cylinder volume.*
$$V_{\text{cyl}} = \pi r^2 h = \pi (6)^2 (12) = \pi \times 36 \times 12 = 432\pi\text{ cm}^3$$

*Step 2: Compute volume of one sphere.*
$$V_{\text{sph}} = \frac{4}{3}\pi R^3 = \frac{4}{3}\pi (3)^3 = \frac{4}{3}\pi (27) = 36\pi\text{ cm}^3$$

*Step 3: Calculate number of spheres.*
$$N = \frac{V_{\text{cyl}}}{V_{\text{sph}}} = \frac{432\pi}{36\pi} = 12\text{ spheres}$$

## 6. Data Interpretation and Elementary Statistics [PYQ: High]

Data Interpretation in GATE presents quantitative information in tables, bar charts, pie charts, or line graphs, requiring rapid multi-step calculations.

```text
       ┌────────────────────────────────────────────────────────┐
       │             Distribution of Employees (N = 600)        │
       │                                                        │
       │   Dept        Total Employees     Male : Female Ratio  │
       │   ───────────────────────────────────────────────────  │
       │   R&D               150                   3 : 2        │
       │   Sales             200                   1 : 1        │
       │   Marketing         120                   1 : 3        │
       │   HR                130                   2 : 3        │
       └────────────────────────────────────────────────────────┘
```

### 6.1 Principles of Chart Analysis
- **Angle to percentage conversion:** In a pie chart, an arc of angle $\theta^\circ$ corresponds to $\frac{\theta}{360} \times 100\%$.
- **Growth rate vs. absolute increase:**
  - Absolute increase $= \text{Value}_{t} - \text{Value}_{t-1}$.
  - Percentage growth rate $= \frac{\text{Value}_{t} - \text{Value}_{t-1}}{\text{Value}_{t-1}} \times 100\%$.

### 6.2 Elementary Statistics
- **Mean:** $\bar{x} = \frac{\sum f_i x_i}{\sum f_i}$.
- **Median:** Sort $n$ elements in non-decreasing order:
  - If $n$ is odd, median is the element at rank $\frac{n+1}{2}$.
  - If $n$ is even, median is the average of elements at ranks $\frac{n}{2}$ and $\frac{n}{2} + 1$.
- **Mode:** Value appearing with greatest frequency.
- **Variance and Standard Deviation:** $\sigma^2 = \frac{1}{n}\sum_{i=1}^n (x_i - \bar{x})^2$, $\sigma = \sqrt{\sigma^2}$.

**Worked example 6 (Data Interpretation with ratios and percentages).** Using the employee table above:
1. Find the total number of female employees in the organization.
2. What percentage of the total male workforce belongs to the R&D department?

*Step 1: Compute male and female headcounts by department.*
- **R&D (150 total, ratio 3:2):**
  - Male $= 150 \times \frac{3}{5} = 90$
  - Female $= 150 \times \frac{2}{5} = 60$
- **Sales (200 total, ratio 1:1):**
  - Male $= 200 \times \frac{1}{2} = 100$
  - Female $= 200 \times \frac{1}{2} = 100$
- **Marketing (120 total, ratio 1:3):**
  - Male $= 120 \times \frac{1}{4} = 30$
  - Female $= 120 \times \frac{3}{4} = 90$
- **HR (130 total, ratio 2:3):**
  - Male $= 130 \times \frac{2}{5} = 52$
  - Female $= 130 \times \frac{3}{5} = 78$

*Step 2: Total female employees.*
$$\text{Total females} = 60 + 100 + 90 + 78 = 328$$

*Step 3: Total male employees and R&D percentage.*
$$\text{Total males} = 90 + 100 + 30 + 52 = 272$$
$$\text{R\&D males percentage} = \frac{90}{272} \times 100\% \approx 33.09\%$$

## Formulas and facts to memorise

| Concept / Quantity | Formula / Rule | When to use in GATE |
|---|---|---|
| Successive changes | Net $\Delta\% = a + b + \frac{ab}{100}$ | Two successive percentage shifts |
| Expenditure invariance | Consumption drop $= \frac{P}{100 + P} \times 100\%$ | Price hikes with constant budget |
| Average speed (equal dist) | $v_{\text{avg}} = \frac{2v_1 v_2}{v_1 + v_2}$ | Two-way journey over identical length |
| Work rate sum | $\text{Rate}_{\text{total}} = \sum \frac{1}{T_i}$ | Workers or pipes acting concurrently |
| Divisors count | $\tau(n) = \prod (a_i + 1)$ for $n = \prod p_i^{a_i}$ | Number of positive integer factors |
| Unit digit cyclicity | Check power $\pmod 4$ | Unit digit of large integer powers |
| Sum of AP | $S_n = \frac{n}{2}(a + l) = \frac{n}{2}[2a + (n-1)d]$ | Uniform step sums |
| Infinite GP sum | $S_\infty = \frac{a}{1 - r}$ ($\vert r\vert < 1$) | Repeated fractional decrements |
| Right triangle inradius | $r = \frac{a + b - c}{2}$ | Incircle inside right-angled triangle |
| Sector area | $A = \frac{\theta}{360^\circ} \pi r^2$ | Circular sectors and pie chart regions |
| Pie chart conversion | $1\% = 3.6^\circ \iff 1^\circ = \frac{5}{18}\%$ | Translating chart angles to percentages |
| Median of $n$ elements | $\text{Rank } \frac{n+1}{2}$ (odd) or avg of $\frac{n}{2}, \frac{n}{2}+1$ (even) | Middle value of ordered samples |

## GATE traps

- **Asymmetric percentage base:** A $20\%$ drop followed by a $20\%$ increase does not return to $100$: $1.0 \times 0.8 \times 1.2 = 0.96$ (a net $4\%$ drop). Always anchor percentages to the immediately preceding base.
- **Arithmetic mean trap in speeds:** Never average speed rates directly unless the time durations are identical. If distances are identical, use the harmonic mean.
- **At least one vs at most one:** "At least one" is $1 - P(\text{none})$. "At most one" is $P(0) + P(1)$. Do not confuse the two complementary events.
- **Truncated chart axes:** Bar charts with non-zero baselines visually exaggerate differences. Read explicit numerical axis values instead of measuring graphical bar heights.
- **Arrangements vs selections:** Check whether order matters. If roles are distinct (president, secretary) or seats are numbered, use permutations $P(n,r)$. If forming an unranked team or choosing a subset, use combinations $\binom{n}{r}$.
- **Right triangle hypotenuse:** Inradius $r = \frac{a+b-c}{2}$ applies only when $c$ is the hypotenuse of a right-angled triangle, not for arbitrary scalene triangles.

## Connections

- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — advanced counting, pigeonhole principle, and generating functions.
- [Probability basics](../02-probability-statistics/probability-basics.md) — formal axioms, conditional probability, Bayes theorem, and random variables.
- [Verbal aptitude](verbal-aptitude.md) — interpreting precise English wording in word problems and charts.
- [Analytical aptitude](analytical-aptitude.md) — numerical reasoning, pattern deduction, and constraint solving.
- [Spatial aptitude](spatial-aptitude.md) — 2D and 3D geometric transformations and coordinate projections.

## Practice

**Q1 (MCQ).** If the price of crude oil increases by 25%, by what percentage must a factory reduce its consumption so that its total expenditure on oil remains unchanged?  
(A) 20%  
(B) 25%  
(C) 16.67%  
(D) 33.33%  

<details><summary>Answer</summary>

**Answer:** A (20%)  
**Solution:**  
Let initial price be $P$ and initial consumption be $C$. Total expenditure is $E = P \cdot C$.  
New price is $P' = 1.25 P$.  
For expenditure to remain unchanged:  
$$P' \cdot C' = E \implies 1.25 P \cdot C' = P \cdot C \implies C' = \frac{C}{1.25} = 0.80 C$$  
Reduction in consumption $= C - 0.80 C = 0.20 C$, which is a **20%** reduction.  
Alternatively, using the formula: $\frac{P}{100 + P} \times 100\% = \frac{25}{125} \times 100\% = 20\%$.

</details>

**Q2 (NAT).** A commuter drives 60 km to work at an average speed of 30 km/h, and returns along the exact same route at an average speed of 60 km/h. What is the average speed for the entire round trip in km/h?

<details><summary>Answer</summary>

**Answer:** 40  
**Solution:**  
Total distance covered $= 60 + 60 = 120\text{ km}$.  
Time for forward journey $= \frac{60}{30} = 2\text{ hours}$.  
Time for return journey $= \frac{60}{60} = 1\text{ hour}$.  
Total time taken $= 2 + 1 = 3\text{ hours}$.  
$$\text{Average speed} = \frac{\text{Total distance}}{\text{Total time}} = \frac{120}{3} = 40\text{ km/h}$$  
Notice this equals the harmonic mean: $\frac{2 \times 30 \times 60}{30 + 60} = \frac{3600}{90} = 40\text{ km/h}$.

</details>

**Q3 (NAT).** Worker A can finish a software task alone in 10 days, while Worker B takes 15 days alone. They work together on the task for 3 days, after which Worker A leaves. How many additional days will Worker B take alone to complete the remaining work?

<details><summary>Answer</summary>

**Answer:** 7.5  
**Solution:**  
Work rate of A $= 1/10$ task/day; work rate of B $= 1/15$ task/day.  
Combined rate of A and B $= \frac{1}{10} + \frac{1}{15} = \frac{3 + 2}{30} = \frac{5}{30} = \frac{1}{6}$ task/day.  
Work completed in the first 3 days $= 3 \times \frac{1}{6} = \frac{1}{2}$ of the total task.  
Remaining work $= 1 - \frac{1}{2} = \frac{1}{2}$.  
Time required by B working alone at rate $1/15$:  
$$\text{Additional days} = \frac{\text{Remaining work}}{\text{Rate of B}} = \frac{1/2}{1/15} = \frac{15}{2} = 7.5\text{ days}$$

</details>

**Q4 (MCQ).** For real numbers $x, y > 1$, if $\log_{10}(x + y) = \log_{10} x + \log_{10} y$, then $y$ expressed in terms of $x$ is:  
(A) $\frac{x}{x - 1}$  
(B) $\frac{x - 1}{x}$  
(C) $\frac{x}{x + 1}$  
(D) $x(x - 1)$  

<details><summary>Answer</summary>

**Answer:** A  
**Solution:**  
Using the logarithmic product identity on the right-hand side:  
$$\log_{10}(x + y) = \log_{10}(x \cdot y)$$  
Since the logarithm function is strictly monotonic:  
$$x + y = xy$$  
Rearranging to isolate $y$:  
$$xy - y = x \implies y(x - 1) = x \implies y = \frac{x}{x - 1}$$  
Since $x > 1$, $x - 1 > 0$, so $y$ is well-defined and positive.

</details>

**Q5 (NAT).** A right-angled triangle has perpendicular legs of lengths 6 cm and 8 cm. What is the radius of its inscribed circle (inradius) in cm?

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:**  
Let the legs be $a = 6\text{ cm}$ and $b = 8\text{ cm}$.  
Hypotenuse $c = \sqrt{a^2 + b^2} = \sqrt{36 + 64} = \sqrt{100} = 10\text{ cm}$.  
Area of triangle $\Delta = \frac{1}{2} \times 6 \times 8 = 24\text{ cm}^2$.  
Semi-perimeter $s = \frac{a + b + c}{2} = \frac{6 + 8 + 10}{2} = 12\text{ cm}$.  
Inradius $r = \frac{\Delta}{s} = \frac{24}{12} = 2\text{ cm}$.  
Alternatively, for any right-angled triangle:  
$$r = \frac{a + b - c}{2} = \frac{6 + 8 - 10}{2} = \frac{4}{2} = 2\text{ cm}$$

</details>

**Q6 (MSQ).** Let $n = 2^4 \times 3^3 \times 5^2 = 10800$. Which of the following statements is/are TRUE?  
(A) The total number of positive divisors of $n$ is 60.  
(B) The number of even positive divisors of $n$ is 48.  
(C) The number of positive divisors of $n$ that are multiples of 15 is 36.  
(D) The number of perfect square positive divisors of $n$ is 12.  

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:**  
Any divisor of $n$ has form $2^a 3^b 5^c$ with $0 \le a \le 4$, $0 \le b \le 3$, $0 \le c \le 2$.  
- **(A) Total divisors:** $(4 + 1)(3 + 1)(2 + 1) = 5 \times 4 \times 3 = 60$. **(True)**  
- **(B) Even divisors:** Must contain at least one factor of 2, so $a \in \{1, 2, 3, 4\}$ (4 choices):  
  $$4 \times (3 + 1) \times (2 + 1) = 4 \times 4 \times 3 = 48 \quad \textbf{(True)}$$  
- **(C) Multiples of 15:** Must be divisible by $3 \times 5$, so $b \in \{1, 2, 3\}$ (3 choices) and $c \in \{1, 2\}$ (2 choices):  
  $$(4 + 1) \times 3 \times 2 = 5 \times 3 \times 2 = 30 \ne 36 \quad \textbf{(False)}$$  
- **(D) Perfect squares:** Exponents must be even: $a \in \{0, 2, 4\}$ (3 choices), $b \in \{0, 2\}$ (2 choices), $c \in \{0, 2\}$ (2 choices):  
  $$3 \times 2 \times 2 = 12 \quad \textbf{(True)}$$

</details>

**Q7 (NAT).** A dataset consists of seven distinct integers: $14, 16, 18, x, 26, 28, 30$ listed in strictly increasing order ($18 < x < 26$). If the arithmetic mean of this dataset is exactly equal to its median, find the value of $x$.

<details><summary>Answer</summary>

**Answer:** 22  
**Solution:**  
Since the seven distinct integers are in strictly increasing order, the median is the 4th element:  
$$\text{Median} = x$$  
The sum of the seven numbers is:  
$$\text{Sum} = 14 + 16 + 18 + x + 26 + 28 + 30 = 132 + x$$  
The arithmetic mean is:  
$$\text{Mean} = \frac{132 + x}{7}$$  
Setting $\text{Mean} = \text{Median}$:  
$$\frac{132 + x}{7} = x \implies 132 + x = 7x \implies 6x = 132 \implies x = 22$$  
Check consistency: $18 < 22 < 26$, so the list remains strictly increasing: $14, 16, 18, 22, 26, 28, 30$.  
Mean $= \frac{154}{7} = 22 = \text{Median}$.

</details>
