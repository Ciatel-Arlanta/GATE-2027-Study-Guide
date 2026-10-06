# SQL

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** SQL
> **Prerequisites:** [Relational model, algebra, calculus](relational-model-algebra-calculus.md) · **Leads to:** [Constraints and normalization](constraints-and-normalization.md), [Transactions and concurrency](transactions-and-concurrency.md)

## Quick glance

- **Logical evaluation order:** `FROM` (and joins) → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY`. Aliases defined in SELECT are not visible in WHERE.
- SQL tables are **multisets**: `SELECT` keeps duplicates unless `DISTINCT`; `UNION/INTERSECT/EXCEPT` remove duplicates unless `ALL`.
- **Three-valued logic:** comparison with NULL is *unknown*; `WHERE` keeps only **true** rows. `x NOT IN (…NULL…)` never returns true.
- **Aggregates ignore NULL** (except `COUNT(*)`). `SUM/AVG/MAX/MIN` over an empty or all-NULL group give NULL; `COUNT` gives 0.
- `WHERE` filters rows before grouping; `HAVING` filters **groups** (can use aggregates).
- `x > ALL (empty set)` is **true**; `x > ANY (empty set)` is **false**; `EXISTS (empty)` is false.
- "For all" queries = **double NOT EXISTS** (or GROUP BY + HAVING COUNT).
- **Trap:** `COUNT(*)` after a LEFT JOIN counts the NULL-padded row as 1; `COUNT(col)` counts 0.

## 1. Sample data used in every example

All outputs below were produced by running the queries in SQLite (ANY/ALL results use standard SQL semantics, SQLite lacks those operators).

```text
Student(sid, name, dept, age)     Course(cid, title, credits)    Enroll(sid, cid, marks)
1 Asha   CS 20                    C1 DBMS 4                      (1,C1,80) (1,C2,70) (1,C3,60)
2 Bala   EE 21                    C2 OS   3                      (2,C1,60) (3,C2,NULL)
3 Chitra CS NULL                  C3 CN   3                      (5,C1,90) (5,C2,50) (5,C3,70)
4 Dev    ME 22
5 Esha   EE 21
T(a): 1, 2, NULL     U(a): 2, 2, 3     E(a): empty
```

## 2. DDL and DML

```sql
CREATE TABLE Student (
  sid  INT PRIMARY KEY,
  name VARCHAR(30) NOT NULL,
  dept CHAR(2) DEFAULT 'CS',
  age  INT CHECK (age > 0)
);
CREATE TABLE Enroll (
  sid INT REFERENCES Student(sid) ON DELETE CASCADE,
  cid CHAR(2),
  marks INT,
  PRIMARY KEY (sid, cid)
);
ALTER TABLE Student ADD email VARCHAR(40);
DROP TABLE Enroll;              -- removes structure and data
TRUNCATE TABLE Student;         -- removes all rows, keeps structure
INSERT INTO Student VALUES (6,'Farid','CS',22);
UPDATE Student SET age = age + 1 WHERE dept = 'EE';
DELETE FROM Student WHERE age IS NULL;   -- structure remains
```

| Command class | Statements |
| --- | --- |
| DDL | CREATE, ALTER, DROP, TRUNCATE |
| DML | SELECT, INSERT, UPDATE, DELETE |
| DCL | GRANT, REVOKE |
| TCL | COMMIT, ROLLBACK, SAVEPOINT |

`DELETE` removes rows (can have WHERE, can roll back); `DROP` removes the table; `TRUNCATE` empties it.

## 3. SELECT and evaluation order

```sql
SELECT   dept, COUNT(*) AS n, AVG(age)
FROM     Student
WHERE    sid > 0
GROUP BY dept
HAVING   COUNT(*) > 1
ORDER BY dept;
```

Execution: **FROM** builds the row source → **WHERE** drops rows → **GROUP BY** forms groups → **HAVING** drops groups → **SELECT** computes output columns (aggregates are evaluated per group) → **ORDER BY** sorts (the only clause that may use SELECT aliases).

Rule: every column in SELECT must be a group-by column or inside an aggregate (when GROUP BY or any aggregate is present).

### Worked example 1: aggregates and NULL

`SELECT COUNT(*), COUNT(age), AVG(age), SUM(age), MAX(age) FROM Student;`

- COUNT(*) = 5 (all rows). COUNT(age) = 4 (Chitra's NULL skipped).
- SUM(age) = 20+21+22+21 = 84; AVG = 84 / **4** = 21.0 (denominator counts non-NULLs only).
- MAX = 22.
- Output: **(5, 4, 21.0, 84, 22)**.

### Worked example 2: GROUP BY and HAVING

`SELECT dept, COUNT(*), COUNT(age), AVG(age) FROM Student GROUP BY dept;`

| dept | COUNT(*) | COUNT(age) | AVG(age) |
| --- | --- | --- | --- |
| CS | 2 | 1 | 20.0 |
| EE | 2 | 2 | 21.0 |
| ME | 1 | 1 | 22.0 |

Adding `HAVING COUNT(*) > 1` keeps CS and EE only. `WHERE COUNT(*) > 1` would be a **syntax error** (aggregates are not allowed in WHERE).

## 4. NULLs and three-valued logic

| p | q | p AND q | p OR q | NOT p |
| --- | --- | --- | --- | --- |
| T | U | U | T | F |
| F | U | F | U | T |
| U | U | U | U | U |

(U = unknown.) A WHERE clause keeps a row only if the predicate is **T**.

- `SELECT * FROM Student WHERE age <> 21` returns Asha and Dev only (Chitra's NULL age gives unknown).
- `WHERE age = 21 OR age <> 21` returns **4** rows, not 5: Chitra is still missing.
- Use `IS NULL` / `IS NOT NULL` to test NULL.
- `NULL = NULL` is unknown. In `GROUP BY`, `DISTINCT`, and set operations (UNION etc.), NULLs **are treated as equal** to each other.
- `COUNT(DISTINCT dept)` = 3.

## 5. Joins

```sql
SELECT s.name, e.cid
FROM Student s LEFT JOIN Enroll e ON s.sid = e.sid;
```

Result (9 rows): Asha C1, C2, C3; Bala C1; Chitra C2; **Dev NULL**; Esha C1, C2, C3.

- `INNER JOIN` gives 8 rows (Dev lost). `FULL JOIN` here also 9.
- `FROM Student, Enroll` with no condition = Cartesian product: 5 × 8 = **40** rows.
- `NATURAL JOIN` joins on all same-named columns. `USING (sid)` joins on listed columns.
- A **condition on the right table placed in WHERE** after a LEFT JOIN turns it into an inner join (the NULL-padded row fails the predicate); put it in `ON` to keep the outer behaviour.

### Worked example 3: COUNT after outer join

`SELECT s.name, COUNT(e.cid) FROM Student s LEFT JOIN Enroll e ON s.sid=e.sid GROUP BY s.sid;`
→ Asha 3, Bala 1, Chitra 1, **Dev 0**, Esha 3.

Replacing `COUNT(e.cid)` by `COUNT(*)` gives **Dev 1**: the padded row exists, though no enrollment does.

## 6. Subqueries

### 6.1 IN, NOT IN, EXISTS, NOT EXISTS

- `sid NOT IN (SELECT sid FROM Enroll)` returns {4}.
- `NOT EXISTS (SELECT * FROM Enroll e WHERE e.sid = s.sid)` returns {4} as well.
- **NULL trap:** `SELECT sid FROM Student WHERE sid NOT IN (SELECT a FROM T)` where T contains NULL returns **nothing**: for each sid, `sid <> NULL` is unknown, so the whole conjunction is never true. `IN` over the same set returns {1,2} (matches found). `NOT EXISTS` form is immune to this.

### 6.2 ANY / ALL / SOME

`x θ ANY (S)` is true if θ holds for **some** element; `x θ ALL (S)` for **every** element. `= ANY` ≡ `IN`; `<> ALL` ≡ `NOT IN`.

| Expression (T = {1,2,NULL}) | Result | Reason |
| --- | --- | --- |
| `a > ALL (E)` with E empty | true for **all 3 rows** | Vacuous truth |
| `a > ANY (E)` with E empty | false for all rows | No witness |
| `a > ALL (U)`, U = {2,2,3} | no row | 1>2 false, 2>2 false, NULL gives unknown |
| `a > ANY (U)` | row 2 (2>... false), none: nothing exceeds 3 or 2 except none | a=1: no; a=2: 2>2 no; a=NULL unknown, so 0 rows |

Rule of thumb: `> ALL (S)` ≡ `> MAX(S)` (but true for empty S, while `MAX` of empty is NULL so the comparison is unknown).

### 6.3 Correlated subqueries

The inner query references the outer row and is **re-evaluated per outer row**.

`SELECT name FROM Student s WHERE age > (SELECT AVG(age) FROM Student WHERE dept = s.dept);`

- CS: AVG = 20.0 (NULL ignored): Asha 20 > 20? No. Chitra NULL: unknown.
- EE: AVG 21: Bala 21, Esha 21: no. ME: AVG 22: Dev 22: no.
- Output: **empty**.

### 6.4 "For all" queries

**Students who took every course (double NOT EXISTS):**

```sql
SELECT s.name FROM Student s
WHERE NOT EXISTS (
  SELECT * FROM Course c
  WHERE NOT EXISTS (
    SELECT * FROM Enroll e WHERE e.sid = s.sid AND e.cid = c.cid));
