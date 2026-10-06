# ALU, Datapath and Control Unit

> **Paper:** CS · **Priority:** P1 · **Plan topics:** ALU design; Hardwired control unit; Microprogrammed control unit
> **Prerequisites:** [Combinational circuits](../09-digital-logic/combinational-circuits.md), [Sequential circuits](../09-digital-logic/sequential-circuits.md), [Instruction sets and addressing](instruction-sets-and-addressing.md) · **Leads to:** [Pipelining](pipelining.md)

## Quick glance

- The **datapath** (registers, buses, ALU) does the work; the **control unit** generates the control signals that sequence it. Instruction = fixed sequence of **micro-operations**, one or more per clock step.
- **ALU slice** = AND, OR, full adder, a $B$-inverter, and a MUX; subtraction $=A+\overline B+1$ (Binvert = 1, CarryIn = 1). Zero flag = NOR of outputs; overflow = $C_{n-1}\oplus C_n$.
- **Barrel shifter:** $n\log_2 n$ 2:1 MUXes shifts any amount in one step. **Booth's algorithm** handles signed multiplication in $n$ add/subtract-shift steps.
- **Single-bus datapath:** one transfer per bus per step, so $ADD\ R1,R2$ takes 3 fetch steps + 3 execute steps (R1$\to$Y, R2 + Y $\to$ Z, Z$\to$R1). More buses = fewer steps.
- **Hardwired control:** step counter + instruction decoder + combinational logic. Fast, costly to change, used in RISC.
- **Microprogrammed control:** each machine instruction is interpreted by a **microprogram** held in the **control store**. Flexible, slower, used in CISC.
- **Horizontal** microinstructions: one bit per control signal, wide, parallel, no decoding. **Vertical:** encoded fields, narrow, slower (decoders), more steps.
- **Control memory size** = (number of microinstructions) $\times$ (control word width) where width $=$ sum of field widths + next-address field + condition-select field.
- #1 trap: **encoded field width** is $\lceil\log_2(\text{signals}+1)\rceil$ (the +1 is the "no signal" code) when the group may also be inactive.

## 1. ALU design

An **ALU** takes two operands and a function-select code, and produces a result and status flags (Z, N, C, V). Typical functions: add, subtract, AND, OR, XOR, NOT, shift, compare.

### One-bit ALU slice
Components per bit: AND gate, OR gate, full adder, XOR to optionally invert $B$ (**Binvert**), and a MUX selecting which result leaves the slice.

```text
         Binvert
           |
  b ---+--XOR--+
       |       |      +-----+
  a ---+-------+--AND-|     |
       |       |      |     |
       +-------+--OR--| MUX |-- result
       |       |      |     |
       +-------+--FA--|     |--(Operation select)
              CarryIn  +-----+
                  CarryOut
```

**Control lines for a 32-bit ALU:** Binvert (also fed to the LSB CarryIn), Operation (2 bits: AND, OR, ADD, SLT).

| Function | Binvert | CarryIn(LSB) | Operation |
|---|---|---|---|
| AND | 0 | x | AND |
| OR | 0 | x | OR |
| ADD | 0 | 0 | ADD |
| SUB | 1 | 1 | ADD |
| SLT (set on less than) | 1 | 1 | SLT: result bit0 = sign of $A-B$ (adjusted for overflow), others 0 |
| NOR | (invert A and B, AND) | | |

**Subtraction:** $A-B=A+\overline B+1$, the XOR on $B$ and CarryIn$=1$ do this. **Zero flag:** NOR of all result bits. **Overflow** $=C_{in,MSB}\oplus C_{out,MSB}$. **Number of control bits** for $k$ distinct operations: $\lceil\log_2 k\rceil$.

**Worked example: 4-bit ALU with a 3-bit select.** Define $S=000$: $A+B$; $001$: $A-B$; $010$: $A\wedge B$; $011$: $A\vee B$; $100$: $A\oplus B$; $101$: $\overline A$. Inputs $A=0110$, $B=0011$:
- $000$: $1001$; $001$: $0011$; $010$: $0010$; $011$: $0111$; $100$: $0101$; $101$: $1001$.

