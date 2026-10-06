# Arrays, Stacks and Queues

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Arrays; Stacks; Queues; Data-structure operations and complexity
> **Prerequisites:** [Pointers, arrays, strings](../05-c-programming/pointers-arrays-strings.md) · **Leads to:** [Linked lists](linked-lists.md), [Heaps](heaps.md), [Graph traversals](../08-algorithms/graph-traversals.md)

## Quick glance

- **1-D address:** `A[i]` (lower bound `L`) is at `base + (i - L) * w`.
- **2-D row-major** with bounds `[L1..U1][L2..U2]`: `base + ((i-L1)*N2 + (j-L2)) * w`, `N2 = U2-L2+1` (number of **columns**). **Column-major:** `base + ((j-L2)*N1 + (i-L1)) * w`, `N1` = number of rows.
- **Lower-triangular** (1-based, row-major, store `i >= j`): offset `i(i-1)/2 + (j-1)`; total `n(n+1)/2` cells.
- **Stack** = LIFO: push, pop, peek all O(1). **Queue** = FIFO: enqueue, dequeue O(1).
- **Infix to postfix:** operands go straight to output; an operator pops operators of **higher or equal** precedence first (equal pops only if left-associative; `^` is right-associative).
- **Valid stack permutations** of `1..n` pushed in order: Catalan `C(n) = (1/(n+1)) * C(2n, n)`: 1, 2, 5, 14, 42. For n = 3 the only impossible output is `3 1 2`.
- **Circular queue** of array size `n`: full when `(rear+1) % n == front` (one slot sacrificed), empty when `front == rear`.
- **Queue from two stacks** gives amortised O(1) per operation (each element moves at most twice); a single dequeue can cost O(n).
- A **deque** allows insertion/deletion at both ends; a **priority queue** removes the highest-priority element, usually a [heap](heaps.md).

## 1. Arrays

An **array** is a block of equal-sized cells in consecutive memory, reached by index. Because the address is computed by arithmetic, **access by index is O(1)**; search is O(n) unless sorted (then O(log n) by binary search); inserting or deleting in the middle shifts elements, **O(n)**.

| Operation | Unsorted | Sorted |
|---|---|---|
| Access `A[i]` | O(1) | O(1) |
| Search | O(n) | O(log n) |
| Insert at end (space available) | O(1) | O(n) (keep order) |
| Insert/delete at position | O(n) shifts | O(n) |

### 1.1 One-dimensional address

For `A[L..U]` with element size `w`: **address(`A[i]`) = base + (i - L)·w.** Number of elements `U - L + 1`.

**Worked example 1.** `A[-5..20]`, base 3000, `w = 2`. Address of `A[7]` = 3000 + (7 - (-5))·2 = 3000 + 24 = **3024**. Size = 26 elements = 52 bytes, last address 3000 + 25·2 = 3050.

### 1.2 Two-dimensional arrays

Memory is one-dimensional, so a matrix is flattened.

```text
 matrix (3 rows x 4 cols)           row-major order (C, Python lists)
 a00 a01 a02 a03                    a00 a01 a02 a03 | a10 a11 a12 a13 | a20 ...
 a10 a11 a12 a13
 a20 a21 a22 a23                    column-major order (Fortran, MATLAB)
                                    a00 a10 a20 | a01 a11 a21 | a02 a12 a22 | ...
```

For `A[L1..U1][L2..U2]`, `N1 = U1-L1+1` rows, `N2 = U2-L2+1` columns:

| Layout | Address of `A[i][j]` |
|---|---|
| Row-major | base + ((i-L1)·**N2** + (j-L2))·w |
| Column-major | base + ((j-L2)·**N1** + (i-L1))·w |

**Row-major multiplies the row offset by the number of columns; column-major multiplies the column offset by the number of rows.**

**Worked example 2.** `A[-2..5][3..9]`, base 1000, `w = 4`. Find `A[1][5]`.
- `N1 = 5 - (-2) + 1 = 8` rows; `N2 = 9 - 3 + 1 = 7` columns.
- Row-major: `(1-(-2))·7 + (5-3) = 3·7 + 2 = 23` elements, so 1000 + 23·4 = **1092**.
- Column-major: `(5-3)·8 + (1-(-2)) = 2·8 + 3 = 19` elements, so 1000 + 19·4 = **1076**.

