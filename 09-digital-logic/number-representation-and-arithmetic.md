# Number Representation and Arithmetic

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Number representation; Fixed-point arithmetic; Floating-point arithmetic
> **Prerequisites:** [Combinational circuits](combinational-circuits.md) · **Leads to:** [ALU and control unit](../10-computer-organization/alu-and-control-unit.md), [Instruction sets and addressing](../10-computer-organization/instruction-sets-and-addressing.md)

## Quick glance

- Base conversion: integer part by repeated division, fraction by repeated multiplication; binary $\leftrightarrow$ octal/hex by grouping 3/4 bits from the point.
- **$n$-bit ranges:** unsigned $0..2^n-1$; sign-magnitude and 1's complement $-(2^{n-1}-1)..2^{n-1}-1$ (two zeros); **2's complement $-2^{n-1}..2^{n-1}-1$** (one zero, one extra negative).
- 2's complement negate = invert and add 1; subtract = add the 2's complement. **Overflow** (signed): operands same sign, result different sign, equivalently $C_{in,MSB}\oplus C_{out,MSB}$.
- Sign-extend by copying the MSB; zero-extend for unsigned.
- BCD (8421) digit per nibble; excess-3 = BCD + 3 (self-complementing); Gray code: one bit changes between successive values, $g=b\oplus(b\gg1)$.
- **IEEE 754 single:** 1 sign, 8 exponent (bias 127), 23 fraction; value $=(-1)^s\,(1.f)\,2^{e-127}$. **Double:** 1, 11 (bias 1023), 52.
- Exponent all 0: zero/denormal ($0.f\times2^{-126}$); all 1: infinity ($f=0$) or NaN ($f\ne0$).
- Float add: align exponents (shift smaller), add, normalise, round. Float addition is **not associative**.
- #1 trap: bias vs actual exponent, and the **hidden leading 1**.

## 1. Number systems and base conversion

Position value: a digit $d_i$ in base $b$ contributes $d_i\,b^i$ (negative $i$ after the point).

**Integer part: divide repeatedly by the target base, read remainders bottom-up.**
$45_{10}$ to binary: $45/2=22$ r1, $22/2=11$ r0, $11/2=5$ r1, $5/2=2$ r1, $2/2=1$ r0, $1/2=0$ r1: read upward: $101101_2$.

**Fraction part: multiply by the base repeatedly, read integer parts top-down.**
$0.375$: $0.375\times2=0.75\to0$; $0.75\times2=1.5\to1$; $0.5\times2=1.0\to1$. So $0.011_2$. Hence $45.375_{10} = 101101.011_2$.

**A fraction terminates in binary iff its denominator (lowest terms) is a power of 2.** $0.1_{10}$ never terminates: $0.0\overline{0011}_2$ (verified digits $0.000110011001\ldots$). This is why $0.1+0.2\ne0.3$ in floating point.

**Grouping.** Binary $\leftrightarrow$ octal: 3 bits; binary $\leftrightarrow$ hex: 4 bits, starting from the binary point outward (pad left of the integer part, right of the fractional part).
- $ABC_{16} = 1010\,1011\,1100_2 = 101\,010\,111\,100_2 = 5274_8$.
- $101101.011_2 = 101\,101.011_2 = 55.3_8$; hex: $0010\,1101.0110 = 2D.6_{16}$.

**Any base to decimal:** $(1A.8)_{16} = 16+10+8/16 = 26.5$.

**Base-$r$ digit counting:** an $n$-digit base-$r$ number holds $r^n$ values; to represent $N$ distinct values need $\lceil\log_r N\rceil$ digits. Number of bits for decimal $d$ digits $=\lceil d\log_210\rceil\approx 3.32\,d$.

## 2. Signed integer representations ($n$ bits)

| Scheme | Negation | Range | Zeros |
|---|---|---|---|
| Sign-magnitude | flip sign bit | $-(2^{n-1}-1)\ldots2^{n-1}-1$ | 2 ($+0$, $-0$) |
| 1's complement | flip all bits | $-(2^{n-1}-1)\ldots2^{n-1}-1$ | 2 |
| **2's complement** | flip all bits, add 1 | $-2^{n-1}\ldots2^{n-1}-1$ | 1 |
| Excess-$K$ (biased) | $\text{stored}=x+K$ | $-K\ldots 2^n-1-K$ | 1 |

