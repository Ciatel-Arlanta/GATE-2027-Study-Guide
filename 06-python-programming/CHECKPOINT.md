# Python programming: checkpoint

12 mixed questions across the whole section, easy to hard. Trace on paper first, then open the solution. Notes: [basics](python-basics.md) · [collections](python-collections.md) · [OOP, recursion, tracing](python-oop-recursion-tracing.md) · [cheat sheet](CHEATSHEET.md)

**Q1 (MCQ).** What is printed by `print(-9 // 4, -9 % 4, 9 // -4, 9 % -4)`?
(A) `-2 -1 -2 1`  (B) `-3 3 -3 -3`  (C) `-2 -1 -3 -3`  (D) `-3 3 -2 1`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `//` floors: $\lfloor -2.25\rfloor = -3$ for both `-9//4` and `9//-4`. `%` takes the divisor's sign: `-9 % 4 = -9 - (-3)(4) = 3`; `9 % -4 = 9 - (-3)(-4) = -3`. Output `-3 3 -3 -3`.

</details>

**Q2 (NAT).** How many times does the loop body run, and what is `n` at the end?

```python
n = 0
for i in range(17, 2, -5):
    n += i
```

Give the final value of `n`.

<details><summary>Answer</summary>

**Answer:** 36  
**Solution:** `range(17, 2, -5)` gives 17, 12, 7 (next is 2, not greater than 2, so excluded): 3 iterations, matching $\lceil (17-2)/5\rceil = 3$. $n = 17 + 12 + 7 = 36$.

</details>

**Q3 (MCQ).** What is printed?

```python
a = [1, 2, 3]
b = a
b += [4]
c = a[:]
c.append(5)
print(a, b, c)
```

(A) `[1,2,3] [1,2,3,4] [1,2,3,4,5]`  (B) `[1,2,3,4] [1,2,3,4] [1,2,3,4,5]`  (C) `[1,2,3,4] [1,2,3,4] [1,2,3,4]`  (D) `[1,2,3] [1,2,3] [1,2,3,5]`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `b = a` aliases; `b += [4]` extends the shared list in place, so `a` and `b` are both `[1,2,3,4]`. `a[:]` is a new list, so `c` becomes `[1,2,3,4,5]` without touching `a`.

</details>

**Q4 (MCQ).** What is printed?

```python
m = [[0] * 2] * 2
m[0][0] = 1
print(m)
```

(A) `[[1, 0], [0, 0]]`  (B) `[[1, 0], [1, 0]]`  (C) `[[1, 1], [0, 0]]`  (D) `[[0, 0], [0, 0]]`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `[[0]*2]*2` repeats a reference to the same inner list, so the two rows are one object; setting column 0 of "row 0" changes both.

</details>

**Q5 (MSQ).** Which of the following are true for `s = "abcdef"`?
(A) `s[::-2] == "fdb"`  (B) `s[-2:0:-1] == "edcb"`  (C) `s[2:2] == "c"`  (D) `s[10:] == ""`

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)  
**Solution:** (A) start at the last index 5 and step by -2: indices 5, 3, 1 = `f d b`. (B) start at index 4 (`e`), go down to index 1 (stop 0 excluded): `e d c b`. (C) `s[2:2]` is empty, not `"c"`. (D) out-of-range slices are clipped to empty, no error.

</details>

**Q6 (NAT).** For `s = {1, 2, 3, 4}` and `t = {3, 4, 5}`, what is `len(s ^ t) + len(s | t) + len(s - t)`?

<details><summary>Answer</summary>

**Answer:** 10  
**Solution:** `s ^ t = {1, 2, 5}` (3 elements), `s | t = {1, 2, 3, 4, 5}` (5), `s - t = {1, 2}` (2). Total $3 + 5 + 2 = 10$.

</details>

**Q7 (MCQ).** What is printed?

```python
def k(a, b=[]):
    b.append(a)
    return b
print(k(1), k(2), len(k(3)))
```

