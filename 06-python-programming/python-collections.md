# Python collections: lists, tuples, dictionaries, sets, comprehensions

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Lists; Tuples; Dictionaries; Sets; Comprehensions
> **Prerequisites:** [Python basics](python-basics.md) · **Leads to:** [OOP, recursion, tracing](python-oop-recursion-tracing.md) · [Data structures](../07-data-structures/arrays-stacks-queues.md)

## Quick glance

- **list** = ordered, mutable, allows duplicates, `O(1)` index and append, `O(n)` insert/delete at the front or middle. **tuple** = ordered, immutable. **dict** = key to value, keys unique and hashable, insertion-ordered. **set** = unordered unique hashable items.
- **Mutating methods return `None`**: `append`, `extend`, `insert`, `sort`, `reverse`, `remove`, `clear`. `x = L.sort()` makes `x` equal to `None`. `pop` returns the removed item.
- **Slicing** `s[i:j:k]`: stop is exclusive, out-of-range indices are clipped, slicing never raises `IndexError`, and a slice is always a **new (shallow) copy**. Negative step walks backwards: defaults flip to `[-1 : -len-1]`.
- **Aliasing:** `b = a` copies the label. `a[:]`, `list(a)`, `a.copy()` make a **shallow** copy (inner lists still shared); `copy.deepcopy` copies recursively.
- `[[0]*2]*2` makes **two references to one inner list**; use `[[0]*2 for _ in range(2)]`.
- `(1)` is an int; **`(1,)` is a tuple**. `{}` is an empty **dict**; empty set is `set()`.
- Dict keys must be **hashable** (no lists, dicts, sets). `1`, `1.0`, `True` are the *same key*.
- Never change a dict's size or remove from a list **while iterating over it**.
- Comprehension: `[expr for x in it if cond]`; the loop variable is local to the comprehension (not leaked in Python 3). Generator expressions `( ... )` are lazy and single-use.

## 1. Lists

**Intuition.** A list is a dynamic array of *references* to objects (like an array of pointers in C). Elements may have different types. See [arrays](../07-data-structures/arrays-stacks-queues.md).

### 1.1 Indexing and slicing

Index $i$ from the left starts at 0; from the right at $-1$. For `L = [10, 20, 30, 40, 50]`:

```text
index    0    1    2    3    4
value   10   20   30   40   50
neg    -5   -4   -3   -2   -1
```

**Slice rule `L[i:j:k]`.** Start at `i`, move by `k`, stop *before* reaching `j`.
- Positive `k` (default 1): default `i = 0`, default `j = len`.
- Negative `k`: default `i = -1` (last), default `j` = before the first element.
- Negative `i`/`j` are first adjusted by adding `len`; then clipped into range.

**Worked example 1.** `L = [1, 2, 3, 4, 5, 6]`:

| Expression | Steps | Result |
| --- | --- | --- |
| `L[::-1]` | start at last, step -1, to beginning | `[6, 5, 4, 3, 2, 1]` |
| `L[-2:]` | from index 4 to end | `[5, 6]` |
| `L[:-2]` | index 0 up to (not incl.) 4 | `[1, 2, 3, 4]` |
| `L[10:]` | start clipped to 6 = empty | `[]` (no error) |
| `L[1:5:2]` | indices 1, 3 | `[2, 4]` |
| `L[5:1:-2]` | indices 5, 3 (stop before 1) | `[6, 4]` |
| `L[-1:-4:-1]` | indices 5, 4, 3 | `[6, 5, 4]` |

`L[10]` **does** raise `IndexError`, but `L[10:]` does not.

### 1.2 Slice assignment and `del`

