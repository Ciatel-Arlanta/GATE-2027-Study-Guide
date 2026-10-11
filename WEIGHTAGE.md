# PYQ weightage

How much each subject is actually asked, counted from the PYQ books in `PYQs Qbs/` (sibling folder of this repo) (GATE CSE/IT 1987–2026 and GATE DA 2024–2026; UGC-NET excluded). Raw per-subtopic counts are in [tools/pyq-weightage-data.md](tools/pyq-weightage-data.md). Counted in October 2026; re-count when new papers are added to the PYQ books.

Each paper has 65 questions (10 General Aptitude + 55 subject). 2015–2020 and 2021–2026 both contain 10 CSE paper sets, so "Qs per CSE paper" is directly comparable. Counts are of questions the books tag to a subject, so treat them as close estimates, not official marks.

## Subjects ranked by recent CSE weight

Section | Qs per CSE paper 2015–20 | Qs per CSE paper 2021–26 | Trend | DA papers 2024–26 (total Qs) | All-time CSE Qs
--- | --- | --- | --- | --- | ---
[00 General Aptitude](00-general-aptitude/README.md) | 9.9 | 9.9 | steady | 30 | 268
[08 Algorithms](08-algorithms/README.md) | 9.0 | 7.5 | steady | 16 | 428
[14 Databases](14-databases/README.md) | 7.4 | 5.8 | falling | 21 | 339
[15 Computer Networks](15-computer-networks/README.md) | 4.5 | 5.3 | steady | 0 | 225
[10 Computer Organization](10-computer-organization/README.md) | 3.9 | 5.0 | rising | 0 | 251
[13 Operating Systems](13-operating-systems/README.md) | 5.0 | 5.0 | steady | 0 | 343
[01 Discrete Mathematics](01-discrete-mathematics/README.md) | 6.8 | 4.9 | falling | 2 | 407
[11 Theory of Computation](11-theory-of-computation/README.md) | 5.0 | 4.7 | steady | 0 | 293
[09 Digital Logic](09-digital-logic/README.md) | 4.0 | 4.5 | steady | 0 | 310
[12 Compiler Design](12-compiler-design/README.md) | 3.3 | 4.5 | rising | 0 | 242
[07 Data Structures](07-data-structures/README.md) | 6.4 | 3.4 | falling | 6 | 279
[02 Probability & Statistics](02-probability-statistics/README.md) | 2.1 | 2.6 | steady | 36 | 112
[05 C Programming](05-c-programming/README.md) | 3.8 | 2.5 | falling | 0 | 132
[03 Linear Algebra](03-linear-algebra/README.md) | 3.3 | 2.4 | falling | 20 | 121
[04 Calculus & Optimization](04-calculus-optimization/README.md) | 2.3 | 1.5 | falling | 13 | 79
[06 Python](06-python-programming/README.md) | 0.0 | 0.0 | - | 3 | 0
[16 Machine Learning](16-machine-learning/README.md) | 0.0 | 0.0 | - | 21 | 0
[17 Artificial Intelligence](17-artificial-intelligence/README.md) | 0.0 | 0.0 | - | 7 | 0

## How to use this

- Chapter-level and section-level weightage is marked in each chapter header (`PYQ weightage:` line) and on each numbered heading (`[PYQ: High|Medium|Low]`). Levels follow the thresholds in [tools/AUTHORING-GUIDE.md](tools/AUTHORING-GUIDE.md).
- Low DA counts for ML/AI/Python reflect that the DA paper only started in 2024, not that the topics are unimportant: in the DA paper they carry a large share of the marks.
- Algorithms and Data Structures counts fall in some years because the books tag many questions under the closest subtopic; read per-subtopic counts alongside the subject total.

[Roadmap](ROADMAP.md) · [Coverage](COVERAGE.md) · [Progress](PROGRESS.md) · [Exam strategy](EXAM-STRATEGY.md)
