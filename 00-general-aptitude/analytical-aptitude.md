# Analytical Aptitude: Deduction, Logical Reasoning, and Structural Puzzles

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** deduction, induction, analogy, numerical relations, reasoning puzzles
> **Prerequisites:** [Verbal aptitude](verbal-aptitude.md) · **Leads to:** [Propositional logic](../01-discrete-mathematics/propositional-logic.md), [First-order logic](../01-discrete-mathematics/first-order-logic.md)
> **PYQ weightage:** High · 35 Qs since 2015 (22 since 2021) · DA: 3 · Trend: rising · Most asked: Logical Reasoning & Deductions, Syllogisms (Statements Follow), Direction Sense, Seating Arrangements & Series

## Quick glance

- **Exam focus:** Multi-premise deduction, syllogisms ("statements follow"), linear/circular seating arrangements, and direction-distance problems appear consistently as 1-mark and 2-mark MCQs; recent papers (2024–2026) show a sharp increase in analytical reasoning questions.
- **Deductive validity:** A deduction is valid iff whenever all premises are true, the conclusion is impossible to be false. If even one counterexample exists, the deduction is invalid.
- **Contrapositive equivalence:** The conditional $P \to Q$ is logically equivalent to its contrapositive $\neg Q \to \neg P$. The converse ($Q \to P$) and inverse ($\neg P \to \neg Q$) are common logical fallacies.
- **Necessary vs. Sufficient:** "$P$ is sufficient for $Q$" means $P \to Q$. "$P$ is necessary for $Q$" means $Q \to P$ (or $\neg P \to \neg Q$). "$P$ only if $Q$" means $P \to Q$.
- **Syllogisms rule:** "Some $A$ are $B$" means $A \cap B \ne \emptyset$ (at least one). It does *not* imply "Some $A$ are not $B$". A conclusion is valid only if true in *every* valid Venn diagram.
- **Linear arrangement strategy:** Draw fixed slots $1 \dots n$. Anchor the most constrained entity first (e.g., fixed ends, adjacent pairs). Distinguish "immediately left" ($pos(A) = pos(B) - 1$) from "left" ($pos(A) < pos(B)$).
- **Circular arrangements (facing center):** A person facing the center has **Right = Counter-Clockwise** and **Left = Clockwise**. For opposite seats with $n$ even, there are $\frac{n}{2} - 1$ persons between them on either side.
- **Direction vector tracking:** Map movements as 2D vectors: North $= +y$, South $= -y$, East $= +x$, West $= -x$. Net straight-line distance is $d = \sqrt{(\Delta x)^2 + (\Delta y)^2}$.
- **Letter coding ranks:** Memorize alphabetical ranks $A=1 \dots Z=26$, and reverse ranks $A=26 \dots Z=1$. Note that $\text{rank}(L) + \text{reverse\_rank}(L) = 27$.
- **#1 trap:** Assuming the converse in reading deductions: believing that "all students stand when the teacher enters" implies "if students stand, the teacher must be in the room".

## 1. Deductive and Inductive Logic: Conditionals and Inferences [PYQ: High]

Logical reasoning in GATE distinguishes between **deductive** reasoning (which guarantees certainty from premises) and **inductive** reasoning (which infers probable general rules from specific observations).

```text
       Deductive Reasoning:   General Rule + Case   ──► Certain Conclusion
       Inductive Reasoning:   Observed Cases        ──► Probable Rule (Conjecture)
```

### 1.1 Conditional Statements and Valid Forms
A conditional proposition "If $P$, then $Q$" ($P \to Q$) states that hypothesis $P$ implies conclusion $Q$.

| Form | Symbolic | Status | Common Fallacy Name |
|---|---|---|---|
| **Original** | $P \to Q$ | Given premise | — |
| **Contrapositive** | $\neg Q \to \neg P$ | **Logically equivalent** | Valid deduction |
| **Converse** | $Q \to P$ | **Not equivalent** | Fallacy of Affirming the Consequent |
| **Inverse** | $\neg P \to \neg Q$ | **Not equivalent** | Fallacy of Denying the Antecedent |

