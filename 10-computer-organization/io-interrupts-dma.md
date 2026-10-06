# I/O Interface, Interrupts and DMA

> **Paper:** CS · **Priority:** P1 · **Plan topics:** I/O interface; Interrupts; DMA
> **Prerequisites:** [Memory hierarchy and cache](memory-hierarchy-and-cache.md), [Sequential circuits](../09-digital-logic/sequential-circuits.md) · **Leads to:** [Operating systems](../13-operating-systems/README.md) (device management, interrupt handling)

## Quick glance

- An **I/O interface** (controller) sits between the CPU bus and a device: it has **data, status and control registers** (ports) and translates speed/format.
- **Memory-mapped I/O:** device registers share the memory address space, ordinary LOAD/STORE; **isolated I/O:** separate address space with IN/OUT.
- Three transfer techniques: **programmed I/O** (CPU polls status; busy-waiting), **interrupt-driven I/O** (device signals when ready; CPU does useful work meanwhile), **DMA** (controller moves blocks between device and memory without the CPU per word).
- **Interrupt service:** finish current instruction, **save PC/flags**, identify the source, run the ISR, **restore and return**. Overhead per interrupt is the fixed cycles to enter, save/restore and return, plus ISR work.
- **Vectored** interrupts: device supplies the ISR address (vector); **non-vectored**: one common ISR must poll. **Priority:** daisy chain (position = priority), parallel priority encoder, polling.
- **DMA modes:** **burst** (block transfer, CPU locked out), **cycle stealing** (one word per bus cycle then release), **transparent** (use only cycles the CPU is not using the bus).
- **CPU fraction** for polling $=\dfrac{\text{polls/s}\times\text{cycles per poll}}{\text{clock rate}}$; for interrupts $=\dfrac{\text{interrupts/s}\times\text{cycles per interrupt}}{\text{clock rate}}$; for DMA only the setup + completion interrupt per **block** count.
- #1 trap: interrupt cost is per **transfer unit** (per word/byte); DMA cost is per **block**, so DMA wins for fast block devices.

## 1. I/O interface

**Peripheral devices** differ in speed (keyboard: bytes/s; disk: MB/s), data format, and timing, so each is attached through an **interface/controller**.

Typical registers:
- **Data register** (input/output buffer),
- **Status register** (READY/BUSY, ERROR bits),
- **Control register** (start, interrupt enable, mode).

The CPU addresses these registers as either memory locations (memory-mapped) or I/O ports (isolated).

| | Memory-mapped | Isolated (port-mapped) |
|---|---|---|
| Address space | shared with memory | separate (e.g. 64K ports) |
| Instructions | any load/store/ALU op can touch the device | special IN/OUT |
| Control line | none extra | IO/$\overline{M}$ distinguishes |
| Memory addresses lost | yes | no |
| Flexibility | high (all addressing modes) | limited |
| Examples | ARM, MIPS, RISC-V | x86 IN/OUT |

**Handshaking:** data transfer between asynchronous parties uses a strobe (one-wire) or a two-wire request/acknowledge handshake: the sender places data and raises REQ; the receiver reads and raises ACK; both withdraw and repeat. Synchronous buses use a common clock instead. **Bus bandwidth** $=\dfrac{\text{bus width (bytes)}\times\text{clock}}{\text{cycles per transfer}}$; e.g. 32-bit bus, 100 MHz clock, 4 cycles per transfer gives $4\times100\text{ MHz}/4=100$ MB/s; with burst transfers (4 cycles for the first word, 1 cycle for each of the next 3) 16 bytes need 7 cycles.

**I/O processors / channels** are small processors that execute channel programs, offloading even the DMA bookkeeping from the CPU.

## 2. Programmed I/O (polling)

The CPU loops reading the status register until READY, then moves the data:

```text
loop:  IN   R1, STATUS
       AND  R1, #READY
       JZ   loop          ; busy-wait
       IN   R2, DATA
```

