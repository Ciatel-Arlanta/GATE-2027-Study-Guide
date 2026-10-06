# Instruction Sets and Addressing Modes

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Instruction sets; Addressing modes
> **Prerequisites:** [Number representation and arithmetic](../09-digital-logic/number-representation-and-arithmetic.md), [Sequential circuits](../09-digital-logic/sequential-circuits.md) · **Leads to:** [ALU and control unit](alu-and-control-unit.md), [Pipelining](pipelining.md)

## Quick glance

- An **instruction** = opcode (what to do) + operand specifiers (where the data is). The **instruction set architecture (ISA)** is the programmer-visible contract.
- **0/1/2/3-address** machines: stack, accumulator, 2-operand (dest = dest op src), 3-operand (dest, src1, src2). Fewer addresses = shorter instructions but more of them.
- **Expanding opcode:** fixed-length instruction, unused opcode codes at one level become prefixes for the next level; count by multiplying remaining codes by $2^{\text{extra bits}}$ at each level.
- **RISC:** fixed length, load/store only, many registers, simple modes, easy pipelining. **CISC:** variable length, memory operands, many modes, complex instructions.
- Instruction cycle: **fetch, decode, execute** (+ memory access, write-back), then check interrupts. PC is incremented during fetch; **branches overwrite PC**.
- **Effective address (EA):** direct = address field; indirect = $M[\text{field}]$; register indirect = $R$; indexed/base = field $+R$; PC-relative = PC (already incremented) + offset.
- Memory references per instruction = 1 (fetch) + operand references. Indirect adds one reference per level.
- Auto-increment/decrement suits array scans and stacks; PC-relative gives **position-independent code**; base register gives **relocation**; indexed gives **array access**.
- #1 trap: **PC value used for relative addressing is the address of the next instruction** (after fetch increment).

## 1. Instruction format and the instruction set

An instruction word is divided into fields: **opcode**, one or more **address/operand fields**, and addressing-mode bits.

```text
| opcode | mode | operand 1 | mode | operand 2 | ...
```

Typical categories: data transfer (LOAD, STORE, MOV, PUSH, POP), arithmetic/logic (ADD, SUB, AND, SHIFT), control transfer (JMP, conditional branch, CALL, RET), I/O, system (TRAP, HALT).

**Design issues:** instruction length (fixed vs variable), number of registers, which operands may be in memory, addressing modes, operand sizes. **Wider fields = larger addressable space but longer instructions.**

**Worked example: field widths.** 32-bit fixed instruction with 80 opcodes, 64 registers, 3-register format with a 5-bit shift/other field:
- opcode needs $\lceil\log_2 80\rceil=7$ bits;
- each register $\lceil\log_2 64\rceil = 6$ bits, three registers $=18$;
- total $7+18=25$ bits; $32-25 = 7$ spare bits (could hold a function field or small immediate).

**Worked example: immediate size.** If an I-format instruction has 6-bit opcode, two 5-bit registers and a 16-bit signed immediate, the immediate range is $-2^{15}\ldots2^{15}-1$ and a branch offset counted in words reaches $\pm2^{15}$ words $=\pm2^{17}$ bytes.

## 2. Number of addresses: one expression in each style

Evaluate $X = (A+B)\times(C+D)$. Let `T1`, `T2` be memory temporaries or registers.

**3-address** (`OP dest, src1, src2`): 3 instructions.
```text
ADD T1, A, B      ; T1 = A + B
ADD T2, C, D      ; T2 = C + D
MUL X, T1, T2     ; X  = T1 * T2
```

**2-address** (`OP dest, src`, dest also source): 6 instructions.
```text
MOV R1, A     ; R1 = A
ADD R1, B     ; R1 = A + B
MOV R2, C
ADD R2, D
MUL R1, R2    ; R1 = (A+B)(C+D)
MOV X, R1
```

**1-address** (accumulator `AC` implied): 7 instructions.
```text
LOAD A      ; AC = A
ADD B       ; AC = A+B
STORE T     ; T = AC
LOAD C
ADD D       ; AC = C+D
MUL T       ; AC = (C+D)*T
STORE X
```

