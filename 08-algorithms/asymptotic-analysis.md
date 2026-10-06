# Asymptotic Analysis and Recurrences

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Asymptotic notation and growth rates; Worst-case time complexity; Worst-case space complexity
> **Prerequisites:** [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) · [Complexity reference](../07-data-structures/complexity-reference.md) · **Leads to:** [Searching and sorting](searching-and-sorting.md) · [Divide and conquer](divide-and-conquer.md)

## Quick glance

- **Big-O** is an upper bound, **Omega** a lower bound, **Theta** a tight bound (both). $f=\Theta(g)$ iff $f=O(g)$ and $f=\Omega(g)$.
- Growth ladder: $1 < \log^* n < \log\log n < \log n < \sqrt n < n < n\log n < n^2 < n^3 < 2^n < n! < n^n$.
- Compare two functions by taking logs (or the limit of the ratio $f/g$): limit $0\Rightarrow f=o(g)$; finite non-zero $\Rightarrow\Theta$; $\infty\Rightarrow f=\omega(g)$.
- Sum rule: $O(f)+O(g)=O(\max(f,g))$. Product rule: $O(f)\cdot O(g)=O(fg)$.
- A loop that doubles (or halves) its variable runs $\Theta(\log n)$ times; a loop $j=i,2i,3i,\dots\le n$ runs $n/i$ times, giving the harmonic sum $\Theta(n\log n)$.
- **Master theorem** for $T(n)=aT(n/b)+f(n)$: compare $f(n)$ with $n^{\log_b a}$. Smaller $\Rightarrow\Theta(n^{\log_b a})$; equal $\Rightarrow\Theta(n^{\log_b a}\log n)$; larger (plus regularity) $\Rightarrow\Theta(f)$.
- Extended case: $f=\Theta(n^{\log_b a}\log^k n)$ gives $\Theta(n^{\log_b a}\log^{k+1}n)$ for $k\ge0$.
- Space complexity of recursion = **depth of the recursion stack** (times frame size) plus auxiliary arrays.
- **#1 trap:** "worst-case" is about the input, "O" is about the bound. You can state a best-case $\Omega$, worst-case $O$, or any case with any notation.

## 1. What asymptotic analysis measures

An algorithm's running time depends on the machine, so we count **primitive steps** (comparisons, assignments, arithmetic) as a function $T(n)$ of the input size $n$. We then ignore constant factors and small inputs and look only at how $T(n)$ grows when $n\to\infty$.

Three common "cases" for the same algorithm:

| Case | Meaning | Example (linear search for $x$ in $n$ items) |
|---|---|---|
| Best case | Input of size $n$ that makes it fastest | $x$ is first: $1$ comparison |
| Worst case | Input of size $n$ that makes it slowest | $x$ absent or last: $n$ comparisons |
| Average case | Expected cost over a stated input distribution | $x$ equally likely anywhere: $(n+1)/2$ |

**GATE asks almost always about worst-case time unless told otherwise.** The input size $n$ is the number of elements, except when the input is a number (then size is the number of bits, $\log n$ — see the traps).

## 2. The notations, with constants

Let $f,g:\mathbb N\to\mathbb R_{\ge0}$.

| Notation | Reads | Definition | Analogy |
|---|---|---|---|
| $f=O(g)$ | $f$ grows no faster than $g$ | $\exists c>0,n_0$: $f(n)\le c\,g(n)\ \forall n\ge n_0$ | $\le$ |
| $f=\Omega(g)$ | no slower than $g$ | $\exists c>0,n_0$: $f(n)\ge c\,g(n)\ \forall n\ge n_0$ | $\ge$ |
| $f=\Theta(g)$ | same rate | $\exists c_1,c_2>0,n_0$: $c_1g\le f\le c_2g$ | $=$ |
| $f=o(g)$ | strictly slower | $\forall c>0\ \exists n_0$: $f(n)<c\,g(n)$ | $<$ |
| $f=\omega(g)$ | strictly faster | $\forall c>0\ \exists n_0$: $f(n)>c\,g(n)$ | $>$ |

