# CPU Scheduling

> **Paper:** CS · **Priority:** P0 · **Plan topics:** CPU scheduling
> **Prerequisites:** [Processes, threads, system calls](processes-threads-syscalls.md) · **Leads to:** [Synchronization](synchronization.md) (priority inversion) · [Deadlocks](deadlocks.md)

## Quick glance

- Definitions: **TAT = CT - AT**, **WT = TAT - BT**, **RT = (first time on CPU) - AT**. Throughput = processes/time. CPU utilisation = busy time / total time.
- **FCFS:** non-pre-emptive, simple, **convoy effect**, can give a long average wait.
- **SJF (non-pre-emptive)** and **SRTF (pre-emptive SJF)**: give the **minimum average waiting time** (SJF among non-pre-emptive when all arrive together; SRTF overall). Need burst prediction. Can **starve** long jobs.
- **Priority:** pre-emptive or not; **starvation** of low priority fixed by **aging**.
- **Round Robin:** FCFS + time quantum q. Large q -> FCFS; tiny q -> overhead dominates. Worst wait for a ready process = (n - 1) q. With switch overhead s: utilisation = q / (q + s).
- **HRRN:** response ratio = (W + S)/S, non-pre-emptive, favours short jobs but never starves.
- **Multilevel queue** (fixed queues) vs **multilevel feedback queue** (processes move based on behaviour).
- Tie-break convention: if a new process arrives at the same instant a quantum expires, **the new arrival enters the ready queue first**, then the pre-empted process (state this when you solve).
- #1 trap: waiting time in a pre-emptive schedule is **TAT - BT**, not "time of first start - arrival".

## 1. Why scheduling, and what is measured

**Intuition.** Processes alternate **CPU bursts** and **I/O bursts**. While a process waits for I/O, the CPU should run another one. The short-term scheduler picks which Ready process runs; the dispatcher performs the switch.

**CPU-bound** processes have few long CPU bursts; **I/O-bound** ones have many short ones. A good scheduler gives short bursts priority to keep devices busy.

### Decision points and pre-emption

CPU scheduling decisions occur when a process: (1) goes Running -> Waiting, (2) Running -> Ready (interrupt/quantum end), (3) Waiting -> Ready (I/O done), (4) terminates.

- **Non-pre-emptive:** decisions only at 1 and 4. A process keeps the CPU until it blocks or finishes.
- **Pre-emptive:** also at 2 and 3. Can cause race conditions on shared kernel data and costs more switches.

### Criteria

| Term | Symbol | Meaning |
|---|---|---|
| Arrival time | AT | when the process enters the ready queue |
| Burst time | BT | total CPU time needed |
| Completion time | CT | when it finishes |
| Turnaround time | TAT = CT - AT | total time in the system |
| Waiting time | WT = TAT - BT | time spent in the ready queue (for CPU-only processes) |
| Response time | RT | first CPU allocation time - AT |
| Throughput | | processes completed per unit time |
| CPU utilisation | | fraction of time CPU is busy doing useful work |

**Goal:** maximise utilisation and throughput; minimise TAT, WT, RT. **Waiting time is the quantity that SJF/SRTF minimise on average.**

## 2. First-Come First-Served (FCFS)

Ready queue is a FIFO. Non-pre-emptive.

**Convoy effect:** short processes queue behind one long CPU-bound process, so devices and the CPU are poorly utilised.

**Worked example.** Used for all of Sections 2-6 (all times in ms):

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 8 |
| P2 | 1 | 4 |
| P3 | 2 | 9 |
| P4 | 3 | 5 |

```text
FCFS Gantt:  | P1 (0-8) | P2 (8-12) | P3 (12-21) | P4 (21-26) |
```

| Process | CT | TAT = CT - AT | WT = TAT - BT |
|---|---|---|---|
| P1 | 8 | 8 | 0 |
| P2 | 12 | 11 | 7 |
| P3 | 21 | 19 | 10 |
| P4 | 26 | 23 | 18 |