**Simple, but the CPU is occupied for the whole time**, even when the device is slow. Suitable for fast devices read in short bursts or when the system has nothing else to do.

**CPU fraction used by polling** $=\dfrac{\text{polls per second}\times\text{cycles per poll}}{\text{clock rate}}$. The device must be polled **at least as often as it produces data units** so no data is lost.

**Worked example 1.** CPU 1 GHz; a poll takes 400 cycles.
- **Mouse**, polled 30 times/s: $30\times400=12000$ cycles/s $=0.0012\%$ of the CPU.
- **Floppy-style device** delivering 50 KB/s in 2-byte units $\to25000$ polls/s: $25000\times400=10^7$ cycles/s $=1\%$.
- **Disk** at 4 MB/s ($10^6$ bytes = 1 MB here) in 16-byte chunks $\to250000$ polls/s: $2.5\times10^5\times400=10^8$ cycles/s $=10\%$ (while the transfer is active).

## 3. Interrupt-driven I/O

The device raises an **interrupt request (IRQ)** when it is ready; the CPU continues computing until then.

### Interrupt service sequence
1. Device asserts INTR.
2. The CPU finishes the **current instruction** (interrupts are recognised at instruction boundaries; a long instruction may be interruptible).
3. If interrupts are enabled (IF = 1), the CPU **acknowledges** (INTA) and disables further interrupts (or raises its priority).
4. The CPU **saves the PC** (and flags/status) on the stack.
5. The CPU obtains the **ISR address**: fixed location (non-vectored), or from the device's **vector** (vectored; vector $\to$ interrupt vector table).
6. ISR executes: save registers, handle device, re-enable interrupts if nested interrupts allowed.
7. **Return from interrupt (IRET):** restore flags and PC; resume the interrupted program.

**Interrupt latency** = time from the request to the first ISR instruction (waiting for the current instruction to end + acknowledge + save + vector fetch). **Interrupt overhead** = cycles consumed by this entry/exit sequence plus ISR run time.

**Worked example 2 (overhead).** CPU 1 GHz. A disk transfers at 4 MB/s ($=4\times10^6$ B/s) in 4-byte words, one interrupt per word, and each interrupt costs 500 cycles in total.
- Interrupts per second $=4\times10^6/4=10^6$.
- Cycles per second $=10^6\times500=5\times10^8$.
- **CPU fraction $=5\times10^8/10^9=50\%$.** Interrupt-driven I/O is a bad fit for fast block devices.

**Worked example 3 (periodic interrupt).** A timer interrupts every 10 ms; the ISR takes 100 $\mu$s plus 20 $\mu$s entry/exit. CPU time used $=120/10000=1.2\%$.

**Worked example 4 (useful time).** A program needs 4 s of pure CPU time. An interrupt every 1 ms with 50 $\mu$s per interrupt adds $5\%$ overhead $\Rightarrow$ total time $=4/(1-0.05)=4.21$ s (interrupts occupy 5% of wall-clock time). Alternatively, if overhead is added to the work: $4\times1.05=4.2$ s. **Read whether the rate is per unit wall-clock time or per unit work.**

### Types of interrupts
| Type | Source | Example |
|---|---|---|
| Hardware, **maskable** | I/O device on INTR | keyboard, disk |
| Hardware, **non-maskable (NMI)** | cannot be disabled | power failure, memory parity error |
| **Software (trap)** | executes `INT n`/syscall | system calls |
| **Exception** | caused by the executing instruction | divide by zero, page fault, illegal opcode, overflow |

Synchronous (exceptions, traps) vs asynchronous (device interrupts). **Precise interrupts** leave the machine in a state as if all prior instructions completed and none after (important in pipelines).

### Priority and multiple devices
1. **Polling (software):** a common ISR reads each device's status in priority order; simple, slow.
2. **Daisy chain (hardware):** the INTA signal passes serially from the device with the highest priority outward; the first requesting device blocks the signal and puts its vector on the bus. **Priority = position in the chain.**

