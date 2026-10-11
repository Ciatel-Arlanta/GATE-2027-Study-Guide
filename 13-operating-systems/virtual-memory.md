# Virtual memory and page replacement

> **Paper:** CS · **Priority:** P0 · **Plan topics:** demand paging, page faults, replacement, working set, TLB/EMAT, thrashing
> **Prerequisites:** [Memory management](memory-management.md) · **Leads to:** [Computer architecture cache](../10-computer-organization/memory-hierarchy-and-cache.md)
> **PYQ weightage:** High · 24 Qs since 2015 (13 since 2021) · DA: 0 · Trend: steady · Most asked: Virtual Memory, Page Replacement, Demand Paging, TLB

## Quick glance

- **Exam focus:** Page replacement traces (FIFO, LRU, OPT), Belady's anomaly, Translation Lookaside Buffer (TLB) hit ratios and Effective Memory Access Time (EMAT) calculations, demand paging page-fault penalty, and working set model / thrashing. Usually asked as high-scoring numerical NATs and conceptual MSQs.
- **Demand paging** brings pages into physical RAM only upon reference. An unmapped reference raises a **page-fault trap** to the OS kernel.
- **Page fault handling:** Trap to OS $\to$ save registers $\to$ check legality $\to$ locate free frame (or evict victim; write back if dirty) $\to$ fetch page from swap disk $\to$ update page table and TLB $\to$ restore state and **restart the faulting instruction**.
- **Effective Memory Access Time (EMAT):** $\text{EMAT} = (1 - p) \times t_m + p \times T_{\text{fault}}$, where $p$ is page fault rate, $t_m$ is RAM access time, and $T_{\text{fault}}$ is page-fault service time (typically milliseconds).
- **TLB EMAT:** With serial TLB lookup and an $L$-level page table: $\text{EMAT} = h(t_{\text{tlb}} + t_m) + (1 - h)(t_{\text{tlb}} + (L + 1)t_m)$.
- **FIFO (First-In, First-Out):** Replaces the oldest resident page. Can exhibit **Belady's anomaly** (allocating more frames causes *more* page faults).
- **Optimal (OPT / MIN):** Replaces the page not referenced for the longest duration into the future. Lowest possible page-fault rate; theoretical benchmark (unrealisable online).
- **LRU (Least Recently Used):** Replaces the page not referenced for the longest time in the past. It is a **stack algorithm** and never suffers from Belady's anomaly.
- **Clock (Second-Chance) algorithm:** Approximates LRU using a circular buffer and a hardware reference bit ($R$). If $R = 1$, set $R = 0$ and advance; if $R = 0$, replace.
- **Thrashing:** Occurs when total memory demand across processes exceeds available physical frames ($\sum WSS_i > \text{Total Frames}$). Processes spend more time paging than executing; CPU utilisation collapses to near zero.

## 1. Demand paging and page-fault service routine [PYQ: High]

Pure demand paging executes processes without pre-loading any pages: a page is brought into RAM only when the CPU references an address within that page.

### 1.1 The Valid-Invalid bit

Every Page Table Entry (PTE) contains a hardware **Valid / Invalid ($V/I$) bit**:
- $V = 1$: The page is currently resident in physical RAM at the specified frame.
- $I = 0$: The page is either outside the process's logical address space (illegal memory reference $\implies$ segmentation violation) or valid but currently stored on the backing store / swap disk.

### 1.2 Step-by-step page-fault service routine

When the CPU issues a logical address whose page table entry has $V = 0$, the MMU generates a hardware exception: a **page fault trap**.

```text
CPU ---> Generates address ---> MMU checks Page Table
                                      |
                         Valid bit = 0? (Page Fault)
                                      |
                                      v
1. Trap to kernel  -------------> OS Page Fault Handler
                                      |
2. Verify reference (Legal?) -------> If illegal: abort process
                                      | Legal
                                      v
3. Find free frame  ------------> Is frame available?
                                      | No: select victim via replacement algorithm
                                      |     If victim is dirty (M=1): write to disk!
                                      v
4. Schedule Disk I/O -----------> Read demanded page into frame
                                      | (Process put in Wait state; CPU yields)
                                      v
5. Disk Interrupt  -------------> I/O complete
                                      |
6. Update Page Table -----------> Set frame number, Valid = 1, Dirty = 0
                                      |
7. Restart Instruction ---------> CPU re-executes original instruction
```

