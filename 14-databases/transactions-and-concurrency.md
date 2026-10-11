# Transactions and concurrency control

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** ACID properties, transactions, serializability (conflict and view), recoverability, concurrency control (locking, 2PL, strict 2PL), timestamp ordering, deadlocks in DBMS
> **Prerequisites:** [SQL](sql.md) · [Constraints and normalization](constraints-and-normalization.md) · **Leads to:** [File organization and indexing](file-organization-and-indexing.md)
> **PYQ weightage:** High · 36 Qs since 2015 (15 since 2021) · DA: 0 · Trend: steady · Most asked: Conflict Serializability & Precedence Graphs (18), Recoverable & Cascadeless Schedules (10), Two-Phase Locking (2PL & Strict 2PL) (5), Timestamp Ordering & Deadlocks (3)

## Quick glance

- **Exam focus:** Testing conflict serializability via precedence graphs (finding directed cycles and topological ordering), classifying schedules as recoverable/cascadeless/strict, checking 2PL/strict 2PL lock rules, and Thomas Write Rule; asked frequently in GATE CS as MCQ, MSQ, and NAT.
- **ACID properties:** **Atomicity** (all or nothing, managed by Recovery manager/log), **Consistency** (database constraints preserved), **Isolation** (concurrency control manager), **Durability** (committed updates survive crashes, managed by recovery manager/WAL).
- **Conflicting operations:** Two operations conflict iff they belong to **different transactions**, access the **same data item**, and **at least one is a write** ($R_i(X)\text{--}W_j(X)$, $W_i(X)\text{--}R_j(X)$, $W_i(X)\text{--}W_j(X)$). Read-read never conflicts.
- **Conflict serializability:** Construct the **precedence graph**. Add edge $T_i \to T_j$ if an operation of $T_i$ precedes and conflicts with an operation of $T_j$. The schedule is conflict serializable **iff the graph has no directed cycles**. Topological sort yields the equivalent serial schedule(s).
- **View serializability:** Broader than conflict serializability. If a schedule has **no blind writes** (every write is preceded by a read on the same item), then **View Serializability $\iff$ Conflict Serializability**.
- **Recoverability hierarchy:** $\text{Strict} \subset \text{Cascadeless (ACA)} \subset \text{Recoverable} \subset \text{All schedules}$.
  - **Recoverable:** If $T_j$ reads data written by $T_i$, then $T_i$ must commit before $T_j$ commits ($c_i < c_j$).
  - **Cascadeless:** $T_j$ can read $X$ only *after* the transaction that wrote $X$ has committed ($c_i < r_j(X)$).
  - **Strict:** $T_j$ can neither read nor overwrite $X$ until the transaction that wrote $X$ has committed ($c_i < r_j(X)$ and $c_i < w_j(X)$).
- **Two-Phase Locking (2PL):** Growing phase (locks acquired only) followed by shrinking phase (locks released only). Guarantees **conflict serializability**, but does **not prevent deadlocks** and can suffer from cascading rollbacks.
- **Strict 2PL:** Basic 2PL + all **exclusive locks held until transaction commit/abort**. Guarantees conflict serializability, recoverability, and strictness (no cascading rollbacks).
- **Timestamp Ordering (TO):** Older transactions have smaller timestamps ($TS$). Ensures serializability without deadlocks, but may cause starvation/cascading rollbacks. **Thomas Write Rule** ignores obsolete writes, allowing non-conflict-serializable view serial schedules.

## 1. ACID properties, transaction states, and concurrency anomalies [PYQ: Medium]

A **transaction** is a logical unit of database processing that includes one or more database access operations.

```text
                  +----------------+
                  |     Active     |
                  +-------+--------+
                          |
              +-----------+-----------+
              |                       |
              v                       v
    +-------------------+   +-------------------+
    |Partially Committed|   |      Failed       |
    +---------+---------+   +---------+---------+
              |                       |
              v                       v
    +-------------------+   +-------------------+
    |     Committed     |   |      Aborted      |
    +-------------------+   +-------------------+
```

- **Atomicity:** Entire transaction executes to completion or its effects are completely undone. Managed by transaction management and Write-Ahead Logging (WAL).
- **Consistency:** Execution in isolation preserves all schema integrity constraints (declarative constraints, foreign keys, assertions).
- **Isolation:** Execution of one transaction is unaffected by concurrently executing transactions. Managed by concurrency control protocols.
- **Durability:** Once committed, state changes survive any subsequent system crashes. Maintained by redo logs on stable storage.

