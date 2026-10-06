# Virtual memory and page replacement

> **Paper:** CS · **Priority:** P0 · **Plan topics:** demand paging, page faults, replacement, working set, TLB/EMAT, thrashing
> **Prerequisites:** [Memory management](memory-management.md) · **Leads to:** [Computer architecture cache](../10-computer-organization/memory-hierarchy-and-cache.md)

## Quick glance
- Page fault: referenced page is not resident; OS loads it and updates mapping.
- FIFO can show Belady's anomaly; LRU and OPT are stack algorithms and do not.
- OPT replaces page whose next use is farthest in future; benchmark, not implementable online.
- TLB hit avoids page-table memory access; page fault is much more expensive than TLB miss.
- Thrashing: too few frames cause excessive faults, reducing useful CPU work.

## 1. Page-fault handling
Trap to OS → validate reference → locate free frame or choose victim → write dirty victim if needed → read demanded page → update page table/TLB → restart instruction. Invalid addresses cause protection failure, not ordinary page-in.

## 2. Replacement example
Reference string 1,2,3,1,4 with 3 frames, initially empty. FIFO faults on 1,2,3; 1 hits; 4 replaces oldest page 1: total 4 faults. LRU also has 4 here. Always state frame count and initial state when counting.

## 3. TLB effective access time
If TLB lookup time is $t$, memory time $m$, hit ratio h, and a hit needs one memory access while miss needs page-table plus data access, EAT=$h(t+m)+(1-h)(t+2m)$ (assuming one-level page table and serial lookup). Substitute carefully; some questions define timing differently.

## GATE traps
- FIFO order is load order, not most/least recently used order.
- A dirty victim needs write-back; clean victim does not.
- Replacement policy comparisons depend on the exact reference string and frame count.
- Page fault penalty is distinct from a TLB miss without a page fault.

## Connections
- [Memory management](memory-management.md) — page numbers and frames.
- [Cache hierarchy](../10-computer-organization/memory-hierarchy-and-cache.md) — locality explains both TLB/cache effectiveness.
- [CPU scheduling](cpu-scheduling.md) — thrashing can make utilisation fall and mislead load control.

## Practice
**Q1.** Which policy can exhibit Belady's anomaly?
<details><summary>Answer</summary> FIFO.</details>

**Q2.** With h=0.9, TLB time 10 ns, memory time 100 ns and one-level page table, find EAT.
<details><summary>Answer</summary> $0.9(110)+0.1(210)=120$ ns.</details>

**Q3.** What does OPT replace?
<details><summary>Answer</summary> The resident page whose next reference is farthest in the future.</details>