**Instruction Restart:** The faulting instruction must be restarted from scratch. The architecture must guarantee idempotency or restore modified registers (crucial for auto-increment / auto-decrement addressing modes, e.g., in `MOV (R1)+, (R2)+`).

### 1.3 Effective Memory Access Time (EMAT) with page faults

Let $p$ be the page fault rate ($0 \le p \le 1$), $t_m$ be normal memory access time, and $T_{\text{fault}}$ be the page fault service time:

$$\text{EMAT} = (1 - p) \times t_m + p \times T_{\text{fault}}$$

If a fraction $\beta$ of replaced pages are dirty (modified, requiring swap-out):

$$T_{\text{fault}} = (1 - \beta) \times T_{\text{clean}} + \beta \times T_{\text{dirty}}$$

**Worked example 1.1 (Maximum tolerable page fault rate).** A computer system has a memory access time $t_m = 100\text{ ns}$. Page fault service time is $8\text{ ms}$ if the victim is clean and $20\text{ ms}$ if the victim is dirty. On average, $70\%$ of victim pages are dirty. What is the maximum page fault rate $p$ to ensure that effective access time does not exceed $200\text{ ns}$ (at most $100\%$ slowdown)?

*Solution:*
1. Calculate average page-fault service time:
   $$T_{\text{fault}} = 0.3 \times (8\text{ ms}) + 0.7 \times (20\text{ ms}) = 2.4 + 14.0 = 16.4\text{ ms} = 16{,}400{,}000\text{ ns}$$
2. Set up the EMAT inequality:
   $$\text{EMAT} = (1 - p) \times 100\text{ ns} + p \times 16{,}400{,}000\text{ ns} \le 200\text{ ns}$$
   $$100 - 100p + 16{,}400{,}000 p \le 200$$
   $$16{,}399{,}900 p \le 100 \implies p \le \frac{100}{16{,}399{,}900} \approx 6.0976 \times 10^{-6}$$
At most $1$ page fault per $164{,}000$ memory references is tolerated.

## 2. Translation Lookaside Buffer (TLB) and EMAT [PYQ: High]

To eliminate the extra memory lookup per memory reference, the MMU uses an on-chip, high-speed associative hardware cache called the **Translation Lookaside Buffer (TLB)**.

```text
CPU ---> Logical Address (p, d)
                |
                +------------------> TLB Lookup (t_tlb)
                                          |
                        +-----------------+-----------------+
                        | Hit (ratio h)                     | Miss (ratio 1 - h)
                        v                                   v
             Extract Frame f from TLB             Access Page Table in RAM
                        |                         (k memory accesses for k levels)
                        |                                   |
                        +-----------------+-----------------+
                                          |
                                          v
                              Physical Address: (f, d)
                                          |
                                          v
                               Access Data in RAM (t_m)
```

### 2.1 EMAT formulas

Let $h$ be the TLB hit ratio, $t_{\text{tlb}}$ be the TLB access time, $t_m$ be physical memory access time, and $L$ be the number of page table levels:

**Sequential (Serial) search:** TLB is searched first; memory is accessed subsequently.
- On TLB hit: access TLB ($t_{\text{tlb}}$) $+$ access data in RAM ($t_m$) $= t_{\text{tlb}} + t_m$.
- On TLB miss ($1$-level): access TLB ($t_{\text{tlb}}$) $+$ read PTE in RAM ($t_m$) $+$ read data in RAM ($t_m$) $= t_{\text{tlb}} + 2t_m$.
- General formula for $L$-level page table:
  $$\text{EMAT} = h \times (t_{\text{tlb}} + t_m) + (1 - h) \times (t_{\text{tlb}} + (L + 1)t_m)$$
  $$\text{EMAT} = t_{\text{tlb}} + t_m + (1 - h) \times L \times t_m$$