```python
L = [1, 2, 3]; L[1:2] = [7, 8, 9]; print(L)     # [1, 7, 8, 9, 3]   (length can change)
L[::2] = [0, 0, 0]; print(L)                    # [0, 7, 0, 9, 0]   (extended slice: sizes must match)
L = [1, 2, 3, 4, 5]; del L[1:3]; print(L)       # [1, 4, 5]
L[1:1] = [9]; print(L)                          # [1, 9, 4, 5]      (insert without removing)
L[-1:] = []; print(L)                           # [1, 9, 4]         (delete last)
```

A plain slice (`step 1`) may be assigned a sequence of any length; an **extended slice** (`step != 1`) must be assigned a sequence of exactly the same length or you get `ValueError`.

### 1.3 Methods: what they return

| Method | Effect | Returns | Cost |
| --- | --- | --- | --- |
| `L.append(x)` | add at end | **`None`** | $O(1)$ amortised |
| `L.extend(it)` / `L += it` | add all items | `None` | $O(k)$ |
| `L.insert(i, x)` | insert before index `i` | `None` | $O(n)$ |
| `L.remove(x)` | delete first equal value (`ValueError` if absent) | `None` | $O(n)$ |
| `L.pop()` / `L.pop(i)` | delete & return (default last) | the item | $O(1)$ / $O(n)$ |
| `L.sort()` | in-place sort (stable) | **`None`** | $O(n \log n)$ |
| `sorted(L)` | new sorted list | list | $O(n \log n)$ |
| `L.reverse()` | in-place | `None` | $O(n)$ |
| `L.index(x)` | first position | int | $O(n)$ |
| `L.count(x)` | occurrences | int | $O(n)$ |
| `x in L` | membership | bool | $O(n)$ |

**Worked example 2 (return values).**

```python
L = [3, 1, 2]
r = L.append(4)      # L = [3, 1, 2, 4], r = None
s = L.sort()         # L = [1, 2, 3, 4], s = None
t = L.reverse()      # L = [4, 3, 2, 1], t = None
print(r, s, t, L)    # None None None [4, 3, 2, 1]
print(sorted(L), L)  # [1, 2, 3, 4] [4, 3, 2, 1]   (sorted() does not touch L)
```

`L.sort()` is the classic "I assigned the result and got `None`" trap.

**Operators.** `[1,2] + [3]` is `[1, 2, 3]`; `[1] * 3` is `[1, 1, 1]`; lists compare **lexicographically**: `[1,2] < [1,3]` is `True`, and `[1,2] < [1,2,0]` is `True` (a prefix is smaller).

### 1.4 Aliasing and copying

```text
a = [1, 2, 3]
b = a            # same object        a ─┬─► [1,2,3]
c = a[:]         # new list            b ─┘
d = list(a)      # new list            c ───► [1,2,3]    d ───► [1,2,3]
a.append(4)
```

Result: `b` is `[1, 2, 3, 4]` (alias); `c` and `d` stay `[1, 2, 3]`.

**Worked example 3 (shallow vs deep).**

```python
import copy
a  = [[1, 2], [3]]
s  = a.copy()               # shallow: new outer list, SAME inner lists
dd = copy.deepcopy(a)       # fully independent
a[0].append(9)              # mutate an inner list (shared with s)
a.append(0)                 # change the outer list (only a)
print(s)    # [[1, 2, 9], [3]]       inner change visible, outer append not
print(dd)   # [[1, 2], [3]]          untouched
print(a)    # [[1, 2, 9], [3], 0]
```

**Worked example 4 (the `*` trap).**

```python
a = [[0] * 2] * 2         # [[0,0],[0,0]] but BOTH rows are the same list
a[0][0] = 1
print(a)                  # [[1, 0], [1, 0]]

b = [[0] * 2 for _ in range(2)]   # independent rows
b[0][0] = 1
print(b)                  # [[1, 0], [0, 0]]
```

**`+=` vs `+` with aliases.**

```python
L = [1, 2]; L += [3]      # in-place: same object
M = L
L = L + [4]               # new object bound to L
print(L, M)               # [1, 2, 3, 4] [1, 2, 3]
```

