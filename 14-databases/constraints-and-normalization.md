# Constraints, functional dependencies, and normalization

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** integrity constraints, functional dependencies, candidate keys and superkeys, minimal cover, normal forms (1NF, 2NF, 3NF, BCNF), lossless join decomposition, dependency preservation
> **Prerequisites:** [Relational model](relational-model-algebra-calculus.md) · **Leads to:** [Transactions and concurrency](transactions-and-concurrency.md), [Data warehousing and preprocessing](data-warehousing-and-preprocessing.md)
> **PYQ weightage:** High · 39 Qs since 2015 (26 since 2021) · DA: 8 · Trend: rising · Most asked: Normal Forms & Highest NF (25), Candidate Keys & Superkeys (4), Functional Dependencies & Minimal Cover (4), Lossless Join & Dependency Preservation (2), Referential Integrity (5)

## Quick glance

- **Exam focus:** Candidate key determination, testing highest normal form (especially 3NF vs BCNF), checking lossless join vs dependency preservation, and counting superkeys; asked heavily as NAT numericals and MCQ/MSQ.
- An **FD $X \to Y$** means any two tuples agreeing on attribute set $X$ must also agree on attribute set $Y$.
- **Attribute closure $X^+$** is computed by repeatedly applying FDs whose LHS is a subset of the accumulated set; $X$ is a **superkey** iff $X^+$ contains all attributes of $R$.
- A **candidate key (CK)** is a minimal superkey (no proper subset is a superkey). Attributes appearing in at least one CK are **prime**; all others are **non-prime**.
- Any attribute that **never appears on the RHS** of any FD must belong to **every** candidate key.
- **Normal forms hierarchy:** $\text{BCNF} \subset \text{3NF} \subset \text{2NF} \subset \text{1NF}$.
  - **1NF:** Atomic domains (no multi-valued or composite attributes).
  - **2NF:** 1NF + no non-prime attribute is partially dependent on any CK (no proper subset of a CK determines a non-prime attribute).
  - **3NF:** For every non-trivial $X \to A$, either $X$ is a superkey OR $A$ is a prime attribute.
  - **BCNF:** For every non-trivial $X \to A$, $X$ must be a superkey.
- **2-attribute rule:** Any relation schema with exactly 2 attributes is **always in BCNF**.
- **All-prime rule:** If every attribute in $R$ is prime, $R$ is **guaranteed to be in at least 3NF**.
- Binary decomposition $R \to (R_1, R_2)$ is **lossless** iff $(R_1 \cap R_2) \to R_1$ or $(R_1 \cap R_2) \to R_2$ holds in $F^+$.
- A lossless-join, dependency-preserving decomposition into **3NF is always possible**; into **BCNF**, lossless join is always achievable, but dependency preservation may be impossible.

## 1. Functional dependencies, Armstrong axioms, and minimal cover [PYQ: Medium]

A **functional dependency (FD)** $X \to Y$ on relation schema $R$ is a semantic constraint: for any legal relation instance $r(R)$ and any two tuples $t_1, t_2 \in r$, if $t_1[X] = t_2[X]$, then $t_1[Y] = t_2[Y]$.

An FD $X \to Y$ is **trivial** if $Y \subseteq X$; otherwise it is non-trivial (or strictly non-trivial if $X \cap Y = \emptyset$).

### Armstrong's axioms

The sound and complete inference rules for functional dependencies are:
1. **Reflexivity:** If $Y \subseteq X$, then $X \to Y$.
2. **Augmentation:** If $X \to Y$, then $XZ \to YZ$ for any attribute set $Z$.
3. **Transitivity:** If $X \to Y$ and $Y \to Z$, then $X \to Z$.

**Derived secondary rules:**
- **Union:** If $X \to Y$ and $X \to Z$, then $X \to YZ$.
- **Decomposition:** If $X \to YZ$, then $X \to Y$ and $X \to Z$ (note: LHS cannot be split!).
- **Pseudo-transitivity:** If $X \to Y$ and $WY \to Z$, then $WX \to Z$.

### Attribute closure algorithm ($X^+$)

To find all attributes functionally determined by $X$ under an FD set $F$:
1. Initialize $\text{closure} = X$.
2. Repeatedly scan $F$: if $W \to Z \in F$ and $W \subseteq \text{closure}$, then $\text{closure} = \text{closure} \cup Z$.
3. Terminate when no new attribute can be added. The result is $X^+$.

### Minimal cover (canonical cover)