**Worked example 3 (reverse question).** `int B[M][20]` is 0-based, row-major, `int` = 4 bytes, and `B[3][7]` is at 1288. Find the base and the address of `B[5][2]`.
- `B[3][7]` is element number `3·20 + 7 = 67`, i.e. 268 bytes from the base: base = 1288 - 268 = **1020**.
- `B[5][2]` is element `5·20 + 2 = 102`: 1020 + 408 = **1428**. (Check by difference: `(5-3)·20 + (2-7) = 35` elements = 140 bytes, 1288 + 140 = 1428 ✓.)

For 3-D `A[N1][N2][N3]` (0-based, row-major): offset = `(i·N2·N3 + j·N3 + k)·w`.

### 1.3 Triangular and special matrices

An `n×n` **lower-triangular** matrix has zeros above the diagonal; only `n(n+1)/2` cells are stored.

**Row-major, 1-based, element `(i, j)` with `i >= j`:** rows before row `i` hold `1 + 2 + ... + (i-1) = i(i-1)/2` elements, so
**offset = i(i-1)/2 + (j - 1)** (0-based position in the packed array).

**Worked example 4.** `n = 5`, element `a[4][2]`: `4·3/2 + (2-1) = 6 + 1 = 7`. Packed order: `a11 | a21 a22 | a31 a32 a33 | a41 a42 ...` → positions 0, 1-2, 3-5, 6-9: `a42` is at position 7 ✓. Storage = 15 cells instead of 25.

**Upper-triangular, row-major, 1-based, `i <= j`:** row `r` stores `n - r + 1` elements, so offset = `(i-1)·n - (i-1)(i-2)/2 + (j - i)`. Check `n = 4`, `(2,3)`: `1·4 - 0 + 1 = 5` ✓ (row 1 has 4 elements, then `a22` at 4, `a23` at 5).

| Matrix | Stored cells | Idea |
|---|---|---|
| Diagonal | n | one array |
| Tridiagonal (band of width 3) | 3n - 2 | offset for `a[i][j]` with `|i-j| <= 1`, row-major: `2i + j - 3` (1-based) |
| Symmetric | n(n+1)/2 | store one triangle, mirror the index |
| Sparse | number of non-zeros `t` | triplets `(row, col, value)` |

Check tridiagonal formula: `n = 4`, `a[2][1]`: rows before row 2 hold 2 elements (row 1: `a11,a12`), then `a21` is position 2; formula `2·2 + 1 - 3 = 2` ✓.

**Sparse matrix.** If most entries are zero, store only non-zeros as triplets (row, column, value) or in compressed row form. A matrix with `t` non-zeros uses about `3t` numbers; it beats a dense array when `t < mn/3`. Transpose of a triplet list is O(t + n) with the counting approach.

## 2. Stacks

A **stack** is a collection with **last-in, first-out (LIFO)** access, like a pile of plates: you can only touch the top. Operations: `push(x)`, `pop()`, `peek()/top()`, `isEmpty()`, `isFull()` (array version). **All O(1).**

```c
#define MAX 100
int stack[MAX], top = -1;               /* empty when top == -1 */
void push(int x) { if (top == MAX - 1) { /* overflow */ return; } stack[++top] = x; }
int  pop(void)   { if (top == -1)      { /* underflow */ return -1; } return stack[top--]; }
```

- Overflow: push on full. Underflow: pop on empty.
- **Linked implementation:** push/pop at the head of a singly linked list, O(1), no fixed capacity ([linked lists](linked-lists.md)).
- If `top` starts at `0` (pointing to the next free slot), the code is `stack[top++] = x` / `return stack[--top]`; read the convention in the question.

Uses: function-call frames ([recursion](../05-c-programming/functions-recursion-structures.md), [runtime environments](../12-compiler-design/runtime-environments.md)), expression evaluation, undo, backtracking, DFS ([graph traversals](../08-algorithms/graph-traversals.md)), balanced brackets.