**Simultaneous (Parallel) search:** Hardware queries TLB and page table/cache concurrently.
$$\text{EMAT} = h \times \max(t_{\text{tlb}}, t_m) + (1 - h) \times (t_{\text{tlb}} + (L + 1)t_m)$$

**Worked example 2.1 (Two-level page table EMAT).** A system uses a 2-level page table. TLB access time $t_{\text{tlb}} = 20\text{ ns}$, memory access time $t_m = 100\text{ ns}$, and TLB hit ratio $h = 90\%$. Assume sequential lookup.
1. On TLB hit: $T_{\text{hit}} = t_{\text{tlb}} + t_m = 20 + 100 = 120\text{ ns}$.
2. On TLB miss: Needs Level 1 lookup, Level 2 lookup, and final data access (3 memory accesses):
   $$T_{\text{miss}} = t_{\text{tlb}} + 3 \times t_m = 20 + 3(100) = 320\text{ ns}$$
3. $\text{EMAT} = 0.90 \times 120 + 0.10 \times 320 = 108 + 32 = 140\text{ ns}$.

## 3. Page replacement algorithms [PYQ: High]

When a page fault occurs and all physical frames are occupied, the OS must choose a **victim page** to evict.

### 3.1 FIFO (First-In, First-Out)
Replaces the page that was brought into memory earliest (tracked via a FIFO queue of frames).
- Simple to implement; does not use page access frequency or recency.
- Susceptible to **Belady's anomaly**.

### 3.2 Optimal Page Replacement (OPT / MIN)
Replaces the page that will **not be used for the longest period of time in the future**.
- Produces the absolute minimum number of page faults for any fixed number of frames.
- Cannot be implemented in a general-purpose OS because future references are unknown. Used as a benchmark.

### 3.3 LRU (Least Recently Used)
Replaces the page that has not been referenced for the longest period of time in the past.
- Approximates OPT by using past behaviour as a predictor of future references.
- Implementation requires either a hardware **counter / logical clock** on each access or a **doubly-linked stack** (where referenced page is moved to the top). Expensive in hardware.

### 3.4 Second-Chance (Clock) Algorithm
Approximates LRU with minimal hardware overhead using a circular queue and a single **reference bit ($R$)** per frame:
1. The pointer traverses the circular frame buffer.
2. If $R = 0$: select this page as the victim, advance pointer, and terminate.
3. If $R = 1$: clear bit to $R = 0$ (gives the page a "second chance"), advance pointer, and repeat search.

**Worked example 3.1 (Page replacement comparison).** Reference string: `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1` with **$3$ physical frames**, initially empty. Trace FIFO, LRU, and OPT.

*Trace summary across all 20 references:*

| Algorithm | Page Faults | Page Hits | Fault Ratio |
|---|---|---|---|
| **FIFO** | **$15$ faults** | $5$ hits | $15 / 20 = 75\%$ |
| **LRU** | **$12$ faults** | $8$ hits | $12 / 20 = 60\%$ |
| **OPT** | **$9$ faults** | $11$ hits | $9 / 20 = 45\%$ |

*Step-by-step trace of first 8 references:*
- `7`: Fault $\to$ Frame state `[7, -, -]`
- `0`: Fault $\to$ Frame state `[7, 0, -]`
- `1`: Fault $\to$ Frame state `[7, 0, 1]`
- `2`: Fault $\to$
  - FIFO evicts oldest (`7`): `[2, 0, 1]`
  - LRU evicts least recently used (`7`): `[2, 0, 1]`
  - OPT looks ahead (`0` used at pos 5, `1` used at pos 14, `7` used at pos 18): evicts `7`: `[2, 0, 1]`