An FD set $F_c$ is a **minimal cover** of $F$ iff $F_c \equiv F$ (both imply the same closures) and:
1. Every FD in $F_c$ has a singleton RHS (RHS decomposition: $X \to YZ \implies X \to Y, X \to Z$).
2. No FD in $F_c$ has an **extraneous attribute on the LHS**: for $XA \to B$, attribute $A$ is extraneous if $B \in X^+$ under $F$.
3. No FD in $F_c$ is **redundant**: an FD $X \to Y$ is redundant if $Y \subseteq X^+$ under $F \setminus \{X \to Y\}$.

**Worked example 1 (Canonical cover computation).**  
Find the minimal cover for relation $R(A, B, C, D)$ with FDs:
$$F = \{A \to BC,\; B \to C,\; A \to B,\; AB \to C,\; AC \to D\}$$

*Step 1: Singleton RHS.*  
$$F_1 = \{A \to B,\; A \to C,\; B \to C,\; A \to B,\; AB \to C,\; AC \to D\}$$
Remove duplicate $A \to B$:
$$F_1 = \{A \to B,\; A \to C,\; B \to C,\; AB \to C,\; AC \to D\}$$

*Step 2: Eliminate extraneous attributes on LHS.*
- In $AB \to C$: test if $A$ is extraneous by finding $B^+$ under $F_1$: $B^+ = \{B, C\}$. Since $C \in B^+$, $A$ is extraneous! $AB \to C$ reduces to $B \to C$ (which is already present).
- In $AC \to D$: test if $A$ is extraneous: $C^+ = \{C\}$ (does not derive $D$). Test if $C$ is extraneous: $A^+$ without using $AC \to D$: $A^+ = \{A, B, C\}$ (does not contain $D$). Thus neither is extraneous in $AC \to D$.
Now the set is:
$$F_2 = \{A \to B,\; A \to C,\; B \to C,\; AC \to D\}$$

*Step 3: Eliminate redundant FDs.*
- Test $A \to B$: in $F_2 \setminus \{A \to B\}$, $A^+ = \{A, C, D\}$. Does not contain $B$. Not redundant.
- Test $A \to C$: in $F_2 \setminus \{A \to C\}$, $A^+ = \{A, B\}$. Since $B \to C \in F_2$, $B$ gives $C$, so $A^+ = \{A, B, C\}$. Contains $C$! Therefore $A \to C$ is **redundant** and can be eliminated.
- Test $B \to C$: in $F_2 \setminus \{B \to C\}$, $B^+ = \{B\}$. Not redundant.
- Test $AC \to D$: in $F_2 \setminus \{AC \to D\}$, $(AC)^+ = \{A, B, C\}$. Not redundant.

**Resulting minimal cover:**
$$F_c = \{A \to B,\; B \to C,\; AC \to D\}$$

## 2. Superkeys, candidate keys, and counting superkeys [PYQ: High]

- A **superkey** of $R$ is any attribute subset $K \subseteq R$ such that $K^+ = R$.
- A **candidate key (CK)** is a minimal superkey: $K^+ = R$, and for every proper subset $K' \subset K$, $(K')^+ \neq R$.
- **Primary key:** One candidate key arbitrarily chosen by the database designer.
- **Prime attribute:** An attribute that belongs to *at least one* candidate key.
- **Non-prime attribute:** An attribute that does *not* belong to any candidate key.

### Systematic strategy to find all candidate keys

1. Partition attributes into four sets based on their appearance in $F$:
   - **$L$ (Left only):** Appears only on LHS of FDs.
   - **$R$ (Right only):** Appears only on RHS of FDs.
   - **$B$ (Both):** Appears on both LHS and RHS.
   - **$N$ (Neither):** Appears in neither LHS nor RHS.
2. **Essential attributes:** Every attribute in $L \cup N$ **must belong to every candidate key**.
3. Compute $(L \cup N)^+$:
   - If $(L \cup N)^+ = R$, then $L \cup N$ is the **unique candidate key**.
   - If not, systematically combine $L \cup N$ with subsets of $B$ in increasing order of size until all candidate keys are found.
4. Attributes in $R$ (right-only) are never part of any candidate key, and can never form a candidate key alone.

**Worked example 2 (Finding all candidate keys & counting superkeys).**  
Given relation schema $R(A, B, C, D, E)$ with functional dependencies:
$$F = \{AB \to C,\; C \to D,\; D \to E,\; E \to A\}$$

*Step 1: Partition attributes.*
- LHS attributes: $\{A, B, C, D, E\}$
- RHS attributes: $\{A, C, D, E\}$
- Left only ($L$): $\{B\}$
- Right only ($R$): $\emptyset$
- Both ($B$): $\{A, C, D, E\}$
- Neither ($N$): $\emptyset$

