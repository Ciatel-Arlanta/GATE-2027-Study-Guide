# Memory Hierarchy, Interfacing and Cache

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Memory interfacing; Memory hierarchy and performance; Cache memory mapping
> **Prerequisites:** [Combinational circuits](../09-digital-logic/combinational-circuits.md) (decoders), [Instruction sets and addressing](instruction-sets-and-addressing.md) · **Leads to:** [I/O, interrupts and DMA](io-interrupts-dma.md), [Pipelining](pipelining.md), [Virtual memory](../13-operating-systems/README.md)

## Quick glance

- **Hierarchy:** registers $\to$ L1 $\to$ L2 $\to$ L3 $\to$ main memory $\to$ disk: faster = smaller and costlier. It works because of **locality**: temporal (reuse) and spatial (neighbours).
- **Chip count** for $N\times M$ memory from $k\times m$ chips $=\dfrac{N}{k}\cdot\dfrac{M}{m}$. Address lines $=\log_2N$; the top $\log_2(N/k)$ lines feed a decoder for chip select.
- **AMAT (hierarchical access):** $t_{avg}=h\,t_1+(1-h)(t_1+t_2)$. **Simultaneous access:** $t_{avg}=h\,t_1+(1-h)t_2$. Multi-level: $t_1+m_1(t_2+m_2\,t_3)$ with local miss rates.
- **Address split** for cache: **tag | index | block offset**. Offset $=\log_2(\text{block size})$; index $=\log_2(\#\text{sets})$; tag $=$ rest. #sets $=\dfrac{\text{cache size}}{\text{block size}\times\text{associativity}}$.
- Direct mapped: 1 line per index, 1 comparator. Fully associative: no index, tag $=$ address $-$ offset, one comparator per line. $k$-way: $k$ comparators.
- **Write-through:** every write goes to memory (use a write buffer); **write-back:** dirty bit, memory updated on eviction. Write-allocate usually pairs with write-back; no-write-allocate with write-through.
- Misses: **compulsory** (first reference), **capacity** (cache too small), **conflict** (set too small; absent in fully associative).
- Tag memory $=\#\text{lines}\times(\text{tag}+\text{valid}+\text{dirty bits})$.
- #1 trap: the **tag/index/offset** split uses the block size, number of *sets* (not lines) and **physical vs virtual** address width; also hierarchical vs simultaneous AMAT.

## 1. Memory interfacing

### Chip organisation arithmetic
A chip is $k\times m$: $k$ words of $m$ bits. To build $N\times M$:
- **Width expansion:** $M/m$ chips side by side share address lines.
- **Depth expansion:** $N/k$ banks, selected by a decoder on the extra address bits.
- **Total chips** $=(N/k)(M/m)$.

**Worked example 1.** Build $64K\times16$ from $16K\times8$ chips.
- Chips $=(64K/16K)\times(16/8)=4\times2=8$.
- Address lines $=16$ ($64K=2^{16}$); each chip needs 14 address lines ($16K$); the 2 upper lines go to a **2-to-4 decoder** generating the 4 chip-select signals; each select enables a pair of chips (one per byte).

**Worked example 2.** $1M\times8$ from $128K\times4$: chips $=(1M/128K)(8/4)=8\times2=16$. Address lines 20; chip uses 17; upper 3 lines feed a 3-to-8 decoder.

**Worked example 3 (address map).** 4 chips of $16K\times8$ forming a $64K\times8$ memory (decoder on $A_{15},A_{14}$):

| Chip | $A_{15}A_{14}$ | Address range (hex) |
|---|---|---|
| 0 | 00 | 0000–3FFF |
| 1 | 01 | 4000–7FFF |
| 2 | 10 | 8000–BFFF |
| 3 | 11 | C000–FFFF |

**Partial decoding** (leaving some high address lines out of the chip-select logic) causes **aliasing** (foldback): the same chip responds to several address ranges. For example, if only $A_{15}$ is decoded to select between two chips in a 64K address space, each chip occupies a 32K range; if $A_{14}$ is ignored where fewer chips exist, the chip also appears at the mirror addresses.

**Number of address lines** for a memory of $W$ bytes $=\log_2W$; data lines $=$ word width. A $4K\times1$ DRAM may be multiplexed: $\log_2$ lines are time-shared as row and column addresses (RAS/CAS), halving the address pins.

### Memory-mapped vs isolated I/O
- **Memory-mapped I/O:** device registers occupy part of the memory address space; use ordinary LOAD/STORE; no special instructions; reduces available memory addresses.
- **Isolated (port-mapped) I/O:** separate I/O address space with special IN/OUT instructions and a control line (IO/$\overline M$); full memory space remains, but needs extra instructions.
See [I/O, interrupts and DMA](io-interrupts-dma.md).

### Types of memory
SRAM: 6 transistors/bit, fast, no refresh (cache). DRAM: 1 transistor + capacitor, must refresh, dense (main memory). ROM/PROM/EPROM/EEPROM/Flash: non-volatile. Access time = latency from request to data; cycle time $\ge$ access time (DRAM precharge). **Interleaving:** with $b$ banks and low-order interleaving (consecutive words in consecutive banks), sequential words are fetched from different banks in an overlapped manner. If a bank needs time $T$ per access and the bus moves one word per time $t$, a stream of sequential words is delivered every $t$ as long as $T\le b\,t$. Example: $T=40$ ns, $t=10$ ns needs $b\ge4$ banks.

## 2. Memory hierarchy and performance

**Locality of reference:** programs reuse recently used items (**temporal**: loop variables) and items near recently used ones (**spatial**: array scans). Caches exploit temporal locality by keeping data and spatial locality by fetching whole blocks.

| Level | Typical size | Typical access time |
|---|---|---|
| Registers | ~1 KB | < 1 ns |
| L1 cache | 32–64 KB | 1–2 ns |
| L2/L3 cache | 256 KB–32 MB | 5–30 ns |
| Main memory (DRAM) | GBs | 50–100 ns |
| Disk (SSD/HDD) | TBs | $10^5$–$10^7$ ns |

**Hit ratio** $h$ = fraction of references found in this level; miss ratio $=1-h$.

### Average memory access time (AMAT)
Two-level, access times $t_1$ (cache) and $t_2$ (memory).

1. **Hierarchical (sequential) access** (check cache first, then memory): a miss pays both times.
 $$t_{avg}=h\,t_1+(1-h)(t_1+t_2)=t_1+(1-h)t_2.$$
2. **Simultaneous access** (cache and memory started together; memory access aborted on a hit): $t_{avg}=h\,t_1+(1-h)t_2$.

**Worked example 4.** $t_1=2$ ns, $t_2=20$ ns, $h=0.9$:
- Hierarchical: $0.9\times2+0.1\times(2+20)=1.8+2.2=4.0$ ns.
- Simultaneous: $0.9\times2+0.1\times20=3.8$ ns.

**Multi-level (hierarchical):** $t_{avg}=t_{L1}+m_1\,(t_{L2}+m_2\,t_{mem})$ with **local** miss rates $m_1,m_2$ (misses of that level divided by accesses to that level).

**Worked example 5.** $t_{L1}=1$ ns with $h_1=0.8$; $t_{L2}=10$ ns with local hit ratio $0.9$; memory 100 ns. $t_{avg}=1+0.2\,(10+0.1\times100)=1+0.2\times20=5$ ns. Global view: 80% hit L1, 18% hit L2, 2% go to memory: $0.8(1)+0.18(1+10)+0.02(1+10+100)=0.8+1.98+2.22=5.0$ ns ✓.

**Finding the hit ratio:** if $t_{avg}=5$ ns, $t_1=2$, $t_2=20$ (hierarchical): $2+(1-h)\cdot20=5\Rightarrow h=0.85$.

**CPI with memory stalls.** $\text{CPI}=\text{CPI}_{base}+(\text{memory refs per instruction})\times(\text{miss rate})\times(\text{miss penalty in cycles})$. With base CPI 1, 1.3 references/instruction (1 fetch + 0.3 data), miss rate 2%, penalty 100 cycles: CPI $=1+1.3\times0.02\times100=3.6$.

### Write policies
| Policy | On write hit | On write miss | Notes |
|---|---|---|---|
| **Write-through** | update cache and memory | usually no-write-allocate (write straight to memory) | simple, memory always consistent; **write buffer** hides latency |
| **Write-back** | update cache only, set **dirty** bit | usually write-allocate (fetch block, then write) | fewer memory writes; evicting a dirty block costs a write-back |

**Effective miss penalty with write-back:** if a fraction $d$ of replaced blocks is dirty, penalty $=t_{fetch}+d\cdot t_{writeback}$.

**Worked example 6 (write traffic).** 1000 references: 75% reads, 25% writes. Read hit ratio 0.9, write hit ratio 0.8. Write-through: each of the 250 writes goes to memory: 250 memory writes plus read-miss fetches $=750\times0.1=75$ block fetches (plus write-miss fetches if write-allocate). Write-back: memory is written only for dirty evictions, far fewer than 250.

### Secondary storage (as needed for AMAT)
**Disk access time** $=\text{seek}+\text{rotational latency}+\text{transfer}$. Average rotational latency $=\frac12\cdot\frac{60}{\text{rpm}}$. **Worked example 7:** 7200 rpm $\Rightarrow$ 8.33 ms/rev, average latency 4.17 ms; seek 5 ms; read 4 KB from a 400 KB track: transfer $=4/400\times8.33=0.083$ ms. Total $=5+4.17+0.08=9.25$ ms.

**Effective access time with virtual memory** (page-fault rate $p$): $t=(1-p)\,t_{mem}+p\,t_{fault}$. With $t_{mem}=100$ ns, $t_{fault}=8$ ms, $p=10^{-6}$: $t=99.9999+8.0=108.0$ ns. See [Virtual memory](../13-operating-systems/README.md).

## 3. Cache organisation and address mapping

A **cache line (block)** holds $B$ bytes plus a **tag**, **valid** bit (and **dirty** bit for write-back). Memory is divided into blocks of $B$ bytes; the block number $=\lfloor\text{address}/B\rfloor$.

### Address breakdown
$$\text{address} = \underbrace{\text{tag}}_{\text{remaining}}\;|\;\underbrace{\text{index}}_{\log_2(\text{sets})}\;|\;\underbrace{\text{offset}}_{\log_2B}$$

| Mapping | Rule | Index bits | Tag bits | Comparators |
|---|---|---|---|---|
| **Direct** | block $b\to$ line $b\bmod L$ | $\log_2L$ | $A-\text{index}-\text{offset}$ | 1 |
| **Fully associative** | block can go in any line | 0 | $A-\text{offset}$ | $L$ (parallel) |
| **$k$-way set associative** | block $b\to$ set $b\bmod S$, any of $k$ lines | $\log_2S$ | $A-\text{index}-\text{offset}$ | $k$ |

where $A$ = address bits, $L=\dfrac{C}{B}$ lines, $S=\dfrac{L}{k}=\dfrac{C}{Bk}$ sets, $C$ = cache capacity. Direct mapped is $k=1$; fully associative is $k=L$ ($S=1$).

**Worked example 8.** 32-bit address, 32 KB cache, 64 B blocks. $L=32K/64=512$ lines.
- Offset $=\log_264=6$.
- Direct: index $=9$, tag $=32-9-6=17$.
- 4-way: sets $=128$, index $=7$, tag $=19$.
- Fully associative: tag $=26$.
- **Tag memory (direct)** $=512\times(17+1\text{ valid})=9216$ bits $=1152$ bytes; with a dirty bit $512\times19=9728$ bits. 4-way: $512\times(19+1)=10240$ bits. Fully assoc: $512\times27=13824$ bits. More associativity $\Rightarrow$ more tag bits and comparators.
- **Data store** $=32$ KB $=262144$ bits; total including tags is a few percent larger.

**Worked example 9.** 24-bit physical address (16 MB main memory), 64 KB cache, 16 B blocks, 4-way.
- Lines $=64K/16=4096$; sets $=4096/4=1024$.
- Offset $=4$, index $=10$, tag $=24-14=10$.
- Block number in memory has $24-4=20$ bits: $2^{20}$ blocks; each set receives $2^{20}/2^{10}=1024$ different blocks competing for 4 lines.

**Worked example 10 (which set?).** Same cache as Example 9. Address $\text{0x12345A}=1193050$, binary `0001001000 1101000101 1010` (tag | index | offset). Offset $=1010_2=10$. Block number $=\lfloor1193050/16\rfloor=74565$; set $=74565\bmod1024=837$ (index $1101000101_2$); tag $=\lfloor74565/1024\rfloor=72$ ($0001001000_2$). (Python-checked.)

### Simulating hits and misses

Block-address sequence: $0,8,0,6,8$ with a 4-line cache (LRU).

| Mapping | Placement | Outcomes | Misses |
|---|---|---|---|
| Direct (4 lines, $b\bmod4$) | 0 and 8 both map to line 0; 6 to line 2 | M M M M M | **5** |
| 2-way (2 sets, $b\bmod2$) | 0, 8, 6 all in set 0 (2 ways) | M M H M M | **4** |
| Fully associative (LRU) | all four fit | M M H M H | **3** |

Trace of the 2-way: set 0 $=[0]$; $[0,8]$; hit on 0 $\Rightarrow$ LRU order $[8,0]$; 6 evicts LRU 8 $\Rightarrow[0,6]$; 8 evicts LRU 0 $\Rightarrow[6,8]$ (miss). Fully associative: after 0,8,6 the cache holds $\{0,8,6\}$; the last 8 hits.

**Associativity is not always better:** with byte addresses $0,4,16,32,48,64,0,16,80,64,32$, block size 16 B, 4 lines LRU: direct $=8$ misses, 2-way $=8$, fully associative $=9$ (verified by simulation); LRU can thrash on cyclic patterns slightly larger than the cache. More associativity usually reduces conflict misses, but on a particular trace it is not guaranteed to win.

### Replacement policies
- **LRU:** evict the least recently used line (needs age bits: $\log_2k$ per line or $k!$ states per set).
- **FIFO:** evict the oldest loaded; **random**; pseudo-LRU.
- **Optimal (Belady/MIN):** evict the block used furthest in the future (benchmark, not implementable).

**Worked example 11 (FIFO vs LRU, fully associative).** References $1,2,3,4,1,2,5,1,2,3,4,5$.

| Frames | FIFO faults | LRU faults |
|---|---|---|
| 3 | **9** | 10 |
| 4 | **10** | 8 |

FIFO with 4 frames faults **more** than with 3 frames: **Belady's anomaly**. LRU is a *stack algorithm* and never shows it. The same analysis applies to page replacement ([Operating systems](../13-operating-systems/README.md)).

### Types of misses (3 Cs)
- **Compulsory (cold):** first access to a block; reduced by larger blocks/prefetch.
- **Capacity:** working set larger than the cache; reduced only by a larger cache. Fully associative cache of the same size still has these.
- **Conflict:** too many blocks map to the same set; eliminated by full associativity, reduced by higher associativity.

### Array traversal miss counting
`int A[256][256]` (4 B ints, row-major), direct-mapped 4 KB cache, 64 B blocks (16 ints per block, 64 lines). One row $=1$ KB $=16$ blocks.

**Row-major loop** (`for i for j sum += A[i][j]`): consecutive ints share a block; every block is loaded once, all 16 ints used: misses $=65536/16=4096$ (only compulsory). **Hit ratio $=15/16$.**

**Column-major loop** (`for j for i sum += A[i][j]`): consecutive accesses are 1 KB apart (a different block each time), and the 256 blocks of a column map to line $(16i+\lfloor j/16\rfloor)\bmod64$, which takes only 4 distinct values as $i$ varies, so by the time you return to the same block it was evicted: **every access misses: 65536 misses** (simulated; also true for a 64-way fully associative LRU cache of the same size, since a column touches 256 blocks $>64$ lines). If the whole array fit in the cache (1 MB cache) the misses would drop to 4096 again.

**Rule:** spatial locality counts the stride in bytes: stride $\ge B$ gives one miss per access (no spatial reuse); stride $s<B$ gives $1$ miss per $B/s$ accesses at best. **Loop interchange and tiling** improve locality.

**Worked example 12 (stride).** Reading every 4th int (stride 16 B) from a long array with 64 B blocks: each block serves $64/16=4$ accesses: miss ratio $=1/4$.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Chips needed | $(N/k)(M/m)$ | interfacing |
| Decoder for banks | $\log_2(\text{chips in depth})$ address bits | interfacing |
| AMAT hierarchical | $t_1+(1-h)t_2$ | AMAT |
| AMAT simultaneous | $ht_1+(1-h)t_2$ | AMAT |
| Multi-level | $t_1+m_1(t_2+m_2t_3)$ | AMAT |
| CPI with cache | $\text{CPI}_{base}+\text{refs/instr}\times m\times\text{penalty}$ | performance |
| Lines / sets | $C/B$ ; $C/(Bk)$ | cache geometry |
| Bits | offset $\log_2B$; index $\log_2S$; tag = rest | address split |
| Tag store | lines $\times$ (tag + valid [+ dirty]) | overhead |
| Disk time | seek + $\frac12$rotation + transfer | secondary storage |
| Belady | FIFO can increase faults with more frames; LRU cannot | replacement |

## GATE traps

- **Index bits are $\log_2(\text{sets})$, not $\log_2(\text{lines})$,** for set-associative caches.
- Count **tag** from the **physical** address width in physically tagged caches; a virtually indexed cache changes the question (use the width given).
- AMAT: read whether the second-level is accessed **after** (hierarchical) or **in parallel** (simultaneous). If unspecified, "hierarchical" is the usual GATE default for cache + memory when penalties include both; if the question gives "time to access main memory after a miss" it is hierarchical.
- Use **local** miss ratios in the nested formula; global ones in the weighted sum.
- Fully associative has **no conflict misses**; direct mapped has no replacement choice.
- Write-back needs a **dirty bit**; write-through does not, so tag-store sizes differ.
- Cache size questions normally exclude tag/valid bits unless asked "total storage".
- Array traversal: use **stride in bytes**; when the question gives a whole array that fits in cache, only compulsory misses remain.
- LRU is not always better than FIFO on a given trace (Example 11: 3 frames).
- Memory-mapped I/O consumes address space; isolated I/O needs separate instructions.
- Word size vs byte addressing: if memory is **word addressable**, the offset counts words.

## Connections

- [Combinational circuits](../09-digital-logic/combinational-circuits.md) — address decoders, comparators for tag matching, MUXes for way selection.
- [Instruction sets and addressing](instruction-sets-and-addressing.md) — addressing modes generate the reference stream; word vs byte addressing.
- [Pipelining](pipelining.md) — cache misses and structural hazards on the memory port stall the pipeline; the MEM stage time depends on the L1 hit time.
- [I/O, interrupts and DMA](io-interrupts-dma.md) — memory-mapped I/O; DMA competes with the CPU for the memory bus.
- [Operating systems](../13-operating-systems/README.md) — paging and TLBs use the same hit-ratio and replacement ideas; Belady's anomaly.
- [Arrays and linked lists](../07-data-structures/arrays-stacks-queues.md) — array layout and traversal order decide cache behaviour.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — cache-friendly algorithms (merge sort/quick sort locality, binary search misses).
- [Hashing](../08-algorithms/hashing.md) — direct-mapped cache indexing is hashing by $\bmod$.

## Practice

**Q1 (NAT).** How many $16K\times4$ memory chips are needed to build a $256K\times16$ memory?

<details><summary>Answer</summary>

**Answer:** 64.
**Solution:** Depth $256K/16K=16$; width $16/4=4$; $16\times4=64$ chips.

</details>

**Q2 (NAT).** A cache with hit time 4 ns and a main memory with access time 100 ns. Hit ratio 0.95, hierarchical access. Average access time (ns)?

<details><summary>Answer</summary>

**Answer:** 9.
**Solution:** $0.95\times4+0.05\times(4+100)=3.8+5.2=9.0$ ns. (Equivalently $4+0.05\times100$.)

</details>

**Q3 (NAT).** A 32-bit address, 16 KB 4-way set associative cache, 32 B blocks. Number of tag bits?

<details><summary>Answer</summary>

**Answer:** 20.
**Solution:** Lines $=16K/32=512$; sets $=512/4=128$ so index $=7$; offset $=5$; tag $=32-7-5=20$.

</details>

**Q4 (MCQ).** Which type of miss is eliminated in a fully associative cache of the same capacity?
(a) compulsory (b) capacity (c) conflict (d) all

<details><summary>Answer</summary>

**Answer:** (c).
**Solution:** A block can be placed anywhere, so there are no set conflicts. Compulsory and capacity misses remain.

</details>

**Q5 (NAT).** Direct-mapped cache with 8 lines; block-address sequence $0,8,16,0,8,16,1,9$. How many misses?

<details><summary>Answer</summary>

**Answer:** 8.
**Solution:** Blocks $0,8,16$ all map to line 0 and evict one another: each of the first 6 references misses; $1$ and $9$ map to line 1 and conflict: both miss. Total $8$ (all references miss).

</details>

**Q6 (NAT).** A system uses a write-back L1 cache. 20% of references are writes; assume all block replacements are of dirty blocks with probability 0.3. Miss ratio $0.05$, memory block fetch $=100$ ns, write-back $=100$ ns, hit time 2 ns. Average time per reference (ns)?

<details><summary>Answer</summary>

**Answer:** 8.5.
**Solution:** Miss penalty (beyond the hit time) $=100+0.3\times100=130$ ns. $t=2+0.05\times130=8.5$ ns. (With write-allocate, write misses also fetch a block, so the 20% write fraction does not change the formula.)

</details>

**Q7 (NAT).** `int A[512][512]`, 4-byte ints, row-major. Direct-mapped 8 KB cache, 32 B blocks. A loop sums the array column by column (`for j for i`). How many misses? (A row is 2 KB.)

<details><summary>Answer</summary>

**Answer:** 262144.
**Solution:** Cache has 256 lines. Consecutive accesses are 2 KB apart, i.e. 64 blocks apart, so the 512 blocks of one column map to only $256/64=4$ lines (rows $i$ and $i+4$ collide). Each access touches a block that has been evicted since its previous use (the column has 512 blocks, more than the cache holds), so every access misses: $512\times512=262144$ misses.

</details>

**Q8 (MCQ).** A 2-way set-associative cache has 4 sets (8 lines). Block sequence $0,4,8,0,4,8$ with LRU. Misses?
(a) 3 (b) 4 (c) 6 (d) 5

<details><summary>Answer</summary>

**Answer:** (c).
**Solution:** All blocks map to set 0 ($b\bmod4=0$), two ways. Sequence cycles over 3 blocks in a 2-line set with LRU: each reference evicts the one needed next, so every access misses: $6$.

</details>
