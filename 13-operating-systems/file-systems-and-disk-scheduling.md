# File Systems and Disk Scheduling

> **Paper:** CS · **Priority:** P1 · **Plan topics:** file allocation, directories, free space, disk scheduling
> **Prerequisites:** [Memory management](memory-management.md) · **Leads to:** [Database indexing](../14-databases/file-organization-and-indexing.md)
> **PYQ weightage:** Medium · 13 Qs since 2015 (5 since 2021) · DA: 0 · Trend: steady · Most asked: Disk Access & Inodes, Disk Scheduling, File Allocation

## Quick glance

- **Exam focus:** Multi-level indexed allocation / UNIX Inode maximum file size calculations (NAT), disk access time (seek + rotational latency + transfer rate), and head movement in disk scheduling algorithms (SSTF, SCAN, C-SCAN, LOOK, C-LOOK).
- **Disk structure:** Platter $\to$ Surfaces $\to$ Tracks $\to$ Sectors. Cylinder = set of all tracks across surfaces at identical arm radius. **Number of cylinders = number of tracks per surface**.
- **Disk access time:** $T_{\text{access}} = T_{\text{seek}} + T_{\text{rotational}} + T_{\text{transfer}} + T_{\text{controller}}$.
- **Average rotational latency:** $\frac{1}{2} \times \frac{60}{\text{RPM}}$ seconds (time for half a revolution).
- **Disk scheduling policies:**
  - **FCFS:** Serves requests strictly in arrival order; fair, but causes large head travel.
  - **SSTF:** Greedily serves closest request to current head position; minimizes average seek time, but causes **starvation** for distant cylinders.
  - **SCAN (Elevator):** Sweeps to the **physical disk boundary (0 or max cylinder)** before reversing.
  - **C-SCAN:** Sweeps in one direction to the boundary, jumps directly to cylinder 0 without servicing on the return trip, then sweeps again. Provides uniform waiting time.
  - **LOOK / C-LOOK:** Like SCAN / C-SCAN, but reverses at the **furthest pending request** rather than traveling to the physical disk edge.
- **File allocation strategies:**
  - **Contiguous:** Fast direct ($O(1)$) and sequential access; suffers from external fragmentation and rigid file growth.
  - **Linked:** Non-contiguous blocks linked via pointers; no external fragmentation, but slow direct access ($k$ disk accesses for block $k$) and internal fragmentation in the last block.
  - **Indexed / UNIX Inode:** Direct pointers + single/double/triple indirect blocks. Max file size: $[D + I + I^2 + I^3] \times B$, where $I = \frac{B}{P}$ (block size $B$, pointer size $P$).
- **Free space management:** Bitmaps (1 bit/block; fast contiguous search), linked free lists, grouping, counting.

## 1. Magnetic Disk Geometry and Access Time [PYQ: Medium]

A hard disk drive consists of stacked platters spinning on a central spindle. Both surfaces of each platter are coated with magnetic material and read by mechanical arms holding read/write heads.

```text
       Spindle
          |
     [ Platter 1 ]  ==== Head (Surface 0, Track t)
          |         ==== Head (Surface 1, Track t)   <-- Cylinder t = set of all
     [ Platter 2 ]  ==== Head (Surface 2, Track t)       tracks at radius t
          |         ==== Head (Surface 3, Track t)
```

### 1.1 Disk Geometry and Capacity Formulas

$$\text{Total Disk Capacity} = (\text{Surfaces}) \times (\text{Tracks/surface}) \times (\text{Sectors/track}) \times (\text{Bytes/sector})$$
$$\text{Number of Cylinders} = \text{Tracks per surface}$$
$$\text{Track Capacity} = (\text{Sectors per track}) \times (\text{Sector size})$$

### 1.2 Access Time Components

1. **Seek Time ($T_{\text{seek}}$):** Time required for the mechanical arm to move the read/write head to the designated cylinder.
2. **Rotational Latency ($T_{\text{rot}}$):** Time waiting for the target sector to rotate underneath the head. On average:
   $$T_{\text{rot, avg}} = \frac{1}{2} \times \left(\frac{60}{\text{RPM}}\right) \text{ seconds}$$