### 1.2 Keyword Translation
- "$P$ **implies** $Q$" / "$P$ **only if** $Q$" / "$P$ **is sufficient for** $Q$": $P \to Q$.
- "$P$ **is necessary for** $Q$": $Q \to P$.
- "$P$ **if and only if** $Q$" ($P \iff Q$): $(P \to Q) \land (Q \to P)$.
- "$P$ **unless** $Q$": $\neg Q \to P$ (or $P \lor Q$).

**Worked example 1 (Multi-premise conditional deduction).** Consider the following rules enforced in a datacenter:
1. "If server cluster Alpha is active, then backup cooling unit C is turned ON." ($A \to C$)
2. "If backup cooling unit C is turned ON, then auxiliary generator G is running." ($C \to G$)
3. "At 03:00 AM, auxiliary generator G was NOT running." ($\neg G$)

Determine whether server cluster Alpha was active at 03:00 AM.

*Step 1: Chain the conditionals.*
By hypothetical syllogism (transitivity of implication):
$$A \to C \quad\text{and}\quad C \to G \implies A \to G$$
"If Alpha is active, then generator G must be running."

*Step 2: Apply Modus Tollens with the observed fact.*
We are given $\neg G$ (generator G was not running). By contrapositive $\neg G \to \neg A$. Since $\neg G$ is true, $\neg A$ must be true.  
*Conclusion:* Server cluster Alpha was **definitely NOT active** at 03:00 AM.

## 2. Syllogisms and Categorical Propositions [PYQ: High]

A syllogism evaluates the logical consequence of quantified categorical statements.

```text
   Universal Affirmative (A)     Universal Negative (E)     Particular Affirmative (I)
       "All S are P"                  "No S is P"                 "Some S are P"
          ┌───────┐                  ┌───┐     ┌───┐                  ┌───┬───┐
          │ P ┌─┐ │                  │ S │     │ P │                  │ S │∩│ P │
          │   │S│ │                  └───┘     └───┘                  └───┴───┘
          └───┴─┴─┘                 S ∩ P = ∅                  S ∩ P ≠ ∅
```

### 2.1 The Four Standard Categorical Propositions
1. **A (All $S$ are $P$):** $S \subseteq P$.
2. **E (No $S$ is $P$):** $S \cap P = \emptyset$.
3. **I (Some $S$ are $P$):** $S \cap P \ne \emptyset$ ("at least one element exists in common").
4. **O (Some $S$ are not $P$):** $S \setminus P \ne \emptyset$ ("at least one element of $S$ is outside $P$").

### 2.2 Venn Diagram Test for Syllogistic Validity
To check if a conclusion follows: draw the **minimal condition** Venn diagram for all premises, then search for any alternative valid diagram that satisfies all premises but violates the proposed conclusion. If an alternative diagram exists, the conclusion **does not follow**.

**Worked example 2 (Syllogism evaluation).**  
*Statements:*  
1. All database engines ($D$) are software systems ($S$). ($D \subseteq S$)  
2. Some software systems ($S$) are open-source projects ($O$). ($S \cap O \ne \emptyset$)  
3. No open-source project ($O$) contains proprietary blobs ($B$). ($O \cap B = \emptyset$)  

*Conclusions:*  
- **Conclusion I:** Some software systems are not proprietary blobs.  
- **Conclusion II:** Some database engines are open-source projects.  

*Analysis:*  
- For **Conclusion I**: By Premise 2, there exists $x \in S \cap O$. Since $x \in O$ and $O \cap B = \emptyset$, $x \notin B$. Thus $x \in S$ and $x \notin B$, proving at least one software system is not a proprietary blob.  
  *Verdict:* **Conclusion I follows definitely.**  
- For **Conclusion II**: $D \subseteq S$ and $S \cap O \ne \emptyset$ do not force $D$ and $O$ to intersect. A diagram where $D$ and $O$ are disjoint sets inside $S$ satisfies all premises.  
  *Verdict:* **Conclusion II does NOT follow.**

## 3. Seating Arrangements, Ordering, and Linear Puzzles [PYQ: Medium]

Arrangement problems require placing entities into discrete slots subject to relational constraints.

```text
   Linear Arrangement (Left to Right):
   Slot:   [ 1 ]     [ 2 ]     [ 3 ]     [ 4 ]     [ 5 ]     [ 6 ]
          ───────► "Left of" (rank <)              ───────► "Right of" (rank >)
          Adjacent: |rank(A) - rank(B)| = 1
```