**ALU speed** is set by the adder: ripple-carry makes a 32-bit ALU slow ($\approx2n$ gate delays), so real ALUs use carry-lookahead. See [Combinational circuits](../09-digital-logic/combinational-circuits.md).

### Shifters
Logical shift fills with 0; arithmetic right shift replicates the sign bit; rotate wraps around. A **barrel shifter** shifts by any amount in a single combinational delay: $\log_2n$ stages, stage $i$ either passes or shifts by $2^i$ using 2:1 MUXes. **Hardware: $n\log_2n$ MUXes** (32-bit: $32\times5=160$).

**Worked example.** Left shift $00001101$ by 3: stage 0 (by 1): $00011010$; stage 1 (by 2): $01101000$ (shifted by 3 total: stage 0 and stage 1 both enabled, amount $11_2$). Result $01101000 = 13\times8=104$.

### Multiplication (shift-add and Booth)
**Unsigned shift-add:** for each multiplier bit, add the multiplicand to the accumulator if the bit is 1, then shift. $n$ cycles.

**Booth's algorithm** (signed 2's complement): registers $A$ (accumulator, 0), $Q$ (multiplier), $Q_{-1}=0$, $M$ (multiplicand). Repeat $n$ times: look at $(Q_0,Q_{-1})$: $10\Rightarrow A\leftarrow A-M$; $01\Rightarrow A\leftarrow A+M$; $00$ or $11\Rightarrow$ nothing. Then **arithmetic shift right** the combined $A,Q,Q_{-1}$. Result in $A\,Q$ ($2n$ bits).

**Worked example: $5\times(-3)$, 4 bits.** $M=0101$, $-M=1011$, $Q=1101$ ($-3$).

| Step | $(Q_0,Q_{-1})$ | Operation | $A$ after op | After shift: $A\;Q\;Q_{-1}$ |
|---|---|---|---|---|
| init | | | 0000 | 0000 1101 0 |
| 1 | 1,0 | $A-M$ | 1011 | 1101 1110 1 |
| 2 | 0,1 | $A+M$ | $1101+0101=0010$ | 0001 0111 0 |
| 3 | 1,0 | $A-M$ | $0001+1011=1100$ | 1110 0011 1 |
| 4 | 1,1 | none | 1110 | 1111 0001 1 |

Result $1111\,0001_2=-15$. ✓ (verified by simulation). Booth is faster for runs of 1s; **worst case it still takes $n$ steps**; **bit-pair (radix-4)** halves the step count to $n/2$.

**Restoring division** takes $n$ iterations of shift, subtract, restore if negative; non-restoring avoids the restore step.

## 2. Datapath and micro-operations

A **micro-operation** is a register-transfer such as $MAR\leftarrow PC$ or $Z\leftarrow Y+\text{bus}$. Control signals gate registers onto buses (`Rout`) and load them from the bus (`Rin`).

### Single-bus datapath
All registers, the ALU input, MDR, MAR, PC, IR connect to one bus. ALU has inputs: bus and register $Y$; output goes to register $Z$. **Only one value can be on the bus per step**, so each ALU operation needs: put operand 1 into $Y$, put operand 2 on bus and add into $Z$, move $Z$ to the destination.

**Fetch (3 steps):**
1. $PC_{out}$, $MAR_{in}$, Read, Select4 (adds 4 to PC via the ALU), Add, $Z_{in}$
2. $Z_{out}$, $PC_{in}$, $Y_{in}$ (or WMFC: wait for memory function complete)
3. $MDR_{out}$, $IR_{in}$

**Execute `ADD R1, R2` ($R1\leftarrow R1+R2$):**
4. $R1_{out}$, $Y_{in}$
5. $R2_{out}$, Add, $Z_{in}$
6. $Z_{out}$, $R1_{in}$, End

**Total 6 control steps** (3 fetch + 3 execute). Note step 2 may overlap fetching with PC update.

**Execute `LOAD R1, (R3)`** (register indirect):
4. $R3_{out}$, $MAR_{in}$, Read
5. WMFC (wait), $MDR_{in}E$ (memory data to MDR)
6. $MDR_{out}$, $R1_{in}$, End

**Execute `ADD R1, (R3)`:** read memory into MDR (steps 4–5), $R1_{out}\to Y$ (can be parallel with the read), $MDR_{out}$, Add, $Z_{in}$, then $Z_{out}$, $R1_{in}$.

