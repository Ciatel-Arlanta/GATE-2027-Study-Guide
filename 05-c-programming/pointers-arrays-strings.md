# Pointers, Arrays and Strings

> **Paper:** CS · **Priority:** P0 · **Plan topics:** Pointers; Arrays in C; Memory and pointer reasoning (strings are covered here too)
> **Prerequisites:** [C basics and expressions](c-basics-and-expressions.md) · **Leads to:** [Functions, recursion, structures](functions-recursion-structures.md), [Linked lists](../07-data-structures/linked-lists.md)

Assumptions: 64-bit model (`int` = 4, `char` = 1, `double` = 8, pointer = 8 bytes), little-endian. Sample addresses (1000, 2000, ...) are illustrative.

## Quick glance

- A **pointer** stores an address. `&x` = address of x; `*p` = the object p points to. `sizeof(pointer)` is 8 whatever it points to.
- **Pointer arithmetic is scaled:** `p + k` moves `k * sizeof(*p)` bytes. `p - q` (same array) = number of **elements** between them.
- **`a[i] == *(a+i) == *(i+a) == i[a]`.** An array name in an expression decays to `&a[0]`, except under `sizeof`, `&`, and as a string-literal initialiser.
- `sizeof(a)` for an array = total bytes; for a pointer parameter (even written `int a[]`) = 8.
- `a` and `&a` have the same address but different types: `a+1` moves by one element, `&a + 1` moves by the **whole array**.
- 2D row-major: `&m[i][j] = base + (i*COLS + j) * sizeof(elem)`; `m[i][j] == *(*(m+i)+j)`.
- `char s[] = "hi"` is a modifiable array (3 bytes); `char *p = "hi"` points at a read-only literal (writing = UB).
- `strlen` excludes `'\0'`; `sizeof` includes it. `==` on strings compares addresses.
- `*p++` = `*(p++)`; `(*p)++` increments the pointee; `++*p` increments the pointee and yields the new value.
- Returning the address of a local variable = dangling pointer (UB).

## 1. Memory model

Memory is a long row of numbered bytes. A variable is a name for a few consecutive bytes; its address is the number of its first byte. A process sees:

```text
high addresses
+----------------------+
| stack  (grows down)  |  local variables, return addresses, parameters
|          |           |
|          v           |
|                      |
|          ^           |
|          |           |
| heap   (grows up)    |  malloc / calloc
+----------------------+
| BSS: uninitialised   |  globals and statics, zeroed
| data: initialised    |  globals and statics with values
+----------------------+
| text: code + string  |  read-only (string literals live here or in rodata)
+----------------------+
low addresses
```

## 2. Pointer basics

```c
int x = 10;
int *p = &x;      // p holds the address of x
*p = 20;          // writes through p: x becomes 20
int **pp = &p;    // pointer to pointer
**pp = 30;        // x becomes 30
```
```text
 x   : [ 30 ]  @1000
 p   : [1000]  @2000      *p   = x
 pp  : [2000]  @3000      *pp  = p,  **pp = x
```

`NULL` (address 0) means "points to nothing"; dereferencing it is UB (crash). An uninitialised pointer holds garbage; dereferencing is UB.

**Reading declarations** (start at the name, go right for `[]` and `()`, left for `*`; parentheses override):

| Declaration | Meaning |
|---|---|
| `int *p, q;` | `p` is `int*`; **`q` is a plain `int`** |
| `int *a[3]` | array of 3 pointers to int |
| `int (*a)[3]` | pointer to an array of 3 ints |
| `int *f(int)` | function returning `int*` |
| `int (*f)(int)` | pointer to a function taking int, returning int |
| `int **pp` | pointer to pointer to int |
| `int (*a[4])(int)` | array of 4 function pointers |

## 3. Pointer arithmetic

If `p` has type `T*`, then `p + k` is `p + k*sizeof(T)` in bytes.

| Expression (p = 1000) | `int*` | `char*` | `double*` |
|---|---|---|---|
| `p + 1` | 1004 | 1001 | 1008 |
| `p + 3` | 1012 | 1003 | 1024 |
| `(char*)p + 1` | 1001 | 1001 | 1001 |

Legal: pointer ± integer, pointer − pointer (same array; result type `ptrdiff_t`, counts elements), comparisons within one array. **Illegal/meaningless:** pointer + pointer, pointer × anything, subtracting pointers into different arrays (UB).

