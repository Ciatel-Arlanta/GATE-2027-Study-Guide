# Digital Logic: Cheat sheet

One-page revision. Details: [Boolean algebra](boolean-algebra-and-minimization.md) · [Combinational](combinational-circuits.md) · [Sequential](sequential-circuits.md) · [Number representation](number-representation-and-arithmetic.md).

## Boolean algebra and minimization

| Item | Fact |
|---|---|
| Functions of $n$ variables | $2^{2^n}$; self-dual: $2^{2^{n-1}}$ |
| $m_i$, $M_i$ | $M_i = m_i'$; $A$ is MSB; maxterm uses complemented literal for a 1 bit |
| $\Sigma m$ vs $\Pi M$ | complementary index sets over $0..2^n-1$ |
| Laws | $A+A'B=A+B$; $A+BC=(A+B)(A+C)$; absorption; **consensus** $AB+A'C+BC=AB+A'C$ |
| Dual | swap $+\leftrightarrow\cdot$, $0\leftrightarrow1$ (not complement) |
| De Morgan | $(AB)'=A'+B'$; NAND-NAND = AND-OR; NOR-NOR = OR-AND |
| Complete sets | NAND, NOR, {AND,NOT}, {OR,NOT}; **not** {AND,OR}, {XOR,AND} w/o 1 |
| XOR | odd parity; $A\oplus1=A'$; $A(B\oplus C)=AB\oplus AC$ |
| K-map | Gray order 00,01,11,10; groups $2^k$ cells kill $k$ vars; wrap-around |
| PI / EPI | PI = maximal group; EPI = only cover of some **1** cell (not X) |
| POS | group zeros, flip polarity |
| Q–M | group by # of 1s; combine pairs differing in 1 bit; PI chart; EPIs then Petrick |

## Combinational

| Block | Equations / facts |
|---|---|
| Half adder | $S=A\oplus B$, $C=AB$ |
| Full adder | $S=A\oplus B\oplus C_{in}$, $C_{out}=AB+C_{in}(A\oplus B)$; 2 HA + OR |
| Subtractor | $D=A\oplus B(\oplus B_{in})$, borrow $=A'B$ (+$(A\odot B)B_{in}$) |
| Adder-subtractor | XOR on $B$ with $M$, $C_{in}=M$; overflow $=C_{n-1}\oplus C_n$ |
| Ripple adder | $\approx2n$ gate delays / $n\,t_{carry}$ |
| CLA | $G=AB$, $P=A\oplus B$, $C_{i+1}=G_i+P_iC_i$; delay const (P/G 1, carry 2, sum 1) |
| MUX | $2^n{:}1$, $n$ selects; $2^n-1$ of $2{:}1$; $f$ of $n$ vars on $2^{n-1}{:}1$ with data $\in\{0,1,X,X'\}$ |
| Decoder | outputs = minterms; $4{:}16$ from $2{:}4$ = 5; full adder = 3:8 + 2 ORs |
| Encoder | $2^n\to n$; priority encoder + valid bit |
| Comparator | $A>B$: $AB'$; $=$: XNOR; chain from MSB |
| Binary to Gray | $g_i=b_{i+1}\oplus b_i$ (parallel); Gray to binary: XOR chain |
| BCD adder | $C=K+Z_3Z_2+Z_3Z_1$, add $0110$ |
| Hazard | static-1: fix by consensus PI |

## Sequential

| FF | $Q^+$ | Excitation $Q\to Q^+$ |
|---|---|---|
| SR | $S+R'Q$ ($SR=0$) | $0\to0$: $0X$; $0\to1$: $10$; $1\to0$: $01$; $1\to1$: $X0$ |
| D | $D$ | $D=Q^+$ |
| JK | $JQ'+K'Q$ | $0X$, $1X$, $X1$, $X0$ |
| T | $T\oplus Q$ | $T=Q\oplus Q^+$ |

- Conversions: D from T: $T=D\oplus Q$; JK from T: $T=JQ'+KQ$; T from JK: $J=K=T$; D from JK: $J=D,K=D'$; JK from SR: $S=JQ'$, $R=KQ$.
- Race-around: level JK, $J=K=1$, $t_{pw}>t_{pd}$ → use master-slave/edge.
- Timing: $T\ge t_{cq}+t_{comb}+t_{su}(+skew)$; hold: $t_{cq}+t_{comb,min}\ge t_h$ (not fixed by slower clock).
- Counters: mod-$N$ needs $\lceil\log_2N\rceil$ FFs; ripple delay $n\,t_{pd}$; stage $k$ gives $f/2^k$; sync up counter $T_i=\prod_{j<i}Q_j$.
- Ring: $n$ states; Johnson: $2n$ states; both need self-start care.
- Mealy: output = f(state, input), fewer states; Moore: f(state), glitch-free, Moore output lags one clock.
- Analysis: write $D/J/K/T$ equations → next-state table → cycle.
- LFSR $n$ bits: up to $2^n-1$ states.

## Number representation

| Item | Fact |
|---|---|
| Fraction conversion | multiply by base; terminates iff denominator is power of base |
| Ranges ($n$ bits) | unsigned $0..2^n-1$; SM/1's $\pm(2^{n-1}-1)$, 2 zeros; 2's $-2^{n-1}..2^{n-1}-1$ |
| Negate (2's) | invert + 1; $-x\equiv2^n-x$ |
| Overflow | same-sign operands → different-sign result; $C_{n-1}\oplus C_n$ |
| Sign extension | copy MSB |
| Gray | $B\oplus(B\gg1)$ |
| Excess-3 | BCD+3, self-complementing |
| Qm.n | resolution $2^{-n}$; multiply: product has $2n$ frac bits, shift right $n$ |

**IEEE 754**

| | Single | Double |
|---|---|---|
| Fields | 1/8/23 | 1/11/52 |
| Bias | 127 | 1023 |
| Normal | $(-1)^s(1.f)2^{e-127}$, $e=1..254$ | $e=1..2046$ |
| Denormal | $e=0,f\ne0$: $0.f\cdot2^{-126}$ | $2^{-1022}$ |
| $\infty$ / NaN | $e=255$: $f=0$ / $f\ne0$ | $e=2047$ |
| Max | $(2-2^{-23})2^{127}\approx3.4\times10^{38}$ | $\approx1.8\times10^{308}$ |
| Min normal / denormal | $2^{-126}$ / $2^{-149}$ | $2^{-1022}$ / $2^{-1074}$ |
| Epsilon | $2^{-23}$ (~7 digits) | $2^{-52}$ (~16 digits) |

- Float add: align (shift smaller right), add, normalise, round (nearest-even; guard/round/sticky).
- Not associative: $(2^{24}+1)-2^{24}=0$ but $2^{24}+(1-2^{24})=1$ in single.
- Examples: $1.0$ = 0x3F800000; $-13.625$ = 0xC15A0000; $0.1$ = 0x3DCCCCCD.

## Remember

- Gray-code axes on K-maps. EPI counts from 1-cells only.
- Excitation tables for design, characteristic tables for analysis.
- Hidden 1 and bias in IEEE 754; denormal exponent is $1-\text{bias}$.
