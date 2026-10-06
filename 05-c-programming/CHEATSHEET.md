# C Programming: Cheat sheet

Assumed model: LP64, little-endian, two's complement. Full chapters: [basics](c-basics-and-expressions.md) · [pointers](pointers-arrays-strings.md) · [functions/recursion/structs](functions-recursion-structures.md).

## Types and conversions

| Type | Bytes | Notes |
|---|---|---|
| `char` / `short` / `int` / `long` / `long long` | 1 / 2 / 4 / 8 / 8 | `unsigned int` max 4294967295 |
| `float` / `double` / pointer | 4 / 8 / 8 | pointer size independent of pointee |
| `'A'` | int 65 | `'a'-'A'` = 32, `'7'-'0'` = 7 |
| Literals | `010`=8, `0x1F`=31, `10u`, `3.5` is double | |

| Rule | Consequence |
|---|---|
| `char`/`short` promote to `int` | `char a=100, b=100; a+b` = 200 |
| int meets unsigned: int becomes unsigned | `-1 > 1u` true; `sizeof(int) > -1` false |
| Unsigned overflow wraps mod 2^n | `unsigned char 250 + 10` stored = 4 |
| Signed overflow, `1<<32`, `x/0` | undefined |
| Integer `/` truncates to 0; `%` sign = dividend | `-7/2=-3`, `-7%2=-1` |
| `1/2*4.0` = 0.0; `(float)5/2` = 2.5 | int division happens before conversion |

## Operators

Precedence high to low: `() [] -> . x++ x--` > `! ~ ++x --x * & sizeof (cast)` (R to L) > `* / %` > `+ -` > `<< >>` > `< <= > >=` > `== !=` > `&` > `^` > `|` > `&&` > `||` > `?:` (R to L) > `= op=` (R to L) > `,`.

| Fact | Example |
|---|---|
| `x & 1 == 0` is `x & (1==0)` | parenthesise |
| `&&`, `\|\|`, `?:`, `,` are sequence points; `&&`/`\|\|` short-circuit | `a-- > 0 \|\| b++` skips `b++` |
| UB: modify twice / read+modify between sequence points | `i=i++`, `a[i]=i++`, `f(i++,i++)` |
| Unspecified (not UB): operand/argument order | `f()+g()` |
| `x & (x-1)` clears lowest set bit; `x & -x` isolates it | popcount loop |
| `~x = -x-1`; `x<<n = x*2^n`; `x>>n = floor(x/2^n)` | |
| Macro = text | `SQ(a+1)` with `x*x` is `a+1*a+1` |
| `sizeof` does not evaluate operand | `sizeof(i++)` leaves `i` |
| `printf` returns chars printed | `printf("%d", printf("ab"))` gives `ab2` |

Control: `switch` falls through, `default` may be anywhere, `continue` inside switch affects the loop, `i --> 0` is `(i--) > 0`.

## Storage and scope

| Kind | Init | Lifetime |
|---|---|---|
| auto local | garbage | block |
| static local | 0, once | program |
| global | 0 | program |
| `static` global | 0 | program, file scope |

Static (lexical) scoping: free variable resolved where the function is **defined** (C). Dynamic: resolved on the **call chain**.

## Pointers and arrays

| Item | Rule |
|---|---|
| `p+k` | `+k*sizeof(*p)` bytes; `p-q` = elements |
| `a[i]` | `*(a+i)` = `i[a]` |
| decay | not under `sizeof`, `&`, string-literal init |
| `a` vs `&a` | `a+1` one element; `&a+1` whole array |
| `*(&a+1) - a` | array length |
| 2D | `&m[i][j] = base + (i*COLS+j)*s`; `m[i][j] = *(*(m+i)+j)` |
| `*p++`, `(*p)++`, `++*p` | pointer moves; pointee+1 (old); pointee+1 (new) |
| `int *a[3]` / `int (*a)[3]` | array of pointers / pointer to array |
| `const int *p` / `int *const p` | pointee fixed / pointer fixed |
| `char s[]="abc"` vs `char *p="abc"` | sizeof 4 vs 8; `p[0]=..` UB |
| strings | `sizeof` = `strlen`+1; `==` compares addresses |
| `strcmp` | `<0, 0, >0`, not exactly -1/+1 |
| Little-endian | `int 0x12345678`: byte 0 is `0x78` |

Bugs: dangling (return `&local`, use after `free`), double free, leak, uninitialised pointer, out-of-bounds, write to literal.

## Functions, recursion, structs

| Item | Rule |
|---|---|
| C parameter passing | by value only; pointers simulate reference |
| Modes | value / reference / value-result (copy back at return) / name (re-evaluate expression at each use) |
| Recursion cost | stack O(depth) |
| fib calls | `C(n)=C(n-1)+C(n-2)+1 = 2Fib(n+1)-1`: 1,1,3,5,9,15,25 |
| `h(n)=h(n-1)+h(n-1)` | value 2^n, calls 2^(n+1)-1 |
| Hanoi | 2^n - 1 moves |
| Print before call | descending; after call | ascending |
| Static in recursion | one shared copy |
| Struct size | pad each member to its alignment, round total to max alignment |
| Union size | max member, rounded to alignment |
| `p->m` | `(*p).m`; `p->y++` increments member |
| `malloc` / `calloc` / `realloc` / `free` | uninit / zeroed / may move / no auto NULL |
| Self-referential struct | holds pointer, never itself |

| Struct | sizeof |
|---|---|
| `{char; int; char}` | 12 |
| `{int; char; char}` | 8 |
| `{char; double; int}` | 24 |
| `{int; char*}` | 16 |