### Concurrency anomalies

When transactions interleave without proper concurrency control:
1. **Dirty Read (Temporary Update / Read-Write conflict):** $T_1$ modifies $X$, $T_2$ reads modified $X$, then $T_1$ aborts and rolls back. $T_2$ read invalid uncommitted data.
2. **Unrepeatable Read (Fuzzy Read / Read-Write-Read conflict):** $T_1$ reads $X$, $T_2$ overwrites or deletes $X$ and commits; $T_1$ reads $X$ again and sees a different value.
3. **Lost Update (Write-Write conflict):** $T_1$ and $T_2$ read $X$; $T_1$ updates $X$ based on read value; $T_2$ also updates $X$, overwriting and obliterating $T_1$'s update without knowing it.
4. **Phantom Read:** $T_1$ executes a range query (e.g. `WHERE salary > 50000`), $T_2$ inserts a new row matching the predicate and commits; $T_1$ re-executes the query and sees a "phantom" new tuple.

## 2. Schedules and conflict serializability [PYQ: High]

A **schedule $S$** over a set of transactions $\{T_1, T_2, \dots, T_n\}$ is an ordering of their operations preserving each transaction's internal execution order.
- A schedule is **serial** if operations of different transactions do not interleave (one finishes completely before the next starts).
- A schedule is **concurrent (non-serial)** if operations interleave.

### Conflicting operations

Two operations $I_i$ and $I_j$ in schedule $S$ are in **conflict** iff:
1. They belong to different transactions ($i \neq j$).
2. They access the exact same data item $X$.
3. At least one of the two operations is a write ($W$).

| Pair | Different transactions? | Same data item? | At least one write? | Conflict? |
|---|---|---|---|---|
| $R_i(X), R_j(X)$ | Yes | Yes | No | **No** |
| $R_i(X), W_j(X)$ | Yes | Yes | Yes | **Yes (RW)** |
| $W_i(X), R_j(X)$ | Yes | Yes | Yes | **Yes (WR)** |
| $W_i(X), W_j(X)$ | Yes | Yes | Yes | **Yes (WW)** |
| $W_i(X), W_j(Y)$ | Yes | No ($X \neq Y$) | Yes | **No** |

### Conflict equivalence and serializability

Two schedules $S$ and $S'$ are **conflict equivalent** if $S'$ can be obtained from $S$ by a sequence of swaps of consecutive non-conflicting operations.  
A schedule $S$ is **conflict serializable** if it is conflict equivalent to some serial schedule.

### The precedence graph test

To determine if schedule $S$ is conflict serializable:
1. Create a directed graph $G = (V, E)$, where $V$ contains a vertex for each transaction in $S$.
2. For each pair of conflicting operations where $O_i$ precedes $O_j$ in time in $S$:
   - Add a directed edge $T_i \to T_j$.
3. **Serializability Theorem:** Schedule $S$ is conflict serializable **if and only if $G$ is a Directed Acyclic Graph (DAG)**.
4. If $G$ has no cycles, every **topological sort** of $G$ is an equivalent serial execution order.
5. If $G$ contains a directed cycle, $S$ is **not conflict serializable**.

**Worked example 1 (Precedence graph construction & topological sorting).**  
Consider schedule $S$ with transactions $T_1, T_2, T_3$:
$$S: R_1(X),\; R_2(X),\; R_1(Y),\; W_2(X),\; R_3(Y),\; W_1(Y),\; W_3(Y)$$

Let us find all conflicting pairs and construct the precedence graph:
1. Data item $X$:
   - $R_1(X)$ precedes $W_2(X) \implies$ edge $T_1 \to T_2$.
   - $R_2(X)$ and $W_2(X)$ belong to the same transaction ($T_2$), so no edge between transactions.
2. Data item $Y$:
   - $R_1(Y)$ precedes $W_3(Y) \implies$ edge $T_1 \to T_3$.
   - $R_3(Y)$ precedes $W_1(Y) \implies$ edge $T_3 \to T_1$.
   - $W_1(Y)$ precedes $W_3(Y) \implies$ edge $T_1 \to T_3$.

Now inspect the directed edges in $G$:
- $T_1 \to T_3$ (from $R_1(Y)$ before $W_3(Y)$, and $W_1(Y)$ before $W_3(Y)$).
- $T_3 \to T_1$ (from $R_3(Y)$ before $W_1(Y)$).

