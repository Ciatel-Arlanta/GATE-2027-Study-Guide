# Operating Systems checkpoint

**Q1.** Arrival 0, burst 5, completion 12. Turnaround and waiting?
<details><summary>Answer</summary> Turnaround 12; waiting 7 in the one-burst model.</details>

**Q2.** Page size 8 KiB. Offset bits?
<details><summary>Answer</summary> 13.</details>

**Q3.** Can FIFO page faults increase when frame count increases?
<details><summary>Answer</summary> Yes, Belady's anomaly.</details>

**Q4.** Need vector if Max=(7,5), Allocation=(2,1)?
<details><summary>Answer</summary> (5,4).</details>

**Q5.** Name the three critical-section requirements.
<details><summary>Answer</summary> Mutual exclusion, progress, bounded waiting.</details>

**Q6.** Is unsafe state the same as deadlock?
<details><summary>Answer</summary> No; unsafe means no safe completion sequence is guaranteed.</details>

**Q7.** Compare linked and indexed file allocation for random access.
<details><summary>Answer</summary> Indexed supports direct block lookup; linked requires following the chain.</details>

**Q8.** Which disk policy may starve distant requests?
<details><summary>Answer</summary> SSTF.</details>

≥80%: proceed; below 60%: revisit scheduling timelines, page translation, and semaphore ordering.
