# Boolean Algebra and Minimization

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Boolean algebra; Boolean minimization: algebraic technique; Karnaugh maps; Tabular minimization method
> **Prerequisites:** [Propositional logic](../01-discrete-mathematics/propositional-logic.md) · **Leads to:** [Combinational circuits](combinational-circuits.md), [Sequential circuits](sequential-circuits.md)

## Quick glance

- A Boolean function of $n$ variables has a truth table of $2^n$ rows, so there are **$2^{2^n}$ distinct functions** ($n=2$: 16, $n=3$: 256, $n=4$: 65536).
- Canonical forms: **SOP** = OR of minterms ($\Sigma m$), **POS** = AND of maxterms ($\Pi M$). Minterm $m_i$ and maxterm $M_i$ are complements: $M_i = m_i'$.
- Duality: swap AND/OR and 0/1 (keep literals). **De Morgan:** $(AB)' = A'+B'$, $(A+B)' = A'B'$.
- Consensus: $AB + A'C + BC = AB + A'C$ (the term $BC$ is redundant).
- **Self-dual** functions of $n$ variables: $2^{2^{n-1}}$. NAND alone and NOR alone are functionally complete; {AND, OR} is not.
- K-map: cells in **Gray-code order**; group 1, 2, 4, 8, 16 adjacent cells (wrap-around allowed); largest groups = prime implicants (PI); a PI is **essential (EPI)** if it alone covers some minterm.
- Don't-cares ($X$) may be used in groups to enlarge them, but never need to be covered.
- Quine–McCluskey = K-map made algorithmic: group minterms by number of 1s, combine pairs differing in one bit, then pick PIs from a chart (EPIs first, Petrick's method for cyclic leftovers).
- #1 trap: **number of minimal expressions is not the number of PIs**, and a minimal SOP uses EPIs + a smallest set of other PIs, not all PIs.

## 1. Boolean algebra basics

A Boolean algebra works on $\{0,1\}$ with AND ($\cdot$), OR ($+$), NOT ($'$). Think of it as logic where every wire is either low or high.

| Law | AND form | OR form |
|---|---|---|
| Identity | $A\cdot 1 = A$ | $A+0 = A$ |
| Null / domination | $A\cdot 0 = 0$ | $A+1 = 1$ |
| Idempotent | $AA = A$ | $A+A = A$ |
| Complement | $AA' = 0$ | $A+A' = 1$ |
| Involution | $(A')' = A$ | |
| Commutative | $AB = BA$ | $A+B = B+A$ |
| Associative | $(AB)C = A(BC)$ | $(A+B)+C = A+(B+C)$ |
| Distributive | $A(B+C) = AB+AC$ | $A+BC = (A+B)(A+C)$ |
| Absorption | $A(A+B) = A$ | $A + AB = A$ |
| Simplification | $A(A'+B) = AB$ | $A + A'B = A+B$ |
| De Morgan | $(AB)' = A'+B'$ | $(A+B)' = A'B'$ |
| Consensus | $(A+B)(A'+C)(B+C) = (A+B)(A'+C)$ | $AB + A'C + BC = AB + A'C$ |

**The second distributive law ($A+BC=(A+B)(A+C)$) has no analogue in ordinary arithmetic; it is the one students forget.**

### Duality
The **dual** of an expression is obtained by interchanging $+\leftrightarrow\cdot$ and $0\leftrightarrow 1$, leaving variables and complements unchanged. Every law above appears together with its dual. Do not confuse the *dual* $f^d(x_1,\dots,x_n)$ with the *complement* $f'$: $f^d(x_1,\dots,x_n) = [f(x_1',\dots,x_n')]'$.

**Self-dual function:** $f = f^d$, i.e. $f(x_1',\dots,x_n') = f(x_1,\dots,x_n)'$. Truth-table meaning: the output on row $i$ is the complement of the output on row $2^n-1-i$. So the first half of the table (rows $0\ldots 2^{n-1}-1$) is free, the second half is forced: **number of self-dual functions $= 2^{2^{n-1}}$** ($n=2$: 4, $n=3$: 16). Examples: $A$, $A'$, majority $AB+BC+CA$, $A\oplus B\oplus C$.

### De Morgan for bubbles
A NAND gate = OR gate with inverted inputs; a NOR gate = AND gate with inverted inputs. To convert any two-level AND-OR network to **NAND-NAND**, replace every gate by NAND (double inversion cancels). OR-AND converts to NOR-NOR.

### XOR / XNOR identities
$A\oplus B = A'B + AB'$, $A \odot B = (A\oplus B)' = AB + A'B'$.

| Identity | Value |
|---|---|
| $A\oplus 0$, $A\oplus 1$ | $A$, $A'$ |
| $A\oplus A$, $A\oplus A'$ | $0$, $1$ |
| Commutative, associative | $A\oplus B = B\oplus A$, $(A\oplus B)\oplus C = A\oplus(B\oplus C)$ |
| Distributes over AND | $A(B\oplus C) = AB \oplus AC$ |
| Complement | $(A\oplus B)' = A'\oplus B = A\oplus B'$ |
| Parity | $A_1\oplus\cdots\oplus A_n = 1$ iff an **odd** number of inputs are 1 |
| XNOR parity | $n$-input XNOR chain: depends on $n$ even/odd; for even $n$ it is 1 on an even number of 1s |

## 2. Canonical forms: minterms and maxterms

A **minterm** is an AND of all $n$ variables, each in true or complemented form, which is 1 on exactly one input row. Row number $i$ (binary of $ABC$) gives $m_i$: use $A$ if the bit is 1 and $A'$ if 0. A **maxterm** is an OR of all variables that is 0 on exactly one row: use $A'$ if the bit is 1 and $A$ if the bit is 0 (opposite of minterm).

For $n=3$: $m_5 = AB'C$ (101), $M_5 = A'+B+C'$ (it is 0 when $A=1,B=0,C=1$), and $M_5 = m_5'$.

**Worked example: canonical forms.** $F(A,B,C) = A'C + AB'$.

- $A'C = A'C(B+B') = A'BC + A'B'C = m_3 + m_1$.
- $AB' = AB'(C+C') = AB'C + AB'C' = m_5 + m_4$.
- $F = \Sigma m(1,3,4,5)$.
- Zeros are rows $\{0,2,6,7\}$, so $F = \Pi M(0,2,6,7)$.

**The set of minterm numbers and maxterm numbers of a function are complementary subsets of $\{0,\dots,2^n-1\}$.** $F'$ has $\Sigma m$ = the zeros of $F$.

Counting facts (all for $n$ variables):

| Question | Answer |
|---|---|
| Distinct Boolean functions | $2^{2^n}$ |
| Functions with exactly $k$ minterms | $\binom{2^n}{k}$ |
| Functions of 3 variables with exactly 3 minterms | $\binom{8}{3}=56$ |
| Self-dual functions | $2^{2^{n-1}}$ |
| Functions where $f(0,\dots,0)=0$ | $2^{2^n-1}$ |
| Functions that depend only on $A$ (of 3 variables) | 4 ($0,1,A,A'$) |
| Number of minterms in a product with $k$ literals | $2^{n-k}$ |

## 3. Functional completeness

A set of gates is **functionally complete** if every Boolean function can be built from it.

- Complete: {AND, NOT}, {OR, NOT}, **{NAND}**, **{NOR}**, {AND, OR, NOT}, {XOR, AND, constant 1}.
- Not complete: {AND, OR} (monotone only; cannot produce $A'$), {XOR, XNOR}, {AND, XOR} without constant 1 (every function built has output 0 on all-zero input, so cannot make $A'$ or constant 1), {NOT} alone.
- Constructions: $A' = (A\,\text{NAND}\,A)$; $AB = ((AB)')'$; $A+B = (A'\,\text{NAND}\,B')$. Similarly for NOR.

**Post's test (quick):** a set is complete iff it contains a function that is (i) not 0-preserving, (ii) not 1-preserving, (iii) not monotone, (iv) not self-dual, (v) not linear (XOR-affine). {AND, OR}: all five properties fail to be violated at (iii) and (v)... in particular both are monotone, so incomplete. NAND is not 0-preserving ($0\,\text{NAND}\,0 = 1$), not 1-preserving, not monotone, not self-dual, not linear, so complete.

**Checking a given set in GATE:** try to produce NOT and one of AND/OR. If you have a gate with constants available, e.g. a MUX with inputs tied to 0/1, you can get NOT directly: MUX with select $A$, data $(1,0)$ gives $A'$. A 2:1 MUX with constants alone is complete.

## 4. Algebraic minimization

No algorithm: apply the laws to merge terms, absorb terms, remove redundant (consensus) terms. Strategies: factor, use $A+A'B=A+B$, add redundant term then remove two others, check with a truth table.

**Example 1.** $F = AB + A'C + BC$ (verified equal to $AB+A'C$ on the truth table).

- $BC = BC(A+A') = ABC + A'BC$.
- $ABC$ is absorbed by $AB$, and $A'BC$ is absorbed by $A'C$.
- $F = AB + A'C$ (consensus theorem). 4 literals instead of 6.

**Example 2.** $F = \Sigma m(1,3,4,5)$ from Section 2.

- $F = A'B'C + A'BC + AB'C' + AB'C$
- Group 1+2: $A'C(B'+B) = A'C$. Group 3+4: $AB'(C'+C) = AB'$.
- $F = A'C + AB'$. This equals $A \oplus C$ when $B=0$? No: check $B$ doesn't appear, $F = A'C + AB'$ is not an XOR; keep it as is.

**Example 3.** $F = (A+B)(A+C)(B'+C')$... expand using $(A+B)(A+C) = A + BC$.

- $F = (A + BC)(B'+C') = AB' + AC' + BCB' + BCC' = AB' + AC'$ (since $BB'=0$ and $CC'=0$).
- $F = A(B'+C') = A(BC)'$ , i.e. a NAND followed by AND: 2 gates.

**Example 4 (reduce $A+A'B$).** $F = A'B + AB' + AB = A'B + A(B'+B) = A'B + A = A + B$.

Literal counts are how GATE measures "minimal" in SOP; an alternative is gate count or gate-input count, so read the question.

## 5. Karnaugh maps

A K-map is a truth table arranged so that **adjacent cells differ in exactly one variable**. Hence grouping adjacent 1s applies $XY + XY' = X$ geometrically. Rows and columns use **Gray code order: 00, 01, 11, 10** (not 00, 01, 10, 11). The map wraps around: left edge touches right edge, top touches bottom (a torus), and the four corners of a 4-variable map are adjacent.

```text
2 variables          3 variables (A rows, BC columns)
   B                       BC
A   0   1               A   00  01  11  10
0  m0  m1               0   m0  m1  m3  m2
1  m2  m3               1   m4  m5  m7  m6
```

```text
4 variables
        CD
AB     00   01   11   10
00     m0   m1   m3   m2
01     m4   m5   m7   m6
11     m12  m13  m15  m14
10     m8   m9   m11  m10
```

**5 variables** (A, B, C, D, E): draw two 4-variable maps, one for $A=0$ (minterms 0-15) and one for $A=1$ (16-31). A cell in one map is adjacent to the same cell position in the other map (as if the maps were stacked). Groups can span both layers only if they occupy the same cell positions in both.

### Rules for grouping
1. Group size must be a power of two (1, 2, 4, 8, 16). A group of $2^k$ cells eliminates $k$ variables.
2. Make each group as **large** as possible; overlapping is allowed.
3. Cover every 1 at least once; use as few groups as possible.
4. **Don't-cares** (X) may be treated as 1 to enlarge a group, never need to be covered, and a group of only X cells is never selected.

Terminology:

- **Implicant:** any valid group of 1s (a product term that implies $F$).
- **Prime implicant (PI):** an implicant not contained in any larger implicant.
- **Essential prime implicant (EPI):** a PI that covers at least one 1-cell (not an X) covered by no other PI.
- **Selection:** every EPI must be in the minimal cover; add the fewest remaining PIs for the uncovered 1s.

### Worked example A: 4 variables, six PIs, two EPIs

$F(A,B,C,D) = \Sigma m(0,1,2,5,6,7,8,9,10,14)$.

```text
        CD
AB     00  01  11  10
00      1   1   0   1      (m0,m1,m3,m2)
01      0   1   1   1      (m4,m5,m7,m6)
11      0   0   0   1      (m12,m13,m15,m14)
10      1   1   0   1      (m8,m9,m11,m10)
```

Prime implicants (maximal groups):

| PI | Cells | Size | Term |
|---|---|---|---|
| $B'D'$ | 0, 2, 8, 10 (four corners) | 4 | $B'D'$ |
| $CD'$ | 2, 6, 10, 14 (column CD=10) | 4 | $CD'$ |
| $B'C'$ | 0, 1, 8, 9 | 4 | $B'C'$ |
| $A'C'D$ | 1, 5 | 2 | $A'C'D$ |
| $A'BD$ | 5, 7 | 2 | $A'BD$ |
| $A'BC$ | 6, 7 | 2 | $A'BC$ |

Check each minterm for unique coverage:

- 9 is only in $B'C'$, so **$B'C'$ is essential**.
- 14 is only in $CD'$ (column 10, row 11), so **$CD'$ is essential**.
- 0, 1, 2, 6, 8, 9, 10, 14 are covered by the two EPIs. Left: 5 and 7.
- 5 and 7 are both in $A'BD$ (one term, 3 literals) vs. $A'C'D + A'BC$ (two terms): choose $A'BD$.

**Minimal SOP: $F = B'C' + CD' + A'BD$.** Total PIs = 6, EPIs = 2. ($B'D'$ is a PI that is not essential and is not used.)

### Worked example B: all PIs essential

$F = \Sigma m(0,1,2,3,5,7,8,9,11,14)$. PIs (computed by Quine–McCluskey and checked on the map): $A'B'$ (0,1,2,3), $B'C'$ (0,1,8,9), $B'D$ (1,3,9,11), $A'D$ (1,3,5,7), $ABCD'$ (14). Essential: $A'B'$ (covers 2), $B'C'$ (covers 8), $B'D$ (covers 11), $A'D$ (covers 5 and 7), $ABCD'$ (covers 14, a lone cell). Wait, 7 is only in $A'D$ and 5 only in $A'D$, so yes; $B'D$ covers 11 alone; so **5 PIs, all 5 essential**, and $F = A'B' + B'C' + B'D + A'D + ABCD'$.

(That $F$ has no simplification beyond the PIs because the isolated minterm 14 has no neighbour among the 1s.)

### Worked example C: don't-cares, two minimal covers

$F = \Sigma m(1,3,7,11,15) + d(0,2,5)$.

```text
        CD
AB     00  01  11  10
00      X   1   1   X      (m0,m1,m3,m2)
01      0   X   1   0      (m4,m5,m7,m6)
11      0   0   1   0      (m12,m13,m15,m14)
10      0   0   1   0      (m8,m9,m11,m10)
```

- $CD$ = column 11 (cells 3, 7, 15, 11): size 4, covers 11 and 15 uniquely, so essential.
- Left to cover: minterm 1. Candidates: $A'D$ (cells 1, 3, 5, 7, using X at 5) or $A'B'$ (cells 0, 1, 2, 3, using X at 0 and 2). Both have 2 literals.
- **$F = CD + A'D$ or $F = CD + A'B'$** (two equally minimal answers). PIs = 3: $CD$, $A'D$, $A'B'$. EPIs = 1.

### Worked example D: with a don't-care and an unusual wrap

$F = \Sigma m(4,8,10,11,12,15) + d(9,14)$.

```text
        CD
AB     00  01  11  10
00      0   0   0   0
01      1   0   0   0
11      1   0   1   X
10      1   X   1   1
```

PIs: $AB'$ (8, 9, 10, 11), $AC$ (10, 11, 14, 15), $BC'D'$ (4, 12), $AD'$ (8, 10, 12, 14). EPIs: $BC'D'$ (only PI with minterm 4), $AC$ (only PI with 15). Remaining minterm 8 is covered by $AB'$ or $AD'$ (2 literals each).

**Minimal: $F = BC'D' + AC + AB'$ or $F = BC'D' + AC + AD'$.** 4 PIs, 2 EPIs, 2 minimal expressions.

### Worked example E: 5-variable map

$F(A,B,C,D,E) = \Sigma m(0,1,2,3,8,9,12,13,16,17,18,19,24,25,28,29)$.

- Layer $A=0$ (minterms 0-15): ones at 0, 1, 2, 3, 8, 9, 12, 13.
- Layer $A=1$ (minterms 16-31): ones at 16, 17, 18, 19, 24, 25, 28, 29, which are the same positions (0-3, 8, 9, 12, 13) shifted by 16.
- Because both layers have the same pattern, any group in a layer can be doubled, eliminating $A$.
- Prime implicants: $B'C'$ (0,1,2,3,16,17,18,19), $C'D'E'$... compute: $D'$ with $C'$... the verified list is $B'C'$, $C'D'$ (0,1,8,9,16,17,24,25) and $BD'$ (8,9,12,13,24,25,28,29).
- Essential: $B'C'$ (only one covering 2, 3, 18, 19), $BD'$ (only one covering 12, 13, 28, 29). $C'D'$ covers 0, 1, 16, 17, 8, 9, 24, 25 but all of these are already covered by the EPIs, so it is a PI that is not essential and not needed.
- **$F = B'C' + BD'$** (independent of $A$ and $E$). 3 PIs, 2 EPIs.

### POS from the K-map

To get the minimal POS, group the **0s**, write each group as a sum term with variables **complemented relative to SOP** (a variable that is 0 throughout the group appears uncomplemented, 1 throughout appears complemented), and AND them. Equivalently minimise $F'$ in SOP and complement with De Morgan.

Example: $F = \Sigma m(1,3,4,5)$ over $A,B,C$ (Section 2). Zeros: 0, 2, 6, 7. Groups of zeros: $\{0,2\}$: $A=0,C=0$: sum term $(A+C)$. $\{6,7\}$: $A=1,B=1$: sum term $(A'+B')$. **$F = (A+C)(A'+B')$**. Check minterm 4 ($A=1,B=0,C=0$): $(1+0)(0+1) = 1$ ok. Minterm 0: $(0+0)=0$ ok.

## 6. Quine–McCluskey (tabular) method

The same job as a K-map but systematic, so it works for any number of variables. Steps:

1. List minterms (and don't-cares) in binary; **group by number of 1s**.
2. Compare adjacent groups; two terms that differ in exactly one bit combine, replacing that bit by `-`. Tick the terms used.
3. Repeat on the new column (dashes must be in the same positions) until nothing combines. Unticked terms are the PIs.
4. Build a **PI chart** (rows = PIs, columns = minterms only, not don't-cares). Select EPIs (columns with a single X), remove covered columns, then cover the rest with fewest PIs (Petrick's method if cyclic).

### Worked example: $F=\Sigma m(0,1,2,5,6,7,8,9,10,14)$ (same as Example A)

**Column 1 (by number of ones):**

| Ones | Minterm | Binary |
|---|---|---|
| 0 | 0 | 0000 |
| 1 | 1 | 0001 |
| 1 | 2 | 0010 |
| 1 | 8 | 1000 |
| 2 | 5 | 0101 |
| 2 | 6 | 0110 |
| 2 | 9 | 1001 |
| 2 | 10 | 1010 |
| 3 | 7 | 0111 |
| 3 | 14 | 1110 |

**Column 2 (pairs):**

| Pair | Pattern |
|---|---|
| 0,1 | 000- |
| 0,2 | 00-0 |
| 0,8 | -000 |
| 1,5 | 0-01 |
| 1,9 | -001 |
| 2,6 | 0-10 |
| 2,10 | -010 |
| 8,9 | 100- |
| 8,10 | 10-0 |
| 5,7 | 01-1 |
| 6,7 | 011- |
| 6,14 | -110 |
| 10,14 | 1-10 |

Terms $0\!-\!1$, $2$-$6$, etc. are ticked as they are used; the **unticked** ones from this column are $0\text{-}01$ (1,5), $01\text{-}1$ (5,7) and $011\text{-}$ (6,7), which are PIs of size 2.

**Column 3 (quads):**

| Quad | Pattern | Formed from |
|---|---|---|
| 0,1,8,9 | -00- | 000- with 100-; -000 with -001 |
| 0,2,8,10 | -0-0 | 00-0 with 10-0; -000 with -010 |
| 2,6,10,14 | --10 | 0-10 with 1-10; -010 with -110 |

No further combination, so all three are PIs.

**PIs:** $-00-=B'C'$, $-0-0=B'D'$, $--10=CD'$, $0\text{-}01=A'C'D$, $01\text{-}1=A'BD$, $011\text{-}=A'BC$ (matches the K-map: 6 PIs).

**PI chart** (✓ = PI covers the minterm):

| PI | 0 | 1 | 2 | 5 | 6 | 7 | 8 | 9 | 10 | 14 |
|---|---|---|---|---|---|---|---|---|---|---|
| $B'C'$ (0,1,8,9) | ✓ | ✓ | | | | | ✓ | ✓ | | |
| $B'D'$ (0,2,8,10) | ✓ | | ✓ | | | | ✓ | | ✓ | |
| $CD'$ (2,6,10,14) | | | ✓ | | ✓ | | | | ✓ | ✓ |
| $A'C'D$ (1,5) | | ✓ | | ✓ | | | | | | |
| $A'BD$ (5,7) | | | | ✓ | | ✓ | | | | |
| $A'BC$ (6,7) | | | | | ✓ | ✓ | | | | |

- Column 9 has a single ✓ (in $B'C'$): EPI. Column 14: only $CD'$: EPI.
- Those two cover 0, 1, 2, 6, 8, 9, 10, 14. Remaining: 5, 7. Column 5: $A'C'D$ or $A'BD$; column 7: $A'BD$ or $A'BC$. $A'BD$ covers both: choose it.
- **$F = B'C' + CD' + A'BD$.**

### Petrick's method (cyclic charts)

When no essential PIs remain (a cyclic chart), write a Boolean product of sums: for each remaining minterm, the sum of PIs that cover it; multiply out, using absorption ($X+XY=X$); each product term is a valid cover, pick the one with fewest PIs/literals.

**Example.** $F(A,B,C) = \Sigma m(0,1,2,5,6,7)$. PIs: $P_1 = A'B'$ (0,1), $P_2 = B'C$ (1,5), $P_3 = AC$ (5,7), $P_4 = AB$ (6,7), $P_5 = BC'$ (2,6), $P_6 = A'C'$ (0,2). Each minterm is covered by exactly two PIs, so there are no EPIs.

Petrick: $(P_1+P_6)(P_1+P_2)(P_5+P_6)(P_2+P_3)(P_4+P_5)(P_3+P_4)$ for minterms 0, 1, 2, 5, 6, 7.
Two minimal covers of 3 PIs exist: $\{P_1, P_3, P_5\}$ = $A'B' + AC + BC'$ and $\{P_2, P_4, P_6\}$ = $B'C + AB + A'C'$. Verify the first: $A'B'$ (0,1), $AC$ (5,7), $BC'$ (2,6): union $\{0,1,2,5,6,7\}$ ok. **A cyclic function has multiple minimal solutions.**

With don't-cares in QM, include them in column 1 so PIs can grow, but **do not list them as columns of the PI chart**.

## 7. Choosing a method in the exam

| Situation | Method |
|---|---|
| 2-4 variables, quick | K-map |
| 5 variables, few minterms | K-map (two layers) or counting by hand |
| Count PIs / EPIs | K-map, enumerate maximal groups; for each 1-cell count covering PIs |
| Many variables, need algorithm | QM |
| Check equivalence of two expressions | compare minterm sets (truth table) |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Number of functions | $2^{2^n}$ | counting questions |
| Self-dual | $2^{2^{n-1}}$ | counting; check $f(x')=f(x)'$ |
| $M_i$ vs $m_i$ | $M_i = m_i'$ | converting $\Sigma\leftrightarrow\Pi$ |
| Consensus | $AB+A'C+BC = AB+A'C$ | algebraic minimization |
| Absorption | $A+AB=A$; $A+A'B = A+B$ | quick reductions |
| De Morgan | $(XY)'=X'+Y'$ | NAND/NOR conversion |
| Group of $2^k$ cells | eliminates $k$ variables | K-map |
| Complete gate sets | NAND; NOR; {AND, NOT}; {OR, NOT} | completeness |
| Gray order | 00, 01, 11, 10 | K-map axes |
| Parity | XOR of $n$ vars = 1 iff odd number of 1s | parity trees |
| Max PIs of a 4-var function | 8 (e.g. checkerboard: all 8 minterms isolated gives 8 PIs, all essential) | counting |

## GATE traps

- **Gray order on the axes.** Using 00, 01, 10, 11 breaks adjacency. Always 00, 01, 11, 10.
- **Counting essential PIs:** a PI is essential only because of a 1-cell, not a don't-care. A PI covered entirely by other PIs, or covering only X cells, is not essential.
- **Prime implicants must be maximal:** a group of 2 inside a group of 4 is an implicant, not a prime implicant.
- **POS from zeros:** variable polarity flips. A group of 0s where $A=1,B=0$ gives $(A'+B)$.
- **Don't-cares** enlarge groups but never force coverage; a cell set to 1 for one group may be 0 for the complement function.
- **Duality is not complement.** The dual of $A+BC'$ is $A(B+C')$, not $A'(B'+C)$.
- **Self-dual count** uses $2^{n-1}$ in the exponent; do not answer $2^{2^n}/2$.
- **{AND, OR} is not complete;** with constant 1 and XOR you can get NOT, but {AND, XOR} without constants cannot.
- **Minterm numbering:** $A$ is the MSB. $F(A,B,C)$, $m_3 = A'BC$ (011), not $AB'C$.
- **NAND-NAND = AND-OR; NOR-NOR = OR-AND.** Converting SOP to NAND-NAND does not change literals.
- A function can have several minimal SOPs; "the number of minimal expressions" differs from "the number of PIs".

## Connections

- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — Boolean algebra is propositional logic with 0/1; minterms are full conjunctions, and normal forms (DNF/CNF) are SOP/POS.
- [Posets and lattices](../01-discrete-mathematics/posets-and-lattices.md) — a Boolean algebra is a complemented distributive lattice; the subset lattice of $n$ elements has $2^n$ elements.
- [Combinational circuits](combinational-circuits.md) — each minimised expression becomes a gate network; MUX/decoder realisations use minterms directly.
- [Sequential circuits](sequential-circuits.md) — flip-flop input equations are minimised with K-maps (using unused states as don't-cares).
- [ALU and control unit](../10-computer-organization/alu-and-control-unit.md) — control signals are Boolean functions of opcode/state bits (hardwired control).
- [Pumping lemma](../11-theory-of-computation/pumping-lemma.md) — not related directly; see [Regular languages](../11-theory-of-computation/regular-languages-and-finite-automata.md) for FSM minimization, which mirrors state-reduction in sequential design.
- [Optimization and dataflow](../12-compiler-design/optimization-and-dataflow.md) — algebraic simplification and common-subexpression ideas parallel Boolean identities.

## Practice

**Q1 (MCQ).** How many Boolean functions of 3 variables are self-dual?
(a) 8  (b) 16  (c) 64  (d) 256

<details><summary>Answer</summary>

**Answer:** (b) 16  
**Solution:** A self-dual function is fixed by its first $2^{n-1} = 4$ truth-table rows; the remaining 4 rows are forced to be complements of the mirror rows. Count $= 2^4 = 16$.

</details>

**Q2 (NAT).** $F(A,B,C,D) = \Sigma m(0,2,8,10)$ is simplified. How many literals are in its minimal SOP?

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** Minterms 0, 2, 8, 10 are the four corners of the K-map, forming one group of 4. Variables $B$ and $D$ are 0 throughout, $A$ and $C$ vary. $F = B'D'$: 2 literals.

</details>

**Q3 (MCQ).** Which is the minimal SOP of $F = AB + A'C + BC + B'C'$... use simplification of $F = AB + A'C + BC$?
(a) $AB + A'C$  (b) $AB + BC$  (c) $A'C + BC$  (d) $A + C$

<details><summary>Answer</summary>

**Answer:** (a)  
**Solution:** $BC$ is the consensus of $AB$ and $A'C$ (variable $A$ appears true in one and complemented in the other, leaving $BC$). By the consensus theorem, $BC$ is redundant. $F = AB + A'C$.

</details>

**Q4 (NAT).** For $F(A,B,C,D) = \Sigma m(0,1,2,5,6,7,8,9,10,14)$, find (number of prime implicants) $-$ (number of essential prime implicants).

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** From the worked example, there are 6 PIs ($B'C'$, $B'D'$, $CD'$, $A'C'D$, $A'BD$, $A'BC$) and 2 EPIs ($B'C'$ for minterm 9, $CD'$ for minterm 14). $6-2 = 4$.

</details>

**Q5 (MSQ).** Which of the following gate sets are functionally complete?
(a) {NAND}  (b) {NOR}  (c) {AND, OR}  (d) {AND, XOR}  (e) {XOR, AND, constant 1}

<details><summary>Answer</summary>

**Answer:** (a), (b), (e)  
**Solution:** NAND and NOR each give NOT ($x$ NAND $x = x'$) and AND/OR. {AND, OR} gives only monotone functions (cannot make $x'$). {AND, XOR} without constants: every function built is 0 at the all-zero input, so cannot realise $x'$ or 1. With constant 1, $x' = x\oplus 1$ and NOT+AND is complete.

</details>

**Q6 (NAT).** $F(A,B,C,D) = \Sigma m(1,3,7,11,15) + d(0,2,5)$. How many literals does a minimal SOP have?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** $CD$ is essential (covers 11, 15). Minterm 1 is covered by $A'D$ or $A'B'$ (both using don't-cares): 2 literals. $F = CD + A'D$: $2+2 = 4$ literals.

</details>

**Q7 (NAT).** How many of the $2^{16}$ Boolean functions of 4 variables have exactly 3 minterms?

<details><summary>Answer</summary>

**Answer:** 560  
**Solution:** Choose which 3 of the 16 rows are 1: $\binom{16}{3} = \dfrac{16\cdot15\cdot14}{6} = 560$.

</details>

**Q8 (NAT).** Using Quine–McCluskey on $F(A,B,C) = \Sigma m(0,1,2,5,6,7)$, how many essential prime implicants exist, and how many distinct minimal SOP expressions (each with 3 two-literal terms)?

<details><summary>Answer</summary>

**Answer:** 0 EPIs; 2 minimal expressions  
**Solution:** The 6 PIs $A'B', B'C, AC, AB, BC', A'C'$ each cover two minterms and every minterm is covered by exactly two PIs, so no column has a single tick: a cyclic chart with 0 EPIs. Covering 6 minterms with 2-minterm PIs needs at least 3 PIs, and exactly two disjoint covers exist: $\{A'B', AC, BC'\}$ and $\{B'C, AB, A'C'\}$ (found with Petrick's method).

</details>

[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)