### 3.1 Slot-Filling Strategy
1. **Identify the frame:** Count total entities $n$. Draw $n$ numbered positions.
2. **Anchor definite facts:** Place entities assigned to fixed endpoints or unique slots.
3. **Block formulation:** Treat adjacent entities as a consolidated super-element (e.g., "$A$ and $B$ sit together" $\to [AB]$ or $[BA]$).
4. **Test remaining options:** Narrow possibilities by checking contradictory constraints.

### 3.2 Circular Arrangements
When entities face the center of a circular table:
- Looking inward: **Right** is counter-clockwise; **Left** is clockwise.
- When entities face outward: **Right** is clockwise; **Left** is counter-clockwise.

**Worked example 3 (Vertical rack server placement).** Six servers $S_1, S_2, S_3, S_4, S_5, S_6$ are installed in a vertical rack with slots numbered 1 (bottom) to 6 (top).
- $S_6$ is placed in slot 6.
- $S_3$ is located on an odd-numbered slot.
- $S_1$ is immediately above $S_3$.
- $S_5$ is immediately below $S_2$.
- $S_4$ is located at a slot lower than $S_5$.

Find the exact slot occupied by each server.

*Step 1: Analyze odd slots for $S_3$.*
Odd slots are $\{1, 3, 5\}$. If $S_3 = 5$, then $S_1 = 6$, conflicting with $S_6$. Thus $S_3 \in \{1, 3\}$.

*Step 2: Test candidate $S_3 = 3$.*
If $S_3 = 3 \implies S_1 = 4$. Empty slots remaining: $\{1, 2, 5\}$. Adjacent pair for $S_5$ (immediately below $S_2$) forces $S_5 = 1, S_2 = 2$. Remaining slot for $S_4 = 5$. But constraint "$S_4 < S_5$" fails ($5 \not< 1$). Hence $S_3 \ne 3$.

*Step 3: Establish unique valid placement with $S_3 = 1$.*
$S_3 = 1 \implies S_1 = 2$. Empty slots: $\{3, 4, 5\}$. To satisfy $S_5$ below $S_2$ and $S_4 < S_5$, we must place $S_4 = 3, S_5 = 4, S_2 = 5$. Check: $S_4 < S_5 \implies 3 < 4$ (True).  
*Resulting layout:* Slot 1: $S_3$, Slot 2: $S_1$, Slot 3: $S_4$, Slot 4: $S_5$, Slot 5: $S_2$, Slot 6: $S_6$.

## 4. Direction Sense and Path Geometry [PYQ: Medium]

Direction problems track movement along cardinal and ordinal axes, requiring geometric reasoning.

```text
                             North (+y)
                                 ▲
                     NW          │          NE
                         \       │       /
                          \      │      /
         West (-x) ◄─────────────┼─────────────► East (+x)
                          /      │      \
                         /       │       \
                     SW          │          SE
                                 ▼
                             South (-y)
```

### 4.1 Turn and Angle Conventions
- **Turn Right:** Rotate $90^\circ$ clockwise. **Turn Left:** Rotate $90^\circ$ counter-clockwise.
- Intermediate directions (NE, NW, SE, SW) bisect the cardinal axes at $45^\circ$.
- Sun shadows: In the **morning (sunrise East)**, shadows fall **West**. In the **evening (sunset West)**, shadows fall **East**.

### 4.2 Vector Coordinates
Track all movements as $(\Delta x, \Delta y)$ coordinates from origin $(0, 0)$:
- Moving East: $x \leftarrow x + d$; Moving West: $x \leftarrow x - d$.
- Moving North: $y \leftarrow y + d$; Moving South: $y \leftarrow y - d$.
- Final straight-line distance: $d = \sqrt{x^2 + y^2}$.

**Worked example 4 (Multi-leg path with turns).** An autonomous delivery vehicle starts at depot $O(0, 0)$ facing North. It performs the following maneuvers:
1. Drives 12 m North $\to (0, 12)$, facing North.
2. Turns right and travels 5 m $\to (5, 12)$, facing East.
3. Turns right and travels 4 m $\to (5, 8)$, facing South.
4. Turns left and travels 10 m $\to (15, 8)$, facing East.
5. Turns right and travels 8 m $\to (15, 0)$, facing South.

