# Coverage audit

Every topic in the *GATE 2027 CS + DA Master Study Plan*, mapped to the chapter that teaches it. Both official GATE 2027 syllabi (CS and DA, IIT Madras) were checked against the plan.

## Audit result

- **Missing from the plan:** General Aptitude (15 marks in both papers). It is now [section 00](00-general-aptitude/README.md).
- **Everything else in the official CS and DA syllabi is in the plan.** The plan's topic list follows the 2027 syllabus wording closely.
- **Extra items in the plan were kept as instructed.** Some are tagged CS+DA but appear in only one official syllabus. Data types, data-transformation normalization, discretization, sampling, compression, warehouse schemas, concept hierarchies and measures are **DA-only**. Mean value theorem and integration are **CS-only**. Projection/orthogonal/idempotent/partition matrices, quadratic forms, Gaussian elimination, rank, nullity, projections and SVD are **DA-only** (CS lists only matrices, determinants, systems of equations, eigen-things and LU). Most of probability (t / chi-squared distributions, CLT, confidence intervals, tests, covariance/correlation, conditional expectation/variance, conditional PDF) is **DA-only**. CS lists random variables, the uniform/normal/exponential/Poisson/binomial distributions, mean/median/mode/SD, conditional probability and Bayes. CS students can treat those chapters as P2.
- **2027 changes to note:** the CS networks syllabus is narrower than in earlier years. It names layering; switching and performance metrics; error detection, MAC and Ethernet; DV/LS routing; IPv4 fragmentation, CIDR and NAT; TCP flow/congestion control and sockets; DNS and HTTP. ARP/DHCP/ICMP, UDP, SMTP/FTP and framing/bridging are no longer named. Older PYQs on those topics are lower priority.

## Plan topics → chapters

