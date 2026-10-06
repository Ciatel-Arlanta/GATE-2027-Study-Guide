# Database checkpoint

**Q1.** If A→B and B→C, what follows?
<details><summary>Answer</summary> A→C by transitivity.</details>

**Q2.** R(A,B,C), F={A→B,B→C}. Is A a candidate key?
<details><summary>Answer</summary> Yes; A+={A,B,C} and A is a singleton.</details>

**Q3.** What does $\sigma_{age>20}(Student)$ do?
<details><summary>Answer</summary> Selects rows whose age exceeds 20.</details>

**Q4.** A precedence graph contains a cycle. Conflict serializable?
<details><summary>Answer</summary> No.</details>

**Q5.** Which index is preferable for range lookup, B+ tree or hash?
<details><summary>Answer</summary> B+ tree.</details>

**Q6.** Which SQL predicate finds null values?
<details><summary>Answer</summary> `IS NULL`, not `= NULL`.</details>

**Q7.** Name the two independent goals often checked for decomposition.
<details><summary>Answer</summary> Lossless join and dependency preservation.</details>

**Q8.** Which OLAP operation fixes one dimension to a single value?
<details><summary>Answer</summary> Slice.</details>

**Q9.** Do shared reads conflict in a schedule?
<details><summary>Answer</summary> No; neither operation writes.</details>

≥80%: proceed; below 60%: revisit relational algebra, key closure, and schedule graphs.