3. **Transfer Time ($T_{\text{trans}}$):** Time to stream data under the head:
   $$T_{\text{trans}} = \frac{\text{Bytes to transfer}}{\text{Transfer Rate}} = \frac{\text{Sectors to transfer}}{\text{Sectors per track}} \times \left(\frac{60}{\text{RPM}}\right)$$
   $$\text{Transfer Rate} = \text{Track Capacity} \times \left(\frac{\text{RPM}}{60}\right) \text{ bytes/sec}$$

### Worked Example 1 (Random vs. Contiguous Sector Access)

A disk rotates at 7200 RPM, has an average seek time of 4 ms, 500 sectors per track, and 512 bytes per sector. Calculate:
(a) Average rotational latency.
(b) Time to read 100 sectors located randomly across the disk.
(c) Time to read 100 sectors located contiguously on the same track.

**Step 1: Single revolution time and rotational latency.**
$$T_{\text{rev}} = \frac{60}{7200} = \frac{1}{120}\text{ s} \approx 8.333\text{ ms}$$
$$T_{\text{rot, avg}} = \frac{8.333}{2} \approx 4.167\text{ ms}$$

**Step 2: Single sector transfer time.**
$$T_{\text{sector}} = \frac{T_{\text{rev}}}{\text{sectors/track}} = \frac{8.333\text{ ms}}{500} = 0.0167\text{ ms}$$

**Step 3: 100 random sectors.**
Each randomly located sector requires an independent seek, rotational latency, and transfer:
$$T_{\text{random}} = 100 \times (T_{\text{seek}} + T_{\text{rot, avg}} + T_{\text{sector}}) = 100 \times (4 + 4.167 + 0.0167)\text{ ms} = 100 \times 8.1837\text{ ms} = 818.37\text{ ms}$$

**Step 4: 100 contiguous sectors on one track.**
Requires only 1 seek and 1 rotational latency, followed by streaming 100 consecutive sectors:
$$T_{\text{contiguous}} = T_{\text{seek}} + T_{\text{rot, avg}} + (100 \times T_{\text{sector}}) = 4 + 4.167 + (100 \times 0.0167)\text{ ms} = 9.837\text{ ms}$$
*(Contiguous allocation is ~83 times faster than random access).*

## 2. Disk Scheduling Algorithms [PYQ: Medium]

The OS disk scheduler reorders pending cylinder I/O requests to minimize head travel and prevent starvation.

- **FCFS:** Serves requests in arrival order. Fair, simple, but has poor performance.
- **SSTF:** Greedily picks the request closest to the current head position ($|r - \text{current}|$). Minimizes seek time locally, but can cause **starvation** for distant requests under heavy loads.
- **SCAN (Elevator):** Head moves in one direction servicing requests until it hits the **physical boundary of the disk** (cylinder 0 or max cylinder), then reverses direction.
- **C-SCAN (Circular SCAN):** Sweeps toward one end servicing requests until the boundary, then immediately jumps back to the opposite boundary (cylinder 0) without servicing any requests on the return journey, providing more uniform wait times.
- **LOOK and C-LOOK:** Like SCAN and C-SCAN, but the head only travels as far as the **furthest pending request** in the current direction before reversing or jumping, never wasting seeks to empty boundary tracks.

### Worked Example 2 (Comparing All Schedulers on a Request Queue)

Disk cylinders: 0 to 199. Current head position: **53**, moving towards **higher** cylinder numbers.
Request queue: **98, 183, 37, 122, 14, 124, 65, 67**.
Compute Total Head Movement (THM) for each algorithm.

```text
Pending Requests Partitioned:
Lower than 53:  [14, 37]
Higher than 53: [65, 67, 98, 122, 124, 183]
```

1. **FCFS:** Sequence: $53 \to 98 \to 183 \to 37 \to 122 \to 14 \to 124 \to 65 \to 67$.
   $$\text{THM} = |98-53| + |183-98| + |37-183| + |122-37| + |14-122| + |124-14| + |65-124| + |67-65|$$
   $$\text{THM} = 45 + 85 + 146 + 85 + 108 + 110 + 59 + 2 = \mathbf{640}$$

