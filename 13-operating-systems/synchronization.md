# Synchronization and Concurrency

> **Paper:** CS · **Priority:** P0 · **Plan topics:** critical sections, mutual exclusion, semaphores, monitors, classical synchronisation problems
> **Prerequisites:** [Processes and threads](processes-threads-syscalls.md) · **Leads to:** [Deadlocks](deadlocks.md)
> **PYQ weightage:** Medium · 14 Qs since 2015 (5 since 2021) · DA: 0 · Trend: steady · Most asked: Process Synchronization, Semaphores

## Quick glance

- **Exam focus:** Semaphore value tracing and concurrent interleaving (NAT / MCQ), verifying Critical Section criteria (Mutual Exclusion, Progress, Bounded Waiting for software and hardware locks), and deadlock prevention in classical problems (Producer-Consumer, Dining Philosophers).
- A **race condition** occurs when multiple concurrent processes access shared mutable state and the final outcome depends on the execution interleaving.
- A **critical section (CS)** is a code segment accessing shared resources. Any valid solution must satisfy three requirements:
  1. **Mutual Exclusion (mandatory):** at most one process in CS at any time.
  2. **Progress (mandatory):** if CS is free, only processes competing to enter decide who enters next; selection cannot be postponed indefinitely (processes in remainder section cannot block others).
  3. **Bounded Waiting (mandatory):** a finite bound on bypasses after a process requests entry (prevents starvation).
- **Strict alternation (`turn` variable):** satisfies Mutual Exclusion and Bounded Waiting, but **violates Progress**.
- **Peterson's algorithm:** complete 2-process software solution satisfying all 3 criteria using shared `flag[2]` and `turn`.
- **Hardware instructions (`TestAndSet`, `CompareAndSwap`):** atomic primitives ensuring Mutual Exclusion and Progress; bounded waiting requires an explicit queue.
- **Counting semaphore $S$:** integer accessed atomically via `wait(S)` ($P$, down) and `signal(S)` ($V$, up). When $S < 0$, $|S|$ equals the number of blocked processes.
- **Producer-Consumer ordering trap:** must execute `wait(empty)` before `wait(mutex)`. Inverting them causes **deadlock** when the buffer is full.
- **Monitors:** language construct with compiler-enforced mutual exclusion. Condition variables have **no memory**: a `signal()` with no waiting threads is lost.

## 1. Concurrency, Race Conditions, and Critical Section [PYQ: High]

When concurrent processes or threads share mutable state without coordination, arbitrary preemption causes lost updates and data corruption:

```text
P1: LOAD R1, x (x=5) ---> Preempted ---> P2: LOAD R2, x; INCR R2; STORE x, R2 (x=6)
---> P1 resumes: INCR R1; STORE x, R1 (x=6)  [Lost update: expected x=7]
```

### 1.1 Critical Section Structure and The Three Criteria

Every cooperating process follows: **Entry Section $\to$ Critical Section $\to$ Exit Section $\to$ Remainder Section**.

1. **Mutual Exclusion:** If process $P_i$ executes in its critical section, no other process can execute in its critical section.
2. **Progress:** If no process is in CS and some wish to enter, only processes **not in their remainder section** can participate in deciding who enters next, and this selection cannot be postponed indefinitely.
3. **Bounded Waiting:** A bound exists on the number of times other processes may enter CS after a process requests entry, before that request is granted.

### Worked Example 1 (Min/Max Counter Value Under Preemption)

Two threads $T_1$ and $T_2$ share `counter = 0`. Each executes a loop $N = 5$ times: `LOAD R, counter; INCR R; STORE counter, R;`
Determine the maximum and minimum possible final values of `counter`.

- **Maximum value:** $2 \times N = 10$ (serial, non-conflicting execution).
- **Minimum value:** **2** (achieved via overlapping stale writes for any $N \ge 2$):
  1. $T_1$ executes iteration 1: reads `counter = 0` into $R_1$. Preempted before incrementing.
  2. $T_2$ completes iterations 1 through 4 cleanly: `counter = 4`.
  3. $T_1$ resumes iteration 1: stores $R_1 = 0 + 1 = 1$ into `counter`. Now `counter = 1`.
  4. $T_2$ starts iteration 5: reads `counter = 1` into $R_2$. Preempted before storing.
  5. $T_1$ runs its remaining 4 iterations (2 through 5) cleanly: `counter` becomes $1 + 4 = 5$.
  6. $T_2$ resumes iteration 5: writes $R_2 = 1 + 1 = 2$ into `counter`. Final value = **2**.