- `0`: Hit $\to$ all algorithms retain `[2, 0, 1]`. For LRU, `0` becomes most recent.
- `3`: Fault $\to$
  - FIFO evicts oldest resident (`0`): `[2, 3, 1]`
  - LRU evicts least recently used (`1`, since `0` and `2` were used more recently): `[2, 0, 3]`
  - OPT looks ahead (`0` at pos 7, `2` at pos 9, `1` at pos 14): evicts `1`: `[2, 0, 3]`

## 4. Belady's anomaly and stack algorithms [PYQ: High]

### 4.1 Belady's Anomaly

**Belady's Anomaly** is the counter-intuitive phenomenon where **increasing the number of page frames allocated to a process results in an increased number of page faults** for the exact same reference string.

**Canonical Reference String:** `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5` under FIFO replacement.

```text
With 3 frames:
Ref:    1  2  3  4  1  2  5  1  2  3  4  5
F1:     1  1  1  4  4  4  5  5  5  5  5  5
F2:        2  2  2  1  1  1  1  1  3  3  3
F3:           3  3  3  2  2  2  2  2  4  4
Fault:  *  *  *  *  *  *  *        *  *      ===> Total: 9 faults

With 4 frames:
Ref:    1  2  3  4  1  2  5  1  2  3  4  5
F1:     1  1  1  1  1  1  5  5  5  5  4  4
F2:        2  2  2  2  2  2  1  1  1  1  5
F3:           3  3  3  3  3  3  2  2  2  2
F4:              4  4  4  4  4  4  3  3  3
Fault:  *  *  *  *        *  *  *  *  *  *   ===> Total: 10 faults!
```

Adding one additional frame increased page faults from **9 to 10**!

### 4.2 Stack algorithms

A page replacement algorithm is a **stack algorithm** if the set of pages resident in an $m$-frame memory is **always a strict subset** of the pages resident in an $(m + 1)$-frame memory for any reference string at any time $t$:

$$B(t, m) \subseteq B(t, m + 1)$$

- **Theorem:** A stack algorithm **never** suffers from Belady's anomaly.
- **LRU and OPT are stack algorithms:** Their resident pages can be modelled as a stack of pages ordered by past recency (LRU) or future distance (OPT). Adding frames merely exposes lower elements of the stack.
- **FIFO is NOT a stack algorithm:** Frame residence depends on arrival order, violating the subset inclusion property.

## 5. Frame allocation, working set model, and thrashing [PYQ: Medium]

### 5.1 Frame allocation policies

- **Minimum frame count:** Determined by instruction-set architecture (an instruction must hold frames for opcode, operands, and indirect pointers simultaneously).
- **Allocation schemes:** Equal allocation ($m / n$ frames per process) vs. **Proportional allocation**:
  $$a_i = \frac{s_i}{\sum_{j=1}^n s_j} \times m \quad (s_i = \text{virtual size of process } P_i)$$
- **Local vs. Global replacement:**
  - Local replacement: A process evicts only from its own allocated frames. Prevents cascading thrashing across processes.
  - Global replacement: A process can steal frames from any process in RAM. Higher system throughput, but one thrashing process can drag down the entire system.

### 5.2 Thrashing

**Thrashing** occurs when a process spends more time swapping pages in and out of secondary storage than executing CPU instructions.

```text
High multiprogramming ---> Insufficient frames per process ---> Frequent page faults
           ^                                                                |
           |                  Low CPU utilisation <-------------------------+
           |                            |
           +-- OS increases concurrency + (vicious cycle!)
```

**Root cause:** The sum of the locality sizes of all active processes exceeds available physical memory frames: $\sum \text{Locality}_i > \text{Total Frames}$.

### 5.3 Working Set Model (Denning)

The **working set model** is based on the principle of locality.
- **Working set window ($\Delta$):** A fixed parameter measuring a specific number of page references.
- **Working Set ($WSS_i(t)$):** The set of distinct pages referenced by process $P_i$ during the time interval $[t - \Delta + 1, t]$.
- Total memory demand: $D = \sum_{i=1}^n |WSS_i|$.
- **Thrashing prevention rule:**
  - If $D \le \text{Total Frames } M$: allocate frames matching each process's working set.
  - If $D > M$: the OS must suspend (swap out) one or more processes to free frames, preventing thrashing.

