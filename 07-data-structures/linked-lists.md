# Linked Lists

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Linked lists; Data-structure operations and complexity
> **Prerequisites:** [Functions, recursion, structures](../05-c-programming/functions-recursion-structures.md), [Pointers, arrays, strings](../05-c-programming/pointers-arrays-strings.md) · **Leads to:** [Trees and BST](trees-and-bst.md), [Graphs](graphs.md), [Hashing](../08-algorithms/hashing.md)

## Quick glance

- A **node** = data + pointer(s) to neighbour(s): `struct node { int data; struct node *next; };`. The list is reached through a `head` pointer; the last node's `next` is `NULL`.
- **Insert/delete at head: O(1). Insert at tail: O(n)** (O(1) with a tail pointer). **Search and access by position: O(n).** No random access.
- Delete a node given **only a pointer to it** in a singly list: copy the successor's data into it and unlink the successor (O(1); fails for the last node). Otherwise you need the predecessor.
- **Reverse** by walking with three pointers `prev, curr, next`: O(n) time, O(1) space.
- **Middle:** `slow` moves 1, `fast` moves 2; when `fast` ends, `slow` is at index `floor(n/2)` (0-based).
- **Cycle detection (Floyd):** `slow`/`fast` meet iff a cycle exists. After the meeting, reset one pointer to `head` and move both by 1: they meet at the **cycle start**.
- **Merge** two sorted lists in O(m + n) relinking nodes, O(1) extra space.
- **Doubly linked:** `prev` + `next`; delete a given node in O(1). **Circular:** last points to first, so any node reaches all.
- The classic code trap: **order of pointer assignments** (overwriting a link before using it) and **NULL checks**.

## 1. Idea and node structure

Arrays store neighbours side by side, so inserting in the middle shifts everything. A **linked list** keeps elements anywhere in memory and joins them with pointers, like a treasure hunt where each clue tells you where the next one is.

```c
struct node { int data; struct node *next; };
struct node *head = NULL;                 /* empty list */
```

```text
head
 |
 v
[10|*]--->[20|*]--->[30|*]--->[40|NULL]
```

Memory per node: data + one pointer (16 bytes for `int` + pointer under LP64 with padding). The list needs no pre-declared size; memory is allocated per node with `malloc` ([dynamic memory](../05-c-programming/functions-recursion-structures.md)).

| Property | Array | Linked list |
|---|---|---|
| Access `i`-th | O(1) | O(n) |
| Insert/delete at front | O(n) | O(1) |
| Insert/delete after a known node | O(n) | O(1) |
| Memory overhead | none | one (or two) pointers per node |
| Locality / cache | excellent | poor |
| Size | fixed (or realloc) | dynamic |

## 2. Singly linked list operations

### 2.1 Traversal, search, length

```c
int length(struct node *h) { int c = 0; while (h) { c++; h = h->next; } return c; }
struct node *search(struct node *h, int x) { while (h && h->data != x) h = h->next; return h; }
```
Both are O(n). In `search`, the test `h &&` must come first (short-circuit) to avoid dereferencing `NULL`.

### 2.2 Insertion

```c
struct node *newnode(int x) {
    struct node *n = malloc(sizeof(struct node));
    n->data = x; n->next = NULL; return n;
}
/* at front: O(1) */
struct node *insertFront(struct node *head, int x) {
    struct node *n = newnode(x);
    n->next = head;            /* 1. new node points to old head */
    return n;                  /* 2. new node becomes head */
}
/* after a given node p: O(1) */
void insertAfter(struct node *p, int x) {
    struct node *n = newnode(x);
    n->next = p->next;         /* link the tail of the list FIRST */
    p->next = n;               /* then cut in */
}
/* at the end: O(n) walk (O(1) with a tail pointer) */
struct node *insertEnd(struct node *head, int x) {
    struct node *n = newnode(x);
    if (head == NULL) return n;
    struct node *t = head;
    while (t->next) t = t->next;
    t->next = n;
    return head;
}
```

**Order matters.** In `insertAfter`, doing `p->next = n;` first destroys the only reference to the rest of the list (it is leaked and `n->next` would point to itself).

### 2.3 Deletion