## 2. Software and Hardware Synchronization Primitives [PYQ: High]

### 2.1 Software Protocols (2 Processes: $P_0$ and $P_1$)

- **Strict Alternation (`turn` variable):** `while (turn != i); /* CS */ turn = j;`
  Satisfies Mutual Exclusion and Bounded Waiting. **Violates Progress**: if $P_0$ finishes and enters its remainder section, $P_1$ cannot re-enter CS because $P_0$ will not set `turn = 1` until it decides to run again.
- **Flag array only:** `flag[i] = true; while (flag[j]); /* CS */ flag[i] = false;`
  If both set flags before checking, both spin forever. **Violates Progress (Deadlock)**.
- **Peterson's Algorithm:**
  ```c
  flag[i] = true;
  turn = j;
  while (flag[j] && turn == j); /* Entry */
  /* Critical Section */
  flag[i] = false;              /* Exit */
  ```
  Satisfies all three conditions:
  - *Mutual Exclusion:* Requires `flag[0] == true`, `flag[1] == true`, and `turn == 0 && turn == 1`, impossible.
  - *Progress:* The `turn` variable resolves ties immediately if both request entry.
  - *Bounded Waiting:* Process $P_j$ resets `flag[j] = false` upon exit, letting $P_i$ enter within 1 turn.

### 2.2 Hardware Instructions (`TestAndSet`, `CompareAndSwap`)

```c
// Atomic TestAndSet: returns old value and sets target to true
bool TestAndSet(bool *target) { bool rv = *target; *target = true; return rv; }
while (TestAndSet(&lock)); /* Spinlock */
/* CS */
lock = false;
```
- Satisfies **Mutual Exclusion** and **Progress**.
- **Bounded Waiting is NOT guaranteed** with basic spinlocks: an unlucky process can be starved by faster competitors.

### Worked Example 2 (Testing Critical Section Properties)

Analyze the following protocol using shared `boolean flag[2] = {false, false}`:
```c
// Process Pi (Pj is other):
flag[i] = true;
while (flag[j]) { flag[i] = false; sleep(1); flag[i] = true; }
/* CS */
flag[i] = false;
```
- **Mutual Exclusion holds:** A process enters CS only when the other's flag is false.
- **Progress is VIOLATED (Livelock):** If both set flags simultaneously, both detect `flag == true`, both reset to false, sleep, and repeat in lockstep. Neither enters CS, postponing selection indefinitely.

## 3. Semaphores and Value Tracing [PYQ: High]

A counting semaphore $S$ is an integer accessed atomically via `wait(S)` ($P$) and `signal(S)` ($V$):

```c
void wait(semaphore *S) {
    S->value--;
    if (S->value < 0) { add calling process to S->queue; block(); }
}
void signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) { remove process P from S->queue; wakeup(P); }
}
```

- **Invariant:**
  - $S > 0$: $S$ processes can execute `wait(S)` without blocking.
  - $S = 0$: no available permits, no blocked processes.
  - $S < 0$: exactly $|S|$ processes are blocked in the semaphore queue.
- **Value formula:** $S_{\text{final}} = S_{\text{init}} - N_{\text{wait}} + N_{\text{signal}}$.

### Worked Example 3 (Semaphore Value Tracing NAT)

A counting semaphore $S$ is initialized to 8. During execution, 22 `wait(S)` and 11 `signal(S)` operations are requested. Find: (a) final value of $S$, (b) number of blocked processes.
- Final value: $S_{\text{final}} = 8 - 22 + 11 = -3$.
- Blocked processes: $|S_{\text{final}}| = |-3| = 3$.

### Worked Example 4 (Thread Interleaving with Semaphores — GATE 2024 Style)