The little notations use "for **every** constant $c$"; the big ones use "there **exists** a constant $c$". Equivalent limit tests (when the limit exists), with $L=\lim_{n\to\infty} f(n)/g(n)$:

- $L=0\Rightarrow f=o(g)$ (so $f=O(g)$, not $\Omega(g)$)
- $0<L<\infty\Rightarrow f=\Theta(g)$
- $L=\infty\Rightarrow f=\omega(g)$

**Worked example 1 — finding constants.** Show $3n^2+10n=\Theta(n^2)$.
- Upper: for $n\ge10$, $10n\le n^2$, so $3n^2+10n\le4n^2$. Take $c_2=4,n_0=10$.
- Lower: $3n^2+10n\ge3n^2$ always. Take $c_1=3$.
- Hence $3n^2\le f\le4n^2$ for $n\ge10$.

**Worked example 2 — a false claim.** Is $2^{2n}=O(2^n)$? Suppose $2^{2n}\le c\,2^n$. Then $2^n\le c$ for all large $n$, impossible. **False.** But $2^{n+1}=O(2^n)$ is **true** ($2^{n+1}=2\cdot2^n$, $c=2$). A constant in the exponent's *additive* part is a constant factor; a constant *multiplier* of $n$ in the exponent is not.

### Properties

| Property | Statement |
|---|---|
| Reflexive | $f=\Theta(f)$ (also for $O$, $\Omega$) |
| Transitive | $f=O(g),g=O(h)\Rightarrow f=O(h)$ (same for $\Omega,\Theta,o,\omega$) |
| Symmetric | only $\Theta$: $f=\Theta(g)\iff g=\Theta(f)$ |
| Transpose symmetry | $f=O(g)\iff g=\Omega(f)$; $f=o(g)\iff g=\omega(f)$ |
| Max rule | $f+g=\Theta(\max(f,g))$ |
| Product | $f_1=O(g_1),f_2=O(g_2)\Rightarrow f_1f_2=O(g_1g_2)$ |
| Not total | Some pairs are incomparable, e.g. $n$ and $n^{1+\sin n}$ |
| Constants | $O(c\,f)=O(f)$; $\log_a n=\Theta(\log_b n)$ for fixed bases |

**Beware:** $f=O(g)\not\Rightarrow 2^{f}=O(2^{g})$. Counter: $f=2n,g=n$ gives $4^n$ vs $2^n$. Similarly $f=O(g)\not\Rightarrow \log f=\ldots$ is safe only when $g\to\infty$ and logs of constants are ignored.

## 3. The growth-rate ladder

From slowest to fastest:

```text
1  <  log* n  <  log log n  <  log n  <  (log n)^k  <  n^(1/2)  <  n  <  n log n
   <  n^2  <  n^3  <  n^k  <  2^n  <  3^n  <  n!  <  n^n  <  2^(2^n)
```

Facts that settle most GATE comparisons:

- Any polylog $(\log n)^k$ is $o(n^\epsilon)$ for every $\epsilon>0$. So $\log^{100}n=o(\sqrt n)$.
- Any polynomial is $o(2^{\epsilon n})$ for every $\epsilon>0$. So $n^{1000}=o(1.0001^n)$.
- $n!=\omega(2^n)$ and $n!=o(n^n)$. Stirling: $n!\approx\sqrt{2\pi n}(n/e)^n$, so $\log(n!)=\Theta(n\log n)$.
- $\log^* n$ (iterated log: how many times you apply $\log_2$ to reach $\le1$) is below every other entry; $\log^*(2^{65536})=5$.
- $n^{\log n}$ sits **between** polynomials and $2^n$: $n^{k}\ll n^{\log n}\ll 2^{n^\epsilon}$.

### Comparing by taking logs

When both functions are exponentials or powers, compare $\log f$ and $\log g$ (log is monotone increasing).

