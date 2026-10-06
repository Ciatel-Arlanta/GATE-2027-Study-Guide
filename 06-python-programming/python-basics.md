# Python basics: syntax, variables, expressions, iteration, functions, scope

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Python syntax; Variables and expressions; Functions; Iteration and loops
> **Prerequisites:** [C basics and expressions](../05-c-programming/c-basics-and-expressions.md) (helpful for contrast) · **Leads to:** [Python collections](python-collections.md) · [OOP, recursion, tracing](python-oop-recursion-tracing.md)

## Quick glance

- Python is **dynamically typed**: a variable is a *name* bound to an object; the type lives on the object, not the name. `x = 5; x = "a"` is legal.
- **Indentation is syntax.** A block is the set of lines indented equally after a colon. No braces, no semicolons needed.
- `/` is always **true division** (returns float). `//` is **floor** division (rounds toward $-\infty$). `%` has the **sign of the divisor**. Identity: `a == (a // b) * b + a % b`.
- Precedence (high to low): `**` (right-assoc, binds tighter than unary minus on its left), unary `- + ~`, `* / // %`, `+ -`, shifts, `&`, `^`, `|`, comparisons (chain!), `not`, `and`, `or`.
- `and` / `or` **return an operand**, not a bool, and short-circuit: `0 or "x"` is `"x"`; `3 and 5` is `5`.
- `range(a, b, s)` is half-open: includes `a`, excludes `b`. `range(5, 0)` is empty.
- Arguments are passed by **object reference** ("call by assignment"): mutating a mutable argument is visible to the caller; rebinding the parameter is not.
- Scope is **LEGB** (Local, Enclosing, Global, Built-in). Assigning to a name anywhere in a function makes it local for the *whole* function.
- **#1 trap:** a mutable default argument (`def f(x, L=[])`) is created once, at definition time, and shared across calls.

## 1. Syntax, names and objects

**Intuition.** In C a variable is a labelled box holding a value. In Python a variable is a *sticky label* attached to an object. Assignment never copies the object; it just attaches another label.

```text
x = [1, 2]      x ──────┐
y = x           y ──────┴──►  [1, 2]      (one object, two labels)
y.append(3)     both see [1, 2, 3]
y = [9]         y now labels a new object; x still labels [1, 2, 3]
```

- A statement ends at the newline. A colon `:` introduces a block (`if`, `for`, `while`, `def`, `class`, `try`, `with`).
- Comments start with `#`. Docstrings are string literals as the first statement of a function.
- Names are case-sensitive, may contain letters, digits, underscore, and cannot start with a digit. Keywords (`if`, `for`, `None`, `True`, `lambda`, ...) cannot be names.
- **Everything is an object**: ints, functions, classes. `type(x)` gives the type; `id(x)` the identity.

**Core built-in types**

| Type | Example | Mutable? | Notes |
| --- | --- | --- | --- |
| `int` | `42`, `-7`, `10**30` | no | arbitrary precision, no overflow (unlike C) |
| `float` | `3.14`, `1e-3` | no | IEEE 754 double; `0.1 + 0.2 != 0.3` |
| `bool` | `True`, `False` | no | subclass of `int`: `True + True == 2` |
| `str` | `"abc"`, `'abc'` | no | immutable sequence of characters |
| `NoneType` | `None` | no | the "nothing" value; what a function with no `return` gives |
| `list` | `[1, 2]` | **yes** | see [collections](python-collections.md) |
| `tuple` | `(1, 2)` | no | |
| `dict` | `{'a': 1}` | **yes** | |
| `set` | `{1, 2}` | **yes** | |

**Truthiness.** In a condition, these are *false*: `False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`, `range(0)`. Everything else is true. **`"0"` and `[0]` are true** (non-empty).

### 1.1 Identity vs equality

`==` asks "do the two objects have equal value?" `is` asks "are they the *same object*?"

```python
x = [1, 2]; y = [1, 2]
print(x == y, x is y)     # True False
z = x
print(z is x)             # True
```