Threads $T_1, T_2$ share semaphores $S_1 = 1$, $S_2 = 0$, and `x = 0`:
```c
// T1: wait(S1); x = x+1; print(x); wait(S2); signal(S1);
// T2: wait(S1); x = x+1; print(x); signal(S2); signal(S1);
```
- **If $T_1$ runs first:** $T_1$ acquires $S_1$ ($S_1=0$), increments and prints `1`. Then $T_1$ calls `wait(S2)` and **blocks** holding $S_1$. $T_2$ calls `wait(S1)` and **blocks**. **Deadlock:** $T_1$ prints 1, $T_2$ never prints.
- **If $T_2$ runs first:** $T_2$ acquires $S_1$ ($S_1=0$), prints `1`, signals $S_2$ ($S_2=1$), signals $S_1$ ($S_1=1$). $T_1$ acquires $S_1$, prints `2`, passes `wait(S2)` cleanly, signals $S_1$. **Output: 1 then 2.**

## 4. Classical Synchronization Problems [PYQ: Medium]

### 4.1 Bounded-Buffer (Producer-Consumer)

Shared state: buffer of size $N$, `mutex = 1`, `empty = N`, `full = 0`.
- **Producer:** `wait(empty); wait(mutex); add_item(); signal(mutex); signal(full);`
- **Consumer:** `wait(full); wait(mutex); remove_item(); signal(mutex); signal(empty);`
- **Crucial GATE Trap:** If producer executes `wait(mutex)` before `wait(empty)`, when the buffer is full (`empty == 0`), the producer grabs `mutex` and blocks on `empty`. The consumer cannot acquire `mutex` to drain items $\implies$ **Deadlock**. Always acquire resource counts before the mutex.

### 4.2 Readers-Writers Problem

Shared state: `int readcount = 0; semaphore mutex = 1, wrt = 1;`
- **Writer:** `wait(wrt); write(); signal(wrt);`
- **Reader:** `wait(mutex); readcount++; if (readcount == 1) wait(wrt); signal(mutex); read(); wait(mutex); readcount--; if (readcount == 0) signal(wrt); signal(mutex);`
- The first reader locks `wrt`; subsequent readers enter without blocking writers further. Starvation: continuous reader arrivals can starve writers indefinitely.

### 4.3 Dining Philosophers Problem

5 philosophers share 5 chopsticks (`chopstick[5]`, all initialized to 1). If all pick up their left chopstick simultaneously, all block on their right chopstick (**Deadlock**).
- **Deadlock-free strategies:**
  1. *Resource dimensioning:* allow at most 4 philosophers to sit simultaneously (`semaphore seats = 4`).
  2. *Asymmetric pickup:* odd philosophers pick left then right; even philosophers pick right then left.
  3. *Atomic pickup:* acquire both chopsticks inside a critical section.

### Worked Example 5 (Dining Philosophers Resource Bound)

For $N$ philosophers each needing 2 chopsticks, what is the minimum total chopsticks $M$ required to guarantee deadlock freedom without asymmetric rules?
- Worst-case allocation without eating: each holds 1 chopstick $\implies N \times (2 - 1) = N$.
- By Pigeonhole Principle: $M \ge N + 1$ chopsticks guarantees at least one philosopher gets 2 chopsticks and eats. For $N=5$, $M \ge 6$.

## 5. Monitors and Condition Variables [PYQ: Low]

A **monitor** is an abstract data type where the compiler automatically enforces mutual exclusion: only one thread executes inside the monitor at any instant.
- **Condition variables (`condition x;`):**
  - `x.wait()`: calling thread releases monitor lock and blocks on $x$.
  - `x.signal()`: wakes up one blocked thread. **If no thread is waiting, `signal()` is a no-op (discarded).**
- **Hoare vs Mesa Semantics:**
  - *Hoare (Signal-and-Wait):* Signaler immediately yields monitor to the resumed thread. Condition is guaranteed true: `if (!ready) x.wait();`
  - *Mesa / Hansen (Signal-and-Continue):* Signaler keeps monitor; resumed thread enters ready queue. Condition must be re-checked in a loop: `while (!ready) x.wait();`