Directed cycle detected: $T_1 \to T_3 \to T_1$.  
**Conclusion:** The precedence graph has a cycle between $T_1$ and $T_3$. Schedule $S$ is **NOT conflict serializable**.

## 3. View serializability and blind writes [PYQ: Medium]

Two schedules $S$ and $S'$ over the same transactions are **view equivalent** iff:
1. **Initial read:** For each data item $X$, if transaction $T_i$ reads the initial database value of $X$ in $S$, then $T_i$ must read the initial value of $X$ in $S'$.
2. **Updated read ("read-from"):** If transaction $T_j$ reads the value of $X$ produced by $T_i$ in $S$, then $T_j$ must read the value of $X$ produced by $T_i$ in $S'$.
3. **Final write:** For each data item $X$, whichever transaction performs the final write on $X$ in $S$ must perform the final write on $X$ in $S'$.

A schedule is **view serializable** if it is view equivalent to some serial schedule.

```text
+-------------------------------------------------------+
| View Serializable Schedules                           |
|   +---------------------------------------------------+
|   | Conflict Serializable Schedules                   |
|   | (Precedence graph has NO cycle)                   |
|   +---------------------------------------------------+
+-------------------------------------------------------+
```

### Relationship and the blind write rule

- Every conflict serializable schedule is view serializable: $\text{Conflict Serializable} \subset \text{View Serializable}$.
- **Blind write:** An operation $W_i(X)$ without a preceding $R_i(X)$ in transaction $T_i$.
- **The Blind Write Theorem:** If a schedule $S$ has **no blind writes**, then:
$$\text{View Serializable} \iff \text{Conflict Serializable}$$
Thus, to find a schedule that is view serializable but *not* conflict serializable, it **must contain at least one blind write**!

**Worked example 2 (View serializable but not conflict serializable).**  
Consider schedule $S$:
$$S: R_1(X),\; W_2(X),\; W_1(X),\; W_3(X)$$
Notice $W_2(X)$ and $W_3(X)$ are blind writes (neither reads $X$ first).
- Conflicting pairs:
  - $R_1(X)$ before $W_2(X) \implies T_1 \to T_2$.
  - $W_2(X)$ before $W_1(X) \implies T_2 \to T_1$.
  - $W_1(X)$ before $W_3(X) \implies T_1 \to T_3$.
  - $W_2(X)$ before $W_3(X) \implies T_2 \to T_3$.
- Cycle: $T_1 \to T_2 \to T_1$. Precedence graph has a cycle $\implies$ **NOT conflict serializable**.

Now test view equivalence against serial schedule $S_{\text{serial}} = T_2 \to T_1 \to T_3$:
1. Initial read: $T_1$ reads initial $X$ in both $S$ and $S_{\text{serial}}$ $\checkmark$.
2. Read-from: No transaction reads any written value in either schedule $\checkmark$.
3. Final write: $T_3$ performs the final write on $X$ in both $S$ and $S_{\text{serial}}$ $\checkmark$.  
All view conditions match! Therefore $S$ is **view equivalent to $T_2 \to T_1 \to T_3$**.  
**Conclusion:** Schedule $S$ is **view serializable**, despite not being conflict serializable.

## 4. Recoverability, cascadelessness, and strict schedules [PYQ: High]

When transactions abort unexpectedly, the database must roll them back without leaving inconsistent data or violating isolation.

```text
+-------------------------------------------------------------+
| All Schedules                                               |
|   +---------------------------------------------------------+
|   | Recoverable Schedules (c_i < c_j for dirty reads)       |
|   |   +-----------------------------------------------------+
|   |   | Cascadeless Schedules (c_i < r_j(X) for reads)      |
|   |   |   +-------------------------------------------------+
|   |   |   | Strict Schedules (c_i < r_j(X) and c_i < w_j(X))|
|   |   |   +-------------------------------------------------+
```

### Definitions

1. **Recoverable schedule:** If transaction $T_j$ reads a data item previously written by $T_i$, then $T_i$ must commit **before** $T_j$ commits:
   $$\text{If } W_i(X) < R_j(X), \text{ then } C_i < C_j$$
   If $T_i$ aborts, $T_j$ can still be aborted. If $T_j$ committed before $T_i$, and $T_i$ aborts, the database cannot undo $T_j$'s committed output $\implies$ **unrecoverable**!
