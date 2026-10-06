# Syntax-Directed Translation

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Syntax-directed translation
> **Prerequisites:** [Parsing](parsing.md) · **Leads to:** [Intermediate code generation](intermediate-code-generation.md) · [Runtime environments](runtime-environments.md)

## Quick glance

- **Syntax-directed definition (SDD):** a CFG where each grammar symbol has **attributes** and each production has **semantic rules** that compute them. **Syntax-directed translation scheme (SDT):** a CFG with **semantic actions** `{...}` embedded in the right sides; the action runs when the parser reaches that position.
- **Synthesised attribute:** computed from the attributes of the node's **children** (and its own). **Inherited attribute:** computed from the **parent and left/right siblings**.
- **S-attributed SDD:** only synthesised attributes. Evaluated bottom-up in any bottom-up parser, rules run at reduction time, in post-order. **L-attributed SDD:** each inherited attribute of X_i depends only on the parent's inherited attributes and attributes of the siblings **to the left** of X_i (and X_i's own attributes), never on a right sibling. Every S-attributed SDD is L-attributed. L-attributed suits one-pass depth-first left-to-right evaluation (and LL parsing).
- **Annotated parse tree:** the parse tree with attribute values filled in. Evaluation order comes from the **dependency graph**, which must be acyclic.
- The output of an SDT with print actions is the order in which actions fire in a **depth-first, left-to-right traversal** of the parse tree; with a bottom-up (LR) parser an action at the end of a production fires at the **reduction**.
- Classic uses: evaluating expressions, translating infix to postfix, building syntax trees and DAGs, passing declared types to identifiers (inherited), generating three-address code.
- #1 trap: tracing the wrong order. For `E → E + T {print '+'}` the `+` is printed **after** both operands' actions (post-order), so infix becomes postfix; a print action placed **before** the children produces pre-order (prefix) output.

## 1. Attributes and semantic rules

**Intuition.** A parse tree says *what the program is made of*; attributes say *what it means* (value, type, code, address). Synthesised attributes flow **up** the tree (a node's value from its children's values); inherited attributes flow **down or sideways** (a declaration's type reaches each declared identifier; the left operand's context reaches the right).

**Definitions.**
- An attribute of nonterminal A defined by a rule at a production **A → α** in terms of the attributes of **A's children** is **synthesised**.
- An attribute of a right-side symbol X_i defined at a production A → …X_i… in terms of attributes of **A, or of other X_j** in the same production, is **inherited**.
- Terminals have only synthesised ("lexical") attributes supplied by the lexer, e.g. `num.lexval`, `id.name`.

An SDD with only synthesised attributes is **S-attributed**. An SDD where, for each production A → X₁X₂…X_n and each inherited attribute X_i.a, the rule uses only (a) inherited attributes of A, (b) attributes (inherited or synthesised) of X₁…X_{i−1}, and (c) attributes of X_i itself (no cycles), is **L-attributed** ("left to right").

| | S-attributed | L-attributed |
| --- | --- | --- |
| Inherited attributes | none | yes, restricted to left dependencies |
| Evaluation | bottom-up, at reductions | depth-first, left to right |
| Parser | LR (natural), also LL | LL natural, LR via marker nonterminals |
| Relation | ⊂ | |

**Classification quiz.** Production A → X Y Z with rules:
- Y.i = A.i + X.s : inherited, uses parent and **left** sibling: L-attributed.
- Y.i = Z.s : uses a **right** sibling: **not** L-attributed.
- A.s = X.s + Y.s : synthesised.
- X.i = Y.s : inherited from the right sibling: not L-attributed.

## 2. Worked SDDs and annotated parse trees

### 2.1 Desk calculator (S-attributed)

| Production | Semantic rule |
| --- | --- |
| L → E n | print(E.val) |
| E → E₁ + T | E.val = E₁.val + T.val |
| E → T | E.val = T.val |
| T → T₁ * F | T.val = T₁.val × F.val |
| T → F | T.val = F.val |
| F → ( E ) | F.val = E.val |
| F → digit | F.val = digit.lexval |

Input `3 * 5 + 4 n`. Annotated tree (values in brackets):

```text
L: prints 19
└─ E.val=19
   ├─ E.val=15
   │  └─ T.val=15
   │     ├─ T.val=3  (F.val=3, digit 3)
   │     ├─ *
   │     └─ F.val=5  (digit 5)
   ├─ +
   └─ T.val=4  (F.val=4, digit 4)
```