```text
CPU --INTA--> [Dev1] --> [Dev2] --> [Dev3]  (highest priority nearest the CPU)
        <------------- INTR (wired-OR) -----
```

3. **Parallel priority:** each device has its own request line feeding a **priority encoder** (and mask register); fastest, most hardware; vector from the encoder. See [Combinational circuits](../09-digital-logic/combinational-circuits.md).
4. **Nested interrupts:** a higher-priority request can interrupt an ISR; lower or equal priority are held pending (masking).

**Masking:** an interrupt mask register enables or disables each source; the CPU's interrupt-enable flag disables all maskable interrupts.

**Worked example 5 (priority).** Devices D1, D2, D3 on a daisy chain (D1 nearest). D2 and D3 request simultaneously: D2 receives INTA first and blocks it, so **D2 is served first**; D3 waits until D2's request clears. If D1 requests during D2's ISR and nesting is enabled, D1 preempts.

## 4. Direct memory access (DMA)

A **DMA controller** moves data between an I/O device and memory by itself acting as a bus master. Registers: **address register** (memory address), **word/byte count**, **control/status** (direction, start).

**Transfer sequence:**
1. CPU programs the DMA controller: memory address, count, direction. **(Setup)**
2. DMA asserts a **bus request** (HOLD/BR); the CPU finishes the current bus cycle and grants the bus (HLDA/BG).
3. For each word: the DMA controller drives address and control lines, data goes device $\leftrightarrow$ memory directly; address register increments, count decrements.
4. When count reaches 0, the DMA controller raises an **interrupt** to the CPU. **(Completion)**

The CPU is involved only at the start and the end of the block.

### DMA modes
| Mode | Behaviour | Effect on CPU |
|---|---|---|
| **Burst (block)** | DMA holds the bus for the entire block | CPU blocked from memory during the burst; highest transfer speed |
| **Cycle stealing** | DMA takes the bus for one word (one memory cycle) then returns it | CPU slowed slightly; CPU and DMA interleave |
| **Transparent (hidden)** | DMA uses the bus only in cycles the CPU is not using it (e.g. during internal ALU operations) | no CPU slowdown; slowest, needs special support |

**Interrupt vs DMA: while DMA is running, the CPU can still execute from cache or registers; contention occurs only for the memory bus.**

**Worked example 6 (DMA overhead).** CPU 1 GHz. DMA setup 1000 cycles; completion interrupt 500 cycles. The same disk (4 MB/s) transfers 8 KB blocks ($8000$ B).
- Blocks per second $=4\times10^6/8000=500$.
- Cycles per second $=500\times(1000+500)=750000$.
- **CPU fraction $=7.5\times10^5/10^9=0.075\%$**, versus 50% for per-word interrupts and 10% for polling. **DMA wins by a factor of about 650 over interrupts here.**

**Worked example 7 (bus stealing).** Memory cycle 100 ns; DMA moves 4-byte words. Disk at 4 MB/s $\Rightarrow10^6$ words/s.
- **Cycle stealing:** the bus is taken $10^6$ times per second for 100 ns each: $10^6\times100\text{ ns}=0.1$ s per second $=10\%$ of memory bandwidth.
- **Burst mode:** an 8 KB block = 2000 words $=200\ \mu s$ per burst; $500$ bursts/s $=0.1$ s per second $=10\%$ of the time the CPU is locked out (but in long uninterrupted chunks of $200\ \mu s$, which may harm other time-critical devices).
Both modes steal the **same total** bandwidth, but burst mode makes the CPU wait longer at a stretch (poor interrupt latency).

**Worked example 8 (time for a block transfer).** Burst of 1024 words over a bus with 10 ns per transfer cycle: $1024\times10\text{ ns}=10.24\ \mu s$ plus setup time. In cycle-stealing mode, if the device produces a word every $1\ \mu s$ and the DMA steals one 10 ns cycle per word, the CPU is slowed by $10/1000=1\%$.