2. **Cascadeless schedule (Avoids Cascading Aborts - ACA):** A transaction $T_j$ is only allowed to read a data item $X$ **after** the transaction $T_i$ that last wrote $X$ has **committed**:
   $$\text{If } W_i(X) < R_j(X), \text{ then } C_i < R_j(X)$$
   Every cascadeless schedule is guaranteed to be recoverable.
3. **Strict schedule:** A transaction is neither allowed to read nor overwrite $X$ until the previous transaction that wrote $X$ has committed (or aborted):
   $$C_i < R_j(X) \quad \text{and} \quad C_i < W_j(X)$$
   Every strict schedule is cascadeless and recoverable. Strict schedules simplify recovery: restoring the before-image of an aborted transaction's writes never overwrites active concurrent updates.

### Independence of serializability and recoverability

Serializability and recoverability are completely independent properties:
- A schedule can be serializable and recoverable.
- A schedule can be serializable and **unrecoverable**.
- A schedule can be non-serializable and recoverable.
- A schedule can be non-serializable and unrecoverable.

**Worked example 3 (Classifying schedules).**  
Classify the following schedules into: (i) Recoverable, (ii) Cascadeless, (iii) Strict, (iv) Conflict serializable.

Schedule $S_A$:  
$$S_A: W_1(X),\; R_2(X),\; W_2(Y),\; C_2,\; C_1$$
- $T_2$ reads $X$ written by $T_1$ ($W_1(X) < R_2(X)$).
- Commit order: $C_2$ occurs before $C_1$ ($C_2 < C_1$).
- Since $T_2$ read from $T_1$ but committed before $T_1$, if $T_1$ aborts, $T_2$ has already committed with dirty data.
- **Classification:** **Unrecoverable** (and therefore neither cascadeless nor strict). Conflict serializable: edges are $T_1 \to T_2$ (no cycle, so yes).

Schedule $S_B$:  
$$S_B: W_1(X),\; R_2(X),\; C_1,\; C_2$$
- $T_2$ reads $X$ from $T_1$. $C_1$ occurs before $C_2$ ($C_1 < C_2$).
- **Recoverable:** Yes ($C_1 < C_2$).
- **Cascadeless:** No, because $R_2(X)$ occurred *before* $C_1$ (reads uncommitted write; if $T_1$ aborts, $T_2$ must cascade abort).
- **Strict:** No.
- **Conflict serializable:** Yes ($T_1 \to T_2$).

Schedule $S_C$:  
$$S_C: W_1(X),\; C_1,\; R_2(X),\; C_2$$
- $R_2(X)$ occurs after $C_1$.
- **Recoverable:** Yes.
- **Cascadeless:** Yes ($C_1 < R_2(X)$).
- **Strict:** Yes ($C_1 < R_2(X)$ and no overwrites).
- **Conflict serializable:** Yes.

## 5. Concurrency control: locking, 2PL, and timestamp ordering [PYQ: High]

### Lock compatibility

- **Shared Lock ($S$):** Read lock. Multiple transactions can concurrently hold shared locks on $X$.
- **Exclusive Lock ($X$):** Write lock. Only one transaction can hold an exclusive lock on $X$; excludes all other readers and writers.

| Requested \ Held | Shared ($S$) | Exclusive ($X$) |
|---|---|---|
| **Shared ($S$)** | **Compatible** (Granted) | Incompatible (Blocked) |
| **Exclusive ($X$)** | Incompatible (Blocked) | Incompatible (Blocked) |

### Two-Phase Locking (2PL)

A transaction follows the **Two-Phase Locking (2PL)** protocol iff:
1. **Growing Phase:** It may acquire locks, but cannot release any lock.
2. **Shrinking Phase:** It may release locks, but cannot acquire any new lock.
3. The point where the transaction acquires its final lock is its **lock point**.

**Key properties of Basic 2PL:**
- Guarantees **conflict serializability** (serial order corresponds to the ordering of transaction lock points).
- **Does NOT prevent deadlocks** (transactions can wait circularly for locks held by each other).
- **Does NOT prevent cascading rollbacks** (dirty reads can still occur if a transaction releases an exclusive lock early during shrinking phase and later aborts).

### Variants of 2PL

