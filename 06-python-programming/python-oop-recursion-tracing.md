# Python OOP, recursion and tracing programs

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Classes/basic OOP; Recursion; Tracing Python programs
> **Prerequisites:** [Python basics](python-basics.md) · [Python collections](python-collections.md) · **Leads to:** [Data structures](../07-data-structures/linked-lists.md) · [Algorithms](../08-algorithms/divide-and-conquer.md)

## Quick glance

- A **class** is a blueprint; an **object** is an instance. `__init__(self, ...)` initialises a new instance; `self` is the instance, passed implicitly: `obj.m(a)` is `Class.m(obj, a)`.
- **Class attribute** (defined in the class body) is shared by all instances; **instance attribute** (`self.x = ...`) is per object. Reading `obj.x` looks in the instance first, then the class, then the base classes. **Assigning `obj.x = v` always creates/updates an *instance* attribute**, shadowing the class one.
- **Mutating** a shared class-level list (`obj.tags.append`) affects every instance; **rebinding** (`obj.tags = [...]`) only that instance.
- **Inheritance:** `class B(A)`. A method is found along the **MRO** (method resolution order, C3 linearisation); `super()` means "next class in the MRO", not "the parent".
- Dunder methods: `__str__` (used by `print`/`str`), `__repr__` (used inside containers and the REPL), `__eq__`, `__add__`, `__len__`. Defining `__eq__` without `__hash__` makes instances **unhashable**.
- **Recursion** = a function calls itself with a smaller input; needs a **base case**. Python's default depth limit is **1000** (`RecursionError`), and it does no tail-call optimisation.
- **Memoization** turns exponential recursion like naive Fibonacci into linear: `fib(10)` takes **177 calls** naively, **19** with a memo.
- **Tracing method:** write a table of variable values after each statement; for recursion, draw the call tree and note what is printed *before* and *after* the recursive call.

## 1. Classes and objects

**Intuition.** Bundle data (attributes) and the functions that act on it (methods) under one name. `self` is just the first parameter; Python fills it in with the object before the dot.

```python
class Acc:
    def __init__(self, bal=0):      # constructor; runs on Acc(...)
        self.bal = bal              # instance attribute
    def dep(self, x):
        self.bal += x
        return self                 # returning self allows chaining

ac = Acc()
ac.dep(5).dep(10)
print(ac.bal)       # 15
```

- `Acc.dep(ac, 5)` and `ac.dep(5)` are the same call. `Acc.dep()` with no object raises `TypeError` (missing `self`).
- Objects are **references**: `y = x; y.dep(3)` changes what `x` sees.

```python
x = Acc(); y = x; y.dep(3); print(x.bal)   # 3
```

- `isinstance(obj, C)` tests the class or any subclass; `type(obj) == C` tests exact class. Note `isinstance(True, int)` is `True` (`bool` subclasses `int`).
- `obj.__dict__` shows the instance attributes: for `class E: x = 1` with `__init__` setting `self.y = 2`, `E().__dict__` is `{'y': 2}` while `x` lives in `E.__dict__`.
- A name starting with double underscore (`self.__p`) is *name-mangled* to `_Class__p`: `hasattr(F(), '__p')` is `False`, `F()._F__p` is `1`. Python has no real private members.

### 1.1 Class vs instance attributes

**Worked example 1.**

```python
class A:
    n = 0                      # class attribute
    def __init__(self):
        A.n += 1               # modifies the class attribute
        self.n = self.n + 10   # reads class n, then creates an INSTANCE n

a = A(); b = A()
print(A.n, a.n, b.n)           # 2 11 12
```

Trace:

| Step | `A.n` | `a.n` | `b.n` | Explanation |
| --- | --- | --- | --- | --- |
| `A()` for `a`: `A.n += 1` | 1 | (no instance attr) | | class value becomes 1 |
| `self.n = self.n + 10` | 1 | 11 | | right side reads class `n = 1`; creates `a.n = 11` |
| `A()` for `b`: `A.n += 1` | 2 | 11 | | class `n = 2` |
| `self.n = self.n + 10` | 2 | 11 | 12 | reads class `n = 2` -> `b.n = 12` |

