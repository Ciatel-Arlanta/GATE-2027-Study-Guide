# Hashing

> **Paper:** CS+DA · **Priority:** P1 · **Plan topics:** Hashing
> **Prerequisites:** [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) · [Linked lists](../07-data-structures/linked-lists.md) · [Asymptotic analysis](asymptotic-analysis.md) · **Leads to:** [Complexity reference](../07-data-structures/complexity-reference.md) · [File organization and indexing](../14-databases/file-organization-and-indexing.md)

## Quick glance

- A **hash table** of size $m$ stores key $k$ at slot $h(k)\in[0,m)$; search, insert, delete are **expected $O(1)$** with a good $h$ and bounded load factor, $O(n)$ worst case.
- **Load factor** $\alpha=n/m$ (keys per slot). Chaining allows $\alpha>1$; open addressing needs $\alpha\le1$.
- **Chaining:** each slot holds a list. Expected probes: unsuccessful $\alpha$ (list scanned) , successful $1+\alpha/2$.
- **Open addressing probe sequences:** linear $h(k)+i$; quadratic $h(k)+c_1i+c_2i^2$; double $h_1(k)+i\,h_2(k)$ (with $h_2(k)\ne0$ and coprime to $m$).
- **Linear probing causes primary clustering**; quadratic causes secondary clustering; double hashing has neither.
- Uniform-hashing expectations (open addressing): unsuccessful $\le\frac1{1-\alpha}$, successful $\le\frac1\alpha\ln\frac1{1-\alpha}$.
- **Deletion in open addressing needs tombstones** (a "deleted" marker); emptying the slot breaks later searches.
- **#1 trap:** in an insertion trace, check *every* probe: skipped occupied slots count as probes, and a table of size $m$ can still fail to insert under quadratic probing even if not full.

## 1. Idea and terminology

**Intuition:** a library where the shelf number is computed from the book title. You compute the shelf in one step instead of scanning all books. Two titles can compute the same shelf; that is a **collision** and the table must resolve it.

- **Universe** $U$ of possible keys (huge); **table** $T[0..m-1]$ with $m\ll|U|$. By pigeonhole, collisions are unavoidable.
- A **hash function** $h:U\to\{0,\dots,m-1\}$ should be fast, and spread keys **uniformly** (simple uniform hashing: every key equally likely to land in any slot, independently).
- **Load factor** $\alpha=n/m$.

## 2. Hash functions

| Method | Formula | Notes |
|---|---|---|
| Division | $h(k)=k\bmod m$ | $m$ should be a prime not close to a power of 2 (avoid $m=2^p$: only low $p$ bits used) |
| Multiplication | $h(k)=\lfloor m\,(kA\bmod1)\rfloor$, $0<A<1$ | Knuth: $A\approx(\sqrt5-1)/2=0.618\ldots$; $m$ can be a power of 2 |
| Mid-square | square $k$, take middle digits/bits | needs enough digits |
| Folding | split digits into parts and add | then reduce mod $m$ |
| Universal | $h_{a,b}(k)=((ak+b)\bmod p)\bmod m$, $p$ prime $>|U|$, $a\in[1,p-1],b\in[0,p-1]$ random | for any $x\ne y$, $\Pr[h(x)=h(y)]\le1/m$ |

**Worked example 1.**
- Division: $k=100$, $m=13$: $100=7\cdot13+9\Rightarrow h=9$.
- Multiplication: $k=123456$, $A=0.6180339887$, $m=10000$. $kA=76300.0041\ldots$, fractional part $0.0041151$, $h=\lfloor10000\cdot0.0041151\rfloor=41$.
- Mid-square: $k=1234$, $k^2=1522756$ (7 digits); middle 3 digits $=227$, so $h=227$.
- Folding: $k=123456789$ into groups $123,456,789$: sum $=1368$; with $m=1000$, $h=368$.

**Universal hashing** defeats an adversary who knows $h$: choose $h$ at random from a family at run time, so no fixed key set is bad on average. The expected chain length for any key is $\le1+n/m$.

A good $h$ for strings: polynomial rolling hash $h=\left(\sum s_ip^i\right)\bmod m$.

## 3. Collision resolution by chaining (separate chaining)

Each slot points to a linked list of the keys that hash there. Insert at the head ($O(1)$); search scans the chain.

**Worked example 2.** $m=9$, $h(k)=k\bmod9$, insert $5,28,19,15,20,33,12,17,10$ (each at the head of its list).

| Slot | Keys (head → tail) |
|---|---|
| 1 | 10 → 19 → 28 |
| 2 | 20 |
| 3 | 12 |
| 5 | 5 |
| 6 | 33 → 15 |
| 8 | 17 |

