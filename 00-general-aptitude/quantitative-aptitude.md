# Quantitative aptitude: arithmetic, charts, and elementary counting

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** ratios, percentages, powers and logarithms, counting, series, mensuration, data interpretation, elementary statistics
> **Prerequisites:** none · **Leads to:** [Probability basics](../02-probability-statistics/probability-basics.md)

## Quick glance
- Convert a percentage to a multiplier: increase by $p\%$ means multiply by $1+p/100$; decrease means $1-p/100$.
- For ratio $a:b$, write the quantities as $ak,bk$ before using a total or difference.
- Speed = distance/time; work rates add when workers act together.
- In charts, read the title, units, legend, and time period before calculating.
- $\log_a(xy)=\log_a x+\log_a y$; $a^{\log_a x}=x$.
- Count ordered selections with permutations; count unordered selections with combinations.

## 1. Ratios, percentages, and averages
A ratio compares quantities in the same units. If red and blue balls are in ratio $3:5$ and there are 40 total, one share is $40/(3+5)=5$, giving 15 red and 25 blue.

Percentage change uses the old value as the base. A price rises from 80 to 100: change $=20$, percentage rise $=20/80\cdot100=25\%$. Successive changes multiply: +20% then −20% gives $1.2\times0.8=0.96$, a net 4% decrease.

For values $x_i$ with frequencies $f_i$, weighted mean is $\sum f_ix_i/\sum f_i$. Example: scores 60 and 80 with weights 2 and 3 have mean $(2\cdot60+3\cdot80)/5=72$.

## 2. Rates and proportional reasoning
If a vehicle covers 150 km in 3 hours, average speed is 50 km/h. For equal distances, average speed is not the arithmetic mean: at 40 km/h outward and 60 km/h back, total distance 120 km takes $1.5+1=2.5$ h, so average is 48 km/h.

If A completes a job in 6 days, A's rate is $1/6$ job/day. B takes 3 days, so together their rate is $1/6+1/3=1/2$ job/day and they finish in 2 days. Keep units attached to rates.

## 3. Powers, logarithms, and sequences
Use exponent laws before evaluating: $2^3\cdot2^4=2^7$. Logarithms turn products into sums and are defined only for positive arguments with base $a>0,a\ne1$. For $3^x=81$, $x=\log_3 81=4$.

Check a sequence by taking first differences, then second differences. For $2,5,10,17,26$, first differences are $3,5,7,9$, so the next difference is 11 and next term is 37. A pattern is a conjecture: verify it against every given term.

## 4. Counting and elementary probability
Choose a committee of 2 from 5 people: $\binom52=10$ because order does not matter. Assign president and secretary: $5\cdot4=20$ because roles distinguish order. If every outcome is equally likely, probability is favourable outcomes divided by total outcomes.

## 5. Data interpretation and geometry
For a table or chart, note whether values are counts, percentages, thousands, or cumulative totals. If sales increase from 200 to 250, the relative increase is 25%; the absolute increase is 50 units.

Rectangle area $lw$; triangle area $bh/2$; circle area $\pi r^2$ and circumference $2\pi r$. Similar figures with length scale $k$ have area scale $k^2$ and volume scale $k^3$. Keep units squared or cubed.

## GATE traps
- Percentage increase then decrease by the same rate does not restore the original value.
- Do not average rates directly when travel times or distances differ.
- “At least one” is often simpler as $1-P(\text{none})$.
- Chart axes may start above zero; compare values numerically, not by visual bar height.
- Check whether a counting question asks for arrangements or selections.

## Connections
- [Probability basics](../02-probability-statistics/probability-basics.md) — counting outcomes becomes probability.
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — systematic counting and inclusion-exclusion.
- [Verbal aptitude](verbal-aptitude.md) — chart questions depend on precise interpretation of labels.

## Practice
**Q1 (NAT).** A quantity rises 10% and then falls 10%. What is the net percentage change?
<details><summary>Answer</summary> **−1%.** Multiplier $1.1\cdot0.9=0.99$.</details>

**Q2 (NAT).** A and B finish a task alone in 8 and 12 days. How many days together?
<details><summary>Answer</summary> Rate $1/8+1/12=5/24$; time $24/5=4.8$ days.</details>

**Q3 (MCQ).** Number of ways to choose 3 people from 7: (A) 35 (B) 210 (C) 343 (D) 21.
<details><summary>Answer</summary> **A.** $\binom73=35$; order is irrelevant.</details>
