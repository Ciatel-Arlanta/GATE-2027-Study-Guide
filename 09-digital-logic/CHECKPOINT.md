# Digital Logic: Checkpoint

12 mixed GATE-style questions, easy to hard. Attempt without the notes, then open the solutions. Chapters: [Boolean algebra](boolean-algebra-and-minimization.md) (BA), [Combinational](combinational-circuits.md) (CC), [Sequential](sequential-circuits.md) (SC), [Number representation](number-representation-and-arithmetic.md) (NR).

**Q1 (NAT, BA).** How many distinct self-dual Boolean functions of 3 variables exist?

<details><summary>Answer</summary>

**Answer:** 16.
**Solution:** A self-dual function is determined by the first $2^{n-1}=4$ rows; the other 4 are forced. $2^4=16$.

</details>

**Q2 (MCQ, NR).** The 8-bit 2's complement representation of $-37$ is (a) 11011011 (b) 11011010 (c) 10100101 (d) 11011100.

<details><summary>Answer</summary>

**Answer:** (a).
**Solution:** $37=00100101$, invert $11011010$, add 1: $11011011$.

</details>

**Q3 (NAT, SC).** How many flip-flops are needed in total for a mod-14 binary counter, a mod-14 Johnson counter and a mod-14 ring counter?

<details><summary>Answer</summary>

**Answer:** 25.
**Solution:** Binary $\lceil\log_214\rceil=4$; Johnson $14/2=7$; ring $14$. $4+7+14=25$.

</details>

**Q4 (MCQ, CC).** $F(A,B,C)=\Sigma m(0,3,5,6)$ is implemented with a $4{:}1$ MUX with $A$ (MSB), $B$ as selects. Data inputs $I_0..I_3$ are (a) $C',C,C,C'$ (b) $C,C',C',C$ (c) $0,1,1,0$ (d) $C',C',C,C$.

<details><summary>Answer</summary>

**Answer:** (a).
**Solution:** $AB=00$: rows 0,1 give $(1,0)\Rightarrow C'$. $01$: rows 2,3 give $(0,1)\Rightarrow C$. $10$: rows 4,5 give $(0,1)\Rightarrow C$. $11$: rows 6,7 give $(1,0)\Rightarrow C'$. ($F$ is the 3-input XNOR, even parity.)

</details>

**Q5 (NAT, BA).** For $f(A,B,C)=\Sigma m(0,1,2,5,6,7)$, find the number of prime implicants and the number of essential prime implicants (give PIs $\times 10$ + EPIs).

<details><summary>Answer</summary>

**Answer:** 60 (6 PIs, 0 EPIs).
**Solution:** The PIs are all the adjacent pairs: $(0,1)=A'B'$, $(0,2)=A'C'$, $(1,5)=B'C$, $(2,6)=BC'$, $(5,7)=AC$, $(6,7)=AB$. Every 1-cell is in exactly two PIs, so none is essential (a cyclic K-map). Minimal covers use 3 PIs (two different covers).

</details>

**Q6 (NAT, SC).** Two FFs connected through logic: $t_{cq}=3$ ns, $t_{su}=2$ ns, maximum logic delay 7 ns, clock skew 1 ns (worst case). Maximum clock frequency in MHz (to the nearest integer)?

<details><summary>Answer</summary>

**Answer:** 77.
**Solution:** $T=3+7+2+1=13$ ns; $f=1/13$ ns $=76.9\approx77$ MHz.

</details>

**Q7 (MCQ, NR).** The single-precision number `0xC0600000` equals (a) $-3.5$ (b) $-1.75$ (c) $-7.0$ (d) $-3.75$.

<details><summary>Answer</summary>

**Answer:** (a).
**Solution:** Sign 1; exponent field $10000000=128\Rightarrow2^1$; fraction $11000\ldots$: $1.11_2=1.75$. $-1.75\times2=-3.5$.

</details>

**Q8 (MCQ, CC+NR).** A 4-bit adder-subtractor (XOR on $B$, $C_{in}=M$) computes $0110-1001$ with $M=1$ (operands as 4-bit 2's complement). The result bits, carry out, and the overflow flag are (a) 1101, 0, 1 (b) 1101, 1, 0 (c) 0101, 1, 1 (d) 1101, 0, 0.

<details><summary>Answer</summary>

**Answer:** (a).
**Solution:** $0110 + \overline{1001}+1 = 0110+0110+1$. Bit 0: $0+0+1=1$, carry 0; bit 1: $1+1+0=0$, carry 1; bit 2: $1+1+1=1$, carry 1; bit 3: $0+0+1=1$, carry 0. Sum $1101$, carry out $0$. Carry into MSB $=1$, carry out $=0$, so $V=1\oplus0=1$ (overflow: $6-(-7)=13>7$).

</details>

**Q9 (NAT, SC).** $D_A=A\oplus B$, $D_B=A'$ with initial state $AB=11$. After 100 clock pulses, what is the state as the decimal number $2A+B$?

<details><summary>Answer</summary>

**Answer:** 0.
**Solution:** Sequence $11 \to 00 \to 01 \to 11$: cycle of length 3 starting at $11$. After $k$ pulses the state is entry $(k \bmod 3)$ of the list $(11, 00, 01)$. $100 \bmod 3 = 1$, so the state is $00$, decimal $0$.

</details>

**Q10 (MSQ, SC).** Which statements are true?
(a) A level-triggered JK flip-flop with $J=K=1$ can exhibit race-around.
(b) The Moore machine for a sequence detector never has more states than the Mealy machine.
(c) A Johnson counter with 5 flip-flops has 10 states in its main cycle.
(d) A hold violation can always be fixed by reducing the clock frequency.

<details><summary>Answer</summary>

**Answer:** (a), (c).
**Solution:** (b) false: Moore usually needs more. (d) false: the hold constraint is independent of the period.

</details>

**Q11 (NAT, NR).** In single precision, evaluate $s_1=((2^{25}+1)+1)+1)+1$ (left to right) and $s_2=2^{25}+(1+1+1+1)$. What is $s_2-s_1$?

<details><summary>Answer</summary>

**Answer:** 4.
**Solution:** Near $2^{25}$ the spacing of floats is $2^{25-23}=4$. Each $+1$ is below half a spacing and is rounded away: $s_1=2^{25}$. $1+1+1+1=4$ is exact and $2^{25}+4$ is representable: $s_2=2^{25}+4$. Difference $4$ (numerically verified). Floating-point addition is not associative.

</details>

**Q12 (NAT, CC+BA).** A decoder-based realisation of the full adder uses a 3:8 decoder with active-high outputs. If the carry is realised as an OR of minterm outputs, how many minterm outputs go to the carry OR gate and how many to the sum OR gate (answer the total of both)?

<details><summary>Answer</summary>

**Answer:** 8.
**Solution:** $S=\Sigma m(1,2,4,7)$: 4 outputs; $C=\Sigma m(3,5,6,7)$: 4 outputs. Total $4+4=8$ (output 7 is shared). An alternative using fewer wires is $C=AB+C_{in}(A\oplus B)$ but the decoder approach needs these OR inputs.

</details>

**Scoring.** 11–12 correct (about $\ge80\%$): move on to [Computer organization](../10-computer-organization/README.md). 8–10: revise the weakest chapters. Below 60% (fewer than 8): re-read the chapters against the questions you missed — BA: Q1, Q5; CC: Q4, Q8, Q12; SC: Q3, Q6, Q9, Q10; NR: Q2, Q7, Q8, Q11.