2. **SSTF:** From 53, nearest is 65 ($|12|$), then 67 ($|2|$), then 37 ($|30|$), then 14 ($|23|$), then 98 ($|84|$), 122 ($|24|$), 124 ($|2|$), 183 ($|59|$).
   $$\text{Sequence: } 53 \to 65 \to 67 \to 37 \to 14 \to 98 \to 122 \to 124 \to 183$$
   $$\text{THM} = (67 - 53) + (67 - 14) + (183 - 14) = 14 + 53 + 169 = \mathbf{236}$$

3. **SCAN:** Sweeps up to the physical disk boundary (**199**), then reverses down to 14:
   $$\text{Sequence: } 53 \to 65 \to 67 \to 98 \to 122 \to 124 \to 183 \to \mathbf{199} \to 37 \to 14$$
   $$\text{THM} = (199 - 53) + (199 - 14) = 146 + 185 = \mathbf{331}$$

4. **C-SCAN:** Sweeps up to boundary (**199**), jumps to **0**, then sweeps up to 37:
   $$\text{Sequence: } 53 \to 65 \to 67 \to 98 \to 122 \to 124 \to 183 \to \mathbf{199} \to \mathbf{0} \to 14 \to 37$$
   $$\text{THM} = (199 - 53) + (199 - 0) + (37 - 0) = 146 + 199 + 37 = \mathbf{382}$$

5. **LOOK:** Reverses at the furthest request (**183**), not 199:
   $$\text{Sequence: } 53 \to 65 \to 67 \to 98 \to 122 \to 124 \to 183 \to 37 \to 14$$
   $$\text{THM} = (183 - 53) + (183 - 14) = 130 + 169 = \mathbf{299}$$

6. **C-LOOK:** Sweeps to **183**, jumps directly to lowest request (**14**), then up to 37:
   $$\text{Sequence: } 53 \to 65 \to 67 \to 98 \to 122 \to 124 \to 183 \to 14 \to 37$$
   $$\text{THM} = (183 - 53) + (183 - 14) + (37 - 14) = 130 + 169 + 23 = \mathbf{322}$$

## 3. File Allocation Methods and UNIX Inode [PYQ: Medium]

### 3.1 Allocation Strategies

| Feature | Contiguous Allocation | Linked Allocation | Indexed Allocation (Inode) |
|---|---|---|---|
| **Directory stores** | Start block, Length | Start block, End block | Inode / Index block pointer |
| **Sequential access** | Fast | Fast | Fast |
| **Direct (Random) access** | Fast ($O(1)$) | Slow ($O(k)$ disk reads) | Fast (via index levels) |
| **External fragmentation** | Yes | None | None |
| **Internal fragmentation** | Only in last block | Only in last block | Only in last block |
| **File growth** | Difficult (pre-allocation) | Trivial | Trivial (dynamically add index blocks) |
| **Storage overhead** | Negligible | Pointer per block ($P$ bytes) | Dedicated index blocks / Inode |

- **Linked Allocation Usable Space (GATE 2025):** If block size is $B$ and pointer is $P$ bytes, usable data per block is $B - P$. Total blocks = $\lceil \text{FileSize} / (B - P) \rceil$.
- **File Allocation Table (FAT):** A variation of linked allocation where all block pointers are kept in a single table cached in RAM. Random access does not require disk seeks to trace pointers.

### 3.2 UNIX Inode and Maximum File Size

A UNIX Inode stores file attributes and a fixed set of block addresses:
- $D$ direct pointers: point directly to data blocks.
- 1 single indirect pointer: points to an index block of block pointers.
- 1 double indirect pointer: points to an index block of index blocks.
- 1 triple indirect pointer: points to 3 levels of index blocks.

```text
[ Inode ]
  |-- Direct [0..D-1] --------------> Data Block
  |-- Single Indirect --------------> [ Index Block ] --------------> Data Block
  |-- Double Indirect --------------> [ Index Block ] --> [ Index ] --> Data Block
  |-- Triple Indirect --------------> [ Index ] --> [ Index ] --> [ Index ] --> Data
```