**Worked example 3.** Order $f_1=n^{\log n}$, $f_2=2^{\sqrt{n}}$, $f_3=n^{10}$, $f_4=(\log n)^{\log n}$.
- Take $\log_2$: $\log f_1=(\log n)^2$; $\log f_2=\sqrt n$; $\log f_3=10\log n$; $\log f_4=\log n\cdot\log\log n$.
- Order of the logs: $10\log n\ <\ \log n\log\log n\ <\ (\log n)^2\ <\ \sqrt n$.
- Hence $f_3<f_4<f_1<f_2$.

**Worked example 4.** Is $2^{n}$ vs $n^{\log n}$? $\log(2^n)=n$ and $\log(n^{\log n})=\log^2n$. Since $n=\omega(\log^2 n)$, $2^n=\omega(n^{\log n})$.

**Worked example 5 (a trap).** $\log(n!)=\Theta(n\log n)$ vs $\log(2^n)=n$, so $n!=\omega(2^n)$ (the logs differ by an $\omega$ gap, which is safe). But $\log(n^n)=n\log n=\Theta(\log n!)$ does **not** make $n^n=\Theta(n!)$: in fact $n^n/n!\approx e^n\to\infty$. **If the logs are $\omega$-separated the functions are; if the logs are merely $\Theta$-equal, nothing follows.**

## 4. Counting loop iterations

Turn the code into a sum, then simplify.

| Pattern | Iterations | Result |
|---|---|---|
| `for (i=1;i<=n;i++)` | $n$ | $\Theta(n)$ |
| `for (i=1;i<=n;i*=2)` | $\lfloor\log_2 n\rfloor+1$ | $\Theta(\log n)$ |
| `for (i=n;i>=1;i/=2)` | $\lfloor\log_2 n\rfloor+1$ | $\Theta(\log n)$ |
| `for (i=2;i<=n;i=i*i)` | $\lceil\log_2\log_2 n\rceil$ | $\Theta(\log\log n)$ |
| nested independent | product | $n\cdot m$ |
| dependent `j<=i` | $\sum_{i=1}^n i=\frac{n(n+1)}2$ | $\Theta(n^2)$ |
| `j=i; j<=n; j+=i` | $\sum_i \lfloor n/i\rfloor\approx n\ln n$ | $\Theta(n\log n)$ |
| `i*=2` outer, inner runs `i` | $1+2+4+\dots+n\approx2n$ | $\Theta(n)$ |

**Worked example 6 — logarithmic inner loop.**
```c
for (i = 1; i <= n; i++)
    for (j = 1; j <= n; j *= 2)
        count++;
```
Inner loop runs $\lfloor\log_2 n\rfloor+1$ times for each of $n$ values of $i$. For $n=16$: $16\times5=80$ (checked by running). **Answer: $\Theta(n\log n)$.**

**Worked example 7 — dependent loops.**
```c
for (i = 1; i <= n; i++)
    for (j = 1; j <= i; j++)
        count++;
```
$\sum_{i=1}^n i=n(n+1)/2$; for $n=10$ this is $55$. $\Theta(n^2)$.

**Worked example 8 — harmonic sum.**
```c
for (i = 1; i <= n; i++)
    for (j = i; j <= n; j += i)
        count++;
```
Inner loop runs $\lfloor n/i\rfloor$ times. Total $=\sum_{i=1}^n\lfloor n/i\rfloor\approx n H_n\approx n\ln n$. For $n=16$ it is exactly $50$ (and $16\cdot H_{16}\approx54$). $\Theta(n\log n)$.

**Worked example 9 — doubling outer loop with linear inner.**
```c
for (i = 1; i < n; i *= 2)
    for (j = 0; j < i; j++)
        count++;
```
Iterations $1+2+4+\dots+2^{k}$ with $2^k<n$: for $n=64$ this is $1+2+4+8+16+32=63$; for $n=1024$, $1023$. Geometric series, $\Theta(n)$ — **not** $n\log n$.

**Worked example 10 — the classic.**
```c
for (i = n/2; i <= n; i++)
  for (j = 1; j <= n/2; j++)
    for (k = 1; k <= n; k *= 2)
      count++;
```
$(n/2+1)\cdot(n/2)\cdot(\lfloor\log_2 n\rfloor+1)=\Theta(n^2\log n)$.

