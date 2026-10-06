# Constraints, functional dependencies, and normalization

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** integrity constraints, functional dependencies, keys, normal forms, lossless join, dependency preservation
> **Prerequisites:** [Relational model](relational-model-algebra-calculus.md) · **Leads to:** [Transactions](transactions-and-concurrency.md)

## Quick glance
- FD $X\to Y$: any two tuples agreeing on X must agree on Y.
- Attribute closure $X^+$ finds all attributes implied by X; X is a superkey iff $X^+$ is all attributes.
- Candidate key is a minimal superkey; prime attribute belongs to some candidate key.
- 2NF removes partial dependency on part of a composite key; 3NF removes problematic transitive dependency; BCNF requires every nontrivial determinant be a superkey.
- Decomposition should be lossless; dependency preservation is a separate property.

## 1. Closure and keys
For R(A,B,C,D) with FDs A→B, B→C, AC→D, closure A+ starts {A}, adds B then C, and does not add D; hence A is not a key. Closure AC+ adds D as well, so AC+={A,B,C,D}. Since neither A nor C alone is a key, AC is a candidate key.

## 2. Normal forms
1NF: atomic values under the chosen model. 2NF: 1NF and no non-prime attribute depends on a proper subset of any candidate key. 3NF: for each FD X→A, X is a superkey or A is prime. BCNF: for every nontrivial FD X→Y, X is a superkey.

Example relation Enrol(Student,Course,Instructor), key (Student,Course), with Course→Instructor. Instructor depends on only Course, part of the composite key: violates 2NF. Split Course-Instructor and Student-Course.

## 3. Lossless join and preservation
Binary decomposition R→R1,R2 is lossless under FDs if common attributes functionally determine all attributes of at least one component: $(R1\cap R2)\to R1$ or $(R1\cap R2)\to R2$. Dependency preservation means constraints can be checked on components without joining. A decomposition may have one property and not the other.

## GATE traps
- “Minimal” in candidate key means no attribute can be removed while retaining superkey status.
- 3NF allows a prime attribute on the RHS even if determinant is not a superkey; BCNF does not.
- A relation can have multiple candidate keys.
- Test lossless join and dependency preservation independently.

## Connections
- [Relational model](relational-model-algebra-calculus.md) — keys and constraints govern valid tuples.
- [Transactions](transactions-and-concurrency.md) — constraints must remain true across updates.
- [Data preprocessing](data-warehousing-and-preprocessing.md) — database normalization differs from feature normalization.

## Practice
**Q1.** R(A,B,C), F={A→B,B→C}. Find A+.
<details><summary>Answer</summary> {A,B,C}; A is a candidate key.</details>

**Q2.** What distinguishes BCNF from 3NF?
<details><summary>Answer</summary> 3NF permits RHS prime attributes for a non-superkey determinant; BCNF requires every determinant to be a superkey.</details>

**Q3.** Is dependency preservation the same as lossless join?
<details><summary>Answer</summary> No; they are distinct decomposition properties.</details>