Small ints (-5..256) and some short strings are cached by CPython, so `a = 256; b = 256; a is b` is `True` but this is an implementation detail. **GATE questions only use `is` for lists/objects or `is None`; never rely on integer or string identity.**

## 2. Arithmetic and the division family

**Intuition.** Floor division rounds *down* (toward $-\infty$), not toward zero as C does. `%` is defined so that the identity $a = (a // b)\cdot b + (a \% b)$ always holds, which forces the remainder to take the **sign of the divisor**.

**Worked example 1 (the sign table).** Verify with $a = 7, b = 2$ and the sign variants:

| Expression | `a // b` | Check: `(a//b)*b + a%b` | `a % b` |
| --- | --- | --- | --- |
| `7 // 2`, `7 % 2` | 3 | $3\cdot2 + 1 = 7$ | 1 |
| `-7 // 2`, `-7 % 2` | -4 (floor of -3.5) | $-4\cdot 2 + 1 = -7$ | 1 |
| `7 // -2`, `7 % -2` | -4 (floor of -3.5) | $-4\cdot(-2) + (-1) = 7$ | -1 |
| `-7 // -2`, `-7 % -2` | 3 (floor of 3.5) | $3\cdot(-2) + (-1) = -7$ | -1 |

```python
print(7//2, -7//2, 7//-2, -7//-2)    # 3 -4 -4 3
print(7%3, -7%3, 7%-3, -7%-3)        # 1 2 -2 -1
print(divmod(-7, 2))                 # (-4, 1)
print(7.5 // 2, -7.5 // 2, -7.5 % 2) # 3.0 -4.0 0.5
```

Why `-7 % 3 == 2`: $\lfloor -7/3 \rfloor = -3$ (since $-2.33$ floors to $-3$), so the remainder is $-7 - (-3)(3) = 2$. **Contrast with C**, where `-7 % 3 == -1` (truncation toward zero). See [C basics](../05-c-programming/c-basics-and-expressions.md).

**Other operators**

| Operator | Meaning | Example | Result |
| --- | --- | --- | --- |
| `/` | true division (always `float`) | `10 / 5` | `2.0` |
| `**` | power | `2 ** 10`, `2 ** -1` | `1024`, `0.5` |
| `abs`, `pow(a, n, m)` | absolute value, modular power | `pow(2, 10, 1000)` | `24` |
| `&`, `\|`, `^`, `~` | bitwise AND, OR, XOR, NOT | `5&3, 5\|3, 5^3, ~5` | `1, 7, 6, -6` |
| `<<`, `>>` | shifts (`>>` is arithmetic) | `5 << 2`, `-5 >> 1` | `20`, `-3` |
| `int(x)` | truncates toward zero | `int(-3.7)` | `-3` |
| `round(x)` | **banker's rounding** (ties to even) | `round(0.5), round(1.5), round(2.5)` | `0, 2, 2` |

`~x == -x - 1`. `-5 >> 1` is $\lfloor -5/2 \rfloor = -3$.

### 2.1 Precedence and associativity

From highest to lowest:

| Level | Operators | Associativity |
| --- | --- | --- |
| 1 | `**` | **right** to left |
| 2 | unary `+x -x ~x` | right |
| 3 | `* / // %` | left |
| 4 | `+ -` | left |
| 5 | `<< >>` | left |
| 6 | `&` then `^` then `\|` | left |
| 7 | `== != < > <= >= is in not in` (all **same level, chained**) | chained |
| 8 | `not` | |
| 9 | `and` | left |
| 10 | `or` | left |

**Worked example 2.** Evaluate `-2 ** 2`, `2 ** 3 ** 2`, `2 + 3 * 4 ** 2 // 5 % 7`.

1. `-2 ** 2`: `**` binds tighter than unary minus on its **left**: $-(2^2) = -4$. (Compare `(-2) ** 2 = 4`.)
2. `2 ** 3 ** 2`: right-assoc: $2^{(3^2)} = 2^9 = 512$ (not $8^2 = 64$).
3. `2 + 3 * 4 ** 2 // 5 % 7`:
   - `4 ** 2 = 16`
   - `3 * 16 = 48` (`*`, `//`, `%` are same level, left to right)
   - `48 // 5 = 9`
   - `9 % 7 = 2`
   - `2 + 2 = 4`.

`2 ** -1` is `0.5` (note: `**` allows a unary operator on its *right*).

### 2.2 Chained comparisons

`a < b < c` means `a < b and b < c` (with `b` evaluated once). It is **not** `(a < b) < c`.

```python
print(1 < 2 < 3)     # True
print(1 < 3 < 2)     # False
print(3 > 2 > 1)     # True
print(1 == 1 < 2)    # True   (1 == 1 and 1 < 2)
```

### 2.3 Boolean operators return operands

```text
x or y   -> x if x is truthy else y
x and y  -> x if x is falsy  else y
not x    -> always a bool
```

**Worked example 3.**

```python
print(0 or "x")         # x       (0 is falsy -> second operand)
print([] or [0])        # [0]
print(3 and 5)          # 5       (3 truthy -> second operand)
print(0 and 1)          # 0       (0 falsy -> returns it, never evaluates 1)
print(1 or 0/0)         # 1       (short-circuit: 0/0 is never evaluated)
print(not 1 == 2)       # True    (== binds tighter than not)
print(0 or None)        # None
```

### 2.4 Floating point

Floats are binary fractions. `0.1 + 0.2` is `0.30000000000000004`, so `0.1 + 0.2 == 0.3` is `False`. `round(2.675, 2)` gives `2.67` because 2.675 is stored slightly below. Use `abs(a - b) < eps` for comparisons. See [number representation](../09-digital-logic/number-representation-and-arithmetic.md).

### 2.5 Strings (needed everywhere)

Strings are **immutable** sequences; every method returns a *new* string.

| Operation | Result |
| --- | --- |
| `"ab" * 3` | `'ababab'` |
| `"abc"[::-1]` | `'cba'` |
| `"abcdef"[1:4]`, `[-3:]`, `[::2]`, `[4:1:-1]` | `'bcd'`, `'def'`, `'ace'`, `'edc'` |
| `"a,b,,c".split(",")` | `['a', 'b', '', 'c']` |
| `" ".join(["a","b"])` | `'a b'` |
| `"hello".replace("l","L",1)` | `'heLlo'` (count limits replacements) |
| `"abc".find("z")` | `-1` (`.index` would raise `ValueError`) |
| `"banana".count("an")` | `2` (non-overlapping) |
| `"10" < "9"` | `True` (lexicographic), but `10 < 9` is `False` |
| `"Z" < "a"` | `True` (ASCII 90 < 97) |
| `f"{3.14159:.2f}"`, `f"{5:03d}"` | `'3.14'`, `'005'` |

`"3" + 4` raises `TypeError`; Python does **no** implicit str/int conversion.

## 3. Iteration and loops

### 3.1 `for` and `range`

`for x in iterable:` takes items one by one. `range(start, stop, step)` generates integers from `start` (default 0) while `< stop` (if step > 0) or `> stop` (if step < 0).

| Call | Items | Length |
| --- | --- | --- |
| `range(5)` | 0 1 2 3 4 | 5 |
| `range(2, 20, 5)` | 2 7 12 17 | 4 |
| `range(10, 0, -3)` | 10 7 4 1 | 4 |
| `range(0, 10, 4)` | 0 4 8 | 3 |
| `range(5, 0)` | (empty: step is +1) | 0 |
| `range(-3)` | (empty) | 0 |

Length formula for positive step: $\max\!\left(0, \lceil (stop - start)/step \rceil\right)$.

**Worked example 4 (loop variable leaks).**

```python
for i in range(3):
    pass
print(i)            # 2   (loop variable survives after the loop)

i = 10
for i in range(3):
    i = i * 5       # rebinding i inside the body does NOT affect iteration
print(i)            # 10  (last iteration: i=2 -> 10)
```

Trace: iteration values are 0, 1, 2 regardless of what the body does to `i`. After the last iteration `i = 2 * 5 = 10`.

### 3.2 `while`, `break`, `continue`, loop `else`

- `break` leaves the innermost loop. `continue` jumps to the next iteration.
- **`else` on a loop runs only if the loop finished without `break`.**

**Worked example 5.**

```python
n = 0
while n < 5:
    n += 1
    if n == 2: continue      # skip rest of body when n == 2
    if n == 4: break         # exit loop
else:
    print("else")            # NOT executed: loop ended via break
print(n)
```

| Iteration | n after `n += 1` | action |
| --- | --- | --- |
| 1 | 1 | nothing |
| 2 | 2 | `continue` |
| 3 | 3 | nothing |
| 4 | 4 | `break` -> skip `else` |

Output: `4` only. With `for k in range(3): pass` followed by `else: print("for-else", k)` the else **does** run and prints `for-else 2`.

### 3.3 Useful iteration helpers

| Helper | Example | Result |
| --- | --- | --- |
| `enumerate(it, start=0)` | `list(enumerate("ab", 1))` | `[(1,'a'), (2,'b')]` |
| `zip(a, b, ...)` | `list(zip([1,2,3], "ab"))` | `[(1,'a'), (2,'b')]` (stops at shortest) |
| `reversed(seq)` | `list(reversed([1,2,3]))` | `[3,2,1]` |
| `sum`, `min`, `max`, `any`, `all` | `any([])`, `all([])` | `False`, `True` |
| `sorted(it, key=, reverse=)` | `sorted([3,-5,1], key=abs)` | `[1, 3, -5]` |

## 4. Functions

**Intuition.** A function is an object created by `def` at the moment `def` executes. Calling it creates a fresh local namespace. A function without `return` (or with bare `return`) returns `None`.

### 4.1 Parameter kinds

```python
def f(a, b=2, *args, c=5, **kw):
    return a, b, args, c, kw

print(f(1))                # (1, 2, (), 5, {})
print(f(1, 3, 4, 5))       # (1, 3, (4, 5), 5, {})
print(f(1, c=9, z=0))      # (1, 2, (), 9, {'z': 0})
print(f(b=1, a=0))         # (0, 1, (), 5, {})
```

| Kind | Syntax | Collects |
| --- | --- | --- |
| positional / default | `a`, `b=2` | matched by position or by name |
| `*args` | extra positional | a **tuple** |
| keyword-only | after `*args`: `c=5` | must be passed by name |
| `**kw` | extra keyword | a **dict** |

Defaults must come after non-defaults in the parameter list. Rules for the call: positional arguments first, then keyword arguments.

### 4.2 Pass by object reference

The parameter is a new label for the *same object* the caller passed.

- **Mutating** the object (`L.append`, `L += [..]`, `L[0] = ..`) is visible to the caller.
- **Rebinding** the parameter (`L = L + [2]`, `L = [100]`, `n += 1` for an int) only changes the local label.

**Worked example 6.**

```python
def modify(L, t, n, s):
    L.append(1)     # mutates caller's list
    L = L + [2]     # rebinds local L to a NEW list; caller unaffected
    t += (9,)       # tuple is immutable: += rebinds local t
    n += 1          # int immutable: rebinds
    s += "x"        # str immutable: rebinds

L = [0]; t = (0,); n = 0; s = "a"
modify(L, t, n, s)
print(L, t, n, s)    # [0, 1] (0,) 0 a
```

Step by step: after `append`, the caller's `L` is `[0, 1]`. `L + [2]` creates a third list bound only to the local name. The other three are immutable, so no change is visible.

A subtle one: `L += [7]` on a **list** is *in-place* (`list.__iadd__` extends), whereas `L = L + [7]` creates a new list.

```python
def mod2(L):
    L += [7]       # in-place extend: caller sees it
    L = [100]      # rebinding afterwards: invisible
L = [0]; mod2(L); print(L)    # [0, 7]
```

### 4.3 The mutable default argument trap

Default values are evaluated **once, when `def` runs**, and stored on the function. A mutable default is therefore shared by all calls that omit it.

**Worked example 7.**

```python
def g(x, L=[]):
    L.append(x)
    return L

a = g(1); b = g(2); c = g(3, []); d = g(4)
print(a, b, c, d)     # [1, 2, 4] [1, 2, 4] [3] [1, 2, 4]
```

Trace: the default list `D` starts `[]`. `g(1)` appends to `D` -> `[1]`, returns `D`. `g(2)` -> `D = [1,2]`, returns the *same* `D`. `g(3, [])` uses a fresh list -> `[3]`. `g(4)` -> `D = [1,2,4]`. `a`, `b`, `d` are all the same object `D`, so all print its final contents.

Fix: `def g(x, L=None): if L is None: L = []`. Then `h(1), h(2)` return `[1]`, `[2]`.

### 4.4 Lambda, map, filter

`lambda a, b: a * b` is an anonymous one-expression function.

```python
print((lambda a, b: a * b)(3, 4))                # 12
print(list(map(lambda v: v * 2, [1, 2])))        # [2, 4]
print(list(filter(None, [0, 1, "", 2])))         # [1, 2]   (None means "keep truthy")
print(max("apple", "fig", key=len))              # apple
```

`map` and `filter` return lazy iterators; wrap in `list(...)`. They can be consumed **only once**.

## 5. Scope: LEGB, `global`, `nonlocal`

**Rule.** When a name is *read*, Python looks in **L**ocal, then **E**nclosing functions, then **G**lobal (module), then **B**uilt-in. When a name is *assigned* anywhere in a function body, it is local **throughout that function** (decided at compile time) unless declared `global` or `nonlocal`.

**Worked example 8 (UnboundLocalError).**

```python
x = 1
def s():
    print(x)     # x is local in s (assigned below), but not yet bound
    x = 2
s()              # UnboundLocalError
```

**Worked example 9 (`global` and `nonlocal`).**

```python
x = 10
def p():
    x = 20              # local to p
    def q():
        nonlocal x      # refers to p's x
        x += 1
        return x
    return q()
print(p(), x)           # 21 10

def r():
    global x
    x += 5              # modifies module-level x
r(); print(x)           # 15
```

`p()` returns 21; global `x` is untouched (10) until `r()` adds 5.

A function reads globals **at call time**, not definition time:

```python
x = 5
def tw(): return x
x = 6
print(tw())    # 6
```

### 5.1 Closures and late binding

A **closure** is a function that remembers variables of its enclosing scope. The variable is captured, not its value at creation time.

```python
def counter():
    c = 0
    def inc():
        nonlocal c
        c += 1
        return c
    return inc

a = counter(); b = counter()
print(a(), a(), b(), a())     # 1 2 1 3   (each call to counter() has its own c)
```

**Worked example 10 (late binding in loops).**

```python
fs = [lambda: i for i in range(3)]
print([f() for f in fs])           # [2, 2, 2]

fs = [lambda i=i: i for i in range(3)]
print([f() for f in fs])           # [0, 1, 2]
```

All three lambdas share one `i`, which ends at 2 once the loop finishes; they read it only when called. The default-argument trick `i=i` freezes the value at definition time. The same applies to lambdas appended inside a `for` loop in a function.

### 5.2 Generators (basics)

A function containing `yield` returns a **generator**: calling it runs nothing; each `next()` runs until the next `yield`, then pauses.

```python
def gen(n):
    for i in range(n):
        yield i * i

G = gen(4)
print(next(G), next(G), list(G), list(G))   # 0 1 [4, 9] []
```

After `next` twice the generator is at value 1; `list(G)` drains the remaining `[4, 9]`; the next `list(G)` is `[]` because **generators are exhausted after one pass**. Generator expressions `(x*2 for x in ...)` behave the same.

## Formulas and facts to memorise

| Item | Fact | When to use |
| --- | --- | --- |
| Division identity | `a == (a//b)*b + a%b`; `%` takes sign of divisor | any `//`, `%` with negatives |
| Floor | `//` -> $\lfloor a/b \rfloor$; `int()` truncates toward 0 | `-7//2 = -4`, `int(-3.5) = -3` |
| Power | `**` right-assoc; `-2**2 = -4` | precedence puzzles |
| `and`/`or` | return operand; short-circuit | output of `x or y`, `x and y` |
| range length | $\max(0, \lceil (stop-start)/step \rceil)$ | counting iterations |
| Loop `else` | runs iff no `break` | tracing |
| Default args | evaluated once at `def` | mutable-default questions |
| LEGB | assignment makes a name local for the entire function | `UnboundLocalError` |
| Late binding | closures read the variable when *called* | lambdas in loops |
| Generator | one pass only; lazy | `list(g)` twice |
| Immutable | int, float, str, tuple, bool, None, frozenset | `+=` rebinds |

## GATE traps

- **`-7 // 2` is `-4`, not `-3`.** Floor, not truncation. And `-7 % 3` is `2`, not `-1`.
- **`/` returns a float even for exact division**: `10 / 5` is `2.0`; printing shows `2.0`. Use `//` for ints.
- **`-2 ** 2` is `-4`**; `2 ** 3 ** 2` is `512`.
- **`round(2.5)` is `2`** (ties to even), `round(1.5)` is `2`, `round(0.5)` is `0`.
- **`1 or 0/0` does not crash**; `0 and ...` never evaluates the right side. `x or y` may return a non-bool.
- **`1 == 1 < 2` is a chain**, not `(1 == 1) < 2` (though here both are True).
- **Loop variable persists** after the loop; assigning to it in the body does not alter iteration.
- **Loop `else` with `break`** is skipped.
- **Mutable default** is shared across calls; `print(g(1), g(2))` shows *the same list twice* because both prints happen after both calls.
- **`L = L + [x]` vs `L += [x]`**: new list vs in-place; matters when aliased or passed to a function.
- **`UnboundLocalError`**: reading a global and assigning it in the same function without `global`.
- **Lambdas in a loop** all see the final loop value unless bound via a default argument.
- **Generators are single-use**; `map`/`filter`/`zip`/`enumerate` objects too.
- **`"10" < "9"` is True** (string compare), `max("apple", "fig")` is `"fig"` (lexicographic) but `max(..., key=len)` is `"apple"`.
- **`print` returns `None`**: `type(print(""))` is `NoneType`.

## Connections

- [C basics and expressions](../05-c-programming/c-basics-and-expressions.md) — C truncates `/` and `%` toward zero, has fixed-width overflowing ints and no chained comparison; Python floors, has big ints, and chains.
- [Functions, recursion, structures (C)](../05-c-programming/functions-recursion-structures.md) — C is strictly call-by-value (pointers simulate sharing); Python passes object references.
- [Python collections](python-collections.md) — mutability and aliasing from section 1 and 4.2 drive every list/dict question.
- [Python OOP, recursion, tracing](python-oop-recursion-tracing.md) — scope and closures reappear in method binding and decorators/memoization.
- [Number representation](../09-digital-logic/number-representation-and-arithmetic.md) — why `0.1 + 0.2 != 0.3`; two's complement behind `~x == -x-1` and `>>`.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — counting loop iterations with `range` is the first step of complexity analysis.
- [Machine learning foundations](../16-machine-learning/ml-foundations.md) — NumPy/pandas code builds on these loops, slicing and comprehension idioms.

## Practice

**Q1 (MCQ).** What is printed by `print(-17 // 5, -17 % 5)`?
(A) `-3 -2`  (B) `-4 3`  (C) `-3 2`  (D) `-4 -3`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** $\lfloor -17/5 \rfloor = \lfloor -3.4 \rfloor = -4$. Remainder $= -17 - (-4)(5) = 3$ (sign of divisor 5 is positive). Check: $-4\cdot5 + 3 = -17$.

</details>

**Q2 (NAT).** What is the value of `2 ** 3 ** 2 // 10 % 7 - -3`?

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** `3 ** 2 = 9`; `2 ** 9 = 512` (right-associative). Then `*`/`//`/`%` left to right: `512 // 10 = 51`; `51 % 7 = 2` (since $51 = 7\cdot7 + 2$). Finally `2 - -3 = 2 + 3 = 5`.

</details>

**Q3 (NAT).** How many times is the body executed?

```python
c = 0
for i in range(20, 3, -4):
    c += 1
```

<details><summary>Answer</summary>

**Answer:** 5  
**Solution:** values 20, 16, 12, 8, 4 (next 0 is `<= 3`, stops). Count = $\lceil (20-3)/4 \rceil = \lceil 4.25 \rceil = 5$.

</details>

**Q4 (MCQ).** Output of:

```python
def f(x, L=[]):
    L.append(x)
    return len(L)
print(f(5), f(6), f(7, []), f(8))
```

(A) `1 1 1 1`  (B) `1 2 1 3`  (C) `1 2 3 4`  (D) `1 2 1 2`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `len` is returned as an int immediately, so no aliasing display issue. Shared default `D`: `f(5)` -> len 1; `f(6)` -> 2; `f(7, [])` uses a fresh list -> 1; `f(8)` -> `D` has 5, 6, 8 -> 3. Output `1 2 1 3`.

</details>

**Q5 (MSQ).** Which statements about the code are true?

```python
x = 3
def f():
    x = x + 1
    return x
def g():
    global x
    x = x + 1
    return x
```

(A) `f()` raises `UnboundLocalError`  (B) `g()` returns 4 and changes the global  (C) after `g(); g()` the global `x` is 5  (D) `f()` returns 4

<details><summary>Answer</summary>

**Answer:** (A), (B), (C)  
**Solution:** In `f`, `x` is assigned, so it is local; reading it on the right-hand side before binding raises `UnboundLocalError` (D is false). `g` declares `global`, so `g()` gives 4, and a second call gives 5.

</details>

**Q6 (MCQ).** What is printed?

```python
a = [lambda x, k=k: x * k for k in range(3)]
b = [lambda x: x * k for k in range(3)]
print(a[1](10), b[1](10))
```

(A) `10 10`  (B) `10 20`  (C) `20 10`  (D) `10 0`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `a[1]` has `k` frozen to 1: `10 * 1 = 10`. In `b` all lambdas share the comprehension's `k`, which is 2 at call time (the list-comprehension loop has finished): `b[1](10) = 20`.

</details>

**Q7 (NAT).** What is printed?

```python
def gen(n):
    for i in range(n):
        yield i * i
G = gen(5)
next(G); next(G)
print(sum(G))
print(sum(G))
```

Give the first printed value.

<details><summary>Answer</summary>

**Answer:** 29  
**Solution:** The generator yields 0, 1, 4, 9, 16. Two `next` calls consume 0 and 1. `sum(G)` adds the remaining $4 + 9 + 16 = 29$. The second `sum(G)` prints `0` (exhausted).

</details>

**Q8 (MCQ).** Output of:

```python
n = 0
for i in range(5):
    if i % 2:
        continue
    for j in range(i):
        if j == 2:
            break
        n += 1
    else:
        n += 10
print(n)
```

(A) 24  (B) 25  (C) 26  (D) 35

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** Only even `i` (0, 2, 4) run the inner loop.
- `i = 0`: `range(0)` is empty, no break -> `else` runs: `n = 10`.
- `i = 2`: `j = 0, 1` -> `n += 1` twice -> 12; `j` never reaches 2, no break -> `else`: `n = 22`.
- `i = 4`: `j = 0, 1` -> `n = 24`; `j = 2` -> `break` -> `else` skipped.

Final `n = 24`.

</details>