Result `2 11 12`. Once `a.n` exists, `A.n` changes no longer affect `a.n`.

**Worked example 2 (shared mutable class attribute).**

```python
class A:
    tags = []
a, b = A(), A()
a.tags.append(1)       # mutates the one shared list
print(b.tags, A.tags)  # [1] [1]
a.tags = [9]           # creates instance attribute on a only
print(a.tags, b.tags, A.tags)   # [9] [1] [1]
```

This is the class-level version of the mutable-default trap. The fix is to create the list in `__init__`.

### 1.2 Method kinds

| Kind | Declaration | First parameter | Typical use |
| --- | --- | --- | --- |
| instance method | `def m(self)` | the instance | normal behaviour |
| class method | `@classmethod def m(cls)` | the class | alternative constructors |
| static method | `@staticmethod def m()` | none | utility in the class namespace |

```python
class Cnt:
    calls = 0
    @staticmethod
    def inc(): Cnt.calls += 1
    @classmethod
    def cm(cls): return cls.__name__
Cnt.inc(); Cnt.inc()
print(Cnt.calls, Cnt.cm())     # 2 Cnt
```

## 2. Dunder (special) methods

| Method | Triggered by | Notes |
| --- | --- | --- |
| `__init__(self, ...)` | `C(...)` | returns nothing |
| `__str__` | `print(o)`, `str(o)` | falls back to `__repr__` if absent |
| `__repr__` | REPL, `repr(o)`, **elements printed inside a list** | |
| `__eq__(self, o)` | `o1 == o2` | default compares identity |
| `__add__`, `__sub__`, `__mul__` | `+`, `-`, `*` | |
| `__len__` | `len(o)`, also truthiness | |
| `__lt__`, `__le__`, ... | `<`, `<=` | used by `sorted` |
| `__getitem__` | `o[i]` | |
| `__hash__` | `hash(o)`, set/dict use | **set to `None` implicitly when you define `__eq__` alone** |

**Worked example 3.**

```python
class A:
    count = 0
    def __init__(self, x):
        self.x = x; A.count += 1
    def __str__(self):  return f"A({self.x})"
    def __repr__(self): return f"R{self.x}"
    def __eq__(self, o): return self.x == o.x
    def __add__(self, o): return A(self.x + o.x)
    def __len__(self): return self.x

a = A(1); b = A(1)
print(a == b, a is b)      # True False       (__eq__ compares x; `is` compares identity)
print(a)                   # A(1)             (print uses __str__)
print([a])                 # [R1]             (containers display elements with __repr__)
print(str(a), repr(a))     # A(1) R1
print(len(A(3)))           # 3
print((a + b).x)           # 2                (calls __add__, builds A(2))
```

`print([a])` shows `[R1]`, not `[A(1)]`: this is a favourite GATE trap.

**`__eq__` without `__hash__`.** After defining only `__eq__`, `{obj}` or `d[obj]` raises `TypeError: unhashable type`. `obj1 != obj2` automatically becomes `not (obj1 == obj2)` in Python 3.

## 3. Inheritance and method resolution

A subclass inherits attributes and methods; it may **override** them. Lookup follows the class's **MRO**.

**Worked example 4 (override and `super`).**

```python
class A:
    def __init__(self, x):
        self.x = x
    def show(self): return "A" + str(self.x)

class B(A):
    def __init__(self, x, y):
        super().__init__(x)          # run A's constructor
        self.y = y
    def show(self): return "B" + super().show()

bb = B(2, 3)
print(bb.show())                     # BA2
print(isinstance(bb, A), issubclass(B, A))   # True True
```

`bb.show()` finds `B.show`, returns `"B" + A.show(bb)` = `"B" + "A2"`.

**Dynamic dispatch through `self`.**