Let block size be $B$ and disk block pointer size be $P$. Pointers per index block: $I = \frac{B}{P}$.
$$\text{Max Data Blocks} = D + I + I^2 + I^3$$
$$\text{Maximum File Size} = \text{Max Data Blocks} \times B$$

### Worked Example 3 (UNIX Inode Capacity and Block Access Calculation)

An OS uses an inode with 12 direct pointers, 1 single indirect pointer, 1 double indirect pointer, and 1 triple indirect pointer. Block size is 4 KB ($4096$ bytes) and disk block pointer size is 4 bytes.
Calculate: (a) Maximum file size. (b) Number of disk block reads to access byte offset 20,000,000 (assume inode is already in memory).

**Step 1: Pointers per index block.**
$$I = \frac{\text{Block Size}}{\text{Pointer Size}} = \frac{4096}{4} = 1024 = 2^{10}$$

**Step 2: Capacity per level.**
- Direct blocks: $12 \times 4\text{ KB} = 48\text{ KB}$
- Single indirect: $1024 \times 4\text{ KB} = 4\text{ MB}$
- Double indirect: $1024^2 \times 4\text{ KB} = 1024 \times 4\text{ MB} = 4\text{ GB}$
- Triple indirect: $1024^3 \times 4\text{ KB} = 1024 \times 4\text{ GB} = 4\text{ TB}$
$$\text{Maximum File Size} = 48\text{ KB} + 4\text{ MB} + 4\text{ GB} + 4\text{ TB} \approx \mathbf{4\text{ TB}}$$

**Step 3: Block reads for byte offset 20,000,000 (~19.07 MB).**
- Direct range: bytes 0 to $48\text{ KB} - 1$ ($0$ to $49,151$).
- Single indirect range: bytes $49,152$ to $49,152 + 4\text{ MB} - 1 \approx 4.048\text{ MB}$.
- Since $20,000,000\text{ bytes} \approx 19.07\text{ MB}$, the target byte falls into the **Double Indirect** range!
- Access requires:
  1. Read 1st-level index block (pointed to by double indirect pointer in inode).
  2. Read 2nd-level index block.
  3. Read the actual data block.
Total disk block reads = $1 + 1 + 1 = \mathbf{3}$ disk reads.

## 4. Directory Structures and Free Space Management [PYQ: Low]

### 4.1 Directory Implementations

- **Linear List:** Directory contains a list of filename and pointer pairs.
  - Adding a file requires a **linear scan of the entire directory** to verify filename uniqueness (GATE 2021).
  - Deletion/search: $O(n)$ time.
- **Hash Table:** Computes hash of filename; $O(1)$ average search/insert time.

### 4.2 File Sharing: Hard Links vs. Soft (Symbolic) Links

- **Hard Link:** Directory entry points directly to the file's **inode**.
  - Inode contains an incremented `reference_count`.
  - Deleting one hard link decrements `reference_count`. Data blocks are freed only when `reference_count == 0`.
  - Cannot span across different filesystems or link directories (to prevent cycles).
- **Soft / Symbolic Link:** A separate file containing the pathname string of the target file.
  - Has its own inode and data block.
  - If target file is deleted, the soft link becomes a **dangling link**.
  - Can link across filesystems and link directories.

### 4.3 Free Space Management

1. **Bit Vector (Bitmap):** 1 bit per disk block (1 = free, 0 = allocated).
   - Space overhead: $\lceil \text{Disk Blocks} / 8 \rceil$ bytes.
   - Fast contiguous block search using CPU bitwise instructions.
2. **Linked Free List:** First free block points to the next free block.
   - Zero additional disk space overhead, but finding contiguous runs requires extensive seeks.
3. **Grouping:** First free block stores addresses of $n$ free blocks; the $n$-th points to another index block.
4. **Counting:** Stores list of pairs: `(first_free_block_address, contiguous_free_block_count)`.

## Formulas and facts to memorise

