# Compiler Design checkpoint

**Q1.** Lexer sees two rules matching the same longest prefix. Which typically wins?
<details><summary>Answer</summary> The rule with higher priority/earlier specification, according to lexer convention.</details>

**Q2.** Why remove left recursion for a basic recursive-descent parser?
<details><summary>Answer</summary> It would recurse without consuming input and fail to make progress.</details>

**Q3.** What does FOLLOW(A) contain?
<details><summary>Answer</summary> Terminals that may immediately follow nonterminal A in a sentential form; end marker may occur for start context.</details>

**Q4.** Two productions enter same LL(1) table cell. What does that imply?
<details><summary>Answer</summary> The grammar is not LL(1) as written.</details>

**Q5.** Define a basic block.
<details><summary>Answer</summary> Maximal straight-line sequence with a single entry and exit.</details>

**Q6.** When can a computed expression safely be reused?
<details><summary>Answer</summary> When its operands have not changed and the computation has no relevant side effects.</details>

**Q7.** What does a liveness analysis answer?
<details><summary>Answer</summary> Whether a variable's current value may be used along some future path before redefinition.</details>

**Q8.** What does a call stack activation record represent?
<details><summary>Answer</summary> State for one active procedure invocation, including return information and locals/temporaries.
</details>

≥80%: proceed; below 60%: revisit FIRST/FOLLOW and parsing tables.