Final facing direction: **South**.  
Distance from depot: $d = \sqrt{15^2 + 0^2} = 15\text{ m}$ due East.

## 5. Series, Coding–Decoding, Analogies, and Relations [PYQ: High]

### 5.1 Number and Letter Series
- **Difference analysis:** Compute successive differences $\Delta_1 = x_{k+1} - x_k$. If not constant, compute second differences $\Delta_2$. A sequence generated by a polynomial of degree $d$ has constant $d$-th differences.
- **Letter positions & dual ranks:** $A=1, B=2, \dots, Z=26$. Reverse ranks: $A=26, \dots, Z=1$. Identity: $\text{Forward}(L) + \text{Reverse}(L) = 27$. Opposite letter pairs: $A \leftrightarrow Z$, $B \leftrightarrow Y$, $C \leftrightarrow X$, $M \leftrightarrow N$.

### 5.2 Coding Patterns
- **Shift cipher:** Letter mapped to $(\text{rank} + k) \pmod{26}$.
- **Opposite letters:** Letter of rank $r$ mapped to $27 - r$.
- **Transposition / Interleaving:** Reversing or swapping adjacent character positions.

**Worked example 5 (Alternating positional shift cipher).** In a cipher scheme, the word `BYTE` is encoded as `DWVC`. Determine the coding rule and encode the word `DATA`.

*Step 1: Check letter shifts.*
`B`(2) $\to$ `D`(4): $+2$. `Y`(25) $\to$ `W`(23): $-2$. `T`(20) $\to$ `V`(22): $+2$. `E`(5) $\to$ `C`(3): $-2$.  
*Rule:* Alternating shift: $+2$ at odd positions, $-2$ at even positions.

*Step 2: Encode `DATA`.*
- 1st `D`(4) $+ 2 = 6 \to$ `F`
- 2nd `A`(1) $- 2 \equiv 25 \pmod{26} \to$ `Y`
- 3rd `T`(20) $+ 2 = 22 \to$ `V`
- 4th `A`(1) $- 2 \to$ `Y`  
*Coded result:* **`FYVY`**.

## Formulas and facts to memorise

| Concept | Rule / Identity | When to use in GATE |
|---|---|---|
| Contrapositive | $(P \to Q) \equiv (\neg Q \to \neg P)$ | Valid inference from conditional rule |
| Converse fallacy | $(P \to Q) \not\implies (Q \to P)$ | Rejecting invalid deductions |
| Particular quantifier | "Some" $\equiv \exists x \, (x \in A \land x \in B)$ | Syllogisms with existential claims |
| Syllogism disproof | Single counterexample Venn diagram | Rejecting proposed categorical conclusions |
| Linear "Between" | $A$ between $B$ and $C \implies B < A < C$ or $C < A < B$ | Non-strict intermediate slot placement |
| Circular facing center | Right is CCW; Left is CW | Determining relative position around table |
| Euclidean displacement | $d = \sqrt{(\Delta x)^2 + (\Delta y)^2}$ | Straight-line distance after multiple turns |
| Letter rank duality | $\text{Forward} + \text{Reverse} = 27$ | Opposite-letter coding and decoding |

## GATE traps

- **Converse trap:** When told "all compile errors stop the build," never deduce that "the build stopped, so there was a compile error" (the user could have pressed cancel).
- **"Some" is not "Some not":** In standard logic, "Some students passed" does not prove that "Some students failed". All students might have passed.
- **Left of vs Immediately Left of:** "Alice sits to the left of Bob" means $\text{pos}(A) < \text{pos}(B)$ with any number of seats between them; "immediately left" means $\text{pos}(A) = \text{pos}(B) - 1$.
- **Facing direction inversion:** When people at a circular table face *away* from the center, right and left directions reverse compared to facing inward.
- **Opposite corners in directions:** If a person travels East then North, the return heading to start is South-West, not North-East.
- **Alphabet wrapping:** Under modular shifts, shifting backwards from $A$ yields $Z$ ($1 - 1 = 0 \equiv 26 \pmod{26}$).

## Connections

- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — formal truth tables, logical equivalences, and resolution refutation.
- [First-order logic](../01-discrete-mathematics/first-order-logic.md) — universal ($\forall$) and existential ($\exists$) quantification in predicate calculus.
- [Verbal aptitude](verbal-aptitude.md) — critical reasoning, reading comprehension, and argument analysis.
- [Quantitative aptitude](quantitative-aptitude.md) — algebraic formulation of word problems and rate relations.
- [Spatial aptitude](spatial-aptitude.md) — visual analogies, 2D/3D rotations, and symmetry.

## Practice

**Q1 (MCQ).** Consider the statement: "If a connected graph $G$ is bipartite, then $G$ contains no cycle of odd length."  
Suppose a connected graph $H$ contains a simple cycle of length 7. Which one of the following statements is a logically valid deduction?  
(A) $H$ is bipartite  
(B) $H$ is not bipartite  
(C) $H$ has an even number of vertices  
(D) No definite conclusion can be drawn about bipartiteness  

<details><summary>Answer</summary>

**Answer:** B ($H$ is not bipartite)  
**Solution:**  
Let $P$: "$G$ is bipartite" and $Q$: "$G$ contains no odd cycle".  
The given statement is $P \to Q$.  
Graph $H$ contains a cycle of length 7, which is an odd cycle. Thus $Q$ is FALSE ($\neg Q$).  
By Modus Tollens (the contrapositive law):  
$$(P \to Q) \land \neg Q \implies \neg P$$  
Therefore, $H$ is **not bipartite**.

</details>

**Q2 (MCQ).** Given the statements:
1. Some software developers are musicians.
2. All musicians are creative thinkers.

Consider the following conclusions:
- **I.** Some software developers are creative thinkers.
- **II.** All creative thinkers are musicians.

Which of the conclusions logically follow(s)?  
(A) Only I  
(B) Only II  
(C) Both I and II  
(D) Neither I nor II  

<details><summary>Answer</summary>

**Answer:** A (Only I)  
**Solution:**  
Let $D$ = developers, $M$ = musicians, $C$ = creative thinkers.  
- Premise 1: $D \cap M \ne \emptyset$.  
- Premise 2: $M \subseteq C$.  
Any element in $D \cap M$ belongs to $M$, and because $M \subseteq C$, that element also belongs to $C$. Thus $D \cap C \ne \emptyset$, which guarantees "Some developers are creative thinkers." Conclusion I holds.  
Conclusion II claims $C \subseteq M$, which is the converse of Premise 2. $M \subseteq C$ does not imply $C \subseteq M$. A counterexample where non-musicians are creative thinkers satisfies all premises. Thus II does not follow.

</details>

**Q3 (NAT).** What is the next number in the sequence: $2, 9, 28, 65, 126, \dots$?

<details><summary>Answer</summary>

**Answer:** 217  
**Solution:**  
Analyze the terms in relation to perfect cubes ($n^3$):  
- For $n = 1$: $1^3 + 1 = 1 + 1 = 2$  
- For $n = 2$: $2^3 + 1 = 8 + 1 = 9$  
- For $n = 3$: $3^3 + 1 = 27 + 1 = 28$  
- For $n = 4$: $4^3 + 1 = 64 + 1 = 65$  
- For $n = 5$: $5^3 + 1 = 125 + 1 = 126$  
The general term is $T_n = n^3 + 1$.  
For $n = 6$:  
$$T_6 = 6^3 + 1 = 216 + 1 = 217$$

</details>

**Q4 (MCQ).** In a secret code, the letters at odd positions are shifted forward by 2 in the alphabetical sequence, while letters at even positions are shifted backward by 2. Under this encryption rule, the word `CODE` is encoded as `EMFC`. What is the encoded form of the word `TEST`?  
(A) `VCUR`  
(B) `UDRS`  
(C) `VDTR`  
(D) `VBTQ`  

<details><summary>Answer</summary>

**Answer:** A (`VCUR`)  
**Solution:**  
Apply the alternating rule to each letter of `TEST`:  
- 1st letter `T` (rank 20, odd position): shift $+2 \to 22$ = `V`  
- 2nd letter `E` (rank 5, even position): shift $-2 \to 3$ = `C`  
- 3rd letter `S` (rank 19, odd position): shift $+2 \to 21$ = `U`  
- 4th letter `T` (rank 20, even position): shift $-2 \to 18$ = `R`  
Result: **`VCUR`**.

</details>

