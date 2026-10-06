# File systems and disk scheduling

> **Paper:** CS · **Priority:** P1 · **Plan topics:** file allocation, directories, free space, disk scheduling
> **Prerequisites:** [Memory management](memory-management.md) · **Leads to:** [Database indexing](../14-databases/file-organization-and-indexing.md)

## Quick glance
- Contiguous allocation gives fast sequential/direct access but external fragmentation and hard growth.
- Linked allocation grows easily and supports sequential access; direct access is slow.
- Indexed allocation stores pointers in an index block and supports direct access, with index overhead.
- FCFS disk scheduling is fair by arrival; SSTF minimises next seek but can starve distant requests.
- SCAN sweeps like an elevator; C-SCAN services in one direction for more uniform wait.

## 1. File allocation
Suppose a file occupies blocks 10, 11, 12 contiguously: block lookup is simple and sequential reads are efficient. If it grows but block 13 is occupied, relocation or fragmentation is needed. Linked allocation stores next-block pointers; indexed allocation keeps a per-file block list or tree.

## 2. Disk scheduling
Head starts at cylinder 50; requests are 10, 55, 90. FCFS in listed order travels $40+45+35=120$. SSTF goes 55, 90, 10: $5+35+80=120$; other sets may differ. Calculate absolute movement and follow the exact sweep direction/boundary rule given.

## 3. Directories and free space
Directories map names to metadata; a file control block/inode stores attributes and block pointers. Free space can be tracked by bitmap, linked list, grouping, or counting. A bitmap uses one bit per block, making runs easy to find.

## GATE traps
- Total head movement sums absolute differences, not signed displacement.
- SSTF can starve a far-away request under a steady nearby load.
- SCAN reaches an end before reversing; LOOK reverses at the last request.
- File allocation and memory allocation use similar words but different structures.

## Connections
- [Database indexing](../14-databases/file-organization-and-indexing.md) — indexes also trade storage overhead for faster lookup.
- [Memory management](memory-management.md) — internal/external fragmentation distinction.
- [Computer organisation I/O](../10-computer-organization/io-interrupts-dma.md) — devices transfer data to storage.

## Practice
**Q1.** Which allocation method provides direct access naturally: linked or indexed?
<details><summary>Answer</summary> Indexed allocation.</details>

**Q2.** Head moves from cylinder 20 to 70. Movement?
<details><summary>Answer</summary> 50 cylinders.</details>

**Q3.** Which policy risks starvation by always serving nearest request?
<details><summary>Answer</summary> SSTF.</details>
