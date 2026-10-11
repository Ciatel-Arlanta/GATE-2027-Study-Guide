# Deadlocks

> **Paper:** CS · **Priority:** P1 · **Plan topics:** deadlock characterisation, prevention, avoidance, detection, recovery, Banker’s algorithm
> **Prerequisites:** [Processes, threads, syscalls](processes-threads-syscalls.md) · [Synchronization](synchronization.md) · **Leads to:** [Memory management](memory-management.md)
> **PYQ weightage:** Medium · 11 Qs since 2015 (5 since 2021) · DA: 0 · Trend: steady · Most asked: Resource Allocation, Deadlock Prevention/Avoidance/Detection, Banker's algorithm

## Quick glance

- **Exam focus:** Banker's algorithm safety test and resource-request algorithm (NAT/MCQ), minimum resource count to prevent deadlock ($R \ge \sum (m_i - 1) + 1$), Resource Allocation Graph (RAG) cycle analysis (single vs multi-instance), and Coffman conditions for deadlock prevention.
- **Deadlock definition:** A situation where a set of processes are blocked because each process holds resources and waits for resources held by other processes in the set.
- **Coffman conditions:** Deadlock can arise if and only if all four conditions hold simultaneously:
  1. **Mutual exclusion:** At least one non-shareable resource.
  2. **Hold and wait:** A process holding $\ge 1$ resource waits for additional resources held by others.
  3. **No preemption:** Resources cannot be forcibly taken; released only voluntarily upon completion.
  4. **Circular wait:** A closed chain of processes $\{P_0, \dots, P_n\}$ exists where $P_i$ waits for a resource held by $P_{(i+1) \bmod (n+1)}$.
- **Deadlock prevention:** System design guarantees that at least one of the four Coffman conditions cannot hold (e.g., total resource ordering to eliminate circular wait).
- **Deadlock avoidance:** Dynamic resource allocation ensures the system never enters an **unsafe state**. Banker's algorithm checks if a **safe sequence** exists before granting requests.
- **Safe state vs. Unsafe state:** Safe state $\implies$ no deadlock is possible. Unsafe state $\implies$ possibility of deadlock (not guaranteed deadlock). Deadlock is a strict subset of unsafe states.
- **RAG cycle theorem:** In a Resource Allocation Graph, a cycle is **necessary and sufficient** for deadlock with **single-instance** resources. With **multiple-instance** resources, a cycle is **necessary but NOT sufficient**.
- **Detection & Recovery:** Deadlock detection algorithm identifies whether the current state contains deadlocked processes; recovery preempts resources or terminates processes.

## 1. Deadlock characterisation and Coffman conditions [PYQ: High]

A set of processes is in a **deadlocked state** when every process in the set is waiting for an event (resource acquisition) that can be triggered only by another process in the set.

### 1.1 The four Coffman conditions

All four conditions must hold simultaneously for a deadlock to exist:

```text
+-----------------------+     +-----------------------+
| 1. Mutual Exclusion   |     | 2. Hold and Wait      |
| Resource held by one  |     | Holds resources while |
| process exclusively   |     | waiting for others    |
+-----------------------+     +-----------------------+
            ^                             ^
            |       DEADLOCK              |
            v      (All 4 hold)           v
+-----------------------+     +-----------------------+
| 3. No Preemption      |     | 4. Circular Wait      |
| Resources released    |     | Closed wait loop:     |
| only voluntarily      |     | P0 -> P1 -> ... -> P0 |
+-----------------------+     +-----------------------+
```

### 1.2 Minimum resource allocation formula

Consider $n$ processes sharing a single resource type with $R$ identical instances. Process $P_i$ requires at most $m_i$ instances to complete.

**Worst-case allocation (deadlock threshold):**
Each process $P_i$ is allocated $m_i - 1$ instances (one unit short of its maximum demand). Every process is now blocked, waiting for its final instance.

$$\text{Total allocated units in deadlock state} = \sum_{i=1}^n (m_i - 1)$$

If even **one additional instance** is available in the system, at least one process can acquire its required unit, run to completion, release all its held instances, and trigger a cascade where all remaining processes finish.

$$\text{Deadlock-free condition}: \quad R \ge \sum_{i=1}^n (m_i - 1) + 1$$

If all $n$ processes have identical maximum demand $m$:

