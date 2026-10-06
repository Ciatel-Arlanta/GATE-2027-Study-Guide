# Instruction Pipelining and Hazards

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Instruction pipelining; Pipeline hazards
> **Prerequisites:** [ALU and control unit](alu-and-control-unit.md), [Instruction sets and addressing](instruction-sets-and-addressing.md), [Sequential circuits](../09-digital-logic/sequential-circuits.md) · **Leads to:** [Compiler optimization (instruction scheduling)](../12-compiler-design/optimization-and-dataflow.md)

## Quick glance

- **Pipelining** overlaps instruction execution like an assembly line: split into $k$ stages, start a new instruction every cycle. It raises **throughput**, not the latency of one instruction (which even grows slightly because of stage registers).
- $n$ instructions, $k$ stages, equal stage delay: time $=(k+n-1)$ cycles; **speedup** $S=\dfrac{nk}{k+n-1}\to k$ as $n\to\infty$; **efficiency** $=S/k$; **throughput** $=n/((k+n-1)\tau)$.
- Clock period $\tau=\max(\text{stage delay})+\text{latch/register overhead}$. Non-pipelined time per instruction $=\sum\text{stage delays}$.
- **Hazards:** **structural** (resource conflict), **data** (RAW true dependence; WAR/WAW name dependences arise in out-of-order or variable-latency pipelines), **control** (branches).
- Classic 5-stage pipeline (IF, ID, EX, MEM, WB): **without forwarding** a dependent instruction right after an ALU producer stalls **2 cycles** (register file written first half, read second half) or 3 cycles (otherwise). **With forwarding:** ALU result $\to$ ALU **0 stalls**; **load-use: 1 stall**.
- **CPI** $=1+\text{stall cycles per instruction}$; branch term $=f_{branch}\times p_{taken\ or\ mispredicted}\times\text{penalty}$.
- Branch handling: stall, predict not-taken, predict taken, delayed branch, dynamic prediction (1-bit, 2-bit counters, BTB).
- #1 trap: read the **convention** stated in the question (forwarding or not, same-cycle write/read of register file, where branch resolves) before counting stalls.

## 1. The idea and the speedup formulas

Without pipelining, an instruction passes through fetch, decode, execute, memory, write-back one after another before the next starts. With pipelining, while instruction $i$ executes, $i+1$ is being decoded and $i+2$ fetched.

```text
cycle:     1    2    3    4    5    6    7    8
I1:       IF   ID   EX   MEM  WB
I2:            IF   ID   EX   MEM  WB
I3:                 IF   ID   EX   MEM  WB
I4:                      IF   ID   EX   MEM  WB
```

Space-time diagram: columns are cycles, rows are instructions (or stages vs time). Four instructions finish in $k+n-1=5+4-1=8$ cycles (versus $4\times5=20$ unpipelined).

### Formulas (equal stage times, no stalls)
- Pipelined time for $n$ instructions: $T_p=(k+n-1)\,\tau$.
- Non-pipelined: $T_{np}=n\,k\,\tau$.
- **Speedup** $S=\dfrac{nk}{k+n-1}$; maximum $k$ (approached for large $n$).
- **Efficiency** $\eta=\dfrac{S}{k}=\dfrac{n}{k+n-1}$ (fraction of time stage-slots are busy).
- **Throughput** $=\dfrac{n}{(k+n-1)\tau}\to\dfrac1\tau$.

**Worked example 1.** $k=5$, $n=100$: $T_p=104\tau$, $T_{np}=500\tau$, $S=500/104=4.81$, $\eta=96.2\%$. For $n=4$: $S=20/8=2.5$.

### Unequal stage delays and register overhead
With stage delays $d_1..d_k$ and per-stage latch delay $L$:
- Non-pipelined time per instruction $=\sum d_i$ (no latches needed).
- Pipelined clock $\tau=\max d_i+L$.
- Speedup (many instructions) $=\dfrac{\sum d_i}{\max d_i+L}$; for exactly $n$ instructions $S=\dfrac{n\sum d_i}{(k+n-1)(\max d_i+L)}$.

**Worked example 2.** Stage delays 150, 120, 160, 140 ps, latch 10 ps. $\tau=160+10=170$ ps; unpipelined $=570$ ps. **Asymptotic speedup $=570/170=3.35$** (less than $k=4$). For $n=100$: $S=\dfrac{100\times570}{103\times170}=3.26$ (computed).