```

Read as: no course exists that the student has not taken. Output: **Asha, Esha** (this is relational division).

**Counting alternative:** `SELECT sid FROM Enroll GROUP BY sid HAVING COUNT(*) = (SELECT COUNT(*) FROM Course)` returns sids 1 and 5 (valid when (sid, cid) is unique).

### 6.5 Second-highest and similar

`SELECT MAX(marks) FROM Enroll WHERE marks < (SELECT MAX(marks) FROM Enroll);` = 90 is the max, so the answer is **80**.

## 7. Set operations

| Operator | Duplicates | Multiplicity rule (m, n copies) |
| --- | --- | --- |
| UNION | removed | 1 if m+n > 0 |
| UNION ALL | kept | m + n |
| INTERSECT | removed | 1 if both > 0 |
| INTERSECT ALL | kept | min(m, n) |
| EXCEPT | removed | 1 if m > 0 and n = 0 |
| EXCEPT ALL | kept | max(m − n, 0) |

With U = {2,2,3}, T = {1,2,NULL}: `U UNION T` = {1,2,3,NULL}; `U UNION ALL T` has 6 rows; `U INTERSECT T` = {2}; `U EXCEPT T` = {3}; `U EXCEPT ALL T` = {2,3} (one 2 cancelled). The operands need compatible columns.

## 8. Aggregation and grouping details

- `SELECT sid, SUM(marks) FROM Enroll GROUP BY sid` → (1,210), (2,60), (3,**NULL**), (5,210). Chitra's only mark is NULL, so SUM is NULL, not 0.
- `GROUP BY sid HAVING AVG(marks) > 65` → sids **1 and 5** (averages 70; Bala 60; Chitra NULL, excluded as unknown).
- `SELECT COUNT(*) FROM Student, Enroll` = 40.
- `SELECT DISTINCT cid FROM Enroll` = {C1,C2,C3}.
- Aggregate over an empty relation: `COUNT(*)` = 0, `SUM/AVG/MAX/MIN` = NULL.
- Cannot nest aggregate functions directly (`MAX(AVG(x))`) in standard SQL.
- `ORDER BY` default is ASC; NULL ordering is implementation-defined.

## 9. Views

```sql
CREATE VIEW EEStudents AS SELECT sid, name FROM Student WHERE dept = 'EE';
```

A view is a **stored query**, not stored data (unless materialised). Queries on a view are rewritten onto base tables. Updatable only if it is defined on one base table without aggregates, DISTINCT or GROUP BY and includes the key; `WITH CHECK OPTION` rejects updates that would make a row leave the view. Views provide security and logical data independence.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Evaluation order | FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY | Alias/aggregate placement |
| Aggregates | Ignore NULL except COUNT(*) | Output-tracing |
| Empty group/set | COUNT = 0; SUM, AVG, MAX, MIN = NULL | Edge cases |
| `> ALL (empty)` | true | Vacuous truth |
| `NOT IN` with NULL | never true | Subquery traps |
| UNION vs UNION ALL | distinct vs multiset | Row counts |
| INTERSECT ALL / EXCEPT ALL | min / max(m−n,0) | Multiset counts |
| Cartesian product size | m × n | FROM with two tables, no join |
| Left join rows | ≥ rows in the left table | Outer-join counts |
| Division | double NOT EXISTS or HAVING COUNT | All-of queries |

## GATE traps

- **Aggregate in WHERE** is illegal; use HAVING.
- **AVG divides by non-NULL count**, not by COUNT(*).
- **NOT IN with a NULL in the subquery returns an empty result.** NOT EXISTS is safe.
- **COUNT(\*) vs COUNT(col)** after outer joins: padded rows count in the former only.
- **`age = 21 OR age <> 21` is not a tautology** with NULLs.
- **`UNION` removes duplicates** and costs a sort/hash; `UNION ALL` does not. Row counts differ.
- **Cartesian product with no join condition** multiplies row counts; count it before filtering.
- **SELECT without DISTINCT keeps duplicates**, unlike relational algebra's π.
- **Correlated subqueries are evaluated per outer row**: trace for each row, not once.
- **Filtering the right table in WHERE after LEFT JOIN** discards the NULL-padded rows.
- `> ALL` over an empty subquery is **true**; `> ANY` is false.
- `DELETE` vs `DROP` vs `TRUNCATE`: rows only, whole table, all rows without row-by-row logging.

## Connections

- [Relational algebra and calculus](relational-model-algebra-calculus.md): SELECT-FROM-WHERE is σ, π, ×; EXISTS is ∃; double NOT EXISTS is ÷; but SQL keeps duplicates.
- [Constraints and normalization](constraints-and-normalization.md): PRIMARY KEY, FOREIGN KEY, ON DELETE actions, CHECK, triggers are declared in DDL.
- [Transactions and concurrency](transactions-and-concurrency.md): COMMIT/ROLLBACK, isolation levels, locking on rows.
- [File organisation and indexing](file-organization-and-indexing.md): indexes speed up WHERE and joins; the optimizer chooses plans.
- [Data warehousing](data-warehousing-and-preprocessing.md): GROUP BY CUBE/ROLLUP computes cuboids; star-schema queries join a fact table to dimensions.
- [Python collections](../06-python-programming/python-collections.md): GROUP BY resembles grouping with dictionaries in data-analysis code.

## Practice

**Q1 (NAT).** Using the Student table above, what does `SELECT COUNT(*) FROM Student WHERE age <> 21` return?

<details><summary>Answer</summary>

**Answer:** 2.
**Solution:** Ages 20 and 22 satisfy; 21 fails; NULL gives unknown and is dropped. Rows: Asha, Dev.

</details>

**Q2 (NAT).** `SELECT COUNT(*) FROM Student s, Enroll e WHERE s.sid = e.sid` on the sample data.

<details><summary>Answer</summary>

**Answer:** 8.
**Solution:** Every Enroll row (8 rows) matches exactly one student. Dev has none.

</details>

**Q3 (MCQ).** For T = {1, 2, NULL}, how many rows does `SELECT * FROM T WHERE a NOT IN (SELECT a FROM U)` return where U = {2,2,3}? (a) 0 (b) 1 (c) 2 (d) 3

<details><summary>Answer</summary>

**Answer:** (b) 1.
**Solution:** a=1: not equal to 2, not equal to 3: true → kept. a=2: matches → false. a=NULL: unknown. U has no NULL, so only 1 survives.

</details>

**Q4 (MCQ).** Which query returns students who took at least one course but **not** C1?
(a) `SELECT sid FROM Enroll WHERE cid <> 'C1'`
(b) `SELECT sid FROM Enroll EXCEPT SELECT sid FROM Enroll WHERE cid = 'C1'`
(c) `SELECT sid FROM Enroll WHERE cid <> 'C1' GROUP BY sid`
(d) both (a) and (c)

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** (a), (c) return anyone with some non-C1 enrollment, including students who took C1 too (1 and 5). (b) removes students with C1: leaves {3}. Chitra (3) took only C2.

</details>

**Q5 (NAT).** `SELECT s.name, COUNT(*) FROM Student s LEFT JOIN Enroll e ON s.sid = e.sid GROUP BY s.sid` returns how many output rows with count equal to 1?

<details><summary>Answer</summary>

**Answer:** 3.
**Solution:** Counts: Asha 3, Bala 1, Chitra 1, Dev 1 (padded row), Esha 3. Three rows have 1.

</details>

**Q6 (MSQ).** Which are true of `SELECT dept, AVG(age) FROM Student GROUP BY dept HAVING COUNT(age) >= 2`?
(a) Returns EE (b) Returns CS (c) Returns ME (d) Returns exactly one row

<details><summary>Answer</summary>

**Answer:** (a), (d).
**Solution:** COUNT(age): CS 1 (Chitra's NULL), EE 2, ME 1. Only EE passes: (EE, 21.0).

</details>

**Q7 (MCQ).** `SELECT COUNT(*) FROM T WHERE a > ALL (SELECT a FROM E)` where E is empty and T = {1, 2, NULL}. The result is (a) 0 (b) 2 (c) 3 (d) NULL

<details><summary>Answer</summary>

**Answer:** (c) 3.
**Solution:** `> ALL` over an empty set is vacuously true for every row, including the one with NULL.

</details>

**Q8 (NAT).** Using the sample data, the query `SELECT COUNT(*) FROM (SELECT sid FROM Enroll GROUP BY sid HAVING AVG(marks) > 65)` returns?

<details><summary>Answer</summary>

**Answer:** 2.
**Solution:** Averages: sid1 = (80+70+60)/3 = 70; sid2 = 60; sid3 = NULL (unknown, excluded); sid5 = (90+50+70)/3 = 70. Two groups pass.

</details>