Average TAT = 61/4 = **15.25**; average WT = 35/4 = **8.75**.

## 3. Shortest Job First (SJF) and Shortest Remaining Time First (SRTF)

**Idea.** Pick the process with the smallest next CPU burst.

- **SJF:** non-pre-emptive; at each completion pick the shortest among arrived processes.
- **SRTF:** pre-emptive SJF; whenever a process arrives, if its burst is **less than the remaining time of the running process** it pre-empts.

**Optimality.** SJF gives the minimum average waiting time among non-pre-emptive algorithms (for processes available together), by an exchange argument: swapping a longer job ahead of a shorter adjacent one increases the shorter's wait by more than it decreases the longer's. SRTF is optimal over all schedules (pre-emption allowed). Neither is implementable exactly, since the next burst is unknown.

**Predicting the next burst (exponential averaging):**

$$\tau_{n+1} = \alpha\, t_n + (1-\alpha)\,\tau_n,\quad 0\le\alpha\le 1$$

where t_n is the actual n-th burst and tau_n the prediction. alpha = 0: history ignored; alpha = 1: only the last burst counts. *Example:* alpha = 0.5, tau_0 = 10, actual bursts 6, 4, 6, 4: tau_1 = 8, tau_2 = 6, tau_3 = 6, tau_4 = 5.

**SJF (non-pre-emptive) on the same data.** At t = 8, ready: P2 (4), P3 (9), P4 (5) -> P2, then P4, then P3.

```text
SJF Gantt:  | P1 (0-8) | P2 (8-12) | P4 (12-17) | P3 (17-26) |
```

| Process | CT | TAT | WT |
|---|---|---|---|
| P1 | 8 | 8 | 0 |
| P2 | 12 | 11 | 7 |
| P3 | 26 | 24 | 15 |
| P4 | 17 | 14 | 9 |

Average TAT = 57/4 = **14.25**; average WT = 31/4 = **7.75**.

**SRTF on the same data.**

- t = 0: P1 starts. t = 1: P2 arrives (4) < P1 remaining (7) -> pre-empt, run P2.
- t = 2: P3 (9) arrives; P2 remaining 3 -> continue. t = 3: P4 (5) arrives; P2 remaining 2 -> continue.
- t = 5: P2 done. Ready: P1 (7 left), P3 (9), P4 (5) -> P4 until 10. Then P1 (7) until 17, then P3 until 26.

```text
SRTF Gantt:  | P1 (0-1) | P2 (1-5) | P4 (5-10) | P1 (10-17) | P3 (17-26) |
```

| Process | CT | TAT | WT |
|---|---|---|---|
| P1 | 17 | 17 | 9 |
| P2 | 5 | 4 | 0 |
| P3 | 26 | 24 | 15 |
| P4 | 10 | 7 | 2 |

Average TAT = 52/4 = **13.0**; average WT = 26/4 = **6.5** (the smallest of FCFS 8.75, SJF 7.75, SRTF 6.5).

**Another SRTF example** (checks the pre-emption rule): P1 (AT 0, BT 7), P2 (2, 4), P3 (4, 1), P4 (5, 4).

```text
| P1 0-2 | P2 2-4 | P3 4-5 | P2 5-7 | P4 7-11 | P1 11-16 |
```

- t = 2: P2 (4) < P1 remaining 5 -> pre-empt. t = 4: P3 (1) < P2 remaining 2 -> pre-empt. t = 5: P3 done; P4 (4) arrives, ready: P2 (2), P4 (4), P1 (5) -> P2 (2) to t = 7, then P4, then P1.
- CT: P1 16, P2 7, P3 5, P4 11. TAT = 16, 5, 1, 6 (avg 7.0). WT = 9, 1, 0, 2 (avg **3.0**). Non-pre-emptive SJF on the same set gives WT = 0, 6, 3, 7 = avg 4.0.

**Drawback: starvation** of long processes if short ones keep arriving.

## 4. Priority scheduling