**Worked example 1 (`*p++`, `(*p)++`, `++*p`).**
```c
int a[] = {10, 20, 30, 40, 50};
int *p = a;
printf("%d ", *p++);    // (1)
printf("%d ", *++p);    // (2)
printf("%d ", (*p)++);  // (3)
printf("%d ", ++*p);    // (4)
printf("%d ", *p--);    // (5)
printf("%d", *p);       // (6)
```
Postfix `++`/`--` and unary `*` are different levels: postfix is higher, so `*p++` = `*(p++)`; prefix `++` and `*` are both level 2, right-to-left, so `*++p` = `*(++p)` and `++*p` = `++(*p)`.

| Step | Effect | Prints | p points to | array a |
|---|---|---|---|---|
| start | | | a[0] | 10 20 30 40 50 |
| (1) `*p++` | read a[0], then p moves | 10 | a[1] | unchanged |
| (2) `*++p` | p moves to a[2], read | 30 | a[2] | unchanged |
| (3) `(*p)++` | yields 30, a[2] becomes 31 | 30 | a[2] | 10 20 31 40 50 |
| (4) `++*p` | a[2] becomes 32, yields 32 | 32 | a[2] | 10 20 32 40 50 |
| (5) `*p--` | read a[2]=32, then p moves back | 32 | a[1] | unchanged |
| (6) `*p` | a[1] | 20 | a[1] | |

Output: **10 30 30 32 32 20**.

## 4. Arrays

An array is a block of `n` consecutive elements of the same type. `int a[5]` occupies 20 bytes; `a[i]` is the element at address `a + i*4`.

**Decay.** In nearly every expression the name `a` becomes a pointer to its first element (`&a[0]`). Exceptions: operand of `sizeof`, operand of `&`, and a string literal initialising a `char` array. Consequences:
- `a[i]` is *defined* as `*(a+i)`; since `+` commutes, `i[a]` also works.
- An array name is **not a modifiable lvalue**: `a = p;` and `a++` are errors.
- Arrays are passed to functions as pointers: `void f(int a[])` ≡ `void f(int *a)`, so `sizeof(a)` inside is 8, and callee changes are visible to the caller.

**`a` vs `&a` vs `&a[0]`** with `int a[5]` at 1000:

| Expression | Value | Type | `+1` gives |
|---|---|---|---|
| `a`, `&a[0]` | 1000 | `int*` | 1004 |
| `&a` | 1000 | `int (*)[5]` | **1020** |
| `sizeof(a)` | 20 | | |
| `sizeof(&a)`, `sizeof(a+0)` | 8 | | |

**Worked example 2.** `int n = *(&a + 1) - a;`
- `&a + 1` = 1020 of type `int(*)[5]`; `*(&a+1)` is the array at 1020, which decays to `int*` 1020.
- `1020 - 1000 = 20 bytes / 4 = 5` elements. So **n = 5** = array length (a known idiom). `(char*)(&a+1) - (char*)a` = 20.

### Two-dimensional arrays

`int m[3][4]` is an array of 3 elements, each an `int[4]`. Stored **row-major**: row 0 fully, then row 1, ...

```text
m[0][0] m[0][1] m[0][2] m[0][3] | m[1][0] ... m[1][3] | m[2][0] ... m[2][3]
  +0      +4      +8     +12        +16                    +32
```
Address of `m[i][j]` = `base + (i*COLS + j) * sizeof(elem)`. With 0-based indices and `COLS` the number of columns. (Lower bounds other than 0, column-major: see [arrays](../07-data-structures/arrays-stacks-queues.md).)

**Worked example 3 (address).** `int m[5][6]` at base 2000. Address of `m[3][4]`:
- offset = (3*6 + 4) * 4 = 22 * 4 = 88 → **2088**.

**Worked example 4 (pointer forms).** `int m[3][4] = {{1,2,3,4},{5,6,7,8},{9,10,11,12}};` at base 2000.

