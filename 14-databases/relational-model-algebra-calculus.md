# Relational Model, Relational Algebra and Tuple Calculus

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Relational model; Relational algebra; Tuple calculus
> **Prerequisites:** [ER model](er-model.md), [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md), [First-order logic](../01-discrete-mathematics/first-order-logic.md) · **Leads to:** [SQL](sql.md), [Constraints and normalization](constraints-and-normalization.md)

## Quick glance

- A **relation** is a **set** of tuples over a schema; **degree** = number of attributes, **cardinality** = number of tuples. Order of tuples and attributes does not matter; **no duplicate tuples** (in the pure model).
- Keys: **super key** (unique), **candidate key** (minimal super key), **primary key** (chosen one, no NULL), **alternate key** (unchosen candidates), **foreign key** (refers to a candidate/primary key of another relation).
- Relational algebra operators: **σ** select, **π** project (removes duplicates), **ρ** rename, **∪ ∩ −** (union-compatible), **×** product, **⋈** joins, **÷** division.
- Division: `R(A,B) ÷ S(B) = π_A(R) − π_A((π_A(R) × S) − R)`: "A-values paired with **all** B-values of S".
- Tuple counts for R (m tuples), S (n tuples): union max(m,n)..m+n; intersection 0..min(m,n); difference max(0,m−n)..m; product mn; join 0..mn.
- TRC: `{t | P(t)}`. Universal quantification: `∀x P` ≡ `¬∃x ¬P`; "for all" queries become **division** in RA and **double NOT EXISTS** in SQL.
- **Trap:** π removes duplicates, SQL SELECT does not; NULL never equals NULL in a join condition.

## 1. The relational model

Data is stored as **relations** (tables). Formally, a relation schema R(A1, ..., An) with domains D1, ..., Dn; a **relation instance** r is a **finite subset** of D1 × ... × Dn.

| Term | Meaning | Table analogy |
| --- | --- | --- |
| Relation | Set of tuples | Table |
| Tuple | One element of the relation | Row |
| Attribute | Named column | Column |
| Domain | Set of allowed values | Data type + constraints |
| Degree (arity) | Number of attributes | #columns |
| Cardinality | Number of tuples | #rows |

**Because a relation is a set**, there are no duplicate rows and no row order. (SQL tables are multisets and relax this, see [SQL](sql.md).)

### 1.1 Keys

For schema R and attribute set K:

- **Super key**: K such that no two tuples agree on all attributes of K.
- **Candidate key**: a super key with **no proper subset** that is a super key (minimal).
- **Primary key**: the candidate key chosen to identify tuples; its attributes cannot be NULL.
- **Alternate key**: a candidate key not chosen as primary.
- **Foreign key**: attributes FK in R1 referencing the primary (or candidate) key of R2; every non-NULL FK value must appear in R2.
- **Prime attribute**: appears in at least one candidate key.

**Counting super keys.** If a relation has n attributes and candidate keys K1..Kk, count subsets containing at least one Ki (inclusion-exclusion).

*Worked example.* R(A,B,C,D) with candidate keys {A} and {B,C}.

- Subsets containing A: 2^3 = 8.
- Subsets containing B and C: 2^2 = 4.
- Subsets containing A, B and C: 2^1 = 2.
- Super keys = 8 + 4 − 2 = **10**.

(Verified by enumeration.) Special cases: one candidate key with m attributes among n gives 2^(n−m) super keys; if **every** attribute set must be checked and the only key is all n attributes, there is exactly 1 super key.

### 1.2 NULL semantics

NULL means "unknown" or "not applicable". Any comparison with NULL (including NULL = NULL) yields **unknown**, not true. In a join or selection only tuples where the predicate is **true** pass. Primary key attributes must be non-NULL (entity integrity). A foreign key may be NULL (the reference is simply absent). See [SQL](sql.md) for the three-valued logic tables.

## 2. Relational algebra