**Recursive code** is analysed by writing a recurrence (Section 5). Cost of `if (cond) A else B` is $\max$ of the branches in the worst case.

## 5. Solving recurrences

A recurrence expresses $T(n)$ via smaller inputs. Three tools.

### 5.1 Substitution (guess and verify by induction)

**Worked example 11.** $T(n)=2T(n/2)+n$, $T(1)=1$. Guess $T(n)\le c\,n\log n$. Inductive step: $T(n)\le2c\frac n2\log\frac n2+n=cn\log n-cn+n\le cn\log n$ when $c\ge1$. So $T(n)=O(n\log n)$.

**Subtle trap:** for $T(n)=2T(n/2)+1$, guess $T(n)\le cn$: $2c\frac n2+1=cn+1\not\le cn$ — induction fails even though the claim is true. Strengthen the hypothesis to $T(n)\le cn-d$.

### 5.2 Recursion tree

Draw the tree, sum the work per level, sum over levels.

**Worked example 12.** $T(n)=T(n/3)+T(2n/3)+n$.
- Every level of the tree does at most $n$ work (sizes add up to $n$).
- Longest path follows the $2/3$ branch: depth $\log_{3/2}n$.
- Total $\le n\log_{3/2}n=O(n\log n)$; shortest path (the $1/3$ branch) has depth $\log_3 n$ with full levels, giving $\Omega(n\log n)$. **$\Theta(n\log n)$.** (A numeric run with $n=3^{12}$ gives $T/(n\log_2n)\approx0.53$, stable.)

### 5.3 Master theorem

For $T(n)=aT(n/b)+f(n)$ with constants $a\ge1,b>1$ and $f$ asymptotically positive. Let $p=\log_b a$ (the exponent of the number of leaves $n^{p}$).

| Case | Condition | Result |
|---|---|---|
| 1 | $f(n)=O(n^{p-\epsilon})$, some $\epsilon>0$ ($f$ polynomially smaller) | $T=\Theta(n^{p})$ |
| 2 | $f(n)=\Theta(n^{p})$ | $T=\Theta(n^{p}\log n)$ |
| 2 (extended) | $f(n)=\Theta(n^{p}\log^{k}n)$, $k\ge0$ | $T=\Theta(n^{p}\log^{k+1}n)$ |
| 3 | $f(n)=\Omega(n^{p+\epsilon})$ **and** $af(n/b)\le c f(n)$ for some $c<1$ (regularity) | $T=\Theta(f(n))$ |

Intuition: the leaves cost $n^p$ in total, the root costs $f(n)$; whichever is bigger (polynomially) dominates; if equal, every level costs the same and there are $\log n$ levels.

**Worked example 13 — all three cases.**

| Recurrence | $a,b$ | $n^p$ | Compare $f$ | Case | Answer |
|---|---|---|---|---|---|
| $T=9T(n/3)+n$ | 9,3 | $n^2$ | $n\ll n^2$ | 1 | $\Theta(n^2)$ |
| $T=2T(n/2)+n$ | 2,2 | $n$ | $n=n$ | 2 | $\Theta(n\log n)$ |
| $T=3T(n/2)+n^2$ | 3,2 | $n^{1.585}$ | $n^2\gg$ | 3 | $\Theta(n^2)$ |
| $T=4T(n/2)+n^2\log n$ | 4,2 | $n^2$ | $n^2\log^1n$ | 2 ext. $k=1$ | $\Theta(n^2\log^2 n)$ |
| $T=T(n/2)+1$ | 1,2 | $n^0=1$ | $1=1$ | 2 | $\Theta(\log n)$ |
| $T=7T(n/2)+n^2$ | 7,2 | $n^{2.807}$ | $n^2\ll$ | 1 | $\Theta(n^{\log_27})$ |
| $T=2T(n/2)+n/\log n$ | 2,2 | $n$ | $f=n\log^{-1}n$ | **gap** | not Master; tree gives $\Theta(n\log\log n)$ |