$B$ never appears on the RHS, so **$B$ must be in every candidate key**.

*Step 2: Closures of 2-attribute combinations containing $B$.*
- $(BA)^+$: $BA \to C \to D \to E \implies (BA)^+ = \{A, B, C, D, E\} = R$. $\implies \mathbf{AB}$ is a candidate key!
- $(BC)^+$: $BC \to D \to E \to A \implies (BC)^+ = \{A, B, C, D, E\} = R$. $\implies \mathbf{BC}$ is a candidate key!
- $(BD)^+$: $BD \to E \to A \to C \implies (BD)^+ = \{A, B, C, D, E\} = R$. $\implies \mathbf{BD}$ is a candidate key!
- $(BE)^+$: $BE \to A \to C \to D \implies (BE)^+ = \{A, B, C, D, E\} = R$. $\implies \mathbf{BE}$ is a candidate key!

All candidate keys have size 2: $\{AB, BC, BD, BE\}$.  
**Prime attributes:** $\{A, B, C, D, E\}$ (all attributes are prime!).  
**Non-prime attributes:** $\emptyset$.

*Step 3: Number of superkeys.*  
A superkey is any subset of $\{A, B, C, D, E\}$ containing at least one candidate key.
Every candidate key has $B$ plus one attribute from $\{A, C, D, E\}$.
Any subset containing $B$ and at least one attribute from $\{A, C, D, E\}$ contains a candidate key and is therefore a superkey.
- Total subsets containing $B$: $2^{5 - 1} = 2^4 = 16$.
- Among these 16 subsets, the only one that contains *no* attribute from $\{A, C, D, E\}$ is $\{B\}$ alone.
- Since $\{B\}^+ = \{B\} \neq R$, $\{B\}$ is not a superkey.
- Every other subset containing $B$ is a superkey.
$$\text{Total superkeys} = 16 - 1 = 15$$

## 3. Normal forms (1NF, 2NF, 3NF, BCNF) and testing highest NF [PYQ: High]

Normalization reduces data redundancy and eliminates update, insertion, and deletion anomalies.

```text
+-------------------------------------------------------+
| 1NF: Atomic attribute values only                     |
|   +---------------------------------------------------+
|   | 2NF: 1NF + No partial dependencies                |
|   |   +-----------------------------------------------+
|   |   | 3NF: 2NF + No transitive dependencies         |
|   |   |   +-------------------------------------------+
|   |   |   | BCNF: Determinant of every FD is superkey |
|   |   |   +-------------------------------------------+
```

### Definitions of normal forms

| Normal Form | Formal condition for every non-trivial FD $X \to A$ ($A$ is a single attribute $\notin X$) | What it eliminates |
|---|---|---|
| **1NF** | Domain of each attribute contains only atomic (indivisible) values. | Repeating groups, multi-valued attributes |
| **2NF** | In 1NF and no non-prime attribute is partially dependent on any candidate key ($X$ cannot be a proper subset of any CK if $A$ is non-prime). | Partial dependencies on composite keys |
| **3NF** | In 2NF and for every $X \to A$: either **$X$ is a superkey** OR **$A$ is a prime attribute**. | Transitive dependencies for non-prime attributes |
| **BCNF** | For every non-trivial $X \to A$: **$X$ must be a superkey**. | All functional dependency redundancy |

### Rapid classification rules for GATE

1. **Two-attribute theorem:** Any relation schema $R(A, B)$ with any valid FD set is **always in BCNF**.
2. **All-prime theorem:** If all attributes of relation $R$ are prime, $R$ is **guaranteed to be in 3NF** (since the RHS of every FD is prime, satisfying the 3NF condition). It may or may not be in BCNF.
3. **Single candidate key with all simple attributes (size 1):** If all candidate keys consist of a single attribute, the relation is **automatically in 2NF** (no proper subset of a key can exist).
4. **BCNF vs 3NF test:**
   - Check BCNF first: is every LHS a superkey? If yes, it is in BCNF.
   - If not, check if every failing FD has a prime attribute on the RHS. If yes, it is in 3NF (but not BCNF).
   - If a failing FD has a non-prime attribute on the RHS, check if the LHS is a proper subset of a candidate key. If yes $\implies$ not in 2NF (in 1NF only). If no $\implies$ in 2NF (not 3NF).

**Worked example 3 (Testing highest normal form).**  
Consider relation $R(A, B, C, D, E, F)$ with $F = \{AB \to C,\; C \to D,\; D \to E,\; E \to F,\; F \to A\}$.  
Determine the highest normal form of $R$.