**Conditional branch (relative):** step 4: $Offset_{out}$ (from IR), Add, $Z_{in}$; step 5: $Z_{out}$, $PC_{in}$ if condition true.

### Multi-bus datapath
Three buses (two source buses A, B and one destination bus C) with a dual-read register file: `ADD R4, R5, R6` executes in **one step**: $R5_{out}\to A$, $R6_{out}\to B$, ALU add, result on C into $R4_{in}$. Fetch also shrinks (PC incrementer is separate): about 2 steps. **Cost: more buses and wider registers, but CPI decreases.** Single-bus: low hardware cost, many steps.

| Datapath | Steps for reg-reg ADD (incl. fetch) | Hardware |
|---|---|---|
| Single bus | 6 | minimal |
| Two bus | 5 | medium |
| Three bus | 2–4 | most |

(Convention varies between textbooks; a GATE question normally supplies the step table.)

## 3. Hardwired control unit

Control signals are produced by a fixed network of gates from the instruction bits, the step counter and status flags.

```text
IR --> instruction decoder --+
                             +--> combinational logic --> control signals
clock --> step counter ---+--+
                          |
status flags -------------+
```

- **Step (timing) counter** (ring counter or binary counter + decoder) emits $T_1,T_2,\dots$
- **Instruction decoder** emits one line per opcode (ADD, LOAD, ...).
- Each control signal is an OR of AND terms, e.g. $Z_{in}=T_1+T_5\cdot ADD+T_4\cdot BR+\dots$, $R1_{out}=T_4\cdot ADD\cdot(\text{src}=R1)+\dots$

**Properties:** fastest (limited only by gate delays), many gates, hard to change or debug, design complexity grows with instruction set. Used in RISC processors. Number of steps varies per instruction; the counter is reset at the end of each instruction (End signal).

**Hardwired size sketch:** for $m$ opcodes and $s$ steps one needs an $\log m\to m$ decoder and an $s$-state counter; each control signal is a sum of products over these lines.

## 4. Microprogrammed control unit

A control unit built like a tiny computer. Each machine instruction maps to a **microroutine** stored in the **control store (control memory)**, usually a ROM.

```text
IR (opcode) -> starting address generator -> uPC (micro program counter / CMAR)
                      ^                          |
                      |                          v
            branch/condition logic <--- control store --> control word -> datapath signals
```

**Terms.** *Microinstruction:* one word of the control store: it encodes the control signals for one step (plus sequencing information). *Microprogram:* sequence of microinstructions implementing a machine instruction (microroutine). *$\mu$PC (control address register):* address of the next microinstruction. *Control word:* the bits that drive control lines.

**Sequencing.** Next address = $\mu PC+1$ (default), or a jump address from the microinstruction's next-address field (branch), or a starting address derived from the opcode via **mapping logic/ROM**, or a return address (microsubroutine). Conditional branching tests status flags selected by a **condition-select field**.

### Microinstruction formats

| | **Horizontal** | **Vertical** |
|---|---|---|
| Encoding | one bit per control signal (no/little encoding) | signals grouped in fields, each field encoded |
| Width | wide (50–100+ bits) | narrow |
| Parallelism | high: many signals at once | limited: one operation per field |
| Decoding delay | none | decoders needed |
| Microprogram length | short | longer |
| Control memory | fewer words, wider | more words, narrower |
| Flexibility | very flexible but costly | compact, slower |

**Encoded (field) formats** are between the two: signals that can never be active in the same step (mutually exclusive) are grouped; a group of $g$ signals needs $\lceil\log_2(g+1)\rceil$ bits if "none active" must be encodable, $\lceil\log_2g\rceil$ otherwise.

### Sizing the control memory (GATE numerics)

**Worked example 1.** A control word has 5 groups of mutually exclusive signals with $7,15,3,31,12$ signals; "no signal" must be encodable.
- **Horizontal:** $7+15+3+31+12=68$ bits.
- **Encoded fields:** $\lceil\log_28\rceil=3$, $\lceil\log_216\rceil=4$, $\lceil\log_24\rceil=2$, $\lceil\log_232\rceil=5$, $\lceil\log_213\rceil=4$ $\Rightarrow$ $3+4+2+5+4=18$ bits.
- Control memory has 512 microinstructions: next-address field $=\log_2512=9$ bits; 4 branch conditions: condition-select field 2 bits.
- **Control word width** $=18+9+2=29$ bits.
- **Control memory size** $=512\times29=14848$ bits $=1856$ bytes. (Horizontal version: $68+9+2=79$ bits, $512\times79=40448$ bits.)

