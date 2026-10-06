# Functions, Recursion, Structures and Dynamic Memory

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Functions; Recursion in C; Structures; Memory and pointer reasoning
> **Prerequisites:** [Pointers, arrays, strings](pointers-arrays-strings.md) · **Leads to:** [Linked lists](../07-data-structures/linked-lists.md), [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md)

Assumptions: 64-bit model (`int` 4, `short` 2, `long`/`double`/pointer 8), each scalar aligned to its own size.

## Quick glance

- **C passes everything by value** (copies). To change a caller's variable, pass its address (`&x`) and use `*p`.
- A function call creates an **activation record** (frame) on the stack holding parameters, return address, locals. Frames die on return.
- **Parameter-passing modes:** value, reference, value-result (copy-in/copy-out), name (textual substitution, re-evaluated at each use). The same program gives different answers; the traced example below separates all four.
- **Recursion** = base case + smaller self-call; **stack depth** is the space cost. Calls of naive fib(n): `2*Fib(n+1) - 1`.
- **Head recursion:** work after the call (reversed order). **Tail recursion:** call is last.
- **Static variables in recursion** are shared by all activations.
- **Struct size** = members laid out with padding so each is aligned to its size; total rounded up to the largest member alignment. Reordering can shrink a struct.
- **Union** members share one location; size = largest member (rounded to alignment).
- `p->x` ≡ `(*p).x`. Self-referential struct holds a **pointer** to its own type, never the type itself.
- `malloc(n)` uninitialised; `calloc(k, s)` zeroed; `realloc` may move the block; every `malloc` needs one `free`.

## 1. Functions and call by value

A function has a return type, name, parameters and body. A **prototype** declares it before use. Each call copies the argument values into the parameter variables.

```c
void swap(int a, int b) { int t = a; a = b; b = t; }      // swaps only local copies
void swap2(int *a, int *b) { int t = *a; *a = *b; *b = t; }
int x = 1, y = 2;
swap(x, y);     // x = 1, y = 2
swap2(&x, &y);  // x = 2, y = 1
```
The pointer itself is still passed by value: assigning to the parameter pointer `a = NULL;` does not change the caller's pointer; writing `*a = ...` changes the pointee. To modify a caller's pointer, pass `int **`.

**Arrays** decay to pointers when passed, so elements are shared with the caller; `sizeof` inside gives the pointer size.

**Returning pointers.** Never return the address of a non-static local (it dies with the frame). Allowed: address of a static, a global, a `malloc`ed block, or a caller's object.

**`main`, `void`, defaults.** `f()` in old C means "unspecified parameters", `f(void)` means none. If the return statement is missing in a non-void function and the value is used, behaviour is undefined.

### Activation records and the call stack

```text
main() calls f(3), which calls f(2):

 high addr  +----------------+
            | main's frame   |  locals of main
            +----------------+
            | f(3): n=3      |  return addr, param n=3, locals
            +----------------+
            | f(2): n=2      |  <- stack pointer (top)
            +----------------+
 low addr
```
Frames are pushed on call and popped on return, so the **most recent call finishes first** (LIFO). Local variables of one call do not interfere with those of another call of the same function. Frame layout, access links and displays: [Runtime environments](../12-compiler-design/runtime-environments.md).

## 2. Parameter-passing modes

| Mode | What the callee gets | Writes visible to caller? | When the actual argument's address/value is computed |
|---|---|---|---|
| Value | a copy of the value | no | at call |
| Reference | alias of the actual (same location) | immediately | location fixed at call |
| Value-result (copy-in, copy-out) | copy in; copy back at return | at return | value copied at call; **location** normally fixed at call |
| Name | the *expression text*, substituted into the body | yes, via the expression | **re-evaluated at every use** |

C itself is only "value" (pointers simulate reference). GATE writes pseudo-code and asks for the result.