**Worked example 14 — cases where the theorem does not apply.**
- $T(n)=2^nT(n/2)+n^n$: $a$ is not a constant.
- $T(n)=0.5\,T(n/2)+n$: $a<1$.
- $T(n)=2T(n/2)+n\log n$: ratio $f/n^p=\log n$ is smaller than any $n^\epsilon$, so case 3 fails; but the **extended** case 2 with $k=1$ applies: $\Theta(n\log^2n)$.
- $T(n)=2T(n/2)+n/\log n$: neither. Level $i$ costs $n/\log(n/2^i)$; summing over $i=0..\log n-1$ gives $n(1/\log n+\dots+1/1)=n H_{\log n}=\Theta(n\log\log n)$.
- $T(n)=T(n/2)+n(2-\cos n)$: $f$ oscillates, regularity fails; answer is still $\Theta(n)$ by direct bounding (since $n\le f\le3n$ and geometric sum).

### 5.4 Non-divide-and-conquer recurrences

| Recurrence | Solution | Why |
|---|---|---|
| $T(n)=T(n-1)+1$ | $\Theta(n)$ | unroll: $n$ ones |
| $T(n)=T(n-1)+n$ | $\Theta(n^2)$ | $n+(n-1)+\dots+1$ |
| $T(n)=T(n-1)+\log n$ | $\Theta(n\log n)$ | $\log n!$ |
| $T(n)=2T(n-1)+1$ | $\Theta(2^n)$ | $T=2^n-1$ if $T(0)=0$ |
| $T(n)=T(n-1)+T(n-2)$ | $\Theta(\phi^n)$, $\phi=1.618$ | Fibonacci, naive recursion |
| $T(n)=T(n/2)+T(n/4)+n$ | $\Theta(n)$ | $1/2+1/4<1$: root dominates |
| $T(n)=T(\alpha n)+T((1-\alpha)n)+n$ | $\Theta(n\log n)$ | each level costs $n$ |

### 5.5 Change of variable

**Worked example 15.** $T(n)=2T(\sqrt n)+\log n$.
- Let $m=\log_2 n$, so $n=2^m$ and $\sqrt n=2^{m/2}$.
- Define $S(m)=T(2^m)$. Then $S(m)=2S(m/2)+m$.
- Master case 2: $S(m)=\Theta(m\log m)$.
- Back-substitute: $T(n)=\Theta(\log n\cdot\log\log n)$.

**Worked example 16.** $T(n)=T(\sqrt n)+1$: $S(m)=S(m/2)+1\Rightarrow S=\Theta(\log m)$, so $T(n)=\Theta(\log\log n)$.

## 6. Space complexity

**Space complexity** counts memory used by the algorithm beyond the input (auxiliary), as a function of $n$.

- **Iterative, fixed variables:** $O(1)$ (in-place: selection, bubble, insertion, heap sort).
- **Extra array of size $n$:** $O(n)$ (merge sort's merge buffer, counting sort's output).
- **Recursion:** each active call occupies a stack frame. Space $=$ (max depth) $\times$ (frame size) $+$ heap memory.

| Algorithm | Recursion depth | Stack space |
|---|---|---|
| Binary search (recursive) | $\log n$ | $O(\log n)$ |
| Merge sort | $\log n$ | $O(\log n)$ stack $+O(n)$ buffer |
| Quicksort, balanced | $\log n$ | $O(\log n)$ |
| Quicksort, worst case | $n$ | $O(n)$ |
| DFS (recursive) on $V$ vertices | up to $V$ | $O(V)$ |
| Naive Fibonacci `fib(n-1)+fib(n-2)` | $n$ | $O(n)$ (time is $\phi^n$, space only $n$) |
| Tail-recursive with TCO | 1 | $O(1)$ |

**Worked example 17.** `int f(int n){ if(n<=1) return 1; return f(n/2)+f(n/2); }` Time: $T=2T(n/2)+1\Rightarrow\Theta(n)$. Space: depth is $\log n$ (the two calls run **one after the other**, so only one root-to-leaf chain is live), hence $\Theta(\log n)$.