**Worked example 2 (words needed).** Machine with 32 instructions, each microroutine at most 8 microinstructions, plus a common fetch microroutine of 4 microinstructions: control memory $\ge 32\times8+4=260$ words, so 9 address bits (512 words). A 5-bit opcode can be mapped to a start address by placing it above a 3-bit micro-offset (start address $=8\times\text{opcode}$), so each routine owns an 8-word slot and the low 3 bits of the $\mu$PC count within the routine.

**Worked example 3 (speed).** Control store access 20 ns, datapath step 30 ns: with the microinstruction fetch pipelined with execution, the step time is $\max(20,30)=30$ ns; without overlap it is $50$ ns. A hardwired unit takes about the 30 ns datapath time only.

### Nanoprogramming and microprogram optimisations
A two-level control store: first level (micro store) holds addresses into a smaller second-level **nanostore**, which holds the actual control words (saves space when many control words repeat). Also: writable control store (updateable), micro-subroutines, microprogram-level pipelining.

## 5. Hardwired vs microprogrammed

| Aspect | Hardwired | Microprogrammed |
|---|---|---|
| Speed | **fast** | slower (extra control-store access) |
| Flexibility | hard to modify | easy: change the ROM |
| Complexity (large ISA) | grows fast, error-prone | regular, systematic |
| Cost of changes | redesign | new microcode |
| Typical ISA | RISC | CISC |
| Design | sequential logic/gates | memory + sequencer |
| Debug | hard | easier |
| Instruction set size | small | large OK |