Each process has a priority (convention in GATE: **a smaller number = higher priority** unless stated). Non-pre-emptive: highest priority among arrived runs to completion. Pre-emptive: a newly arrived higher-priority process pre-empts. SJF is priority scheduling with priority = predicted burst.

**Problem: starvation (indefinite blocking).** Low-priority processes may never run. **Solution: aging** (raise the priority of a waiting process over time).

**Worked example** (smaller number = higher priority):

| Process | AT | BT | Priority |
|---|---|---|---|
| P1 | 0 | 10 | 3 |
| P2 | 1 | 1 | 1 |
| P3 | 2 | 2 | 4 |
| P4 | 3 | 1 | 5 |
| P5 | 4 | 5 | 2 |

**Non-pre-emptive:** P1 runs 0-10. Then ready: P2 (p1), P3 (p4), P4 (p5), P5 (p2): P2 10-11, P5 11-16, P3 16-18, P4 18-19.
CT: 10, 11, 18, 19, 16. TAT: 10, 10, 16, 16, 12. WT: 0, 9, 14, 15, 7 -> avg WT = 45/5 = **9.0**.

**Pre-emptive:** P1 0-1; P2 (p1) arrives at 1 -> runs 1-2; P1 resumes 2-4 (P3 arrives at 2 with p4 and P4 at 3 with p5, both lower than P1's p3); P5 (p2) arrives at 4 -> pre-empts, runs 4-9; P1 9-16; P3 16-18; P4 18-19.

```text
| P1 0-1 | P2 1-2 | P1 2-4 | P5 4-9 | P1 9-16 | P3 16-18 | P4 18-19 |
```

CT: P1 16, P2 2, P3 18, P4 19, P5 9. TAT: 16, 1, 16, 16, 5. WT: 6, 0, 14, 15, 0 -> avg WT = 35/5 = **7.0**.

## 5. Round Robin (RR)

FCFS with a **time quantum q**. The running process is pre-empted after q if not finished and goes to the **back** of the ready queue. Designed for time-sharing; response time is good.

**Effect of q:**

| q | Behaviour |
|---|---|
| very large (>= longest burst) | degenerates to FCFS |
| very small | looks like processor sharing but **context-switch overhead** dominates |
| rule of thumb | ~80% of CPU bursts should be shorter than q |

**Bound:** with n processes and quantum q (ignoring switch time), no process waits more than **(n - 1) q** for its next turn.

**CPU utilisation with switch overhead s** (every process uses its full quantum): **U = q / (q + s)**. For q = 4 ms, s = 1 ms: U = 80%.

**Worked example 1: data of Section 2, q = 3.** Ready queue evolution (new arrival enters before the pre-empted process on ties):

| Time | Event | Queue after (front -> back) |
|---|---|---|
| 0-3 | P1 runs 3 (rem 5); P2, P3 arrived at 1, 2; P4 arrives at 3 | P2, P3, P4, P1 |
| 3-6 | P2 runs 3 (rem 1) | P3, P4, P1, P2 |
| 6-9 | P3 runs 3 (rem 6) | P4, P1, P2, P3 |
| 9-12 | P4 runs 3 (rem 2) | P1, P2, P3, P4 |
| 12-15 | P1 runs 3 (rem 2) | P2, P3, P4, P1 |
| 15-16 | P2 runs 1, **done** | P3, P4, P1 |
| 16-19 | P3 runs 3 (rem 3) | P4, P1, P3 |
| 19-21 | P4 runs 2, **done** | P1, P3 |
| 21-23 | P1 runs 2, **done** | P3 |
| 23-26 | P3 runs 3, **done** | |

```text
| P1 0-3 | P2 3-6 | P3 6-9 | P4 9-12 | P1 12-15 | P2 15-16 | P3 16-19 | P4 19-21 | P1 21-23 | P3 23-26 |
```

| Process | CT | TAT | WT | RT |
|---|---|---|---|---|
| P1 | 23 | 23 | 15 | 0 |
| P2 | 16 | 15 | 11 | 2 |
| P3 | 26 | 24 | 15 | 4 |
| P4 | 21 | 18 | 13 | 6 |

Average TAT = 80/4 = **20.0**; average WT = 54/4 = **13.5**. RR has a higher average TAT/WT than SJF but a better response time (average RT = 3).

**Worked example 2: counting context switches.** P1 = 5, P2 = 3, P3 = 8 (all at t = 0), q = 3.
Slices: P1 (0-3), P2 (3-6, done), P3 (6-9), P1 (9-11, done), P3 (11-14), P3 (14-16, done). The CPU is handed from one process to a **different** one at: P1->P2, P2->P3, P3->P1, P1->P3 = **4 context switches** (P3 -> P3 is not a switch). Counting rule: count transitions between consecutive slices of different processes; exclude the very first dispatch and (usually) the last completion unless the question says otherwise.

**Worked example 3: effect of q.** Five processes A(0,5), B(1,3), C(2,8), D(3,2), E(4,4):

| q | Gantt | avg WT | avg TAT |
|---|---|---|---|
| 2 | A0-2 B2-4 C4-6 A6-8 D8-10 E10-12 B12-13 C13-15 A15-16 E16-18 C18-20 C20-22 | 9.4 | 13.8 |
| 4 | A0-4 B4-7 C7-11 D11-13 E13-17 A17-18 C18-22 | 9.0 | 13.4 |

(Both computed with the "new arrival before pre-empted process" rule. Completion times for q = 4: B 7, D 13, E 17, A 18, C 22.)

## 6. Highest Response Ratio Next (HRRN)

Non-pre-emptive. When the CPU frees up, compute for each ready process

$$\text{Response ratio} = \frac{W + S}{S}$$

(W = waiting time so far, S = service/burst time) and run the highest. A long-waiting process's ratio grows, so **no starvation**, while short jobs still get a boost.

**Worked example:** A(0,3), B(2,6), C(4,4), D(6,5), E(8,2).

- t = 0: only A; A runs 0-3. t = 3: only B has arrived (C arrives at 4) -> B runs 3-9.
- t = 9: ready C (W = 5, S = 4): (5+4)/4 = 2.25; D (W = 3, S = 5): 8/5 = 1.6; E (W = 1, S = 2): 3/2 = 1.5 -> **C** runs 9-13.
- t = 13: D (W = 7, S = 5): 12/5 = 2.4; E (W = 5, S = 2): 7/2 = 3.5 -> **E** runs 13-15; then D 15-20.

```text
| A 0-3 | B 3-9 | C 9-13 | E 13-15 | D 15-20 |
```

TAT: A 3, B 7, C 9, D 14, E 7 (avg 8.0). WT: 0, 1, 5, 9, 5 (avg **4.0**). For comparison FCFS avg WT = 4.6, SJF avg WT = 3.6.

## 7. Multilevel queue and multilevel feedback queue

**Multilevel queue (MLQ).** The ready queue is split into permanent queues (e.g. system, interactive, batch). Each has its own algorithm (e.g. RR for interactive, FCFS for batch). Between queues: **fixed priority** (lower queues run only if higher are empty -> starvation) or **time slicing** (e.g. 80%/20%). Processes **do not move** between queues.

**Multilevel feedback queue (MLFQ).** Processes move between queues based on behaviour. Parameters: number of queues, algorithm per queue, rule to **demote** (used its whole quantum -> CPU-bound), rule to **promote** (waited too long -> aging), which queue a new process enters.

**Worked example.** Q0: RR q = 8; Q1: RR q = 16; Q2: FCFS. A new process enters Q0.

A job needing a CPU burst of 25 ms: runs 8 in Q0 (does not finish -> demoted), 16 in Q1 (demoted), then the last 1 ms in Q2. Total 8 + 16 + 1 = 25. A job with a 5 ms burst finishes in Q0. This favours short/I/O-bound jobs and gradually pushes CPU-bound work down.

## 8. I/O-bound plus CPU-bound mixes: utilisation

When processes alternate CPU and I/O bursts, utilisation can be computed from the timeline.

**Worked example (FCFS, I/O device handles one request at a time per process, both at t = 0):**
P1: CPU 3, I/O 2, CPU 1. P2: CPU 2, I/O 4, CPU 2.

```text
CPU: | P1 0-3 | P2 3-5 | P1 5-6 | idle 6-9 | P2 9-11 |
I/O:            P1 3-5    P2 5-9
```

P1: I/O 3-5, ready at 5 but CPU busy with P2 until 5 -> runs 5-6. P2: I/O 5-9, runs 9-11. Busy time = 3 + 2 + 1 + 2 = 8, total 11, **utilisation = 8/11 = 72.7%**. Idle 6-9.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| TAT, WT, RT | CT - AT; TAT - BT; first run - AT | every scheduling numeric |
| Avg WT optimal | SJF (non-pre-emptive) / SRTF (overall) | "which minimises average wait" |
| RR waiting bound | (n - 1) q | worst-case wait per turn |
| RR utilisation | q / (q + s) | quantum vs overhead |
| Exponential average | tau_(n+1) = alpha t_n + (1 - alpha) tau_n | burst prediction |
| HRRN ratio | (W + S) / S | HRRN selection |
| Convoy effect | FCFS | explanation questions |
| Starvation / fix | SJF, SRTF, priority / aging | traps |

| Algorithm | Pre-emptive? | Starvation? | Notes |
|---|---|---|---|
| FCFS | No | No | convoy effect |
| SJF | No | Yes (long jobs) | optimal avg WT (non-preemptive) |
| SRTF | Yes | Yes | optimal avg WT overall |
| Priority | Both | Yes | aging fixes |
| RR | Yes | No | good response, depends on q |
| HRRN | No | No | ratio grows with waiting |
| MLQ | depends | Yes (fixed priority) | static queues |
| MLFQ | Yes | Avoidable (promotion) | most general |

## GATE traps

- **Waiting time in a pre-emptive schedule** is TAT - BT (total ready-queue time over all stints), **not** "start - arrival".
- **Idle CPU gaps:** if the ready queue is empty the clock jumps to the next arrival; do not start a process before its arrival time.
- **Tie rules:** state what happens for equal burst/priority (smaller arrival/process id first) and for the arrival-at-quantum-expiry tie (new arrival first).
- **SRTF pre-empts only if the new burst is strictly less than the remaining time** of the running process (equal -> no pre-emption).
- **Priority numbering:** check whether a smaller or larger number means higher priority.
- **Context switch counting in RR:** same process continuing is not a switch; count (or not) the switch cost as the question states.
- **"Minimum average waiting time"** -> SJF/SRTF; **"no starvation and fair"** -> RR/HRRN/FCFS.
- **Throughput and utilisation** change with q: RR with tiny q lowers utilisation.
- MLQ vs MLFQ: only in MLFQ do processes **move** between queues.
- FCFS in a mix with I/O: a long CPU-bound process holds the CPU, and I/O devices sit idle (convoy effect).

## Connections

- [Processes, threads, system calls](processes-threads-syscalls.md) — the Ready queue, dispatcher and context switch used here.
- [Synchronization](synchronization.md) — **priority inversion** (a low-priority process holding a lock a high-priority one needs) interacts with priority scheduling; priority inheritance fixes it.
- [Deadlocks](deadlocks.md) — starvation vs deadlock distinction.
- [File systems and disk scheduling](file-systems-and-disk-scheduling.md) — disk scheduling (SSTF ~ SJF, starvation, SCAN ~ elevator) is the same idea applied to the disk head.
- [Pipelining](../10-computer-organization/pipelining.md) — context switching and throughput vs latency trade-offs.
- [Algorithms: greedy](../08-algorithms/greedy-algorithms.md) — SJF optimality is an exchange argument, like many greedy proofs.
- [Computer networks: data link layer](../15-computer-networks/data-link-layer.md) — round-robin polling and time-division ideas.

## Practice

**Q1 (NAT).** Four processes arrive at t = 0 with burst times 6, 8, 7, 3. What is the average waiting time under (a) SJF and (b) FCFS in the given order?

<details><summary>Answer</summary>

**Answer:** (a) 7, (b) 10.25.
**Solution:** (a) Order 3, 6, 7, 8: waits 0, 3, 9, 16, sum 28, avg 7. (b) Order 6, 8, 7, 3: waits 0, 6, 14, 21, sum 41, avg 10.25.

</details>

**Q2 (NAT).** P1 (AT 0, BT 7), P2 (2, 4), P3 (4, 1), P4 (5, 4). Average waiting time under SRTF?

<details><summary>Answer</summary>

**Answer:** 3.0.
**Solution:** Gantt: P1 0-2, P2 2-4, P3 4-5, P2 5-7, P4 7-11, P1 11-16. CT: P1 16, P2 7, P3 5, P4 11. WT = CT - AT - BT: 16-0-7 = 9, 7-2-4 = 1, 5-4-1 = 0, 11-5-4 = 2. Sum 12, avg 3.0.

</details>

**Q3 (MCQ).** Which scheduling algorithm can lead to starvation?
(A) FCFS (B) Round Robin (C) SRTF (D) HRRN

<details><summary>Answer</summary>

**Answer:** (C).
**Solution:** Continuous arrival of shorter jobs keeps postponing a long job. FCFS and RR serve everyone in bounded time; HRRN's ratio rises as a process waits.

</details>

**Q4 (NAT).** RR with q = 4 ms and a context-switch time of 1 ms. All processes always use the full quantum. What is the CPU utilisation (percent)?

<details><summary>Answer</summary>

**Answer:** 80.
**Solution:** Each cycle has 4 ms useful work and 1 ms overhead: U = 4/(4+1) = 0.8.

</details>

**Q5 (NAT).** Processes P1 = 5, P2 = 3, P3 = 8 ms arrive at 0. RR, q = 3. How many context switches occur (exclude the first dispatch and the final completion)?

<details><summary>Answer</summary>

**Answer:** 4.
**Solution:** Slices: P1, P2, P3, P1, P3, P3 (last two are the same process). Different consecutive pairs: P1->P2, P2->P3, P3->P1, P1->P3 = 4.

</details>

**Q6 (MSQ).** Which are true?
(A) SJF (non-pre-emptive) minimises average waiting time for a given set of processes available at the same time.
(B) RR with a very large quantum behaves like FCFS.
(C) In MLFQ, a CPU-bound process eventually moves to a lower-priority queue.
(D) HRRN is pre-emptive.

<details><summary>Answer</summary>

**Answer:** (A), (B), (C).
**Solution:** (D) is false: HRRN is non-pre-emptive (ratio computed when the CPU is freed). (C): using the whole quantum repeatedly leads to demotion.

</details>

**Q7 (NAT).** CPU burst predictions use alpha = 0.5, tau_0 = 10 ms. The actual bursts are 6, 4, 6, 4. What is tau_4?

<details><summary>Answer</summary>

**Answer:** 5.
**Solution:** tau_1 = 0.5*6 + 0.5*10 = 8; tau_2 = 0.5*4 + 0.5*8 = 6; tau_3 = 0.5*6 + 0.5*6 = 6; tau_4 = 0.5*4 + 0.5*6 = 5.

</details>

**Q8 (NAT).** Pre-emptive priority scheduling (smaller number = higher priority): P1 (AT 0, BT 10, prio 3), P2 (1, 1, 1), P3 (2, 2, 4), P4 (3, 1, 5), P5 (4, 5, 2). What is the average waiting time?

<details><summary>Answer</summary>

**Answer:** 7.0.
**Solution:** Gantt: P1 0-1, P2 1-2, P1 2-4, P5 4-9, P1 9-16, P3 16-18, P4 18-19. CT: P1 16, P2 2, P3 18, P4 19, P5 9. WT = CT - AT - BT: 6, 0, 14, 15, 0. Sum 35, avg 7.0.

</details>
