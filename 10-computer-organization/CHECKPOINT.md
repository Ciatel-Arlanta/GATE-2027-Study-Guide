# Computer Organization: Checkpoint

12 mixed GATE-style questions, easy to hard. Chapters: [Instruction sets](instruction-sets-and-addressing.md) (IS), [ALU and control](alu-and-control-unit.md) (AC), [Memory and cache](memory-hierarchy-and-cache.md) (MC), [I/O](io-interrupts-dma.md) (IO), [Pipelining](pipelining.md) (PL).

**Q1 (NAT, IS).** A 16-bit instruction has 4-bit address fields and a 4-bit opcode at level 1. The processor supports 15 three-address, 14 two-address and 31 one-address instructions. Maximum number of zero-address instructions?

<details><summary>Answer</summary>

**Answer:** 16.
**Solution:** Free after level 1: $16-15=1$; level 2: $1\times16-14=2$; level 3: $2\times16-31=1$; level 4: $1\times16=16$.

</details>

**Q2 (NAT, IS).** An instruction at address 3000 (4 bytes) uses PC-relative addressing with offset $+120$. What is the effective address?

<details><summary>Answer</summary>

**Answer:** 3124.
**Solution:** PC after fetch $=3004$; $3004+120=3124$.

</details>

**Q3 (NAT, MC).** How many $16K\times4$ chips build a $256K\times16$ memory?

<details><summary>Answer</summary>

**Answer:** 64.
**Solution:** $(256K/16K)\times(16/4)=16\times4=64$.

</details>

**Q4 (NAT, MC).** 32-bit address, 64 KB 8-way set associative cache with 64 B blocks. Number of tag bits? Total tag-store bits if each line has one valid bit (give the tag-store total).

<details><summary>Answer</summary>

**Answer:** Tag $=19$ bits; tag store $=20480$ bits.
**Solution:** Lines $=64K/64=1024$; sets $=1024/8=128$ $\Rightarrow$ index 7; offset 6; tag $=32-7-6=19$. Store $=1024\times(19+1)=20480$ bits.

</details>

**Q5 (NAT, MC).** $t_{L1}=2$ ns with hit ratio 0.9; L2 access 12 ns with local hit ratio 0.75; memory 80 ns (hierarchical). Average access time (ns)?

<details><summary>Answer</summary>

**Answer:** 5.2.
**Solution:** $2+0.1\times(12+0.25\times80)=2+0.1\times32=5.2$.

</details>

**Q6 (NAT, MC).** For the reference string $1,2,3,4,1,2,5,1,2,3,4,5$ in a fully associative cache with 3 lines, how many more misses does LRU incur than FIFO?

<details><summary>Answer</summary>

**Answer:** 1.
**Solution:** FIFO: 9 misses; LRU: 10 misses (simulated). Difference 1. (With 4 lines FIFO gives 10 and LRU 8: Belady's anomaly for FIFO.)

</details>

**Q7 (NAT, AC).** A control word has three groups of mutually exclusive signals with 4, 9 and 20 signals ("none" must be encodable). The control memory has 256 words with an 8-bit next-address field and a 2-bit condition-select field. How many more bits does the horizontal version need than the encoded (vertical) version in total control-memory size?

<details><summary>Answer</summary>

**Answer:** 5376 bits.
**Solution:** Horizontal control field $=4+9+20=33$ bits; encoded $=\lceil\log_25\rceil+\lceil\log_210\rceil+\lceil\log_221\rceil=3+4+5=12$. Word widths: $33+8+2=43$ and $12+8+2=22$. Difference $21\times256=5376$ bits.

</details>

**Q8 (NAT, IO).** CPU 2 GHz. A device transfers 16 MB/s ($16\times10^6$ B/s) and interrupts the CPU once per 8-byte unit, each interrupt costing 300 cycles. What percentage of CPU time is spent servicing it?

<details><summary>Answer</summary>

**Answer:** 30%.
**Solution:** Interrupts/s $=2\times10^6$; cycles $=6\times10^8$; $/2\times10^9=0.3$.

</details>

**Q9 (NAT, PL).** A 6-stage pipeline has stage delays 8, 10, 12, 9, 11, 10 ns and 1 ns of latch overhead per stage. Speedup over the unpipelined machine for 1000 instructions (2 decimals)?

<details><summary>Answer</summary>

**Answer:** 4.59.
**Solution:** Unpipelined per instruction $=60$ ns; clock $=12+1=13$ ns. $S=\dfrac{1000\times60}{(6+999)\times13}=\dfrac{60000}{13065}=4.59$.

</details>

**Q10 (NAT, PL).** With full forwarding in a 5-stage pipeline: `LW R2,0(R1); LW R3,4(R1); ADD R4,R2,R3; SW R4,8(R1)`. Total clock cycles?

<details><summary>Answer</summary>

**Answer:** 9.
**Solution:** Ideal $=4+5-1=8$. ADD needs R3 from the immediately preceding load: load-use, 1 stall. R2 is available (two slots earlier). SW gets R4 by forwarding from ADD: no stall. Total $=9$ (simulated).

</details>

**Q11 (NAT, MC+PL).** A pipelined processor has CPI 1.2 excluding memory stalls. Each instruction makes 1.25 memory references (1 fetch, 0.25 data); L1 miss rate 4%, miss penalty 50 cycles. Effective CPI?

<details><summary>Answer</summary>

**Answer:** 3.7.
**Solution:** $1.2+1.25\times0.04\times50=1.2+2.5=3.7$.

</details>

**Q12 (NAT, AC+DL).** The step-timing logic of a hardwired control unit needs 12 distinct timing signals $T_1..T_{12}$ in sequence. A one-hot ring counter needs $r$ flip-flops and a Johnson counter needs $j$ flip-flops (with 2-input AND gates decoding each state). Find $r-j$.

<details><summary>Answer</summary>

**Answer:** 6.
**Solution:** Ring: 12 FFs for 12 states. Johnson: $2n$ states per $n$ FFs, so $n=6$. $r-j=12-6=6$.

</details>

**Scoring.** 10–12 correct ($\ge80\%$): move on to [Theory of computation](../11-theory-of-computation/README.md). 7–9: revise the weak chapters. Fewer than 7 ($<60\%$): re-read the chapters against the questions you missed: IS: Q1, Q2; MC: Q3, Q4, Q5, Q6, Q11; AC: Q7, Q12; IO: Q8; PL: Q9, Q10, Q11.