**Memory contents:** hardwired needs none; the microprogrammed store is ROM/(W)RAM of size $\text{words}\times\text{width}$.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Subtract in ALU | $A+\overline B+1$ | ALU design |
| Overflow | $C_{n-1}\oplus C_n$ | flags |
| Barrel shifter | $n\log_2n$ 2:1 MUXes | shifter cost |
| Booth | $n$ steps; $(Q_0,Q_{-1})$: $10\to-M$, $01\to+M$ | multiplication |
| Single-bus reg-reg ADD | 3 fetch + 3 exec steps | datapath |
| Encoded field width | $\lceil\log_2(g+1)\rceil$ | microinstruction size |
| Horizontal word | #signals + next-address + condition | microinstruction size |
| Control memory size | #microinstr $\times$ word width | sizing |
| Next-address field | $\lceil\log_2(\#\text{microinstr})\rceil$ | sizing |
| Hardwired | fast, rigid, RISC | comparison |
| Microprogrammed | flexible, slower, CISC | comparison |

## GATE traps

- Field width for a group needs **+1** for "no signal" unless the group is always active.
- Control memory size counts **all** fields: control bits + next address + conditions (+ opcode-related fields), times the number of microinstructions.
- **Horizontal** microprogramming is *faster*, wider and needs *no decoding*; vertical is narrower and slower.
- Single-bus datapath: only **one value on the bus per step**; two source registers cannot both be put on the bus in the same step.
- Booth's algorithm uses **arithmetic** right shift; carry/overflow are ignored in the intermediate adds (use $n$-bit arithmetic and sign extend).
- Subtraction in the ALU also needs CarryIn $=1$, not only inverting $B$.
- Hardwired control is not "no steps": it still has a sequence counter.
- Microprogrammed control is slower due to control-store access per microinstruction, though pipelining the fetch hides some of it.

## Connections

- [Combinational circuits](../09-digital-logic/combinational-circuits.md) — ALU = adder + MUX + logic gates; decoders for the instruction decoder.
- [Sequential circuits](../09-digital-logic/sequential-circuits.md) — registers, shift registers, the ring/step counter and $\mu$PC.
- [Number representation and arithmetic](../09-digital-logic/number-representation-and-arithmetic.md) — 2's complement add/subtract, overflow, Booth.
- [Instruction sets and addressing](instruction-sets-and-addressing.md) — each instruction and addressing mode corresponds to a micro-operation sequence.
- [Pipelining](pipelining.md) — a pipelined datapath divides the single-cycle datapath into stages; control is hardwired and pipelined.
- [Memory hierarchy and cache](memory-hierarchy-and-cache.md) — memory read/write micro-operations (MAR, MDR, WMFC) and wait states.
- [Finite automata](../11-theory-of-computation/regular-languages-and-finite-automata.md) — a control unit is an FSM.

## Practice

**Q1 (NAT).** A control word has fields for 3 groups of mutually exclusive signals containing 6, 14 and 30 signals. Each group may also be inactive. What is the width of the encoded control field (bits)?

<details><summary>Answer</summary>

**Answer:** 12.
**Solution:** $\lceil\log_2(6+1)\rceil=3$; $\lceil\log_2(14+1)\rceil=4$; $\lceil\log_2(30+1)\rceil=5$. Total $12$.

</details>

**Q2 (NAT).** A microprogrammed control unit has 1024 microinstructions, each with a 10-bit next-address field, a 3-bit condition-select field and a 20-bit control field. Control memory size in KB?

<details><summary>Answer</summary>

**Answer:** 4.125 KB (33792 bits).
**Solution:** Width $=10+3+20=33$ bits. Memory $=1024\times33=33792$ bits $=4224$ bytes $=4.125$ KB.

</details>

**Q3 (MCQ).** Which statement about horizontal microprogramming is correct?
(a) It uses encoded fields to minimise width. (b) It allows maximum parallelism of micro-operations with no decoding. (c) It produces longer microprograms than vertical. (d) It is always slower than hardwired control by a factor of 10.

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** One bit per control signal means no decoding and any combination of signals in one step. (a) and (c) describe vertical.

</details>

**Q4 (NAT).** In a single-bus datapath how many control steps does `ADD R1, R2` need (3 fetch steps included)?

<details><summary>Answer</summary>

**Answer:** 6.
**Solution:** Fetch: 3. Execute: $R1\to Y$; $R2$ + add $\to Z$; $Z\to R1$: 3 more. Total 6.

</details>

**Q5 (NAT).** Use Booth's algorithm to compute $(-5)\times(-3)$ with 4-bit registers. Give the product in decimal.

<details><summary>Answer</summary>

**Answer:** 15.
**Solution:** $M=1011$ ($-5$), $Q=1101$ ($-3$), $-M=0101$. Step 1: $(1,0)$: $A=0000+0101=0101$, shift: $A=0010$, $Q=1110$, $Q_{-1}=1$. Step 2: $(0,1)$: $A=0010+1011=1101$, shift: $A=1110$, $Q=1111$, $Q_{-1}=0$. Step 3: $(1,0)$: $A=1110+0101=0011$, shift: $A=0001$, $Q=1111$, $Q_{-1}=1$. Step 4: $(1,1)$: shift only: $A=0000$, $Q=1111$. Result $0000\,1111=15$. (Checked by simulation.)

</details>

**Q6 (MSQ).** Which statements are true?
(a) Hardwired control is generally faster than microprogrammed control.
(b) Changing the instruction set is easier in a microprogrammed unit.
(c) In vertical microprogramming each control signal has its own bit.
(d) A $\mu$PC may be loaded with a mapped address derived from the opcode.

<details><summary>Answer</summary>

**Answer:** (a), (b), (d).
**Solution:** (c) is horizontal.

</details>

**Q7 (NAT).** A 32-bit barrel shifter built from 2:1 multiplexers in $\log_2 n$ stages. How many 2:1 MUXes are required?

<details><summary>Answer</summary>

**Answer:** 160.
**Solution:** $5$ stages $\times32=160$.

</details>

**Q8 (NAT).** A microprogrammed machine has 64 instructions, each needing at most 6 microinstructions, plus a 5-microinstruction fetch routine and 8 words of branch/common routines. Minimum next-address field width (bits) if the control store is sized to the smallest power of two that suffices?

<details><summary>Answer</summary>

**Answer:** 9.
**Solution:** Words needed $=64\times6+5+8=397$; smallest power of two $\ge397$ is $512=2^9$. Next-address field $=9$ bits.

</details>