```python
class P:
    def f(self): return "P"
    def g(self): return self.f()     # self may be a subclass instance
class Q(P):
    def f(self): return "Q"
print(Q().g())                       # Q
```

`g` is inherited from `P`, but `self.f()` is looked up on the *actual object's class* `Q`, so `Q.f` runs.

**Constructor calling an overridable method.**

```python
class Base:
    def __init__(self): print("B-init"); self.setup()
    def setup(self): print("B-setup")
class Der(Base):
    def __init__(self): print("D-init"); super().__init__()
    def setup(self): print("D-setup")
Der()
# D-init
# B-init
# D-setup        <- Base.__init__ calls self.setup(), which resolves to Der.setup
```

### 3.1 Multiple inheritance and MRO

```python
class X:
    def who(self): return "X"
class Y(X):
    def who(self): return "Y" + super().who()
class Z(X):
    def who(self): return "Z" + super().who()
class W(Y, Z):
    def who(self): return "W" + super().who()

print(W().who())                              # WYZX
print([c.__name__ for c in W.__mro__])        # ['W', 'Y', 'Z', 'X', 'object']
```

```text
      X            MRO of W (C3): W, Y, Z, X, object
     / \           Rule: a class comes before its parents; parents keep the
    Y   Z          left-to-right order given in the class statement.
     \ /
      W
```

`super()` inside `Y` goes to the **next in W's MRO**, which is `Z` (not `X`), hence `WYZX` and each `who` runs exactly once.

## 4. Recursion

**Intuition.** Solve a problem by assuming you can already solve a smaller copy of it. Two parts: a **base case** that stops the recursion, and a **recursive case** that reduces the input. Each call gets its own frame (locals), stacked on the **call stack**. See [functions and recursion in C](../05-c-programming/functions-recursion-structures.md).

**Worked example 5 (what prints, and when).**

```python
def g(n):
    if n <= 0: return 0
    print(n, end=' ')       # before the recursive call
    g(n - 1)
    print(n, end=' ')       # after the recursive call
g(3)                        # 3 2 1 1 2 3
```

```text
g(3): print 3 -> g(2): print 2 -> g(1): print 1 -> g(0): return
                                  <- print 1 <- print 2 <- print 3
```

Statements before the recursive call run on the way **down**; statements after run on the way **up**, in reverse order.

**Worked example 6 (two recursive calls: `pr`).**

```python
def pr(n):
    if n > 0:
        pr(n - 1)
        print(n, end=' ')
        pr(n - 2)
pr(3)      # 1 2 3 1
```

Call tree (in-order: left call, print, right call):

```text
pr(3)
├─ pr(2)
│   ├─ pr(1)
│   │   ├─ pr(0)    (nothing)
│   │   ├─ print 1
│   │   └─ pr(-1)   (nothing)
│   ├─ print 2
│   └─ pr(0)        (nothing)
├─ print 3
└─ pr(1)
    ├─ pr(0)
    ├─ print 1
    └─ pr(-1)
```

Printed in order: 1, 2, 3, 1.

### 4.1 Standard recursive functions

```python
def fact(n): return 1 if n <= 1 else n * fact(n - 1)      # fact(5) = 120

def gcd(a, b): return a if b == 0 else gcd(b, a % b)      # gcd(48, 18) = 6

def pw(a, n):                                             # fast power, O(log n) calls
    if n == 0: return 1
    h = pw(a, n // 2)
    return h * h if n % 2 == 0 else h * h * a
```

`gcd(48,18) -> gcd(18,12) -> gcd(12,6) -> gcd(6,0) -> 6`. `gcd(18,48)` swaps on the first step (`18 % 48 = 18`), giving the same 6. `pw(3, 13)`: $n = 13 \to 6 \to 3 \to 1 \to 0$, result $3^{13} = 1594323$.

**Subsets and permutations.**

```python
def sub(s):
    if not s: return [""]
    r = sub(s[1:])
    return r + [s[0] + x for x in r]
print(sub("abc"))   # ['', 'c', 'b', 'bc', 'a', 'ac', 'ab', 'abc']
```

