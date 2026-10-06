# Combinational Circuits

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Combinational circuit design
> **Prerequisites:** [Boolean algebra and minimization](boolean-algebra-and-minimization.md) · **Leads to:** [Sequential circuits](sequential-circuits.md), [ALU and control unit](../10-computer-organization/alu-and-control-unit.md)

## Quick glance

- A **combinational circuit** has no memory: outputs depend only on the present inputs. No feedback, no clock.
- **Half adder:** $S = A\oplus B$, $C = AB$. **Full adder:** $S = A\oplus B\oplus C_{in}$, $C_{out} = AB + C_{in}(A\oplus B)$ (majority of the three inputs).
- **Ripple-carry adder:** delay grows linearly, about $2$ gate delays per bit for the carry. **Carry-lookahead:** $P_i = A_i\oplus B_i$, $G_i = A_iB_i$, $C_{i+1} = G_i + P_iC_i$; carries computed in parallel, constant depth.
- **$2^n{:}1$ MUX** has $n$ select lines and is a universal block: any function of $n$ variables is a MUX with data inputs tied to $0/1$; any function of $n+1$ variables needs data inputs from $\{0,1,X,X'\}$.
- **$n$-to-$2^n$ decoder** generates all minterms; any function = decoder + OR of its minterms (or NAND of the zero-minterm outputs when outputs are active-low).
- A $2^n{:}1$ MUX from $2{:}1$ MUXes needs **$2^n-1$** of them. A $4{:}16$ decoder from $2{:}4$ decoders needs **5**.
- Adder/subtractor: XOR each $B_i$ with mode $M$, set $C_{in}=M$. **Signed overflow = $C_{in,MSB}\oplus C_{out,MSB}$.**
- **Static-1 hazard** in an SOP network is removed by adding the consensus (redundant) prime implicant that bridges the adjacent groups.
- #1 trap: carry delay vs sum delay and the **gate-delay convention** (state what a gate delay is, count XOR as one gate unless told otherwise).

## 1. Adders and subtractors

### Half adder (HA)
Adds two bits. Truth table:

| A | B | S | C |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

$S = A\oplus B$, $C = AB$. Cost: **1 XOR + 1 AND**. As NAND-only: 5 NAND gates.

### Full adder (FA)
Adds $A$, $B$ and an incoming carry $C_{in}$.

| A | B | Cin | S | Cout |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 |

$S = \Sigma m(1,2,4,7) = A\oplus B\oplus C_{in}$ (1 when an odd number of inputs are 1).
$C_{out} = \Sigma m(3,5,6,7) = AB + AC_{in} + BC_{in} = AB + C_{in}(A\oplus B)$.

**Two half adders + one OR make a full adder:** HA1 adds $A,B$ giving $S_1, C_1$; HA2 adds $S_1, C_{in}$ giving $S, C_2$; $C_{out} = C_1 + C_2$. So an FA = **2 XOR + 2 AND + 1 OR = 5 gates**.

**Worked example: add $A=1011$ and $B=0110$ with $C_{in}=0$.**

| bit | A | B | Cin | S | Cout |
|---|---|---|---|---|---|
| 0 | 1 | 0 | 0 | 1 | 0 |
| 1 | 1 | 1 | 0 | 0 | 1 |
| 2 | 0 | 1 | 1 | 0 | 1 |
| 3 | 1 | 0 | 1 | 0 | 1 |

Result: carry out $1$, sum $0001$, i.e. $11 + 6 = 17 = 1\,0001_2$. Correct.

### Half and full subtractor
Half subtractor ($A-B$): $D = A\oplus B$, Borrow $= A'B$.
Full subtractor ($A-B-B_{in}$): $D = A\oplus B\oplus B_{in}$, $B_{out} = A'B + (A\odot B)B_{in}$ (equivalently $A'B + A'B_{in} + BB_{in}$).

Note: the difference output equals the adder's sum output; only the borrow differs from the carry (complement $A$ inside the carry logic).

### Adder-subtractor
Subtraction by 2's complement: $A - B = A + \overline{B} + 1$. Put an XOR gate on each $B_i$ with control $M$ ($M=0$ add, $M=1$ subtract) and connect $C_{in} = M$.

```text
        B3  B2  B1  B0
         |   |   |   |
M ---+--XOR XOR XOR XOR
     |   |   |   |   |
     |  FA3 FA2 FA1 FA0 <-- Cin = M
     v
```

**Overflow (signed):** $V = C_{n-1}\oplus C_n$ (carry into MSB XOR carry out of MSB). Equivalent: two same-sign operands give a different-sign result. Unsigned overflow/borrow: just the carry-out (for subtraction, unsigned $A<B$ gives carry-out $=0$ because of the complemented addition).

## 2. Ripple-carry adder: delay

Chain $n$ full adders, carry of FA$_i$ feeds FA$_{i+1}$. The carry has to ripple through all stages.

**Delay model (state it in the answer).** Let one gate (AND, OR or XOR) = 1 unit, FA carry path: AND then OR = **2 units** per stage. Then:

- $P_i, G_i$ (XOR, AND of inputs) ready at $t=1$.
- $C_1 = G_0 + P_0C_0$ at $t=3$; $C_{i+1}$ at $t = 3 + 2i$; so $C_n$ at $t = 2n+1$.
- $S_i = P_i\oplus C_i$ ready at $t = (1+2i)+1 = 2i+2$; the last sum $S_{n-1}$ at $t = 2n$.

Many textbooks simply say "carry propagates through $2$ gates per stage so total $\approx 2n$ gate delays". If the question gives a delay per full adder ($t_{FA}$), the answer is $n\,t_{FA}$ for $n$ bits (carry through the full chain).

**Worked example.** 16-bit ripple adder, each FA carry delay 10 ns (given), sum delay 12 ns after carry arrives at that stage. Critical path: carry through stages 0..14 into stage 15, then sum: $15\times 10 + 12 = 162$ ns. (If instead the question says "worst-case delay of the 16-bit adder = $16\times 10$ ns", it counts carry-out.)

**The cure is to compute carries without waiting:** carry-lookahead.

## 3. Carry-lookahead adder (CLA)

Define for each bit
- **Generate** $G_i = A_iB_i$ (this stage creates a carry by itself),
- **Propagate** $P_i = A_i\oplus B_i$ (or $A_i+B_i$; both work for carry; use XOR when the same $P_i$ feeds the sum): this stage passes an incoming carry along.

Then $C_{i+1} = G_i + P_iC_i$, and $S_i = P_i\oplus C_i$. Unrolling:

$$\begin{aligned}
C_1 &= G_0 + P_0C_0\\
C_2 &= G_1 + P_1G_0 + P_1P_0C_0\\
C_3 &= G_2 + P_2G_1 + P_2P_1G_0 + P_2P_1P_0C_0\\
C_4 &= G_3 + P_3G_2 + P_3P_2G_1 + P_3P_2P_1G_0 + P_3P_2P_1P_0C_0
\end{aligned}$$

Each carry is a two-level AND-OR of $P$, $G$, $C_0$, so all carries appear after the same delay.

**Delay (unlimited fan-in):** $P,G$: 1 unit; carries: +2 units (AND level, OR level) $\Rightarrow$ $t=3$; sums: +1 $\Rightarrow$ **4 units, independent of $n$** for a single-level CLA. In practice fan-in limits force blocks.

**Worked example: compute carries.** $A = 1101$ ($A_3A_2A_1A_0$), $B = 0111$, $C_0 = 0$ (bits listed MSB first; $A_0=1,B_0=1$).

| i | A | B | G | P |
|---|---|---|---|---|
| 0 | 1 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 2 | 1 | 1 | 1 | 0 |
| 3 | 1 | 0 | 0 | 1 |

$C_1 = G_0 + P_0C_0 = 1$. $C_2 = G_1 + P_1G_0 + P_1P_0C_0 = 0 + 1 + 0 = 1$. $C_3 = G_2 + P_2(\dots) = 1$. $C_4 = G_3 + P_3C_3 = 0 + 1 = 1$.
$S_0 = P_0\oplus C_0 = 0$, $S_1 = 1\oplus1 = 0$, $S_2 = 0\oplus 1 = 1$, $S_3 = 1\oplus1 = 0$. Sum $= 0100$ with carry $1$: $13+7 = 20 = 1\,0100$. Correct.

### Block (group) lookahead
For a 4-bit block define group $P^* = P_3P_2P_1P_0$ and $G^* = G_3 + P_3G_2 + P_3P_2G_1 + P_3P_2P_1G_0$. Then block carry-out $= G^* + P^*C_{in}$. Four-bit CLA blocks can be rippled (cheap, slower) or combined by a second-level lookahead unit (e.g. 16 bits $=$ 4 blocks $+$ one lookahead generator).

| Adder | Delay for $n$ bits | Hardware |
|---|---|---|
| Ripple-carry | $O(n)$ | $n$ FAs (smallest) |
| Carry-lookahead (single level) | $O(1)$ ideal, fan-in grows | large |
| Block CLA, ripple between blocks | $O(n/k)$ blocks | moderate |
| Hierarchical CLA | $O(\log n)$ | larger |
| Carry-select | $O(\sqrt n)$ or $O(\log n)$ | duplicates adders, MUX picks |

**Carry-skip / carry-select (just the idea):** carry-select computes each block for both $C_{in}=0$ and $C_{in}=1$ and a MUX chooses when the real carry arrives.

## 4. Multiplexers (MUX)

A **$2^n{:}1$ MUX** passes one of $2^n$ data inputs $I_0..I_{2^n-1}$ to the output, chosen by $n$ select lines $S$. Output $Y = \sum_i m_i(S)\cdot I_i$. Intuition: a rotary switch.

- $2{:}1$ MUX: $Y = S'I_0 + SI_1$ (3 gates + NOT; or "$2$ AND, $1$ OR, $1$ NOT").
- Building bigger MUXes: $2^n{:}1$ from $2{:}1$: **$2^n-1$** MUXes (a tree). $16{:}1$ from $4{:}1$: $4+1 = 5$. $64{:}1$ from $4{:}1$: $16+4+1 = 21$. $8{:}1$ from $2{:}1$: $7$. Select lines drive the levels: the first select bit goes to the leaf level, the last to the root.
- **Delay** of a tree grows with number of levels.

### Implementing a function with a MUX (GATE favourite)

**Method A: function of $n$ variables with a $2^n{:}1$ MUX.** Connect variables to select lines; tie data input $I_i$ to $f(i)$ (0 or 1).

**Method B: function of $n$ variables with a $2^{n-1}{:}1$ MUX.** Use $n-1$ variables as select; the remaining variable $X$ appears at data inputs. For each select combination the two rows give a pair $(f|_{X=0}, f|_{X=1})$:

| pair | data input |
|---|---|
| (0,0) | $0$ |
| (1,1) | $1$ |
| (0,1) | $X$ |
| (1,0) | $X'$ |

**Worked example 1.** $F(A,B,C) = \Sigma m(1,2,6,7)$ on a $4{:}1$ MUX with $A$ (MSB), $B$ as selects.

| AB | rows | $(f_{C=0}, f_{C=1})$ | input |
|---|---|---|---|
| 00 | m0, m1 | (0,1) | $C$ |
| 01 | m2, m3 | (1,0) | $C'$ |
| 10 | m4, m5 | (0,0) | $0$ |
| 11 | m6, m7 | (1,1) | $1$ |

So $I_0 = C$, $I_1 = C'$, $I_2 = 0$, $I_3 = 1$. Check: $A=1,B=1$ gives 1 for either $C$ (m6, m7 are minterms). Correct.

**Worked example 2.** $F(A,B,C,D) = \Sigma m(1,3,4,9,11,14,15)$ with an $8{:}1$ MUX, selects $A,B,C$, data from $D$.

| ABC | rows | $(f_{D=0}, f_{D=1})$ | input |
|---|---|---|---|
| 000 | 0,1 | (0,1) | $D$ |
| 001 | 2,3 | (0,1) | $D$ |
| 010 | 4,5 | (1,0) | $D'$ |
| 011 | 6,7 | (0,0) | $0$ |
| 100 | 8,9 | (0,1) | $D$ |
| 101 | 10,11 | (0,1) | $D$ |
| 110 | 12,13 | (0,0) | $0$ |
| 111 | 14,15 | (1,1) | $1$ |

Data inputs: $I_0..I_7 = D, D, D', 0, D, D, 0, 1$.

**Worked example 3: fewer selects by choosing which variable goes to data.** Using a $2{:}1$ MUX, $F = A\oplus B$: select $A$, $I_0 = B$, $I_1 = B'$. This uses 1 MUX + 1 NOT.

**Worked example 4: how many MUX for XOR-like functions.** Any function of 2 variables takes at most one $2{:}1$ MUX plus inverter: select $A$, inputs from $\{0,1,B,B'\}$. $F = A+B$: $I_0 = B$, $I_1 = 1$. $F = AB$: $I_0 = 0$, $I_1 = B$.

**Universal property:** a $2{:}1$ MUX plus constants $0$ and $1$ is functionally complete: NOT: $S=A$, $I_0=1$, $I_1=0$; AND: $S=A$, $I_0=0$, $I_1=B$.

## 5. Demultiplexers, decoders and encoders

### Demultiplexer (DEMUX)
Reverse of a MUX: one input $D$ routed to one of $2^n$ outputs chosen by $n$ select lines; others are 0. $Y_i = D\cdot m_i(S)$. **A $1{:}2^n$ DEMUX is the same circuit as an $n$-to-$2^n$ decoder with enable $=D$.**

### Decoder
**$n$-to-$2^n$ decoder:** outputs $Y_i = m_i$ (active-high) of the inputs. A 3-to-8 decoder: 8 three-input AND gates and 3 inverters. Often with an **enable** $E$: $Y_i = E\cdot m_i$.

**Implementing functions:** each function = OR of its minterm outputs. A decoder with $k$ outputs shared by several functions lets one decoder implement a whole set (full adder: $S = \Sigma m(1,2,4,7)$, $C = \Sigma m(3,5,6,7)$ with one 3:8 decoder and two 4-input OR gates). With **active-low outputs** (NAND-based decoder), use a NAND gate to OR the selected outputs.

**Cascading with enables.**
- $3{:}8$ from $2{:}4$ (with enable): 2 decoders; the MSB $A_2$ drives $E$ of one and $\overline{E}$ (via inverter) of the other.
- $4{:}16$ from $2{:}4$: $1 + 4 = 5$ decoders (first level decodes the two MSBs and enables one of four second-level decoders).
- General rule: when $n$ is a multiple of $k$, the number of $k$-input decoders (with enable) needed for an $n$-input decoder is $(2^n-1)/(2^k-1)$ only for $n=2k$ (a two-level tree); e.g. $4{:}16$ from $2{:}4$: $(16-1)/(4-1)=5$; $6{:}64$ from $3{:}8$: $1+8=9=(64-1)/(8-1)$. Check by counting levels directly otherwise.

### Encoder
$2^n$-to-$n$: the inverse; exactly one input is 1, output is its index. For 4-to-2: $Y_1 = I_2 + I_3$, $Y_0 = I_1 + I_3$. Problem: all-zero input and input 0 give the same output $00$ (needs a valid bit); two simultaneous inputs give garbage.

### Priority encoder
Resolves simultaneous inputs by priority (highest index wins), with a valid output $V = I_0+I_1+I_2+I_3$.

| I3 | I2 | I1 | I0 | Y1 | Y0 | V |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | X | X | 0 |
| 0 | 0 | 0 | 1 | 0 | 0 | 1 |
| 0 | 0 | 1 | X | 0 | 1 | 1 |
| 0 | 1 | X | X | 1 | 0 | 1 |
| 1 | X | X | X | 1 | 1 | 1 |

$Y_1 = I_3 + I_2$, $Y_0 = I_3 + I_2'I_1$. Used in interrupt controllers ([I/O and interrupts](../10-computer-organization/io-interrupts-dma.md)).

## 6. Comparators and code converters

### Magnitude comparator
1-bit: $A>B$: $AB'$; $A<B$: $A'B$; $A=B$: $A\odot B$.
$n$-bit equality: $E = \prod_i (A_i\odot B_i)$. $n$-bit $A>B$ (MSB first): $G = G_{n-1} + E_{n-1}G_{n-2} + E_{n-1}E_{n-2}G_{n-3}+\dots$ where $G_i = A_iB_i'$ and $E_i = A_i\odot B_i$. Cascading inputs on the 7485-style chip: lower-order results feed the cascade inputs.

**Worked example.** $A = 1011$, $B = 1001$. Bit 3: $E_3 = 1$. Bit 2: $A_2=0,B_2=0$, $E_2=1$. Bit 1: $A_1=1,B_1=0$, $G_1 = 1$. So $A>B$ ($11>9$).

### Code converters
Binary to Gray: $g_{n-1} = b_{n-1}$, $g_i = b_{i+1}\oplus b_i$ (each Gray bit is XOR of adjacent binary bits). Gray to binary: $b_{n-1} = g_{n-1}$, $b_i = b_{i+1}\oplus g_i$ (a chain of XORs, so a ripple delay of $n-1$ XORs; binary-to-Gray has one XOR level).

**Worked example.** Binary $1011$ to Gray: $g_3 = 1$; $g_2 = 1\oplus0 = 1$; $g_1 = 0\oplus1 = 1$; $g_0 = 1\oplus1 = 0$: Gray $= 1110$. Back: $b_3=1$; $b_2 = 1\oplus1 = 0$; $b_1 = 0\oplus1 = 1$; $b_0 = 1\oplus0 = 1$: $1011$. Correct.

**BCD adder.** Add two BCD digits with a 4-bit binary adder giving $K\,Z_3Z_2Z_1Z_0$ (carry $K$ from the adder). The result is invalid if the sum $>9$; then add $6$ ($0110$) with a second adder and produce a BCD carry:
$$C_{BCD} = K + Z_3Z_2 + Z_3Z_1.$$
Verified: for every binary sum $0..19$, $C_{BCD}=1$ exactly when sum $>9$. Example: $8+7 = 15 = 01111$, $Z_3Z_2=1$ so $C=1$, add $0110$: $1111+0110 = 1\,0101$, BCD $= 1\;5$. Correct. A BCD adder uses 2 four-bit adders and 3 extra gates (two ANDs and an OR).

## 7. ROM, PLA and PAL as combinational logic

- **ROM $2^n\times m$:** fixed AND plane (an $n$-to-$2^n$ decoder) + programmable OR plane; stores a lookup table, so any $m$ functions of $n$ variables.
- **PLA:** both AND and OR planes programmable; a function needs only its product terms. Sizing: programmable points $=2n\cdot p + p\cdot m$ (for $p$ product terms, $m$ outputs).
- **PAL:** programmable AND plane, fixed OR plane.
- A ROM of $2^n$ words ($n$ address bits, $m$ output bits) implements any set of $m$ functions of $n$ variables; capacity $2^n\times m$ bits.

## 8. Gate-count and delay questions

Typical patterns:
1. **"Minimum number of $2{:}1$ MUXes / NAND gates"**: use the formulas above; count NAND gates by NAND-NAND conversion of a minimal SOP (gates = number of product terms with $\ge2$ literals + 1 for the output, plus inverters for complemented inputs if not available).
2. **"Critical path"**: find the longest path in terms of gate delays; shared inputs do not add delay.
3. **Fan-in limits**: a $k$-input AND needs $\lceil\log_2 k\rceil$ levels of 2-input gates.
4. **Multi-bit adder delay** as in sections 2 and 3.

**Worked example.** A 4-input AND built from 2-input ANDs: 3 gates, 2 levels. An 8-input OR: 7 gates, 3 levels.

## 9. Hazards (brief)

A **hazard** is a momentary wrong output (glitch) caused by unequal gate delays, though the steady-state function is right.

- **Static-1 hazard:** output should stay 1 but dips to 0. Occurs in an SOP circuit when a single input changes and the two minterms covering the two adjacent 1s are in different product terms.
- **Static-0 hazard:** the dual, in POS circuits.
- **Dynamic hazard:** output changes more than once on a single intended transition (needs at least three levels of logic).

**Worked example.** $F = AB + A'C$ with $B=C=1$. $F = A + A' = 1$ for both values of $A$. When $A$ goes $1\to0$, the term $AB$ falls before $A'C$ rises (the inverter delay), so $F$ glitches to 0. **Fix:** add the consensus term $BC$: $F = AB + A'C + BC$; $BC=1$ throughout, holding $F=1$. On the K-map the hazard is two adjacent 1-groups not covered by a common group; the cure is a covering group over the boundary.

**Hazard-free design = cover every pair of adjacent 1s by a common product term** (use all prime implicants), which costs extra gates. Hazards matter for asynchronous / level-sensitive logic; synchronous designs avoid them by sampling only after settling.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| HA | $S=A\oplus B$, $C=AB$ | Cost 2 gates |
| FA | $S=A\oplus B\oplus C_{in}$; $C_{out}=AB+C_{in}(A\oplus B)$ | 5 gates (2 HA + OR) |
| Half subtractor | $D=A\oplus B$, $B_{out}=A'B$ | |
| Full subtractor | $D=A\oplus B\oplus B_{in}$, $B_{out}=A'B+(A\odot B)B_{in}$ | |
| CLA | $C_{i+1}=G_i+P_iC_i$; $G=AB$, $P=A\oplus B$ | Fast adders |
| Ripple delay | $\approx 2n$ gate delays ($n\,t_{FA}$ if FA delay given) | Adder timing |
| CLA delay | constant, 4 gate delays (P/G 1, carry 2, sum 1) | Adder timing |
| MUX | $2^n{:}1$ has $n$ selects; tree of $2{:}1$: $2^n-1$ | MUX counting |
| MUX implementation | $n$-var $f$ on $2^{n-1}{:}1$ with data $\in\{0,1,X,X'\}$ | MUX questions |
| Decoder | $Y_i=m_i$; $4{:}16$ from $2{:}4$: 5 | Decoder questions |
| Overflow (2's compl.) | $C_{in,MSB}\oplus C_{out,MSB}$ | Adder-subtractor |
| BCD adder | $C=K+Z_3Z_2+Z_3Z_1$, add $0110$ | Code questions |
| Binary to Gray | $g_i=b_{i+1}\oplus b_i$ | Code converters |
| Static-1 hazard | add consensus PI | Hazards |

## GATE traps

- Confusing the carry of a **subtractor** with that of an adder: borrow $=A'B$, not $AB$.
- Using **$P_i = A_i + B_i$** when the same signal is also used for the sum: sum needs XOR.
- **Ripple-carry delay** is carry-chain delay, not $n\times$ the delay of a whole FA including its sum.
- A MUX can implement an $n$-variable function with **$n-1$ select lines**; students forget data inputs may be $X$ or $X'$.
- **Encoder vs priority encoder:** a plain encoder gives wrong output when two inputs are 1; also cannot distinguish input 0 from "no input" without a valid bit.
- Number of $2{:}1$ MUXes in a $2^n{:}1$ tree is $2^n-1$, **not** $n$.
- Overflow occurs on **signed** arithmetic as $C_{n-1}\oplus C_n$; carry-out alone is the unsigned flag.
- Hazards **cannot** appear in a function with a single input transition when all adjacent 1s are covered by a common term; the minimal SOP is often **not** hazard-free.
- Gray to binary is a **serial** XOR chain; binary to Gray is parallel.

## Connections

- [Boolean algebra and minimization](boolean-algebra-and-minimization.md) — every circuit here is a minimized SOP/POS; K-maps give MUX and decoder implementations.
- [Sequential circuits](sequential-circuits.md) — flip-flops plus combinational logic form counters and FSMs; excitation logic is combinational design.
- [Number representation and arithmetic](number-representation-and-arithmetic.md) — adder-subtractor realises 2's complement arithmetic and overflow detection.
- [ALU and control unit](../10-computer-organization/alu-and-control-unit.md) — ALU = adder + logic unit + MUX; decoders drive hardwired control.
- [Memory hierarchy and cache](../10-computer-organization/memory-hierarchy-and-cache.md) — address decoding for memory chips uses decoders.
- [I/O, interrupts and DMA](../10-computer-organization/io-interrupts-dma.md) — priority encoders resolve interrupt priority.
- [Propositional logic](../01-discrete-mathematics/propositional-logic.md) — the same Boolean laws; functional completeness.

## Practice

**Q1 (MCQ).** The number of half adders required to build one full adder, together with the extra gates needed, is:
(a) 1 HA + 2 gates (b) 2 HA + 1 OR (c) 3 HA (d) 2 HA + 1 AND

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** HA1 adds $A,B$; HA2 adds $S_1$ and $C_{in}$; $C_{out} = C_1 + C_2$ needs one OR.

</details>

**Q2 (NAT).** How many $2{:}1$ multiplexers are needed to build a $32{:}1$ multiplexer?

<details><summary>Answer</summary>

**Answer:** 31.
**Solution:** A tree over $32$ data inputs needs $16+8+4+2+1 = 31$ muxes $= 2^5-1$.

</details>

**Q3 (MCQ).** $F(A,B,C) = \Sigma m(1,2,6,7)$ is implemented with a $4{:}1$ MUX where $A,B$ drive $S_1,S_0$. The data inputs $I_0..I_3$ are:
(a) $C, C', 0, 1$ (b) $C', C, 1, 0$ (c) $0,1,C,C'$ (d) $C, C, 0, 1$

<details><summary>Answer</summary>

**Answer:** (a).
**Solution:** $AB=00$: rows 0,1 give $(0,1)\Rightarrow C$. $AB=01$: rows 2,3 give $(1,0)\Rightarrow C'$. $AB=10$: $(0,0)\Rightarrow 0$. $AB=11$: $(1,1)\Rightarrow1$.

</details>

**Q4 (NAT).** A 4-bit carry-lookahead adder has $A=0110$, $B=1011$, $C_0 = 1$. Find the value of $C_4$ and the sum bits $S_3S_2S_1S_0$ (give $C_4$ as the answer).

<details><summary>Answer</summary>

**Answer:** $C_4 = 1$ (sum $=0010$).
**Solution:** $6+11+1 = 18 = 1\,0010$. By lookahead: $G = (A_iB_i)$: $G_0=0\cdot1=0$, $G_1 = 1\cdot1 = 1$, $G_2 = 1\cdot0 = 0$, $G_3=0$. $P_0 = 1$ ($0\oplus1$), $P_1 = 0$, $P_2 = 1$, $P_3 = 1$. $C_1 = G_0+P_0C_0 = 1$; $C_2 = G_1 + P_1C_1 = 1$; $C_3 = G_2+P_2C_2 = 1$; $C_4 = G_3 + P_3C_3 = 1$. Sums: $S_0 = P_0\oplus C_0 = 1\oplus1 = 0$; $S_1 = P_1\oplus C_1 = 0\oplus1 = 1$; $S_2 = 1\oplus1 = 0$; $S_3 = 1\oplus1 = 0$. So the sum is $S_3S_2S_1S_0 = 0010$. Matches $18 = 10010_2$.

</details>

**Q5 (MSQ).** Which of the following statements about decoders and encoders are true?
(a) A $3{:}8$ decoder with enable can implement a full adder using two OR gates.
(b) A $4{:}16$ decoder can be built from five $2{:}4$ decoders with enable.
(c) An ordinary 4-to-2 encoder gives the correct index when $I_1=I_2=1$.
(d) A priority encoder needs a valid output to distinguish $I_0$ active from no input active.

<details><summary>Answer</summary>

**Answer:** (a), (b), (d).
**Solution:** (a) $S=\Sigma m(1,2,4,7)$, $C=\Sigma m(3,5,6,7)$. (b) $1+4=5$. (c) false: $Y_1Y_0 = 11$ (OR of both) gives index 3. (d) true: $I_0$ gives $00$ and none also gives $00$.

</details>

**Q6 (NAT).** A 16-bit ripple-carry adder uses full adders where carry-in to carry-out delay is 2 ns and sum is available 3 ns after its carry-in arrives. Time (ns) when the most significant sum bit is ready (inputs applied at $t=0$, $C_0$ ready at $t=0$)?

<details><summary>Answer</summary>

**Answer:** 33.
**Solution:** Carry into bit 15 appears after 15 stages: $15\times2 = 30$ ns. Then sum: $+3$. Total $33$ ns.

</details>

**Q7 (MCQ).** For $F = AB + A'C$, which statement is true?
(a) The circuit has a static-0 hazard at $B=C=1$. (b) A static-1 hazard exists when $B=C=1$ and $A$ changes; adding $BC$ removes it. (c) Adding $A'B$ removes it. (d) No hazard can exist in a minimal SOP.

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** The two 1-cells $ABC$ (m7) and $A'BC$ (m3) are adjacent but covered by different terms. The consensus term $BC$ covers both. (d) is false.

</details>

**Q8 (NAT).** A BCD adder adds $9$ and $8$ (BCD). The binary 4-bit sum has $K=1$ and $Z_3Z_2Z_1Z_0 = 0001$ ($17$). What is the final BCD output (as a decimal two-digit number)?

<details><summary>Answer</summary>

**Answer:** 17.
**Solution:** $C_{BCD} = K + \dots = 1$ so add $0110$: $0001 + 0110 = 0111$ (=7), carry digit 1. Output: $1\;7$ = 17. Matches $9+8$.

</details>