**0-address** (stack; operands implied on top of stack, `PUSH/POP` have an address): 8 instructions; this is **postfix** (reverse Polish) $AB+CD+\times$.
```text
PUSH A; PUSH B; ADD; PUSH C; PUSH D; ADD; MUL; POP X
```

| Machine | Instr. count | Instr. length | Memory references |
|---|---|---|---|
| 3-address | 3 (fewest) | longest | 3 fetches + 9 operand refs (A,B,T1; C,D,T2; T1,T2,X) if all in memory |
| 2-address | 6 | medium | |
| 1-address | 7 | short | |
| 0-address | 8 (most) | shortest (arithmetic ops have no address) | |

**Stack machine tracing** of $AB+CD+\times$ with $A=2,B=3,C=4,D=5$: push 2,3 $\to$ ADD $\to5$; push 4,5 $\to$ ADD $\to9$; stack $[5,9]$ $\to$ MUL $\to45$. Infix to postfix: operands stay in order, operators go after their operands (use precedence/associativity). Maximum stack depth for this expression $=3$ (just after pushing $D$ the stack holds $5, 4, 5$).

**Register machines:** GPR machines are **register-memory** (CISC, x86) or **load-store** (RISC: only LOAD/STORE touch memory; arithmetic uses registers).

## 3. Expanding opcodes

With fixed-length instructions you can keep unused opcode bit patterns as **escape prefixes** to give more bits to the opcode of instructions that need fewer operands.

**Algorithm (count at each level):**
1. At level 1 the opcode has $k_1$ bits: $2^{k_1}$ codes. Use $u_1$ of them; free $=2^{k_1}-u_1$.
2. Each free code becomes a prefix; at level 2 the opcode is extended by $e$ bits (the bits of the dropped operand): codes $=\text{free}\times2^{e}$. Use $u_2$; free $=\text{codes}-u_2$.
3. Repeat. The last level's free codes are the number of the last kind of instruction.

**Worked example 1.** 16-bit instruction; 4-bit address fields; 4-bit opcode. Supports 15 three-address, 14 two-address, 31 one-address instructions. Maximum number of zero-address instructions?

- Level 1 (3-address: opcode 4 bits + 3 fields of 4): $16-15=1$ free code.
- Level 2 (2-address: opcode 8 bits): codes $1\times16=16$; use 14, free $=2$.
- Level 3 (1-address: opcode 12 bits): codes $2\times16=32$; use 31, free $=1$.
- Level 4 (0-address: opcode 16 bits): codes $1\times16=16$.

**Answer: 16** (python-checked).

**Worked example 2 (maximise one type).** Same machine with 15 three-address and 14 two-address instructions; max one-address instructions? After level 2, free $=2$ codes, level 3: $2\times16=32$ codes, so up to **32** one-address instructions (and then no zero-address ones).

**Worked example 3.** 12-bit instruction, 3-bit operand fields, 3-bit first-level opcode. Support 6 three-address, 10 two-address and 40 one-address instructions; how many zero-address instructions are possible?

- Level 1 (3-address, opcode 3 bits): $8-6=2$ free codes.
- Level 2 (2-address, opcode 6 bits): $2\times8=16$; use 10, free $=6$.
- Level 3 (1-address, opcode 9 bits): $6\times8=48$; use 40, free $=8$.
- Level 4 (0-address, opcode 12 bits): $8\times8=64$.

**Answer: 64.** Check by counting bit patterns: $6\cdot2^9+10\cdot2^6+40\cdot2^3+64=3072+640+320+64=4096=2^{12}$.

**Check:** the total number of distinct bit patterns of the whole instruction word is $2^{16}$ and each instruction format with $a$ addresses consumes $2^{4a}$ patterns per opcode: $15\cdot2^{12}+14\cdot2^{8}+31\cdot2^{4}+16\cdot1 = 61440+3584+496+16 = 65536 = 2^{16}$ (all patterns used).

## 4. RISC vs CISC

| Aspect | RISC | CISC |
|---|---|---|
| Instruction length | fixed (e.g. 32 bits) | variable |
| Memory access | **load/store only** | any instruction may use memory operands |
| Registers | many (32+) | few |
| Addressing modes | few, simple | many, complex |
| Control unit | **hardwired**, simple decode | often microprogrammed |
| CPI | about 1 (pipeline friendly) | varies, multi-cycle |
| Code size | larger | smaller |
| Examples | MIPS, ARM, RISC-V | x86, VAX |