Build-up: `sub("") = ['']`; `sub("c") = ['', 'c']`; `sub("bc") = ['', 'c', 'b', 'bc']`; `sub("abc")` = that list followed by `a` prepended to each. $2^n$ outputs.

**Tower of Hanoi.** `hanoi(n) = 2*hanoi(n-1) + 1`, so $T(n) = 2^n - 1$; `hanoi(4) = 15`.

**Ackermann.** `ack(2, 3) = 9` (grows extremely fast; used to show recursion that is not primitive).

**Digit sum to a single digit.** `mystery(98765)`: digit sum $9+8+7+6+5 = 35 \to 3+5 = 8$. Result 8.

**Flatten a nested list.**

```python
def flat(L):
    r = []
    for x in L:
        r += flat(x) if isinstance(x, list) else [x]
    return r
print(flat([1, [2, [3, 4]], 5]))     # [1, 2, 3, 4, 5]
```

### 4.2 Counting calls and the exponential blow-up

**Worked example 7.** For `fibc(n) = n if n < 2 else fibc(n-1) + fibc(n-2)`, let $C(n)$ be the number of calls (including the first):

$$C(0) = C(1) = 1,\qquad C(n) = 1 + C(n-1) + C(n-2).$$

| n | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| $C(n)$ | 1 | 1 | 3 | 5 | 9 | 15 | 25 | 41 | 67 | 109 | **177** |

Closed form: $C(n) = 2F_{n+1} - 1$ with $F_1 = F_2 = 1$; check $C(10) = 2\cdot 89 - 1 = 177$. The same recurrence with a different "+1" gives `f(6) = 25` for `f(n) = f(n-1) + f(n-2) + 1`, `f(0) = f(1) = 1` (a question variant): that is the table value $C(6) = 25$.

### 4.3 Memoization

Store results of subproblems in a dict and reuse them. Each `n` is computed once, so calls drop from exponential to $O(n)$.

```python
memo = {}
def fibm(n):
    if n in memo: return memo[n]
    memo[n] = n if n < 2 else fibm(n - 1) + fibm(n - 2)
    return memo[n]
print(fibm(10))      # 55, using 19 calls and 11 memo entries
```

Count of calls for `n = 10`: each of the 11 values 0..10 is computed once, and each value $k \ge 2$ makes two child calls (to $k-1$ and $k-2$). Calls = 1 (top) + 2 per computed node with $k \ge 2$ minus nothing... verify: nodes computed $k = 2..10$ make $2 \times 9 = 18$ calls; plus the top call = 19. Cached lookups still count as calls.

Library version: `@functools.lru_cache(None)` decorates the function with a memo; `ff(50) = 12586269025`.

This is the top-down form of [dynamic programming](../08-algorithms/dynamic-programming.md).

### 4.4 Recursion depth

Python limits the stack to about **1000 frames** (`sys.getrecursionlimit()`), after which `RecursionError` is raised. A function with no reachable base case, e.g. `def inf(n): return inf(n+1)`, fails this way. Python does **not** optimise tail calls, so a tail-recursive loop of 10,000 steps also fails: rewrite it iteratively. In C an infinite recursion is stack overflow (segmentation fault).

### 4.5 Recursion and mutable default arguments

```python
def f2(x, acc=[]):
    acc.append(x)
    return len(acc) if x == 0 else f2(x - 1, acc)
print(f2(3), f2(2))      # 4 7
```

`f2(3)` appends 3, 2, 1, 0 to the shared default list (length 4). `f2(2)` reuses that list and appends 2, 1, 0 for length 7.

## 5. Tracing programs systematically

A repeatable method for output-prediction questions:

1. **Types and mutability first.** For each variable, note whether it labels a mutable object, and which other names alias it.
2. **Make a trace table.** One row per statement execution (or per loop iteration); columns for each variable that changes.
3. **For function calls,** note whether the argument is mutated (visible) or rebound (invisible), and which scope each name belongs to.
4. **For recursion,** draw the call tree; record the return value of each node bottom-up and the print order (before/after the call).
5. **Watch evaluation order:** arguments left to right; the right side of an assignment before binding; `print(f(), g())` runs both calls before printing anything.
6. **Edge conditions:** empty slices, off-by-one in `range`, `//` and `%` with negatives, integer vs float division, `None` returns.

**Worked example 8 (full trace).**

```python
def f(a, b=[]):
    b.append(a)
    a = a * 2
    return b + [a]

x = 3
r1 = f(x)
r2 = f(x + 1)
x = [x]
print(r1, r2, x)
```

| Statement | `a` (local) | default list `b` | returns |
| --- | --- | --- | --- |
| `f(3)`: `b.append(3)` | 3 | `[3]` | |
| `a = 6` | 6 | `[3]` | `b + [6]` = new list `[3, 6]` |
| `f(4)`: `b.append(4)` | 4 | `[3, 4]` | |
| `a = 8` | 8 | `[3, 4]` | `[3, 4, 8]` |

`r1` was built with `+`, so it is an independent new list `[3, 6]`, unaffected by the later append. Output: `[3, 6] [3, 4, 8] [3]`.

**Worked example 9 (binary search, recursive).**

```python
def bs(L, t, lo, hi):
    if lo > hi: return -1
    m = (lo + hi) // 2
    return m if L[m] == t else bs(L, t, m + 1, hi) if L[m] < t else bs(L, t, lo, m - 1)
print(bs([1, 3, 5, 7, 9, 11], 7, 0, 5))     # 3
```

`lo=0, hi=5`: `m = 2`, `L[2] = 5 < 7` -> search `(3, 5)`; `m = 4`, `L[4] = 9 > 7` -> search `(3, 3)`; `m = 3`, `L[3] = 7` found, return 3. See [searching](../08-algorithms/searching-and-sorting.md).

**Worked example 10 (linked list with a class).**

```python
class Node:
    def __init__(self, v, n=None):
        self.v = v; self.next = n

h = Node(1, Node(2, Node(3)))
c = h; t = 0
while c:
    t += c.v
    c = c.next
print(t)                # 6

def rev(n):
    prev = None
    while n:
        nxt = n.next; n.next = prev; prev = n; n = nxt
    return prev
h = rev(h)
print(h.v, h.next.v, h.next.next.v)    # 3 2 1
```

The in-place reversal rewires each `next` pointer to the previous node, exactly as in C. See [linked lists](../07-data-structures/linked-lists.md).

**Worked example 11 (stack class).**

```python
class Stack:
    def __init__(self): self.s = []
    def push(self, v): self.s.append(v)
    def pop(self): return self.s.pop()
    def empty(self): return not self.s

st = Stack()
for i in range(3): st.push(i)
print(st.pop(), st.pop(), st.empty())     # 2 1 False
```

## Formulas and facts to memorise

| Item | Fact | When to use |
| --- | --- | --- |
| Attribute lookup | instance, then class, then bases (MRO) | class vs instance questions |
| Assignment `obj.x = v` | always creates/updates instance attribute | shadowing |
| `print(o)` vs `print([o])` | `__str__` vs `__repr__` | dunder output |
| `__eq__` alone | removes hashability | set/dict of objects |
| `super()` | next class in MRO | diamond inheritance |
| Recursion limit | default 1000 frames | `RecursionError` |
| Naive Fibonacci | calls $C(n) = 2F_{n+1} - 1$; $C(10) = 177$ | counting calls |
| Memoised Fibonacci | $2n - 1$ calls for $n \ge 1$ ($n=10$: 19) | memo questions |
| Hanoi | $2^n - 1$ moves | recursion counts |
| Subsets | $2^n$ | `sub(s)` |
| Permutations | $n!$ | `perm(s)` |
| Before/after call | before = pre-order, after = post-order | print-order questions |

## GATE traps

