# Brief: 01-discrete-mathematics (CS paper, ~7-9 marks/year)

Plan topics: Propositional logic; First-order logic; Sets; Relations; Functions; Partial orders; Lattices; Monoids; Groups; Graphs: connectivity; Graphs: matching; Graphs: colouring; Combinatorics: counting; Recurrence relations; Generating functions.

Files: README.md, propositional-logic.md, first-order-logic.md, sets-relations-functions.md, posets-and-lattices.md, algebraic-structures.md, graph-theory.md, combinatorics.md, recurrences-and-generating-functions.md, CHEATSHEET.md, CHECKPOINT.md.

Must cover (beyond basics); brute-force check counts, truth tables and group properties with Python:
- Propositional: connectives, truth tables, tautology/contradiction/contingency, satisfiability vs validity, equivalence-law table, converse/inverse/contrapositive, functional completeness (NAND/NOR), CNF/DNF, rules of inference, fast validity checking (assume the conclusion is false).
- FOL: predicates, quantifiers, negating quantified statements, nested-quantifier order, forall distributes over AND but not OR (dual for exists) with counterexamples, English-to-FOL translation (a GATE favourite), free/bound variables, valid implications such as (exists x forall y P) -> (forall y exists x P).
- Sets: operations, power set, cardinality, inclusion-exclusion, countable vs uncountable (diagonalisation; link to TOC undecidability).
- Relations: reflexive/irreflexive/symmetric/antisymmetric/asymmetric/transitive, counting formulas for each on n elements, closures, equivalence relations and partitions, Bell numbers.
- Functions: injective/surjective/bijective, counting (n^m, onto functions via inclusion-exclusion / Stirling numbers of the 2nd kind), composition and inverses, pigeonhole link.
- Posets: Hasse diagrams, maximal/minimal/greatest/least, bounds, lub/glb, topological-sort link, total orders, well-ordering.
- Lattices: definition, bounded, distributive, complemented, Boolean algebra; diamond M3 and pentagon N5 tests; divisor lattices D_n and when they are complemented.
- Algebraic structures: closure -> semigroup -> monoid -> group -> abelian group; identity/inverse; group and element order; Lagrange's theorem; cyclic groups and generator count phi(n); subgroups; Z_n under + and Z_n* under x; Cayley tables for property checks.
- Graphs: handshaking; complete/bipartite/regular/planar; connectivity (cut vertex, bridge, vertex/edge connectivity, components, max edges with k components); Euler and Hamiltonian; planarity, v - e + f = 2, e <= 3v - 6 (and e <= 2v - 4 if triangle-free); trees (n-1 edges, Cayley n^(n-2)); isomorphism invariants; matching (maximum/maximal/perfect, Hall, Konig); colouring (chromatic number, chromatic polynomial, chi of standard graphs, greedy bound Delta+1, four-colour theorem); independent set, vertex cover, edge cover, Gallai identities.
- Combinatorics: sum/product rules, permutations/combinations with and without repetition, stars and bars, circular permutations, derangements, generalised pigeonhole, inclusion-exclusion, Catalan numbers (what they count: BSTs, parenthesisations, Dyck paths), binomial identities.
- Recurrences: forming them (bit strings without "00", tilings, Tower of Hanoi); solving linear homogeneous/non-homogeneous via characteristic roots (incl. repeated roots, particular solutions); substitution; link to the Master theorem in 08-algorithms/asymptotic-analysis.md. Generating functions: ordinary GFs, standard GF table, coefficient extraction, solving recurrences and counting with GFs.