### 1.5 Modifying while iterating

The iterator walks by position; deleting shifts later items left and one is skipped.

```python
L = [1, 2, 3]
for x in L:
    if x == 1: L.remove(x)
print(L)                  # [2, 3]   -> iteration visited index 0 (1), then index 1 (now 3); 2 was skipped

L = [1, 2, 3, 4]
for x in L[:]:            # iterate over a copy
    if x % 2 == 0: L.remove(x)
print(L)                  # [1, 3]
```

### 1.6 Lists as stack and queue

- **Stack:** `append` to push, `pop()` to pop - both $O(1)$.
- **Queue:** `pop(0)` is $O(n)$. Use `collections.deque`: `append`, `appendleft`, `pop`, `popleft` all $O(1)$.

```python
from collections import deque
q = deque([1, 2, 3]); q.append(4); q.appendleft(0)   # deque([0, 1, 2, 3, 4])
print(q.popleft(), q.pop(), q)                       # 0 4 deque([1, 2, 3])
```

See [stacks and queues](../07-data-structures/arrays-stacks-queues.md).

## 2. Tuples

An **immutable** ordered sequence. It supports indexing, slicing, `+`, `*`, `in`, `len`, `count`, `index`, comparison; it has no `append`/`sort`.

```python
t = (1, 2, 3)
print(t[1:], t + (4,), t * 2)     # (2, 3) (1, 2, 3, 4) (1, 2, 3, 1, 2, 3)
print(type((1)), type((1,)), type(()))   # int tuple tuple
```

- **A tuple is defined by the comma**, not the parentheses: `x = 1,` is a tuple; `(1)` is an int.
- **Immutability is shallow:** a tuple *containing* a list can still have that list mutated.

```python
t = ([1], 2)
t[0].append(5)        # allowed: the list object changes
print(t)              # ([1, 5], 2)
t[1] = 3              # TypeError: tuple does not support item assignment
```

- **Hashable** if all its elements are hashable, so tuples can be dict keys and set members (lists cannot).
- **Unpacking:**

```python
a, b = 1, 2;  a, b = b, a            # swap -> a=2, b=1 (right side evaluated fully first)
a, *b, c = [1, 2, 3, 4, 5]           # a=1, b=[2, 3, 4], c=5   (starred name always gets a list)
```

- Tuples compare lexicographically like lists: `(1, 2) < (1, 3)` is `True`.
- `sorted(...)` on a list of tuples sorts by first field, then second, ...: `sorted([(2,'a'),(1,'z'),(2,'A')])` gives `[(1,'z'), (2,'A'), (2,'a')]` (`'A'` is 65 < `'a'` 97).

## 3. Dictionaries

**Intuition.** A dict is a **hash table** (see [hashing](../08-algorithms/hashing.md)): the key is hashed to a slot, so lookup, insert and delete are $O(1)$ on average. Since Python 3.7 dicts remember **insertion order**; updating an existing key keeps its original position; deleting and re-inserting moves it to the end.

```python
d = {'a': 1, 'b': 2}
d['c'] = 3          # insert  -> {'a':1,'b':2,'c':3}
d['a'] = 10         # update, position unchanged -> {'a':10,'b':2,'c':3}
print(list(d), list(d.values()), list(d.items()))
# ['a', 'b', 'c'] [10, 2, 3] [('a', 10), ('b', 2), ('c', 3)]
```

| Operation | Behaviour |
| --- | --- |
| `d[k]` | value; **`KeyError`** if absent |
| `d.get(k)` / `d.get(k, default)` | value or `None` / `default`; never raises |
| `d.pop(k)` | remove and return; `KeyError` if absent (`d.pop(k, dflt)` safe) |
| `d.setdefault(k, v)` | insert `k: v` only if absent; **returns the value** now stored |
| `d.update(other)` | overwrite/add; returns `None` |
| `d.popitem()` | remove and return the **last** inserted `(k, v)` |
| `k in d` | tests **keys**, not values |
| iteration `for k in d` | yields keys; `len(d)` counts keys |
| `del d[k]` | remove key |