**Latency of one instruction** $=k\tau=4\times170=680$ ps $>570$ ps: pipelining increases individual latency but raises throughput to one instruction per 170 ps.

**Balancing stages** (making delays equal) improves speedup. **Deeper pipelines** raise the clock rate but increase hazard penalties and latch overhead.

### Pipeline registers and the classic 5-stage RISC pipeline
Stages: **IF** (fetch from memory, PC+4), **ID** (decode, read registers), **EX** (ALU, address/branch target), **MEM** (load/store), **WB** (write register file). Pipeline registers IF/ID, ID/EX, EX/MEM, MEM/WB hold values between stages.

| Instruction | Stages used |
|---|---|
| ALU (R-type) | IF ID EX (MEM idle) WB |
| Load | IF ID EX MEM WB |
| Store | IF ID EX MEM (WB idle) |
| Branch | IF ID EX (MEM) |

## 2. Structural hazards

Two instructions need the **same hardware resource** in the same cycle.

**Example: single memory port.** Instruction fetch (IF of a later instruction) and data access (MEM of a load/store) both use memory in the same cycle $\Rightarrow$ one must wait. If 25% of instructions are loads/stores and each causes one stall: CPI $=1+0.25=1.25$.

**Fixes:** separate instruction and data caches (Harvard-style L1), duplicate/pipeline the functional unit, register file with two read ports and one write port (split-cycle trick), stalls otherwise.

## 3. Data hazards

Instruction $J$ needs data that instruction $I$ (earlier) has not yet produced/stored.

| Type | Meaning | Occurs when |
|---|---|---|
| **RAW** (read after write, true dependence) | $J$ reads a register $I$ writes | always a problem in the simple in-order pipeline |
| **WAR** (anti-dependence) | $J$ writes a register $I$ reads | out-of-order execution, or late reads |
| **WAW** (output dependence) | $J$ writes the register $I$ also writes | out-of-order completion, multi-cycle units |

In the in-order 5-stage pipeline (all writes in WB, all reads in ID) **only RAW** hazards occur.

### Stalls without forwarding
Convention A (**standard textbook, register file written in the first half-cycle and read in the second half**): the consumer's ID can be in the same cycle as the producer's WB. Convention B: ID of the consumer must be after WB (one extra stall). **Always check the question's convention.**

**Worked example 3 (space-time, no forwarding, convention A).**
```text
I1: ADD R1,R2,R3
I2: SUB R4,R1,R5   (depends on R1)
```
```text
cycle:   1   2   3   4   5   6   7   8
I1:     IF  ID  EX  MEM WB
I2:         IF  s   s   ID  EX  MEM WB      (ID waits until I1's WB cycle 5)
```
I2 must read R1 in cycle 5 (when I1 writes in the first half). I2's ID would normally be cycle 3; it is delayed to cycle 5: **2 stall cycles**. A dependent instruction two slots later needs 1 stall, three slots later none. Convention B: 3, 2, 1, 0.

**Worked example 4 (a sequence).** Four instructions:
```text
I1: ADD R1,R2,R3
I2: SUB R4,R1,R5
I3: AND R6,R1,R7
I4: OR  R8,R1,R9
```
Without forwarding (convention A): I2 stalls 2 cycles (ID at cycle 5). I3 follows I2 and its ID is at cycle 6 (R1 already written in cycle 5, fine). I4 ID at cycle 7. Last WB at $7+3=10$: **10 cycles** instead of the ideal $4+5-1=8$ (2 stalls, simulated). **With forwarding: 8 cycles, no stalls.** Under convention B without forwarding: 11 cycles.

### Operand forwarding (bypassing)
Add **bypass paths** from the EX/MEM and MEM/WB pipeline registers back to the ALU inputs, so the result is used as soon as it is computed, without waiting for WB.

```text
cycle:   1   2   3   4   5   6   7
ADD R1:  IF  ID  EX  MEM WB
SUB R4:      IF  ID  EX  MEM WB      EX of SUB uses ALU result from EX/MEM reg: 0 stalls
```
- **ALU result $\to$ next ALU instruction:** forwarded EX/MEM $\to$ EX: **0 stalls**.
- **Load result $\to$ next instruction:** data exists only at the end of MEM, so the dependent instruction's EX must wait: **1 stall** (the **load-use hazard**), even with forwarding (MEM/WB $\to$ EX).