**Worked example 5.1 (Working set size trace).** Consider page reference string:
`2, 6, 1, 5, 7, 7, 7, 7, 5, 1, 6, 2, 3, 4, 1, 2, 3, 4, 4, 4` with working set window $\Delta = 5$.
1. At $t = 5$ (references $2, 6, 1, 5, 7$): Distinct pages $\{1, 2, 5, 6, 7\} \implies WSS(5) = 5$.
2. At $t = 8$ (references $7, 7, 7, 7, 5$): Distinct pages $\{5, 7\} \implies WSS(8) = 2$.
3. At $t = 12$ (references $7, 5, 1, 6, 2$): Distinct pages $\{1, 2, 5, 6, 7\} \implies WSS(12) = 5$.

### 5.4 Page Fault Frequency (PFF)

An alternative dynamic algorithm that controls thrashing by directly monitoring the page-fault rate:
- If page-fault rate $> \text{Upper threshold}$: Process needs more frames $\implies$ allocate additional frames.
- If page-fault rate $< \text{Lower threshold}$: Process has surplus frames $\implies$ reclaim excess frames.

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| **Demand Paging EMAT** | $\text{EMAT} = (1 - p)t_m + p \times T_{\text{fault}}$ | Quantifying slowdown caused by page faults |
| **Fault penalty with dirty pages** | $T_{\text{fault}} = (1 - \beta)T_{\text{clean}} + \beta T_{\text{dirty}}$ | Factoring in dirty page write-back overhead |
| **TLB EMAT (Sequential, 1-level)** | $\text{EMAT} = h(t_{\text{tlb}} + t_m) + (1 - h)(t_{\text{tlb}} + 2t_m)$ | Effective memory access time with TLB |
| **TLB EMAT ($L$-level hierarchical)** | $\text{EMAT} = t_{\text{tlb}} + t_m + (1 - h) \cdot L \cdot t_m$ | Computing EMAT with multilevel page tables |
| **Stack algorithm property** | $B(t, m) \subseteq B(t, m + 1)$ | Proving absence of Belady's anomaly |
| **Proportional frame allocation** | $a_i = \frac{s_i}{\sum s_j} \times m$ | Distributing frames according to process size |
| **Thrashing condition** | $\sum \vert WSS_i\vert > \text{Total Physical Frames } M$ | Identifying memory overload threshold |
| **Instruction restart** | Hardware must restore registers and PC | Handling page faults mid-instruction |

## GATE traps

- **TLB miss without a page fault:** A TLB miss merely requires reading the page table in RAM (cost: tens/hundreds of nanoseconds). A page fault requires reading from disk (cost: milliseconds). **Never confuse a TLB miss with a page fault.**
- **Belady's anomaly occurs only in non-stack algorithms:** FIFO and Random replacement can exhibit Belady's anomaly. **LRU and OPT never exhibit Belady's anomaly.**
- **FIFO order is arrival order, not frequency or recency:** Re-referencing a page already in memory does NOT reset its position in the FIFO queue. In contrast, it DOES reset its recency in LRU.
- **Dirty bit write-back:** A clean page is discarded on eviction with zero write-back cost; a dirty page must be written to disk, virtually doubling the page-fault service time.
- **Sequential vs. Simultaneous TLB lookup:** Check if the question states that TLB and memory/cache are searched in parallel or serially. Serial lookup adds $t_{\text{tlb}}$ to the hit path.

## Connections

- [Memory management](memory-management.md) — Supplies the underlying page tables, frame allocations, and address translation logic.
- [Computer architecture cache](../10-computer-organization/memory-hierarchy-and-cache.md) — TLB design parallels hardware CPU cache design (set-associative lookup, tags, hit rates).
- [CPU scheduling](cpu-scheduling.md) — Thrashing collapses CPU utilisation; naive OS schedulers exacerbate thrashing by introducing more processes to the ready queue.
- [File systems and disk scheduling](file-systems-and-disk-scheduling.md) — Page fault swap I/O operations are serviced by disk controllers and swap partitions.