*Step 1: Find candidate keys.*
- $B$ never appears on the RHS of any FD $\implies B$ must be in every candidate key.
- $(AB)^+ = \{A, B, C, D, E, F\} = R \implies AB$ is a CK.
- Since $F \to A$, $(FB)^+ = \{F, B, A, C, D, E\} = R \implies FB$ is a CK.
- Since $E \to F$, $(EB)^+ = R \implies EB$ is a CK.
- Since $D \to E$, $(DB)^+ = R \implies DB$ is a CK.
- Since $C \to D$, $(CB)^+ = R \implies CB$ is a CK.
Candidate keys: $\{AB, BC, BD, BE, BF\}$.  
Prime attributes: $\{A, B, C, D, E, F\}$ — **all attributes are prime!**  
Non-prime attributes: $\emptyset$.

*Step 2: Check normal forms.*
- **Check BCNF:** Consider $C \to D$. Is $C$ a superkey? $C^+ = \{C, D, E, F, A\} \neq R$ (missing $B$). Thus $C$ is not a superkey. BCNF is violated.
- **Check 3NF:** For $C \to D$, the RHS attribute $D$ is prime. For every FD in $F$, the RHS attribute is prime!
Therefore, $R$ satisfies the 3NF condition for all FDs.  
**Highest normal form of $R$ is 3NF.**

## 4. Decomposition: lossless join and dependency preservation [PYQ: High]

When decomposing a relation schema $R$ into $D = \{R_1, R_2, \dots, R_k\}$:
1. **Attribute preservation:** $\bigcup_{i=1}^k R_i = R$.
2. **Lossless-join decomposition:** $\pi_{R_1}(r) \bowtie \pi_{R_2}(r) \bowtie \dots \bowtie \pi_{R_k}(r) = r$ for every valid instance $r(R)$. No spurious tuples are generated.
3. **Dependency preservation:** $(\bigcup_{i=1}^k \pi_{R_i}(F))^+ = F^+$. Constraints can be enforced locally on sub-relations without computing expensive joins.

### Binary decomposition lossless test

For a decomposition into two relations $R_1$ and $R_2$, the decomposition is **lossless** if and only if the common attributes functionally determine all attributes of at least one relation:
$$(R_1 \cap R_2) \to R_1 \in F^+ \quad \text{OR} \quad (R_1 \cap R_2) \to R_2 \in F^+$$
Equivalently: $(R_1 \cap R_2)$ must be a **superkey** of $R_1$ or a **superkey** of $R_2$.

### Dependency preservation test

For each FD $X \to Y \in F$:
1. Initialize $Z = X$.
2. Repeatedly update $Z = Z \cup ((Z \cap R_i)^+ \cap R_i)$ for each sub-schema $R_i$.
3. If $Y \subseteq Z$, then $X \to Y$ is preserved.
4. If all FDs in $F$ are preserved, the decomposition is **dependency preserving**.

**Worked example 4 (Decomposition properties).**  
Given $R(A, B, C, D)$ with $F = \{A \to B,\; B \to C,\; C \to D,\; D \to A\}$.  
Evaluate decomposition $D_1 = \{R_1(A, B),\; R_2(B, C),\; R_3(C, D)\}$:

*Step 1: Test Lossless join.*
Combine $R_1(A, B)$ and $R_2(B, C)$ first:
- $R_1 \cap R_2 = \{B\}$.
- $B^+ = \{B, C, D, A\} \supseteq R_2$.
- Since $B \to R_2$, the join $R_1 \bowtie R_2$ is lossless, producing $R_{12}(A, B, C)$.
Now combine $R_{12}(A, B, C)$ with $R_3(C, D)$:
- $R_{12} \cap R_3 = \{C\}$.
- $C^+ = \{C, D, A, B\} \supseteq R_3$.
- Since $C \to R_3$, the join $R_{12} \bowtie R_3$ is lossless.  
$\implies D_1$ is **lossless**.