(A) `[1] [2] 1`  (B) `[1] [1, 2] 3`  (C) `[1, 2, 3] [1, 2, 3] 3`  (D) `[1, 2] [1, 2] 3`

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** The default list is created once. All three calls append to it. `print` evaluates all three arguments first: the first two arguments are the *same* list, which by then holds `[1, 2, 3]`; `len(k(3))` is 3.

</details>

**Q8 (NAT).** What is the sum of the values printed by `print(sum(f() for f in fs))`, where `fs = [lambda: i for i in range(3)]`?

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:** All three lambdas read the comprehension variable `i` when called. After the comprehension finishes, `i` is 2, so each returns 2: $2 + 2 + 2 = 6$ (not $0+1+2 = 3$). The `i=i` default trick would give 3.

</details>

**Q9 (MCQ).** Output of the following (note the print positions)?

```python
def h(n):
    if n == 0:
        return
    print(n, end=' ')
    h(n - 1)
    print(n, end=' ')
h(3)
```

(A) `3 2 1`  (B) `1 2 3`  (C) `3 2 1 1 2 3`  (D) `3 2 1 0 1 2 3`

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** On the way down each call prints `n` before recursing: 3, 2, 1. `h(0)` returns without printing. On the way back up each call prints `n` after the recursive call returns: 1, 2, 3.

</details>

**Q10 (NAT).** The function counts calls to itself (including the first call):

```python
cnt = 0
def g(n):
    global cnt
    cnt += 1
    if n <= 1:
        return n
    return g(n - 1) + g(n - 2)
g(6)
```

What is `cnt`? (Combines recursion with the `global` rule.)

<details><summary>Answer</summary>

**Answer:** 25  
**Solution:** Calls $C(n) = 1 + C(n-1) + C(n-2)$ with $C(0) = C(1) = 1$: $C(2)=3, C(3)=5, C(4)=9, C(5)=15, C(6)=25$. Closed form $2F_{n+1} - 1 = 2\cdot13 - 1 = 25$. (`g(6)` itself returns 8.)

</details>

**Q11 (MCQ).** What is printed?

```python
class A:
    n = 0
    def __init__(self):
        A.n += 1
        self.id = A.n

x, y, z = A(), A(), A()
print(x.id, y.id, z.id, x.n)
```

(A) `1 1 1 3`  (B) `1 2 3 3`  (C) `3 3 3 3`  (D) `1 2 3 0`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `A.n += 1` updates the class attribute (named through the class, so no shadowing). Each constructor copies the current value into the instance attribute `id`: 1, 2, 3. `x.n` has no instance attribute `n`, so it reads the class value 3.

</details>

**Q12 (MSQ).** Consider:

```python
class P:
    def h(self): return "P"
    def go(self): return self.h()
class Q(P):
    def h(self): return "Q" + super().h()
t = (1, [2, 3])
t[1].append(4)
d = {}
for ch in "abracadabra":
    d[ch] = d.get(ch, 0) + 1
```

Which are true?
(A) `Q().go()` is `"QP"`  (B) `t` is now `(1, [2, 3, 4])`  (C) `t[1] += [5]` succeeds with no error  (D) `max(d, key=d.get)` is `'a'`

<details><summary>Answer</summary>

**Answer:** (A), (B), (D)  
**Solution:** (A) `go` is inherited, calls `self.h()`, which resolves to `Q.h`; `super().h()` is `P.h`, giving `"QP"`. (B) A tuple is immutable but the list it references can mutate. (C) is false: `t[1] += [5]` extends the list in place and then tries to assign `t[1] = ...`, raising `TypeError` (the list does change before the error). (D) counts: a 5, b 2, r 2, c 1, d 1, so the maximum key is `'a'`.

</details>

**Scoring:** 10 or more correct (at least 80%) means move on to [Data structures](../07-data-structures/README.md). Fewer than 8 correct (below 60%): revisit the chapter for the questions you missed: Q1, Q2, Q7, Q8 and Q5 in [Python basics](python-basics.md) (Q5 also in [collections](python-collections.md)); Q3, Q4, Q6, Q12 in [Python collections](python-collections.md); Q9, Q10, Q11 in [OOP, recursion, tracing](python-oop-recursion-tracing.md).