| Item | Formula / Rule | When to use |
|---|---|---|
| Cylinders | $\text{Cylinders} = \text{Tracks per surface}$ | Disk geometry calculations |
| Disk Capacity | $\text{Surfaces} \times \text{Tracks/surface} \times \text{Sectors/track} \times \text{Sector Size}$ | Total disk size |
| Rotational Latency | $T_{\text{rot, avg}} = \frac{1}{2} \times \frac{60}{\text{RPM}}$ seconds | Disk access time |
| Transfer Rate | $\text{Track Capacity} \times \frac{\text{RPM}}{60}$ | Sustained read/write throughput |
| SCAN Boundary Rule | Head must travel to cylinder 0 or max cylinder before reversing | Tracing SCAN / C-SCAN |
| LOOK Boundary Rule | Head reverses at the furthest pending request in that direction | Tracing LOOK / C-LOOK |
| Inode Index Pointers | $I = \frac{\text{Block Size } B}{\text{Pointer Size } P}$ | Index block capacity |
| Inode Max Blocks | $\text{Blocks} = D + I + I^2 + I^3$ | Calculating maximum file size |
| Linked Data Per Block | $\text{Data} = \text{Block Size} - \text{Pointer Size}$ | Linked allocation overhead |
| Bitmap Size | $\text{Bytes} = \lceil \text{Disk Blocks} / 8 \rceil$ | Free space bitmap overhead |

## GATE traps

- **SCAN Hits the Edge:** In SCAN and C-SCAN, the head travels all the way to cylinder 0 or max cylinder (e.g. 199) even if no request exists there. In LOOK and C-LOOK, the head reverses at the furthest request.
- **Direction Reversal Calculation:** Total head movement is the sum of **absolute differences**, not signed displacement.
- **Number of Cylinders $\ne$ Total Tracks:** Number of cylinders equals tracks on **one surface**, not the sum across all surfaces.
- **Bytes vs Blocks in Inode:** When asked for maximum file size, multiply the total block count by the block size $B$.
- **Inode in Memory:** Inode is kept in memory during open file operations; do not count an extra disk read for the inode itself unless explicitly stated.
- **Dangling Links:** Deleting the original file breaks a symbolic link, but leaves a hard link intact.

## Connections

- [Memory management](memory-management.md) — internal/external fragmentation concepts apply directly to file allocation blocks.
- [Database indexing](../14-databases/file-organization-and-indexing.md) — multi-level inode indexing is the direct operating system foundation for database B+ Trees.
- [I/O, interrupts, DMA](../10-computer-organization/io-interrupts-dma.md) — disk controllers transfer data directly to RAM via DMA.
- [CPU scheduling](cpu-scheduling.md) — SSTF is the disk analog of Shortest Job First (SJF) and shares its starvation property.

## Practice

**Q1 (NAT, easy).** A hard disk rotates at 10,000 RPM. What is its average rotational latency in milliseconds?

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:**
$$T_{\text{rot, avg}} = \frac{1}{2} \times \left(\frac{60}{10000}\right)\text{ s} = \frac{1}{2} \times 0.006\text{ s} = 0.003\text{ s} = 3\text{ ms}$$

</details>

**Q2 (NAT).** Consider a disk queue with requests for cylinders: 82, 170, 43, 140, 24, 16, 190. The read/write head is currently at cylinder 50. Using Shortest Seek Time First (SSTF) scheduling, what is the Total Head Movement (in cylinders)?

<details><summary>Answer</summary>

**Answer:** 208  
**Solution:**
Trace SSTF from 50:
1. From 50: distances are $|43-50|=7$, $|82-50|=32$, etc. Nearest is 43.
2. From 43: nearest is 24 ($|19|$).
3. From 24: nearest is 16 ($|8|$).
4. From 16: nearest is 82 ($|66|$).
5. From 82: nearest is 140 ($|58|$).
6. From 140: nearest is 170 ($|30|$).
7. From 170: nearest is 190 ($|20|$).
$$\text{Order: } 50 \to 43 \to 24 \to 16 \to 82 \to 140 \to 170 \to 190$$
$$\text{THM} = (50 - 16) + (190 - 16) = 34 + 174 = 208$$

</details>