Plan group | Paper | Topic | Taught in | Priority
--- | --- | --- | --- | ---
Discrete Mathematics | CS | Propositional logic | [propositional-logic](01-discrete-mathematics/propositional-logic.md) | P0
Discrete Mathematics | CS | First-order logic | [first-order-logic](01-discrete-mathematics/first-order-logic.md) | P0
Discrete Mathematics | CS | Sets | [sets-relations-functions](01-discrete-mathematics/sets-relations-functions.md) | P1
Discrete Mathematics | CS | Relations | [sets-relations-functions](01-discrete-mathematics/sets-relations-functions.md) | P0
Discrete Mathematics | CS | Functions | [sets-relations-functions](01-discrete-mathematics/sets-relations-functions.md) | P0
Discrete Mathematics | CS | Partial orders | [posets-and-lattices](01-discrete-mathematics/posets-and-lattices.md) | P1
Discrete Mathematics | CS | Lattices | [posets-and-lattices](01-discrete-mathematics/posets-and-lattices.md) | P1
Discrete Mathematics | CS | Monoids | [algebraic-structures](01-discrete-mathematics/algebraic-structures.md) | P1
Discrete Mathematics | CS | Groups | [algebraic-structures](01-discrete-mathematics/algebraic-structures.md) | P0
Discrete Mathematics | CS | Graphs: connectivity | [graph-theory](01-discrete-mathematics/graph-theory.md) | P0
Discrete Mathematics | CS | Graphs: matching | [graph-theory](01-discrete-mathematics/graph-theory.md) | P1
Discrete Mathematics | CS | Graphs: colouring | [graph-theory](01-discrete-mathematics/graph-theory.md) | P0
Discrete Mathematics | CS | Combinatorics: counting | [combinatorics](01-discrete-mathematics/combinatorics.md) | P0
Discrete Mathematics | CS | Recurrence relations | [recurrences-and-generating-functions](01-discrete-mathematics/recurrences-and-generating-functions.md) | P0
Discrete Mathematics | CS | Generating functions | [recurrences-and-generating-functions](01-discrete-mathematics/recurrences-and-generating-functions.md) | P1
Probability & Statistics | CS+DA | Counting: permutations and combinations | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Probability axioms | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Sample spaces and events | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Independent events | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Mutually exclusive events | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Marginal probability | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Conditional probability | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Joint probability | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Bayes theorem | [probability-basics](02-probability-statistics/probability-basics.md) | P0
Probability & Statistics | CS+DA | Conditional expectation | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P1
Probability & Statistics | CS+DA | Conditional variance | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P1
Probability & Statistics | CS+DA | Mean, median and mode | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P0
Probability & Statistics | CS+DA | Standard deviation | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P0
Probability & Statistics | CS+DA | Correlation | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P1
Probability & Statistics | CS+DA | Covariance | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P1
Probability & Statistics | CS+DA | Random variables | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P0
Probability & Statistics | CS+DA | Discrete random variables and PMFs | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P0
Probability & Statistics | CS+DA | Bernoulli distribution | [discrete-distributions](02-probability-statistics/discrete-distributions.md) | P0
Probability & Statistics | CS+DA | Binomial distribution | [discrete-distributions](02-probability-statistics/discrete-distributions.md) | P0
Probability & Statistics | CS+DA | Uniform distribution (discrete) | [discrete-distributions](02-probability-statistics/discrete-distributions.md) | P1
Probability & Statistics | CS+DA | Continuous random variables and PDFs | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P0
Probability & Statistics | CS+DA | Uniform distribution (continuous) | [continuous-distributions](02-probability-statistics/continuous-distributions.md) | P0
Probability & Statistics | CS+DA | Exponential distribution | [continuous-distributions](02-probability-statistics/continuous-distributions.md) | P0
Probability & Statistics | CS+DA | Poisson distribution | [discrete-distributions](02-probability-statistics/discrete-distributions.md) | P0
Probability & Statistics | CS+DA | Normal distribution | [continuous-distributions](02-probability-statistics/continuous-distributions.md) | P0
Probability & Statistics | CS+DA | Standard normal distribution | [continuous-distributions](02-probability-statistics/continuous-distributions.md) | P0
Probability & Statistics | CS+DA | t-distribution | [continuous-distributions](02-probability-statistics/continuous-distributions.md) | P1
Probability & Statistics | CS+DA | Chi-squared distribution | [continuous-distributions](02-probability-statistics/continuous-distributions.md) | P1
Probability & Statistics | CS+DA | Cumulative distribution function | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P0
Probability & Statistics | CS+DA | Conditional PDF | [random-variables-and-moments](02-probability-statistics/random-variables-and-moments.md) | P1
Probability & Statistics | CS+DA | Central limit theorem | [statistical-inference](02-probability-statistics/statistical-inference.md) | P1
Probability & Statistics | CS+DA | Confidence intervals | [statistical-inference](02-probability-statistics/statistical-inference.md) | P1
Probability & Statistics | CS+DA | z-test | [statistical-inference](02-probability-statistics/statistical-inference.md) | P1
Probability & Statistics | CS+DA | t-test | [statistical-inference](02-probability-statistics/statistical-inference.md) | P1
Probability & Statistics | CS+DA | Chi-squared test | [statistical-inference](02-probability-statistics/statistical-inference.md) | P1
Linear Algebra | CS+DA | Vectors and vector spaces | [vector-spaces](03-linear-algebra/vector-spaces.md) | P1
Linear Algebra | CS+DA | Subspaces | [vector-spaces](03-linear-algebra/vector-spaces.md) | P1
Linear Algebra | CS+DA | Linear dependence and independence | [vector-spaces](03-linear-algebra/vector-spaces.md) | P0
Linear Algebra | CS+DA | Matrices and matrix operations | [matrices-and-determinants](03-linear-algebra/matrices-and-determinants.md) | P0
Linear Algebra | CS+DA | Projection matrices | [orthogonality-projections-svd](03-linear-algebra/orthogonality-projections-svd.md) | P1
Linear Algebra | CS+DA | Orthogonal matrices | [orthogonality-projections-svd](03-linear-algebra/orthogonality-projections-svd.md) | P1
Linear Algebra | CS+DA | Idempotent matrices | [matrices-and-determinants](03-linear-algebra/matrices-and-determinants.md) | P1
Linear Algebra | CS+DA | Partition matrices and properties | [matrices-and-determinants](03-linear-algebra/matrices-and-determinants.md) | P2
Linear Algebra | CS+DA | Quadratic forms | [matrices-and-determinants](03-linear-algebra/matrices-and-determinants.md) | P1
Linear Algebra | CS+DA | Systems of linear equations and solutions | [linear-systems-and-lu](03-linear-algebra/linear-systems-and-lu.md) | P0
Linear Algebra | CS+DA | Gaussian elimination | [linear-systems-and-lu](03-linear-algebra/linear-systems-and-lu.md) | P0
Linear Algebra | CS+DA | Eigenvalues and eigenvectors | [eigenvalues-and-eigenvectors](03-linear-algebra/eigenvalues-and-eigenvectors.md) | P0
Linear Algebra | CS+DA | Determinants | [matrices-and-determinants](03-linear-algebra/matrices-and-determinants.md) | P0
Linear Algebra | CS+DA | Rank | [vector-spaces](03-linear-algebra/vector-spaces.md) | P0
Linear Algebra | CS+DA | Nullity | [vector-spaces](03-linear-algebra/vector-spaces.md) | P0
Linear Algebra | CS+DA | Projections | [orthogonality-projections-svd](03-linear-algebra/orthogonality-projections-svd.md) | P1
Linear Algebra | CS+DA | LU decomposition | [linear-systems-and-lu](03-linear-algebra/linear-systems-and-lu.md) | P1
Linear Algebra | CS+DA | Singular value decomposition | [orthogonality-projections-svd](03-linear-algebra/orthogonality-projections-svd.md) | P1
Calculus & Optimization | CS+DA | Functions of a single variable | [limits-continuity-differentiability](04-calculus-optimization/limits-continuity-differentiability.md) | P1
Calculus & Optimization | CS+DA | Limits | [limits-continuity-differentiability](04-calculus-optimization/limits-continuity-differentiability.md) | P0
Calculus & Optimization | CS+DA | Continuity | [limits-continuity-differentiability](04-calculus-optimization/limits-continuity-differentiability.md) | P0
Calculus & Optimization | CS+DA | Differentiability | [limits-continuity-differentiability](04-calculus-optimization/limits-continuity-differentiability.md) | P0
Calculus & Optimization | CS+DA | Taylor series | [mean-value-theorems-and-taylor](04-calculus-optimization/mean-value-theorems-and-taylor.md) | P1
Calculus & Optimization | CS+DA | Maxima and minima | [maxima-minima-optimization](04-calculus-optimization/maxima-minima-optimization.md) | P0
Calculus & Optimization | CS+DA | Mean value theorem | [mean-value-theorems-and-taylor](04-calculus-optimization/mean-value-theorems-and-taylor.md) | P1
Calculus & Optimization | CS+DA | Integration | [integration](04-calculus-optimization/integration.md) | P1
Calculus & Optimization | CS+DA | Single-variable optimization | [maxima-minima-optimization](04-calculus-optimization/maxima-minima-optimization.md) | P0
C Programming | CS | C syntax and semantics | [c-basics-and-expressions](05-c-programming/c-basics-and-expressions.md) | P0
C Programming | CS | Pointers | [pointers-arrays-strings](05-c-programming/pointers-arrays-strings.md) | P0
C Programming | CS | Arrays in C | [pointers-arrays-strings](05-c-programming/pointers-arrays-strings.md) | P0
C Programming | CS | Structures | [functions-recursion-structures](05-c-programming/functions-recursion-structures.md) | P1
C Programming | CS | Functions | [functions-recursion-structures](05-c-programming/functions-recursion-structures.md) | P0
C Programming | CS | Recursion in C | [functions-recursion-structures](05-c-programming/functions-recursion-structures.md) | P0
C Programming | CS | Memory and pointer reasoning | [pointers-arrays-strings](05-c-programming/pointers-arrays-strings.md) | P0
C Programming | CS | Tracing C programs and expressions | [c-basics-and-expressions](05-c-programming/c-basics-and-expressions.md) | P0
Python Programming | DA | Python syntax | [python-basics](06-python-programming/python-basics.md) | P0
Python Programming | DA | Variables and expressions | [python-basics](06-python-programming/python-basics.md) | P0
Python Programming | DA | Functions | [python-basics](06-python-programming/python-basics.md) | P0
Python Programming | DA | Lists | [python-collections](06-python-programming/python-collections.md) | P0
Python Programming | DA | Tuples | [python-collections](06-python-programming/python-collections.md) | P1
Python Programming | DA | Dictionaries | [python-collections](06-python-programming/python-collections.md) | P0
Python Programming | DA | Sets | [python-collections](06-python-programming/python-collections.md) | P1
Python Programming | DA | Classes/basic OOP | [python-oop-recursion-tracing](06-python-programming/python-oop-recursion-tracing.md) | P1
Python Programming | DA | Iteration and loops | [python-basics](06-python-programming/python-basics.md) | P0
Python Programming | DA | Recursion | [python-oop-recursion-tracing](06-python-programming/python-oop-recursion-tracing.md) | P0
Python Programming | DA | Comprehensions | [python-collections](06-python-programming/python-collections.md) | P1
Python Programming | DA | Tracing Python programs | [python-oop-recursion-tracing](06-python-programming/python-oop-recursion-tracing.md) | P0
Data Structures | CS+DA | Arrays | [arrays-stacks-queues](07-data-structures/arrays-stacks-queues.md) | P0
Data Structures | CS+DA | Stacks | [arrays-stacks-queues](07-data-structures/arrays-stacks-queues.md) | P0
Data Structures | CS+DA | Queues | [arrays-stacks-queues](07-data-structures/arrays-stacks-queues.md) | P0
Data Structures | CS+DA | Linked lists | [linked-lists](07-data-structures/linked-lists.md) | P0
Data Structures | CS+DA | Trees | [trees-and-bst](07-data-structures/trees-and-bst.md) | P0
Data Structures | CS+DA | Binary search trees | [trees-and-bst](07-data-structures/trees-and-bst.md) | P0
Data Structures | CS+DA | Binary heaps | [heaps](07-data-structures/heaps.md) | P0
Data Structures | CS+DA | Graphs | [graphs](07-data-structures/graphs.md) | P1
Data Structures | CS+DA | Data-structure operations and complexity | [complexity-reference](07-data-structures/complexity-reference.md) | P0
Algorithms | CS+DA | Asymptotic notation and growth rates | [asymptotic-analysis](08-algorithms/asymptotic-analysis.md) | P0
Algorithms | CS+DA | Worst-case time complexity | [asymptotic-analysis](08-algorithms/asymptotic-analysis.md) | P0
Algorithms | CS+DA | Worst-case space complexity | [asymptotic-analysis](08-algorithms/asymptotic-analysis.md) | P1
Algorithms | CS+DA | Linear search | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P1
Algorithms | CS+DA | Binary search | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P0
Algorithms | CS+DA | Selection sort | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P1
Algorithms | CS+DA | Bubble sort | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P1
Algorithms | CS+DA | Insertion sort | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P0
Algorithms | CS+DA | Merge sort | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P0
Algorithms | CS+DA | Quicksort | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | P0
Algorithms | CS+DA | Hashing | [hashing](08-algorithms/hashing.md) | P0
Algorithms | CS+DA | Divide and conquer | [divide-and-conquer](08-algorithms/divide-and-conquer.md) | P0
Algorithms | CS+DA | Greedy algorithms | [greedy-algorithms](08-algorithms/greedy-algorithms.md) | P0
Algorithms | CS+DA | Dynamic programming | [dynamic-programming](08-algorithms/dynamic-programming.md) | P0
Algorithms | CS+DA | Graph representations | [graphs](07-data-structures/graphs.md) | P1
Algorithms | CS+DA | Graph traversal: BFS | [graph-traversals](08-algorithms/graph-traversals.md) | P0
Algorithms | CS+DA | Graph traversal: DFS | [graph-traversals](08-algorithms/graph-traversals.md) | P0
Algorithms | CS+DA | Minimum spanning trees | [minimum-spanning-trees](08-algorithms/minimum-spanning-trees.md) | P0
Algorithms | CS+DA | Shortest paths | [shortest-paths](08-algorithms/shortest-paths.md) | P0
Digital Logic | CS | Boolean algebra | [boolean-algebra-and-minimization](09-digital-logic/boolean-algebra-and-minimization.md) | P0
Digital Logic | CS | Boolean minimization: algebraic technique | [boolean-algebra-and-minimization](09-digital-logic/boolean-algebra-and-minimization.md) | P1
Digital Logic | CS | Karnaugh maps | [boolean-algebra-and-minimization](09-digital-logic/boolean-algebra-and-minimization.md) | P0
Digital Logic | CS | Tabular minimization method | [boolean-algebra-and-minimization](09-digital-logic/boolean-algebra-and-minimization.md) | P2
Digital Logic | CS | Combinational circuit design | [combinational-circuits](09-digital-logic/combinational-circuits.md) | P0
Digital Logic | CS | Sequential circuit design | [sequential-circuits](09-digital-logic/sequential-circuits.md) | P0
Digital Logic | CS | Number representation | [number-representation-and-arithmetic](09-digital-logic/number-representation-and-arithmetic.md) | P0
Digital Logic | CS | Fixed-point arithmetic | [number-representation-and-arithmetic](09-digital-logic/number-representation-and-arithmetic.md) | P1
Digital Logic | CS | Floating-point arithmetic | [number-representation-and-arithmetic](09-digital-logic/number-representation-and-arithmetic.md) | P0
Computer Organization & Architecture | CS | Instruction sets | [instruction-sets-and-addressing](10-computer-organization/instruction-sets-and-addressing.md) | P1
Computer Organization & Architecture | CS | Addressing modes | [instruction-sets-and-addressing](10-computer-organization/instruction-sets-and-addressing.md) | P0
Computer Organization & Architecture | CS | ALU design | [alu-and-control-unit](10-computer-organization/alu-and-control-unit.md) | P2
Computer Organization & Architecture | CS | Hardwired control unit | [alu-and-control-unit](10-computer-organization/alu-and-control-unit.md) | P1
Computer Organization & Architecture | CS | Microprogrammed control unit | [alu-and-control-unit](10-computer-organization/alu-and-control-unit.md) | P1
Computer Organization & Architecture | CS | Memory interfacing | [memory-hierarchy-and-cache](10-computer-organization/memory-hierarchy-and-cache.md) | P1
Computer Organization & Architecture | CS | Memory hierarchy and performance | [memory-hierarchy-and-cache](10-computer-organization/memory-hierarchy-and-cache.md) | P0
Computer Organization & Architecture | CS | Cache memory mapping | [memory-hierarchy-and-cache](10-computer-organization/memory-hierarchy-and-cache.md) | P0
Computer Organization & Architecture | CS | I/O interface | [io-interrupts-dma](10-computer-organization/io-interrupts-dma.md) | P1
Computer Organization & Architecture | CS | Interrupts | [io-interrupts-dma](10-computer-organization/io-interrupts-dma.md) | P1
Computer Organization & Architecture | CS | DMA | [io-interrupts-dma](10-computer-organization/io-interrupts-dma.md) | P1
Computer Organization & Architecture | CS | Instruction pipelining | [pipelining](10-computer-organization/pipelining.md) | P0
Computer Organization & Architecture | CS | Pipeline hazards | [pipelining](10-computer-organization/pipelining.md) | P0
Theory of Computation | CS | Regular expressions | [regular-languages-and-finite-automata](11-theory-of-computation/regular-languages-and-finite-automata.md) | P0
Theory of Computation | CS | Finite automata | [regular-languages-and-finite-automata](11-theory-of-computation/regular-languages-and-finite-automata.md) | P0
Theory of Computation | CS | Regular languages | [regular-languages-and-finite-automata](11-theory-of-computation/regular-languages-and-finite-automata.md) | P0
Theory of Computation | CS | Context-free grammars | [context-free-languages-and-pda](11-theory-of-computation/context-free-languages-and-pda.md) | P0
Theory of Computation | CS | Push-down automata | [context-free-languages-and-pda](11-theory-of-computation/context-free-languages-and-pda.md) | P1
Theory of Computation | CS | Context-free languages | [context-free-languages-and-pda](11-theory-of-computation/context-free-languages-and-pda.md) | P0
Theory of Computation | CS | Pumping lemma | [pumping-lemma](11-theory-of-computation/pumping-lemma.md) | P1
Theory of Computation | CS | Turing machines | [turing-machines-and-undecidability](11-theory-of-computation/turing-machines-and-undecidability.md) | P1
Theory of Computation | CS | Undecidability | [turing-machines-and-undecidability](11-theory-of-computation/turing-machines-and-undecidability.md) | P0
Compiler Design | CS | Lexical analysis | [lexical-analysis](12-compiler-design/lexical-analysis.md) | P1
Compiler Design | CS | Parsing | [parsing](12-compiler-design/parsing.md) | P0
Compiler Design | CS | Syntax-directed translation | [syntax-directed-translation](12-compiler-design/syntax-directed-translation.md) | P0
Compiler Design | CS | Runtime environments | [runtime-environments](12-compiler-design/runtime-environments.md) | P1
Compiler Design | CS | Intermediate code generation | [intermediate-code-generation](12-compiler-design/intermediate-code-generation.md) | P1
Compiler Design | CS | Local optimization | [optimization-and-dataflow](12-compiler-design/optimization-and-dataflow.md) | P1
Compiler Design | CS | Data-flow analysis | [optimization-and-dataflow](12-compiler-design/optimization-and-dataflow.md) | P1
Compiler Design | CS | Constant propagation | [optimization-and-dataflow](12-compiler-design/optimization-and-dataflow.md) | P1
Compiler Design | CS | Liveness analysis | [optimization-and-dataflow](12-compiler-design/optimization-and-dataflow.md) | P0
Compiler Design | CS | Common subexpression elimination | [optimization-and-dataflow](12-compiler-design/optimization-and-dataflow.md) | P1
Operating Systems | CS | System calls | [processes-threads-syscalls](13-operating-systems/processes-threads-syscalls.md) | P1
Operating Systems | CS | Processes | [processes-threads-syscalls](13-operating-systems/processes-threads-syscalls.md) | P0
Operating Systems | CS | Threads | [processes-threads-syscalls](13-operating-systems/processes-threads-syscalls.md) | P1
Operating Systems | CS | Inter-process communication | [processes-threads-syscalls](13-operating-systems/processes-threads-syscalls.md) | P2
Operating Systems | CS | Concurrency | [synchronization](13-operating-systems/synchronization.md) | P0
Operating Systems | CS | Synchronization | [synchronization](13-operating-systems/synchronization.md) | P0
Operating Systems | CS | Deadlocks | [deadlocks](13-operating-systems/deadlocks.md) | P0
Operating Systems | CS | CPU scheduling | [cpu-scheduling](13-operating-systems/cpu-scheduling.md) | P0
Operating Systems | CS | I/O scheduling | [file-systems-and-disk-scheduling](13-operating-systems/file-systems-and-disk-scheduling.md) | P1
Operating Systems | CS | Memory management | [memory-management](13-operating-systems/memory-management.md) | P0
Operating Systems | CS | Virtual memory | [virtual-memory](13-operating-systems/virtual-memory.md) | P0
Operating Systems | CS | File systems | [file-systems-and-disk-scheduling](13-operating-systems/file-systems-and-disk-scheduling.md) | P1
DBMS & Data Warehousing | CS+DA | ER model | [er-model](14-databases/er-model.md) | P1
DBMS & Data Warehousing | CS+DA | Relational model | [relational-model-algebra-calculus](14-databases/relational-model-algebra-calculus.md) | P0
DBMS & Data Warehousing | CS+DA | Relational algebra | [relational-model-algebra-calculus](14-databases/relational-model-algebra-calculus.md) | P0
DBMS & Data Warehousing | CS+DA | Tuple calculus | [relational-model-algebra-calculus](14-databases/relational-model-algebra-calculus.md) | P1
DBMS & Data Warehousing | CS+DA | SQL | [sql](14-databases/sql.md) | P0
DBMS & Data Warehousing | CS+DA | Integrity constraints | [constraints-and-normalization](14-databases/constraints-and-normalization.md) | P1
DBMS & Data Warehousing | CS+DA | Normal forms | [constraints-and-normalization](14-databases/constraints-and-normalization.md) | P0
DBMS & Data Warehousing | CS+DA | File organization | [file-organization-and-indexing](14-databases/file-organization-and-indexing.md) | P1
DBMS & Data Warehousing | CS+DA | Indexing | [file-organization-and-indexing](14-databases/file-organization-and-indexing.md) | P0
DBMS & Data Warehousing | CS+DA | B trees | [file-organization-and-indexing](14-databases/file-organization-and-indexing.md) | P0
DBMS & Data Warehousing | CS+DA | B+ trees | [file-organization-and-indexing](14-databases/file-organization-and-indexing.md) | P0
DBMS & Data Warehousing | CS+DA | Transactions | [transactions-and-concurrency](14-databases/transactions-and-concurrency.md) | P0
DBMS & Data Warehousing | CS+DA | Concurrency control | [transactions-and-concurrency](14-databases/transactions-and-concurrency.md) | P0
DBMS & Data Warehousing | CS+DA | Data types | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
DBMS & Data Warehousing | CS+DA | Data transformation: normalization | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
DBMS & Data Warehousing | CS+DA | Discretization | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
DBMS & Data Warehousing | CS+DA | Sampling | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
DBMS & Data Warehousing | CS+DA | Compression | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P2
DBMS & Data Warehousing | CS+DA | Data warehouse schemas for multidimensional models | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
DBMS & Data Warehousing | CS+DA | Concept hierarchies | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
DBMS & Data Warehousing | CS+DA | Measures: categorization and computation | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | P1
Computer Networks | CS | Principles of layering | [layering-switching-performance](15-computer-networks/layering-switching-performance.md) | P1
Computer Networks | CS | Circuit switching | [layering-switching-performance](15-computer-networks/layering-switching-performance.md) | P1
Computer Networks | CS | Packet switching | [layering-switching-performance](15-computer-networks/layering-switching-performance.md) | P0
Computer Networks | CS | Virtual circuit switching | [layering-switching-performance](15-computer-networks/layering-switching-performance.md) | P2
Computer Networks | CS | Network performance metrics | [layering-switching-performance](15-computer-networks/layering-switching-performance.md) | P0
Computer Networks | CS | Data-link layer | [data-link-layer](15-computer-networks/data-link-layer.md) | P0
Computer Networks | CS | Error detection | [data-link-layer](15-computer-networks/data-link-layer.md) | P0
Computer Networks | CS | Medium Access Control | [data-link-layer](15-computer-networks/data-link-layer.md) | P0
Computer Networks | CS | Ethernet | [data-link-layer](15-computer-networks/data-link-layer.md) | P1
Computer Networks | CS | Distance-vector routing | [routing](15-computer-networks/routing.md) | P0
Computer Networks | CS | Link-state routing | [routing](15-computer-networks/routing.md) | P1
Computer Networks | CS | IPv4 | [ipv4-addressing](15-computer-networks/ipv4-addressing.md) | P0
Computer Networks | CS | IPv4 fragmentation | [ipv4-addressing](15-computer-networks/ipv4-addressing.md) | P0
Computer Networks | CS | CIDR notation | [ipv4-addressing](15-computer-networks/ipv4-addressing.md) | P0
Computer Networks | CS | Network Address Translation | [ipv4-addressing](15-computer-networks/ipv4-addressing.md) | P1
Computer Networks | CS | TCP flow control | [transport-layer-tcp](15-computer-networks/transport-layer-tcp.md) | P0
Computer Networks | CS | TCP congestion control | [transport-layer-tcp](15-computer-networks/transport-layer-tcp.md) | P0
Computer Networks | CS | Socket API | [transport-layer-tcp](15-computer-networks/transport-layer-tcp.md) | P1
Computer Networks | CS | DNS | [application-layer](15-computer-networks/application-layer.md) | P1
Computer Networks | CS | HTTP | [application-layer](15-computer-networks/application-layer.md) | P1
Machine Learning | DA | Regression vs classification problems | [ml-foundations](16-machine-learning/ml-foundations.md) | P0
Machine Learning | DA | Simple linear regression | [linear-and-logistic-regression](16-machine-learning/linear-and-logistic-regression.md) | P0
Machine Learning | DA | Multiple linear regression | [linear-and-logistic-regression](16-machine-learning/linear-and-logistic-regression.md) | P0
Machine Learning | DA | Ridge regression | [linear-and-logistic-regression](16-machine-learning/linear-and-logistic-regression.md) | P1
Machine Learning | DA | Logistic regression | [linear-and-logistic-regression](16-machine-learning/linear-and-logistic-regression.md) | P0
Machine Learning | DA | k-nearest neighbours | [classification-methods](16-machine-learning/classification-methods.md) | P0
Machine Learning | DA | Naive Bayes classifier | [classification-methods](16-machine-learning/classification-methods.md) | P0
Machine Learning | DA | Linear discriminant analysis | [classification-methods](16-machine-learning/classification-methods.md) | P1
Machine Learning | DA | Support vector machines | [classification-methods](16-machine-learning/classification-methods.md) | P0
Machine Learning | DA | Decision trees | [classification-methods](16-machine-learning/classification-methods.md) | P0
Machine Learning | DA | Bias-variance trade-off | [ml-foundations](16-machine-learning/ml-foundations.md) | P0
Machine Learning | DA | Leave-one-out cross-validation | [ml-foundations](16-machine-learning/ml-foundations.md) | P1
Machine Learning | DA | k-fold cross-validation | [ml-foundations](16-machine-learning/ml-foundations.md) | P1
Machine Learning | DA | Multi-layer perceptron | [neural-networks](16-machine-learning/neural-networks.md) | P0
Machine Learning | DA | Feed-forward neural networks | [neural-networks](16-machine-learning/neural-networks.md) | P0
Machine Learning | DA | Clustering | [clustering](16-machine-learning/clustering.md) | P0
Machine Learning | DA | k-means | [clustering](16-machine-learning/clustering.md) | P0
Machine Learning | DA | k-medoids | [clustering](16-machine-learning/clustering.md) | P1
Machine Learning | DA | Hierarchical clustering: top-down | [clustering](16-machine-learning/clustering.md) | P1
Machine Learning | DA | Hierarchical clustering: bottom-up | [clustering](16-machine-learning/clustering.md) | P0
Machine Learning | DA | Single-linkage clustering | [clustering](16-machine-learning/clustering.md) | P0
Machine Learning | DA | Multiple-linkage clustering | [clustering](16-machine-learning/clustering.md) | P1
Machine Learning | DA | Dimensionality reduction | [dimensionality-reduction-pca](16-machine-learning/dimensionality-reduction-pca.md) | P1
Machine Learning | DA | Principal component analysis | [dimensionality-reduction-pca](16-machine-learning/dimensionality-reduction-pca.md) | P0
Artificial Intelligence | DA | Uninformed search | [search](17-artificial-intelligence/search.md) | P0
Artificial Intelligence | DA | Informed search | [search](17-artificial-intelligence/search.md) | P0
Artificial Intelligence | DA | Adversarial search | [search](17-artificial-intelligence/search.md) | P0
Artificial Intelligence | DA | Propositional logic | [logic-and-inference](17-artificial-intelligence/logic-and-inference.md) | P0
Artificial Intelligence | DA | Predicate logic | [logic-and-inference](17-artificial-intelligence/logic-and-inference.md) | P1
Artificial Intelligence | DA | Conditional independence | [probabilistic-reasoning](17-artificial-intelligence/probabilistic-reasoning.md) | P0
Artificial Intelligence | DA | Representation of conditional independence | [probabilistic-reasoning](17-artificial-intelligence/probabilistic-reasoning.md) | P0
Artificial Intelligence | DA | Exact inference via variable elimination | [probabilistic-reasoning](17-artificial-intelligence/probabilistic-reasoning.md) | P0
Artificial Intelligence | DA | Approximate inference via sampling | [probabilistic-reasoning](17-artificial-intelligence/probabilistic-reasoning.md) | P1

