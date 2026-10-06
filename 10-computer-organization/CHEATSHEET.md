# Computer Organization: Cheat sheet

One-page revision. Details: [Instruction sets](instruction-sets-and-addressing.md) · [ALU and control](alu-and-control-unit.md) · [Memory and cache](memory-hierarchy-and-cache.md) · [I/O](io-interrupts-dma.md) · [Pipelining](pipelining.md).

## Instruction sets and addressing

| Item | Fact |
|---|---|
| Bits | opcode $\lceil\log_2\#\text{instr}\rceil$, register $\lceil\log_2\#\text{regs}\rceil$, mode $\lceil\log_2\#\text{modes}\rceil$ |
| Expanding opcode | free codes $\times2^{\text{released bits}}$ at each level (16-bit, 4-bit fields: 15,14,31 $\Rightarrow$ 16 zero-address) |
| $(A+B)\times(C+D)$ | 3-addr: 3 instr; 2-addr: 6; 1-addr: 7; 0-addr (stack): 8 |
| Fetch | MAR$\leftarrow$PC; MBR$\leftarrow$M[MAR]; PC$\leftarrow$PC+len; IR$\leftarrow$MBR |
| RISC | fixed length, load/store, many registers, hardwired, easy to pipeline |
| CISC | variable length, memory operands, many modes, microprogrammed |

| Mode | EA | Operand mem refs |
|---|---|---|
| Immediate | none | 0 |
| Direct | field | 1 |
| Indirect | $M[\text{field}]$ | 2 |
| Register | none | 0 |
| Register indirect | $R$ | 1 |
| Indexed / base / displacement | field $+R$ | 1 |
| PC-relative | $PC_{next}+\text{field}$ | 1 |
| Auto-inc / dec | $R$ then $R\pm d$ / $R\mp d$ then $R$ | 1 |

Use: indexed = arrays; base = relocation; PC-relative = position-independent code, branches; indirect = pointers, jump tables; auto-inc = scans, stacks.

## ALU and control unit

| Item | Fact |
|---|---|
| Subtract | $A+\overline B+1$ (Binvert=1, CarryIn=1); overflow $C_{n-1}\oplus C_n$; Z = NOR of result |
| Barrel shifter | $n\log_2n$ 2:1 MUXes |
| Booth | $n$ steps; $(Q_0,Q_{-1})$: $10\to A-M$, $01\to A+M$; arithmetic shift right |
| Single-bus reg-reg ADD | 3 fetch + 3 execute steps (R1$\to$Y; R2+Y$\to$Z; Z$\to$R1) |
| 3-bus ADD | 1 execute step |
| Hardwired | step counter + decoder + gates; fast, rigid, RISC |
| Microprogrammed | control store + $\mu$PC; flexible, slower, CISC |
| Horizontal | 1 bit/signal, wide, parallel, no decode |
| Vertical / encoded | fields; width $\lceil\log_2(g+1)\rceil$; narrower, decode delay |
| Control memory size | #microinstr $\times$ (control field + next-address $\log_2$#microinstr + condition select) |

## Memory interfacing and hierarchy

| Item | Formula |
|---|---|
| Chips | $(N/k)(M/m)$; chip-select decoder on upper $\log_2(N/k)$ address bits |
| AMAT hierarchical | $t_1+(1-h)t_2$ |
| AMAT simultaneous | $ht_1+(1-h)t_2$ |
| Multi-level | $t_1+m_1(t_2+m_2t_3)$ (local miss rates) |
| CPI with cache | CPI$_{base}$ + refs/instr $\times$ miss rate $\times$ penalty |
| Disk time | seek + $\frac12\cdot60/\text{rpm}$ + transfer |
| Virtual-memory EMAT | $(1-p)t_m+p\,t_{fault}$ |

## Cache

| Mapping | Index bits | Tag bits | Comparators |
|---|---|---|---|
| Direct | $\log_2L$ | $A-\text{idx}-\text{off}$ | 1 |
| $k$-way | $\log_2(L/k)$ | $A-\text{idx}-\text{off}$ | $k$ |
| Fully assoc. | 0 | $A-\text{off}$ | $L$ |

- $L=C/B$ lines; offset $=\log_2B$; tag store $=L\times(\text{tag}+\text{valid}+\text{dirty})$.
- Write-through (no dirty bit, write buffer) vs write-back (dirty bit, evict-time write). Write-allocate vs no-allocate.
- 3 Cs: compulsory, capacity, conflict (none in fully associative).
- LRU never has Belady's anomaly; FIFO can (1 2 3 4 1 2 5 1 2 3 4 5: FIFO 9 faults with 3 frames, 10 with 4).
- Row-major scan: 1 miss per $B/\text{elem}$ elements; column-major with large stride: a miss per access.

## I/O, interrupts, DMA

| Item | Fact |
|---|---|
| Polling CPU % | polls/s $\times$ cycles per poll / clock |
| Interrupt CPU % | (rate / unit size) $\times$ cycles per interrupt / clock |
| DMA CPU % | blocks/s $\times$ (setup + completion) / clock |
| Interrupt sequence | finish instruction; save PC/flags; vector; ISR; IRET |
| Priority | daisy chain (position), parallel (priority encoder), polling |
| DMA modes | burst (block hold), cycle stealing (word at a time), transparent |
| Memory-mapped I/O | shared address space, normal load/store; isolated: IN/OUT |
| Cycle-stealing share | words/s $\times$ memory cycle time |

## Pipelining

| Item | Formula |
|---|---|
| Time | $(k+n-1)\tau$; unpipelined $nk\tau$ |
| Speedup | $nk/(k+n-1)\to k$; unequal: $\sum d/(\max d+L)$ |
| Efficiency | $S/k$ |
| Clock | $\max d_i+L$ |
| CPI | $1+$ stalls per instr |
| Branch | CPI $=1+f_{br}\,p\,\text{penalty}$ (predict not-taken: $p$ = taken fraction) |

| Hazard | Stalls (5-stage, RF written first half) |
|---|---|
| RAW, no forwarding, adjacent | 2 (3 if no same-cycle write/read) |
| RAW distance 2 | 1 (no forwarding) |
| ALU $\to$ ALU, forwarding | 0 |
| Load-use, forwarding | 1 |
| Branch resolved in EX / ID | 2 / 1 |

- Hazards: structural, data (RAW/WAR/WAW), control; only RAW in in-order 5-stage.
- Delayed branch: delay slot always executes. 2-bit predictor: two misses to flip.

## Remember

- PC-relative uses the **next** instruction's address.
- State the convention: forwarding yes/no, RF same-cycle, branch resolved in EX/ID, hierarchical vs simultaneous access.
- Index bits come from the **number of sets**.
