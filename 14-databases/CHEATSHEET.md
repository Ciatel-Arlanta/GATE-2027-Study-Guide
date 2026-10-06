# Databases — revision sheet

| Concept | Recall |
|---|---|
| Candidate key | Minimal superkey |
| FD closure | Add RHS whenever LHS subset of current closure |
| 3NF | For X→A: X superkey or A prime |
| BCNF | Every determinant of nontrivial FD is a superkey |
| Lossless binary split | Common attributes determine one component |
| Selection / projection | $\sigma$ filters rows; $\pi$ chooses columns |
| Natural join | Equijoin on same-named attributes, duplicate join columns removed |
| Conflict | Same item, different transactions, at least one write |
| Conflict serializable | Precedence graph acyclic |
| Strict 2PL | Hold write locks until commit/abort |
| B+ tree | Ordered leaves support range scans |
| Hash index | Equality lookup; no natural range order |
| OLAP | Roll-up aggregate; drill-down detail; slice/dice filter; pivot reorient |

SQL NULL uses three-valued logic: comparisons with NULL are UNKNOWN; test using `IS NULL`. Selection and projection are set-based in relational algebra unless bag semantics are explicitly introduced.
