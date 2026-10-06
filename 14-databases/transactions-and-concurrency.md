# Transactions and concurrency control

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** ACID, schedules, serializability, recoverability, locking, timestamp ordering
> **Prerequisites:** [SQL](sql.md) · [Constraints and normalization](constraints-and-normalization.md)

## Quick glance
- ACID: atomicity, consistency, isolation, durability.
- Conflict serializable iff precedence graph is acyclic; edges follow conflicting operation order.
- Conflicts: same data item, different transactions, at least one write.
- Strict 2PL holds exclusive locks until commit/abort; guarantees conflict serializability and strictness.
- Recoverable schedule: a reader commits after the transaction it read from; cascadeless schedules avoid reading uncommitted writes.

## 1. Serializability
For each conflicting pair, add edge $T_i\to T_j$ if $T_i$'s operation occurs first. Example: `T1: W(A)` before `T2: R(A)` creates T1→T2. A cycle means not conflict serializable; a topological order gives equivalent serial order.

Read-read is not a conflict. Read-write, write-read, and write-write on the same item conflict across transactions.

## 2. Locking
Shared locks permit concurrent reads; exclusive lock permits one writer and excludes readers. Two-phase locking has a growing phase (acquire only) followed by shrinking (release only). Basic 2PL ensures conflict serializability but may deadlock. Strict 2PL retains write locks to transaction end, simplifying recovery.

## 3. Timestamp ordering and recovery
Each transaction receives timestamp TS(T). For item X, track read_TS and write_TS; reject an operation that would violate timestamp order. Write-ahead logging requires log record reach stable storage before corresponding data page, enabling redo/undo after failure.

## GATE traps
- Acyclic precedence graph tests conflict serializability, not view serializability in general.
- 2PL can deadlock; serializable does not mean deadlock-free.
- Recoverable does not imply cascadeless; cascadeless does imply recoverable.
- A schedule may be serializable yet unrecoverable if commit ordering is wrong.

## Connections
- [Synchronization](../13-operating-systems/synchronization.md) — locks coordinate concurrent access at OS level.
- [Normalization](constraints-and-normalization.md) — schema design reduces update anomalies.
- [SQL](sql.md) — transactions group statements atomically.

## Practice
**Q1.** Precedence graph has a directed cycle. Conflict serializable?
<details><summary>Answer</summary> No.</details>

**Q2.** Is read-read on same item a conflict?
<details><summary>Answer</summary> No; neither operation writes.</details>

**Q3.** Does 2PL prevent deadlocks?
<details><summary>Answer</summary> No. It guarantees conflict serializability but deadlocks remain possible.</details>