## Practice

**Q1. (MCQ)** Which of the following page replacement algorithms CANNOT exhibit Belady's anomaly under any circumstances?
- (A) FIFO (First-In, First-Out)
- (B) Second-Chance (Clock)
- (C) LRU (Least Recently Used)
- (D) Random replacement

<details><summary>Answer</summary>

**Answer:** (C)

**Solution:**
LRU is a stack algorithm satisfying $B(t, m) \subseteq B(t, m + 1)$. The set of pages in $m$ frames is always a subset of the pages in $m + 1$ frames, mathematically precluding Belady's anomaly. FIFO and Clock do not satisfy the stack property and can suffer from Belady's anomaly.
</details>

**Q2. (NAT)** Consider a paging system with a 2-level page table. TLB lookup time is $15\text{ ns}$ and main memory access time is $120\text{ ns}$. The TLB hit ratio is $95\%$. Assuming sequential lookup and no page faults, calculate the Effective Memory Access Time (EMAT) in nanoseconds.

<details><summary>Answer</summary>

**Answer:** $147\text{ ns}$

**Solution:**
1. On TLB hit: $T_{\text{hit}} = t_{\text{tlb}} + t_m = 15 + 120 = 135\text{ ns}$.
2. On TLB miss: With a 2-level page table, we need $2$ memory accesses to read the page table entries plus $1$ memory access to read the data:
   $$T_{\text{miss}} = t_{\text{tlb}} + 3 \times t_m = 15 + 3(120) = 375\text{ ns}$$
3. $\text{EMAT} = 0.95 \times 135 + 0.05 \times 375 = 128.25 + 18.75 = 147\text{ ns}$.
</details>

**Q3. (NAT)** A virtual memory system uses the Optimal (OPT) page replacement policy. Physical memory has $3$ initially empty frames. The page reference string is: `2, 3, 2, 1, 5, 2, 4, 5, 3, 2, 5, 2`. Calculate the total number of page faults that occur.

<details><summary>Answer</summary>

**Answer:** $6$

**Solution:**
- `2`: Fault $\to [2]$
- `3`: Fault $\to [2, 3]$
- `2`: Hit $\to [2, 3]$
- `1`: Fault $\to [2, 3, 1]$
- `5`: Fault $\to$ Next uses: `2` at pos 6, `1` never, `3` at pos 9. Evict `1`: $[2, 3, 5]$
- `2`: Hit $\to [2, 3, 5]$
- `4`: Fault $\to$ Next uses: `5` at pos 8, `3` at pos 9, `2` at pos 10. Evict `2` (used latest): $[4, 3, 5]$
- `5`: Hit $\to [4, 3, 5]$
- `3`: Hit $\to [4, 3, 5]$
- `2`: Fault $\to$ Next uses: `5` at pos 11, `4` never, `3` never. Evict `4` (or `3`): $[2, 3, 5]$
- `5`: Hit $\to [2, 3, 5]$
- `2`: Hit $\to [2, 3, 5]$
Total page faults $= 6$.
</details>

**Q4. (NAT)** Consider the reference string: `1, 2, 3, 4, 2, 1, 5, 6, 2, 1, 2, 3, 7, 6, 3, 2, 1, 2, 3, 6` with $4$ frames initially empty. How many page faults occur under the LRU page replacement algorithm?

<details><summary>Answer</summary>

**Answer:** $10$

