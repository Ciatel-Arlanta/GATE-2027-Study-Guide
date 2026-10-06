# Brief: 09-digital-logic + 10-computer-organization

Verify every K-map result, number conversion, IEEE 754 encoding and cache/pipeline calculation with Python.

## 1) 09-digital-logic/ (CS; ~4-6 marks)
Plan topics: Boolean algebra; Boolean minimization: algebraic technique; Karnaugh maps; Tabular minimization method; Combinational circuit design; Sequential circuit design; Number representation; Fixed-point arithmetic; Floating-point arithmetic.
Files: README.md, boolean-algebra-and-minimization.md, combinational-circuits.md, sequential-circuits.md, number-representation-and-arithmetic.md, CHEATSHEET.md, CHECKPOINT.md.
Must cover:
- Boolean algebra: laws, duality, De Morgan, consensus theorem, SOP/POS, canonical forms (minterms/maxterms, Sigma/Pi notation), counting Boolean functions (2^(2^n)), self-dual functions (count), functional completeness (NAND/NOR, and checking a given gate set), XOR/XNOR identities.
- Algebraic minimization worked step by step.
- K-maps for 2-5 variables (Gray-code ordering, ASCII K-map diagrams), prime implicants vs essential prime implicants (counting both — a GATE favourite), don't-cares, POS minimisation from zeros.
- Quine-McCluskey tabular method fully worked (grouping by number of 1s, combining, prime implicant chart, essential PIs, Petrick's method briefly).
- Combinational: half/full adder and subtractor, ripple-carry adder delay, carry-lookahead (generate/propagate, delay), multiplexers (implementing functions with a MUX — a GATE favourite), demultiplexers, decoders/encoders, priority encoder, comparators, code converters, gate-count/delay questions, static hazards (brief).
- Sequential: latches vs flip-flops, SR/D/JK/T characteristic tables and equations and excitation tables, race-around condition and master-slave, flip-flop conversions, setup/hold time and maximum clock frequency, registers and shift registers, counters (asynchronous/ripple vs synchronous, mod-N design, ring and Johnson counters with state counts), designing a synchronous counter from a state diagram, Mealy vs Moore FSMs, finding the state sequence of a given circuit (trace table).
- Number representation: base conversions incl. fractions; sign-magnitude, 1's complement, 2's complement (ranges, negation, overflow detection rules); sign extension; BCD, excess-3, Gray code; fixed-point representation (Q format) and arithmetic; IEEE 754 single and double precision (bias, normalised/denormalised, special values, worked encode/decode, smallest/largest values, precision/ULP); floating-point addition steps (align, add, normalise, round) and rounding; why floating-point addition is not associative.

## 2) 10-computer-organization/ (CS; ~8-10 marks)
Plan topics: Instruction sets; Addressing modes; ALU design; Hardwired control unit; Microprogrammed control unit; Memory interfacing; Memory hierarchy and performance; Cache memory mapping; I/O interface; Interrupts; DMA; Instruction pipelining; Pipeline hazards.
Files: README.md, instruction-sets-and-addressing.md, alu-and-control-unit.md, memory-hierarchy-and-cache.md, io-interrupts-dma.md, pipelining.md, CHEATSHEET.md, CHECKPOINT.md.
Must cover:
- Instruction sets: instruction formats, 0/1/2/3-address instructions (evaluate one expression in each), expanding opcodes (count instructions — a GATE favourite), RISC vs CISC, instruction cycle, register-transfer notation, stack/accumulator/GPR machines, program counter behaviour.
- Addressing modes: immediate, direct, indirect, register, register-indirect, displacement/indexed/base, relative, auto-increment/decrement; effective-address and memory-reference counts with worked examples; which mode suits arrays, pointers, position-independent code.
- ALU and datapath: ALU design from adders and MUXes, single-bus vs multi-bus datapaths, micro-operations, control-signal sequences for an instruction.
- Control unit: hardwired (state/sequence counter, decoder) vs microprogrammed (control memory, control word, microinstruction formats: horizontal vs vertical, encoded fields), sizing the control memory and control-word width (worked numeric), next-address field, comparison table.
- Memory interfacing: chip-organisation arithmetic (how many k x m chips for an N x M memory, address lines, decoder size), memory-mapped vs isolated I/O.
- Memory hierarchy: locality, hit ratio, average memory access time for hierarchical vs simultaneous access, multi-level caches, write-through vs write-back (and write-allocate), effective access time with misses.
- Cache mapping: direct, fully associative, k-way set-associative; tag/index/offset bit split (many worked examples); tag-memory size; mapping a given address sequence to hits/misses for each mapping; replacement policies (LRU, FIFO); miss types (compulsory, capacity, conflict); array-traversal miss counting (row- vs column-major loops); secondary storage only as needed for AMAT.
- I/O: programmed I/O, interrupt-driven I/O (vectored, priority, daisy chaining, interrupt latency/overhead computation), DMA (burst, cycle stealing, transparent modes; percentage of CPU time consumed — worked numerics).
- Pipelining: stages, speedup = nk/(k + n - 1), throughput and efficiency, pipeline with unequal stage delays and latch overhead, CPI; hazards: structural, data (RAW/WAR/WAW), control; stalls, operand forwarding/bypassing (stall counts with and without forwarding — space-time diagrams in ASCII), branch penalty and effective CPI with branch frequency, delayed branch, branch prediction basics.
