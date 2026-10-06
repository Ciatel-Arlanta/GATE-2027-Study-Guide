# C Programming: Checkpoint

12 mixed questions across the section, easy to hard. Assumed model: LP64 (`int` 4, pointer 8), little-endian. Attempt without looking at the chapters: [basics](c-basics-and-expressions.md) · [pointers](pointers-arrays-strings.md) · [functions/recursion/structs](functions-recursion-structures.md).

**Q1 (MCQ).** `printf("%d", sizeof(int) > -1);` prints  (A) 1  (B) 0  (C) -1  (D) undefined

<details><summary>Answer</summary>

**Answer:** (B) 0  
**Solution:** `sizeof` yields unsigned `size_t`; `-1` converts to a huge unsigned value, so 4 > 18446744073709551615 is false.

</details>

**Q2 (NAT).** `unsigned char c = 200; c += 100; printf("%d", c);`

<details><summary>Answer</summary>

**Answer:** 44  
**Solution:** 200 + 100 = 300 computed in `int`; stored back into `unsigned char`: 300 mod 256 = 44.

</details>

**Q3 (NAT).** `int a[] = {1,2,3,4,5}; int *p = a + 1; printf("%d", p[2] + *(a+4) - p[-1]);`

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:** `p[2]` = a[3] = 4; `*(a+4)` = 5; `p[-1]` = a[0] = 1. 4 + 5 - 1 = 8.

</details>

**Q4 (NAT).** After `int i = 0, j = 0; if (i++ && j++) { } `, what is `10*i + j`?

<details><summary>Answer</summary>

**Answer:** 10  
**Solution:** `i++` yields 0 (false) and sets i = 1; `&&` short-circuits, so `j++` never runs. i = 1, j = 0 → 10.

</details>

**Q5 (NAT).** `int f(int x) { static int s = 0; s += x; return s; }`. If `a = f(1); b = f(2); c = f(3);` are executed in order, what is `a + b + c`?

<details><summary>Answer</summary>

**Answer:** 10  
**Solution:** s becomes 1, 3, 6 so a = 1, b = 3, c = 6; sum 10.

</details>

**Q6 (NAT).** `int m[4][5]` stored row-major at address 1000, `int` = 4 bytes. Value of `(char*)(*(m+2)+3) - (char*)m`?

<details><summary>Answer</summary>

**Answer:** 52  
**Solution:** `*(m+2)+3` = `&m[2][3]`; offset (2·5 + 3)·4 = 52 bytes from the start (address 1052).

</details>

**Q7 (NAT).** `int g(int n) { if (n <= 0) return 0; return n + g(n - 2); }`. Return value of `g(9)` and total number of calls (including `g(9)`), as "value,calls".

<details><summary>Answer</summary>

**Answer:** 25,6  
**Solution:** Arguments: 9, 7, 5, 3, 1, -1. Value = 9 + 7 + 5 + 3 + 1 + g(-1) = 25 + 0. Six calls.

</details>

**Q8 (MCQ).** `struct { char a; int b; short c; }` has `sizeof` = (A) 7  (B) 8  (C) 12  (D) 16

<details><summary>Answer</summary>

**Answer:** (C) 12  
**Solution:** a@0, b@4, c@8 ends at 10; round up to a multiple of the struct alignment 4 gives 12.

</details>

**Q9 (MCQ).** `int x = 10; void f(){printf("%d ",x);} void g(){int x=20; f();} int main(){int x=30; g(); f();}`. Output under static vs dynamic scoping:  (A) 10 10 / 20 30  (B) 20 30 / 10 10  (C) 10 10 / 10 10  (D) 30 20 / 10 10

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** Static: `f` always sees the global x = 10, output `10 10`. Dynamic: from `g`, the nearest binding on the call stack is g's 20; from `main`, main's 30, output `20 30`.

</details>

**Q10 (NAT).** Global `int a = 2;` and `void Q(int x) { a = a + x; x = x + a; }` called as `Q(a)`. Let the final `a` be `a_v`, `a_r`, `a_vr`, `a_n` under value, reference, value-result and name. Compute `a_v + a_r + a_vr + a_n`.

<details><summary>Answer</summary>

**Answer:** 26  
**Solution:** Value: x = 2; a = 4; x changes locally; a = 4. Reference: x aliases a; a = a + a = 4; x = x + a means a = 4 + 4 = 8. Value-result: x = 2 copied in; a = 4; x = 2 + 4 = 6; copy back a = 6. Name: `x` is `a`: same as reference, 8. Sum = 4 + 8 + 6 + 8 = 26.

</details>

**Q11 (MSQ).** Which are correct? (A) `*(&a + 1) - a` equals the element count for `int a[N]` (B) `char *p = "hi"; p[0] = 'H';` is well-defined (C) `int *p, q;` declares two pointers (D) `a[i]` can be written `i[a]` (E) After `free(p)`, `p` becomes NULL

<details><summary>Answer</summary>

**Answer:** A, D  
**Solution:** (A) `&a + 1` jumps the whole array, difference in elements is N. (B) writes a literal: UB. (C) only `p` is a pointer. (E) `free` does not change `p`.

</details>

**Q12 (NAT, combined).** A singly linked list is built from `struct node { int d; struct node *next; };` with five `malloc(sizeof(struct node))` calls, then the head pointer is overwritten with NULL without freeing. How many bytes are leaked (ignore allocator overhead)? Also state `sizeof(struct node)`.

<details><summary>Answer</summary>

**Answer:** 80 bytes leaked; `sizeof` = 16  
**Solution:** d@0 (4 bytes), padding 4–7, next@8 (8 bytes) → 16. Five unreachable nodes → 80 bytes. See [linked lists](../07-data-structures/linked-lists.md).

</details>

**Scoring.** ≥ 80% (10+ correct) → move on to [Data structures](../07-data-structures/README.md). 60-80% → redo the missed traces on paper. < 60% → re-read: Q1, Q2, Q4, Q9 → [basics](c-basics-and-expressions.md); Q3, Q6, Q11 → [pointers](pointers-arrays-strings.md); Q5, Q7, Q8, Q10, Q12 → [functions, recursion, structures](functions-recursion-structures.md).