**Q5 (NAT).** An autonomous drone departs from base $O(0, 0)$ and flies 30 m East, then turns left (North) and flies 40 m, then turns left (West) and flies 60 m, and finally turns left (South) and flies 80 m. What is the shortest straight-line distance (in meters) of the drone from base $O$?

<details><summary>Answer</summary>

**Answer:** 50  
**Solution:**  
Track displacements along $x$ (East/West) and $y$ (North/South):  
1. 30 m East: $\Delta x_1 = +30, \Delta y_1 = 0$  
2. Turn left $\to$ North 40 m: $\Delta x_2 = 0, \Delta y_2 = +40$  
3. Turn left $\to$ West 60 m: $\Delta x_3 = -60, \Delta y_3 = 0$  
4. Turn left $\to$ South 80 m: $\Delta x_4 = 0, \Delta y_4 = -80$  
Net coordinates:  
$$X = 30 - 60 = -30\text{ m}$$  
$$Y = 40 - 80 = -40\text{ m}$$  
Shortest distance from origin by Pythagorean theorem:  
$$d = \sqrt{X^2 + Y^2} = \sqrt{(-30)^2 + (-40)^2} = \sqrt{900 + 1600} = \sqrt{2500} = 50\text{ m}$$

</details>

**Q6 (MSQ).** A 4-digit code $d_1 d_2 d_3 d_4$ consisting of four distinct digits from $\{0, 1, \dots, 9\}$ satisfies the following four conditions:
1. $d_1$ is a prime number.
2. $d_2 = 3 \times d_1$.
3. $d_3 = d_2 - d_1 + 1$.
4. $d_4 = \frac{d_1 + d_3}{2}$.

Which of the following statements is/are TRUE?  
(A) The 4-digit code is 3975.  
(B) The sum of all four digits is 24.  
(C) $d_4$ is a composite number.  
(D) The 4-digit number is divisible by 5.  

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:**  
Since $d_2 = 3 d_1$ must be a single digit, $3 d_1 \le 9 \implies d_1 \le 3$.  
Primes $\le 3$ are $\{2, 3\}$.  
- If $d_1 = 2$: $d_2 = 6$; $d_3 = 6 - 2 + 1 = 5$; $d_4 = \frac{2 + 5}{2} = 3.5$ (not an integer, invalid).  
- If $d_1 = 3$: $d_2 = 3 \times 3 = 9$; $d_3 = 9 - 3 + 1 = 7$; $d_4 = \frac{3 + 7}{2} = 5$.  
All four digits $\{3, 9, 7, 5\}$ are distinct and valid.  
- Code: 3975. **(A is True)**  
- Sum of digits: $3 + 9 + 7 + 5 = 24$. **(B is True)**  
- $d_4 = 5$, which is prime, not composite. **(C is False)**  
- 3975 ends in 5, hence is divisible by 5. **(D is True)**

</details>

**Q7 (MCQ).** Five researchers P, Q, R, S, and T sit around a circular conference table facing inwards towards the center.
- P is sitting between Q and T.
- S is sitting immediately to the left of Q.
- R is sitting immediately to the right of T.

Who is sitting immediately to the right of P?  
(A) Q  
(B) T  
(C) S  
(D) R  

<details><summary>Answer</summary>

**Answer:** B (T)  
**Solution:**  
Represent positions around the circle. Facing the center means:  
- **Right** = Counter-Clockwise (CCW).  
- **Left** = Clockwise (CW).  
P is between Q and T, so P's two neighbors are Q and T.  
If we place P at seat 0, its neighbors are seats $-1$ (Right/CCW) and $+1$ (Left/CW).  
- S is immediately left of Q $\implies \text{pos}(S) = \text{pos}(Q) + 1$ (CW).  
- R is immediately right of T $\implies \text{pos}(R) = \text{pos}(T) - 1$ (CCW).  
If Q were to the right of P (seat $-1$), then Left of Q would be seat 0 (where P sits), leaving no room for S.  
Thus Q must be to the left of P (seat $+1$), and T must be to the right of P (seat $-1$).  
Checking the complete clockwise circular arrangement: (P, Q, S, R, T).  
- Left of Q is S. (Satisfied)  
- Right of T is R. (Satisfied)  
- Neighbors of P are T and Q. (Satisfied)  
The person sitting immediately to the right of P is **T**.

</details>