## Topics added beyond the plan

These are not separate items in the plan. They are either missing (General Aptitude) or are the standard machinery GATE uses to test a listed topic, so each is taught inside the chapter it supports.

Area | Paper | Topic | Taught in | Why it is included
--- | --- | --- | --- | ---
General Aptitude | CS+DA | Verbal aptitude | [verbal-aptitude](00-general-aptitude/verbal-aptitude.md) | 15-mark GA section is in every paper; absent from the plan
General Aptitude | CS+DA | Quantitative aptitude | [quantitative-aptitude](00-general-aptitude/quantitative-aptitude.md) | same
General Aptitude | CS+DA | Analytical aptitude | [analytical-aptitude](00-general-aptitude/analytical-aptitude.md) | same
General Aptitude | CS+DA | Spatial aptitude | [spatial-aptitude](00-general-aptitude/spatial-aptitude.md) | same
Discrete Mathematics | CS | Pigeonhole, inclusion-exclusion, Catalan numbers, derangements | [combinatorics](01-discrete-mathematics/combinatorics.md) | standard tools behind 'Combinatorics: counting'
Discrete Mathematics | CS | Euler/Hamiltonian graphs, planarity, trees | [graph-theory](01-discrete-mathematics/graph-theory.md) | routinely asked under 'Graphs'
Probability & Statistics | CS+DA | Geometric distribution, memorylessness, Poisson process | [discrete-distributions](02-probability-statistics/discrete-distributions.md) | needed to work exponential/Poisson problems
Linear Algebra | CS+DA | Cayley-Hamilton, diagonalisation, positive definiteness | [eigenvalues-and-eigenvectors](03-linear-algebra/eigenvalues-and-eigenvectors.md) | standard eigenvalue tools
C Programming | CS | Static vs dynamic scoping, parameter-passing modes | [functions-recursion-structures](05-c-programming/functions-recursion-structures.md) | frequent PYQ theme under 'Functions'
Data Structures | CS+DA | AVL trees, infix/postfix conversion, circular queues | [trees-and-bst](07-data-structures/trees-and-bst.md) | frequent PYQ themes under 'Trees' / 'Stacks'
Algorithms | CS+DA | Master theorem and recursion trees | [asymptotic-analysis](08-algorithms/asymptotic-analysis.md) | needed for 'Worst-case time complexity'
Algorithms | CS+DA | Heap sort, counting/radix sort, sorting lower bound | [searching-and-sorting](08-algorithms/searching-and-sorting.md) | part of 'sorting' in the official syllabus
Algorithms | CS+DA | Topological sort, SCCs, Bellman-Ford, Floyd-Warshall | [graph-traversals](08-algorithms/graph-traversals.md) | applications of traversals / shortest paths
Theory of Computation | CS | DFA minimisation, closure & decidability tables, Rice's theorem | [turing-machines-and-undecidability](11-theory-of-computation/turing-machines-and-undecidability.md) | how 'Undecidability' and 'Regular languages' are asked
Compiler Design | CS | FIRST/FOLLOW, LL(1), LR(0)/SLR/LALR/CLR tables | [parsing](12-compiler-design/parsing.md) | the substance of 'Parsing'
Compiler Design | CS | Basic blocks, CFGs, register allocation by colouring | [intermediate-code-generation](12-compiler-design/intermediate-code-generation.md) | prerequisite for data-flow analysis
Operating Systems | CS | fork() counting, page replacement, Banker's algorithm, TLB/EMAT | [virtual-memory](13-operating-systems/virtual-memory.md) | the substance of 'Processes', 'Virtual memory', 'Deadlocks'
DBMS | CS+DA | Serializability, recoverability, 2PL, timestamp ordering | [transactions-and-concurrency](14-databases/transactions-and-concurrency.md) | the substance of 'Concurrency control'
DBMS | CS+DA | FD closure, candidate keys, lossless join, dependency preservation | [constraints-and-normalization](14-databases/constraints-and-normalization.md) | the substance of 'Normal forms'
DBMS | CS+DA | OLAP operations, data cube, cuboid counting | [data-warehousing-and-preprocessing](14-databases/data-warehousing-and-preprocessing.md) | context for warehouse schemas
Computer Networks | CS | Sliding-window protocols (stop-and-wait, GBN, SR) | [data-link-layer](15-computer-networks/data-link-layer.md) | underpin TCP flow control; classic numericals
Computer Networks | CS | TCP handshake, sequence numbers, UDP contrast | [transport-layer-tcp](15-computer-networks/transport-layer-tcp.md) | needed to reason about TCP flow/congestion control
Machine Learning | DA | Evaluation metrics, loss functions, gradient descent, backpropagation | [ml-foundations](16-machine-learning/ml-foundations.md) | needed to answer model questions
Artificial Intelligence | DA | Bayesian networks, d-separation, resolution, unification | [probabilistic-reasoning](17-artificial-intelligence/probabilistic-reasoning.md) | the substance of 'representation of conditional independence' and logic

[Roadmap](ROADMAP.md) · [Progress](PROGRESS.md) · [Structure](STRUCTURE.md)
