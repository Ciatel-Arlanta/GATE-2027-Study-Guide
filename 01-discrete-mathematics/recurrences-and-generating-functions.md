# Recurrences and generating functions

> **Paper:** CS · **Priority:** P0 · **Plan topics:** recurrence relations, generating functions
> **Prerequisites:** [Combinatorics](combinatorics.md) · **Leads to:** [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md)

## Quick glance
- A recurrence defines terms using earlier terms plus base cases.
- For $T(n)=aT(n/b)+f(n)$, compare $f(n)$ with $n^{\log_b a}$ (Master theorem).
- Linear homogeneous recurrences: solve characteristic equation; repeated roots add polynomial factors.
- Ordinary generating function: $A(x)=\sum_{n\ge0}a_nx^n$.
- $1/(1-x)=\sum x^n$, and $1/(1-x)^2=\sum(n+1)x^n$.

## 1. Unrolling and characteristic roots
For $T(n)=T(n-1)+1$, $T(1)=1$, repeated substitution gives $T(n)=n$. For Fibonacci $F_n=F_{n-1}+F_{n-2}$, characteristic equation $r^2-r-1=0$ has roots $(1\pm\sqrt5)/2$, so growth is $\Theta(\phi^n)$ for the naive recursive algorithm (the sequence itself is also $\Theta(\phi^n)$).

For $a_n=5a_{n-1}-6a_{n-2}$, characteristic roots are 2 and 3, so $a_n=c_1 2^n+c_2 3^n$; use initial values to solve constants.

## 2. Divide-and-conquer recurrence
For $T(n)=2T(n/2)+n$, there are $\log_2 n$ levels, each doing total work n, so $T(n)=\Theta(n\log n)$. This is also Master theorem case 2.

## 3. Generating functions
If $a_n=1$ for all n, $A(x)=1+x+x^2+\dots=1/(1-x)$. If $a_n=n$, then $A(x)=x/(1-x)^2$. Generating functions turn sequence addition into function addition and shifts into multiplication by x; they are useful for counting combinations and solving recurrences.

## GATE traps
- Recurrence needs initial/base values to determine a unique sequence.
- Master theorem does not apply to every recurrence (e.g. unequal subproblem sizes or irregular terms).
- Distinguish the running time of naive recursive Fibonacci from the value of $F_n$.
- Check indexing when shifting a generating-function series.

## Connections
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — recurrence solution predicts algorithm cost.
- [Combinatorics](combinatorics.md) — generating functions encode counts.
- [Dynamic programming](../08-algorithms/dynamic-programming.md) — memoisation avoids repeated recurrence work.

## Practice
**Q1.** Solve $T(n)=T(n/2)+1$, $T(1)=1$ for powers of 2.
<details><summary>Answer</summary> $T(n)=1+\log_2 n=\Theta(\log n)$.</details>

**Q2.** Solve $a_n=3a_{n-1}$, $a_0=2$.
<details><summary>Answer</summary> $a_n=2\cdot3^n$.</details>

**Q3.** Coefficient of $x^4$ in $1/(1-x)^2$?
<details><summary>Answer</summary> 5, since coefficient is $n+1$.</details>