| Expression | Meaning | Value |
|---|---|---|
| `m + 1` | pointer to row 1 (moves 16 bytes) | 2016 |
| `*(m + 1)` / `m[1]` | row 1 as `int[4]`, decays to `&m[1][0]` | 2016 |
| `**m` | `m[0][0]` | 1 |
| `*(*(m+1)+2)` | `m[1][2]` | 7 |
| `*(m[2]+1)` | `m[2][1]` | 10 |
| `(*(m+2))[3]` | `m[2][3]` | 12 |
| `*(&m[0][0] + 5)` | flat 5th element = `m[1][1]` | 6 |
| `sizeof(m)`, `sizeof(m[0])`, `sizeof(m[0][0])` | | 48, 16, 4 |
| `*(m+2)+3` (address) | `&m[2][3]` = 2000 + 2*16 + 12 | 2044 |

Passing: `void f(int m[][4])` ≡ `void f(int (*m)[4])`; all dimensions except the first must be given.

**Array of pointers vs pointer to array.**

| | `int *a[3]` | `int (*p)[3]` |
|---|---|---|
| What it is | 3 pointers (24 bytes) | 1 pointer (8 bytes) |
| Typical use | ragged rows, array of strings | pointing at one row of a 2D array |
| `p+1` | n/a (array) | moves 12 bytes |

**Worked example 5 (pointer to pointer, a classic).**
```c
int a[] = {10, 20, 30, 40};
int *p[] = {a, a+1, a+2, a+3};   // p[k] points to a[k]
int **pp = p;
pp++;                              // pp -> p[1]
printf("%d %d %d\n", (int)(pp - p), (int)(*pp - a), **pp);
*pp++;                             // *(pp++): pp -> p[2]
printf("%d ", **pp);
++*pp;                             // ++(*pp): p[2] becomes a+3
printf("%d %d\n", **pp, (int)(pp - p));
```
- After `pp++`: `pp - p = 1`; `*pp = p[1] = a+1`, `*pp - a = 1`; `**pp = a[1] = 20`. Line 1: **1 1 20**.
- `*pp++`: postfix first: pp moves to `p+2` (the value read is discarded).
- `**pp` = `*p[2]` = `a[2]` = 30.
- `++*pp`: increments `p[2]` from `a+2` to `a+3`. Now `**pp` = `a[3]` = 40, `pp - p` = 2.
- Output: `1 1 20`, then `30 40 2`.

## 5. Strings

A C string is a `char` array ending with the null character `'\0'` (value 0). Functions find the end by looking for it.

```c
char s[] = "GATE2027";   // array of 9 bytes (8 chars + '\0'), modifiable copy
char *p  = "GATE2027";   // p points to a read-only literal; p[0] = 'g' is UB
```

| | `char s[] = "abc"` | `char *p = "abc"` |
|---|---|---|
| Storage | 4-byte array on stack/data | 8-byte pointer + literal in read-only memory |
| `sizeof` | 4 | 8 |
| Modify `s[0]` | OK | **UB** |
| Reassign | `s = ...` illegal | `p = "xyz"` OK |

`char s[10] = "abc"`: `sizeof(s)` = 10, `strlen(s)` = 3 (remaining 7 bytes zero). `char t[] = {'a','b'}`: **not** a string (no terminator).

**Worked example 6.**
```c
char s[] = "GATE2027";
printf("%zu %zu\n", sizeof(s), strlen(s));  // 9 8
s[4] = '\0';
printf("%zu %s\n", strlen(s), s);           // 4 GATE
printf("%s\n", s + 5);                      // "027": bytes after s[4]: '2','0','2','7' → starts at s[5]
```
`s[5]` is `'0'`, so `s+5` is "027". Also `"abc"[1]` is `'b'`; `printf("%s", "hello" + 2)` prints `llo`.

**Standard functions** (`<string.h>`): `strlen(s)` length without `'\0'`; `strcpy(d, s)` copies including `'\0'` (d must be big enough); `strcat(d, s)` appends; `strcmp(a, b)` returns `<0`, `0`, `>0` by lexicographic comparison of unsigned chars (not necessarily -1/+1); `strchr`, `strstr` search. **`a == b` compares addresses, not contents.**

**Worked example 7 (idioms).**
```c
int len(char *s)  { char *p = s; while (*p) p++; return p - s; }   // len("abc") = 3
void cpy(char *d, char *s) { while ((*d++ = *s++)) ; }             // strcpy idiom
```
Trace `cpy(d, "ab")`: iteration 1 copies 'a' (nonzero → continue); iteration 2 copies 'b'; iteration 3 copies `'\0'`, the assignment value is 0 → loop ends. The terminator is copied too.

