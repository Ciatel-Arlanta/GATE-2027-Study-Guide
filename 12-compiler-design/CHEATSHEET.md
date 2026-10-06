# Compiler Design — revision sheet

| Phase / item | Recall |
|---|---|
| Lexer | Regex/specification → tokens; longest match, priority resolves ties |
| FIRST | Terminals that can begin strings derived from a grammar symbol |
| FOLLOW(A) | Terminals that can appear immediately after nonterminal A |
| LL(1) | Predictive top-down; no left recursion; table cell conflicts mean not LL(1) |
| LR | Bottom-up shift/reduce; handles larger grammar class |
| TAC | Three-address instructions, temporaries, labels |
| Basic block | Maximal straight-line code with one entry and one exit |
| Liveness | Live if value may be used before redefinition |
| Common subexpression | Reuse expression only if operands unchanged |
| Activation record | Parameters, return address, saved state, locals/temporaries |

Left factoring removes common prefixes; left-recursion elimination transforms grammar shape. They aid parsing but do not change language. Register interference graph connects simultaneously live values; colouring approximates register assignment.