**Quicksort worst-case space fix:** recurse on the smaller half and loop on the larger; depth is then $\le\log_2 n$ always.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| $O,\Omega,\Theta$ | $\le,\ge,=$ up to constants for large $n$ | classify any bound |
| Limit test | $\lim f/g=0,\ c,\ \infty$ | quick comparison |
| $\sum_{i=1}^n i$ | $n(n+1)/2$ | dependent loops |
| $\sum_{i=1}^n 1/i=H_n$ | $\ln n+0.577\dots$ | harmonic loops |
| $\sum_{i=0}^{k}2^i$ | $2^{k+1}-1$ | doubling loops |
| $\log(n!)$ | $\Theta(n\log n)$ | decision-tree bounds |
| Master theorem | $n^{\log_ba}$ vs $f(n)$ | divide and conquer |
| Extended case 2 | $f=n^p\log^kn\Rightarrow n^p\log^{k+1}n$ | merge-sort-like with extra logs |
| $\log_b a$ values | $\log_23=1.585$, $\log_27=2.807$ | Karatsuba, Strassen |
| Space of recursion | depth $\times$ frame | stack questions |

## GATE traps

- **$O$ is not "worst case".** Binary search best case is $O(1)$ and is also $\Omega(1)$; "linear search is $O(n)$" is a bound, not the case.
- **Exponent constants matter, additive terms don't.** $2^{n+1}=\Theta(2^n)$ but $2^{2n}\ne O(2^n)$; $(n+1)^2=\Theta(n^2)$ but $(2n)!\ne O(n!)$ in any useful way.
- **Taking logs of $\Theta$:** $\log f=\Theta(\log g)$ does *not* imply $f=\Theta(g)$ ($n^n$ vs $n!$, or $n^2$ vs $n^3$ both have $\Theta(\log n)$ logs).
- **Master theorem case 3 needs regularity,** and **case 1/3 need a *polynomial* gap.** $f=n\log n$ vs $n^p=n$ is *not* case 3; use the extended case 2.
- **Input that is a number:** testing primality by trial division up to $\sqrt N$ is $O(\sqrt N)$ in value but $\Theta(2^{b/2})$ in the bit length $b=\log N$ — exponential in input size.
- **Space of a recursion is depth, not number of calls.** Naive Fibonacci uses $O(n)$ space, not $O(2^n)$.
- **Nested-loop counts:** `i *= 2` outer with inner `j < i` is $\Theta(n)$ (geometric), not $n\log n$. Check whether the inner bound depends on the outer variable.
- **$\log$ bases** never matter inside $\Theta$, but **do** matter in exponents: $2^{\log_2 n}=n$ while $2^{\log_4 n}=\sqrt n$.
- **Transitivity holds** for $O$, $\Omega$, $\Theta$; **symmetry** holds only for $\Theta$. A statement "$f=O(g)$ and $g=O(f)$" means $f=\Theta(g)$.
- **Loose bounds are still true:** $n=O(n^2)$ holds, so "is $f=O(g)$?" asks only for an upper bound, while "is $f=\Theta(g)$?" asks for tightness.

## Connections

- [Recurrences and generating functions](../01-discrete-mathematics/recurrences-and-generating-functions.md) — the same recurrence-solving techniques (characteristic roots, unrolling) in exact form.
- [Complexity reference](../07-data-structures/complexity-reference.md) — the costs of data-structure operations that you plug into loop analysis.
- [Searching and sorting](searching-and-sorting.md) — every sort's time and space is derived with the tools here.
- [Divide and conquer](divide-and-conquer.md) — Master theorem applied to merge sort, Strassen, Karatsuba.
- [Dynamic programming](dynamic-programming.md) — table size and per-cell cost give the DP complexity.
- [Functions-recursion-structures](../05-c-programming/functions-recursion-structures.md) — stack frames behind recursion space.
- [Graph traversals](graph-traversals.md) — $O(V+E)$ vs $O(V^2)$ analysis for adjacency list vs matrix.

## Practice