($28\bmod9=1$, $19\bmod9=1$, $10\bmod9=1$; $15\bmod9=6$, $33\bmod9=6$.) $\alpha=9/9=1$. Comparisons for a successful search of each key: $10{:}1,19{:}2,28{:}3,33{:}1,15{:}2$ and $1$ each for $20,12,5,17$, total $13$; average $13/9=1.44$ (formula $1+\alpha/2=1.5$).

**Expected costs under simple uniform hashing** ($\alpha=n/m$):

| Operation | Expected comparisons |
|---|---|
| Unsuccessful search / insertion with duplicate check | $\alpha$ (chain length; some books count $1+\alpha$ with the slot access) |
| Successful search | $1+\frac\alpha2-\frac{\alpha}{2n}\approx1+\frac\alpha2$ |
| Delete | same as search, $O(1)$ extra |

If $n=O(m)$ then $\alpha=O(1)$ and all operations are $O(1)$ expected. **Worst case** (all keys in one chain): $\Theta(n)$. Chains can be kept sorted (unsuccessful search stops early) or be BSTs.

## 4. Open addressing

All keys live in the table itself ($n\le m$). On collision, try a **probe sequence** $h(k,0),h(k,1),\dots$, ideally a permutation of $0..m-1$.

| Scheme | $h(k,i)$ | Clustering |
|---|---|---|
| Linear probing | $(h'(k)+i)\bmod m$ | **primary clustering**: runs of occupied slots grow and attract more keys |
| Quadratic probing | $(h'(k)+c_1i+c_2i^2)\bmod m$ | **secondary clustering**: keys with the same $h'$ follow the same sequence |
| Double hashing | $(h_1(k)+i\,h_2(k))\bmod m$ | none (nearly uniform hashing); need $\gcd(h_2(k),m)=1$ |

Quadratic probing visits all slots only for special $m$ (e.g. $m$ a power of 2 with $c_1=c_2=\frac12$, or a prime $m$ and $\alpha\le\frac12$ with $h'+i^2$ guarantees an empty slot is found). **Double hashing** with prime $m$ and $h_2(k)=1+(k\bmod(m-1))$ (or $m-2$) always visits every slot.

### 4.1 Linear probing trace

**Worked example 3 (a classic GATE form).** $m=7$, $h(k)=k\bmod7$, insert $50,700,76,85,92,73,101$.

| Key | $h(k)$ | Probe sequence | Probes | Placed at |
|---|---|---|---|---|
| 50 | 1 | 1 | 1 | 1 |
| 700 | 0 | 0 | 1 | 0 |
| 76 | 6 | 6 | 1 | 6 |
| 85 | 1 | 1(50), 2 | 2 | 2 |
| 92 | 1 | 1,2 occupied, 3 | 3 | 3 |
| 73 | 3 | 3(92), 4 | 2 | 4 |
| 101 | 3 | 3,4 occupied, 5 | 3 | 5 |

Final table (index 0..6): `700, 50, 85, 92, 73, 101, 76`. Total probes $=1+1+1+2+3+2+3=13$. Notice the cluster at 1–5: keys with hash 1 and 3 interfered with each other (primary clustering).

### 4.2 Linear vs quadratic

**Worked example 4.** $m=10$, $h(k)=k\bmod10$, insert $12,22,32,42,15,25$.

*Linear probing* ($h+i$): 12→2; 22→2,3; 32→2,3,4; 42→2,3,4,5; 15→5,6; 25→5,6,7.
Table: idx 2:12, 3:22, 4:32, 5:42, 6:15, 7:25. Probes $1+2+3+4+2+3=15$.

*Quadratic probing* ($h+i^2$): 12→2; 22→2,3; 32→2,3,**6** ($2+4=6$ is checked as $i=2$; wait $i=1$ gives 3, $i=2$ gives 6); 42→2,3,6,**1** ($i=3$: $2+9=11\to1$); 15→5; 25→5,6($5+1$),9 ($5+4$).
Table: idx 1:42, 2:12, 3:22, 5:15, 6:32, 9:25. Probes $1+2+3+4+1+3=14$. (Both runs verified by program.)

Quadratic spread the keys; linear built a block of six. **Quadratic probing can fail:** with $m=10$ and $h=2$ the offsets $\{0,1,4,9,16,25,\ldots\}\bmod10=\{0,1,4,9,6,5\}$ hit only 6 of the 10 slots, so a 7th key hashing to 2 could not be placed even though the table is not full.

### 4.3 Double hashing trace

**Worked example 5.** $m=11$, $h_1(k)=k\bmod11$, $h_2(k)=1+(k\bmod5)$. Insert $22,30,14,41,3,52$.

| Key | $h_1$ | $h_2$ | Sequence | Probes | Slot |
|---|---|---|---|---|---|
| 22 | 0 | 3 | 0 | 1 | 0 |
| 30 | 8 | 1 | 8 | 1 | 8 |
| 14 | 3 | 5 | 3 | 1 | 3 |
| 41 | 8 | 2 | 8 (occ), 10 | 2 | 10 |
| 3 | 3 | 4 | 3 (occ), 7 | 2 | 7 |
| 52 | 8 | 3 | 8, $11\to0$, $14\to3$ (all occ), $17\to6$ | 4 | 6 |

Final: idx 0:22, 3:14, 6:52, 7:3, 8:30, 10:41. (Verified by program.)

### 4.4 Deletion: tombstones

If you simply empty a slot, a later search for a key that was *probed past* it stops early and wrongly reports "absent".

**Worked example 6.** Use the final table of Example 3. Delete 85 (slot 2). Then search 92: $h=1$ → 50 (≠), slot 2 now **empty** → search stops: "not found" (wrong).

Fix: mark slot 2 as **DELETED** (tombstone). Search continues past tombstones (so finds 92 at slot 3); insertion may reuse a tombstone slot. Too many tombstones lengthen searches, so tables are periodically **rehashed**.

## 5. Expected probes and load factor

Assume **uniform hashing** (each key's probe sequence is equally likely to be any permutation).

| Scheme | Unsuccessful search / insert | Successful search |
|---|---|---|
| Chaining | $\alpha$ | $1+\alpha/2$ |
| Uniform open addressing | $\dfrac1{1-\alpha}$ | $\dfrac1\alpha\ln\dfrac1{1-\alpha}$ |
| Linear probing (Knuth) | $\dfrac12\left(1+\dfrac1{(1-\alpha)^2}\right)$ | $\dfrac12\left(1+\dfrac1{1-\alpha}\right)$ |

**Worked example 7.**

| $\alpha$ | Uniform unsucc. | Uniform succ. | Linear unsucc. | Linear succ. |
|---|---|---|---|---|
| 0.5 | 2.00 | 1.39 | 2.50 | 1.50 |
| 0.75 | 4.00 | 1.85 | 8.50 | 2.50 |
| 0.9 | 10.0 | 2.56 | 50.5 | 5.50 |

So open addressing must keep $\alpha$ well below 1 (rehash at about 0.5–0.7); linear probing degrades much faster than double hashing.

**Why unsuccessful $=1/(1-\alpha)$:** each probe hits an occupied slot with probability $\alpha$ (roughly), so the number of probes is geometric: $1+\alpha+\alpha^2+\dots=\frac1{1-\alpha}$.

**Rehashing/resizing:** when $\alpha$ passes a threshold, allocate a table of about twice the size and reinsert everything. Cost $O(n)$ but **amortised $O(1)$** per insert.

## 6. Chaining vs open addressing

| Aspect | Chaining | Open addressing |
|---|---|---|
| Max $\alpha$ | any ($>1$ ok) | $\le1$ |
| Deletion | easy | tombstones |
| Extra memory | pointers | none, but table must be sparse |
| Cache | poorer | better (linear probing) |
| Clustering | none | yes (linear, quadratic) |

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Load factor | $\alpha=n/m$ | all expected-cost questions |
| Division hash | $k\bmod m$, prime $m$ | default |
| Chaining unsuccessful / successful | $\alpha$ / $1+\alpha/2$ | chaining questions |
| Uniform OA unsuccessful | $1/(1-\alpha)$ | open addressing |
| Uniform OA successful | $\frac1\alpha\ln\frac1{1-\alpha}$ | open addressing |
| Linear probe | $h(k)+i$; clustering primary | traces |
| Quadratic | $h(k)+c_1i+c_2i^2$; may miss slots | traces |
| Double hashing | $h_1+ih_2$, $h_2\ne0$ | traces |
| Universal family | $\Pr[h(x)=h(y)]\le1/m$ | theory |
| Worst-case lookup | $\Theta(n)$ | always state |

## GATE traps

- **Count probes, not just placements.** A key that hops over 2 occupied slots costs 3 probes.
- **Insert in the given order**; the final table depends on it.
- **Quadratic probing may never place a key** even with free slots; GATE sometimes asks "how many keys can be inserted".
- **Never "just empty" a deleted slot** in open addressing; use a tombstone and note that search must skip it.
- **$h_2(k)=0$ is fatal** (infinite loop at the same slot); the step must also be coprime to $m$ so all slots are visited.
- **Load factor $>1$ is impossible with open addressing** and perfectly fine with chaining.
- **Chaining insert is $O(1)$ at the head**, but insertion with duplicate check costs a chain scan.
- **Average successful search under chaining counts the key itself:** $1+\alpha/2$ (not $\alpha/2$); unsuccessful is $\alpha$ comparisons (or $1+\alpha$ if you count the slot access; read the question).
- **Worst-case hashing is $\Theta(n)$**, whatever the function, if all keys collide.
- **Sorting-based dictionaries** (BST, sorted array) give worst-case $O(\log n)$ and ordered traversal; hashing gives no order, and range queries and min/max are slow.

## Connections

- [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) — the table is an array indexed by the hash value.
- [Linked lists](../07-data-structures/linked-lists.md) — chains are linked lists.
- [Complexity reference](../07-data-structures/complexity-reference.md) — dictionary operation costs compared with BSTs.
- [Probability basics](../02-probability-statistics/probability-basics.md) — birthday paradox: with $m$ slots, collisions appear after about $\sqrt m$ keys; expected chain lengths via indicator variables.
- [Searching and sorting](searching-and-sorting.md) — hashing is the "expected $O(1)$" alternative to binary search.
- [Number representation](../09-digital-logic/number-representation-and-arithmetic.md) — bit-level hashing and mod-power-of-two pitfalls.
- [File organization and indexing](../14-databases/file-organization-and-indexing.md) — static, extendible and linear hashing of disk pages; hash joins.
- [Lexical analysis](../12-compiler-design/lexical-analysis.md) — symbol tables are hash tables.

## Practice

**Q1 (MCQ).** A hash table of size 10 uses $h(k)=k\bmod10$ and linear probing. After inserting $31,42,22,13$ (in that order), where is 13?

<details><summary>Answer</summary>

**Answer:** slot 4  
**Solution:** 31→1; 42→2; 22→2 occupied→3; 13→3 occupied→4. 

</details>

**Q2 (NAT).** Table size 7, $h(k)=k\bmod7$, linear probing. Keys $50,700,76,85,92,73,101$ inserted in order. What is the total number of probes (counting every examined slot)?

<details><summary>Answer</summary>

**Answer:** 13  
**Solution:** $1+1+1+2+3+2+3=13$ (Example 3).

</details>

**Q3 (NAT).** In a chained hash table with 100 slots and 250 keys, uniformly hashed, what is the expected number of key comparisons for a successful search ($1+\alpha/2$)?

<details><summary>Answer</summary>

**Answer:** 2.25  
**Solution:** $\alpha=2.5$; $1+2.5/2=2.25$.

</details>

**Q4 (NAT).** Under uniform hashing with open addressing, load factor 0.8: expected probes for an unsuccessful search?

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** $1/(1-0.8)=5$.

</details>

**Q5 (MSQ).** Which are true? (A) Linear probing suffers from primary clustering. (B) Double hashing suffers from secondary clustering. (C) With open addressing, load factor can exceed 1. (D) Quadratic probing may fail to find an empty slot although the table is not full.

<details><summary>Answer</summary>

**Answer:** A, D  
**Solution:** (B) double hashing avoids both; (C) $n\le m$ so $\alpha\le1$.

</details>

**Q6 (MCQ).** Table size 11, $h_1(k)=k\bmod11$, $h_2(k)=1+(k\bmod5)$, double hashing. Keys $22,30,14,41$ are inserted. Where does 41 go?

<details><summary>Answer</summary>

**Answer:** slot 10  
**Solution:** $h_1(41)=8$ is occupied by 30. $h_2(41)=1+1=2$, next probe $(8+2)\bmod11=10$ is free.

</details>

**Q7 (MCQ).** Using open addressing with linear probing, a table has keys at slots 3(A, $h=3$), 4(B, $h=3$), 5(C, $h=4$). Key B is deleted by emptying slot 4. Searching for C ($h(C)=4$) now…

<details><summary>Answer</summary>

**Answer:** Fails to find C.  
**Solution:** $h(C)=4$ is empty so the search stops immediately ("not found") — wrongly, since C sits at 5 because B was there. A tombstone avoids this (here C's own probe starts at 4, the slot B vacated).

</details>

**Q8 (NAT).** A hash table of size $m=13$ with $h(k)=k\bmod13$ and quadratic probing $h(k)+i^2$. Keys $5,18,31$ (all hash to 5) are inserted. In which slots do they land?

<details><summary>Answer</summary>

**Answer:** 5, 6, 9  
**Solution:** 5→5. 18: 5 occupied, $5+1=6$ free. 31: 5 occ, 6 occ, $5+4=9$ free.

</details>