**Worked example 5 (load-use).**
```text
I1: LW  R1, 0(R2)
I2: ADD R3, R1, R4
```
```text
cycle:   1   2   3   4   5   6   7
LW:     IF  ID  EX  MEM WB
ADD:        IF  ID  s   EX  MEM WB
```
ADD's EX at cycle 5 receives the loaded data via MEM/WB forwarding: **1 stall**. A compiler can fill the slot with an independent instruction (instruction scheduling).

**Worked example 6 (longer sequence, simulated).**
```text
I1: LW  R1,0(R2)
I2: ADD R3,R1,R4     (R1: load-use)
I3: SUB R5,R3,R6     (R3 from ALU)
I4: AND R7,R3,R1
I5: OR  R8,R5,R7
```
- **With forwarding:** only I2 stalls (1). Total cycles $=5+5-1+1=10$.
- **Without forwarding (convention A):** I2 waits for LW's WB (ID at cycle 5), I3 waits for I2's WB (ID at 8), I4 ID at 9, I5 waits for I4's WB (ID at 12); last WB at cycle $12+3=15$: **15 cycles** (6 stalls).

**Hazard detection unit:** compares destination registers in the pipeline registers against the source registers of the instruction in ID; for loads in EX (`MemRead`), if the destination matches a source of the instruction in ID, insert a bubble (stall PC and IF/ID).

### Stall counting rule of thumb (5-stage, convention A)

| Producer $\to$ consumer, distance 1 (adjacent) | No forwarding | Forwarding |
|---|---|---|
| ALU $\to$ ALU | 2 | 0 |
| Load $\to$ ALU/use | 2 | **1** |

For distance 2 subtract one stall (no forwarding), distance $\ge3$ none. **Load-use with forwarding: distance 1 $\to$ 1 stall; distance $\ge2\to0$.**

### Software (compiler) fixes
Reorder independent instructions to fill delay slots; insert NOPs where hardware does not interlock (MIPS I). See [Optimization and dataflow](../12-compiler-design/optimization-and-dataflow.md).

## 4. Control hazards

A branch changes the PC; instructions fetched after it may be wrong. The branch outcome and target are known late: in **ID** (with early compare hardware) or in **EX/MEM** (basic pipeline).

**Branch penalty** = number of cycles lost = (stage where the branch resolves) $-1$. Resolution in EX (stage 3) $\Rightarrow$ 2 wasted fetches; in ID (stage 2) $\Rightarrow$ 1 wasted fetch.

```text
cycle:   1   2   3   4   5   6   7
BEQ:    IF  ID  EX  MEM WB
I+1:        IF  ID  x               (flushed: wrong path)
I+2:            IF  x
target:             IF  ID  EX ...  (fetched after EX resolves)
```

### Effective CPI with branches
$$\text{CPI}=1+f_{br}\times(\text{fraction of branches that incur the penalty})\times\text{penalty}.$$

**Worked example 7.** 20% of instructions are branches, 60% of them taken, penalty 2 cycles.
- **Always stall at each branch:** $\text{CPI}=1+0.2\times2=1.4$.
- **Predict not-taken** (penalty only when taken): $1+0.2\times0.6\times2=1.24$.
- **Predict taken** (assume target is known early; penalty when not taken): $1+0.2\times0.4\times2=1.16$ (with penalty 2 for the not-taken case).
- If branch resolves in ID (penalty 1) and predict not-taken: $1+0.2\times0.6\times1=1.12$.
- Execution time $=\text{IC}\times\text{CPI}\times\tau$; the pipeline with CPI 1.24 and $\tau=1$ ns is $\dfrac{1.4}{1.24}=1.13\times$ faster than the always-stall design.

**Worked example 8 (combined).** 5-stage pipeline with forwarding: 25% loads, of which 40% are immediately followed by a dependent instruction (1 stall each); 20% branches, 60% taken, 2-cycle penalty, predict not-taken. $\text{CPI}=1+0.25\times0.4\times1+0.2\times0.6\times2=1+0.10+0.24=1.34$. Speedup over the unpipelined machine with CPI 5 at the same clock $=5/1.34=3.73$.

### Branch handling techniques
| Technique | Idea | Penalty |
|---|---|---|
| **Stall (freeze)** | stop fetching until branch resolved | full penalty always |
| **Predict not-taken** | keep fetching sequentially; flush if taken | penalty only when taken |
| **Predict taken** | fetch from target (needs target address early) | penalty when not taken |
| **Delayed branch** | the instruction(s) after the branch (delay slots) always execute; compiler fills them with useful instructions | 0 if slot filled usefully, else NOP |
| **Static prediction** | backward branches taken (loops), forward not-taken | |
| **Dynamic prediction** | hardware history per branch | |
| **Branch target buffer (BTB)** | cache of (branch address $\to$ target, prediction) | removes target-computation delay |

