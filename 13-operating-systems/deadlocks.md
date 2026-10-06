# Deadlocks

> **Paper:** CS · **Priority:** P1 · **Plan topics:** deadlock characterisation, prevention, avoidance, detection, recovery, Banker’s algorithm
> **Prerequisites:** [Processes, threads, syscalls](processes-threads-syscalls.md) · [Synchronization](synchronization.md)

## Quick glance
- Deadlock requires all four Coffman conditions: mutual exclusion, hold-and-wait, no preemption, circular wait.
- Prevention breaks at least one necessary condition.
- Avoidance grants requests only if the resulting state is safe; Banker’s algorithm tests this.
- Detection permits deadlock, then finds cycles/unsatisfied processes; recovery preempts or aborts.
- Safe state means some completion order exists; unsafe does not necessarily mean already deadlocked.

## 1. Resource-allocation graph
Process→resource means request; resource→process means allocation. A cycle is necessary and sufficient for deadlock when each resource type has one instance; with multiple instances it is necessary but not sufficient.

## 2. Safe sequence and Banker’s test
For each process, Need = Max − Allocation. Repeatedly find an unfinished process with Need ≤ Available; pretend it finishes, then return its allocation to Available. If all can finish, state is safe.

Example Available=(3,3), P1 Need=(1,2), Allocation=(2,0); P2 Need=(2,1), Allocation=(0,2). P1 can finish, Available becomes (5,3); P2 can then finish. Safe sequence P1,P2.

## GATE traps
- Need is Max − Allocation, not Max − Available.
- Compare vectors componentwise.
- A state can be unsafe without an existing deadlock.
- Cycle inference depends on number of instances per resource type.

## Connections
- [Synchronization](synchronization.md) — locks and resource acquisition can create circular wait.
- [CPU scheduling](cpu-scheduling.md) — deadlock differs from starvation and poor utilisation.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — cycles represent wait dependencies.

## Practice
**Q1.** List Coffman conditions.
<details><summary>Answer</summary> Mutual exclusion, hold-and-wait, no preemption, circular wait.</details>

**Q2.** A process has Max 7 units and Allocation 3. Need?
<details><summary>Answer</summary> 4 units.</details>

**Q3.** Does unsafe always mean deadlocked?
<details><summary>Answer</summary> No; it means no guaranteed safe completion sequence under current claims.</details>
