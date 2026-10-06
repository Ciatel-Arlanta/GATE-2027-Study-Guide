# C Basics and Expressions

> **Paper:** CS · **Priority:** P0 · **Plan topics:** C syntax and semantics; Tracing C programs and expressions
> **Prerequisites:** [Number representation](../09-digital-logic/number-representation-and-arithmetic.md) · **Leads to:** [Pointers, arrays, strings](pointers-arrays-strings.md), [Functions, recursion, structures](functions-recursion-structures.md)

Machine assumption used in this whole section (state it in the exam answer if the question does not): **64-bit Linux-style model (LP64): `char`=1, `short`=2, `int`=4, `long`=8, `long long`=8, `float`=4, `double`=8, pointer=8 bytes; little-endian; two's complement.** Many GATE questions say "assume `int` is 4 bytes" (older ones say 2); always read the stated sizes.

## Quick glance

- **Integer promotion:** `char`/`short` operands become `int` before arithmetic. If an `int` meets an `unsigned int`, the `int` becomes unsigned: `-1 > 1u` is **true**.
- **Signed overflow is undefined; unsigned overflow wraps modulo 2^n.** `unsigned char` 255 + 1 stored back gives 0.
- **Precedence ladder:** `() [] -> .` > unary (`! ~ ++ -- * & sizeof`, casts) > `* / %` > `+ -` > `<< >>` > `< <= > >=` > `== !=` > `&` > `^` > `|` > `&&` > `||` > `?:` > `= op=` > `,`.
- `&&`, `||`, `?:`, `,` have **sequence points** and left operand evaluated first; `&&`/`||` **short-circuit**.
- **Undefined behaviour (UB):** modifying a variable twice (or modifying and reading it for another purpose) between sequence points: `i = i++`, `a[i] = i++`, `f(i++, i++)`. GATE "output" questions avoid UB; if you see it, the answer is usually "undefined / compiler-dependent".
- `switch` falls through until `break`; `default` can sit anywhere.
- `static` local: initialised **once**, keeps value across calls. Static (lexical) scoping is what C uses; dynamic scoping looks up the **call chain**.
- `#define` macros are textual: `SQ(a+1)` with `#define SQ(x) x*x` becomes `a+1*a+1`.
- `x & (x-1)` clears the lowest set bit; `x & -x` isolates it.

## 1. Types, sizes and constants

A **type** tells the compiler how many bytes a value occupies and how to interpret the bits.