*Step 2: Test Dependency preservation.*
- $A \to B$ is covered by $R_1(A, B)$ ($A, B \in R_1$). Preserved.
- $B \to C$ is covered by $R_2(B, C)$ ($B, C \in R_2$). Preserved.
- $C \to D$ is covered by $R_3(C, D)$ ($C, D \in R_3$). Preserved.
- Test $D \to A$:
  - Start with $Z = \{D\}$.
  - In $R_3(C, D)$: $(Z \cap R_3)^+ = \{D\}^+ = \{D, A, B, C\}$. Intersect with $R_3$: $\{C, D\}$. So $Z$ becomes $\{C, D\}$.
  - In $R_2(B, C)$: $(Z \cap R_2)^+ = \{C\}^+ = \{C, D, A, B\}$. Intersect with $R_2$: $\{B, C\}$. So $Z$ becomes $\{B, C, D\}$.
  - In $R_1(A, B)$: $(Z \cap R_1)^+ = \{B\}^+ = \{B, C, D, A\}$. Intersect with $R_1$: $\{A, B\}$. So $Z$ becomes $\{A, B, C, D\}$.
  - Since $A \in Z$, $D \to A$ is **preserved**!  
$\implies D_1$ is **both lossless and dependency preserving**.

### 3NF vs BCNF decomposition guarantees

```text
Decomposition Goal:
Lossless Join?              Always achievable for both 3NF and BCNF
Dependency Preserving?      Always achievable for 3NF; NOT always possible for BCNF
```
*Classic non-preservable BCNF example:* $R(A, B, C)$ with $F = \{AB \to C, C \to B\}$.  
Candidate keys: $AB, AC$. In 3NF (since $B$ is prime in $C \to B$), but not BCNF ($C$ is not superkey). Decomposing to BCNF gives $R_1(C, B)$ and $R_2(A, C)$ — which is lossless, but loses the dependency $AB \to C$!

## 5. Integrity constraints and referential integrity [PYQ: Medium]

Relational databases enforce integrity constraints to ensure data correctness:
1. **Domain constraint:** Attribute values must be atomic and lie in the attribute's domain.
2. **Entity integrity:** Primary key attributes can **never take NULL values**.
3. **Referential integrity:** A foreign key in referencing relation $R_1$ referencing primary key of referenced relation $R_2$ must either:
   - Match a valid primary key value currently existing in $R_2$, OR
   - Be completely NULL (unless foreign key is declared `NOT NULL`).

### Referential actions on update and delete

When a tuple in the referenced relation $R_2$ is deleted or updated:
- **ON DELETE / UPDATE RESTRICT (or NO ACTION):** Rejects the operation if matching referencing tuples exist.
- **ON DELETE / UPDATE CASCADE:** Propagates the deletion/update to referencing tuples.
- **ON DELETE / UPDATE SET NULL:** Sets the foreign key column in referencing tuples to NULL.
- **ON DELETE / UPDATE SET DEFAULT:** Sets the foreign key column to its defined default value.

**Worked example 5 (Referential actions and cascade tracing).**  
Table `Dept(did PK, dname)` has tuples: `(1, 'CS'), (2, 'EE'), (3, 'ME')`.  
Table `Emp(eid PK, ename, did FK references Dept(did) ON DELETE CASCADE)` has:  
`(101, 'A', 1), (102, 'B', 1), (103, 'C', 2), (104, 'D', 3), (105, 'E', 2)`.  
Table `Proj(pid PK, eid FK references Emp(eid) ON DELETE SET NULL)` has:  
`(501, 101), (502, 103), (503, 105), (504, 104)`.  

If `DELETE FROM Dept WHERE did = 2;` is executed:
1. Tuples with `did = 2` in `Dept` are deleted: `(2, 'EE')` removed.
2. Due to `ON DELETE CASCADE` in `Emp`, all employees with `did = 2` are deleted: `(103, 'C', 2)` and `(105, 'E', 2)` are removed.
3. Due to `ON DELETE SET NULL` in `Proj`, any project referencing `eid = 103` or `eid = 105` has its `eid` set to NULL:
   - `(502, 103)` becomes `(502, NULL)`.
   - `(503, 105)` becomes `(503, NULL)`.
4. Remaining rows in `Emp`: 3 rows (`101, 102, 104`).  
Remaining rows in `Proj`: 4 rows (2 rows now have `eid = NULL`).

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| Attribute closure | $X_{i+1} = X_i \cup \{Z \mid W \to Z \in F, W \subseteq X_i\}$ | Finding candidate keys, superkeys, and testing FDs |
| Superkeys of candidate key $K$ | If single CK $K$ of size $k$ in relation with $n$ attributes: $2^{n-k}$ superkeys | Counting superkeys with single candidate key |
| Superkeys with multiple CKs | Inclusion-exclusion: $\lvert S_1 \cup S_2 \rvert = \lvert S_1 \rvert + \lvert S_2 \rvert - \lvert S_1 \cap S_2 \rvert$ | Counting superkeys with 2 or more candidate keys |
| Binary lossless join | $(R_1 \cap R_2) \to R_1 \in F^+$ OR $(R_1 \cap R_2) \to R_2 \in F^+$ | Testing binary decomposition for lossless join |
| 2-attribute relation | Always in BCNF | Shortcut for schemas with $\lvert R \rvert = 2$ |
| All-prime relation | Guaranteed in $\ge \text{3NF}$ | Shortcut when all attributes belong to candidate keys |
| 3NF decomposition guarantee | Lossless join + dependency preserving always exists | Theoretical existence questions on 3NF |
| BCNF decomposition guarantee | Lossless join always exists; dependency preservation NOT guaranteed | Theoretical existence questions on BCNF |