```text
+-----------------------+-------------------------+----------------------+-----------------------+
| 2PL Variant           | Conflict Serializable?  | Deadlock Free?       | Cascading Abort Free? |
+-----------------------+-------------------------+----------------------+-----------------------+
| Basic 2PL             | Yes                     | No                   | No                    |
| Conservative 2PL      | Yes                     | Yes (all locks at start) | No               |
| Strict 2PL            | Yes                     | No                   | Yes (strict schedule) |
| Rigorous 2PL          | Yes (commit = lock order)| No                  | Yes (strict schedule) |
+-----------------------+-------------------------+----------------------+-----------------------+
```
- **Conservative 2PL (Static 2PL):** Transaction locks all needed items before execution begins. If any lock is unavailable, it acquires none and waits. Completely **prevents deadlocks**, but drastically limits concurrency.
- **Strict 2PL:** Follows 2PL and **holds all exclusive ($X$) locks until transaction completion (COMMIT/ABORT)**. Guarantees **strict, cascadeless schedules** and conflict serializability.
- **Rigorous 2PL:** Follows 2PL and **holds ALL locks (both $S$ and $X$) until commit/abort**.

### Timestamp Ordering (TO) Protocol

Each transaction $T_i$ is assigned a globally unique timestamp $TS(T_i)$ when it begins. Older transactions have smaller timestamps ($TS(T_1) < TS(T_2) \implies T_1$ is older).  
For each data item $X$:
- $W\text{-}TS(X)$: Largest timestamp of any transaction that successfully executed $W(X)$.
- $R\text{-}TS(X)$: Largest timestamp of any transaction that successfully executed $R(X)$.

**Basic TO Protocol Rules:**
1. **Transaction $T_i$ issues $R(X)$:**
   - If $TS(T_i) < W\text{-}TS(X)$: $T_i$ is trying to read a value that was already overwritten by a younger transaction. **Reject $R(X)$ and abort/rollback $T_i$**.
   - If $TS(T_i) \ge W\text{-}TS(X)$: Execute $R(X)$, and set $R\text{-}TS(X) = \max(R\text{-}TS(X), TS(T_i))$.
2. **Transaction $T_i$ issues $W(X)$:**
   - If $TS(T_i) < R\text{-}TS(X)$: A younger transaction has already read the value of $X$. **Reject $W(X)$ and abort $T_i$**.
   - If $TS(T_i) < W\text{-}TS(X)$: A younger transaction has already written $X$. **Reject $W(X)$ and abort $T_i$**.
   - Otherwise: Execute $W(X)$, and set $W\text{-}TS(X) = TS(T_i)$.

### Thomas Write Rule (TWR)

Modifies Rule 2 when $TS(T_i) < W\text{-}TS(X)$:
- Instead of aborting $T_i$, simply **ignore (skip) the write operation $W(X)$** and proceed!
- Reasoning: A younger transaction has already written the latest value of $X$; $T_i$'s write is obsolete.
- **Key GATE fact:** Thomas Write Rule can generate schedules that are **view serializable but NOT conflict serializable**!

### Deadlock handling

A **deadlock** occurs when two or more transactions are in a simultaneous circular wait, each waiting for a lock held by another.
- **Deadlock detection:** Build the **Wait-For Graph (WFG)** (directed edge $T_i \to T_j$ if $T_i$ waits for a lock held by $T_j$). A directed cycle $\iff$ deadlock.
- **Deadlock prevention using timestamps:**
  - **Wait-Die (Non-preemptive):** If older requests lock held by younger: older is allowed to wait. If younger requests lock held by older: younger dies (aborts and restarts with original timestamp). *"Older waits, younger dies."*
  - **Wound-Wait (Preemptive):** If older requests lock held by younger: older preempts (wounds/aborts) younger. If younger requests lock held by older: younger is allowed to wait. *"Older wounds, younger waits."*
  - Both protocols are **deadlock-free** because edges in the WFG can only point in one direction of timestamp ordering!