| Type | Bytes | Range (two's complement) |
|---|---|---|
| `char` | 1 | -128..127 (signedness is implementation-defined; `unsigned char` 0..255) |
| `short` | 2 | -32768..32767 |
| `int` | 4 | -2^31..2^31-1 = -2147483648..2147483647 |
| `unsigned int` | 4 | 0..2^32-1 = 4294967295 |
| `long`, `long long` | 8 | -2^63..2^63-1 |
| `float` / `double` | 4 / 8 | about 7 / 15 decimal digits of precision |

**Constants.** `10` decimal, `010` **octal** (= 8), `0x1F` hex (= 31), `10u` unsigned, `10L` long, `3.5` is `double`, `3.5f` is `float`, `'A'` is an `int` with value 65.

**`sizeof`** is a compile-time operator returning `size_t` (unsigned). It does **not evaluate** its operand:

```c
int i = 5;
printf("%zu %d", sizeof(i++), i);   // prints "4 5": i++ is never executed
```

**Char arithmetic.** `'a'` is just 97. `'a' + 1` is the `int` 98; `printf("%c", 'a'+1)` prints `b`. `'a' - 'A'` = 32. `'7' - '0'` = 7 (digit char to value). `'0' + 5` = `'5'`.

## 2. Conversions and the signed/unsigned trap

**Rules, in order, for a binary operator:**
1. `char`, `short` (and `bool`, bit-fields) are **promoted to `int`**.
2. If types still differ, convert to the "higher" one: `int` < `unsigned int` < `long` < `unsigned long` < `float` < `double` < `long double`. (On LP64, `long` can hold every `unsigned int`, so `long` vs `unsigned int` becomes `long`.)
3. **Signed vs unsigned of the same rank: the signed one becomes unsigned** (bit pattern reinterpreted).

**Worked example 1.** `printf("%d", -1 > 1u);`
- `-1` is `int`, `1u` is `unsigned int` → `-1` converts to unsigned: bit pattern 0xFFFFFFFF = 4294967295.
- `4294967295 > 1` → true. Output **1**.

**Worked example 2.** `unsigned int x = 5; int y = -10; printf("%d", x + y > 0);`
- `y` → unsigned: 2^32 - 10 = 4294967286.
- `x + y` = 5 + 4294967286 = 4294967291 (no wrap yet, < 2^32). `> 0` true → **1**. (Mathematically 5-10 = -5, but unsigned arithmetic never goes negative.)

**Worked example 3 (promotion).**
```c
char a = 100, b = 100;
int c = a + b;            // a, b promoted to int: 200, no overflow
char d = a + b;           // 200 does not fit signed char: implementation-defined (usually -56)
unsigned char u = 255;
int e = u + 1;            // 256 (promoted)
u = u + 1;                // 256 converted back to unsigned char: 256 mod 256 = 0
```

**Division and mixing.**

| Expression | Value | Why |
|---|---|---|
| `5 / 2` | 2 | integer division truncates toward zero |
| `-7 / 2`, `-7 % 2` | -3, -1 | C99: quotient truncates to 0; `a == (a/b)*b + a%b`; sign of `%` follows the dividend |
| `5 / 2.0`, `(float)5 / 2` | 2.5 | cast binds tighter than `/`, so 5 becomes float first |
| `(float)(5/2)` | 2.0 | integer division happened first |
| `1/2 * 4.0` | 0.0 | `1/2` is integer 0 |
| `int x = 3.99` | 3 | float to int truncates |
| `0.1 + 0.2 == 0.3` | false | binary floating point is inexact |

**Overflow.** Unsigned: wraps (`unsigned u = 0; u--;` → 4294967295). Signed: **undefined** (`INT_MAX + 1`). `1 << 32` on a 32-bit `int` is also undefined (shift count >= width).

**Classic unsigned traps.**
- `for (unsigned i = 5; i >= 0; i--)` never terminates: `i >= 0` is always true.
- `sizeof(int) > -1` is **false**: `sizeof` is unsigned, `-1` becomes huge.
- `strlen(s) - 10 > 0` is true even when `strlen(s) < 10`.

## 3. Operators, precedence and associativity

| Level | Operators | Assoc. |
|---|---|---|
| 1 | `()` `[]` `->` `.` postfix `++` `--` | L→R |
| 2 | prefix `++` `--`, unary `+` `-` `!` `~` `*` `&` `sizeof`, `(type)` | **R→L** |
| 3 | `*` `/` `%` | L→R |
| 4 | `+` `-` | L→R |
| 5 | `<<` `>>` | L→R |
| 6 | `<` `<=` `>` `>=` | L→R |
| 7 | `==` `!=` | L→R |
| 8 | `&` | L→R |
| 9 | `^` | L→R |
| 10 | `\|` | L→R |
| 11 | `&&` | L→R |
| 12 | `\|\|` | L→R |
| 13 | `?:` | **R→L** |
| 14 | `=` `+=` `-=` ... | **R→L** |
| 15 | `,` | L→R |

**Precedence says how to group; it does not say which operand is evaluated first.** `f() + g()` may call either first (unspecified, not UB, but output order of prints inside them is not defined).

**Worked example 4.** `printf("%d", 5 + 3 * 2 % 4);`
- `*` and `%` same level, left to right: `3*2` = 6, `6 % 4` = 2; then `5 + 2` = **7**.

**Worked example 5.** `printf("%d", 2 + 3 > 4 && 1 << 2 == 4);`
- `+` first: 5; `<<` first among the right side: `1<<2` = 4.
- `5 > 4` = 1; `4 == 4` = 1; `1 && 1` = **1**. (Shift binds tighter than `==`.)

**Worked example 6.** `printf("%d", 7 & 3 | 4 ^ 1);`
- `&` > `^` > `|`: `7&3` = 3; `4^1` = 5; `3 | 5` = **7**.

**Ternary (right-assoc.).** `x > 3 ? x < 10 ? 1 : 2 : 3` with `x = 5`: outer condition true → inner `x < 10 ? 1 : 2` → **1**.

**Comma.** `a = (1, 2, 3);` → 3 (value of last). `x = 5, 6;` parses as `(x = 5), 6` → x is 5 because `=` binds tighter than `,`.

**Assignment is an expression.** `a = b = c = 7` assigns right to left. `if (x = 5)` assigns and tests (always true): the classic typo for `==`.

**Relational chains.** `a < b < c` means `(a < b) < c`, comparing 0/1 to `c`.

## 4. Increment/decrement, sequence points, undefined behaviour

`i++` yields the **old** value then increments; `++i` increments then yields the **new** value.

```c
int i = 5, j;
j = i++ + 10;   // j = 15, i = 6
j = ++i + 10;   // i = 7, j = 17
```

**Sequence point:** a point at which all earlier side effects are complete. They occur at: end of a full expression (`;`), after the first operand of `&&`, `||`, `?:`, `,`; before a function body runs (after arguments are evaluated); at the end of a full declarator.

**Rule that creates UB:** between two sequence points, an object may be modified **at most once**, and if modified, may be read **only to compute the value to store**.

| Expression | Verdict |
|---|---|
| `i = i++;` | UB (two modifications of `i`) |
| `a[i] = i++;` | UB (reads `i` for the index and modifies it) |
| `x = i++ + ++i;` | UB |
| `printf("%d %d", i++, i);` | UB |
| `f(i++, i++);` | UB |
| `i = i + 1;` | fine (read used to compute the new value) |
| `i++ && i++` | fine (sequence point at `&&`) |
| `f() + g()` | **unspecified order**, not UB |
| `INT_MAX + 1`, `1 << 32`, `x / 0` (int) | UB |

**GATE use:** if the output depends on a compiler's evaluation order, the intended answer is "undefined" (or the question will not be asked). Do not "pick the usual behaviour".

## 5. Short-circuit, logical and bitwise operators

`A && B`: if `A` is 0, `B` is **not evaluated**. `A || B`: if `A` is nonzero, `B` is not evaluated. Result is always 0 or 1.

**Worked example 7.**
```c
int a = 1, b = 2, c = 3, r;
r = (a-- > 0) || (b++ > 0) && (c-- > 0);
```
- `&&` binds tighter: `(a-- > 0) || ((b++ > 0) && (c-- > 0))`.
- Left of `||`: `a-- > 0` → 1 > 0 true, `a` becomes 0. Short-circuit: right side skipped.
- `r = 1`, `a = 0`, `b = 2`, `c = 3`.

**Worked example 8 (bit tricks).** Count set bits of `x = 44` (binary 101100):
```c
int count = 0;
while (x) { x = x & (x - 1); count++; }
```
| iteration | x (binary) | x-1 | x & (x-1) |
|---|---|---|---|
| 1 | 101100 (44) | 101011 | 101000 (40) |
| 2 | 101000 (40) | 100111 | 100000 (32) |
| 3 | 100000 (32) | 011111 | 0 |

`count = 3` = number of 1-bits (Kernighan's algorithm). One iteration per set bit.

| Operation | Meaning |
|---|---|
| `x << n` | x · 2^n (if no overflow) |
| `x >> n` | floor(x / 2^n) for non-negative; for negative signed it is implementation-defined (usually arithmetic: `-8 >> 1` = -4) |
| `~x` | bitwise NOT = `-x - 1`; `~5` = -6 |
| `x & -x` | isolates lowest set bit (44 → 4) |
| `x & (x-1) == 0` | x is 0 or a power of two (beware precedence: write `(x & (x-1)) == 0`) |
| `a ^ b ^ b` | = `a`; XOR swap: `a^=b; b^=a; a^=b;` |
| `x ^ x` | 0 |

**Worked example 9.** `int x = 0x0F, y = 0x3C; printf("%d", (x & y) | (x ^ y) << 1);`
- `x` = 00001111, `y` = 00111100.
- `x & y` = 00001100 = 12. `x ^ y` = 00110011 = 51; `<< 1` = 102 (01100110).
- `<<` binds tighter than `|`: `12 | 102` = 00001100 | 01100110 = 01101110 = **110**.

## 6. Control flow and loop tracing

**`switch` falls through.** Execution enters at the matching label and continues down until `break` or the end. `default` may be anywhere; it is used only when no case matches, but falling through from it continues into the next label.

**Worked example 10.**
```c
int n = 2, s = 0;
switch (n) { case 1: s += 1; case 2: s += 2; case 3: s += 3; break; default: s += 100; }
```
Enter at `case 2`: s = 2, fall into `case 3`: s = 5, `break`. **s = 5**.

**Worked example 11 (default in the middle).**
```c
int s = 0;
switch (4) { case 1: s += 1; default: s += 10; case 2: s += 2; }
```
No match → jump to `default`: s = 10, falls into `case 2`: s = 12. **s = 12**.

**Worked example 12 (continue/break).**
```c
int i, s = 0;
for (i = 0; i < 10; i++) { if (i % 3 == 0) continue; if (i == 8) break; s += i; }
```
| i | action | s |
|---|---|---|
| 0 | continue | 0 |
| 1 | add | 1 |
| 2 | add | 3 |
| 3 | continue | 3 |
| 4 | add | 7 |
| 5 | add | 12 |
| 6 | continue | 12 |
| 7 | add | 19 |
| 8 | break | 19 |

Final **s = 19, i = 8**. (`continue` in a `for` still runs `i++`.)

**Worked example 13.** `int i = 5; while (i --> 0) printf("%d ", i);` There is no `-->` operator; it is `(i--) > 0`. Test uses the old value then decrements: prints **4 3 2 1 0**, and ends with `i = -1`.

**Other loop traps.** `for (i = 0; i < 5; i++);` (stray semicolon) runs an empty body; a following `printf("%d", i)` prints 5. A `do { } while (c);` body runs at least once. Dangling `else` binds to the nearest `if`.

## 7. `printf` formats and macros

| Spec | Meaning | Example (value) → output |
|---|---|---|
| `%d` `%i` | signed int | `-5` → `-5` |
| `%u` | unsigned | `-1` → `4294967295` |
| `%o` / `%x` | octal / hex | `65` → `101`; `255` → `ff` |
| `%c` | character | `65` → `A` |
| `%s` | string | |
| `%f` `%.2f` `%5.2f` | double | `3.14159` with `%5.2f` → ` 3.14` (width 5) |
| `%-5d` / `%05d` | left-justify / zero-pad | `42` → `42   ` / `00042` |
| `%zu` | `size_t` | |
| `%%` | literal percent | |

`printf` returns the number of characters printed: `printf("%d", printf("ab"));` prints `ab` first (inner call), then `2` → output **ab2**.

**Macros are text substitution** (no types, no evaluation until use).
```c
#define SQ(x) x*x
SQ(2+3)       // 2+3*2+3 = 11, not 25
SQ(3)/SQ(3)   // 3*3/3*3 = 9, not 1  (left-to-right: ((3*3)/3)*3)
#define MAX(a,b) ((a)>(b)?(a):(b))
int i = 3, j = 2, m = MAX(i++, j);
// expands to ((i++)>(j)?(i++):(j)): first i++ gives 3>2 true, i=4; sequence point at ?:;
// then (i++) yields 4, i=5.  m = 4, i = 5 (argument evaluated twice).
```
Fully parenthesised macro bodies and arguments avoid the first two problems; double evaluation remains.

## 8. Storage classes, scope and lifetime

| Class | Where stored | Initial value | Lifetime | Scope |
|---|---|---|---|---|
| `auto` (default local) | stack | **garbage** | block execution | block |
| `register` | register/stack hint | garbage | block | block; cannot take `&` |
| `static` local | data/BSS segment | **0** | whole program | block |
| global | data/BSS | 0 | whole program | file (and other files via `extern`) |
| `static` global/function | data/BSS | 0 | whole program | **this file only** |
| `extern` | declares a global defined elsewhere | | | |

**Scope** = where a name is visible (compile-time idea). **Lifetime** = how long the storage exists (run-time idea). A `static` local has block scope but program lifetime.

**Worked example 14 (static across calls; a GATE favourite).**
```c
int f(void) { static int c = 0; c++; return c; }
int g(void) { int c = 0; c++; return c; }
int a = f(), b = f(), d = f();   // 1, 2, 3
int e = g(), h = g();            // 1, 1
```
`static int c = 0` is initialised **once** at program start (the line is not re-executed); `g`'s `c` is recreated each call.

**Worked example 15 (shadowing).**
```c
int x = 10;
void f(void) { printf("%d ", x); }
int main(void) { int x = 20; f(); { int x = 30; printf("%d ", x); } printf("%d", x); }
```
`f` sees the global `x` (10). Inner block prints 30. After the block, `x` is 20. Output **10 30 20**.

### Static vs dynamic scoping

**Static (lexical) scoping:** a free variable in a function is resolved in the environment where the function was **written**. **Dynamic scoping:** resolved in the environment of the **caller chain at run time**. C is statically scoped; GATE asks what a program *would* print under dynamic scoping.

```c
int x = 1;               /* global */
void f() { printf("%d ", x); }
void g() { int x = 2; f(); }
int main() { int x = 3; g(); f(); }
```
| Call | Static scoping | Dynamic scoping |
|---|---|---|
| `f()` from `g` | looks at where `f` is defined → global `x` = 1 | most recent binding of `x` in call chain: `g`'s x = 2 |
| `f()` from `main` | 1 | `main`'s x = 3 |

Output: static **1 1**, dynamic **2 3**. Trick: draw the call stack; under dynamic scoping search it top-down for the name.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Sizes (LP64) | 1, 2, 4, 8, 8, 4, 8, ptr 8 | `sizeof` questions |
| Unsigned wrap | result mod 2^n | overflow, `u--` |
| `-1 > 1u` | true | mixed compare |
| Integer division | truncates toward 0; `a%b` has sign of `a` | negatives |
| `x<<n`, `x>>n` | x·2^n, floor(x/2^n) | bit ops |
| `x & (x-1)` | clears lowest set bit | popcount loop |
| `x & -x` | lowest set bit | Fenwick, bit tricks |
| Short-circuit | `0 && _`, `1 \|\| _` skip right | side-effect tracing |
| Static local | init once, persists | repeated calls |
| `i++` vs `++i` | old vs new value | expressions |
| Macro | text substitution | `SQ(a+b)` |

## GATE traps

- **Mixed signed/unsigned comparison:** the signed side converts; `-1 < 1u` is false.
- **`char` arithmetic is done in `int`:** `char c = 'a'; c + 1` prints 98 with `%d`, `b` with `%c`.
- **Division:** `float f = 1/2;` stores 0.0; `1/2.0` gives 0.5.
- **`=` vs `==`** in conditions; **stray `;`** after `if`/`for`/`while`.
- **`switch` without `break`** falls through; `default` position does not change when it runs.
- **UB expressions** (`i = i++`, `a[i] = i++`): do not attempt a "usual" answer.
- **Operator precedence:** `x & 1 == 0` is `x & (1 == 0)`; `*p++` is `*(p++)`; `a << 1 + 2` is `a << 3`.
- **Octal literals:** `010` is 8; `printf("%d", 010 + 0x10)` → 8 + 16 = 24.
- **Static locals** are *not* reset each call; **auto locals** are uninitialised (garbage), not 0.
- **`sizeof` does not evaluate** its operand; `sizeof(arr)` inside a function receiving `int arr[]` is a pointer size (see [pointers](pointers-arrays-strings.md)).
- **Macro arguments with side effects** are evaluated as many times as they appear.

## Connections

- [Number representation](../09-digital-logic/number-representation-and-arithmetic.md) — two's complement explains signed/unsigned reinterpretation, wrap-around and `~x = -x-1`.
- [Pointers, arrays, strings](pointers-arrays-strings.md) — `char` arithmetic, `*p++` precedence, `sizeof` on arrays.
- [Functions, recursion, structures](functions-recursion-structures.md) — static locals inside recursion; call stack and scope.
- [Runtime environments](../12-compiler-design/runtime-environments.md) — static vs dynamic scoping, activation records, storage layout.
- [Lexical analysis](../12-compiler-design/lexical-analysis.md) — why `i --> 0` tokenises as `i`, `--`, `>`, `0` (longest match / maximal munch).
- [Parsing](../12-compiler-design/parsing.md) — precedence and associativity are encoded in the expression grammar.
- [Instruction sets](../10-computer-organization/instruction-sets-and-addressing.md) — shifts/bitwise ops map to single instructions; endianness.

## Practice

**Q1 (MCQ, easy).** What does `printf("%d", -1 > 1u);` print?  (A) 0  (B) 1  (C) -1  (D) undefined

<details><summary>Answer</summary>

**Answer:** (B) 1  
**Solution:** `-1` converts to `unsigned int` (4294967295). `4294967295 > 1` is true, which prints as 1.

</details>

**Q2 (NAT).** After `int x = 0xF0F; int c = 0; while (x) { x &= x - 1; c++; }`, what is `c`?

<details><summary>Answer</summary>

**Answer:** 8  
**Solution:** The loop executes once per set bit. 0xF0F = 1111 0000 1111, which has 4 + 4 = 8 ones.

</details>

**Q3 (MCQ).** With `#define SQ(x) x*x` and `int a = 3;`, the value of `SQ(a+1)` is  (A) 16  (B) 7  (C) 10  (D) 13

<details><summary>Answer</summary>

**Answer:** (B) 7  
**Solution:** Expands to `a+1*a+1` = 3 + 3 + 1 = 7 (multiplication first).

</details>

**Q4 (MSQ).** Which of the following have undefined behaviour? (A) `i = i++;` (B) `unsigned u = UINT_MAX; u++;` (C) `int a = INT_MAX; a++;` (D) `int r = f() + g();` where both print (E) `int y = 1 << 32;` (32-bit `int`)

<details><summary>Answer</summary>

**Answer:** A, C, E  
**Solution:** A modifies `i` twice with no sequence point. C is signed overflow. E shifts by >= width. B is well-defined wrap to 0. D has unspecified (not undefined) evaluation order: either order is acceptable behaviour.

</details>

**Q5 (NAT).** Value of `s` at the end?
```c
int s = 0, i;
for (i = 1; i <= 6; i++) {
    switch (i % 3) { case 0: s += 10; case 1: s += 1; break; case 2: continue; }
    s += 100;
}
```

<details><summary>Answer</summary>

**Answer:** 424  
**Solution:** `continue` inside a `switch` applies to the enclosing loop; `break` only leaves the switch.
- i=1: `1%3=1` → s=1, break, then s += 100 → 101.
- i=2: `case 2: continue` → s stays 101.
- i=3: `0` → s=111, falls into case 1: s=112, break, += 100 → 212.
- i=4: 213, then 313.
- i=5: continue → 313.
- i=6: case 0 → 323, case 1 → 324, += 100 → **424**.

</details>

**Q6 (NAT).** What is printed (three calls)?
```c
void f(void) { static int s = 0; int a = 0; s += 2; a += 2; printf("%d,%d ", s, a); }
/* main calls f() three times */
```

<details><summary>Answer</summary>

**Answer:** `2,2 4,2 6,2`  
**Solution:** `s` is static (initialised once, 2, 4, 6); `a` is re-created as 0 each call, becoming 2.

</details>

**Q7 (MCQ).** In the static/dynamic scoping program of section 8 (`x=1` global, `g` defines `x=2` and calls `f`, `main` defines `x=3`), what does `main` print under dynamic scoping if `main` calls `g()` and then `f()`?  (A) 1 1  (B) 2 3  (C) 3 2  (D) 2 2

<details><summary>Answer</summary>

**Answer:** (B) 2 3  
**Solution:** `f` called from `g` finds the most recent `x` on the call stack = `g`'s 2. `f` called from `main` finds `main`'s 3.

</details>

**Q8 (NAT, harder).** `unsigned char c = 250; int i; for (i = 0; i < 10; i++) c++; printf("%d", c);` — what is printed?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** Ten increments give 260; stored in an `unsigned char`, 260 mod 256 = 4. `c` is promoted to `int` for `printf`, so `%d` shows 4.

</details>