### 2.1 Balanced parentheses

Scan left to right: push every opener; on a closer, the stack must be non-empty and its top must match, then pop. At the end the stack must be empty.

Trace on `{ [ ( ) ] } )`:

| Symbol | Action | Stack (bottom→top) |
|---|---|---|
| `{` | push | `{` |
| `[` | push | `{ [` |
| `(` | push | `{ [ (` |
| `)` | matches `(`, pop | `{ [` |
| `]` | matches `[`, pop | `{` |
| `}` | matches `{`, pop | empty |
| `)` | stack empty → **unbalanced** | |

For one bracket type only a counter is enough (never negative, zero at end). The maximum stack depth equals the maximum nesting depth.

### 2.2 Infix to postfix (operator-stack algorithm)

**Notations.** Infix `A+B`, prefix `+AB`, postfix `AB+`. Postfix and prefix need no parentheses and no precedence rules once written.

**Algorithm (postfix).** Scan the infix string:
1. Operand → output.
2. `(` → push.
3. `)` → pop to output until `(`; discard `(`.
4. Operator `o` → while the stack top is an operator with **higher precedence than `o`, or equal precedence and `o` is left-associative**, pop it to output; then push `o`.
5. At the end pop everything to output.

Precedence: `^` (right-assoc.) > `* /` > `+ -` (left-assoc.).

**Worked example 5.** `A*(B+C)/D-E`