**Worked example 4 (Timestamp ordering execution trace).**  
Let transactions $T_1, T_2, T_3$ have timestamps $TS(T_1) = 10, TS(T_2) = 20, TS(T_3) = 30$. Initial $R\text{-}TS(X) = W\text{-}TS(X) = 0$.  
Trace the following operations under Basic TO:
1. $T_1: R(X) \implies TS(T_1) = 10 \ge W\text{-}TS(X) (0)$. Allowed! $R\text{-}TS(X) = \max(0, 10) = 10$.
2. $T_2: W(X) \implies TS(T_2) = 20 \ge R\text{-}TS(X) (10)$ and $20 \ge W\text{-}TS(X) (0)$. Allowed! $W\text{-}TS(X) = 20$.
3. $T_3: R(X) \implies TS(T_3) = 30 \ge W\text{-}TS(X) (20)$. Allowed! $R\text{-}TS(X) = \max(10, 30) = 30$.
4. $T_1: W(X) \implies TS(T_1) = 10$. Check: $TS(T_1) < R\text{-}TS(X) (30)$. Since $10 < 30$, younger transaction $T_3$ already read a later value. **Operation rejected; $T_1$ is aborted and rolled back!**

## Formulas and facts to memorise

| Item | Formula / Rule | When to use |
|---|---|---|
| Conflict condition | $i \neq j$, same item $X$, at least one write | Identifying edges in precedence graph |
| Conflict serializability | Precedence graph has **no directed cycles** | Testing conflict serializability |
| View serializable shortcut | If no blind writes: View Serial $\iff$ Conflict Serial | Quick test for view serializability |
| Recoverable schedule | $W_i(X) < R_j(X) \implies C_i < C_j$ | Checking recoverability |
| Cascadeless schedule | $W_i(X) < R_j(X) \implies C_i < R_j(X)$ | Checking freedom from cascading aborts |
| Strict schedule | $W_i(X) < R_j(X) \implies C_i < R_j(X)$ AND $W_i(X) < W_j(X) \implies C_i < W_j(X)$ | Checking strictness |
| 2PL lock rule | No lock can be acquired after any lock is released | Testing whether schedule obeys 2PL |
| Strict 2PL | Basic 2PL + hold all exclusive ($X$) locks until commit/abort | Guarantees strictness and serializability |
| Wait-Die | Older waits for younger; younger dies when requesting older | Non-preemptive deadlock prevention |
| Wound-Wait | Older preempts/wounds younger; younger waits for older | Preemptive deadlock prevention |
| Thomas Write Rule | If $TS(T_i) < W\text{-}TS(X)$, ignore $W_i(X)$ without aborting | Non-conflict-serializable view serial schedules |

## GATE traps

- **Serial schedule vs Serializable schedule:** A serial schedule has *no interleaving*. A serializable schedule is an interleaved concurrent schedule whose net effect is identical to some serial schedule.
- **Precedence graph checks CONFLICT serializability, NOT view serializability:** An acyclic precedence graph guarantees conflict serializability (and view serializability). But a cycle in the graph only means *not conflict serializable*; it may still be view serializable if blind writes exist.
- **2PL does NOT prevent deadlocks:** Basic 2PL and Strict 2PL can both experience deadlocks! Only Conservative 2PL (or timestamp protocols) prevent deadlocks.
- **Serializability does NOT imply recoverability:** A schedule can be conflict serializable and yet completely unrecoverable (e.g. $W_1(X), R_2(X), C_2, C_1$).
- **Cascadeless implies recoverable, but NOT vice-versa:** Recoverable only requires $C_1 < C_2$. Cascadeless requires $C_1 < R_2(X)$.
- **Deadlock detection vs prevention:** Wait-For Graph (WFG) is used for deadlock *detection*. Wait-Die and Wound-Wait are used for deadlock *prevention*.

## Connections

- [Constraints and normalization](constraints-and-normalization.md) — Normalization ensures schema consistency at rest; transaction concurrency ensures consistency across interleaved runtime state changes.
- [Operating systems: Synchronization](../13-operating-systems/synchronization.md) — Mutual exclusion and lock acquisition share the exact same theoretical foundations as 2PL and deadlock avoidance.
- [Operating systems: Deadlocks](../13-operating-systems/deadlocks.md) — Resource allocation graphs in OS map directly to Wait-For Graphs in DBMS.

## Practice

**Q1 (MCQ).** Consider the following schedule $S$ involving transactions $T_1$ and $T_2$:
$$S: R_1(X),\; W_1(X),\; R_2(X),\; W_2(X),\; R_1(Y),\; W_1(Y),\; C_1,\; C_2$$
Which of the following statements is TRUE?  
(A) $S$ is conflict serializable and recoverable.  
(B) $S$ is conflict serializable but NOT recoverable.  
(C) $S$ is NOT conflict serializable but recoverable.  
(D) $S$ is NEITHER conflict serializable NOR recoverable.  
<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:**  
*Conflict serializability:*
- On item $X$: $W_1(X)$ precedes $R_2(X)$ and $W_2(X) \implies T_1 \to T_2$.
- On item $Y$: Only $T_1$ accesses $Y$, so no conflicting pairs between different transactions.
- Precedence graph edges: only $T_1 \to T_2$.
- The graph has no cycle $\implies S$ is **conflict serializable** (equivalent serial order $T_1 \to T_2$).