A **procedural** query language: operators take relations and return relations, so they compose. **Pure RA treats relations as sets** (duplicates removed after every operation that could create them).

### 2.1 Running example

Used throughout this chapter (and in [SQL](sql.md)):

```text
Student(sid, name, dept, age)      Course(cid, title, credits)      Enroll(sid, cid, marks)
1 Asha   CS 20                      C1 DBMS 4                        1 C1 80    3 C2 NULL
2 Bala   EE 21                      C2 OS   3                        1 C2 70    5 C1 90
3 Chitra CS NULL                    C3 CN   3                        1 C3 60    5 C2 50
4 Dev    ME 22                                                       2 C1 60    5 C3 70
5 Esha   EE 21
```

### 2.2 Unary operators

| Operator | Notation | Meaning |
| --- | --- | --- |
| Selection | σ_condition(R) | Rows satisfying the condition; same schema |
| Projection | π_attrs(R) | Keep listed columns, **remove duplicate rows** |
| Rename | ρ_S(A1,...)(R) | Rename relation and/or attributes |

- σ_dept='CS'(Student) = {(1,Asha,CS,20), (3,Chitra,CS,NULL)}.
- σ_age>20(Student): the row with NULL age fails (unknown is not true). Result sids {2,4,5}.
- π_dept(Student) = {CS, EE, ME}: 3 tuples out of 5 because duplicates vanish.
- Selections commute: σ_c1(σ_c2(R)) = σ_c2(σ_c1(R)) = σ_{c1∧c2}(R). Cascade of projections: π_A(π_{A,B}(R)) = π_A(R).

### 2.3 Set operators