```c
struct node *deleteFront(struct node *head) {
    if (!head) return NULL;
    struct node *t = head; head = head->next; free(t); return head;
}
struct node *deleteValue(struct node *head, int x) {
    struct node *prev = NULL, *cur = head;
    while (cur && cur->data != x) { prev = cur; cur = cur->next; }
    if (!cur) return head;                 /* not found */
    if (!prev) head = cur->next;           /* deleting the head */
    else prev->next = cur->next;
    free(cur);
    return head;
}
```
Cost O(n) because the predecessor must be found.

**Pointer-to-pointer idiom** removes the special case for the head:
```c
void deleteValue2(struct node **pp, int x) {   /* call with &head */
    while (*pp && (*pp)->data != x) pp = &(*pp)->next;
    if (*pp) { struct node *t = *pp; *pp = t->next; free(t); }
}
```
`pp` always points at "the pointer that points at the current node" (first `head`, then some node's `next` field), so the same assignment `*pp = t->next` works for head and interior.

**Worked example 1 (delete given only a pointer, not the last node).** List `10 → 20 → 30 → 40`, pointer `p` to the node `20`, no head available.
```c
struct node *t = p->next;     /* node 30 */
p->data = t->data;            /* node now holds 30 */
p->next = t->next;            /* skips the old 30 node */
free(t);
```
Result `10 → 30 → 40` (in effect `20` was deleted). Fails if `p` is the last node (`p->next == NULL`). Also invalid if other pointers refer to the removed node.

### 2.4 Reversal

**Iterative.** Keep `prev` (reversed part), `curr` (current), `next` (saved successor).
```c
struct node *reverse(struct node *head) {
    struct node *prev = NULL, *curr = head, *next;
    while (curr) {
        next = curr->next;   /* save */
        curr->next = prev;   /* flip */
        prev = curr;         /* advance prev */
        curr = next;         /* advance curr */
    }
    return prev;
}
```

**Worked example 2.** List `1 → 2 → 3 → 4`.

| Step | prev | curr | next saved | List after the flip |
|---|---|---|---|---|
| start | NULL | 1 | | |
| 1 | 1 | 2 | 2 | `1 → NULL` |
| 2 | 2 | 3 | 3 | `2 → 1 → NULL` |
| 3 | 3 | 4 | 4 | `3 → 2 → 1 → NULL` |
| 4 | 4 | NULL | NULL | `4 → 3 → 2 → 1 → NULL` |

Returns `prev` = 4. **O(n) time, O(1) space.**

**Recursive.**
```c
struct node *rev(struct node *h) {
    if (h == NULL || h->next == NULL) return h;     /* base: 0 or 1 node */
    struct node *newHead = rev(h->next);            /* reverse the rest */
    h->next->next = h;                              /* the old successor now points back */
    h->next = NULL;                                 /* h becomes the tail */
    return newHead;
}
```
Trace on `1 → 2 → 3`: `rev(3)` returns 3. In `rev(2)`: `2->next` (= 3) `->next = 2`; `2->next = NULL`; now `3 → 2 → NULL`. In `rev(1)`: `1->next` (= 2) `->next = 1`; `1->next = NULL`: `3 → 2 → 1 → NULL`. Space O(n) for the stack.

**Printing in reverse without reversing** (recursion, "after" position): `void pr(struct node *h) { if (h) { pr(h->next); printf("%d ", h->data); } }`.

### 2.5 Middle node and k-th from end

**Middle (slow/fast).**
```c
struct node *mid(struct node *h) {
    struct node *s = h, *f = h;
    while (f && f->next) { s = s->next; f = f->next->next; }
    return s;
}
```
For `n` nodes, `slow` ends at index `floor(n/2)` (0-based): n = 5 → the 3rd node; n = 6 → the **4th** (the second of the two middles). One pass because `fast` covers the list while `slow` covers half.

**k-th from the end.** Advance `ahead` k nodes first, then move `ahead` and `behind` together; when `ahead` falls off, `behind` is the k-th from the end. O(n), one pass.

### 2.6 Cycle detection (Floyd's tortoise and hare)

A list with a cycle never reaches `NULL`. Let `slow` move 1 and `fast` move 2 each step.
```c
int hasCycle(struct node *h) {
    struct node *s = h, *f = h;
    while (f && f->next) { s = s->next; f = f->next->next; if (s == f) return 1; }
    return 0;
}
```
**Why they meet:** once both are inside the cycle, `fast` closes the gap by 1 per step, so it cannot jump over `slow`.

**Worked example 3.** Nodes 0..6 with `next` links `0→1→2→3→4→5→6→3` (tail length μ = 3, cycle length λ = 4).

| Step k | slow | fast |
|---|---|---|
| 1 | 1 | 2 |
| 2 | 2 | 4 |
| 3 | 3 | 6 |
| 4 | 4 | 4 |

They meet at node 4 after 4 steps. **Finding the cycle start:** put one pointer at `head` (node 0), keep the other at the meeting node 4, move both one step at a time: (0,4), (1,5), (2,6), (3,3): they meet at **node 3**, the start of the cycle. Reason: when they meet, `slow` has walked `k` steps with `k` a multiple of λ (here 4) and `fast` `2k`; the meeting node is `k - μ` steps into the cycle, so the remaining distance to the cycle start is `λ - (k - μ) ≡ μ (mod λ)` since `k` is a multiple of λ. That is exactly the `μ` steps a pointer from the head needs to reach the start. **Cycle length:** after meeting, keep one pointer fixed and count steps for the other to return: 4.

Time O(n), space O(1) (versus O(n) space with a hash set).

### 2.7 Merging sorted lists

```c
struct node *merge(struct node *a, struct node *b) {
    struct node dummy, *t = &dummy;
    dummy.next = NULL;
    while (a && b) {
        if (a->data <= b->data) { t->next = a; a = a->next; }
        else                    { t->next = b; b = b->next; }
        t = t->next;
    }
    t->next = a ? a : b;          /* append the remaining list */
    return dummy.next;
}
```

**Worked example 4.** `a = 1 → 4 → 7`, `b = 2 → 3 → 9`:

| Compare | Taken | Result so far |
|---|---|---|
| 1 vs 2 | 1 (a) | 1 |
| 4 vs 2 | 2 (b) | 1 2 |
| 4 vs 3 | 3 (b) | 1 2 3 |
| 4 vs 9 | 4 (a) | 1 2 3 4 |
| 7 vs 9 | 7 (a) | 1 2 3 4 7 |
| a empty | append 9 | 1 2 3 4 7 9 |

O(m + n) comparisons at most, no new nodes. The **dummy node** avoids special-casing the head. This is the merge step of merge sort on lists ([sorting](../08-algorithms/searching-and-sorting.md)); a linked list can be merge-sorted in O(n log n) time with O(log n) stack.

**Why list sorting prefers merge sort:** quicksort and heap sort need random access; insertion sort on a list is still O(n²) (but each insertion needs no shifting).

### 2.8 Typical GATE code-completion questions

**Move the last node to the front.**
```c
struct node *moveLast(struct node *head) {
    if (head == NULL || head->next == NULL) return head;
    struct node *prev = NULL, *last = head;
    while (last->next) { prev = last; last = last->next; }
    prev->next = NULL;       /* detach */
    last->next = head;       /* attach in front */
    return last;
}
```
On `1 → 2 → 3 → 4`: `prev` = 3, `last` = 4; list becomes `4 → 1 → 2 → 3`.

**Swap adjacent pairs** (`1 2 3 4 5 → 2 1 4 3 5`) and **rotate by k** are the same pointer-juggling drills. For each, write the new `next` of every affected node on paper before coding; every GATE completion question is solved by checking the answer's assignments for (a) the right order, (b) no lost reference, (c) NULL at the end.

**What does this do?**
```c
void f(struct node *h) { if (h == NULL) return; printf("%d ", h->data); if (h->next) f(h->next->next); printf("%d ", h->data); }
```
On `1 → 2 → 3 → 4 → 5`: prints `1 3 5 5 3 1` (visits alternate nodes going down, prints each again going up). On `1 → 2 → 3 → 4`: `1 3`, then `f(NULL)` returns: output `1 3 3 1`.

## 3. Doubly linked and circular lists

**Doubly linked:** `struct dnode { int data; struct dnode *prev, *next; };`

```text
NULL <-[ |10| ]<->[ |20| ]<->[ |30| ]-> NULL
```
- Traverse both ways; **delete a given node `p` in O(1)** (`p->prev->next = p->next; p->next->prev = p->prev;` with NULL checks).
- Insert after `p`: set the new node's `prev`/`next` first, then `p->next->prev`, then `p->next` (four pointer changes).
- Costs one extra pointer per node. The XOR-linked list stores `prev ^ next` in one field to save space (rarely examined).

**Circular singly linked:** the last node's `next` is `head`. Keeping a pointer to the **last** node gives O(1) access to both ends: `last->next` is the head. Insert at front: `n->next = last->next; last->next = n;`; insert at end: the same, then `last = n`. Traversal ends when you return to the start, not at `NULL`. Typical uses: round-robin scheduling, Josephus problem, circular buffers.

**Dummy/sentinel node:** a header node whose `next` is the first real node simplifies insert/delete at the head.

| Operation | Singly | Singly + tail | Doubly | Circular (last pointer) |
|---|---|---|---|---|
| Insert at front | O(1) | O(1) | O(1) | O(1) |
| Insert at end | O(n) | O(1) | O(1) with tail | O(1) |
| Delete first | O(1) | O(1) | O(1) | O(1) |
| Delete last | O(n) | **O(n)** | O(1) with tail | O(n) |
| Delete given node (pointer) | O(n) (O(1) trick) | O(n) | **O(1)** | O(n) |
| Search | O(n) | O(n) | O(n) | O(n) |

**Deleting the last node of a singly linked list is O(n) even with a tail pointer** because the predecessor must be found.

## 4. Linked list as other structures

- **Stack:** push/pop at the head, both O(1) ([stacks](arrays-stacks-queues.md)).
- **Queue:** enqueue at the tail, dequeue at the head, with head and tail pointers, both O(1).
- **Polynomial:** each node stores `(coefficient, exponent)`; addition is a merge on exponents.
- **Hash table chaining:** each bucket is a list ([hashing](../08-algorithms/hashing.md)).
- **Adjacency list** of a graph ([graphs](graphs.md)).
- **Skip list / unrolled list:** list variants that improve search/locality (beyond GATE).

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Access/search | O(n) | all lists |
| Insert/delete at head | O(1) | stack |
| Insert after known node | O(1) | pointer questions |
| Delete a given node | singly O(n) (O(1) trick for non-last); doubly O(1) | cost questions |
| Reverse | 3 pointers, O(n) time, O(1) space (iterative) | code completion |
| Middle | slow/fast, index `floor(n/2)` | one-pass middle |
| Floyd | meet inside cycle; reset one to head, step 1 each: meet at cycle start | cycle questions |
| Merge sorted lists | O(m+n), O(1) space | merge step |
| Node size | data + pointer (+ padding) | memory questions |
| n nodes, doubly | 2n pointers | memory comparison |

## GATE traps

- **Order of assignments:** `p->next = n` before `n->next = p->next` loses the rest of the list.
- **Head change:** a function that may change the head must return it or take `struct node **`; modifying a local copy of `head` has no effect on the caller (call by value).
- **Missing NULL checks:** `p->next->next` when `p->next` may be `NULL`; empty list; single-node list.
- **`while (h->next)` vs `while (h)`:** the first stops at the last node, the second after it.
- **Deleting the tail in a singly list** is O(n) even with a tail pointer.
- **Finding the middle of even-length lists:** `slow` lands on the second middle with the `f && f->next` condition.
- **Free before use:** reading `t->next` after `free(t)`.
- **Binary search on a linked list** is O(n) (no random access), not O(log n).
- **Cycle start vs meeting point** are different nodes.
- **Time to insert in a sorted list** is O(n) (find the position) even though relinking is O(1).

## Connections

- [Functions, recursion, structures](../05-c-programming/functions-recursion-structures.md) — self-referential structs, `malloc`, `->`.
- [Pointers, arrays, strings](../05-c-programming/pointers-arrays-strings.md) — pointer-to-pointer, dangling pointers, memory leaks.
- [Arrays, stacks, queues](arrays-stacks-queues.md) — array vs list trade-offs; stack and queue implementations.
- [Trees and BST](trees-and-bst.md) — a tree node is a list node with two (or more) links.
- [Graphs](graphs.md) — adjacency lists are arrays of linked lists.
- [Hashing](../08-algorithms/hashing.md) — separate chaining uses a list per bucket.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — merge sort on lists; why binary search needs arrays.
- [Memory management (OS)](../13-operating-systems/memory-management.md) — free lists and linked allocation of files/blocks.
- [File systems](../13-operating-systems/file-systems-and-disk-scheduling.md) — linked allocation of file blocks (FAT) is a linked list.
- [Combinatorics / recurrences](../01-discrete-mathematics/recurrences-and-generating-functions.md) — the Josephus problem on a circular list.

## Practice

**Q1 (MCQ, easy).** What is the worst-case time to insert a node at the end of a singly linked list of n nodes with only a `head` pointer?  (A) O(1)  (B) O(log n)  (C) O(n)  (D) O(n log n)

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** The whole list must be walked to reach the last node. (With a tail pointer it would be O(1).)

</details>

**Q2 (NAT).** A singly linked list has 9 nodes. The `mid` function above (`while (f && f->next)`) returns the node at which 1-based position?

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** `slow` ends at 0-based index `floor(9/2) = 4`, which is the 5th node. (`fast` path: indices 0, 2, 4, 6, 8, then `f->next` is NULL: 4 slow steps.)

</details>

**Q3 (MCQ).** Complete the reversal: `while (curr) { next = curr->next; ____ ; prev = curr; curr = next; }`  (A) `curr->next = next;`  (B) `curr->next = prev;`  (C) `prev->next = curr;`  (D) `next->next = curr;`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Each node's link must be flipped to point to the already-reversed part (`prev`). Others keep the original direction or crash on `NULL`.

</details>

**Q4 (NAT).** List `5 → 8 → 12 → 15 → 20`, pointer `p` to node 12. Execute `t = p->next; p->data = t->data; p->next = t->next; free(t);`. What is the sum of the data values remaining in the list?

<details><summary>Answer</summary>

**Answer:** 48  
**Solution:** The list becomes `5 → 8 → 15 → 20` (12 is overwritten with 15, the old 15-node is removed, giving 5, 8, 15, 20). Sum = 5 + 8 + 15 + 20 = 48.

</details>

**Q5 (NAT).** In a singly linked list the tail node's `next` points to the node at index 2 (0-based) of a list `0 → 1 → 2 → 3 → 4 → 5`. Using slow (1 step) and fast (2 steps) pointers from node 0, at which node do they first meet?

<details><summary>Answer</summary>

**Answer:** Node 4  
**Solution:** μ = 2 (nodes 0, 1), λ = 4 (nodes 2, 3, 4, 5). Positions after each step (slow, fast): (1, 2), (2, 4), (3, 2), (4, 4). They first meet after k = 4 steps (the smallest multiple of λ that is at least μ), at node 4.

</details>

**Q6 (MCQ).** What does `f(head)` print for `1 → 2 → 3 → 4 → 5`, with `void f(struct node *h) { if (h == NULL) return; printf("%d ", h->data); if (h->next) f(h->next->next); printf("%d ", h->data); }`?  (A) 1 3 5 5 3 1  (B) 1 2 3 4 5 5 4 3 2 1  (C) 1 3 5  (D) 5 3 1 1 3 5

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** `f(1)` prints 1, calls `f(3)`; `f(3)` prints 3, calls `f(5)`; `f(5)` prints 5, `h->next` is NULL so no call, prints 5. Unwinding prints 3 then 1.

</details>

**Q7 (MSQ).** Which are true? (A) Binary search on a sorted singly linked list takes O(log n). (B) Deleting a node given a pointer to it is O(1) in a doubly linked list. (C) Merging two sorted lists of lengths m, n takes O(m+n) time and O(1) extra space. (D) Reversing a singly linked list can be done in O(1) extra space. (E) Floyd's algorithm needs a visited-set.

<details><summary>Answer</summary>

**Answer:** B, C, D  
**Solution:** (A) reaching the middle costs O(n), so total is O(n). (E) Floyd uses two pointers only.

</details>

**Q8 (NAT, harder).** For the list `1 → 2 → 3 → 4 → 5 → 6`, `g(head)` is called, where
```c
struct node *g(struct node *h) {
    if (h == NULL || h->next == NULL) return h;
    struct node *t = h->next;
    h->next = g(t->next);
    t->next = h;
    return t;
}
```
What is the data value of the third node of the returned list?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** `g` swaps adjacent pairs recursively. `g(5)`: t = 6, `5->next = g(NULL) = NULL`, `6->next = 5`, returns 6. `g(3)`: t = 4, `3->next = 6`, `4->next = 3`, returns 4. `g(1)`: t = 2, `1->next = 4`, `2->next = 1`, returns 2. Result `2 → 1 → 4 → 3 → 6 → 5`; the third node is **4**.

</details>