**Delayed branch example.**
```text
ADD R1,R2,R3     ; independent of the branch
BEQ R4,R5,L      ; branch
NOP/or filled    ; delay slot
```
Move `ADD` into the delay slot: `BEQ R4,R5,L ; ADD R1,R2,R3`. The ADD executes whichever way the branch goes, hiding the penalty (valid only if ADD doesn't affect the branch condition).

### Dynamic prediction: 1-bit and 2-bit
- **1-bit:** remember the last outcome; mispredicts twice per loop execution (on exit and on the first iteration of the next execution).
- **2-bit saturating counter** (states: strongly taken 11, weakly taken 10, weakly not-taken 01, strongly not-taken 00; predict taken if state $\ge10$): needs two consecutive mispredictions to flip the prediction.

**Worked example 9.** A loop branch executes TTTTTTTTTN (9 taken, 1 not taken) repeatedly. **1-bit:** mispredicts at N and then at the first T of the next round: 2 per 10 $=80\%$ accuracy. **2-bit** (steady state, entering round in state 11): N moves the state to 10 (still predict taken), the next T goes back to 11: only the N is mispredicted: 1 per 10 $=90\%$ accuracy.

## 5. Performance summary

$$\text{Speedup from pipelining}=\frac{\text{CPI}_{unpipelined}\times\tau_{np}}{\text{CPI}_{pipelined}\times\tau_{p}},\qquad \text{ideal CPI}_{p}=1.$$
For an equal-stage $k$-stage pipeline with the same clock ratio: $\text{Speedup}=\dfrac{k}{1+\text{stalls per instruction}}$.

**Worked example 10.** $k=5$, stalls per instruction $=0.3$: speedup $=5/1.3=3.85$.

**Beyond the basic pipeline (awareness only):** superscalar (several instructions per cycle, CPI $<1$), out-of-order execution with scoreboarding/Tomasulo (resolves WAR/WAW with renaming), speculative execution; multi-cycle floating-point units introduce WAW hazards and structural hazards.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Pipelined time | $(k+n-1)\tau$ | $n$ instructions |
| Speedup | $nk/(k+n-1)\to k$ | equal stages |
| Efficiency | $S/k$ | |
| Throughput | $n/((k+n-1)\tau)$ | |
| Clock | $\max d_i+L$ | unequal stages |
| Unpipelined time | $\sum d_i$ | |
| CPI | $1+$ stalls/instr | |
| Branch CPI | $1+f_{br}\,p\,\text{penalty}$ | |
| ALU $\to$ ALU, forwarding | 0 stalls | |
| Load-use, forwarding | 1 stall | |
| No forwarding, adjacent RAW | 2 stalls (split-cycle RF), else 3 | |
| Hazard types | structural, data (RAW/WAR/WAW), control | |
| 2-bit predictor | flip prediction after 2 consecutive misses | |

## GATE traps

- **Speedup** formula needs **instruction count**: $nk/(k+n-1)$ for finite $n$; for unequal stages use the **max** delay (plus latch overhead).
- Non-pipelined clock uses the **sum** of the stage delays, not the max.
- Pipelining does **not** reduce single-instruction latency.
- **Load-use** still stalls one cycle even with full forwarding.
- Without forwarding, the number of stalls depends on whether the register file can be written and read in the same cycle; look for the phrase "write in the first half and read in the second half".
- Count **total cycles** as $k+n-1+\text{stalls}$; a stall delays all following instructions.
- Branch penalty = (resolution stage $-1$); predict not-taken costs only on **taken** branches.
- WAR and WAW do not occur in the simple in-order 5-stage pipeline.
- A delayed branch **always** executes the delay-slot instruction, even if the branch is taken.
- Stage delay **includes** the pipeline register overhead only in the pipelined clock.
- Cache misses add stalls on top of hazards (memory stage time, not hazard).

## Connections

- [ALU and control unit](alu-and-control-unit.md) — a pipeline is the multi-cycle datapath with registers between steps; pipelined control signals travel with the instruction.
- [Instruction sets and addressing](instruction-sets-and-addressing.md) — fixed-length load/store RISC instructions make the pipeline regular; complex addressing modes create hazards.
- [Memory hierarchy and cache](memory-hierarchy-and-cache.md) — IF and MEM stage times depend on cache hit time; misses stall the pipeline; separate I/D caches avoid structural hazards.
- [Sequential circuits](../09-digital-logic/sequential-circuits.md) — pipeline registers and clock period $=t_{cq}+t_{logic}+t_{su}$.
- [I/O, interrupts and DMA](io-interrupts-dma.md) — precise interrupts: flush younger instructions, finish older ones.
- [Optimization and dataflow](../12-compiler-design/optimization-and-dataflow.md) — instruction scheduling and dependence analysis are the compiler counterpart of hazards.
- [Intermediate code generation](../12-compiler-design/intermediate-code-generation.md) — branch layout and code generation consider delay slots and branch prediction.

## Practice

**Q1 (NAT).** A 5-stage pipeline with equal stage delays executes 20 instructions with no stalls. What is the speedup over the non-pipelined machine (nearest 2 decimals)?

<details><summary>Answer</summary>

**Answer:** 4.17.
**Solution:** $S=nk/(k+n-1)=20\times5/(5+19)=100/24=4.17$.

</details>

**Q2 (NAT).** A 4-stage pipeline has stage delays 5, 10, 8, 7 ns and a latch delay of 1 ns per stage. What is the maximum asymptotic speedup over the unpipelined processor (whose delay is the sum of stage delays)?

<details><summary>Answer</summary>

**Answer:** 2.73.
**Solution:** Unpipelined $=30$ ns; pipelined clock $=10+1=11$ ns; $30/11=2.73$.

</details>

**Q3 (MCQ).** In a 5-stage pipeline with full forwarding, `LW R1,0(R2)` is immediately followed by `ADD R3,R1,R4`. Number of stall cycles?
(a) 0 (b) 1 (c) 2 (d) 3

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** The loaded value exists only after MEM; ADD's EX must follow, giving 1 stall (load-use hazard).

</details>

**Q4 (NAT).** Instructions: `I1: ADD R1,R2,R3; I2: SUB R4,R1,R5; I3: AND R6,R1,R7; I4: OR R8,R1,R9` in a 5-stage pipeline without forwarding, register file written in the first half and read in the second half of the cycle. Total number of clock cycles?

<details><summary>Answer</summary>

**Answer:** 10.
**Solution:** I2's ID is delayed until I1's WB (cycle 5), 2 stalls; I3, I4 then read R1 fine. Cycles $=4+5-1+2=10$. With forwarding: 8.

</details>

**Q5 (NAT).** 15% of instructions are branches, 70% of them taken. Branch penalty is 3 cycles when the prediction is wrong. With predict-not-taken, what is the CPI (ideal CPI 1)?

<details><summary>Answer</summary>

**Answer:** 1.315.
**Solution:** $1+0.15\times0.7\times3=1+0.315=1.315$.

</details>

**Q6 (MSQ).** Which statements are correct?
(a) A pipeline reduces the latency of a single instruction.
(b) RAW hazards occur in an in-order 5-stage pipeline without forwarding.
(c) Separate instruction and data caches remove the IF/MEM structural hazard.
(d) With a delayed branch the delay slot instruction executes only if the branch is not taken.

<details><summary>Answer</summary>

**Answer:** (b), (c).
**Solution:** (a) latency does not decrease. (d) the delay slot always executes.

</details>

**Q7 (NAT).** A loop-ending branch is taken 99 times then not taken once, repeatedly. A 1-bit predictor and a 2-bit predictor are used (steady state). Misprediction counts per 100 executions are $m_1$ and $m_2$. Give $m_1+m_2$.

<details><summary>Answer</summary>

**Answer:** 3.
**Solution:** 1-bit: mispredicts on the exit (N) and on the first T of the next round: $m_1=2$. 2-bit: only the exit is mispredicted: $m_2=1$. Sum $=3$.

</details>

**Q8 (NAT).** A pipelined machine with forwarding has base CPI 1. 30% of instructions are loads, 50% of loads are followed immediately by a dependent instruction (1 stall each); 10% of instructions are branches, always causing a 2-cycle penalty. What is the CPI?

<details><summary>Answer</summary>

**Answer:** 1.35.
**Solution:** $1+0.3\times0.5\times1+0.1\times2=1+0.15+0.2=1.35$.

</details>