## Formulas and facts to memorise

| Item | Formula / Rule | When to use |
|---|---|---|
| Critical Section Criteria | Mutual Exclusion (primary), Progress (primary), Bounded Waiting (fairness) | Verifying synchronization code |
| Final Semaphore Value | $S_{\text{final}} = S_{\text{init}} - N_{\text{wait}} + N_{\text{signal}}$ | Tracing semaphore values |
| Blocked Processes | If $S < 0$, $\text{blocked} = \|S\|$; if $S \ge 0$, $\text{blocked} = 0$ | Finding queue length in counting semaphores |
| Min Counter Value | 2 threads doing $N$ increments each ($N \ge 2$), $\min = 2$ | Race condition questions on shared counter |
| Max Concurrency | Initial semaphore value $K$ allows at most $K$ processes in CS | Resource allocation limits |
| Bounded Buffer Invariant | $\text{empty} + \text{full} = N$ (buffer capacity) | Verifying producer-consumer consistency |
| Deadlock-Free Diners | $M \ge N + 1$ chopsticks guarantees progress for $N$ diners | Sizing shared resources |
| Condition Variable Signal | No-op if no waiter; has no memory | Distinguishing monitors from semaphores |

## GATE traps

- **Progress vs Mutual Exclusion:** Strict alternation (`turn = 1 - turn`) satisfies Mutual Exclusion but **violates Progress** because a process in its remainder section blocks the other.
- **Deadlock in Bounded Buffer:** `wait(mutex); wait(empty);` causes deadlock when the buffer is full. Always acquire the resource counter before the mutex.
- **Hardware Spinlocks and Starvation:** Basic `TestAndSet` loops satisfy Mutual Exclusion and Progress, but **do not guarantee Bounded Waiting** without an explicit ticket queue.
- **Condition Variables Do Not Count:** A signal on an empty condition variable queue is discarded. Never assume a condition signal satisfies a subsequent wait.
- **Counting Semaphore Negative Values:** $S < 0$ indicates $|S|$ waiting processes only in Dijkstra's blocking definition. In POSIX/non-blocking definitions, $S$ does not drop below 0.

## Connections

- [Processes and threads](processes-threads-syscalls.md) — threads share the process address space where race conditions occur.
- [Deadlocks](deadlocks.md) — circular wait and hold-and-wait arise from improper semaphore ordering.
- [CPU scheduling](cpu-scheduling.md) — preemptive scheduling induces race conditions, and synchronization primitives invoke scheduler block/unblock hooks.
- [Transactions and concurrency](../14-databases/transactions-and-concurrency.md) — serializability, two-phase locking (2PL), and lost updates are database analogs of critical sections.
- [Instruction sets and addressing](../10-computer-organization/instruction-sets-and-addressing.md) — atomic instructions (`TestAndSet`, `CompareAndSwap`, Load-Linked/Store-Conditional).

## Practice

**Q1 (NAT, easy).** A counting semaphore $S$ is initialized to 12. Then, 27 `wait` operations and 19 `signal` operations are completed on $S$. What is the final value of $S$?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** $S_{\text{final}} = S_{\text{init}} - N_{\text{wait}} + N_{\text{signal}} = 12 - 27 + 19 = 4$. Since $S > 0$, no processes are blocked.

</details>

**Q2 (MCQ).** In Peterson's algorithm for two processes $P_0$ and $P_1$, if $P_0$'s entry section is rewritten as `turn = 1; flag[0] = true; while (flag[1] && turn == 1);` and $P_1$ is rewritten symmetrically, what is the consequence?  
(A) Mutual exclusion is violated  
(B) Progress is violated (deadlock can occur)  
(C) Bounded waiting is violated  
(D) The algorithm works correctly  

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** If $P_0$ sets `turn = 1`, and preemption occurs before setting `flag[0] = true`, $P_1$ runs, sets `turn = 0, flag[1] = true`. $P_1$ sees `flag[0] == false` and enters CS. Then $P_0$ resumes, sets `flag[0] = true`, and checks `turn == 1`. Since $P_1$ set `turn = 0`, the while loop terminates and $P_0$ also enters CS. Both enter CS simultaneously, violating Mutual Exclusion.