**RISC is easy to pipeline because fixed length makes fetch/decode uniform and load/store separates memory access.** CISC reduces instruction count and memory use but decoding is harder.

## 5. Instruction cycle and register transfer notation

```text
fetch:   MAR <- PC
         MBR <- M[MAR]
         PC  <- PC + 1        (+ instruction length)
         IR  <- MBR
decode:  control unit interprets opcode
execute: operand fetch (EA computation), ALU operation, store result
interrupt check
```

PC **always** points to the next instruction after fetch. Jump: $PC\leftarrow EA$. Conditional branch: if condition (flags) then $PC\leftarrow PC+\text{offset}$. CALL: push PC (return address), then PC $\leftarrow$ target. RET: PC $\leftarrow$ pop. Interrupt: push PC and status, load ISR address.

**RTN examples:** `ADD R1, R2, R3`: $R1\leftarrow R2+R3$. `LOAD R1, 100(R2)`: $R1\leftarrow M[R2+100]$. `PUSH R1`: $SP\leftarrow SP-1;\ M[SP]\leftarrow R1$ (if the stack grows downward and SP points to the top element).

**Instruction cycle count:** if each memory access takes one cycle, a 2-word `ADD` with a memory operand needs: fetch word 1, fetch word 2, read operand, (write result): 3 or 4 memory cycles.

## 6. Addressing modes

An addressing mode defines how the operand's **effective address (EA)** or value is obtained.

| Mode | Operand / EA | Memory refs for operand | Typical use |
|---|---|---|---|
| Implied/implicit | operand fixed (accumulator, stack top) | 0 | 0-/1-address ops |
| **Immediate** | operand $=$ field itself | 0 | constants |
| **Direct (absolute)** | $EA=\text{field}$ | 1 | global variables |
| **Indirect** | $EA=M[\text{field}]$ | 2 | pointers, jump tables |
| **Register** | operand $=R$ | 0 | fast local data |
| **Register indirect** | $EA=R$ | 1 | pointers, array scan |
| **Displacement (base+offset)** | $EA=R+\text{field}$ | 1 | structures, stack frames |
| **Indexed** | $EA=\text{field}+X$ (X index reg; field = array base) | 1 | arrays |
| **Base-register** | $EA=B+\text{field}$ (B base reg; field = offset) | 1 | relocation |
| **PC-relative** | $EA=PC+\text{field}$ | 1 (0 for branch target) | branches, position-independent code |
| **Auto-increment** | $EA=R$, then $R\leftarrow R+d$ | 1 | array scan, stack pop |
| **Auto-decrement** | $R\leftarrow R-d$, then $EA=R$ | 1 | stack push |
| Stack | top of stack | 1 | stack machines |

(Counts exclude the instruction fetch; add it separately. Multi-level indirection adds one reference per level.)

**Worked example: all modes.** Instruction at address 1000, size 4 bytes, address field $=500$. Memory: $M[500]=700$, $M[700]=900$, $M[600]=25$, $M[1504]=77$. Registers: $R1=600$, $X=100$ (index), $PC=1000$ before fetch.

| Mode | EA | Operand value | Mem refs (operand) |
|---|---|---|---|
| Immediate | – | 500 | 0 |
| Direct | 500 | $M[500]=700$ | 1 |
| Indirect | $M[500]=700$ | $M[700]=900$ | 2 |
| Register ($R1$) | – | 600 | 0 |
| Register indirect ($R1$) | 600 | $M[600]=25$ | 1 |
| Indexed (500 + X) | 600 | 25 | 1 |
| PC-relative (offset 500) | $1004+500=1504$ | $M[1504]=77$ | 1 |
| Auto-increment ($R1$) | 600, then $R1=601$ (byte data) | 25 | 1 |

**Why PC-relative uses 1004:** during fetch the PC is incremented by the instruction size, so by execute time it holds the next instruction's address.

**Worked example: total memory references.** An instruction `ADD (A), R1` with indirect first operand in a 2-word instruction: fetch 2 words $=2$; indirect operand: 2 reads; write result back to memory (if destination is memory): 1. Total $= 2+2+1 = 5$ memory references (state assumptions: one reference per word).