**Worked example 1 (one program, four modes).** Global `i = 1`, array `A = {10, 20, 30}` (index from 0).
```text
procedure P(x):
    i = i + 1
    x = x + 5
    A[1] = A[1] + 1000
call P(A[i])        // i = 1 at the call, so the actual is A[1] = 20
```
- **Value.** `x` = 20 (copy). `i` = 2. `x` = 25 (local only). `A[1]` = 1020. Result **i = 2, A = {10, 1020, 30}**.
- **Reference.** `x` aliases `A[1]` (address fixed at the call). `i` = 2. `A[1]` = 25 (through x). `A[1]` = 1025. Result **A = {10, 1025, 30}**.
- **Value-result.** `x` = 20 copied in. `i` = 2. `x` = 25. `A[1]` = 1020 (direct). At return `x` (25) is copied back into the location noted at call, `A[1]`: **A = {10, 25, 30}** (the direct update 1020 is overwritten). If a book evaluates the address at return instead, it would write `A[2]`; state which convention you use. GATE usually fixes the location at call.
- **Name.** Every `x` is replaced by `A[i]`. `i = i + 1` → i = 2. `A[i] = A[i] + 5` now means `A[2] = A[2] + 5` → 35. `A[1] = 1020`. Result **i = 2, A = {10, 1020, 35}**.

| Mode | i | A[0] | A[1] | A[2] |
|---|---|---|---|---|
| Value | 2 | 10 | 1020 | 30 |
| Reference | 2 | 10 | 1025 | 30 |
| Value-result | 2 | 10 | 25 | 30 |
| Name | 2 | 10 | 1020 | 35 |

**Worked example 2 (the swap that breaks under call by name).** `swap(x, y) { t = x; x = y; y = t; }`, `i = 1`, `A = {5, 2, 7}`, call `swap(i, A[i])`.
- **Reference:** `x` = &i, `y` = &A[1]. `t` = 1; `i` = A[1] = 2; `A[1]` = 1. Result **i = 2, A = {5, 1, 7}** (a proper swap).
- **Name:** `t = i` → 1. `x = y` is `i = A[i]` → `i = A[1]` = 2. `y = t` is `A[i] = t` with the **new** `i` = 2 → `A[2] = 1`. Result **i = 2, A = {5, 2, 1}** (wrong element changed). Name passing re-evaluates `A[i]` after `i` changed.

## 3. Recursion

A recursive function calls itself on a smaller input until a **base case** returns directly. Each call has its own frame, so depth `d` costs `O(d)` stack space. Missing base case → stack overflow.

**Worked example 3 (head vs tail position: printing order).**
```c
void f(int n) { if (n > 0) { printf("%d ", n); f(n - 1); printf("%d ", n); } }
```
`f(3)`: print 3 → `f(2)`: print 2 → `f(1)`: print 1 → `f(0)` returns → print 1 → print 2 → print 3. Output **3 2 1 1 2 3**. Code before the call runs on the way down, code after it on the way up.

**Worked example 4 (binary representation).**
```c
void g(int n) { if (n == 0) return; g(n / 2); printf("%d", n % 2); }
```
`g(10)` → `g(5)` → `g(2)` → `g(1)` → `g(0)` returns. Printing on the way up: `1%2=1`, `2%2=0`, `5%2=1`, `10%2=0` → **1010**. (Printing before the call would give 0101, reversed.)

**Worked example 5 (returning values; digit sum).** `int s(int n) { if (n == 0) return 0; return n % 10 + s(n / 10); }`
`s(2027) = 7 + s(202) = 7 + 2 + s(20) = 7 + 2 + 0 + s(2) = 7 + 2 + 0 + 2 + s(0) = 11`.

**Worked example 6 (counting calls; fib).**
```c
int fib(int n) { if (n <= 1) return n; return fib(n-1) + fib(n-2); }
```
Let `C(n)` be the total number of calls made by `fib(n)`: `C(0) = C(1) = 1`, `C(n) = C(n-1) + C(n-2) + 1`.
| n | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| C(n) | 1 | 1 | 3 | 5 | 9 | 15 | 25 |