**Solution:**
Trace with 4 frames:
- `1`: Fault $\to [1]$
- `2`: Fault $\to [1, 2]$
- `3`: Fault $\to [1, 2, 3]$
- `4`: Fault $\to [1, 2, 3, 4]$ (4 faults so far)
- `2`: Hit $\to$ order: $1, 3, 4, 2$
- `1`: Hit $\to$ order: $3, 4, 2, 1$
- `5`: Fault $\to$ evicts $3 \to [4, 2, 1, 5]$ (5th fault)
- `6`: Fault $\to$ evicts $4 \to [2, 1, 5, 6]$ (6th fault)
- `2`: Hit $\to$ order: $1, 5, 6, 2$
- `1`: Hit $\to$ order: $5, 6, 2, 1$
- `2`: Hit $\to$ order: $5, 6, 1, 2$
- `3`: Fault $\to$ evicts $5 \to [6, 1, 2, 3]$ (7th fault)
- `7`: Fault $\to$ evicts $6 \to [1, 2, 3, 7]$ (8th fault)
- `6`: Fault $\to$ evicts $1 \to [2, 3, 7, 6]$ (9th fault)
- `3`: Hit $\to$ order: $2, 7, 6, 3$
- `2`: Hit $\to$ order: $7, 6, 3, 2$
- `1`: Fault $\to$ evicts $7 \to [6, 3, 2, 1]$ (10th fault)
- `2`: Hit $\to$ order: $6, 3, 1, 2$
- `3`: Hit $\to$ order: $6, 1, 2, 3$
- `6`: Hit $\to$ order: $1, 2, 3, 6$
Total page faults $= 10$.
</details>

**Q5. (MSQ)** Which of the following statements is/are TRUE regarding virtual memory and demand paging?
- (A) The working set of a process is always a subset of its resident set.
- (B) If a victim page has its dirty bit cleared ($M = 0$), it does not need to be written back to the disk.
- (C) Optimal page replacement is impossible to implement in practice because it requires knowledge of future memory references.
- (D) Thrashing causes CPU utilisation to drop sharply because ready processes spend most of their time waiting for page-fault I/O.

<details><summary>Answer</summary>

**Answer:** (B), (C), (D)

**Solution:**
- (A) is FALSE: The working set is the set of pages referenced in the last $\Delta$ references. If some of those pages have been paged out, the working set is not necessarily a subset of the resident set.
- (B) is TRUE: Clean pages ($M = 0$) are already identical to the copy on backing store.
- (C) is TRUE: OPT requires future knowledge of reference strings.
- (D) is TRUE: As processes page-fault continuously, the CPU sits idle waiting for disk transfers, causing utilisation to collapse.
</details>

**Q6. (NAT)** In a demand-paged system, memory access time is $200\text{ ns}$. Servicing a page fault takes $10\text{ ms}$. If we want the Effective Memory Access Time to be within $10\%$ of normal memory access time ($\le 220\text{ ns}$), what is the maximum page fault rate $p$ as a multiple of $10^{-6}$? Round to two decimal places.

<details><summary>Answer</summary>

**Answer:** $2.00$

**Solution:**
1. Target: $\text{EMAT} \le 220\text{ ns}$.
2. Normal access time $t_m = 200\text{ ns}$.
3. Page fault service time $T_{\text{fault}} = 10\text{ ms} = 10{,}000{,}000\text{ ns}$.
4. Formula:
   $$(1 - p)(200) + p(10{,}000{,}000) \le 220$$
   $$200 - 200p + 10{,}000{,}000p \le 220$$
   $$9{,}999{,}800 p \le 20 \implies p \le \frac{20}{9{,}999{,}800} \approx 2.00004 \times 10^{-6}$$
As a multiple of $10^{-6}$, $p = 2.00$.
</details>

**Q7. (MCQ)** Consider a system where the working set window $\Delta = 4$ references. Given the page reference string: `1, 2, 3, 2, 4, 1, 5, 2, 3, 4`. What is the size of the working set at time $t = 7$ (after referencing page `5`)?
- (A) 2
- (B) 3
- (C) 4
- (D) 5

<details><summary>Answer</summary>

**Answer:** (C)

**Solution:**
1. At time $t = 7$, the window covers the last $\Delta = 4$ references: references at $t = 4, 5, 6, 7$.
2. The references in this window are:
   - $t = 4$: `2`
   - $t = 5$: `4`
   - $t = 6$: `1`
   - $t = 7$: `5`
3. The distinct set of pages referenced is $\{2, 4, 1, 5\}$.
4. Number of distinct pages in working set $= 4$.
</details>