*Recoverability:*
- $T_2$ reads $X$ written by $T_1$ ($W_1(X) < R_2(X)$).
- Commit order: $C_1$ precedes $C_2$ ($C_1 < C_2$).
- Since $T_1$ commits before $T_2$, the schedule is **recoverable**!  
(Notice it is not cascadeless, since $R_2(X)$ occurs before $C_1$).  
Hence, option (A) is correct.
</details>

**Q2 (MCQ).** Consider schedule $S$ over transactions $T_1, T_2, T_3$:
$$S: R_1(A),\; R_2(A),\; W_1(B),\; W_2(B),\; R_3(B),\; W_1(A),\; C_1,\; C_2,\; C_3$$
The number of directed cycles in the precedence graph of $S$ is:  
(A) 0  
(B) 1  
(C) 2  
(D) 3  
<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:**  
Identify all conflicting pairs:
- On item $A$:
  - $R_2(A)$ precedes $W_1(A) \implies T_2 \to T_1$.
- On item $B$:
  - $W_1(B)$ precedes $W_2(B) \implies T_1 \to T_2$.
  - $W_1(B)$ precedes $R_3(B) \implies T_1 \to T_3$.
  - $W_2(B)$ precedes $R_3(B) \implies T_2 \to T_3$.

Edges in the precedence graph:
- $T_2 \to T_1$
- $T_1 \to T_2$
- $T_1 \to T_3$
- $T_2 \to T_3$

There is a cycle between $T_1$ and $T_2$: $T_1 \to T_2 \to T_1$.  
The number of directed cycles is 1.
</details>

**Q3 (NAT).** Consider the following schedule $S$ involving transactions $T_1, T_2, T_3, T_4$:
$$S: R_1(X),\; W_2(X),\; R_3(X),\; W_4(Y),\; R_1(Y),\; W_3(Z),\; R_2(Z)$$
The number of distinct equivalent serial schedules for $S$ is _______.  
<details><summary>Answer</summary>

**Answer:** 0  
**Solution:**  
Check conflicting pairs:
1. Item $X$:
   - $R_1(X)$ precedes $W_2(X) \implies T_1 \to T_2$.
   - $W_2(X)$ precedes $R_3(X) \implies T_2 \to T_3$.
2. Item $Y$:
   - $W_4(Y)$ precedes $R_1(Y) \implies T_4 \to T_1$.
3. Item $Z$:
   - $W_3(Z)$ precedes $R_2(Z) \implies T_3 \to T_2$.

Edges in graph:
- $T_1 \to T_2$
- $T_2 \to T_3$
- $T_3 \to T_2$ (Conflict! $W_3(Z)$ before $R_2(Z)$)
- $T_4 \to T_1$

Notice the directed cycle between $T_2$ and $T_3$: $T_2 \to T_3 \to T_2$.  
Because the precedence graph contains a cycle, $S$ is **not conflict serializable**.  
Number of equivalent serial schedules = 0.
</details>

**Q4 (MSQ).** Which of the following statements is/are TRUE regarding concurrency control protocols?  
(A) Basic Two-Phase Locking (2PL) guarantees freedom from deadlocks.  
(B) Strict 2PL guarantees that schedules are both conflict serializable and strict.  
(C) Under the Timestamp Ordering protocol, deadlocks are impossible.  
(D) If a schedule has no blind writes, view serializability is strictly broader than conflict serializability.  
<details><summary>Answer</summary>

**Answer:** (B), (C)  
**Solution:**  
- (A) is FALSE: Basic 2PL can deadlock (e.g. $T_1$ locks $X$ and waits for $Y$; $T_2$ locks $Y$ and waits for $X$).
- (B) is TRUE: Strict 2PL holds exclusive locks until commit/abort, ensuring strictness and conflict serializability.
- (C) is TRUE: Timestamp Ordering aborts and restarts older/younger conflicting transactions; transactions never wait circularly, so deadlocks cannot occur.
- (D) is FALSE: In the absence of blind writes, view serializability is equivalent to conflict serializability.
</details>