**In-place reverse** (two indices): `for (i=0, j=len-1; i<j; i++, j--) { t=s[i]; s[i]=s[j]; s[j]=t; }`; "GATE" → "ETAG".

## 6. `const`, `void*`, function pointers

**`const` with pointers** — read the declaration right to left:

| Declaration | Can change `*p`? | Can change `p`? |
|---|---|---|
| `const int *p` (= `int const *p`) | no | yes |
| `int *const p` | yes | no |
| `const int *const p` | no | no |

**`void *`** is a generic pointer: any object pointer converts to/from it without a cast (in C); it cannot be dereferenced or used in arithmetic (standard C) until cast. `malloc` returns `void*`.

**Function pointers.** The name of a function decays to its address.
```c
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int (*op[2])(int, int) = { add, sub };
printf("%d %d", op[0](9, 4), (*op[1])(9, 4));   // 13 5
```
Used for callbacks such as the comparator of `qsort`.

## 7. Pointer bugs

| Bug | Example | Result |
|---|---|---|
| Dangling pointer (local) | `int *f(){ int x=5; return &x; }` | `x` dies on return: UB. Fix: `static int x`, caller-supplied buffer, or `malloc` |
| Dangling after `free` | `free(p); *p = 1;` | UB (use after free) |
| Double free | `free(p); free(p);` | UB |
| Memory leak | `p = malloc(8); p = malloc(8);` | first block unreachable, never freed |
| Uninitialised pointer | `int *p; *p = 3;` | writes to a random address |
| Out-of-bounds | `a[5]` for `int a[5]` | no check; UB |
| Buffer overflow | `strcpy` into too-small array | overwrites neighbours |
| Writing to a literal | `char *p="abc"; p[0]='x';` | UB |

`free(NULL)` is safe. Details of `malloc` are in [functions, recursion, structures](functions-recursion-structures.md).

**Endianness.** On a little-endian machine the least significant byte has the lowest address.
```c
int x = 0x12345678;
char *c = (char *)&x;
printf("%x %x", *c, *(c + 1));   // memory bytes: 78 56 34 12  →  prints "78 56"
```
(On big-endian: `12 34`.)

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Pointer step | `p+k` → `+k*sizeof(*p)` bytes | all pointer arithmetic |
| Pointer difference | `(p-q)` = elements | counting, length idiom |
| Subscript | `a[i] == *(a+i) == i[a]` | rewriting expressions |
| 2D row-major | `base + (i*C + j)*s` | address NAT questions |
| 2D pointer form | `m[i][j] == *(*(m+i)+j)` | trace questions |
| `&a + 1` | jumps whole array | length idiom |
| Strings | `sizeof` = `strlen`+1 for exact-fit array | size questions |
| `*p++`, `(*p)++`, `++*p` | pointer moves / pointee +1 (old) / pointee +1 (new) | trace |
| `const` | right-to-left | declarations |
| Pointer size | 8 (64-bit), independent of pointee | `sizeof` |

## GATE traps

- **`sizeof` of an array parameter** is the pointer size, not the array size.
- **`p + 1` scaling:** `int*` moves 4 bytes; casting to `char*` first moves 1.
- **`&a + 1` vs `a + 1`:** whole array vs one element.
- **`char *p = "..."; p[0] = ...`** is UB; `char s[] = "..."` is fine.
- **`strlen` vs `sizeof`** with explicit array sizes (`char s[10] = "abc"`: 3 vs 10).
- **`*p++` does not increment the value** (that is `(*p)++`).
- **Comparing strings with `==`** tests addresses.
- **`int *p, q;`** makes only `p` a pointer.
- **2D arrays:** forgetting to multiply by the number of **columns** (not rows).
- **Returning `&local`** — the value may even appear to work; the answer is still "undefined".
- **Pointer difference in bytes?** No: elements. `(char*)q - (char*)p` gives bytes.
- **Endianness** matters when an `int` is read through a `char*`; GATE normally states little-endian.

## Connections