**Worked example: relative branch offset.** Branch instruction at address $2000$ (4 bytes) jumps to $1900$. Offset $=1900-(2000+4)=-104$. If the offset field counts words, offset $=-26$. The offset field must hold $-104$, which needs 8 bits signed (range $-128..127$). If PC-relative offsets were measured from the branch instruction itself (some textbooks), offset $=-100$; **read the question's convention**.

**Choosing modes:**

| Need | Mode |
|---|---|
| Constant | immediate |
| Array element $A[i]$ | indexed or register indirect + auto-increment |
| Pointer variable | indirect/register indirect |
| Local variable on stack | base + displacement (frame pointer) |
| Relocatable programs | base register |
| Position-independent code / short branches | PC-relative |
| Jump table | indirect |
| Stack push/pop | auto-decrement/-increment |

**Effect on instruction size:** immediate size limits constants; register modes need fewer bits (log of register count) than memory addresses. Number of mode bits $=\lceil\log_2(\#\text{modes})\rceil$.

**Worked example: instruction size.** Machine has 100 instructions, 8 addressing modes, 32 registers, 24-bit address. Format `opcode, mode, register, address`: opcode 7 bits, mode 3, register 5, address 24: total $39$ bits (round up to 40 for a byte-aligned format).

## 7. Registers, flags and the stack

- **Program counter (PC)**, **instruction register (IR)**, **MAR/MBR (MDR)**, **stack pointer (SP)**, **status/flags (Z, N, C, V)**, general-purpose registers.
- **Stack** in memory: SP points to top. Growing downward: PUSH $=SP\leftarrow SP-1$ then store; POP $=$ load then $SP\leftarrow SP+1$. Subroutine calls push the return address; **nested calls and recursion** use the stack for frames (see [Runtime environments](../12-compiler-design/runtime-environments.md)).
- Condition codes set by the ALU: branch instructions test them (BEQ tests Z; signed less-than tests $N\oplus V$).

**Worked example: CALL/RET.** SP $=1000$, stack grows down, 4-byte return address. `CALL` at $2000$ (4 bytes) pushes $2004$: SP becomes $996$, $M[996]=2004$; $PC\leftarrow\text{target}$. `RET`: $PC\leftarrow M[996]=2004$, SP $=1000$.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Opcode bits | $\lceil\log_2(\#\text{instr})\rceil$ | format sizing |
| Register field | $\lceil\log_2(\#\text{regs})\rceil$ | format sizing |
| Expanding opcode | free$\times2^{\text{bits released}}$ at each level | counting instructions |
| Fetch | MAR$\leftarrow$PC; MBR$\leftarrow$M; PC$\leftarrow$PC+len; IR$\leftarrow$MBR | RTN |
| PC-relative EA | $PC_{next}+\text{offset}$ | branches |
| Indirect refs | one extra memory read per level | counting references |
| Indexed vs base | index: field = base address; base: field = offset | mode identification |
| Branch offset range | $n$-bit signed: $-2^{n-1}..2^{n-1}-1$ (in words or bytes) | jump reach |
| 0-address evaluation | postfix; stack depth | expression questions |

## GATE traps

- **Expanding opcode:** do not forget to multiply free codes by $2^{\text{released bits}}$ at *each* level, and check whether the question asks for the *maximum* of the last type.
- **PC-relative:** target $=$ address of next instruction $+$ offset, not current address (unless stated).
- **Immediate** has no memory reference; **direct** has one; **indirect** two; count instruction fetch separately and note the question's convention.
- Indexed vs base-register: both compute field $+$ register; which one is "index" depends on what the field holds (array base vs offset).
- In RISC only LOAD/STORE access memory, so `ADD R1, 100(R2), R3` is **not** a RISC instruction.
- Auto-increment adds the **operand size**, not always 1.
- 0-address instructions are not "zero operands": PUSH/POP still have an address. Only ALU ops work on the stack top implicitly.
- Stack growth direction (up/down) and whether SP points to the top element or the next free slot change the arithmetic; read the question.
- Variable-length instructions complicate PC update and pipelining.

## Connections

- [Number representation and arithmetic](../09-digital-logic/number-representation-and-arithmetic.md) — immediate and offset ranges are 2's complement ranges; sign extension of displacements.
- [ALU and control unit](alu-and-control-unit.md) — each instruction becomes a sequence of micro-operations; PC and IR in the datapath.
- [Pipelining](pipelining.md) — fixed-length load/store instructions pipeline easily; addressing modes create hazards and extra stages.
- [Memory hierarchy and cache](memory-hierarchy-and-cache.md) — addressing modes determine the memory reference stream seen by the cache.
- [Runtime environments](../12-compiler-design/runtime-environments.md) — stack frames, base/frame pointers and displacement addressing.
- [Pointers, arrays and strings](../05-c-programming/pointers-arrays-strings.md) — C array indexing and pointers compile to indexed and register-indirect modes.
- [Intermediate code generation](../12-compiler-design/intermediate-code-generation.md) — three-address code is the compiler counterpart of 3-address instructions.

## Practice

**Q1 (NAT).** A processor has 16-bit instructions with a 4-bit opcode and three 4-bit register fields. 13 three-address instructions are needed, and the rest of the opcode space is expanded to give the maximum number of 2-address instructions with 8-bit opcodes. How many 2-address instructions can be supported?

<details><summary>Answer</summary>

**Answer:** 48.
**Solution:** Level 1: $16-13=3$ free codes. Level 2: $3\times16=48$ codes available (opcode 8 bits, two 4-bit address fields).

</details>

**Q2 (MCQ).** Which addressing mode makes it possible to relocate a program in memory without changing its instructions? (a) direct (b) base register (c) immediate (d) indirect

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** Changing the base register value moves all addresses together; PC-relative also supports position independence for code, but among the options only the base register is correct.

</details>

**Q3 (NAT).** A 2-word instruction (each word 1 memory cycle to fetch) uses memory-indirect addressing for its source operand and stores the result in a register. How many memory references to execute it?

<details><summary>Answer</summary>

**Answer:** 4.
**Solution:** 2 instruction fetches $+$ 1 read of the pointer $+$ 1 read of the operand $=4$; register destination adds none.

</details>

**Q4 (NAT).** Instruction at address 3000 occupies 4 bytes. It is PC-relative with offset field $+120$ (bytes). Effective address?

<details><summary>Answer</summary>

**Answer:** 3124.
**Solution:** PC after fetch $=3004$; $3004+120=3124$.

</details>

**Q5 (MSQ).** Which statements are true?
(a) RISC processors usually have fixed-length instructions.
(b) In a load-store architecture, `ADD R1, 8(R2), R3` is legal.
(c) Auto-increment addressing is useful for scanning arrays.
(d) A 0-address machine evaluates expressions with a stack.

<details><summary>Answer</summary>

**Answer:** (a), (c), (d).
**Solution:** (b) violates load/store: memory operands only in LOAD/STORE.

</details>

**Q6 (NAT).** Evaluate $Y=(A-B)/(C+D\times E)$ on a 1-address (accumulator) machine with `LOAD`, `STORE`, `ADD`, `SUB`, `MUL`, `DIV` (acc op memory). Minimum number of instructions?

<details><summary>Answer</summary>

**Answer:** 8.
**Solution:** Compute denominator first: `LOAD D; MUL E; ADD C; STORE T` (4) then numerator `LOAD A; SUB B` (2) but division is `AC / mem`: `DIV T` (1); `STORE Y` (1). Total $4+2+1+1=8$.

</details>

**Q7 (NAT).** A 32-bit instruction format has a 6-bit opcode and three 5-bit register specifiers. How many bits are left over for other fields?

<details><summary>Answer</summary>

**Answer:** 11.
**Solution:** $6+3\times5=21$ bits used; $32-21=11$ bits remain (for example a shift amount or function code).

</details>

**Q8 (NAT).** Use the machine state of the worked table in section 6 ($M[500]=700$, $M[700]=900$, $M[600]=25$, $X=100$). The value fetched by an indexed operand (field 500, index register $X$) is added to the value fetched by a one-level indirect operand (field 500). What is the sum?

<details><summary>Answer</summary>

**Answer:** 925.
**Solution:** Indexed: $EA=500+100=600$, value $M[600]=25$. Indirect: $EA=M[500]=700$, value $M[700]=900$. Sum $=25+900=925$.

</details>