**Q5 (MCQ).** Consider the Wait-Die and Wound-Wait schemes for deadlock prevention. Suppose transaction $T_{\text{old}}$ (timestamp 10) requests a lock held by transaction $T_{\text{young}}$ (timestamp 25). What actions will be taken under Wait-Die and Wound-Wait, respectively?  
(A) Wait-Die: $T_{\text{old}}$ waits; Wound-Wait: $T_{\text{young}}$ is wounded (aborted).  
(B) Wait-Die: $T_{\text{old}}$ dies; Wound-Wait: $T_{\text{old}}$ waits.  
(C) Wait-Die: $T_{\text{young}}$ dies; Wound-Wait: $T_{\text{old}}$ is wounded.  
(D) Wait-Die: $T_{\text{old}}$ waits; Wound-Wait: $T_{\text{old}}$ waits.  
<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:**  
- **Wait-Die:** Non-preemptive. If older requests lock held by younger, older is allowed to **wait** ($T_{\text{old}}$ waits). If younger requests older, younger dies.
- **Wound-Wait:** Preemptive. If older requests lock held by younger, older preempts / **wounds** younger ($T_{\text{young}}$ aborts and restarts). If younger requests older, younger waits.  
Therefore, (A) is correct.
</details>

**Q6 (NAT).** Consider a database with transactions $T_1, T_2, T_3$ with timestamps 100, 200, 300 respectively. Initially, $R\text{-}TS(X) = W\text{-}TS(X) = 50$.  
The following sequence of operations arrives:
1. $T_2: R(X)$
2. $T_3: W(X)$
3. $T_1: W(X)$
If the system uses the Basic Timestamp Ordering protocol, which step causes a transaction to abort? Enter the step number (1, 2, or 3).  
<details><summary>Answer</summary>

**Answer:** 3  
**Solution:**  
- Step 1 ($T_2: R(X)$ with $TS = 200$):  
  $TS(T_2) \ge W\text{-}TS(X)$ ($200 \ge 50$). Valid.  
  $R\text{-}TS(X) = \max(50, 200) = 200$.
- Step 2 ($T_3: W(X)$ with $TS = 300$):  
  $TS(T_3) \ge R\text{-}TS(X)$ ($300 \ge 200$) and $TS(T_3) \ge W\text{-}TS(X)$ ($300 \ge 50$). Valid.  
  $W\text{-}TS(X) = 300$.
- Step 3 ($T_1: W(X)$ with $TS = 100$):  
  Check read timestamp: $TS(T_1) < R\text{-}TS(X)$ ($100 < 200$).  
  Also $TS(T_1) < W\text{-}TS(X)$ ($100 < 300$).  
  A younger transaction has already read and overwritten $X$.  
  Under Basic TO, Step 3 is rejected and transaction $T_1$ is aborted.  
Answer is 3.
</details>

**Q7 (MSQ).** Which of the following schedules is/are cascadeless?  
(A) $W_1(A),\; W_2(B),\; C_1,\; R_2(A),\; C_2$  
(B) $W_1(A),\; R_2(A),\; C_1,\; C_2$  
(C) $W_1(A),\; C_1,\; W_2(A),\; C_2$  
(D) $R_1(A),\; W_2(A),\; C_1,\; C_2$  
<details><summary>Answer</summary>

**Answer:** (A), (C), (D)  
**Solution:**  
A schedule is cascadeless iff a transaction reads a value only after the transaction that wrote it has committed ($W_i(X) < R_j(X) \implies C_i < R_j(X)$).
- (A): $T_2$ reads $A$ via $R_2(A)$. The write $W_1(A)$ is followed by $C_1$ before $R_2(A)$ ($C_1 < R_2(A)$). Cascadeless!
- (B): $R_2(A)$ occurs *before* $C_1$ ($W_1(A) < R_2(A) < C_1$). $T_2$ reads uncommitted write of $T_1$. NOT cascadeless (cascading abort possible).
- (C): $W_1(A) < C_1 < W_2(A)$. There is no uncommitted read at all (no read occurs). Cascadeless!
- (D): $R_1(A)$ reads before $W_2(A)$ writes. No transaction reads from an uncommitted write. Cascadeless!  
Thus, (A), (C), and (D) are cascadeless.
</details>
