# Brief: 00-general-aptitude + 06-python-programming

## 1) 00-general-aptitude/
General Aptitude, worth 15 marks in BOTH papers (5 x 1-mark + 5 x 2-mark questions). This section was MISSING from the student's Master Study Plan; say so in its README. Official GATE 2027 GA syllabus:
- Verbal Aptitude: basic English grammar (tenses, articles, adjectives, prepositions, conjunctions, verb-noun agreement, other parts of speech); basic vocabulary (words, idioms, phrases in context); reading comprehension; narrative sequencing.
- Quantitative Aptitude: data interpretation (bar graphs, pie charts, other graphs, 2D and 3D plots, maps, tables); numerical computation and estimation (ratios, percentages, powers, exponents, logarithms, permutations and combinations, series); mensuration and geometry; elementary statistics and probability.
- Analytical Aptitude: logic (deduction and induction), analogy, numerical relations and reasoning.
- Spatial Aptitude: transformation of shapes (translation, rotation, scaling, mirroring), assembling and grouping, paper folding, cutting, patterns in 2 and 3 dimensions.

Files: README.md, verbal-aptitude.md, quantitative-aptitude.md, analytical-aptitude.md, spatial-aptitude.md, CHEATSHEET.md, CHECKPOINT.md.
- Verbal: rules + many short examples of the sentence-correction / fill-in-the-blank / odd-one-out / sequencing items GATE asks.
- Quantitative: speed tricks (percentage-change chains, ratio scaling, log rules, AP/GP sums, mensuration formula table, clocks/calendars if useful).
- Analytical: syllogisms by Venn method, truth-teller/liar, seating/ordering puzzles, blood relations, direction sense, coding-decoding.
- Spatial: ASCII diagrams for folding/cutting, mirror images, cube unfolding (nets, opposite faces), counting painted cubes.

## 2) 06-python-programming/ (DA paper)
Plan topics: Python syntax; Variables and expressions; Functions; Lists; Tuples; Dictionaries; Sets; Classes/basic OOP; Iteration and loops; Recursion; Comprehensions; Tracing Python programs.
Files: README.md, python-basics.md, python-collections.md, python-oop-recursion-tracing.md, CHEATSHEET.md, CHECKPOINT.md.
GATE DA tests output prediction and tracing. Cover: / vs // vs % with negatives, operator precedence, `is` vs `==`, mutability and aliasing, mutable default arguments, slicing rules (incl. negative steps), list methods and their return values (append returns None), shallow vs deep copy, range/enumerate/zip, dict ordering and iteration, set operations, comprehension scoping, LEGB scope with global/nonlocal, closures (late binding of lambdas in loops), generator basics, class vs instance attributes, `__init__`, inheritance and method resolution, `__str__`/`__eq__`, recursion with memoization, recursion depth, stacks/queues via list and collections.deque, dict as a hash table.
Run every code example with Python via Bash to confirm the stated output before writing it down.
Connect to 07-data-structures, 08-algorithms, 05-c-programming (contrast C and Python semantics) and 16-machine-learning.
