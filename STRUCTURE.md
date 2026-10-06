# Repository structure

Every file in this guide, in reading order. Each section folder contains a `README.md` (reading order, priorities, weightage, connections), topic chapters, a one-page `CHEATSHEET.md` for last-minute revision, and a `CHECKPOINT.md` of GATE-style questions with folded solutions.

**Paper tags:** `CS` = only in the CS paper · `DA` = only in the DA paper · `CS+DA` = shared preparation.
**Priority:** **P0** = asked almost every year, master it · **P1** = asked regularly, know it well · **P2** = occasional / low-yield, know the definitions and one example.

## Root files

File | Purpose
--- | ---
[README.md](README.md) | Start here: what this guide is and how to study with it
[ROADMAP.md](ROADMAP.md) | Dependency graph of subjects and the phase-wise study order
[CONNECTIONS.md](CONNECTIONS.md) | Cross-subject concept map: the same idea showing up in different subjects
[COVERAGE.md](COVERAGE.md) | Every topic in the Master Study Plan mapped to the chapter that teaches it, plus topics added beyond the plan
[EXAM-STRATEGY.md](EXAM-STRATEGY.md) | Exam pattern, marking scheme, weightage, PYQ method, exam-day tactics
[PROGRESS.md](PROGRESS.md) | Tick-box tracker mirroring the Master Study Plan
[CHEATSHEETS.md](CHEATSHEETS.md) | Index of every one-page cheat sheet for final revision

## Section manifest

### 00 · General Aptitude (CS+DA, 15 marks, added — missing from the plan)
- `00-general-aptitude/README.md`
- `00-general-aptitude/verbal-aptitude.md` — grammar, vocabulary, reading comprehension, narrative sequencing
- `00-general-aptitude/quantitative-aptitude.md` — data interpretation, ratios, percentages, powers/logs, P&C, series, mensuration, geometry, elementary statistics and probability
- `00-general-aptitude/analytical-aptitude.md` — deduction, induction, analogy, numerical relations, reasoning puzzles
- `00-general-aptitude/spatial-aptitude.md` — translation, rotation, scaling, mirroring, assembling, paper folding/cutting, 2D/3D patterns
- `00-general-aptitude/CHEATSHEET.md`, `00-general-aptitude/CHECKPOINT.md`

### 01 · Discrete Mathematics (CS)
- `01-discrete-mathematics/README.md`
- `01-discrete-mathematics/propositional-logic.md`
- `01-discrete-mathematics/first-order-logic.md`
- `01-discrete-mathematics/sets-relations-functions.md`
- `01-discrete-mathematics/posets-and-lattices.md`
- `01-discrete-mathematics/algebraic-structures.md` — monoids, groups
- `01-discrete-mathematics/graph-theory.md` — connectivity, matching, colouring
- `01-discrete-mathematics/combinatorics.md` — counting
- `01-discrete-mathematics/recurrences-and-generating-functions.md`
- `01-discrete-mathematics/CHEATSHEET.md`, `01-discrete-mathematics/CHECKPOINT.md`

### 02 · Probability & Statistics (CS+DA)
- `02-probability-statistics/README.md`
- `02-probability-statistics/probability-basics.md` — counting for probability, axioms, sample spaces, events, independence, mutual exclusivity, marginal/conditional/joint, Bayes
- `02-probability-statistics/random-variables-and-moments.md` — random variables, PMF/PDF/CDF, mean/median/mode, SD, covariance, correlation, conditional expectation/variance, conditional PDF
- `02-probability-statistics/discrete-distributions.md` — Bernoulli, binomial, discrete uniform, Poisson
- `02-probability-statistics/continuous-distributions.md` — continuous uniform, exponential, normal, standard normal, t, chi-squared
- `02-probability-statistics/statistical-inference.md` — CLT, confidence intervals, z-test, t-test, chi-squared test
- `02-probability-statistics/CHEATSHEET.md`, `02-probability-statistics/CHECKPOINT.md`

### 03 · Linear Algebra (CS+DA)
- `03-linear-algebra/README.md`
- `03-linear-algebra/vector-spaces.md` — vectors, vector spaces, subspaces, linear (in)dependence, basis, dimension, rank, nullity
- `03-linear-algebra/matrices-and-determinants.md` — matrix operations, determinants, partition matrices, idempotent matrices, quadratic forms
- `03-linear-algebra/linear-systems-and-lu.md` — systems of linear equations, Gaussian elimination, LU decomposition
- `03-linear-algebra/eigenvalues-and-eigenvectors.md`
- `03-linear-algebra/orthogonality-projections-svd.md` — orthogonal matrices, projections, projection matrices, SVD
- `03-linear-algebra/CHEATSHEET.md`, `03-linear-algebra/CHECKPOINT.md`