Closed form `C(n) = 2·Fib(n+1) − 1` (Fib(1)=Fib(2)=1): `C(5) = 2·8 − 1 = 15`. Call tree of `fib(4)` (9 nodes):
```text
fib(4)
├─ fib(3)
│   ├─ fib(2)
│   │   ├─ fib(1)
│   │   └─ fib(0)
│   └─ fib(1)
└─ fib(2)
    ├─ fib(1)
    └─ fib(0)
```
Running time is exponential (`Θ(φ^n)`, φ ≈ 1.618) because subproblems repeat ([dynamic programming](../08-algorithms/dynamic-programming.md) removes this); stack depth is only `n`.

**Worked example 7 (doubling).** `int h(int x) { if (x < 1) return 1; return h(x-1) + h(x-1); }` → `h(0) = 1`, `h(n) = 2h(n-1)` ⇒ `h(n) = 2^n`, number of calls `2^(n+1) − 1`.

**Worked example 8 (static variable inside recursion).**
```c
int f(int n) { static int c = 0; if (n == 0) return c; c += n; return f(n - 1); }
```
`f(4)`: `c` = 4 → 7 → 9 → 10, then `f(0)` returns 10. A **second** call `f(4)` starts with `c` = 10: 14 → 17 → 19 → 20, returns **20** (the static is shared by all activations and all calls).

**Worked example 9 (Ackermann).** `A(0,n) = n+1`, `A(m,0) = A(m-1,1)`, `A(m,n) = A(m-1, A(m,n-1))`.
- `A(1,n) = n+2` (by induction), `A(2,n) = 2n+3`.
- `A(2,2) = 7`, `A(2,3) = A(1, A(2,2)) = A(1,7) = 9`.
Grows faster than any primitive recursive function; deep recursion.

**Worked example 10 (ruler printing).**
```c
void r(int n) { if (n == 0) return; r(n - 1); printf("%d ", n); r(n - 1); }
```
`r(1)` → `1`. `r(2)` → `1 2 1`. `r(3)` → `1 2 1 3 1 2 1` (7 outputs = 2^3 − 1, 15 calls).

**Tail recursion** (the recursive call is the last action, no work remains) can be turned into a loop, as compilers often do:
```c
int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }   // gcd(48,18): (18,12) → (12,6) → (6,0) → 6
```

**Recursion → recurrence → complexity** (see [asymptotic analysis](../08-algorithms/asymptotic-analysis.md)):

| Code shape | Recurrence | Time | Stack |
|---|---|---|---|
| `f(n-1)` plus O(1) | T(n) = T(n−1) + 1 | O(n) | O(n) |
| `f(n/2)` plus O(1) (binary search) | T(n) = T(n/2) + 1 | O(log n) | O(log n) |
| two calls on n−1 (Hanoi) | T(n) = 2T(n−1) + 1 | 2^n − 1 moves | O(n) |
| fib | T(n) = T(n−1) + T(n−2) + 1 | Θ(φ^n) | O(n) |
| two calls on n/2 plus O(n) (merge sort) | T(n) = 2T(n/2) + n | O(n log n) | O(log n) frames (+ buffers) |

## 4. Structures and unions

A **structure** groups members of different types; members have distinct addresses laid out in declaration order.

```c
struct student { int roll; char grade; float cgpa; };
struct student s1 = {7, 'A', 8.5f}, *p = &s1;
s1.roll = 8;            // member access via object
p->roll = 9;            // via pointer; same as (*p).roll
struct student s2 = s1; // whole-struct copy (arrays inside are copied too)
typedef struct student Student;   // alias; usually combined with the declaration
```
Struct assignment and passing a struct to a function copy all bytes (large structs are costly; pass a pointer). Structs cannot be compared with `==`.