**Q1 (MCQ).** Which is true? (A) $n^2=O(n)$ (B) $n=\Omega(n^2)$ (C) $2^{n+1}=\Theta(2^n)$ (D) $2^{2n}=\Theta(2^n)$

<details><summary>Answer</summary>

**Answer:** (C)  
**Solution:** $2^{n+1}=2\cdot2^n$, a constant factor, so $\Theta(2^n)$. (D) fails since $2^{2n}/2^n=2^n\to\infty$. (A),(B) are reversed inequalities.

</details>

**Q2 (MCQ).** Arrange in increasing order of growth: $f_1=n^{\sqrt n}$, $f_2=2^n$, $f_3=n^{10}\cdot 2^{n/2}$, $f_4=n!$.

<details><summary>Answer</summary>

**Answer:** $f_1<f_3<f_2<f_4$  
**Solution:** Take $\log_2$: $f_1:\sqrt n\log n$; $f_3:\frac n2+10\log n$; $f_2:n$; $f_4:\Theta(n\log n)$. Since $\sqrt n\log n=o(n/2)$, $f_1<f_3$. The exponents $n/2+10\log n$ and $n$ differ by $\omega(1)$, so $f_3<f_2$. Finally $n=o(n\log n)$ gives $f_2<f_4$.

</details>

**Q3 (NAT).** How many times is `count++` executed for $n=64$?
```c
for (i = 1; i < n; i *= 2)
    for (j = 0; j < i; j++)
        count++;
```

<details><summary>Answer</summary>

**Answer:** 63  
**Solution:** $i\in\{1,2,4,8,16,32\}$ (64 is excluded because `i < n`). Sum $=1+2+4+8+16+32=63$.

</details>

**Q4 (MCQ).** Solve $T(n)=4T(n/2)+n^2\log n$.

<details><summary>Answer</summary>

**Answer:** $\Theta(n^2\log^2n)$  
**Solution:** $n^{\log_24}=n^2$, $f=n^2\log^1n$, so extended case 2 with $k=1$ gives $n^2\log^{2}n$. (Plain case 3 fails: no polynomial gap.)

</details>

**Q5 (MCQ).** $T(n)=2T(\sqrt n)+1$. What is $T(n)$?

<details><summary>Answer</summary>

**Answer:** $\Theta(\log n)$  
**Solution:** $m=\log n$, $S(m)=2S(m/2)+1\Rightarrow\Theta(m)$ (case 1, $p=1$, $f=1=O(m^{0})$). So $T=\Theta(\log n)$.

</details>

**Q6 (MSQ).** Which are $\Theta(n\log n)$?  (A) $\sum_{i=1}^n\lfloor n/i\rfloor$ (B) $\log(n!)$ (C) $T(n)=T(n/3)+T(2n/3)+n$ (D) $T(n)=T(n-1)+\log n$

<details><summary>Answer</summary>

**Answer:** A, B, C, D (all four)  
**Solution:** (A) harmonic: $nH_n$. (B) Stirling. (C) every level costs $\le n$ and $\Theta(\log n)$ levels. (D) unrolls to $\sum\log i=\log n!$.

</details>

**Q7 (NAT).** Function `g(n)`: `if (n<=1) return; g(n/2); g(n/2); for(i=0;i<n;i++) ;` — what is the maximum recursion stack depth for $n=1024$ (counting the initial call as depth 1)?

<details><summary>Answer</summary>

**Answer:** 11  
**Solution:** Calls with $n=1024,512,\dots,2,1$: that is $\log_21024+1=11$ frames live at the deepest point (the two child calls are sequential).

</details>

**Q8 (MCQ).** For which condition is $f(n)=\Theta(g(n))$ guaranteed? (A) $\log f=\Theta(\log g)$ (B) $f=O(g)$ and $g=O(f)$ (C) $f=o(g)$ (D) $f/g\to\infty$

<details><summary>Answer</summary>

**Answer:** (B)  
**Solution:** Mutual $O$ is the definition of $\Theta$. (A) is false ($n^2,n^3$). (C),(D) give strict separation.

</details>