## GATE traps

- **"Minimal" in candidate key means subset-minimal, not minimum cardinality:** A candidate key is minimal in the subset sense ($K' \subset K \implies (K')^+ \neq R$). A relation can have candidate keys of different lengths (e.g., $A$ of size 1 and $BC$ of size 2).
- **Prime attributes on RHS in 3NF:** For $X \to A$, 3NF allows $X$ to NOT be a superkey as long as $A$ is prime. BCNF strictly forbids this. Forgetting this is the #1 mistake in 3NF vs BCNF questions.
- **Decomposition rule applies ONLY to RHS:** $A \to BC \implies A \to B$ and $A \to C$. But $AB \to C \not\Longrightarrow A \to C$ and $B \to C$! Splitting LHS is completely invalid.
- **Lossless join is NOT symmetric with dependency preservation:** A decomposition can be lossless without preserving dependencies, or preserve dependencies without being lossless. Test both independently.
- **Foreign key can be NULL:** Referential integrity is not violated if a foreign key value is NULL, unless an explicit `NOT NULL` constraint is present.
- **Right-only attributes (appear only on the RHS):** Attributes appearing only on the RHS of FDs can *never* be part of any candidate key. Do not waste time computing closures containing them when searching for candidate keys.

## Connections

- [Relational model](relational-model-algebra-calculus.md) — Integrity constraints enforce domain, entity, and referential validity across relational tables.
- [Transactions and concurrency](transactions-and-concurrency.md) — ACID consistency guarantees that transactions transition the database from one valid state satisfying all integrity constraints to another.
- [Data warehousing and preprocessing](data-warehousing-and-preprocessing.md) — Relational normal forms eliminate operational update anomalies; data warehouses intentionally denormalize (star schema) for OLAP query speed.

## Practice

**Q1 (MCQ).** Consider relation $R(A, B, C, D, E)$ with functional dependencies $F = \{A \to B,\; BC \to D,\; E \to C\}$. What is the attribute closure of $\{A, E\}$?  
(A) $\{A, B, C, E\}$  
(B) $\{A, B, C, D, E\}$  
(C) $\{A, E\}$  
(D) $\{A, B, E\}$  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
1. Start with $\text{closure} = \{A, E\}$.
2. Since $A \subseteq \{A, E\}$ and $A \to B \in F$, add $B \implies \{A, B, E\}$.
3. Since $E \subseteq \{A, B, E\}$ and $E \to C \in F$, add $C \implies \{A, B, C, E\}$.
4. Since $BC \subseteq \{A, B, C, E\}$ and $BC \to D \in F$, add $D \implies \{A, B, C, D, E\}$.
5. Thus $(AE)^+ = \{A, B, C, D, E\} = R$.
</details>

**Q2 (NAT).** Consider a relation schema $R(A, B, C, D, E, F)$ with candidate keys $AB$ and $AC$. The total number of superkeys of $R$ is _______.  
<details><summary>Answer</summary>

**Answer:** 24  
**Solution:**  
Relation $R$ has $n = 6$ attributes.  
- Superkeys containing $AB$: $2^{6 - 2} = 2^4 = 16$.
- Superkeys containing $AC$: $2^{6 - 2} = 2^4 = 16$.
- Superkeys containing both $AB$ and $AC$ (i.e. containing $ABC$): $2^{6 - 3} = 2^3 = 8$.  
By the Principle of Inclusion-Exclusion:
$$\lvert S(AB) \cup S(AC) \rvert = \lvert S(AB) \rvert + \lvert S(AC) \rvert - \lvert S(AB \cap AC) \rvert = 16 + 16 - 8 = 24$$
</details>