**Self-referential structure** (linked-list node): a member of the struct's own type would be infinitely large, so it holds a **pointer**:
```c
struct node { int data; struct node *next; };   // sizeof = 16 (4 + 4 padding + 8)
```
Chapter use: [Linked lists](../07-data-structures/linked-lists.md).

**Layout rule (assumed x86-64):** each member is placed at the next offset that is a multiple of its alignment (alignment of a scalar = its size; of an array = its element's; of a struct = the max of its members). The total size is rounded up to a multiple of the struct's alignment.

**Worked example 11 (padding).**

| Struct | Layout (offset: member) | Size |
|---|---|---|
| `{char c; int i; char d;}` | 0:c, 1–3 pad, 4:i, 8:d, 9–11 pad | **12** |
| `{int i; char c; char d;}` (reordered) | 0:i, 4:c, 5:d, 6–7 pad | **8** |
| `{char c; double d; int i;}` | 0:c, 1–7 pad, 8:d, 16:i, 20–23 pad | **24** |
| `{char c[5]; short s;}` | 0–4:c, 5 pad, 6:s | **8** |
| `{int a; char *p;}` | 0:a, 4–7 pad, 8:p | **16** |
| `{char a; short b; char c; int d; char e;}` | 0:a, 2:b, 4:c, 8:d, 12:e, 13–15 pad | **16** |

Putting members in decreasing size order minimises padding. `struct {char c; int i;} arr[10]` has `sizeof` = 8 · 10 = 80. (`#pragma pack(1)` or `__attribute__((packed))` removes padding, if the question says so.)

**Worked example 12 (struct pointer arithmetic).**
```c
struct P { int x, y; } a[3] = { {1,2}, {3,4}, {5,6} };
struct P *p = a;
printf("%d ", (++p)->x);   // p → a[1]; prints 3
printf("%d ", p->y++);     // prints 4, then a[1].y = 5
printf("%d ", (int)(p - a));  // 1
printf("%d",  ++p->x);     // ++(p->x): a[1].x = 4; prints 4
```
`->` has higher precedence than `++`, so `p->y++` increments the member, not `p`. Output **3 4 1 4**; final `a = {{1,2},{4,5},{5,6}}`.

**Unions.** All members start at offset 0 and overlap; only the last written member holds a meaningful value. `sizeof(union)` = largest member, rounded up to the largest alignment.
```c
union U { int i; char c[4]; } u;
u.i = 0x41424344;
printf("%c", u.c[0]);    // little-endian: lowest byte 0x44 first → 'D'
```
| Union | Size |
|---|---|
| `{int i; char c[5]; double d;}` | max(4, 5, 8) = 8, alignment 8 → **8** |
| `{int i; char c[5];}` | max = 5, alignment 4 → rounded to **8** |

Also: `enum {A, B = 5, C}` gives A = 0, B = 5, C = 6 (next integer after the previous constant). Bit-fields (`int f : 3;`) pack members into bits; layout is implementation-dependent, so GATE rarely relies on it.

## 5. Dynamic memory

Stack variables have automatic lifetime. For sizes unknown at compile time or data that must outlive a call, request **heap** memory (`<stdlib.h>`):

| Function | Effect |
|---|---|
| `malloc(n)` | n bytes, **uninitialised**; `NULL` on failure |
| `calloc(k, s)` | `k*s` bytes, **zero-filled** |
| `realloc(p, n)` | resize; may **move** the block (old pointer invalid; contents preserved up to the smaller size) |
| `free(p)` | release; `p` becomes dangling; `free(NULL)` is a no-op |

```c
int *p = malloc(3 * sizeof(int));     // idiom: sizeof(*p) also works
p[0] = 1; p[1] = 2; p[2] = 3;
p = realloc(p, 5 * sizeof(int));      // (safer: use a temporary in case of NULL)
p[3] = 4; p[4] = 5;
free(p);
struct node *n = malloc(sizeof(struct node));  n->data = 5;  n->next = NULL;
```
No cast of `malloc`'s result is needed in C. Bugs: [pointer bugs table](pointers-arrays-strings.md) (leak, double free, use after free).

| | Stack | Heap |
|---|---|---|
| Allocation | automatic on call | explicit `malloc` |
| Release | automatic on return | explicit `free` |
| Speed | very fast (pointer bump) | slower (allocator) |
| Fragmentation | none | possible |
| Size limit | small (a few MB) | large |

**Worked example 13 (leak count).**
```c
for (int k = 0; k < 4; k++) { int *q = malloc(16); }
```
Each iteration allocates a block whose only pointer `q` dies at the end of the iteration: **4 blocks × 16 bytes = 64 bytes leaked** (unreachable but allocated).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Parameter modes | value, reference, value-result, name | pseudo-code tracing |
| fib calls | `C(n) = C(n-1)+C(n-2)+1 = 2Fib(n+1)-1` | call counting |
| Double recursion | `h(n) = 2h(n-1)` → 2^n; calls 2^(n+1) − 1 | doubling |
| Hanoi | 2^n − 1 moves | recurrences |
| Recursion space | O(depth) frames | space complexity |
| Struct size | pad to alignment; round to max alignment | `sizeof(struct)` |
| Union size | max member, rounded to alignment | `sizeof(union)` |
| `p->m` | `(*p).m` | struct pointers |
| `malloc`/`calloc` | uninit / zeroed | memory |
| Static in recursion | one copy shared | trace |

## GATE traps

- **"C has call by reference"** is false; pointers are passed **by value**.
- **Swap with plain ints does nothing**; swap through pointers works; **call by name** can give a wrong swap (`swap(i, A[i])`).
- **Struct padding:** forgetting trailing padding (`char,int,char` is 12, not 6 or 9); remember reordering changes size.
- **Union size is the max**, not the sum; also rounded to alignment.
- **`p->y++`** increments the member; `(p++)->y` moves the pointer.
- **Static variable in a recursive function** is shared, so not reset per call.
- **Order of printing** around the recursive call: before = pre-order/descending, after = post-order/ascending.
- **Number of calls vs value:** `fib(5)` returns 5 but makes 15 calls.
- **Returning a local's address**, **freeing twice**, **using `sizeof(p)` on a malloc'ed pointer** (gives 8, not the block size).
- **`realloc` can move**, invalidating other pointers into the old block.
- **Missing base case or wrong argument** (e.g. `f(n)` calling `f(n)`) → infinite recursion.

## Connections

- [Pointers, arrays, strings](pointers-arrays-strings.md) — pointer parameters, array decay, dangling pointers.
- [C basics and expressions](c-basics-and-expressions.md) — static locals, scope, undefined evaluation order of arguments (`f(i++, i++)`).
- [Runtime environments](../12-compiler-design/runtime-environments.md) — activation records, parameter-passing modes, static vs dynamic links, heap vs stack allocation.
- [Linked lists](../07-data-structures/linked-lists.md) — self-referential structs and `malloc`ed nodes are the implementation.
- [Trees and BST](../07-data-structures/trees-and-bst.md) — recursive traversals are recursion on struct pointers; stack depth = tree height.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — turning recursive code into recurrences and solving them.
- [Dynamic programming](../08-algorithms/dynamic-programming.md) — memoisation removes the exponential recomputation seen in `fib`.
- [Processes, threads, syscalls](../13-operating-systems/processes-threads-syscalls.md) — each process has its own stack/heap; `malloc` ultimately uses `brk`/`mmap` system calls.
- [Memory management](../13-operating-systems/memory-management.md) — fragmentation in the heap parallels external fragmentation.

## Practice

**Q1 (NAT, easy).** `int f(int n) { if (n <= 1) return 1; return f(n-1) + f(n-2); }`. How many calls (including the first) does `f(6)` make?

<details><summary>Answer</summary>

**Answer:** 25  
**Solution:** C(0)=C(1)=1, C(2)=3, C(3)=5, C(4)=9, C(5)=15, C(6)=15+9+1=25. (Also 2·Fib(7) − 1 = 2·13 − 1 = 25.)

</details>

**Q2 (MCQ).** For `void f(int n) { if (n > 0) { f(n-1); printf("%d ", n); f(n-1); } }`, what does `f(3)` print?  (A) 3 2 1 1 2 3  (B) 1 2 1 3 1 2 1  (C) 1 2 3 1 2 3  (D) 3 2 1 3 2 1

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `f(1)` prints `1`; `f(2)` = `f(1) 2 f(1)` = `1 2 1`; `f(3)` = `1 2 1 3 1 2 1`.

</details>

**Q3 (NAT).** On the assumed 64-bit model, `sizeof(struct { char a; short b; char c; int d; char e; })`?

<details><summary>Answer</summary>

**Answer:** 16  
**Solution:** a@0, b@2 (align 2), c@4, d@8 (align 4, pad 5–7), e@12; end = 13, rounded up to a multiple of 4 → 16.

</details>

**Q4 (NAT).** `i = 1; A = {5, 2, 7};` and `swap(x, y) { t = x; x = y; y = t; }` is called as `swap(i, A[i])` using call by name. What is `A[2]` afterwards?

<details><summary>Answer</summary>

**Answer:** 1  
**Solution:** `t = i` = 1. `i = A[i]` = A[1] = 2. `A[i] = t` is now `A[2] = 1`. Final i = 2, A = {5, 2, 1}.

</details>

**Q5 (NAT).** `int f(int n) { static int c = 0; if (n == 0) return c; c += n; return f(n-1); }`. What does the second call `f(4)` (made after a first call `f(4)`) return?

<details><summary>Answer</summary>

**Answer:** 20  
**Solution:** First call leaves c = 4+3+2+1 = 10. The second starts from c = 10: 14, 17, 19, 20; `f(0)` returns 20.

</details>

**Q6 (MSQ).** Which are true? (A) `calloc` zero-initialises memory. (B) After `free(p)`, `p` is set to NULL automatically. (C) `sizeof(union {int i; double d;})` = 8 on the assumed model. (D) `realloc` may return a different address. (E) A struct may contain a member of its own struct type.

<details><summary>Answer</summary>

**Answer:** A, C, D  
**Solution:** `free` does not modify `p` (it dangles). A struct cannot contain itself (infinite size), only a pointer to itself.

</details>

**Q7 (NAT).** With Ackermann `A(0,n)=n+1; A(m,0)=A(m-1,1); A(m,n)=A(m-1,A(m,n-1))`, find `A(2,3)`.

<details><summary>Answer</summary>

**Answer:** 9  
**Solution:** A(1,n) = n+2 and A(2,0)=A(1,1)=3, A(2,1)=A(1,3)=5, A(2,2)=A(1,5)=7, A(2,3)=A(1,7)=9 (A(2,n)=2n+3).

</details>

**Q8 (NAT, harder).** In pseudo-code with global `i = 1`, `A = {10, 20, 30}`, `P(x) { i = i + 1; x = x + 5; A[1] = A[1] + 1000; }` is called as `P(A[i])`. What is `A[1]` after the call under (a) call by value, (b) call by reference, (c) value-result, (d) call by name? Give the four values in order.

<details><summary>Answer</summary>

**Answer:** (a) 1020, (b) 1025, (c) 25, (d) 1020  
**Solution:** See Worked example 1. Value: x is a copy, only the direct update applies. Reference: +5 then +1000 on A[1]. Value-result: final copy-back overwrites A[1] with x = 25. Name: `x` means `A[i]` with i = 2, so +5 lands on A[2] (35), leaving A[1] = 1020.

</details>
