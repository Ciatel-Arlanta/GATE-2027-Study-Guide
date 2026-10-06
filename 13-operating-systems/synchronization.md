# Synchronization and concurrency

> **Paper:** CS · **Priority:** P0 · **Plan topics:** critical sections, mutual exclusion, semaphores, monitors, classical synchronisation problems
> **Prerequisites:** [Processes and threads](processes-threads-syscalls.md) · **Leads to:** [Deadlocks](deadlocks.md)

## Quick glance
- Race condition: result depends on interleaving of unsynchronised accesses.
- Critical-section solution needs mutual exclusion, progress, and bounded waiting.
- Semaphore wait/down decrements or blocks; signal/up increments and may wake a waiter.
- Binary semaphore can guard one resource; counting semaphore tracks multiple identical resources.
- Producer-consumer uses `empty`, `full`, and `mutex`; acquire in the correct order.

## 1. Interleaving and critical section
Two threads execute `x=x+1`. Each may read x, compute, then write. Starting x=0, both can read 0 and both write 1: one update is lost. Protect the read-modify-write as one critical section.

## 2. Semaphores
For a semaphore S, `wait(S)` obtains a permit or blocks; `signal(S)` releases a permit. For a bounded buffer of capacity N, initialise `empty=N`, `full=0`, `mutex=1`. Producer: wait(empty), wait(mutex), insert, signal(mutex), signal(full). Consumer reverses empty/full roles.

## 3. Classical problems
Readers-writers asks for concurrent readers while writers need exclusive access. Dining philosophers demonstrates deadlock when each holds one fork and waits for another; impose a global order or allow at most N−1 philosophers to compete.

## GATE traps
- Semaphore operations must be atomic.
- Signalling before the protected update can expose invalid state.
- A binary semaphore and a mutex are related but not identical in ownership semantics.
- Deadlock, starvation, and race condition are distinct failures.

## Connections
- [Deadlocks](deadlocks.md) — lock ordering prevents circular wait.
- [Processes and threads](processes-threads-syscalls.md) — threads share address space and need coordination.
- [CPU scheduling](cpu-scheduling.md) — blocking changes which process can run.

## Practice
**Q1.** Name the three classic critical-section requirements.
<details><summary>Answer</summary> Mutual exclusion, progress, bounded waiting.</details>

**Q2.** Initial semaphore S=3; perform 5 successful wait operations. What happens?
<details><summary>Answer</summary> Three proceed; the fourth blocks (assuming no signal); fifth cannot execute past the blocked wait.</details>

**Q3.** What is the race in unsynchronised `x++`?
<details><summary>Answer</summary> Read, increment, write are separate steps; interleaving can lose an update.</details>