**Hashability.** Keys must be hashable: ints, floats, strs, tuples of hashables, `frozenset`. Lists, dicts and sets are **not** (`{[1]: 2}` raises `TypeError: unhashable type`).

**Equal keys collapse.** Because `1 == 1.0 == True` and they hash equally: `{1: 'a', 1.0: 'b', True: 'c'}` is `{1: 'c'}` (the key stays `1`, the value is the last one assigned).

**Worked example 5 (frequency count).**

```python
dd = {}
for w in "abracadabra":
    dd[w] = dd.get(w, 0) + 1
print(dd)       # {'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1}
```

Trace the first appearances in order: a, b, r, a, c, a, d, a, b, r, a. Keys are inserted in first-seen order a, b, r, c, d. Final counts: a appears at positions 1, 4, 6, 8, 11 -> 5; b -> 2; r -> 2; c -> 1; d -> 1.

Sort by count descending, then key ascending:

```python
print(sorted(dd.items(), key=lambda kv: (-kv[1], kv[0])))
# [('a', 5), ('b', 2), ('r', 2), ('c', 1), ('d', 1)]
```

**Building dicts.** `{k: v for k, v in zip("abc", range(3))}` gives `{'a':0,'b':1,'c':2}`; `dict(zip("ab",[1,2]))` gives `{'a':1,'b':2}`; `dict.fromkeys("ab", 0)` gives `{'a':0,'b':0}` (all values the *same* object - dangerous with mutable values).

**Size change during iteration.**

```python
d = {'x': 1}
for k in d:
    d[k + 'y'] = 1      # RuntimeError: dictionary changed size during iteration
```

(Changing the *value* of an existing key is allowed; adding/removing keys is not.) Iterate over `list(d)` if you need to mutate.

**Collections.Counter** counts hashables: `Counter("aabbbc").most_common(2)` gives `[('b', 3), ('a', 2)]`.

## 4. Sets

An unordered collection of **unique, hashable** items with $O(1)$ average membership test. Creating from an iterable removes duplicates.

```python
print({1, 2, 2, 3}, len({1, 1, 1}), sorted(set([3, 1, 3, 2])))   # {1, 2, 3} 1 [1, 2, 3]
print(type({}), type({1}), type(set()))     # dict set set
```

**Never depend on the printed order** of a set of strings (hash randomisation); small non-negative ints usually print in increasing order but that is not a guarantee GATE relies on, so questions use `sorted` or `len`.

**Operations**, with `s = {1, 2, 3}`, `t = {3, 4}`:

| Operation | Symbol | Method | Result |
| --- | --- | --- | --- |
| union | `s \| t` | `s.union(t)` | `{1, 2, 3, 4}` |
| intersection | `s & t` | `s.intersection(t)` | `{3}` |
| difference | `s - t` | `s.difference(t)` | `{1, 2}` |
| symmetric difference | `s ^ t` | `s.symmetric_difference(t)` | `{1, 2, 4}` |
| subset / proper subset | `s <= t`, `s < t` | `issubset` | `False`, `False` |
| superset | `>=`, `>` | `issuperset` | |

- `s.add(x)`; `s.discard(x)` (silent if absent); `s.remove(x)` (**`KeyError`** if absent); `s.pop()` removes an arbitrary element.
- `{1,2} == {2,1}` is `True` (order irrelevant). Set elements and dict keys must be hashable: a set of lists fails, a set of tuples works.
- `|`, `&`, `-`, `^` have precedence of the *bitwise* operators, lower than `+`, higher than comparisons - parenthesise.
- Cardinality identity: $|s \cup t| = |s| + |t| - |s \cap t|$. See [sets](../01-discrete-mathematics/sets-relations-functions.md).