**Q3 (MCQ).** A relation $R(A, B, C, D)$ has FDs $F = \{AB \to C,\; C \to D,\; D \to A\}$. The highest normal form of $R$ is:  
(A) 1NF  
(B) 2NF  
(C) 3NF  
(D) BCNF  
<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**  
1. $B$ never appears on the RHS of any FD $\implies B$ must be in every candidate key.
2. $(AB)^+ = \{A, B, C, D\} \implies AB$ is a CK.
3. Since $D \to A$, $(DB)^+ = \{D, B, A, C\} \implies BD$ is a CK.
4. Since $C \to D$, $(CB)^+ = \{C, B, D, A\} \implies BC$ is a CK.
Candidate keys are $\{AB, BC, BD\}$.  
Prime attributes: $\{A, B, C, D\}$ — all attributes of $R$ are prime!  
Since every attribute is prime, the RHS of every FD is prime, satisfying the 3NF condition.  
However, for $C \to D$, $C^+ = \{C, D, A\} \neq R$, so $C$ is not a superkey $\implies$ BCNF is violated.  
Thus, the highest normal form is 3NF.
</details>

**Q4 (MSQ).** Which of the following statements is/are TRUE regarding relational decompositions?  
(A) Any relation schema with only two attributes is always in BCNF.  
(B) Every dependency-preserving decomposition is also lossless.  
(C) It is always possible to decompose a relation schema into a set of 3NF relations such that the decomposition is lossless and dependency preserving.  
(D) If a decomposition is in BCNF and lossless, it is guaranteed to preserve all functional dependencies.  
<details><summary>Answer</summary>

**Answer:** (A), (C)  
**Solution:**  
- (A) is TRUE: for $R(A, B)$, non-trivial FDs can only be $A \to B$ (making $A$ a key) or $B \to A$ (making $B$ a key), so every determinant is a superkey.
- (B) is FALSE: lossless join and dependency preservation are independent properties.
- (C) is TRUE: 3NF synthesis algorithm guarantees both lossless join and dependency preservation.
- (D) is FALSE: BCNF decomposition guarantees lossless join, but dependency preservation cannot always be achieved (e.g. $AB \to C, C \to B$).
</details>

**Q5 (NAT).** Relation $R(A, B, C, D, E, F)$ has $F = \{A \to B,\; B \to C,\; C \to D,\; D \to E\}$. The number of candidate keys of $R$ is _______.  
<details><summary>Answer</summary>

**Answer:** 1  
**Solution:**  
Check which attributes do not appear on the RHS of any FD:
- Attributes appearing on RHS: $\{B, C, D, E\}$.
- Attributes never appearing on RHS: $\{A, F\}$.
Therefore, $\{A, F\}$ must be part of every candidate key.  
Find $(AF)^+$:
$$(AF)^+ = \{A, F\} \cup \{B\} \cup \{C\} \cup \{D\} \cup \{E\} = \{A, B, C, D, E, F\} = R$$
Since $AF$ determines all attributes and neither $A$ alone nor $F$ alone can determine all attributes ($F$ appears nowhere in $F$, so $F$ cannot be derived), $AF$ is the unique minimal superkey.  
Number of candidate keys = 1.
</details>

**Q6 (MCQ).** Relation $R(A, B, C, D)$ with FDs $F = \{A \to B,\; B \to C,\; C \to D\}$ is decomposed into $R_1(A, B)$, $R_2(B, C)$, and $R_3(B, D)$. Which of the following is correct?  
(A) The decomposition is lossless and dependency preserving.  
(B) The decomposition is lossless but NOT dependency preserving.  
(C) The decomposition is NOT lossless but dependency preserving.  
(D) The decomposition is NEITHER lossless NOR dependency preserving.  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
*Lossless test:*
1. $R_1(A, B) \cap R_2(B, C) = \{B\}$. In $R_2$, $B \to C$ holds, so $B$ is a key of $R_2$. Joining $R_1$ and $R_2$ is lossless, producing $R_{12}(A, B, C)$.
2. $R_{12}(A, B, C) \cap R_3(B, D) = \{B\}$. In $R_3$, does $B \to D$ hold? Under $F$, $B^+ = \{B, C, D\}$, so $B \to D$ holds! Thus $B$ is a key of $R_3$, and joining $R_{12}$ and $R_3$ is lossless.  
$\implies$ Decomposition is **lossless**.

*Dependency preservation test:*
- $A \to B$ is in $R_1(A, B)$. Preserved.
- $B \to C$ is in $R_2(B, C)$. Preserved.
- Check $C \to D$: attributes $C$ and $D$ do not appear together in any sub-relation! Projecting $F$ onto $R_1, R_2, R_3$ yields FDs $\{A \to B\}, \{B \to C\}, \{B \to D\}$. In this union, $C^+ = \{C\}$, which cannot derive $D$ (since $C$ is on no LHS). Therefore $C \to D$ is lost.  
$\implies$ Decomposition is **NOT dependency preserving**.
</details>