**Value in 2's complement:** $-b_{n-1}2^{n-1} + \sum_{i<n-1} b_i2^i$. The MSB has negative weight.

**Worked example: $-13$ in 8 bits.** $13 = 00001101$. Sign-magnitude: $10001101$. 1's: $11110010$. 2's: $11110011$. Check: $-128+64+32+16+0+0+2+1 = -13$. ✓.

**Worked example: decode $10110100$ as 2's complement.** $-128+32+16+4 = -76$. As sign-magnitude: $-(0110100_2)=-52$. As 1's: invert to $01001011 = 75$, so $-75$.

**Why 2's complement wins:** addition/subtraction use the same adder for signed and unsigned; one zero; $x + (-x) = 2^n\equiv0$. Two's complement of $x$ is $2^n - x$.

**Special case:** the most negative number $-2^{n-1}$ has no positive counterpart: negating $10000000$ gives $10000000$ again.

### Sign extension
Extending to more bits: **copy the sign bit** (2's complement), e.g. $1011$ ($-5$) to 8 bits $=11111011$ ($-5$). Zero-extend for unsigned. In sign-magnitude, the sign moves to the new MSB and zeros fill between.

### Addition, subtraction and overflow
$A - B = A + \overline B + 1$.

**Overflow rules (2's complement):**
- Adding two positives giving a negative, or two negatives giving a non-negative $\Rightarrow$ overflow.
- Adding operands of opposite sign **never** overflows.
- Hardware: $V = C_{n-1}\oplus C_n$.
- **Unsigned overflow** (carry): $C_n=1$ for addition; for subtraction it is a **borrow** when $A<B$ (carry-out 0 in the adder-subtractor).

**Worked example (8 bits).**
1. $100 + 50$: $01100100 + 00110010 = 10010110$. Carry into MSB $=1$, carry out $=0$, $V=1$: overflow (150 > 127). Pattern reads $-106$.
2. $(-100) + (-50)$: $10011100 + 11001110 = 1\,01101010$; drop carry: $01101010 = 106$. Carry into MSB $=0$, carry out $=1$, $V=1$: overflow ($-150 < -128$).
3. $(-5)+3$: $11111011 + 00000011 = 11111110 = -2$. Carry in $=1$, carry out $=1$, $V=0$.

**1's complement addition** needs the end-around carry: add the carry-out back into the LSB.
Example $6 + (-3)$ in 4 bits: $0110 + 1100 = 1\,0010$; end-around: $0010+1 = 0011 = 3$. ✓.

### Flags
Z (zero), N (negative = MSB), C (carry), V (overflow). Signed comparison after $A-B$: $A<B$ iff $N\oplus V = 1$; unsigned $A<B$ iff borrow (C=0 in subtract-as-add convention).

## 3. BCD, excess-3 and Gray codes

**BCD (8421):** each decimal digit in 4 bits $0000..1001$; $1010..1111$ invalid. $59_{10} = 0101\,1001$. Uses more bits than binary ($4d$ vs $3.32d$).

**Excess-3:** BCD digit $+3$: $0\to0011$, $4\to0111$, $9\to1100$. **Self-complementing:** the 9's complement of a digit is the bitwise complement ($2=0101$, $7=1010$). Used in decimal subtraction.

**Gray code:** successive values differ in exactly one bit (also cyclically). Binary to Gray: $G = B\oplus(B\gg1)$. Gray to binary: running XOR from the MSB.

| Dec | Binary | Gray |
|---|---|---|
| 0 | 000 | 000 |
| 1 | 001 | 001 |
| 2 | 010 | 011 |
| 3 | 011 | 010 |
| 4 | 100 | 110 |
| 5 | 101 | 111 |
| 6 | 110 | 101 |
| 7 | 111 | 100 |

Applications: K-map axes, rotary encoders (no glitch between adjacent positions), counters with single-bit transitions.

**Other codes:** 2-out-of-5, parity bit (even: total number of 1s even), Hamming codes (error correction) appear occasionally.

**Decimal addition in BCD:** add as binary; if the digit sum $>9$ or a carry out occurs, add $0110$ (see the BCD adder in [Combinational circuits](combinational-circuits.md)). Example: $27+35$: $0010\,0111 + 0011\,0101$: low digit $0111+0101 = 1100>9\to+0110 = 1\,0010$: digit 2, carry 1; high digit $0010+0011+1 = 0110$: $62$. ✓

## 4. Fixed-point representation and arithmetic

A fixed-point number is an integer scaled by a constant power of the base; the binary point is implicit at a fixed position.

**Qm.n format:** $m$ integer bits (including sign in the signed form, conventions vary: state it), $n$ fraction bits. Stored integer $X$ represents $X\cdot2^{-n}$.

- **Resolution** $2^{-n}$; unsigned range $0\ldots2^m - 2^{-n}$ (with $m$ integer bits).
- Signed 2's complement with $m$ integer bits (counting sign) and $n$ fraction bits: range $-2^{m-1}\ldots 2^{m-1}-2^{-n}$.

**Worked example: 3.14 in unsigned Q4.4.** $3.14\times16 = 50.24\to50 = 0011\,0010$, i.e. $0011.0010_2 = 3.125$. Error $0.015$ (less than resolution $0.0625$; truncating gives error $< 2^{-4}$).

**Worked example: add and multiply.**
$A = 2.5 = 0010.1000$ (raw $40$), $B = 1.25 = 0001.0100$ (raw $20$).
- **Add/subtract:** raw integers add directly (same scale): $40+20 = 60 = 3.75$ ✓.
- **Multiply:** raw product $40\times20 = 800$ has scale $2^{-8}$: $800/256 = 3.125 = 2.5\times1.25$ ✓. A Qm.n times Qm.n gives Q(2m).(2n); to return to Q.n shift right by $n$ (here $800\gg4 = 50 = 3.125$ in raw Q4.4), with possible overflow and truncation.
- **Divide:** pre-shift the dividend left by $n$.

**Overflow** is checked like integers; **saturation** (clamp to max/min) is an alternative to wrapping.

**Fixed vs floating:** fixed point has uniform absolute precision, simple hardware, limited dynamic range; floating point has relative precision, huge range, complex hardware.

**Worked example (range).** Signed 16-bit Q1.15 ($1$ sign, $15$ fraction): range $-1\ldots1-2^{-15}$, resolution $2^{-15}=3.05\times10^{-5}$. Number of distinct values $=2^{16}$.

## 5. IEEE 754 floating point

### Format

| Format | Sign | Exponent | Fraction | Bias | Total |
|---|---|---|---|---|---|
| Single (binary32) | 1 | 8 | 23 | 127 | 32 |
| Double (binary64) | 1 | 11 | 52 | 1023 | 64 |

Stored fields: $s$, $e$ (unsigned exponent field), $f$ (fraction).

| Exponent field $e$ | Fraction $f$ | Meaning |
|---|---|---|
| all 0 | $0$ | $\pm0$ |
| all 0 | $\ne0$ | **denormal:** $(-1)^s\,(0.f)\,2^{1-\text{bias}}$ |
| $1\ldots e_{max}-1$ | any | **normal:** $(-1)^s\,(1.f)\,2^{e-\text{bias}}$ |
| all 1 | $0$ | $\pm\infty$ |
| all 1 | $\ne0$ | NaN |

Single: normal exponents $-126\ldots+127$; denormal exponent $-126$ (not $-127$!). Double: $-1022\ldots1023$.

**The leading 1 of a normal number is implicit (hidden bit), so single precision has 24 significant bits and double 53.**

### Encode: $-13.625$ to single
1. Sign $=1$. $13.625 = 1101.101_2$.
2. Normalise: $1.101101\times2^3$.
3. Exponent field $=3+127=130 = 10000010_2$.
4. Fraction $=101101$ followed by zeros (23 bits) $=10110100000000000000000$.
5. Result: `1 10000010 10110100000000000000000` $=$ `0xC15A0000` (python-checked).

### Decode: `0xC1590000`
`1100 0001 0101 1001 ...`: sign $=1$; exponent bits $10000010 = 130\to130-127=3$; fraction $=1011001$. Value $=-1.1011001_2\times2^3 = -1101.1001_2 = -(8+4+1+0.5+0.0625) = -13.5625$. ✓

### More encodings (checked)

| Value | Hex | Fields |
|---|---|---|
| $1.0$ | 3F800000 | 0 01111111 000...0 |
| $0.75$ | 3F400000 | $1.1\times2^{-1}$: 0 01111110 1000... |
| $5.0$ | 40A00000 | $1.01\times2^2$: 0 10000001 0100... |
| $-0.15625$ | BE200000 | $-1.01\times2^{-3}$: 1 01111100 0100... |
| $0.1$ | 3DCCCCCD | rounded: fraction $10011001100110011001101$ |

### Range, precision and special values (single)
- Smallest positive normal $=2^{-126}\approx1.18\times10^{-38}$.
- Smallest positive denormal $=2^{-126}\cdot2^{-23}=2^{-149}\approx1.4\times10^{-45}$.
- Largest finite $=(2-2^{-23})\times2^{127}\approx3.4028\times10^{38}$ (exponent field $254$, fraction all 1).
- **Machine epsilon** (gap between 1 and the next float) $=2^{-23}\approx1.19\times10^{-7}$; about **7 decimal digits**. Double: $2^{-52}$, about 16 digits; max $\approx1.7977\times10^{308}$; min normal $2^{-1022}$; min denormal $2^{-1074}$.
- **ULP (unit in the last place)** of a number in $[2^k,2^{k+1})$ is $2^{k-23}$ (single): the gap doubles at each power of two, so **relative precision is roughly constant, absolute precision shrinks with magnitude**.
- Number of floats in each binade $[2^k,2^{k+1})$: $2^{23}$ (single).
- Denormals fill the gap around 0 evenly (spacing $2^{-149}$), giving gradual underflow.
- Comparison: for positive normal floats, **integer comparison of the bit patterns gives the same order** as the values (the reason for the bias).

**Worked example: custom format.** A 16-bit format with 1 sign, 5 exponent (bias 15), 10 fraction bits (IEEE half). Largest finite: exponent field $30\to2^{15}$; $(2-2^{-10})\cdot2^{15} = 65504$. Smallest normal: $2^{-14}$. Smallest denormal: $2^{-14}\cdot2^{-10}=2^{-24}$. Machine epsilon $2^{-10}$. Bias $=2^{k-1}-1=15$ for $k=5$ exponent bits.

### Floating-point addition algorithm
1. Compare exponents; **shift the mantissa of the smaller number right** by the difference (keep guard, round, sticky bits).
2. Add (or subtract) the aligned significands (handle signs).
3. **Normalise** the result: if overflow past the point ($\ge2$), shift right and increment exponent; if leading zeros, shift left and decrement exponent.
4. **Round** (default round to nearest, ties to even); renormalise if rounding overflowed the significand.
5. Check overflow/underflow of the exponent.

**Worked example: $1.5 + 0.15625$ (single).**
- $1.5 = 1.1_2\times2^0$; $0.15625 = 1.01_2\times2^{-3}$.
- Exponent difference $=3$; shift the smaller right: $1.01\to0.00101$.
- Add: $1.10000 + 0.00101 = 1.10101$.
- Already normalised: $1.10101_2 = 1.65625$ ✓. Fields: exponent 127, fraction $10101\,0\ldots$ $=$ `0x3FD40000` (checked).

**Worked example with subtraction/normalisation.** $1.101_2\times2^2 - 1.100_2\times2^2 = 0.001_2\times2^2$; normalise: shift left by 3: $1.000\times2^{-1}$; exponent decreases by 3. This is **cancellation**: low-order bits become zero (exact here, but if operands carried rounding errors these errors dominate).

**Rounding modes (IEEE):** round to nearest, ties to even (default); toward $0$; toward $+\infty$; toward $-\infty$. With guard $G$, round $R$, sticky $S$: round up if $G=1$ and ($R$ or $S$ is 1), or if $GRS=100$ and the last kept bit is 1 (tie to even).

**Worked example: rounding.** Keep 3 fraction bits. Extended value $1.010\,100$ (kept $010$, dropped $G R S=100$): exactly half way, the kept LSB is 0 (even), so round down: $1.010$. Extended value $1.011\,100$: tie again, but the kept LSB is 1, so round up to $1.100$. Extended value $1.010\,101$ ($GRS=101$): above half, round up to $1.011$.

### Why floating-point addition is not associative
Finite precision: results are rounded after each operation. In single precision with $a=2^{24}$, $b=1$, $c=-2^{24}$:
- $(a+b)+c$: $2^{24}+1$ rounds to $2^{24}$ (spacing is 2 at $2^{24}$; tie to even), then $-2^{24}$ gives $0$.
- $a+(b+c)$: $1-2^{24}=-16777215$ is exactly representable, then $+2^{24}$ gives $1$.
Both verified numerically ($0$ vs $1$). Similarly, a very large + small number loses the small one (**absorption**), and $0.1+0.2\ne0.3$. **Floating-point addition is commutative but not associative nor distributive; summing small to large order is more accurate.** Multiplication also isn't associative.

### Floating-point operations
- **Multiplication:** multiply significands, add exponents (subtract one bias), normalise, round, XOR signs.
- **Division:** divide significands, subtract exponents (add the bias back).
- **Conversion int to float** loses precision beyond 24 significant bits: $2^{24}+1$ is not representable in single precision. A 32-bit `int` to `float` can therefore lose precision, `int` to `double` cannot (53 bits).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| 2's complement range | $-2^{n-1}\ldots2^{n-1}-1$ | range questions |
| SM / 1's range | $\pm(2^{n-1}-1)$, two zeros | |
| Negation | invert + 1 | |
| Overflow | $C_{n-1}\oplus C_n$ | signed add |
| Gray | $G=B\oplus(B\gg1)$ | |
| Excess-3 | BCD $+3$, self-complementing | |
| Single | 1+8+23, bias 127 | IEEE |
| Double | 1+11+52, bias 1023 | IEEE |
| Normal value | $(-1)^s1.f\cdot2^{e-bias}$ | |
| Denormal | $(-1)^s0.f\cdot2^{1-bias}$ | |
| Single max | $(2-2^{-23})2^{127}\approx3.4\times10^{38}$ | |
| Single min normal / denormal | $2^{-126}$ / $2^{-149}$ | |
| Machine epsilon | $2^{-23}$ (single), $2^{-52}$ (double) | |
| Fixed-point resolution | $2^{-n}$ for $n$ fraction bits | |
| Bias for $k$ exp bits | $2^{k-1}-1$ | |

## GATE traps

- Denormal exponent is $1-\text{bias}$ ($-126$), **not** $0-\text{bias}$.
- The **hidden 1** is not stored; mantissa precision is 24 (single), 53 (double). Forgetting it shifts every answer.
- 2's complement has **one extra negative** number; negating the most negative number overflows.
- 1's complement and sign-magnitude have **two zeros** (so the count of distinct values is $2^n-1$).
- Overflow in 2's complement is about **sign mismatch**, not carry-out. $C_n=1$ with $C_{n-1}=1$ is fine.
- Decimal $\to$ binary fraction: $0.1$ is recurring. Don't assume exact storage.
- Largest single is exponent field **254** (255 is inf/NaN), fraction all 1s.
- Conversions between formats: converting `float` to `int` truncates toward zero, not rounding.
- Float addition is **not associative**; compilers cannot reorder it without permission.
- Gray code is not weighted; excess-3 is not weighted by place values $8421$.
- When asked "how many numbers lie between $2^k$ and $2^{k+1}$", the answer is $2^{23}$ (single) per binade; they are **not** evenly spaced across binades.

## Connections

- [Combinational circuits](combinational-circuits.md) — the adder-subtractor, overflow detection, BCD adder and Gray converters implement this chapter in hardware.
- [Sequential circuits](sequential-circuits.md) — shift registers multiply/divide by powers of 2 and rotate; Gray-code counters.
- [Instruction sets and addressing](../10-computer-organization/instruction-sets-and-addressing.md) — word sizes, immediate ranges, sign-extended displacements.
- [ALU and control unit](../10-computer-organization/alu-and-control-unit.md) — ALU flags and overflow, multiplier/divider and FP units.
- [C basics and expressions](../05-c-programming/c-basics-and-expressions.md) — integer overflow, `unsigned` wrap-around, implicit conversions, `float` rounding.
- [Linear systems and LU](../03-linear-algebra/linear-systems-and-lu.md) — numerical stability and rounding error in elimination.

## Practice

**Q1 (NAT).** What is the range of integers in 8-bit 1's complement? Give the number of distinct values representable.

<details><summary>Answer</summary>

**Answer:** 255 values ($-127\ldots127$).
**Solution:** Two zeros ($00000000$, $11111111$) share one value; $256-1=255$.

</details>

**Q2 (MCQ).** The 2's complement 8-bit representation of $-37$ is: (a) 11011011 (b) 11011010 (c) 10100101 (d) 11011100

<details><summary>Answer</summary>

**Answer:** (a).
**Solution:** $37=00100101$; invert: $11011010$; $+1$: $11011011$. Check: $-128+64+16+8+2+1 = -37$.

</details>

**Q3 (NAT).** Two 8-bit 2's complement numbers $01101010$ and $01011100$ are added. Result in decimal as read, and does overflow occur (1/0)? Give the overflow bit.

<details><summary>Answer</summary>

**Answer:** overflow bit $=1$ (result reads $-58$).
**Solution:** $106+92 = 198>127$. Sum $=11000110 = -128+64+4+2 = -58$. Both operands positive, result negative: overflow.

</details>

**Q4 (NAT).** Decode the single-precision pattern `0x42280000`.

<details><summary>Answer</summary>

**Answer:** 42.
**Solution:** `0100 0010 0010 1000 ...`: sign 0; exponent bits $10000100 = 132$, so $2^5$; fraction $0101000\ldots$: $1.0101_2 = 1.3125$. $1.3125\times32 = 42$. (Matches $101010_2$.)

</details>

**Q5 (MSQ).** For IEEE 754 single precision which are true?
(a) The smallest positive denormal is $2^{-149}$.
(b) Exponent field 255 with fraction 0 represents infinity.
(c) The gap between consecutive floats is the same for all magnitudes.
(d) The largest finite value has exponent field 254.

<details><summary>Answer</summary>

**Answer:** (a), (b), (d).
**Solution:** (c) false: gap in $[2^k,2^{k+1})$ is $2^{k-23}$.

</details>

**Q6 (NAT).** Find the sum of the single-precision numbers $1.5$ and $0.15625$ as a decimal.

<details><summary>Answer</summary>

**Answer:** 1.65625.
**Solution:** Align $0.15625=1.01\times2^{-3}$ to exponent 0: $0.00101$. Add $1.1+0.00101 = 1.10101_2 = 1+0.5+0.125+0.03125 = 1.65625$. Exactly representable, no rounding.

</details>

**Q7 (NAT).** A floating-point format has 1 sign bit, 6 exponent bits (bias 31) and 9 fraction bits, IEEE-style. Largest finite value as an expression, and its approximate value $\log_2$ of it (give the integer part of $\log_2$).

<details><summary>Answer</summary>

**Answer:** $\lfloor\log_2\rfloor = 31$.
**Solution:** Max exponent field $=62$ (63 is inf/NaN), actual exponent $62-31=31$. Max $=(2-2^{-9})\times2^{31}$, which lies between $2^{31}$ and $2^{32}$, so integer part of $\log_2$ is 31.

</details>

**Q8 (MCQ).** In single precision, with $a=2^{24}$, $b=1$, $c=-2^{24}$, evaluating $(a+b)+c$ and $a+(b+c)$ gives respectively:
(a) 1, 1 (b) 0, 1 (c) 1, 0 (d) 0, 0

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** $a+b = 16777217$ needs 25 bits of significance, rounds to $2^{24}$ (tie to even); minus $2^{24}$ gives 0. $b+c=-16777215$ is exactly representable (24 bits); $a+(b+c) = 1$. Not associative.

</details>