Typical uses: de-duplicate (`len(set(L))`), fast membership, "elements in A but not B".

## 5. Comprehensions

**Intuition.** A comprehension is the Python spelling of "set-builder notation": $\{x^2 \mid x \in \text{range}(5),\ x \text{ even}\}$.

```text
[ expression  for item in iterable  (if condition) ]
        ^             ^                    ^
   what to keep   how to loop         optional filter
```

| Form | Example | Result |
| --- | --- | --- |
| list | `[x*x for x in range(5) if x%2==0]` | `[0, 4, 16]` |
| set | `{x % 3 for x in range(10)}` | `{0, 1, 2}` |
| dict | `{x: x*x for x in range(3)}` | `{0: 0, 1: 1, 2: 4}` |
| generator | `(x*2 for x in range(3))` | lazy; `list(g)` gives `[0, 2, 4]`, then `[]` |

**Conditional expression vs filter.** `[x if x > 1 else -x for x in range(4)]` gives `[0, -1, 2, 3]` (the `if/else` is part of the expression, goes *before* `for`). A bare `if` goes *after* `for` and filters.

**Nested loops** read left to right, exactly like nested `for` statements:

```python
print([(i, j) for i in range(2) for j in range(2)])    # [(0,0), (0,1), (1,0), (1,1)]
print([[j for j in range(i)] for i in range(3)])       # [[], [0], [0, 1]]
print([i*j for i in range(1, 4) for j in range(i)])    # [0, 0, 2, 0, 3, 6]
```

Third trace: `i=1`: `j=0` -> 0. `i=2`: `j=0,1` -> 0, 2. `i=3`: `j=0,1,2` -> 0, 3, 6. Concatenated: `[0, 0, 2, 0, 3, 6]`.

**Scoping.** In Python 3 the comprehension variable does **not** leak:

```python
x = 100
print([x for x in range(3)], x)       # [0, 1, 2] 100
```

(But a plain `for` loop variable *does* leak.) A lambda inside a comprehension still has late binding (see [basics](python-basics.md)).

## 6. Choosing and combining: cost table

| Task | list | tuple | dict | set | deque |
| --- | --- | --- | --- | --- | --- |
| index `x[i]` | $O(1)$ | $O(1)$ | by key $O(1)$ | n/a | $O(n)$ mid |
| `x in c` | $O(n)$ | $O(n)$ | $O(1)$ (keys) | $O(1)$ | $O(n)$ |
| append / add | $O(1)$ | n/a | $O(1)$ | $O(1)$ | $O(1)$ |
| insert/pop at front | $O(n)$ | n/a | n/a | n/a | $O(1)$ |
| mutable | yes | no | yes | yes | yes |
| ordered | yes | yes | insertion | no | yes |

More in [complexity reference](../07-data-structures/complexity-reference.md).

**Worked example 6 (word-index problem).** Given `words = ["to", "be", "or", "not", "to", "be"]`, produce a dict from word to the list of positions.

```python
pos = {}
for i, w in enumerate(words):
    pos.setdefault(w, []).append(i)
print(pos)    # {'to': [0, 4], 'be': [1, 5], 'or': [2], 'not': [3]}
```

`setdefault(w, [])` returns the stored list (new or existing), and `append` mutates it. Each key appears in first-seen order.

## Formulas and facts to memorise

| Item | Fact | When to use |
| --- | --- | --- |
| Slice | `s[i:j:k]`, stop exclusive, clipped, new shallow copy | all slicing output |
| Reverse slice | `s[::-1]`; `s[a:b:-1]` needs `a > b` | negative-step slices |
| `None`-returning methods | `append extend insert sort reverse remove clear update` | "what is printed" |
| Copying | `=` alias; `[:] list() .copy()` shallow; `deepcopy` deep | aliasing questions |
| Tuple | comma makes it: `(1,)` | type questions |
| Dict | hash table, $O(1)$ avg; insertion-ordered; keys hashable | dict tracing |
| `get` vs `[]` | `get` never raises | missing-key cases |
| Set ops | `\| & - ^ <=` | set algebra |
| Comprehension | `[e for x in it if c]`, own scope | rewriting loops |
| Hash equality | `1 == 1.0 == True` one key | dict key collapse |