### 04 · Calculus & Optimization (CS+DA)
- `04-calculus-optimization/README.md`
- `04-calculus-optimization/limits-continuity-differentiability.md` — functions of a single variable, limits, continuity, differentiability
- `04-calculus-optimization/mean-value-theorems-and-taylor.md`
- `04-calculus-optimization/maxima-minima-optimization.md`
- `04-calculus-optimization/integration.md`
- `04-calculus-optimization/CHEATSHEET.md`, `04-calculus-optimization/CHECKPOINT.md`

### 05 · C Programming (CS)
- `05-c-programming/README.md`
- `05-c-programming/c-basics-and-expressions.md` — syntax, semantics, types, operators, precedence, storage classes, scope, tracing expressions
- `05-c-programming/pointers-arrays-strings.md` — pointers, arrays, strings, memory and pointer reasoning
- `05-c-programming/functions-recursion-structures.md` — functions, parameter passing, recursion, structures, unions, dynamic memory
- `05-c-programming/CHEATSHEET.md`, `05-c-programming/CHECKPOINT.md`

### 06 · Python Programming (DA)
- `06-python-programming/README.md`
- `06-python-programming/python-basics.md` — syntax, variables, expressions, iteration, functions, scope
- `06-python-programming/python-collections.md` — lists, tuples, dictionaries, sets, comprehensions
- `06-python-programming/python-oop-recursion-tracing.md` — classes/basic OOP, recursion, tracing programs
- `06-python-programming/CHEATSHEET.md`, `06-python-programming/CHECKPOINT.md`

### 07 · Data Structures (CS+DA)
- `07-data-structures/README.md`
- `07-data-structures/arrays-stacks-queues.md`
- `07-data-structures/linked-lists.md`
- `07-data-structures/trees-and-bst.md` — trees, binary trees, traversals, BSTs, AVL basics
- `07-data-structures/heaps.md` — binary heaps, priority queues
- `07-data-structures/graphs.md` — graph terminology and representations
- `07-data-structures/complexity-reference.md` — data-structure operations and complexity
- `07-data-structures/CHEATSHEET.md`, `07-data-structures/CHECKPOINT.md`

### 08 · Algorithms (CS+DA)
- `08-algorithms/README.md`
- `08-algorithms/asymptotic-analysis.md` — notation, growth rates, worst-case time/space, recurrences, Master theorem
- `08-algorithms/searching-and-sorting.md` — linear search, binary search, selection, bubble, insertion, merge, quick (+ heap, counting, radix)
- `08-algorithms/hashing.md`
- `08-algorithms/divide-and-conquer.md`
- `08-algorithms/greedy-algorithms.md`
- `08-algorithms/dynamic-programming.md`
- `08-algorithms/graph-traversals.md` — BFS, DFS, topological sort, connected components, SCC
- `08-algorithms/minimum-spanning-trees.md`
- `08-algorithms/shortest-paths.md`
- `08-algorithms/CHEATSHEET.md`, `08-algorithms/CHECKPOINT.md`

### 09 · Digital Logic (CS)
- `09-digital-logic/README.md`
- `09-digital-logic/boolean-algebra-and-minimization.md` — Boolean algebra, algebraic minimization, K-maps, tabular (Quine–McCluskey)
- `09-digital-logic/combinational-circuits.md`
- `09-digital-logic/sequential-circuits.md`
- `09-digital-logic/number-representation-and-arithmetic.md` — number systems, complements, fixed point, IEEE 754 floating point
- `09-digital-logic/CHEATSHEET.md`, `09-digital-logic/CHECKPOINT.md`

### 10 · Computer Organization & Architecture (CS)
- `10-computer-organization/README.md`
- `10-computer-organization/instruction-sets-and-addressing.md`
- `10-computer-organization/alu-and-control-unit.md` — ALU design, datapath, hardwired and microprogrammed control
- `10-computer-organization/memory-hierarchy-and-cache.md` — memory interfacing, hierarchy and performance, cache mapping
- `10-computer-organization/io-interrupts-dma.md`
- `10-computer-organization/pipelining.md` — instruction pipelining, pipeline hazards
- `10-computer-organization/CHEATSHEET.md`, `10-computer-organization/CHECKPOINT.md`