</details>

**Q3 (MSQ).** Which of the following statements regarding process synchronization is/are TRUE?  
(A) A spinlock using `TestAndSet` avoids CPU cycles being wasted when waiting for the lock  
(B) If a solution satisfies Mutual Exclusion and Bounded Waiting, it must also satisfy Progress  
(C) Strict alternation using a single `turn` variable fails the Progress requirement  
(D) Disabling interrupts in user mode is a viable method to achieve mutual exclusion  

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:**
- (A) is FALSE: spinlocks waste CPU cycles in busy-waiting.
- (B) is FALSE: strict alternation satisfies Mutual Exclusion and Bounded Waiting, but violates Progress.
- (C) is TRUE: a process in its remainder section prevents the other process from entering CS.
- (D) is FALSE: disabling interrupts is a privileged instruction unavailable in user mode.

</details>

**Q4 (NAT).** Three concurrent processes $P_1, P_2, P_3$ share an integer variable $X$ initialized to 0. Each process executes the statement $X = X + 1$ exactly 10 times in a loop without synchronization. The statement consists of three machine instructions: `LOAD R, X`, `INCR R`, `STORE X, R`. What is the minimum possible final value of $X$ after all three processes complete?

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:**
1. $P_1$ reads $X = 0$ in its 1st iteration and is preempted.
2. $P_2$ and $P_3$ execute all their iterations except the last one, incrementing $X$ up to 18.
3. $P_1$ wakes up, computes $0 + 1 = 1$, and stores $X = 1$.
4. $P_2$ begins its last iteration: reads $X = 1$ and is preempted.
5. $P_1$ and $P_3$ finish all remaining iterations to completion, leaving $X \ge 2$.
6. $P_2$ wakes up: computes $1 + 1 = 2$, and stores $X = 2$.
Thus, the minimum possible value is 2.

</details>

**Q5 (MCQ).** In a system with $N$ processes, each process requires 3 units of a single type of resource. The total number of resource units available is $R$. What is the minimum value of $R$ that guarantees the system is free from deadlock?  
(A) $3N$  
(B) $2N + 1$  
(C) $2N$  
(D) $3N - 1$  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Worst-case allocation without any process finishing: each holds $3 - 1 = 2$ units $\implies 2N$. Providing 1 additional unit guarantees at least one process gets 3 units, finishes, and releases all units: $R \ge 2N + 1$.

</details>

**Q6 (NAT).** A system has 6 processes competing for critical sections guarded by a counting semaphore $S$. At most 2 processes are allowed to execute inside the critical section simultaneously. What should be the initial value of $S$?

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** Initializing a counting semaphore to $K$ allows at most $K$ concurrent processes to pass `wait(S)` without blocking. Here $K = 2$, so initial $S = 2$.

</details>

**Q7 (MSQ).** Which of the following is/are difference(s) between a Monitor condition variable and a Counting Semaphore?  
(A) A condition variable has no integer value, whereas a counting semaphore maintains an integer counter  
(B) A `signal` on a condition variable with no waiting processes is lost, whereas a semaphore `signal` increments the semaphore value  
(C) `wait` on a semaphore always blocks the calling process  
(D) Operations on monitor condition variables can only be invoked from within a monitor procedure  

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)  
**Solution:** (A), (B), (D) are standard differences. (C) is FALSE because `wait(S)` does not block if $S > 0$.

</details>

**Q8 (MCQ).** Consider two processes executing concurrently with semaphores $A$ and $B$, both initialized to 1:
```c
// Process 1: wait(A); wait(B); /* CS 1 */ signal(B); signal(A);
// Process 2: wait(B); wait(A); /* CS 2 */ signal(A); signal(B);
```
What failure can occur?  
(A) Mutual exclusion violation  
(B) Deadlock due to circular wait  
(C) Starvation of Process 1 only  
(D) Data corruption without deadlock  

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** If Process 1 acquires $A$ and Process 2 acquires $B$, Process 1 blocks waiting for $B$ and Process 2 blocks waiting for $A$. This forms circular wait $\implies$ deadlock.

</details>