- **`self.x = self.x + 1` where `x` is a class attribute** reads the class value but creates an instance attribute; later class changes no longer reach that instance.
- **Shared mutable class attribute or default argument:** `append` is visible to all; `obj.attr = new` is not.
- **`print([obj])` uses `__repr__`;** `print(obj)` uses `__str__`.
- **Overriding `__eq__` makes the class unhashable** unless `__hash__` is also defined.
- **`super()` follows the MRO, not "the parent".** In the diamond `W(Y, Z)` the order is `W, Y, Z, X`.
- **A constructor that calls an overridable method runs the subclass version** even inside the base `__init__`.
- **Method called on class without object** (`D.m()`) raises `TypeError`; `D.m(d)` works.
- **Print placement in recursion:** before the call = descending order, after = ascending. Draw the tree.
- **Naive recursion call counts include the base-case calls;** count carefully and say whether the first call counts.
- **Memoised calls still count as calls** even if they return immediately from the cache.
- **`print(f(), g())` evaluates both before printing,** so a shared mutable result shows its final state twice.
- **No tail-call optimisation:** deep linear recursion (about 1000) raises `RecursionError`.
- **Recursive generation of lists: `r1 = acc + [x]` creates a new list; `acc.append(x)` mutates.** Know which one is used.

## Connections

- [Python basics](python-basics.md) — scope, defaults and closures are the machinery behind methods and memoization.
- [Python collections](python-collections.md) — object attributes holding lists/dicts follow the same aliasing rules.
- [Linked lists](../07-data-structures/linked-lists.md) and [trees and BST](../07-data-structures/trees-and-bst.md) — `Node` classes with `next/left/right` and recursive traversals are tested in Python form.
- [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) — the `Stack` class here; the interpreter's call stack is the same LIFO.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — recurrences $T(n) = T(n-1) + T(n-2) + 1$ and $T(n) = 2T(n/2) + n$ come from recursive functions.
- [Divide and conquer](../08-algorithms/divide-and-conquer.md) — recursive binary search, merge sort, fast power.
- [Dynamic programming](../08-algorithms/dynamic-programming.md) — memoization = top-down DP.
- [Functions, recursion, structures in C](../05-c-programming/functions-recursion-structures.md) — C has structs without methods, no GC, fixed-size stack frames; Python methods/objects are heap references.
- [Recurrences](../01-discrete-mathematics/recurrences-and-generating-functions.md) — solving $C(n) = C(n-1) + C(n-2) + 1$.
- [ML foundations](../16-machine-learning/ml-foundations.md) — scikit-learn style `fit/predict` classes are the same OOP pattern.

## Practice

**Q1 (MCQ).** What is printed?

```python
class A:
    n = 0
    def __init__(self):
        A.n += 1
        self.n = self.n + 10
a = A(); b = A()
print(A.n, a.n, b.n)
```

(A) `2 11 12`  (B) `2 11 11`  (C) `2 12 12`  (D) `0 10 10`

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** First `A()`: class `n` becomes 1, `a.n = 1 + 10 = 11`. Second: class `n` becomes 2, `b.n = 2 + 10 = 12`. `A.n = 2`.

</details>

**Q2 (NAT).** How many calls (including the first) does `f(6)` make?

```python
def f(n):
    if n <= 1: return 1
    return f(n - 1) + f(n - 2) + 1
```

Give the number of calls, then the return value of `f(6)`. Report the number of calls.

<details><summary>Answer</summary>

**Answer:** 25 calls (return value is also 25)  
**Solution:** Let $C(n)$ = calls. $C(0) = C(1) = 1$, $C(n) = 1 + C(n-1) + C(n-2)$: $C(2) = 3, C(3) = 5, C(4) = 9, C(5) = 15, C(6) = 25$. The return value $R(n)$ satisfies $R(n) = R(n-1) + R(n-2) + 1$, $R(0) = R(1) = 1$, the same recurrence as $C$, so $R(6) = 25$ as well.

</details>

**Q3 (MCQ).** What is printed?