### 11 · Theory of Computation (CS)
- `11-theory-of-computation/README.md`
- `11-theory-of-computation/regular-languages-and-finite-automata.md` — regular expressions, DFA/NFA, minimization, regular languages, closure
- `11-theory-of-computation/context-free-languages-and-pda.md` — CFGs, PDAs, CFLs, closure
- `11-theory-of-computation/pumping-lemma.md`
- `11-theory-of-computation/turing-machines-and-undecidability.md`
- `11-theory-of-computation/CHEATSHEET.md`, `11-theory-of-computation/CHECKPOINT.md`

### 12 · Compiler Design (CS)
- `12-compiler-design/README.md`
- `12-compiler-design/lexical-analysis.md`
- `12-compiler-design/parsing.md` — FIRST/FOLLOW, LL(1), LR(0), SLR, LALR, CLR, operator precedence
- `12-compiler-design/syntax-directed-translation.md`
- `12-compiler-design/runtime-environments.md`
- `12-compiler-design/intermediate-code-generation.md`
- `12-compiler-design/optimization-and-dataflow.md` — local optimization, data-flow analysis, constant propagation, liveness, CSE
- `12-compiler-design/CHEATSHEET.md`, `12-compiler-design/CHECKPOINT.md`

### 13 · Operating Systems (CS)
- `13-operating-systems/README.md`
- `13-operating-systems/processes-threads-syscalls.md` — system calls, processes, threads, IPC
- `13-operating-systems/cpu-scheduling.md`
- `13-operating-systems/synchronization.md` — concurrency, critical sections, semaphores, monitors, classic problems
- `13-operating-systems/deadlocks.md`
- `13-operating-systems/memory-management.md` — allocation, paging, segmentation, TLB, multilevel page tables
- `13-operating-systems/virtual-memory.md` — demand paging, page replacement, thrashing
- `13-operating-systems/file-systems-and-disk-scheduling.md` — file systems, I/O (disk) scheduling
- `13-operating-systems/CHEATSHEET.md`, `13-operating-systems/CHECKPOINT.md`

### 14 · Databases & Data Warehousing (CS+DA)
- `14-databases/README.md`
- `14-databases/er-model.md`
- `14-databases/relational-model-algebra-calculus.md` — relational model, relational algebra, tuple calculus
- `14-databases/sql.md`
- `14-databases/constraints-and-normalization.md` — integrity constraints, keys, FDs, normal forms
- `14-databases/file-organization-and-indexing.md` — file organization, indexing, B trees, B+ trees
- `14-databases/transactions-and-concurrency.md`
- `14-databases/data-warehousing-and-preprocessing.md` — data types, normalization, discretization, sampling, compression, schemas, concept hierarchies, measures
- `14-databases/CHEATSHEET.md`, `14-databases/CHECKPOINT.md`

### 15 · Computer Networks (CS)
- `15-computer-networks/README.md`
- `15-computer-networks/layering-switching-performance.md` — layering, circuit/packet/virtual-circuit switching, performance metrics
- `15-computer-networks/data-link-layer.md` — framing, error detection, flow control, MAC, Ethernet
- `15-computer-networks/routing.md` — distance-vector, link-state
- `15-computer-networks/ipv4-addressing.md` — IPv4, fragmentation, CIDR, NAT
- `15-computer-networks/transport-layer-tcp.md` — TCP flow control, congestion control, UDP, sockets
- `15-computer-networks/application-layer.md` — DNS, HTTP
- `15-computer-networks/CHEATSHEET.md`, `15-computer-networks/CHECKPOINT.md`

### 16 · Machine Learning (DA)
- `16-machine-learning/README.md`
- `16-machine-learning/ml-foundations.md` — regression vs classification, bias–variance, cross-validation, evaluation
- `16-machine-learning/linear-and-logistic-regression.md` — simple, multiple, ridge, logistic regression
- `16-machine-learning/classification-methods.md` — k-NN, naive Bayes, LDA, SVM, decision trees
- `16-machine-learning/neural-networks.md` — MLP, feed-forward networks
- `16-machine-learning/clustering.md` — k-means, k-medoids, hierarchical (top-down, bottom-up, linkages)
- `16-machine-learning/dimensionality-reduction-pca.md`
- `16-machine-learning/CHEATSHEET.md`, `16-machine-learning/CHECKPOINT.md`

### 17 · Artificial Intelligence (DA)
- `17-artificial-intelligence/README.md`
- `17-artificial-intelligence/search.md` — uninformed, informed, adversarial search
- `17-artificial-intelligence/logic-and-inference.md` — propositional and predicate logic for AI
- `17-artificial-intelligence/probabilistic-reasoning.md` — conditional independence, Bayesian networks, variable elimination, sampling
- `17-artificial-intelligence/CHEATSHEET.md`, `17-artificial-intelligence/CHECKPOINT.md`