R ∪ S, R ∩ S, R − S require **union compatibility**: same degree and compatible domains position by position (result takes the first operand's attribute names).

- ∪ and ∩ are commutative and associative; − is neither.
- R ∩ S = R − (R − S).

### 2.4 Cartesian product and joins

- **R × S**: every row of R paired with every row of S; degree = sum, cardinality = m·n.
- **Theta join** R ⋈_θ S = σ_θ(R × S).
- **Equi-join**: θ uses only equality.
- **Natural join** R ⋈ S: equi-join on **all common attribute names**, with the duplicate columns **dropped**. If no common attribute, it equals R × S.

Worked: Student ⋈ Enroll (common attribute sid). Each Enroll row finds its student: 8 rows of degree 4 + 3 − 1 = 6 columns. Dev (4) matches nothing and is **lost** (dangling tuple).

**Outer joins** keep dangling tuples, padding with NULL:

| Join | Keeps unmatched rows of | Student–Enroll example |
| --- | --- | --- |
| Left outer ⟕ | Left | 8 + Dev with NULLs = 9 rows |
| Right outer ⟖ | Right | 8 rows (every Enroll row has a student) |
| Full outer ⟗ | Both | 9 rows |

### 2.5 Division

`R(A,B) ÷ S(B)` returns the A-values `a` such that **for every** tuple `b` in S, `(a,b)` is in R. Result schema = attributes of R not in S.

Standard expression (derive it): all A-values `π_A(R)` minus the "bad" ones, where a bad `a` is missing some b. Missing pairs: `(π_A(R) × S) − R`. So:

$$R \div S = \pi_A(R) - \pi_A\big((\pi_A(R)\times S) - R\big)$$

**Worked example: students who took all courses.** Let R = π_{sid,cid}(Enroll), S = π_{cid}(Course) = {C1,C2,C3}.

1. π_sid(R) = {1,2,3,5}.
2. π_sid(R) × S has 4 × 3 = 12 pairs.
3. Remove actual pairs of R (8 pairs): missing pairs = {(2,C2),(2,C3),(3,C1),(3,C3)}.
4. π_sid of missing = {2,3}.
5. {1,2,3,5} − {2,3} = **{1,5}** (Asha and Esha). The SQL in [SQL](sql.md) returns the same two students.

Note: Dev (4) never appears in Enroll, so he is not in π_sid(R); division over a student relation would need care: R ÷ S only ranges over A-values present in R. If S is empty, R ÷ S = π_A(R) (everyone vacuously qualifies).

### 2.6 Minimum and maximum tuples (GATE favourite)

R has m tuples, S has n tuples:

| Expression | Min | Max | Notes |
| --- | --- | --- | --- |
| σ(R) | 0 | m | |
| π(R) | 1 (if m>0) | m | Duplicates collapse |
| R ∪ S | max(m,n) | m+n | Min when one contains the other |
| R ∩ S | 0 | min(m,n) | |
| R − S | max(0, m−n) | m | |
| R × S | mn | mn | Exactly |
| R ⋈ S (natural, common attrs) | 0 | mn | Max when all join values equal |
| R ⋈ S, join on S's **key** = R's **FK** (FK NOT NULL) | m | m | Each R row matches exactly one S row |
| R ⟕ S (left outer) | m | mn (n≥1) | Every R row appears at least once |
| R ⟗ S (full outer) | max(m,n) | max(mn, m+n) | |
| R(A,B) ÷ S(B), n>0 | 0 | ⌊m/n⌋ | Each answer needs n tuples of R |

*Worked.* R has 100 tuples, S has 50 tuples, R.fk (NOT NULL) references S's key. Then |R ⋈ S| = **100**. Without the FK guarantee, range is 0..5000.

### 2.7 Equivalences worth knowing

- σ_c(R × S) = R ⋈_c S.
- σ_c(R ⋈ S) = σ_c(R) ⋈ S when c involves only R's attributes (push selection down).
- π_A(R ∪ S) = π_A(R) ∪ π_A(S), but π does **not** distribute over ∩ or −.
- Joins are commutative and associative (natural join).
- Intersection as join: R ∩ S = R ⋈ S for union-compatible R, S.
- Set-difference queries ("never", "not") need −; "all" queries need ÷ or double negation.

### 2.8 Typical query patterns

| Query | Algebra |
| --- | --- |
| Names of CS students | π_name(σ_dept='CS'(Student)) |
| Students who took DBMS | π_name(Student ⋈ Enroll ⋈ σ_title='DBMS'(Course)) |
| Students who took no course | π_sid(Student) − π_sid(Enroll) = {4} |
| Students who took C1 **and** C2 | π_sid(σ_cid='C1'(Enroll)) ∩ π_sid(σ_cid='C2'(Enroll)) = {1,5} |
| Students who took C1 **or** C3 | union |
| Students who took all courses | π_{sid,cid}(Enroll) ÷ π_cid(Course) = {1,5} |

"And" across rows of the same table cannot use a single σ (a row has one cid); use intersection or a self-join with ρ.

## 3. Tuple relational calculus (TRC)

Declarative: describe **what** to return, not how. A query is

$$\{\, t \mid P(t) \,\}$$

the set of tuples `t` for which formula `P` is true. A **tuple variable** ranges over a relation, written `Student(t)` (or `t ∈ Student`). Attribute access is `t.name`.

Atoms: `R(t)`, `t.A θ s.B`, `t.A θ constant`. Formulas: atoms combined with ∧ ∨ ¬, and quantifiers `∃s (…)`, `∀s (…)`.

### 3.1 Translations (worked)

| English | TRC | RA | SQL |
| --- | --- | --- | --- |
| Names of CS students | {t.name \| Student(t) ∧ t.dept='CS'} | π_name(σ_dept='CS') | SELECT name FROM Student WHERE dept='CS' |
| Students who took C1 | {t \| Student(t) ∧ ∃e (Enroll(e) ∧ e.sid=t.sid ∧ e.cid='C1')} | π(Student ⋈ σ_cid='C1'(Enroll)) | `EXISTS` or join |
| Students with no enrollment | {t \| Student(t) ∧ ¬∃e (Enroll(e) ∧ e.sid=t.sid)} | π_sid(Student) − π_sid(Enroll) | NOT EXISTS |
| Students who took **all** courses | {t \| Student(t) ∧ ∀c (Course(c) → ∃e (Enroll(e) ∧ e.sid=t.sid ∧ e.cid=c.cid))} | ÷ | double NOT EXISTS |

### 3.2 Universal quantification pattern

Use the implication form and its equivalents:

- `∀c (P(c) → Q(c))` ≡ `∀c (¬P(c) ∨ Q(c))` ≡ `¬∃c (P(c) ∧ ¬Q(c))`.
- English "for all courses, the student took it" becomes "there is **no** course the student did **not** take" = double negation in SQL ([SQL](sql.md)).

**Trap:** writing `∀c (Course(c) ∧ …)` is wrong; the universal needs the **implication** (`Course(c) → …`), because otherwise it asserts that **every possible tuple c** is a course.

### 3.3 Safe expressions

An expression is **safe** if it produces a finite result and only values from the database. `{t | ¬Student(t)}` is **unsafe**: infinitely many tuples are not students. Safe TRC has the same expressive power as relational algebra (**Codd's theorem**): every safe TRC query has an equivalent RA query and vice versa. SQL (without aggregates and recursion) is also at this level of power; **transitive closure cannot be expressed** in basic RA/TRC.

### 3.4 Domain relational calculus (DRC)

Variables range over **attribute values**, not tuples: `{<n, d> | ∃s ∃a (Student(s, n, d, a) ∧ a > 20)}`. Equivalent in power to safe TRC and RA. QBE is based on it.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Degree/cardinality | #columns / #rows | Output-size questions |
| Super keys | 2^(n−\|K\|) for one key K; inclusion-exclusion for several | Counting super keys |
| Product size | m·n rows, deg R + deg S columns | Size questions |
| Natural join columns | deg R + deg S − #common | Schema of a join |
| Division | π_A(R) − π_A((π_A(R) × S) − R) | "All" queries |
| Intersection | R − (R − S) | Rewrite without ∩ |
| FK join size | exactly \|R\| when FK NOT NULL | Max/min join questions |
| Complete operator set | σ, π, ∪, −, ×, ρ | Others derivable |
| Codd | Safe TRC = RA | Expressive-power questions |

## GATE traps

- **π removes duplicates** in RA. Min/max tuple questions assume sets. SQL does not (see [SQL](sql.md)).
- **Natural join with no common attribute is a Cartesian product.** With common attributes the result can have *fewer* tuples than either input.
- **Dangling tuples vanish in inner joins** but stay in outer joins. Left outer join size is at least m.
- **Min size of R ∪ S is max(m,n)**, not 0; min of R − S is max(0, m−n).
- **Division only ranges over A-values that occur in R.** Entities with no tuples in R never appear.
- **σ cannot compare two rows.** "Took C1 and C2" needs ∩ or a self-join.
- **Unsafe TRC** (like {t | ¬R(t)}) is not a valid relational query.
- **Relation = set:** no repeated tuples and no order; duplicate-based questions belong to SQL multisets.
- **Equal degree is not enough for union compatibility;** domains must match too.

## Connections

- [ER model](er-model.md): tables and FK/PK relationships that RA joins traverse.
- [SQL](sql.md): SELECT-FROM-WHERE = σ π ×; EXISTS corresponds to TRC ∃; NOT EXISTS pairs encode ÷.
- [Constraints and normalization](constraints-and-normalization.md): lossless-join decomposition means natural join of the projections gives the original relation.
- [First-order logic](../01-discrete-mathematics/first-order-logic.md): TRC is first-order logic over relations; quantifier negation rules apply directly.
- [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md): relations as subsets of products; set operators and union compatibility.
- [File organisation and indexing](file-organization-and-indexing.md): join size and selectivity estimates drive access-path choice.

## Practice

**Q1 (NAT).** R has 12 tuples and S has 7 tuples, union compatible. What is the minimum possible number of tuples in R ∪ S minus the maximum possible in R ∩ S?

<details><summary>Answer</summary>

**Answer:** 5.
**Solution:** Min |R ∪ S| = max(12,7) = 12 (S ⊆ R). Max |R ∩ S| = min(12,7) = 7. 12 − 7 = 5.

</details>

**Q2 (NAT).** Relation R(A,B,C,D) has candidate keys {A} and {B,C}. How many super keys does it have?

<details><summary>Answer</summary>

**Answer:** 10.
**Solution:** Supersets of A: 8; of BC: 4; of ABC: 2. 8 + 4 − 2 = 10.

</details>

**Q3 (MCQ).** Orders(oid, cust_id) with cust_id NOT NULL referencing Customers(cust_id), the key of Customers. |Orders| = 500, |Customers| = 80. The number of tuples in Orders ⋈ Customers is (a) 80 (b) 500 (c) 40000 (d) between 0 and 500

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** Every order matches exactly one customer (FK NOT NULL, key on the other side), so the join has 500 tuples.

</details>

**Q4 (MCQ).** Which of the following does NOT hold in general? (a) σ_c(R ∪ S) = σ_c(R) ∪ σ_c(S) (b) π_A(R ∩ S) = π_A(R) ∩ π_A(S) (c) σ_c(R − S) = σ_c(R) − σ_c(S) (d) R ⋈ S = S ⋈ R

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** Take R = {(1,x)}, S = {(1,y)}: R ∩ S = ∅ so the left side is ∅, but π_A(R) ∩ π_A(S) = {1}. The other three are valid identities.

</details>

**Q5 (NAT).** Enroll(sid, cid) has the pairs (1,C1),(1,C2),(2,C1),(3,C1),(3,C2),(3,C3). Course(cid) = {C1,C2}. How many tuples are in Enroll ÷ Course?

<details><summary>Answer</summary>

**Answer:** 2.
**Solution:** Student 1 has C1,C2 (yes); student 2 lacks C2 (no); student 3 has C1,C2,C3 ⊇ {C1,C2} (yes). Result {1,3}, so 2.

</details>

**Q6 (MCQ).** A TRC query `{t | ∀c (Course(c) ∧ ∃e (Enroll(e) ∧ e.sid = t.sid ∧ e.cid = c.cid))}` is intended to find students taking all courses. What is wrong?
(a) It is correct. (b) It uses ∀ with ∧ instead of →, so it demands every tuple c be a Course. (c) It is unsafe because it uses ∃. (d) t is never constrained to Student, but the quantifier is fine.

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** `∀c (Course(c) ∧ …)` asserts that **every** tuple c is a Course, which is false for any c outside Course. The correct form is `∀c (Course(c) → …)`. (Missing `Student(t)` is also a (safety) defect, but (b) names the logical error.)

</details>

**Q7 (NAT).** R(A,B) has m = 20 tuples and S(B) has n = 6 tuples. What is the maximum number of tuples in R ÷ S?

<details><summary>Answer</summary>

**Answer:** 3.
**Solution:** Each qualifying A-value needs 6 distinct tuples (a,b) in R, so at most ⌊20/6⌋ = 3.

</details>

**Q8 (MSQ).** For union-compatible R (m tuples) and S (n tuples) with m, n > 0, which are always true?
(a) |R ∪ S| ≥ max(m,n) (b) |R − S| ≥ m − n (c) |R ∩ S| ≤ min(m,n) (d) |R × S| = m·n

<details><summary>Answer</summary>

**Answer:** (a), (b), (c), (d).
**Solution:** (a) union contains each. (b) If m − n < 0 trivially true; otherwise at most n tuples of R are removed. (c) intersection is within each. (d) product always has mn rows (sets, with distinct tuple pairs).

</details>