```python
class X:
    def who(self): return "X"
class Y(X):
    def who(self): return "Y" + super().who()
class Z(X):
    def who(self): return "Z" + super().who()
class W(Y, Z):
    def who(self): return "W" + super().who()
print(W().who())
```

(A) `WYX`  (B) `WYZX`  (C) `WZYX`  (D) `WXYZ`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** MRO is `W, Y, Z, X, object`. `W.who` calls the next class, `Y`; `Y.who`'s `super()` is the next class after `Y` *in W's MRO*, which is `Z`; `Z`'s `super()` is `X`. Result `W + Y + Z + X`.

</details>

**Q4 (MSQ).** Given the class below, which expressions are true?

```python
class P:
    def __init__(self, v): self.v = v
    def __eq__(self, o): return self.v == o.v
p, q = P(1), P(1)
```

(A) `p == q`  (B) `p is q`  (C) `p != q`  (D) `len({1, 2}) == 2` (that is, creating `{p, q}` raises `TypeError`)

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** `p == q` uses `__eq__` -> `True`. `p is q` is `False` (different objects). `p != q` defaults to `not (p == q)` -> `False`. Statement (D) as worded, `len({1,2}) == 2`, is a plain set of ints and is also `True`, but the intended point is that `{p, q}` raises `TypeError` because defining `__eq__` removed `__hash__`. As written the true expressions are (A) and (D); the intended-lesson answer is (A) alone.

</details>

**Q5 (NAT).** What is the value of `mystery(98765)`?

```python
def mystery(n):
    if n < 10: return n
    return mystery(sum(map(int, str(n))))
```

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:** `98765 -> 9+8+7+6+5 = 35 -> 3+5 = 8 -> 8 < 10` returns 8 (the digital root; equals $98765 \bmod 9$ with 0 mapped to 9, and $98765 = 9\cdot 10973 + 8$).

</details>

**Q6 (MCQ).** What is printed?

```python
def pr(n):
    if n > 0:
        pr(n - 1)
        print(n, end=' ')
        pr(n - 2)
pr(4)
```

(A) `1 2 3 4`  (B) `1 2 3 1 4 1 2`  (C) `1 2 3 1 4 2`  (D) `4 3 2 1 1 2`

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** `pr(4)` = `pr(3)`, print 4, `pr(2)`. `pr(3)` prints `1 2 3 1` (worked example 6). `pr(2)` = `pr(1)`, print 2, `pr(0)` = `1`, `2`. Total: `1 2 3 1`, `4`, `1 2`, i.e. `1 2 3 1 4 1 2`. Recheck against options: that string equals option (B). **Correct answer: (B).**

</details>

**Q7 (NAT).** How many calls does the memoised function make for `fibm(10)` when `memo` starts empty (count every call, including ones answered from the cache)?

```python
memo = {}
def fibm(n):
    if n in memo: return memo[n]
    memo[n] = n if n < 2 else fibm(n - 1) + fibm(n - 2)
    return memo[n]
```

<details><summary>Answer</summary>

**Answer:** 19  
**Solution:** Each value 0..10 is computed once (11 computations). The values $k = 2, \dots, 10$ (9 of them) each make two recursive calls: 18 calls. Add the initial call: 19. (Of these, 11 do real work and 8 are cache hits.)

</details>

**Q8 (MCQ).** What is printed?

```python
class Base:
    def __init__(self):
        print("B", end=' ')
        self.setup()
    def setup(self): print("Bs", end=' ')
class Der(Base):
    def __init__(self):
        print("D", end=' ')
        super().__init__()
    def setup(self): print("Ds", end=' ')
Der()
```

(A) `D B Bs`  (B) `D B Ds`  (C) `B D Ds`  (D) `D Ds B`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `Der()` runs `Der.__init__`: prints `D`, then `super().__init__()` runs `Base.__init__`: prints `B`, then `self.setup()` dispatches on the real object (a `Der`), so `Der.setup` prints `Ds`.

</details>