### Comparison

| | Programmed | Interrupt-driven | DMA |
|---|---|---|---|
| CPU involvement | busy-waits throughout | per word/byte (ISR) | only setup and completion |
| Hardware cost | lowest | moderate | highest (controller) |
| Best for | rare, short, fast-ready devices | slow/sporadic devices (keyboard) | high-speed block devices (disk, network) |
| Overhead scales with | polls/s | interrupts/s (data rate/unit size) | blocks/s |
| CPU does other work | no | yes | yes |

**Cache interaction:** DMA writes to memory bypass the cache, so the cache may hold **stale** data (and a write-back cache may hold newer data than memory). Systems flush/invalidate cache lines or use coherent DMA. See [Memory hierarchy and cache](memory-hierarchy-and-cache.md).

## 5. Bus arbitration (brief)

When several masters (CPU, DMA controllers) want the bus: **daisy-chain arbitration** (bus grant passes along the chain, position = priority), **centralized parallel arbitration** (each master has its own request/grant lines to an arbiter), **distributed** (self-selection by priority codes). The CPU has the lowest priority for the bus in many designs (DMA preempts it at cycle boundaries).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Polling CPU fraction | $\dfrac{\text{polls/s}\times\text{cycles}}{\text{clock}}$ | overhead questions |
| Interrupt CPU fraction | $\dfrac{\text{interrupts/s}\times\text{cycles}}{\text{clock}}$ | overhead questions |
| DMA CPU fraction | $\dfrac{\text{blocks/s}\times(\text{setup}+\text{completion})}{\text{clock}}$ | DMA overhead |
| Interrupts per second | data rate / unit size | per-word interrupts |
| Cycle stealing bandwidth share | words/s $\times$ memory cycle time | bus contention |
| Bus bandwidth | width $\times$ clock / cycles per transfer | bus questions |
| Daisy chain | priority = position nearest CPU | priority |
| Vectored | device sends vector $\to$ ISR address | interrupt types |

## GATE traps

- **Unit of transfer:** interrupts per second $=$ rate/unit size, **not** rate. A disk giving 4 MB/s in 4-byte units is $10^6$ interrupts/s.
- Include **both** setup and completion in DMA overhead; per block, not per word.
- Distinguish **cycle stealing** (CPU delayed by one cycle per DMA word) from **burst** (CPU blocked for the whole block).
- Interrupts are recognised **after the current instruction** (except long-running string instructions), not immediately.
- The CPU saves the **PC** automatically at an interrupt; other registers are the ISR's job.
- **NMI** cannot be masked; **traps/exceptions** are synchronous and cannot be masked either.
- "1 MB/s" may be $10^6$ or $2^{20}$ B/s: use the question's convention (GATE often uses $10^6$ for rates, $2^{20}$ for sizes; stay consistent and state it).
- A device is **not** "polled" in interrupt-driven I/O, except in a shared-ISR software poll to find the source.
- Memory-mapped I/O makes **cache** treatment of device addresses important (device registers must be uncached).

## Connections

- [Combinational circuits](../09-digital-logic/combinational-circuits.md) — priority encoder for interrupt priority; decoders for device selection.
- [Sequential circuits](../09-digital-logic/sequential-circuits.md) — status/control/data registers, handshake logic as FSMs.
- [Memory hierarchy and cache](memory-hierarchy-and-cache.md) — memory-mapped I/O and DMA cache coherence; bus contention on memory.
- [Instruction sets and addressing](instruction-sets-and-addressing.md) — IN/OUT and memory-mapped addressing; saving PC/flags on the stack.
- [Pipelining](pipelining.md) — precise interrupts and exceptions in a pipeline flush in-flight instructions.
- [Operating systems](../13-operating-systems/README.md) — interrupt handlers, system calls (traps), device drivers, context switch on timer interrupt.
- [Computer networks](../15-computer-networks/README.md) — NIC DMA and interrupt coalescing handle high packet rates.

