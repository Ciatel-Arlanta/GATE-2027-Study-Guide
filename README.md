# GATE 2027 Study Guide — CS + DA

A connected set of notes for **GATE 2027 Computer Science (CS)** and **Data Science & AI (DA)**, built on the topic list in the *GATE 2027 CS + DA Master Study Plan* and checked against the official IIT Madras 2027 syllabi. Each chapter teaches its topic from scratch, then compresses it into a quick-glance box and a one-page cheat sheet for revision.

Start with the [roadmap](ROADMAP.md). Use [CONNECTIONS.md](CONNECTIONS.md) to see how ideas cross subject boundaries, [COVERAGE.md](COVERAGE.md) to find where any plan topic is taught, [PROGRESS.md](PROGRESS.md) to tick topics off, and [EXAM-STRATEGY.md](EXAM-STRATEGY.md) for the paper pattern, weightage and PYQ method. [STRUCTURE.md](STRUCTURE.md) lists every file.

## How every chapter is laid out

Each chapter has the same sections, so you always know where to look:

Part | Use it for
--- | ---
Header line | Which paper (CS / DA / CS+DA), priority (P0–P2), which plan topics it covers, prerequisites
**Quick glance** | 5–10 bullets that compress the chapter. Read this first when revising.
Concept sections | Intuition → definition → fully worked example → summary table
Formulas and facts to memorise | One table to scan before a test
GATE traps | The specific mistakes that cost marks
Connections | Where the idea came from and where it comes back, in other subjects
Practice | GATE-style MCQ / MSQ / NAT questions; solutions are folded so you attempt first

Every section folder also has a `README.md` (reading order, priorities, weightage), a one-page `CHEATSHEET.md`, and a `CHECKPOINT.md` that mixes questions across the section.

## Priorities

- **P0 — Must master:** asked almost every year. Solve every PYQ on it.
- **P1 — Know well:** asked regularly. Solve the standard question types.
- **P2 — Know the basics:** occasional. Learn the definitions and one worked example.

## Sections

\# | Section | Paper | Approx. marks*
--- | --- | --- | ---
00 | [General Aptitude](00-general-aptitude/README.md) | CS+DA | 15 (fixed)
01 | [Discrete Mathematics](01-discrete-mathematics/README.md) | CS | 7–9
02 | [Probability & Statistics](02-probability-statistics/README.md) | CS+DA | CS 2–4 · DA 15–20
03 | [Linear Algebra](03-linear-algebra/README.md) | CS+DA | CS 2–3 · DA 8–12
04 | [Calculus & Optimization](04-calculus-optimization/README.md) | CS+DA | 2–5
05 | [C Programming](05-c-programming/README.md) | CS | 10–12 with DS
06 | [Python Programming](06-python-programming/README.md) | DA | part of DA's 10–15 for PDSA
07 | [Data Structures](07-data-structures/README.md) | CS+DA | (see 05)
08 | [Algorithms](08-algorithms/README.md) | CS+DA | CS 8–10
09 | [Digital Logic](09-digital-logic/README.md) | CS | 4–6
10 | [Computer Organization & Architecture](10-computer-organization/README.md) | CS | 8–10
11 | [Theory of Computation](11-theory-of-computation/README.md) | CS | 7–9
12 | [Compiler Design](12-compiler-design/README.md) | CS | 4–6
13 | [Operating Systems](13-operating-systems/README.md) | CS | 8–10
14 | [Databases & Data Warehousing](14-databases/README.md) | CS+DA | CS 7–8 · DA 6–8
15 | [Computer Networks](15-computer-networks/README.md) | CS | 7–9
16 | [Machine Learning](16-machine-learning/README.md) | DA | 15–20
17 | [Artificial Intelligence](17-artificial-intelligence/README.md) | DA | 8–12

\*Typical ranges from recent papers; weightage shifts by a few marks every year. See [EXAM-STRATEGY.md](EXAM-STRATEGY.md).

## What was added beyond the Master Study Plan

The plan follows the official 2027 CS and DA syllabi closely. One gap and a few implicit topics were filled in; the full audit is in [COVERAGE.md](COVERAGE.md).

- **General Aptitude** (15 marks in both papers) was missing entirely. It is now [section 00](00-general-aptitude/README.md).
- Several topics are not named in the syllabus but are asked routinely because they underpin listed ones. Examples: the Master theorem, sliding-window protocols, page replacement, serializability, LL/LR parsing tables, and Bayesian networks. They are taught inside the chapter they support.

## How to study

1. Follow the [roadmap](ROADMAP.md) order (it matches the plan's order).
2. For each chapter: read it, close it, rewrite the Quick glance from memory, then attempt the Practice questions before opening solutions.
3. Solve that topic's PYQs from `../PYQs Qbs/`, then tick it in [PROGRESS.md](PROGRESS.md).
4. After each section, take its `CHECKPOINT.md` cold. Score below 60% → re-read the chapters it points to.
5. In revision cycles, read only the `CHEATSHEET.md` files ([index](CHEATSHEETS.md)) and the GATE traps lists.