**Q7 (NAT).** Given relational schema $R(A, B, C, D, E)$ with functional dependencies:
$$F = \{A \to BC,\; CD \to E,\; B \to D,\; E \to A\}$$
The total number of candidate keys for $R$ is _______.  
<details><summary>Answer</summary>

**Answer:** 4  
**Solution:**  
All attributes $\{A, B, C, D, E\}$ appear on the RHS of at least one FD. No attribute is left-only or right-only.  
Let us test closures of 1-attribute sets:
- $A^+ = \{A, B, C\}$. $B \to D \implies \{A, B, C, D\}$. $CD \to E \implies \{A, B, C, D, E\} = R$. $\implies \mathbf{A}$ is a CK.
- $E^+ = \{E\} \cup A^+ = R$. $\implies \mathbf{E}$ is a CK.
- $B^+ = \{B, D\} \neq R$.
- $C^+ = \{C\} \neq R$.
- $D^+ = \{D\} \neq R$.

Now test 2-attribute combinations that do not contain $A$ or $E$:
- $(BC)^+$: $B \to D \implies \{B, C, D\}$. $CD \to E \implies \{B, C, D, E\}$. $E \to A \implies \{A, B, C, D, E\} = R$. Neither $B$ nor $C$ is a key $\implies \mathbf{BC}$ is a CK.
- $(CD)^+$: $CD \to E \implies \{C, D, E\}$. $E \to A \implies \{C, D, E, A\}$. $A \to BC \implies \{A, B, C, D, E\} = R$. Neither $C$ nor $D$ is a key $\implies \mathbf{CD}$ is a CK.
- $(BD)^+ = \{B, D\} \neq R$.

No 3-attribute set can be minimal without containing one of the existing candidate keys ($A, E, BC, CD$).  
Thus, the candidate keys are $\{A, E, BC, CD\}$.  
Total number of candidate keys = 4.
</details>

**Q8 (MSQ).** Let relation $R(A, B, C, D, E)$ satisfy the functional dependencies $F = \{AB \to C,\; BC \to D,\; CD \to E,\; DE \to A\}$. Which of the following attribute sets is/are candidate keys of $R$?  
(A) $AB$  
(B) $BC$  
(C) $CD$  
(D) $DE$  
<details><summary>Answer</summary>

**Answer:** (A), (B)  
**Solution:**  
Trace the closure of each option:
- $(AB)^+$: $AB \to C \implies \{A, B, C\} \to D \implies \{A, B, C, D\} \to E \implies \{A, B, C, D, E\} = R$. Proper subsets: $A^+ = \{A\}, B^+ = \{B\} \neq R$. So $AB$ is a candidate key.
- $(BC)^+$: $BC \to D \implies \{B, C, D\} \to E \implies \{B, C, D, E\}$. Since $DE \subseteq \{B, C, D, E\}$ and $DE \to A$, we get $\{A, B, C, D, E\} = R$. Proper subsets: $B^+ = \{B\}, C^+ = \{C\} \neq R$. So $BC$ is a candidate key.
- $(CD)^+$: $CD \to E \implies \{C, D, E\} \to A$ (since $DE \to A$) $\implies \{A, C, D, E\}$. Now $AB$ requires $B$; but does $(CD)^+$ derive $B$? Wait! In $(CD)^+ = \{A, C, D, E\}$, does any FD derive $B$? Notice no FD has $B$ on the RHS except possibly through another attribute, but RHS attributes in $F$ are: $C$ (from $AB$), $D$ (from $BC$), $E$ (from $CD$), $A$ (from $DE$). $B$ NEVER appears on the RHS of ANY dependency in $F$!  
Therefore, **$B$ must be in every candidate key** of $R$!  
Thus, neither $CD$ nor $DE$ can be a candidate key, because neither contains $B$, and $B$ cannot be derived!  
Let us re-verify:
- $B$ appears only on LHS!
- Hence only sets containing $B$ can be candidate keys.
- $(AB)^+ = R$, and $AB$ is minimal $\implies AB$ is a candidate key.
- $(BC)^+$: contains $B$. Does $(BC)^+$ derive all attributes? Yes! $BC \to D \to E \to A \implies \{A, B, C, D, E\} = R$. Proper subsets: $B^+ = \{B\}, C^+ = \{C\} \neq R$. So $BC$ is a candidate key.
- For $CD$: $B \notin (CD)^+$. Not a key.
- For $DE$: $B \notin (DE)^+$. Not a key.  
Therefore, the only candidate keys among the options are (A) and (B).
</details>
