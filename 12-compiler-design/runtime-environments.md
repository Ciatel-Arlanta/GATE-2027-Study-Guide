# Runtime Environments

> **Paper:** CS · **Priority:** P1 · **Plan topics:** Runtime environments
> **Prerequisites:** [Functions, recursion and structures in C](../05-c-programming/functions-recursion-structures.md) · [Syntax-directed translation](syntax-directed-translation.md) · **Leads to:** [Intermediate code generation](intermediate-code-generation.md)

## Quick glance

- At run time a program's memory is organised as **code, static data, heap, stack** (stack and heap grow toward each other). Each procedure call gets an **activation record (frame)** on the stack; recursion needs it, so languages with recursion use stack allocation.
- Activation record fields (top of the frame to bottom, one convention): **actual parameters, return value, control (dynamic) link, access (static) link, saved machine status (return address, registers), local variables, temporaries.**
- **Control link** points to the caller's frame (dynamic chain, used to pop). **Access link** points to the frame of the lexically enclosing procedure (static chain, used to reach non-local variables). **Display** replaces the chain by an array: `d[i]` = the latest frame at nesting depth i.
- **Static (lexical) scoping:** a non-local name resolves by the program text (the enclosing declarations). **Dynamic scoping:** it resolves by the call chain (the most recent caller that declared it). Most languages (C, Java, Python) are statically scoped.
- **Parameter passing:** call by value (copy in), by reference (address passed, aliasing), by value-result (copy in and out at return), by name (textual substitution, re-evaluated at each use, using the caller's environment).
- **Heap:** dynamically allocated storage (`malloc`/`new`); managed by free lists and (in managed languages) **garbage collection**: reference counting (fails on cycles), mark-and-sweep, copying, generational.
- Total calls vs depth: `fib(5)` makes 15 calls but at most depth 5 (plus main) activation records alive at once.
- #1 trap: dynamic scoping and call-by-name give different output from static scoping/call-by-reference on the same code; read which discipline the question states.

## 1. Storage organisation

```text
low addresses
+-------------------+
| code (text)       |  fixed size, read-only
+-------------------+
| static data       |  globals, static locals, constants (size known at compile time)
+-------------------+
| heap              |  grows upward   (malloc / new)
|        ...        |
|        ...        |
| stack             |  grows downward (activation records)
+-------------------+
high addresses
```

| Region | Holds | Allocated | Lifetime |
| --- | --- | --- | --- |
| Code | machine instructions | compile/load time | whole run |
| Static | global and static variables, string constants | compile time (fixed address) | whole run |
| Stack | activation records: parameters, locals, temporaries, links | call time | from call to return (LIFO) |
| Heap | `malloc`, `new` objects | explicit request | until `free` / garbage collected |

**Allocation strategies.** *Static allocation* (early FORTRAN): every name has one fixed address, so no recursion and no reentrancy. *Stack allocation*: recursion supported; locals are not retained between calls. *Heap allocation*: needed when a value outlives the call that created it, or its size is unknown at compile time.

**Dangling reference.** Returning the address of a local variable or using freed heap memory: the storage is gone while a pointer remains. (See [Pointers, arrays, strings](../05-c-programming/pointers-arrays-strings.md).)

## 2. Activation records

An **activation** is one execution of a procedure body. Its record is created at the call and destroyed at the return. Fields (order varies by implementation):

| Field | Purpose |
| --- | --- |
| Actual parameters | values/addresses passed by the caller |
| Return value | slot for the function result |
| Control link (dynamic link) | pointer to the caller's activation record |
| Access link (static link) | pointer to the activation record of the lexically enclosing procedure (only for languages with nested procedures) |
| Saved machine status | return address, saved registers, saved program counter and frame pointer |
| Local data | the procedure's local variables |
| Temporaries | intermediate values from expression evaluation |

```text
caller's frame
+-------------------+
| actual params     |   <- pushed by the caller
| return value slot |
+-------------------+
| control link      |   -> caller's frame
| access link       |   -> lexical parent's frame
| saved registers / |
| return address    |
+-------------------+
| local variables   |
| temporaries       |   <- frame pointer / stack pointer region
+-------------------+
```

**Calling sequence.** *Caller:* evaluate arguments, push them, save the return address, transfer control. *Callee prologue:* save old frame pointer and registers, set the frame pointer, allocate locals. *Callee epilogue:* store the return value, restore registers and frame pointer, deallocate the frame, jump to the return address. *Caller after return:* retrieve the return value, pop the arguments. Work is split so that what only the caller knows (arguments) is done by the caller, and what only the callee knows (its local-data size) by the callee; the prologue code then exists once instead of at every call site.

### 2.1 Worked example: counting activations and stack depth

Recursive `int fib(int n) { return n < 2 ? n : fib(n-1) + fib(n-2); }` called as `fib(5)` from main.

- Number of calls c(n): c(0) = c(1) = 1, c(n) = 1 + c(n−1) + c(n−2) giving c(2)=3, c(3)=5, c(4)=9, **c(5)=15** (checked by program).
- Maximum number of activation records alive at the same time: the deepest chain fib(5) → fib(4) → fib(3) → fib(2) → fib(1) has **5** frames; with main, **6**. In general depth(fib(n)) = n for n ≥ 1.
- For `fact(n)` recursion (n calls to depth n, n ≥ 1), the stack holds n frames at the deepest point (plus main); the control links form a chain n → n−1 → … → 1 → main.

## 3. Scope: static vs dynamic

**Static (lexical) scope:** the binding of a non-local name is determined by where the procedure is *written*. **Dynamic scope:** by where it is *called from* (the most recent active binding).

**Worked example 1.**

```c
int x = 10;
void f()  { printf("%d", x); }
void g()  { int x = 20; f(); }
int main() { g(); return 0; }
```

- Static scoping: `f` refers to the global x: prints **10**.
- Dynamic scoping: call chain main → g → f; the most recent x is g's local: prints **20**.

**Worked example 2 (nested procedures, Pascal style).**

```text
program main;
  var x = 1;
  procedure P;
    var x = 2;
    procedure Q;  begin print(x) end;
    procedure R;  var x = 3; begin Q end;
  begin R end;
begin P end.
```

- Static: Q is declared inside P, so `x` in Q is P's x: prints **2**.
- Dynamic: chain main → P → R → Q; the latest x is R's: prints **3**.

**Implementing non-local access (static scoping with nested procedures).**
- **Access links:** each frame has a pointer to the frame of its lexical parent. To reach a variable declared k nesting levels outward, follow k access links, then add the variable's offset. In example 2, Q (depth 3) reading main's x (depth 1) follows 2 links; a variable of P (depth 2) one link.
- **Display:** a global array d[1..max depth]. d[i] always points to the most recent frame at depth i. Accessing a variable at depth i: `d[i] + offset`, a constant-time access instead of a chain walk. On a call to a procedure at depth j the old d[j] is saved in the frame and d[j] set to the new frame; it is restored on return.
- Setting the access link of a new frame: if the callee is nested directly inside the caller (depth j = i + 1), the link is the caller's own frame; if the callee is at depth j ≤ i (a sibling, the caller itself, or an enclosing procedure), follow i − j + 1 access links from the caller's frame to reach the callee's lexical parent.

**Dynamic scoping implementations.** *Deep access:* search the control-link chain for the name. *Shallow access:* keep a central table/stack per name, save and restore its value at entry and exit of each scope. Dynamic scope makes static type checking difficult and is rare (older Lisp, shell variables, Perl's `local`).

## 4. Parameter passing

| Method | What is passed | Effect |
| --- | --- | --- |
| Call by value | copy of the value | callee changes do not affect the caller |
| Call by reference | address of the actual (an l-value) | callee writes change the caller's variable; aliasing possible |
| Call by value-result (copy-restore) | value in, final value copied back at return | like reference, but updates visible only at return; copy-out order matters |
| Call by name | the actual expression, re-evaluated in the caller's environment at each use | like macro substitution; the expression may change meaning if variables in it change |

C passes everything by value (pointers give reference behaviour); C++ has reference parameters; Algol 60 used call by name.

**Worked example 1: aliasing.** `void f(int x, int y) { x = x + 1; y = y + x; }` with `a = 2; f(a, a); print(a)`:

| Method | Steps | a afterwards |
| --- | --- | --- |
| Value | x=2, y=2 are copies; a untouched | **2** |
| Reference | x, y are both a: a = a + 1 = 3; then a = a + a = 6 | **6** |
| Name | x, y both mean a: same as reference | **6** |
| Value-result | in: x=2, y=2; x = 3; y = 2 + 3 = 5; out: copy x then y, so a = 3 then a = 5 | **5** (3 if copy-out is right to left) |

**Worked example 2: call by name versus the others with a changing index.** `swap(x, y) { t = x; x = y; y = t; }` called as `swap(i, a[i])` with `i = 1`, `a[1] = 2`, `a[2] = 7`:

| Method | Execution | Final state |
| --- | --- | --- |
| Value | swaps copies only | i = 1, a = (2, 7) unchanged |
| Reference | y is bound to the address of a[1] at call time. t = i = 1; i = a[1] = 2; a[1] = t = 1 | **i = 2, a[1] = 1, a[2] = 7** |
| Value-result | in: x = 1, y = a[1] = 2 (addresses of i and a[1] fixed at call). swap: x = 2, y = 1. out: i = 2, a[1] = 1 | **i = 2, a[1] = 1, a[2] = 7** |
| Name | t = i = 1; **i = a[i]** = a[1] = 2; **a[i] = t** now means a[2] = 1 (i changed!) | **i = 2, a[1] = 2, a[2] = 1** |

Call by name re-evaluates `a[i]` with the new value of i, so it modifies a different element: the classic reason swap fails with call by name (Jensen's device). All four outcomes were checked by a program.

## 5. Heap management and garbage collection

**Heap allocation.** The allocator keeps a **free list** of blocks; strategies: first fit, best fit, worst fit, next fit. Problems: **external fragmentation** (free memory scattered in pieces too small), internal fragmentation (rounding up), **coalescing** adjacent free blocks on `free`. Malloc blocks carry a header with size.

| Garbage-collection scheme | Idea | Notes |
| --- | --- | --- |
| Reference counting | each object keeps a count of references; freed at 0 | immediate, incremental; **fails on cycles**; overhead on every pointer assignment |
| Mark and sweep | mark everything reachable from the roots (stack, static data); sweep the heap and free unmarked | handles cycles; pauses; fragmentation remains |
| Mark and compact | as above plus slide live objects together | removes fragmentation |
| Copying (semispace) | copy live objects from from-space to to-space, then swap | cost proportional to live data only; needs twice the memory |
| Generational | collect young objects often (most die young) | good average performance |

**Memory-safety terms.** *Memory leak:* allocated memory never freed and no longer reachable (in C, forgetting `free`). *Dangling pointer:* points to freed storage. *Double free:* freeing twice.

## 6. Other runtime topics

- **Symbol table at run time** is not needed in compiled code; names become addresses (offsets from the frame pointer, or fixed static addresses).
- **Variable-length data** (e.g. dynamic arrays declared locally) is placed in the frame using a pointer to the actual data kept at a fixed offset, or on the heap.
- **Tail recursion** can reuse the current frame (optimisation turning it into a loop).
- **Inlining** removes call overhead and activation records.
- **Exceptions** unwind the control-link chain, running cleanup at each frame.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Memory layout | code, static, heap ↑, ↓ stack | segment questions |
| Control link | caller's frame (dynamic chain) | popping / debugging |
| Access link | lexical parent's frame (static chain) | non-local access |
| Non-local access cost | k links for k levels (display: O(1)) | efficiency questions |
| fib(n) calls | c(n) = 1 + c(n−1) + c(n−2) | call-counting |
| Stack depth | max simultaneous frames = longest call chain | stack-size questions |
| Static vs dynamic scope | text vs call chain | scoping outputs |
| Parameter passing | value / reference / value-result / name | trace outputs |
| Reference counting | cannot reclaim cycles | GC questions |

## GATE traps

- **Static scoping is not "the most recent assignment"**: a callee sees the declarations around its *definition*, not its caller's locals (except under dynamic scoping).
- Access link ≠ control link: control link goes to the **caller**, access link to the **lexically enclosing** procedure; they differ except when a procedure is called from its parent.
- Call by name is **re-evaluated** at each use; call by reference binds the address once at call time. `swap(i, a[i])` distinguishes them.
- Value-result final value depends on the copy-out order when two parameters alias.
- The heap and stack grow toward each other; stack allocation is automatic, heap allocation is not (and is not freed on return).
- Recursion needs stack allocation; static allocation cannot support it.
- Number of activation records at once = depth of the call chain, not the total number of calls.
- Reference counting cannot collect circular structures; mark-and-sweep can.

## Connections

- [Functions, recursion and structures in C](../05-c-programming/functions-recursion-structures.md) — C's call-by-value, recursion and local storage are exactly the stack frames above.
- [Pointers, arrays, strings](../05-c-programming/pointers-arrays-strings.md) — dangling pointers, memory leaks, and heap allocation with `malloc`/`free`.
- [Syntax-directed translation](syntax-directed-translation.md) — computes the offsets and widths that determine frame layout.
- [Intermediate code generation](intermediate-code-generation.md) — `param`, `call`, `return` instructions realise the calling sequence.
- [Memory management](../13-operating-systems/memory-management.md) — the OS provides the address space; the compiler's stack and heap sit inside it.
- [Virtual memory](../13-operating-systems/virtual-memory.md) — stack growth and heap growth are served by demand paging.
- [Arrays, stacks and queues](../07-data-structures/arrays-stacks-queues.md) — the call stack.
- [Instruction sets and addressing](../10-computer-organization/instruction-sets-and-addressing.md) — frame-pointer-relative addressing, call/return instructions.

## Practice

**Q1 (MCQ).** In which part of the run-time memory are local variables of a recursive function stored?
(A) static data (B) heap (C) stack (D) code segment

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** Each activation gets its own record on the stack, so each recursive call has its own copy.

</details>

**Q2 (NAT).** `fib(n)` is defined as n if n < 2, else fib(n−1) + fib(n−2). How many calls (including the initial one) are made to evaluate `fib(5)`?

<details><summary>Answer</summary>

**Answer:** 15  
**Solution:** c(0)=c(1)=1, c(2)=3, c(3)=5, c(4)=9, c(5)=1+9+5=15.

</details>

**Q3 (NAT).** What does this program print under static scoping, and under dynamic scoping?
```text
int x = 5;
void f() { print(x); }
void g() { int x = 7; f(); }
g();
```
Answer as the sum of the two printed values.

<details><summary>Answer</summary>

**Answer:** 12  
**Solution:** Static: f uses the global x = 5. Dynamic: f is called from g, whose x = 7 is the most recent binding. 5 + 7 = 12.

</details>

**Q4 (NAT).** For `void f(int x, int y) { x = x + 1; y = y + x; }` called as `f(a, a)` with a = 2 and **call by reference**, the final value of a is

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:** x and y alias a. x = x + 1 makes a = 3; y = y + x makes a = 3 + 3 = 6.

</details>

**Q5 (MCQ).** Which pointer in an activation record is used to access a variable declared in an enclosing (lexical) procedure?
(A) control link (B) access link (C) return address (D) frame pointer of the caller

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** The access (static) link points to the frame of the lexical parent. The control link follows the caller chain, which is a different path in general.

</details>

**Q6 (MSQ).** Which statements about garbage collection are true?
(A) Reference counting cannot reclaim cyclic structures. (B) Mark-and-sweep can reclaim cycles. (C) Copying collectors cost time proportional to the amount of garbage. (D) Mark-and-sweep never leaves fragmentation.

<details><summary>Answer</summary>

**Answer:** A, B  
**Solution:** (C) copying collectors cost proportional to the live data. (D) mark-and-sweep leaves free holes unless compacting.

</details>

**Q7 (NAT).** `swap(x, y) { t = x; x = y; y = t; }` is called as `swap(i, a[i])` with i = 1, a[1] = 2, a[2] = 7, using **call by name**. What is the final value of a[2]?

<details><summary>Answer</summary>

**Answer:** 1  
**Solution:** t = i = 1; x = y means i = a[i] = a[1] = 2; y = t means a[i] = t, and i is now 2, so a[2] = 1. Final i = 2, a[1] = 2, a[2] = 1.

</details>

**Q8 (MCQ).** `main` calls `f(3)`, where `int f(int n) { return n == 0 ? 0 : 1 + f(n-1); }`. The maximum number of activation records on the stack at the same time (counting main) is
(A) 3 (B) 4 (C) 5 (D) 6

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** The chain is main → f(3) → f(2) → f(1) → f(0): 5 frames at the deepest point.

</details>