| Token | Stack (bottom→top) | Output |
|---|---|---|
| A | | A |
| * | * | A |
| ( | * ( | A |
| B | * ( | AB |
| + | * ( + | AB |
| C | * ( + | ABC |
| ) | * | ABC+ |
| / | / (pop `*` since equal, left-assoc.) | ABC+* |
| D | / | ABC+*D |
| - | - (pop `/`, higher) | ABC+*D/ |
| E | - | ABC+*D/E |
| end | | **ABC+*D/E-** |

**Worked example 6 (right associativity).** `A^B^C*D`: first `^` pushed; second `^` has equal precedence but is right-associative, so it **does not** pop the first (stack `^ ^`); `*` pops both `^`:

| Token | Stack | Output |
|---|---|---|
| A | | A |
| ^ | ^ | A |
| B | ^ | AB |
| ^ | ^ ^ | AB |
| C | ^ ^ | ABC |
| * | * | ABC^^ |
| D | * | ABC^^D |
| end | | **ABC^^D*** |

(For a left-associative reading you would get `AB^C^D*`.)

**Prefix** is obtained by reversing the infix string (swapping parentheses), running the algorithm with the rule "pop only strictly higher precedence", and reversing the result; or by building the expression tree and printing preorder. For `A*(B+C)/D-E` the prefix is `-/*A+BCDE`.

**Maximum stack size:** the stack holds operators waiting for their right operand; `A+B*C-D/E` gives final output `ABC*+DE/-` and the operator stack never exceeds 2 (`+*`, `-/`).

### 2.3 Postfix evaluation

Scan; push operands; on an operator pop **two** values (first popped is the **right** operand), apply, push result. At the end the single value left is the answer.

**Worked example 7.** `5 3 2 * + 8 4 / -`

| Token | Action | Stack (bottom→top) |
|---|---|---|
| 5 | push | 5 |
| 3 | push | 5 3 |
| 2 | push | 5 3 2 |
| * | 3·2 = 6 | 5 6 |
| + | 5+6 = 11 | 11 |
| 8 | push | 11 8 |
| 4 | push | 11 8 4 |
| / | 8/4 = 2 | 11 2 |
| - | 11-2 = 9 | **9** |

**Worked example 8 (order matters).** `2 3 1 * + 9 -`: `3·1 = 3`, `2+3 = 5`, then push 9: stack `5 9`; `-` pops 9 (right) and 5 (left): `5 - 9 = -4`.

### 2.4 Stack permutations

Push `1, 2, ..., n` in this order; pops may be interleaved arbitrarily. Which output orders are possible? **The number of distinct output permutations is the Catalan number** `C(n) = C(2n, n)/(n+1)`.

| n | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| valid outputs | 1 | 2 | 5 | 14 | 42 |

For n = 3 the valid ones are `123, 132, 213, 231, 321`; **`312` is impossible**: to output 3 first, both 1 and 2 must already be inside with 2 above 1, so 1 cannot come before 2.

**Rule (the 312 pattern):** an output is invalid exactly when there exist positions `i < j < k` with output values `c, a, b` where `a < b < c` (a "312" subsequence).

**Worked example 9.** Is `4 3 5 6 1 2` (from input `1..6`) achievable? Check for a 312 pattern: the subsequence `6, 1, 2` is `c=6, a=1, b=2` with `a < b < c`, so it is **invalid**. Simulation agrees: after popping 6, stack holds `1 2` (2 on top), so 2 must come out before 1.

**Worked example 10 (counting by first element).** For `n = 4` the number of valid outputs starting with `k` is 5, 5, 3, 1 for k = 1, 2, 3, 4 (total 14). Start with 1: remaining 2,3,4 behave like n=3 → 5. Start with 4: everything is pushed then popped: 1 sequence.

**Another standard result:** push order `1..n`; the output `n, n-1, ..., 1` needs all pushes first (max stack size n); output `1, 2, ..., n` needs stack size 1.

### 2.5 Recursion uses a stack

Each call pushes a frame; the maximum depth equals the maximum stack height. Any recursion can be simulated by an explicit stack (iterative DFS, tree traversals). See [functions and recursion](../05-c-programming/functions-recursion-structures.md).

## 3. Queues

A **queue** is **first-in, first-out (FIFO)**: insert at the **rear**, remove at the **front**. Operations `enqueue`, `dequeue`, `front`, `isEmpty`, `isFull`.

### 3.1 Linear queue and its waste

With an array and two indices, after repeated enqueue/dequeue `front` drifts right and the space before `front` is never reused: the queue reports "full" while the array is mostly empty.

### 3.2 Circular queue

Treat the array as a ring: indices wrap with `% n`.

```c
#define N 5
int q[N], front = 0, rear = 0;           /* rear = next free slot */
int isEmpty() { return front == rear; }
int isFull()  { return (rear + 1) % N == front; }   /* one slot sacrificed */
void enqueue(int x) { if (!isFull())  { q[rear] = x;  rear  = (rear + 1) % N; } }
int  dequeue()      { if (!isEmpty()) { int x = q[front]; front = (front + 1) % N; return x; } return -1; }
```

Usable capacity is `N - 1` because `front == rear` must mean "empty". (Alternative: keep a `count`; then capacity is `N` and no slot is wasted. Another convention uses `front = rear = -1` for empty; GATE states it in the question.)

**Worked example 11.** `N = 5`, start `front = rear = 0`.

| Operation | q[0..4] | front | rear | Note |
|---|---|---|---|---|
| enqueue 10, 20, 30, 40 | 10 20 30 40 - | 0 | 4 | `(4+1)%5 = 0 == front`: full with 4 elements |
| dequeue twice | - - 30 40 - | 2 | 4 | returns 10, 20 |
| enqueue 50 | - - 30 40 50 | 2 | 0 | `rear` wrapped to 0 |
| enqueue 60 | 60 - 30 40 50 | 2 | 1 | stored at index 0; `(1+1)%5 = 2 == front`: full again |

The queue holds 30, 40, 50, 60 (4 = N - 1 elements), stored in a wrapped layout.

**Number of elements** in a circular queue: `(rear - front + N) % N` (with the sacrificed-slot convention).

### 3.3 Deque and priority queue

- **Deque (double-ended queue):** `insertFront/Back`, `deleteFront/Back`, all O(1) in a doubly linked list or circular array. *Input-restricted:* insertion at one end only. *Output-restricted:* deletion at one end only. A deque can act as both a stack and a queue.
- **Priority queue:** each element has a priority; `extract-min`/`max` returns the best. Array/list: O(1)+O(n) or O(n)+O(1); **binary heap: O(log n) insert and extract**. See [heaps](heaps.md).

### 3.4 Queue using two stacks

Keep stack `IN` and stack `OUT`.
- `enqueue(x)`: `push(IN, x)`.
- `dequeue()`: if `OUT` is empty, pop everything from `IN` and push into `OUT` (reverses order); then `pop(OUT)`.

**Worked example 12.** `enqueue 1,2,3`: IN = [1,2,3] (top 3). `dequeue`: OUT empty → move: pop 3→OUT, 2→OUT, 1→OUT; OUT = [3,2,1] with top 1; pop returns **1**. `enqueue 4`: IN = [4]. `dequeue` → OUT top = 2 (**2**), then 3, then OUT empty so move 4 and return 4. FIFO order 1,2,3,4 ✓.

**Amortised cost.** Each element is pushed at most twice (IN, OUT) and popped at most twice. Over `m` operations the total work is ≤ `4m`, so **amortised O(1) per operation**, although one `dequeue` can cost O(n) (the transfer). The worst-case single-operation cost is O(n).

### 3.5 Stack using queues

- **Costly push:** `push(x)`: enqueue `x` into queue `Q`, then rotate the previous `size-1` elements (dequeue and re-enqueue) so `x` becomes the front. `pop` = dequeue, O(1); push is O(n).
- **Costly pop (two queues):** push = enqueue O(1); `pop`: move all but the last element to the other queue, dequeue the last, swap roles; O(n).

Trace of the single-queue method, pushes 1, 2, 3: `Q = [1]`; push 2: enqueue → [1,2], rotate 1 element → [2,1]; push 3: enqueue → [2,1,3], rotate 2 → [3,2,1]. `pop` returns 3 ✓.

**With one of the two structures simulated by the other, one of push/pop must be O(n) (worst case).**

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| 1-D address | base + (i-L)·w | array NAT |
| Row-major | base + ((i-L1)·N2 + (j-L2))·w | 2-D address |
| Column-major | base + ((j-L2)·N1 + (i-L1))·w | 2-D address |
| Lower-triangular (1-based, row-major) | i(i-1)/2 + j - 1 | packed storage |
| Packed size | n(n+1)/2 | symmetric/triangular |
| Tridiagonal | 3n - 2 cells; offset 2i+j-3 | band matrices |
| Stack permutations | C(2n,n)/(n+1) | counting outputs |
| Invalid output pattern | a "312" subsequence | permutation check |
| Circular queue | full `(r+1)%n==f`, empty `f==r`, size `(r-f+n)%n` | queue conditions |
| Two-stack queue | amortised O(1), worst O(n) | amortised analysis |
| Postfix evaluation | pop right operand first | evaluation |

## GATE traps

- **Row- vs column-major:** row-major multiplies by the number of **columns**; mixing up `N1`/`N2` is the most common error. Also check the lower bounds (subtract `L`).
- **Counting elements with bounds:** `U - L + 1`, not `U - L`.
- **Equal precedence, right-assoc.:** `^` does not pop `^`; `-` and `/` do pop equal-precedence operators of the same side.
- **Postfix subtraction/division:** the operand popped first is the right operand: `a b -` is `a - b`.
- **Circular queue capacity** is `n - 1` in the `front == rear` empty convention; with a counter or the `-1` convention read what the question says is "full".
- **Stack permutations:** the count is Catalan, not `n!`; test with a 312 pattern.
- **Amortised vs worst-case:** two-stack queue is O(1) amortised but O(n) worst-case for one operation.
- **Overflow vs underflow:** push on full vs pop on empty.
- **Stack output with pops allowed anywhere:** the *order of pushes is fixed*; only pops are interleaved.

## Connections

- [Pointers, arrays, strings](../05-c-programming/pointers-arrays-strings.md) — the same address formulas in C; array decay.
- [Linked lists](linked-lists.md) — pointer-based stack/queue/deque implementations.
- [Heaps](heaps.md) — priority queue implemented as a binary heap.
- [Trees and BST](trees-and-bst.md) — traversals use a stack (DFS-like) or queue (level order).
- [Graph traversals](../08-algorithms/graph-traversals.md) — DFS uses a stack, BFS a queue.
- [Runtime environments](../12-compiler-design/runtime-environments.md) — the call stack and activation records.
- [Parsing](../12-compiler-design/parsing.md) — a pushdown automaton and shift-reduce parsers keep their state on a stack.
- [Context-free languages and PDA](../11-theory-of-computation/context-free-languages-and-pda.md) — stack as the memory of a PDA; balanced-parentheses language.
- [CPU scheduling](../13-operating-systems/cpu-scheduling.md) — FCFS and round robin use a FIFO ready queue.
- [Memory hierarchy and cache](../10-computer-organization/memory-hierarchy-and-cache.md) — row-major traversal and spatial locality.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — amortised cost intuition.

## Practice

**Q1 (NAT, easy).** `A[1..10][1..10]` is stored row-major, base 100, 2 bytes per element. Address of `A[4][3]`?

<details><summary>Answer</summary>

**Answer:** 166  
**Solution:** `(4-1)·10 + (3-1) = 32` elements; 100 + 32·2 = 166.

</details>

**Q2 (NAT).** The same array stored column-major. Address of `A[4][3]`?

<details><summary>Answer</summary>

**Answer:** 146  
**Solution:** `(3-1)·10 + (4-1) = 23` elements; 100 + 23·2 = 146.

</details>

**Q3 (MCQ).** Convert `a + b * c - d / e` to postfix.  (A) `abc*+de/-`  (B) `ab+c*de/-`  (C) `abc*+d/e-`  (D) `abcde/*+-`

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** `b*c` first → `bc*`; `a + bc*` → `abc*+`; `d/e` → `de/`; then subtraction: `abc*+de/-`.

</details>

**Q4 (NAT).** Evaluate the postfix expression `8 2 3 ^ / 2 3 * + 5 1 * -` where `^` is exponentiation.

<details><summary>Answer</summary>

**Answer:** 2  
**Solution:** `2 3 ^` = 8; `8 8 /` = 1; `2 3 *` = 6; `1 + 6` = 7; `5 1 *` = 5; `7 - 5` = **2**.

</details>

**Q5 (MCQ).** Input `1 2 3 4` is pushed in order with pops interleaved arbitrarily. Which output is impossible?  (A) 4 3 2 1  (B) 1 4 3 2  (C) 3 4 1 2  (D) 2 1 4 3

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** `3 4 1 2` contains the pattern `3, 1, 2` (c=3, a=1, b=2). After popping 3 and 4, the stack holds `1 2` with 2 on top, so 2 must precede 1. The others are valid.

</details>

**Q6 (NAT).** How many distinct permutations of `1..5` can be produced by a stack with the input order fixed?

<details><summary>Answer</summary>

**Answer:** 42  
**Solution:** Catalan `C(5) = C(10,5)/6 = 252/6 = 42`.

</details>

**Q7 (NAT).** A circular queue uses an array of size 8 with the convention empty `front == rear`, full `(rear+1) % 8 == front`. Currently `front = 5`, `rear = 2`. How many elements does it hold, and how many more can be enqueued?

<details><summary>Answer</summary>

**Answer:** 5 elements, 2 more  
**Solution:** Elements = `(2 - 5 + 8) % 8 = 5` (indices 5, 6, 7, 0, 1). Capacity is 7, so 7 - 5 = 2 more.

</details>

**Q8 (MSQ).** A queue is implemented with two stacks (`IN`/`OUT`, transfer only when `OUT` is empty). Which are true? (A) Every operation is O(1) worst case. (B) Enqueue is always O(1). (C) A dequeue can take O(n). (D) A sequence of `m` operations costs O(m) in total. (E) Each element is transferred from `IN` to `OUT` at most once.

<details><summary>Answer</summary>

**Answer:** B, C, D, E  
**Solution:** Enqueue is one push. A dequeue after many enqueues transfers n elements, O(n), so (A) is false. Each element is transferred between the stacks at most once and pushed/popped a constant number of times, giving O(m) in total (amortised O(1)).

</details>