**Q3 (MCQ).** Which of the following disk scheduling algorithms guarantees that requests are serviced with the most uniform waiting time?  
(A) FCFS  
(B) SSTF  
(C) SCAN  
(D) C-SCAN  

<details><summary>Answer</summary>

**Answer:** (D)  
**Solution:** C-SCAN sweeps in one direction and resets to the beginning without servicing requests on the return trip. This avoids the bias of SCAN (which gives preferential lower wait times to cylinders near the edges where the head reverses), providing the most uniform waiting time.

</details>

**Q4 (NAT).** A disk of size 16 GB is divided into blocks of 4096 bytes. A file of 100,000 bytes is stored using linked allocation, where each data block reserves 4 bytes to store the pointer to the next block. How much space (in bytes) is wasted due to internal fragmentation in the last block?

<details><summary>Answer</summary>

**Answer:** 2300  
**Solution:**
Usable data per block = $4096 - 4 = 4092$ bytes.
Number of blocks required = $\lceil 100000 / 4092 \rceil = \lceil 24.438 \rceil = 25$ blocks.
Data stored in the first 24 blocks = $24 \times 4092 = 98,208$ bytes.
Data stored in the 25th (last) block = $100,000 - 98,208 = 1,792$ bytes.
Internal fragmentation in the last block = $4092 - 1792 = 2300$ bytes.

</details>

**Q5 (NAT).** An operating system uses a UNIX-style inode with 10 direct block addresses, 1 single indirect block address, and 1 double indirect block address. The disk block size is 2048 bytes (2 KB) and disk block addresses are 4 bytes each. What is the maximum number of data blocks that can be addressed by a file?

<details><summary>Answer</summary>

**Answer:** 262666  
**Solution:**
Pointers per index block: $I = 2048 / 4 = 512$.
- Direct blocks: 10
- Single indirect: $I = 512$
- Double indirect: $I^2 = 512^2 = 262,144$
$$\text{Total data blocks} = 10 + 512 + 262,144 = 262,666$$

</details>

**Q6 (MSQ).** Which of the following statements about file systems and disk scheduling is/are TRUE?  
(A) SSTF can cause starvation of requests for cylinders far from the current head position  
(B) In UNIX, deleting a file with multiple hard links removes the file data immediately from the disk  
(C) C-LOOK always travels to the physical ends of the disk before reversing  
(D) Contiguous allocation provides $O(1)$ random access to any logical block of a file  

<details><summary>Answer</summary>

**Answer:** (A), (D)  
**Solution:**
- (A) is TRUE: continuous arrivals of nearby requests starve distant cylinders in SSTF.
- (B) is FALSE: hard links increment the reference count; data is deleted only when reference count reaches 0.
- (C) is FALSE: C-LOOK only moves to the furthest pending request, unlike C-SCAN which hits cylinder boundaries.
- (D) is TRUE: block $k$ is located at physical block $\text{start} + k$.

</details>

**Q7 (MCQ).** In a linear list implementation of a directory, which of the following operations necessarily requires a full scan of the directory?  
(A) Opening an existing file  
(B) Creating a new file  
(C) Deleting an existing file located at the front of the list  
(D) Reading an existing file's attributes  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** To create a new file, the operating system must verify that no existing file in the directory already has the same name. Hence, it must scan every entry in the linear list to confirm uniqueness before inserting the new entry.

</details>

**Q8 (NAT).** An inode has 10 direct pointers, 1 single indirect pointer, and 1 double indirect pointer. Block size is 4 KB and disk addresses are 4 bytes. Assuming the inode is already in memory, how many disk block reads are required to access the byte at offset 3,000,000 in the file?

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:**
Block size = 4096 bytes. Pointers per index block = $4096 / 4 = 1024$.
- Direct blocks cover: $10 \times 4096 = 40,960$ bytes (bytes 0 to 40,959).
- Single indirect covers: $1024 \times 4096 = 4,194,304$ bytes (bytes 40,960 to $4,235,263$).
Byte offset 3,000,000 falls within the **Single Indirect** range!
To read this data byte:
1. Read the single indirect index block from disk.
2. Read the target data block from disk.
Total disk block reads = 2.

</details>