$$R \ge n(m - 1) + 1 \iff n \le \left\lfloor \frac{R - 1}{m - 1} \right\rfloor$$

**Worked example 1.1 (Resource requirement calculation).** A system has $4$ processes: $P_1, P_2, P_3, P_4$, with maximum demands of $3, 4, 5,$ and $2$ units of a shared resource, respectively.
1. Compute the maximum number of allocated units that can result in a deadlock:
   $$\text{Units held} = (3 - 1) + (4 - 1) + (5 - 1) + (2 - 1) = 2 + 3 + 4 + 1 = 10\text{ units}$$
   With $10$ units allocated ($P_1$ holds $2$, $P_2$ holds $3$, $P_3$ holds $4$, $P_4$ holds $1$) and $0$ available, every process is indefinitely blocked.
2. The minimum number of resource instances $R$ to guarantee no deadlock is:
   $$R \ge 10 + 1 = 11\text{ units}$$

## 2. Resource-Allocation Graphs (RAG) and wait-for graphs [PYQ: Medium]

A Resource-Allocation Graph is a directed graph $G = (V, E)$ that represents system resource state.

### 2.1 Graph components
- **Vertices ($V$):** Partitioned into two sets:
  - Process nodes: $P = \{P_1, P_2, \dots, P_n\}$ (represented as circles).
  - Resource nodes: $R = \{R_1, R_2, \dots, R_m\}$ (represented as rectangles with dots indicating instances).
- **Edges ($E$):**
  - **Request edge ($P_i \to R_j$):** Process $P_i$ has requested an instance of $R_j$ and is currently waiting.
  - **Assignment edge ($R_j \to P_i$):** An instance of $R_j$ has been allocated to process $P_i$.

```text
Single-Instance Cycle (Deadlock):
    [ R1 ] ----> ( P1 )
      ^            |
      |            v
    ( P2 ) <---- [ R2 ]

Multi-Instance Cycle WITHOUT Deadlock:
    ( P1 ) ----> [ R1: • • ] ----> ( P3 ) [No requests, can finish]
      ^                 |
      |                 v
    [ R2: • ] <------ ( P2 )
```

### 2.2 Cycle theorem in RAGs

- **Single-instance per resource type:** A cycle in the RAG is **necessary and sufficient** for deadlock.
  $$\text{Cycle} \iff \text{Deadlock}$$
- **Multiple-instances per resource type:** A cycle in the RAG is **necessary but NOT sufficient** for deadlock.
  $$\text{Deadlock} \implies \text{Cycle}, \quad \text{but } \text{Cycle} \not\implies \text{Deadlock}$$

**Worked example 2.1 (Multi-instance cycle analysis).** Consider resources $R_1$ ($2$ instances) and $R_2$ ($1$ instance), and processes $P_1, P_2, P_3$:
- Allocations: $R_1 \to P_2$, $R_1 \to P_3$, $R_2 \to P_1$.
- Requests: $P_1 \to R_1$, $P_2 \to R_2$.
- Notice the cycle: $P_1 \to R_1 \to P_2 \to R_2 \to P_1$.
- *Is the system deadlocked?*
  No! Process $P_3$ holds an instance of $R_1$ but is **not waiting for any resource**. $P_3$ will finish execution and release its instance of $R_1$. The newly available instance of $R_1$ satisfies $P_1$'s request, breaking the cycle.

### 2.3 Wait-For Graph (WFG)
For single-instance systems, resource nodes can be collapsed into a **Wait-For Graph**:
- Edge $P_i \to P_j$ exists if process $P_i$ is waiting for a resource held by $P_j$.
- A cycle in a Wait-For Graph exists $\iff$ the system is in deadlock. Cycle detection runs via DFS in $O(V + E)$ time.

## 3. Deadlock prevention [PYQ: Medium]

Deadlock prevention designs the operating system so that at least one of the four Coffman conditions is **structurally impossible**.