## GATE traps

- **`x = L.sort()` / `L.append(..)` give `None`.** `print(L.append(1))` prints `None`.
- **`[[0]*n]*m`** creates aliased rows; changing one cell changes a whole column.
- **`b = a` is not a copy.** `a[:]` is shallow; nested lists are still shared.
- **`L += x` mutates in place; `L = L + x` rebinds** - visible through other names or inside functions.
- **`(5)` is an int.** `(5,)` is a tuple. `{}` is a dict.
- **Extended slice assignment requires equal length**, simple slice assignment does not.
- **`L[10:]` is `[]` but `L[10]` raises `IndexError`.**
- **Removing from a list while iterating it skips elements.**
- **`'b' in d` checks keys;** `2 in d` is `False` for `{'a':1,'b':2}`.
- **Adding keys during iteration raises `RuntimeError`.**
- **`{1: 'a', 1.0: 'b', True: 'c'}` has one key.**
- **`s.remove(x)` raises `KeyError`, `s.discard(x)` does not.**
- **`dict.fromkeys(keys, [])` shares one list** among all keys.
- **Comprehension variable does not leak** (a `for` statement variable does).
- **A generator expression can be consumed once;** a second `list(g)` is `[]`.
- **Do not predict set print order for strings;** compare with `sorted` or `==`.

## Connections

- [Python basics](python-basics.md) — mutability, `is` vs `==`, and `+=` semantics underlie every aliasing question here.
- [Arrays, stacks, queues](../07-data-structures/arrays-stacks-queues.md) — Python list = dynamic array; `append`/`pop` = stack; `deque` = queue.
- [Hashing](../08-algorithms/hashing.md) — dict and set are hash tables; hashability requirement = stable hash; average $O(1)$ vs worst-case $O(n)$.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — `sorted`/`list.sort` is Timsort, $O(n \log n)$ and **stable**; `in` on a list is linear search.
- [Sets, relations, functions](../01-discrete-mathematics/sets-relations-functions.md) — set operations and cardinality; a dict is a finite function.
- [Counting for probability](../02-probability-statistics/probability-basics.md) — `itertools`-style comprehensions enumerate sample spaces (`[(i,j) for i in ... for j in ...]`).
- [Pointers and arrays in C](../05-c-programming/pointers-arrays-strings.md) — a list is an array of references; aliasing is Python's pointer sharing; C arrays have no slicing or bounds clipping.
- [ML foundations](../16-machine-learning/ml-foundations.md) — dataset rows as tuples/dicts, train/test splits as slices, label counts as `Counter`.

## Practice

**Q1 (MCQ).** What is printed?

```python
L = [5, 3, 8]
M = L.sort()
print(M, L)
```

(A) `[3, 5, 8] [3, 5, 8]`  (B) `None [3, 5, 8]`  (C) `None [5, 3, 8]`  (D) `[3, 5, 8] [5, 3, 8]`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `list.sort()` sorts in place and returns `None`, so `M` is `None` and `L` is `[3, 5, 8]`.

</details>

**Q2 (NAT).** For `L = [2, 4, 6, 8, 10, 12, 14]`, what is `sum(L[-1:1:-2])`?

<details><summary>Answer</summary>

**Answer:** 30  
**Solution:** Negative step: start at index $-1 = 6$, stop before index 1, step -2. Indices visited: 6, 4, 2 (the next, 0, is not above the stop index 1). Values: 14, 10, 6. Sum = 30.

</details>

**Q3 (MCQ).** What is printed?

```python
a = [[1, 2], [3, 4]]
b = a[:]
b[0][0] = 99
b[1] = [7]
print(a)
```

