# Python programming: cheat sheet

Full notes: [basics](python-basics.md) · [collections](python-collections.md) · [OOP, recursion, tracing](python-oop-recursion-tracing.md)

## Arithmetic and operators

| Item | Rule | Example |
| --- | --- | --- |
| `/` | true division, always float | `10/5 = 2.0` |
| `//` | floor (toward $-\infty$) | `-7//2 = -4` |
| `%` | sign of **divisor**; `a == (a//b)*b + a%b` | `-7%3 = 2`, `7%-3 = -2` |
| `int(x)` | truncates toward 0 | `int(-3.7) = -3` |
| `round` | ties to even | `round(2.5) = 2`, `round(1.5) = 2` |
| `**` | right-assoc; tighter than unary minus on left | `-2**2 = -4`, `2**3**2 = 512` |
| `~x` | `-x-1`; `>>` is floor shift | `~5 = -6`, `-5>>1 = -3` |
| Precedence | `**` > unary > `* / // %` > `+ -` > shifts > `&` > `^` > `\|` > comparisons > `not` > `and` > `or` | |
| Comparison chain | `a<b<c` = `a<b and b<c` | `1 == 1 < 2` |
| `x or y` / `x and y` | return an operand; short-circuit | `0 or "x" = "x"`, `3 and 5 = 5` |
| Falsy | `False None 0 0.0 "" [] () {} set()` | `"0"`, `[0]` are truthy |
| `==` vs `is` | value vs same object | `[1]==[1]` True, `is` False |
| Float | `0.1+0.2 != 0.3` | compare with tolerance |

## Loops, functions, scope

| Item | Rule |
| --- | --- |
| `range(a,b,s)` | half-open; length $\max(0,\lceil (b-a)/s\rceil)$ ($s>0$); `range(5,0)` empty |
| Loop variable | survives the loop; rebinding it in the body does not change iteration |
| Loop `else` | runs iff no `break` |
| `enumerate`, `zip` | zip stops at shortest; both are single-use iterators |
| `any([])` / `all([])` | `False` / `True` |
| Parameters | positional/default, `*args` (tuple), keyword-only, `**kw` (dict) |
| Pass by object reference | mutate: caller sees; rebind: caller does not |
| `L += x` vs `L = L + x` | in-place for lists vs new list |
| Default argument | evaluated **once** at `def`; mutable default shared |
| LEGB | Local, Enclosing, Global, Built-in; assignment makes name local for whole function |
| `UnboundLocalError` | read before assignment of a name that is assigned in the same function |
| `global` / `nonlocal` | rebind module / enclosing variable |
| Closure late binding | lambdas read the variable at call time; freeze with `i=i` |
| Generator | `yield`; lazy; one pass only |
| `map`, `filter` | lazy iterators, single-use; `filter(None, it)` keeps truthy |

## Strings

| Operation | Result |
| --- | --- |
| immutable; methods return new strings | `s.upper()` does not change `s` |
| `s[::-1]`, `s[1:4]`, `s[-3:]` | reverse, `s[1..3]`, last 3 |
| `"a,b,,c".split(",")` | `['a','b','','c']` |
| `"banana".count("an")` | 2 (non-overlapping) |
| `find` vs `index` | `-1` vs `ValueError` |
| `"10" < "9"` | True (lexicographic) |

## Collections

| Item | Rule |
| --- | --- |
| Slice `s[i:j:k]` | stop exclusive, clipped, new shallow copy, never `IndexError` |
| Negative step | defaults flip; `s[a:b:-1]` needs `a > b` |
| Methods returning `None` | `append extend insert sort reverse remove clear update` |
| `pop()` / `pop(i)` | returns removed item |
| Copy | `b=a` alias; `a[:]`, `list(a)`, `a.copy()` shallow; `deepcopy` deep |
| `[[0]*n]*m` | m references to one row; use comprehension |
| Tuple | immutable; `(1,)` needs the comma; can hold a mutable item |
| Dict | hash table, avg $O(1)$; insertion-ordered; keys hashable; `1 == 1.0 == True` one key |
| `d.get(k, default)` | no exception; `d[k]` raises `KeyError` |
| `in` on dict | tests keys |
| Change dict size while iterating | `RuntimeError` |
| `dict.fromkeys(ks, [])` | one shared list |
| Set ops | `\|` union, `&` intersection, `-` difference, `^` symmetric difference, `<=` subset |
| `remove` vs `discard` | `KeyError` vs silent |
| Empty set | `set()`; `{}` is a dict |
| Comprehension | `[e for x in it if c]`; own scope; nested `for` left to right |
| Removing from a list while iterating | skips elements |

Costs: list index/append $O(1)$, insert/pop(0)/`in` $O(n)$; dict/set lookup avg $O(1)$; deque both ends $O(1)$; `sorted` Timsort $O(n\log n)$ stable.

## OOP

| Item | Rule |
| --- | --- |
| `self` | instance passed implicitly; `o.m(a)` = `C.m(o, a)` |
| Attribute lookup | instance, class, bases (MRO) |
| `o.x = v` | always instance attribute (shadows class one) |
| Class-level mutable | `o.lst.append` shared; `o.lst = [...]` per instance |
| `__str__` / `__repr__` | `print(o)` / elements printed inside containers |
| `__eq__` without `__hash__` | unhashable |
| `super()` | next class in MRO (C3); diamond `W(Y,Z)`: W, Y, Z, X |
| `D.m()` without object | `TypeError` |

## Recursion

| Item | Fact |
| --- | --- |
| Depth limit | 1000 default; no tail-call optimisation |
| Naive Fibonacci calls | $C(n) = 2F_{n+1}-1$, $C(10)=177$ |
| Memoised Fibonacci calls | $2n-1$ ($n=10$: 19) |
| Hanoi | $2^n-1$ moves |
| Subsets / permutations | $2^n$ / $n!$ |
| Print before call / after call | descending (pre-order) / ascending (post-order) |
| Tracing | table of variable values per statement; call tree for recursion |