| Coffman Condition | Prevention Technique | Practical Drawbacks |
|---|---|---|
| **1. Mutual Exclusion** | Make resources shareable (e.g., read-only files, spooled printers). | Intrinsically non-shareable resources (mutexes, write locks, hardware ports) cannot be made shareable. |
| **2. Hold and Wait** | **Protocol 1:** Process must request and acquire all needed resources before starting execution.<br>**Protocol 2:** Process can request resources only when it holds zero resources (must release held resources first). | Severe resource underutilisation; starvation for processes requiring many popular resources. |
| **3. No Preemption** | If a process holding resources requests an unavailable resource, all its currently held resources are preempted and returned to the free pool. | Only viable for state-saveable resources (CPU registers, physical memory). Unusable for stateful hardware (printers, tapes, database locks). |
| **4. Circular Wait** | **Global Linear Ordering:** Assign a unique integer $F(R) \in \mathbb{N}$ to every resource. Processes must request resources in strictly ascending order: $F(R_j) > F(R_i)$. | Most practical method in OS kernels; restricts programmer flexibility and may lower concurrency if resources are acquired too early. |

**Proof (Circular wait elimination via total ordering):**
Assume a circular wait exists: $P_0 \to P_1 \to P_2 \to \dots \to P_n \to P_0$. Process $P_i$ holds $R_i$ and requests $R_{i+1}$. By the protocol rule, $F(R_0) < F(R_1) < F(R_2) < \dots < F(R_n) < F(R_0)$, which yields the contradiction $F(R_0) < F(R_0)$. Thus, a cycle cannot form.

## 4. Deadlock avoidance and Banker's algorithm [PYQ: High]

Deadlock avoidance dynamically examines every resource request and grants it **only if the allocation leaves the system in a safe state**.

### 4.1 Safe, unsafe, and deadlocked states

- **Safe state:** A state is safe if there exists at least one **safe sequence** $\langle P_1, P_2, \dots, P_n \rangle$ such that for each $P_i$, the maximum remaining resources $P_i$ still needs can be satisfied by:
  $$\text{Need}_i \le \text{Available} + \sum_{j < i} \text{Allocation}_j$$
- **Unsafe state:** A state is unsafe if NO safe sequence exists.
- **Key distinction:** An unsafe state is **NOT** necessarily deadlocked! It means the system cannot prevent processes from eventually deadlocking if they all request their declared maximum claims simultaneously.

```text
+---------------------------------------------------+
|               ALL POSSIBLE STATES                 |
|   +-------------------------------------------+   |
|   |             UNSAFE STATES                 |   |
|   |     +-------------------------------+     |   |
|   |     |       DEADLOCKED STATES       |     |   |
|   |     +-------------------------------+     |   |
|   +-------------------------------------------+   |
|                                                   |
|   +-------------------------------------------+   |
|   |               SAFE STATES                 |   |
|   +-------------------------------------------+   |
+---------------------------------------------------+
```

$$\text{Safe} \implies \text{No Deadlock}, \quad \text{Deadlock} \implies \text{Unsafe}, \quad \text{Unsafe} \not\implies \text{Deadlock}$$

### 4.2 Banker's algorithm data structures

For $n$ processes and $m$ resource types:
1. $\text{Available}[m]$: Available units of each resource type.
2. $\text{Max}[n \times m]$: Maximum demand of each process.
3. $\text{Allocation}[n \times m]$: Units currently allocated to each process.
4. $\text{Need}[n \times m]$: Remaining resource demand:
   $$\text{Need}[i][j] = \text{Max}[i][j] - \text{Allocation}[i][j]$$

### 4.3 Safety algorithm

1. Initialize $\text{Work} = \text{Available}$ (vector of size $m$) and $\text{Finish}[i] = \text{false}$ for $i = 0, \dots, n - 1$.
2. Find an index $i$ such that:
   $$\text{Finish}[i] == \text{false} \quad \text{and} \quad \text{Need}_i \le \text{Work}$$
   If no such $i$ exists, proceed to Step 4.
3. Assume $P_i$ executes, finishes, and releases its allocation:
   $$\text{Work} = \text{Work} + \text{Allocation}_i, \quad \text{Finish}[i] = \text{true}$$
   Repeat Step 2.
4. If $\text{Finish}[i] == \text{true}$ for all $i$, the system is in a **safe state**.

### 4.4 Resource-request algorithm

When process $P_i$ makes request $\text{Request}_i$:
1. If $\text{Request}_i \le \text{Need}_i$, proceed; else raise error (exceeded maximum claim).
2. If $\text{Request}_i \le \text{Available}$, proceed; else $P_i$ must wait (insufficient resources).
3. Tentatively allocate resources:
   $$\text{Available} = \text{Available} - \text{Request}_i$$
   $$\text{Allocation}_i = \text{Allocation}_i + \text{Request}_i$$
   $$\text{Need}_i = \text{Need}_i - \text{Request}_i$$