Evaluation order (post-order): digit 3 → F=3 → T=3; digit 5 → F=5; T=3×5=15; E=15; digit 4 → F=4 → T=4; E=15+4=19; print 19. S-attributed, so it runs inside an LR parser by attaching each rule to its reduction (values kept on the parser's value stack).

### 2.2 Binary number to value (S-attributed)

B → B₁ 0 {B.val = 2·B₁.val} | B₁ 1 {B.val = 2·B₁.val + 1} | 0 {B.val = 0} | 1 {B.val = 1}.
For `1101`: bits read left to right: 1 → 1; then 1 → 2·1+1 = 3; then 0 → 2·3 = 6; then 1 → 2·6+1 = 13. Check: 1101₂ = 13.

### 2.3 Declarations (inherited, L-attributed)

| Production | Semantic rule |
| --- | --- |
| D → T L | L.in = T.type |
| T → int | T.type = integer |
| T → float | T.type = float |
| L → L₁ , id | L₁.in = L.in; addtype(id.entry, L.in) |
| L → id | addtype(id.entry, L.in) |

Input `float x, y, z`. T.type = float flows down to L.in, which flows down the left spine through L → L₁ , id, and at each id the type is stored in the symbol table. `L.in` is inherited: in D → T L it uses the attribute of the left sibling T, so the SDD is L-attributed. Dependency graph edges: T.type → L.in → L₁.in → … → addtype. This SDD is **not** S-attributed.

### 2.4 Dependency graphs and order

Edge from attribute b to attribute a if a's rule uses b. Valid evaluation orders are the topological orders of the dependency graph. **If the graph has a cycle, no order exists** (a circular SDD); testing for circularity is expensive (exponential in the worst case) in general. S-attributed and L-attributed SDDs are guaranteed acyclic.

### 2.5 Worked evaluation: operators with unusual semantics

Grammar: E → E₁ # T {E.val = E₁.val × T.val} | T {E.val = T.val}; T → T₁ & F {T.val = T₁.val + F.val} | F {T.val = F.val}; F → num {F.val = num.val}. Evaluate `2 # 3 & 5 # 6 & 4`.

1. The grammar is left-recursive in E and T, so `#` and `&` are left-associative, and `&` binds tighter (it is lower in the grammar).
2. T-level: `3 & 5` = 3 + 5 = 8; `6 & 4` = 6 + 4 = 10; `2` alone is T = 2.
3. E-level, left to right: (2 # 8) = 16; then 16 # 10 = **160**.

(Parse structure: E(E(E(T=2) # T=8) # T=10).)

## 3. Syntax-directed translation schemes (SDT)

A **translation scheme** embeds actions `{…}` in the right sides. The action runs when the parser reaches that position. Rules for actions in L-attributed schemes: an inherited attribute of a symbol must be computed by an action placed **immediately before** that symbol; an action must not refer to a synthesised attribute of a symbol to its right; the synthesised attribute of the left side is computed after (at the end of) the right side.

### 3.1 Infix to postfix (postfix SDT)

```text
E -> E + T   { print('+') }
E -> T
T -> T * F   { print('*') }
T -> F
F -> ( E )
F -> id      { print(id.name) }
```

Actions sit at the **right ends** of productions, so in an LR parser they execute at reductions (post-order). Input `a + b * c`:

1. Reduce F → a: print `a`. (T → F, E → T have no actions.)
2. Reduce F → b: print `b`. Reduce F → c: print `c`.
3. Reduce T → T * F: print `*`.
4. Reduce E → E + T: print `+`.

Output: **abc\*+** (the postfix form). For `(a + b) * c`: `a`, `b`, `+` (inside the parentheses), then `c`, `*`: output **ab+c\***. The parentheses have no action because F → ( E ) is a pure copy.

### 3.2 Removing left recursion from a translation scheme

For top-down parsing the left recursion must go, while keeping the output the same:

```text
E  -> T R
R  -> + T { print('+') } R | - T { print('-') } R | ε
T  -> num { print(num.val) }
```

Input `9 + 5 + 2`: E → T R: T prints 9; R sees `+`: T prints 5, then the action prints `+`, then R again: T prints 2, prints `+`, then R → ε. Output **95+2+**, the postfix of (9+5)+2. Note each print is placed right after the operand, before the recursive R; the order of the actions in the production is what controls the result.

### 3.3 Actions in the middle: depth-first order

Grammar (recursive descent): **S → ( S ) { print 1 } S | ε { print 2 }**. For each input the actions fire as the traversal reaches them, with ε's action firing when ε is chosen:

| Input | Output |
| --- | --- |
| `()` | 2 1 2 |
| `(())` | 2 1 2 1 2 |
| `()()` | 2 1 2 1 2 |
| `(()())` | 2 1 2 1 2 1 2 |

Trace for `()`: S → ( S ) … : the inner S sees `)` so picks ε and prints 2; match `)`; print 1; the trailing S sees end of input so picks ε and prints 2: **212**. In general the output has one `2` per ε-use and one `1` per pair: n pairs give 2n + 1 symbols (n ones, n+1 twos). Verified with a recursive-descent simulation.

**Rule of thumb.** Draw the parse tree and put each action as a leaf at its position in the right side. The output is the sequence of action leaves in left-to-right depth-first order. For bottom-up parsing with all actions at right ends this is the order of reductions.

### 3.4 Evaluating inherited attributes in LR parsers

Bottom-up parsers cannot place an action in the middle of a production without a **marker nonterminal** M → ε that forces a reduction at that point. For D → T {L.in = T.type} L, rewrite as D → T M L; M → ε {…}. The marker's rule can read the value stack below it (the attribute of T) and push L.in. Every L-attributed SDD can be converted to a form usable by an LR parser this way. Left-recursion removal likewise rewrites the SDD to keep the attribute flow correct.

## 4. Building syntax trees and DAGs

**Syntax tree** (abstract syntax tree): operators as interior nodes, operands as children; no parentheses or unit-production nodes. Construction is S-attributed:

| Production | Semantic rule |
| --- | --- |
| E → E₁ + T | E.node = new Node('+', E₁.node, T.node) |
| E → T | E.node = T.node |
| T → ( E ) | T.node = E.node |
| T → id | T.node = new Leaf(id, id.entry) |
| T → num | T.node = new Leaf(num, num.val) |

**DAG.** Before creating a node, search for an existing node with the same operator and the same children (value numbering via a hash table); if found, reuse it. Common subexpressions share a node.

**Worked example.** `a + a * (b - c) + (b - c) * d` parsed as ((a + (a * (b − c))) + ((b − c) * d)).
- Syntax tree: leaves a, a, b, c, b, c, d = 7; operators `*`, `−`, `+`, `−`, `*`, `+` = 6; **13 nodes**.
- DAG: leaves a, b, c, d = 4; operators: one shared `b − c`, `a * (b − c)`, `a + (…)`, `(b − c) * d`, final `+` = 5; **9 nodes**. (Hash-consing check by program: 9.)

The DAG feeds code generation and local common-subexpression elimination ([Intermediate code generation](intermediate-code-generation.md), [Optimization and data flow](optimization-and-dataflow.md)).

## 5. Where SDT is used

| Task | Attributes |
| --- | --- |
| Expression value | synthesised `val` |
| Type checking | synthesised `type` of expressions; inherited expected type |
| Symbol-table filling (declarations) | inherited `in` |
| Three-address code | synthesised `code`, `addr` (see next chapter) |
| Boolean code with jumps | inherited `true`/`false` labels |
| Array address computation | inherited/synthesised width and base |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Synthesised | computed from children | S-attributed checks |
| Inherited | computed from parent / siblings | declaration types, labels |
| S-attributed ⊂ L-attributed | L-attributed: depend only on parent and **left** siblings | classification |
| Bottom-up evaluation | S-attributed at reductions, post-order | LR + SDT |
| Postfix SDT | actions at the right end give postfix; at the start give prefix | output questions |
| DAG vs tree | share equal sub-expressions | node counting |
| Circular SDD | no valid order | well-definedness |
| Marker nonterminal | M → ε lets an LR parser run a mid-production action | LR with inherited attributes |

## GATE traps

- **Action position = output order.** An action at the end of the production runs after its children; one at the start runs before. Reordering the actions inside one production changes the output.
- With an LR parser, actions fire at **reduction time in reduction order**, not in the order of the input tokens. Unit productions without actions print nothing.
- An inherited attribute that depends on a **right** sibling makes the SDD non-L-attributed.
- An SDD can be non-S-attributed (it has inherited attributes) and still be L-attributed. "Every L-attributed SDD is S-attributed" is false.
- Syntax trees omit unit productions, parentheses and nonterminal chains; parse trees keep them. Count nodes carefully (leaves for every operand occurrence in a tree; shared in a DAG).
- Left-recursive grammars give left-associativity naturally; after left-recursion removal the **tree** leans right but the semantic actions must still compute the left-associative result.
- A translation scheme is evaluated during parsing without building the tree; an SDD is a specification and evaluation may use any order from the dependency graph.

## Connections

- [Parsing](parsing.md) — SDT rides on the parser: reductions in LR, expansions in LL.
- [Intermediate code generation](intermediate-code-generation.md) — three-address code and syntax trees are produced by SDDs.
- [Context-free languages and PDA](../11-theory-of-computation/context-free-languages-and-pda.md) — parse trees and ambiguity: an ambiguous grammar gives two annotated trees, hence two meanings.
- [Runtime environments](runtime-environments.md) — attributes such as addresses and offsets computed here are used for storage allocation.
- [Trees and BST](../07-data-structures/trees-and-bst.md) — attribute evaluation is a tree traversal (post-order for synthesised, pre-order for inherited).
- [Hashing](../08-algorithms/hashing.md) — DAG construction uses a hash table for value numbering.

## Practice

**Q1 (MCQ).** In an SDD, an attribute of a node computed from the attributes of its children is
(A) inherited (B) synthesised (C) global (D) lexical

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** By definition; inherited attributes come from the parent or siblings.

</details>

**Q2 (NAT).** The SDT `E → E₁ + T {print '+'}`, `E → T`, `T → T₁ * F {print '*'}`, `T → F`, `F → id {print id}` is applied by an LR parser to `a * b + c * d`. How many symbols are printed?

<details><summary>Answer</summary>

**Answer:** 7  
**Solution:** Output is `ab*cd*+` (a, b, `*`, c, d, `*`, `+`): 4 identifiers + 3 operators = 7 symbols.

</details>

**Q3 (MCQ).** What does the scheme E → T R; R → + T {print '+'} R | ε; T → num {print num.val} print for `1 + 2 + 3`?
(A) 123++ (B) 12+3+ (C) +12+3 (D) 1+2+3

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** T prints 1; R: T prints 2, then `+`; R again: T prints 3, then `+`; R → ε. Output `12+3+`.

</details>

**Q4 (MCQ).** Which statement is true?
(A) Every L-attributed SDD is S-attributed. (B) Every S-attributed SDD is L-attributed. (C) Inherited attributes can be evaluated at reduce time in any LR parser without markers. (D) L-attributed SDDs may depend on right siblings.

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** An S-attributed SDD has no inherited attributes, so the L-attributed condition holds vacuously.

</details>

**Q5 (NAT).** Using the grammar S → ( S ) {print 1} S | ε {print 2}, how many times is `2` printed for the input `(()())`?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** One `2` per ε-expansion: for n = 3 pairs there are n + 1 = 4 ε-uses. Output 2121212 (3 ones, 4 twos).

</details>

**Q6 (NAT).** Using E → E₁ # T {E.val = E₁.val × T.val}, T → T₁ & F {T.val = T₁.val + F.val}, the value of `2 # 3 & 5 # 6 & 4` is

<details><summary>Answer</summary>

**Answer:** 160  
**Solution:** (2 # (3 & 5)) # (6 & 4) = (2 × 8) × 10 = 160.

</details>

**Q7 (NAT).** In the syntax tree for `a + a * (b − c) + (b − c) * d` (parenthesised as ((a + (a*(b−c))) + ((b−c)*d))) the number of nodes is 13. How many nodes does the DAG have?

<details><summary>Answer</summary>

**Answer:** 9  
**Solution:** Leaves a, b, c, d (4); operators: `b−c` (shared), `a*(b−c)`, `a+(…)`, `(b−c)*d`, final `+` (5): 4 + 5 = 9.

</details>

**Q8 (MSQ).** Which rules for A → X Y Z make Y.i's definition acceptable in an L-attributed SDD?
(A) Y.i = A.i (B) Y.i = X.s (C) Y.i = Z.s (D) Y.i = X.s + A.i

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:** (A), (B), (D) use only the parent and the left sibling X. (C) uses the right sibling Z.

</details>