(A) `[[1, 2], [3, 4]]`  (B) `[[99, 2], [3, 4]]`  (C) `[[99, 2], [7]]`  (D) `[[1, 2], [7]]`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** `a[:]` is a shallow copy: `b[0]` and `a[0]` are the same inner list, so `b[0][0] = 99` shows in `a`. `b[1] = [7]` rebinds slot 1 of the *new outer list only*, so `a[1]` stays `[3, 4]`.

</details>

**Q4 (NAT).** How many keys does `d` have at the end, and what is `d[1]`?

```python
d = {}
d[1] = 'a'
d[1.0] = 'b'
d[True] = 'c'
d[(1, 2)] = 'd'
d['1'] = 'e'
```

Give the number of keys.

<details><summary>Answer</summary>

**Answer:** 3 keys (and `d[1] == 'c'`)  
**Solution:** `1`, `1.0` and `True` are equal and have equal hashes, so they are one key (value overwritten each time: `'c'`). Plus `(1, 2)` and `'1'` -> total 3.

</details>

**Q5 (MSQ).** Which of these evaluate to `True`?

(A) `{1, 2, 3} - {2} == {1, 3}`  (B) `{1, 2} | {2, 3} == {1, 2, 3}`  (C) `len({1, 2, 3} ^ {3, 4}) == 3`  (D) `{} == set()`

<details><summary>Answer</summary>

**Answer:** (A), (B), (C)  
**Solution:** (A) `-` binds tighter than `==`: `{1, 3} == {1, 3}` is True. (B) `|` binds tighter than `==` (comparisons are lower than bitwise operators): `{1,2,3} == {1,2,3}` is True. (C) `{1,2,3} ^ {3,4} = {1,2,4}`, length 3: True. (D) `{}` is an empty dict and `set()` an empty set: False.

</details>

**Q6 (MCQ).** What is printed?

```python
d = {'a': 1, 'b': 2}
x = d.setdefault('a', 10)
y = d.setdefault('c', 30)
z = d.pop('b')
print(x, y, z, list(d.items()))
```

(A) `1 30 2 [('a', 1), ('c', 30)]`  (B) `10 30 2 [('a', 10), ('c', 30)]`  (C) `1 30 2 [('a', 1), ('b', 2), ('c', 30)]`  (D) `1 None 2 [('a', 1), ('c', 30)]`

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** `setdefault('a', 10)` finds `a` and returns its existing value 1 without changing it. `setdefault('c', 30)` inserts and returns 30. `pop('b')` returns 2 and removes it. Remaining items in insertion order: `('a', 1), ('c', 30)`.

</details>

**Q7 (NAT).** What is `len(R)`?

```python
R = [(x, y) for x in range(1, 5) for y in range(x, 5) if (x + y) % 2 == 0]
```

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:** pairs with $y \ge x$ and $x + y$ even (same parity):
- `x=1`: y in {1, 3} -> 2 pairs
- `x=2`: y in {2, 4} -> 2 pairs
- `x=3`: y in {3} -> 1 pair
- `x=4`: y in {4} -> 1 pair

Total 6.

</details>

**Q8 (MCQ).** What is printed?

```python
L = [2, 4, 6, 8, 1]
for x in L:
    if x % 2 == 0:
        L.remove(x)
print(L)
```

(A) `[1]`  (B) `[4, 8, 1]`  (C) `[2, 4, 6, 8, 1]`  (D) `[4, 6, 8, 1]`

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** The loop walks by position.
- idx 0: `x = 2`, remove it, list becomes `[4, 6, 8, 1]`.
- idx 1: `x = 6` (4 slid into index 0 and was skipped), remove it, list `[4, 8, 1]`.
- idx 2: `x = 1`, odd, kept. idx 3 is out of range, so the loop stops.

Result `[4, 8, 1]`; 4 and 8 survive even though they are even.

</details>