## Practice

**Q1 (MCQ).** Which I/O technique makes the CPU idle-wait on the device status? (a) interrupt-driven (b) DMA (c) programmed I/O (d) memory-mapped I/O

<details><summary>Answer</summary>

**Answer:** (c).
**Solution:** Programmed I/O loops on the status register (busy-waiting). Memory-mapped is an addressing scheme, not a transfer technique.

</details>

**Q2 (NAT).** CPU 500 MHz. Each interrupt costs 250 cycles. A device generates 100000 interrupts per second. What percentage of CPU time goes to servicing it?

<details><summary>Answer</summary>

**Answer:** 5%.
**Solution:** $10^5\times250=2.5\times10^7$ cycles/s; $/5\times10^8=0.05$.

</details>

**Q3 (NAT).** A disk transfers data at 2 MB/s ($2\times10^6$ B/s) in 2-byte units via programmed I/O polling; each poll costs 200 cycles on a 2 GHz CPU. What fraction of the CPU is spent polling (as %)?

<details><summary>Answer</summary>

**Answer:** 10%.
**Solution:** Polls/s $=10^6$ (one per 2 bytes); cycles $=2\times10^8$; $/2\times10^9=0.1$.

</details>

**Q4 (NAT).** DMA transfers 4 KB blocks ($4000$ B for the rate calculation) from a device at 8 MB/s ($8\times10^6$ B/s). Setup 2000 cycles, completion interrupt 1000 cycles, CPU 1 GHz. What percentage of CPU time does DMA overhead take?

<details><summary>Answer</summary>

**Answer:** 0.6%.
**Solution:** Blocks/s $=8\times10^6/4000=2000$. Cycles $=2000\times3000=6\times10^6$. Fraction $=6\times10^6/10^9=0.006=0.6\%$.

</details>

**Q5 (MSQ).** Which are true of DMA?
(a) In cycle-stealing mode DMA takes the bus for one word at a time.
(b) In burst mode the CPU is blocked from the bus for the whole block.
(c) The DMA controller interrupts the CPU after each word transferred.
(d) A DMA transfer may leave the cache with stale data.

<details><summary>Answer</summary>

**Answer:** (a), (b), (d).
**Solution:** (c) is false: the interrupt comes once, when the count reaches zero.

</details>

**Q6 (MCQ).** In a daisy-chained interrupt system with devices D1 (nearest CPU) to D4 (farthest), D2 and D4 raise requests at the same time. Which is served first?
(a) D4 (b) D2 (c) D1 (d) depends on request time only

<details><summary>Answer</summary>

**Answer:** (b).
**Solution:** The acknowledge signal reaches D2 first and is blocked there; priority is by chain position.

</details>

**Q7 (NAT).** Memory cycle time is 50 ns. A DMA controller in cycle-stealing mode transfers 4-byte words at 20 MB/s ($20\times10^6$ B/s). What percentage of memory cycles does the DMA take?

<details><summary>Answer</summary>

**Answer:** 25%.
**Solution:** Words/s $=20\times10^6/4=5\times10^6$; each steals 50 ns: $5\times10^6\times50\times10^{-9}=0.25$ s per second $=25\%$.

</details>

**Q8 (NAT).** A system transfers a 1 MB file ($10^6$ B) from disk. Option A: interrupt per 4-byte word, 400 cycles per interrupt. Option B: DMA in 10000 B blocks, 3000 cycles setup + completion per block. Ratio of CPU cycles consumed (A : B)?

<details><summary>Answer</summary>

**Answer:** about 333.3.
**Solution:** A: $10^6/4=250000$ interrupts $\times400=10^8$ cycles. B: $10^6/10^4=100$ blocks $\times3000=3\times10^5$ cycles. Ratio $=10^8/(3\times10^5)=333.3$ (A uses about 333 times the CPU cycles of B).

</details>