4. Run the Safety Algorithm on the tentative state:
   - If safe: grant the request.
   - If unsafe: roll back the allocation state and force $P_i$ to wait.

**Worked example 4.1 (Banker's safety test).** System with $5$ processes ($P_0 \dots P_4$) and $3$ resource types $A(10), B(5), C(7)$:

| Process | Allocation ($A, B, C$) | Max ($A, B, C$) | Need = Max $-$ Allocation |
|---|---|---|---|
| $P_0$ | $(0, 1, 0)$ | $(7, 5, 3)$ | $(7, 4, 3)$ |
| $P_1$ | $(2, 0, 0)$ | $(3, 2, 2)$ | $(1, 2, 2)$ |
| $P_2$ | $(3, 0, 2)$ | $(9, 0, 2)$ | $(6, 0, 0)$ |
| $P_3$ | $(2, 1, 1)$ | $(2, 2, 2)$ | $(0, 1, 1)$ |
| $P_4$ | $(0, 0, 2)$ | $(4, 3, 3)$ | $(4, 3, 1)$ |

Total allocated $= (7, 2, 5) \implies \text{Available} = (10 - 7, 5 - 2, 7 - 5) = (3, 3, 2)$.

*Safety execution trace:*
- $\text{Work} = (3, 3, 2)$.
- Check $P_0$: $\text{Need}_0 = (7, 4, 3) \not\le (3, 3, 2)$. Cannot run.
- Check $P_1$: $\text{Need}_1 = (1, 2, 2) \le (3, 3, 2)$. Can run!
  $\text{Work} = (3, 3, 2) + (2, 0, 0) = (5, 3, 2)$. $\text{Finish}[1] = \text{true}$.
- Check $P_3$: $\text{Need}_3 = (0, 1, 1) \le (5, 3, 2)$. Can run!
  $\text{Work} = (5, 3, 2) + (2, 1, 1) = (7, 4, 3)$. $\text{Finish}[3] = \text{true}$.
- Check $P_0$: $\text{Need}_0 = (7, 4, 3) \le (7, 4, 3)$. Can run!
  $\text{Work} = (7, 4, 3) + (0, 1, 0) = (7, 5, 3)$. $\text{Finish}[0] = \text{true}$.
- Check $P_2$: $\text{Need}_2 = (6, 0, 0) \le (7, 5, 3)$. Can run!
  $\text{Work} = (7, 5, 3) + (3, 0, 2) = (10, 5, 5)$. $\text{Finish}[2] = \text{true}$.
- Check $P_4$: $\text{Need}_4 = (4, 3, 1) \le (10, 5, 5)$. Can run!
  $\text{Work} = (10, 5, 5) + (0, 0, 2) = (10, 5, 7)$. $\text{Finish}[4] = \text{true}$.

All processes finish. **State is safe** with valid safe sequence: $\langle P_1, P_3, P_0, P_2, P_4 \rangle$.

## 5. Deadlock detection and recovery [PYQ: Medium]

Unlike avoidance (which inspects requests prospectively), detection allows requests unconditionally and periodically checks whether a deadlock has manifested.

### 5.1 Deadlock detection algorithm
Similar to Banker's algorithm, but evaluates **current actual requests** rather than declared maximum demands:
1. Initialize $\text{Work} = \text{Available}$.
2. For $i = 0 \dots n - 1$: if $\text{Allocation}_i \neq 0$, set $\text{Finish}[i] = \text{false}$; else $\text{Finish}[i] = \text{true}$.
3. Find index $i$ such that $\text{Finish}[i] == \text{false}$ and $\text{Request}_i \le \text{Work}$.
   If found: $\text{Work} = \text{Work} + \text{Allocation}_i, \text{Finish}[i] = \text{true}$; repeat.
4. If $\text{Finish}[i] == \text{false}$ for any process $i$, **the system is deadlocked**, and process $P_i$ is deadlocked.

### 5.2 Recovery strategies
- **Process termination:** Abort all deadlocked processes, or abort one process at a time until the deadlock cycle is broken (selection based on process priority, execution time spent, resources held).
- **Resource preemption:** Preempt resources from a chosen victim process and roll back the victim to a safe checkpoint.
  - *Starvation risk:* If the same process is repeatedly selected as the victim, it starves. To prevent this, the cost function must factor in the number of past rollbacks.
- **The Ostrich Algorithm:** Ignore deadlocks entirely. Used by general-purpose operating systems (Linux, Windows) where deadlocks are rare and runtime prevention/detection overhead is prohibitive.

## Formulas and facts to memorise

| Item | Formula / Fact | When to use |
|---|---|---|
| **Deadlock-free bound (general)** | $R \ge \sum_{i=1}^n (m_i - 1) + 1$ | Finding minimum resource units for distinct max demands |
| **Deadlock-free bound (identical)** | $R \ge n(m - 1) + 1$ | Finding minimum units for $n$ processes with max demand $m$ |
| **Maximum processes avoiding deadlock** | $n \le \lfloor \frac{R - 1}{m - 1} \rfloor$ | Finding max safe degree of multiprogramming |
| **Need matrix** | $\text{Need}[i][j] = \text{Max}[i][j] - \text{Allocation}[i][j]$ | Computing remaining claim in Banker's algorithm |
| **Safety test condition** | $\text{Need}_i \le \text{Work} \implies \text{Work} = \text{Work} + \text{Allocation}_i$ | Verifying whether process $P_i$ can finish |
| **Single-instance RAG** | Cycle $\iff$ Deadlock | Cycle detection in single-instance graphs |
| **Multi-instance RAG** | Cycle is necessary, but NOT sufficient | Evaluating multi-instance deadlock |
| **Total ordering rule** | Request valid iff $F(R_{\text{new}}) > F(R_{\text{held}})$ | Eliminating circular wait condition |

## GATE traps

- **Need is Max minus Allocation, NOT Max minus Available:** Always subtract $\text{Allocation}$ from $\text{Max}$.
- **Unsafe does NOT mean deadlocked:** An unsafe state only means that the OS cannot *guarantee* avoidance if all processes simultaneously demand their peak need. If processes release early, deadlock will not occur.
- **Vector comparison must hold for ALL components:** $\text{Need}_i \le \text{Work}$ requires $\text{Need}_i[j] \le \text{Work}[j]$ for every single resource index $j$. If even one component exceeds $\text{Work}$, the process cannot run.
- **Cycle does not imply deadlock in multi-instance systems:** Always verify if a process involved in a multi-instance cycle (or holding a resource needed by the cycle) can finish independently and release its instances.
- **Resource Request check has two steps:** Before tentative allocation, verify BOTH $\text{Request} \le \text{Need}$ and $\text{Request} \le \text{Available}$.

## Connections

- [Synchronization](synchronization.md) — Unordered semaphore locks (`P(mutex1)`, `P(mutex2)`) across threads are the #1 source of real-world deadlocks.
- [CPU scheduling](cpu-scheduling.md) — Deadlock halts process progress; differs from starvation where processes wait indefinitely due to scheduling bias.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — Wait-for graphs map directly to directed cycle detection via DFS / topological sorting.
- [Databases: transactions and concurrency](../14-databases/transactions-and-concurrency.md) — Two-Phase Locking (2PL) creates transaction wait-for graphs and deadlock aborts.

## Practice

**Q1. (NAT)** A system has $4$ processes sharing a single resource type. The maximum resource requirements for the processes are $3, 4, 5,$ and $2$, respectively. What is the minimum number of resource instances required to guarantee that deadlock will never occur?

<details><summary>Answer</summary>

**Answer:** $11$

**Solution:**
1. Worst-case allocation where each process holds $m_i - 1$ units and is blocked:
   $$\text{Units held} = (3 - 1) + (4 - 1) + (5 - 1) + (2 - 1) = 2 + 3 + 4 + 1 = 10\text{ units}$$
2. Adding $1$ additional unit ensures at least one process receives its maximum claim and finishes.
3. $R_{\min} = 10 + 1 = 11\text{ units}$.
</details>

**Q2. (MCQ)** Consider a system with $5$ processes ($P_0 \dots P_4$) and $4$ resource types ($A, B, C, D$).
- Allocation: $P_0(0,0,1,2)$, $P_1(1,0,0,0)$, $P_2(1,3,5,4)$, $P_3(0,6,3,2)$, $P_4(0,0,1,4)$.
- Max: $P_0(0,0,1,2)$, $P_1(1,7,5,0)$, $P_2(2,3,5,6)$, $P_3(0,6,5,2)$, $P_4(0,6,5,6)$.
- Available: $(1, 5, 2, 0)$.
Which of the following is a valid safe sequence?
- (A) $\langle P_0, P_1, P_2, P_3, P_4 \rangle$
- (B) $\langle P_0, P_2, P_1, P_3, P_4 \rangle$
- (C) $\langle P_1, P_0, P_2, P_3, P_4 \rangle$
- (D) The system is in an unsafe state.

<details><summary>Answer</summary>

**Answer:** (B)

**Solution:**
1. Compute $\text{Need} = \text{Max} - \text{Allocation}$:
   - $\text{Need}_0 = (0, 0, 0, 0)$
   - $\text{Need}_1 = (0, 7, 5, 0)$
   - $\text{Need}_2 = (1, 0, 0, 2)$
   - $\text{Need}_3 = (0, 0, 2, 0)$
   - $\text{Need}_4 = (0, 6, 4, 2)$
2. Initial $\text{Work} = (1, 5, 2, 0)$.
   - $P_0$ can run: $\text{Need}_0 = (0,0,0,0) \le (1,5,2,0)$.
     $\text{Work} = (1, 5, 2, 0) + (0, 0, 1, 2) = (1, 5, 3, 2)$.
   - Check $P_1$: $\text{Need}_1 = (0, 7, 5, 0) \not\le (1, 5, 3, 2)$ (fails on $B$ and $C$).
   - Check $P_2$: $\text{Need}_2 = (1, 0, 0, 2) \le (1, 5, 3, 2)$. Can run!
     $\text{Work} = (1, 5, 3, 2) + (1, 3, 5, 4) = (2, 8, 8, 6)$.
   - Now check $P_1$: $\text{Need}_1 = (0, 7, 5, 0) \le (2, 8, 8, 6)$. Can run!
     $\text{Work} = (2, 8, 8, 6) + (1, 0, 0, 0) = (3, 8, 8, 6)$.
   - Check $P_3$: $\text{Need}_3 = (0, 0, 2, 0) \le (3, 8, 8, 6)$. Can run!
     $\text{Work} = (3, 8, 8, 6) + (0, 6, 3, 2) = (3, 14, 11, 8)$.
   - Check $P_4$: $\text{Need}_4 = (0, 6, 4, 2) \le (3, 14, 11, 8)$. Can run!
3. Valid safe sequence is $\langle P_0, P_2, P_1, P_3, P_4 \rangle$.
</details>

**Q3. (NAT)** In a computer system, $6$ processes compete for $m$ units of an identical resource. Each process requires at most $3$ units. What is the smallest value of $m$ such that deadlock cannot occur?

<details><summary>Answer</summary>

**Answer:** $13$

**Solution:**
1. Number of processes $n = 6$, max requirement per process $k = 3$.
2. In the worst-case deadlock threshold, each process holds $k - 1 = 2$ units:
   $$\text{Allocated units} = 6 \times 2 = 12\text{ units}$$
3. Minimum resources to guarantee freedom from deadlock:
   $$m \ge n(k - 1) + 1 = 6(2) + 1 = 13\text{ units}$$.
</details>

**Q4. (MSQ)** Which of the following statements regarding deadlocks is/are TRUE?
- (A) In a resource allocation graph with multiple instances of each resource type, the presence of a cycle guarantees deadlock.
- (B) Imposing a strict linear ordering on resource acquisitions prevents the circular wait condition.
- (C) If a system is in an unsafe state, it is guaranteed to be in a deadlocked state.
- (D) In Banker's algorithm, if a request results in an unsafe state, the request is denied and the process is made to wait.

<details><summary>Answer</summary>

**Answer:** (B), (D)

**Solution:**
- (A) is FALSE: With multiple instances, a cycle is necessary but not sufficient for deadlock.
- (B) is TRUE: Strict ascending resource acquisition prevents circular wait, eliminating deadlocks.
- (C) is FALSE: An unsafe state is not necessarily deadlocked; deadlock occurs only if processes subsequently request their full maximum claims.
- (D) is TRUE: Deadlock avoidance denies requests that transition the system from safe to unsafe.
</details>

**Q5. (NAT)** A system has $3$ processes $P_1, P_2, P_3$ and $10$ units of resource $R$. Maximum demands are: $P_1 = 7, P_2 = 5, P_3 = 4$. Current allocations are: $P_1 = 3, P_2 = 2, P_3 = 2$. What is the maximum number of additional units that $P_1$ can request such that the system remains in a safe state?

<details><summary>Answer</summary>

**Answer:** $1$

**Solution:**
1. Total allocated $= 3 + 2 + 2 = 7\text{ units}$. Available $= 10 - 7 = 3\text{ units}$.
2. Current Need:
   - $\text{Need}_1 = 7 - 3 = 4$
   - $\text{Need}_2 = 5 - 2 = 3$
   - $\text{Need}_3 = 4 - 2 = 2$
3. In current state: $\text{Available} = 3$.
   $P_3$ can finish ($\text{Need}_3 = 2 \le 3$), releasing $2$ units $\implies \text{Work} = 5$.
   Then $P_2$ can finish ($\text{Need}_2 = 3 \le 5$), releasing $2$ units $\implies \text{Work} = 7$.
   Then $P_1$ finishes. Current state is safe!
4. Suppose $P_1$ requests $1$ additional unit:
   - Tentative: $\text{Alloc}_1 = 4, \text{Need}_1 = 3, \text{Available} = 3 - 1 = 2$.
   - Check safety:
     - Available $= 2$. $P_3$ can run ($\text{Need}_3 = 2 \le 2$). $P_3$ finishes, releasing $2$ units $\implies \text{Work} = 2 + 2 = 4$.
     - Next, can $P_2$ run? $\text{Need}_2 = 3 \le 4$. Yes! $P_2$ finishes, releasing $2$ units $\implies \text{Work} = 4 + 2 = 6$.
     - Next, can $P_1$ run? $\text{Need}_1 = 3 \le 6$. Yes! $P_1$ finishes! **Safe.**
5. Now suppose $P_1$ requests $2$ additional units:
   - Tentative: $\text{Alloc}_1 = 5, \text{Need}_1 = 2, \text{Available} = 3 - 2 = 1$.
   - Check: $\text{Need}_3 = 2 \not\le 1, \text{Need}_2 = 3 \not\le 1, \text{Need}_1 = 2 \not\le 1$.
   - No process can run! Unsafe state!
6. Therefore, $P_1$ can request at most $1$ additional unit and still leave the system in a safe state.
</details>

**Q6. (MCQ)** Consider the prevention of deadlock by invalidating Coffman conditions. Which condition is the most practical to eliminate in general-purpose operating systems, and what technique is used?
- (A) Mutual Exclusion, by making all resources shareable.
- (B) Hold and Wait, by requiring processes to request all resources at system startup.
- (C) No Preemption, by aborting any process that requests a lock.
- (D) Circular Wait, by establishing a global linear hierarchy of resource types.

<details><summary>Answer</summary>

**Answer:** (D)

**Solution:**
Establishing a total ordering on resource types and enforcing that resources be acquired in strictly ascending order is widely used (e.g., in lock hierarchies within OS kernels) because it requires no runtime rollback or premature resource locking.
</details>

**Q7. (NAT)** A system consists of $2$ processes sharing $3$ instances of resource $X$ and $2$ instances of resource $Y$. $P_1$ holds $1$ instance of $X$ and needs $1$ instance of $Y$. $P_2$ holds $1$ instance of $Y$ and $1$ instance of $X$, and needs $1$ instance of $X$. How many processes are currently deadlocked in the system?

<details><summary>Answer</summary>

**Answer:** $0$

**Solution:**
1. Total resources: $X = 3, Y = 2$.
2. Total allocations:
   - $X$ allocated: $1\text{ (to } P_1) + 1\text{ (to } P_2) = 2$ instances.
   - $Y$ allocated: $1\text{ (to } P_2) = 1$ instance.
3. Available resources:
   - Available $X = 3 - 2 = 1$.
   - Available $Y = 2 - 1 = 1$.
4. Requests:
   - $P_1$ needs $1$ unit of $Y$. Available $Y = 1 \ge 1 \implies P_1$'s request can be satisfied immediately!
   - $P_2$ needs $1$ unit of $X$. Available $X = 1 \ge 1 \implies P_2$'s request can be satisfied immediately!
5. Both processes can acquire their needed resources and run to completion. Total deadlocked processes $= 0$.
</details>