- [C basics and expressions](c-basics-and-expressions.md) — precedence of `*`, `++`, `&`; promotion of `char`.
- [Functions, recursion, structures](functions-recursion-structures.md) — pointers as the way to simulate call by reference; `malloc`, struct pointers, linked nodes.
- [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) — row/column-major address formulas with non-zero lower bounds.
- [Linked lists](../07-data-structures/linked-lists.md) — node pointers, `->`, pointer-to-pointer insertion.
- [Memory management (OS)](../13-operating-systems/memory-management.md) — the addresses we use are virtual; stack/heap/text segments belong to a process image.
- [Runtime environments](../12-compiler-design/runtime-environments.md) — stack frames hold locals, so addresses of dead locals dangle.
- [Memory hierarchy and cache](../10-computer-organization/memory-hierarchy-and-cache.md) — row-major traversal is cache-friendly, column-wise traversal is not.
- [Instruction sets and addressing](../10-computer-organization/instruction-sets-and-addressing.md) — indexed addressing mode implements `a[i]`; endianness.

## Practice

**Q1 (NAT, easy).** `int a[10]; int *p = a;` Compute `sizeof(a) / sizeof(p)` on a 64-bit machine (`int` = 4).

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** `sizeof(a)` = 40, `sizeof(p)` = 8, and 40/8 = 5 (integer division, both `size_t`).

</details>

**Q2 (NAT).** `int m[5][6]` is stored row-major at base address 2000 with `int` = 4 bytes. What is the address of `m[3][4]`?

<details><summary>Answer</summary>

**Answer:** 2088  
**Solution:** 2000 + (3·6 + 4)·4 = 2000 + 22·4 = 2088.

</details>

**Q3 (MCQ).** Output of
```c
int a[] = {10, 20, 30, 40, 50}; int *p = a;
printf("%d ", *p++); printf("%d ", *++p); printf("%d ", (*p)++); printf("%d", ++*p);
```
(A) 10 20 30 31  (B) 10 30 30 32  (C) 10 30 31 32  (D) 20 30 30 32

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `*p++` prints 10, p → a[1]. `*++p` moves to a[2], prints 30. `(*p)++` prints 30 then a[2] = 31. `++*p` makes a[2] = 32 and prints 32.

</details>

**Q4 (MSQ).** Which are true for `char s[] = "abc"; char *p = "abc";`? (A) `sizeof(s)` = 4 (B) `sizeof(p)` = 4 (on 64-bit) (C) `s[0] = 'x';` is valid (D) `p[0] = 'x';` is valid (E) `s = p;` is valid

<details><summary>Answer</summary>

**Answer:** A, C  
**Solution:** `s` holds 'a','b','c','\0' = 4 bytes and is modifiable. `sizeof(p)` is 8 on 64-bit. `p[0]='x'` modifies a literal (UB). `s = p` assigns to an array name (error).

</details>

**Q5 (NAT).** `int a[7]; printf("%d", (int)((char *)(&a + 1) - (char *)a));` prints?

<details><summary>Answer</summary>

**Answer:** 28  
**Solution:** `&a + 1` moves past the entire array: 7 × 4 = 28 bytes; the cast to `char*` makes the difference count bytes.

</details>

**Q6 (NAT).** Using Worked example 5's declarations (`a = {10,20,30,40}`, `p[k] = a+k`, `pp = p`), after executing `pp++; *pp++; ++*pp;` what is the value of `**pp + (pp - p)`?

<details><summary>Answer</summary>

**Answer:** 42  
**Solution:** `pp++` → p+1; `*pp++` → p+2; `++*pp` makes `p[2] = a+3`. So `**pp` = a[3] = 40 and `pp - p` = 2; total 42.

</details>

**Q7 (MCQ).** On a little-endian machine, `int x = 0x12345678; char *c = (char*)&x; printf("%x", *(c+1));` prints  (A) 12  (B) 34  (C) 56  (D) 78

<details><summary>Answer</summary>

**Answer:** (C) 56  
**Solution:** Memory order from low address: 78 56 34 12. `*(c+1)` is the second byte, 0x56.

</details>

**Q8 (MCQ, harder).** What is printed?
```c
char s[] = "hello world";
char *p = s;
while (*p) { if (*p == ' ') *p = '\0'; p++; }
printf("%zu %zu %s", sizeof(s), strlen(s), s + 6);
```
(A) 12 5 world  (B) 12 11 world  (C) 11 5 world  (D) 8 5 world

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** `sizeof(s)` = 11 characters + `' '` = 12. The loop writes `' '` over the space at index 5, then `p++` moves to index 6, so the walk continues to the real end. `strlen(s)` stops at the new terminator: 5. `s + 6` points to "world". Output `12 5 world`.

</details>
