# Recurrences and generating functions

> **Paper:** CS · **Priority:** P0 · **Plan topics:** recurrence relations, generating functions
> **Prerequisites:** [Combinatorics](combinatorics.md) · **Leads to:** [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md), [Divide-and-conquer](../08-algorithms/divide-and-conquer.md)
> **PYQ weightage:** Medium · 8 Qs since 2015 (3 since 2021) · DA: 0 · Trend: steady · Most asked: Linear recurrences (characteristic roots), Generating functions (closed forms & coefficients)

## Quick glance

- **Exam focus:** Linear homogeneous and non-homogeneous recurrences (characteristic equations, roots, initial conditions) and ordinary generating functions (closed form representation, convolution, coefficient extraction). Usually tested as 1-mark or 2-mark MCQ/NAT questions.
- A **linear homogeneous recurrence with constant coefficients** has the form $a_n + c_1 a_{n-1} + \dots + c_k a_{n-k} = 0$. Its characteristic equation is $r^k + c_1 r^{k-1} + \dots + c_k = 0$.
- **Distinct roots** $r_1, r_2 \implies a_n = A r_1^n + B r_2^n$. **Repeated root** $r$ of multiplicity $m \implies a_n = (A_0 + A_1 n + \dots + A_{m-1} n^{m-1}) r^n$.
- For **non-homogeneous recurrences** $a_n - c a_{n-1} = f(n)$, the total solution is $a_n = a_n^{(h)} + a_n^{(p)}$. If $f(n) = s^n$ and $s$ is a root of the homogeneous equation of multiplicity $m$, the particular ansatz is $C n^m s^n$.
- **Always apply initial conditions to the complete general solution** ($a_n^{(h)} + a_n^{(p)}$), never to the homogeneous part alone.
- An **ordinary generating function (OGF)** encodes a sequence $\langle a_n \rangle$ as a formal power series: $G(x) = \sum_{n=0}^{\infty} a_n x^n$.
- Essential closed forms: $\frac{1}{1-x} = \sum_{n=0}^\infty x^n$, $\frac{1}{1-ax} = \sum_{n=0}^\infty a^n x^n$, $\frac{1}{(1-x)^2} = \sum_{n=0}^\infty (n+1)x^n$, $\frac{x}{(1-x)^2} = \sum_{n=0}^\infty n x^n$.
- **Coefficient extraction formula:** $[x^n] \frac{1}{(1-x)^k} = \binom{n + k - 1}{k - 1} = \binom{n + k - 1}{n}$.
- **Shifting rules:** $\sum_{n=1}^\infty a_{n-1} x^n = x G(x)$, whereas $\sum_{n=1}^\infty a_n x^n = G(x) - a_0$.

## 1. Linear homogeneous recurrence relations [PYQ: Medium]

A recurrence relation expresses the $n$-th term of a sequence $a_n$ as a function of earlier terms $a_{n-1}, a_{n-2}, \dots$ along with base cases (initial conditions).

A recurrence is **linear homogeneous with constant coefficients (LHRCC)** of order $k$ if it can be written as:
$$a_n + c_1 a_{n-1} + c_2 a_{n-2} + \dots + c_k a_{n-k} = 0, \quad c_k \ne 0$$

### Solving via the characteristic equation
Substitute the trial solution $a_n = r^n$ into the recurrence. Dividing through by $r^{n-k}$ yields the **characteristic equation**:
$$r^k + c_1 r^{k-1} + c_2 r^{k-2} + \dots + c_k = 0$$

The general solution depends on the nature of the roots:
1. **Distinct real roots** $r_1, r_2, \dots, r_k$:
   $$a_n = A_1 r_1^n + A_2 r_2^n + \dots + A_k r_k^n$$
2. **Repeated roots:** If a root $r$ has multiplicity $m$, its contribution to the solution is:
   $$(A_0 + A_1 n + A_2 n^2 + \dots + A_{m-1} n^{m-1}) r^n$$
3. **Complex conjugate roots** $r = \rho e^{\pm i\theta} = \rho (\cos\theta \pm i\sin\theta)$:
   $$a_n = \rho^n (A \cos(n\theta) + B \sin(n\theta))$$

The arbitrary constants $A_1, \dots, A_k$ are uniquely determined by solving the system of linear equations formed by the $k$ initial conditions $a_0, a_1, \dots, a_{k-1}$.

**Worked example 1 (distinct roots).** Solve $a_n - 7a_{n-1} + 10a_{n-2} = 0$ for $n \ge 2$, given initial values $a_0 = 1$ and $a_1 = 8$.

1. *Characteristic equation:*
   $$r^2 - 7r + 10 = 0 \implies (r - 2)(r - 5) = 0$$
   The roots are $r_1 = 2$ and $r_2 = 5$ (both real and distinct).
2. *General solution:*
   $$a_n = A \cdot 2^n + B \cdot 5^n$$
3. *Apply initial conditions:*
   - For $n = 0$: $a_0 = A(2^0) + B(5^0) = A + B = 1 \implies A = 1 - B$
   - For $n = 1$: $a_1 = A(2^1) + B(5^1) = 2A + 5B = 8$
   Substitute $A = 1 - B$:
   $$2(1 - B) + 5B = 8 \implies 2 + 3B = 8 \implies 3B = 6 \implies B = 2$$
   $$A = 1 - 2 = -1$$
4. *Closed form:*
   $$a_n = 2 \cdot 5^n - 2^n$$
   *Verification:* $a_2 = 2(25) - 4 = 46$. From recurrence: $a_2 = 7(8) - 10(1) = 56 - 10 = 46$ ✓.

**Worked example 2 (Fibonacci and Lucas sequences).** The standard Fibonacci recurrence is $F_n = F_{n-1} + F_{n-2}$ with $F_0 = 0, F_1 = 1$. Its characteristic equation is $r^2 - r - 1 = 0$, giving roots $\phi = \frac{1+\sqrt{5}}{2}$ and $\hat{\phi} = \frac{1-\sqrt{5}}{2}$, yielding Binet's formula $F_n = \frac{\phi^n - \hat{\phi}^n}{\sqrt{5}}$.

The **Lucas sequence** (GATE 2023) satisfies the identical recurrence $L_n = L_{n-1} + L_{n-2}$ but with initial values $L_0 = 2, L_1 = 1$. Solving constants gives $L_n = \phi^n + \hat{\phi}^n$. A critical identity connecting both sequences is $L_n = F_{n-1} + F_{n+1}$.

## 2. Linear non-homogeneous recurrences and particular solutions [PYQ: Medium]

When a forcing function $f(n) \ne 0$ appears on the right side:
$$a_n + c_1 a_{n-1} + \dots + c_k a_{n-k} = f(n)$$

The complete general solution is:
$$a_n = a_n^{(h)} + a_n^{(p)}$$
where $a_n^{(h)}$ is the solution to the associated homogeneous equation, and $a_n^{(p)}$ is any **particular solution** that satisfies the non-homogeneous equation.

### Trial forms for particular solutions

| Driving term $f(n)$ | Condition on root of char. eq. | Trial ansatz $a_n^{(p)}$ |
|---|---|---|
| Constant $c$ | $r = 1$ is not a root | $C$ |
| Constant $c$ | $r = 1$ is a root of multiplicity $m$ | $C n^m$ |
| Polynomial $P(n)$ of degree $d$ | $r = 1$ is not a root | $A_d n^d + \dots + A_1 n + A_0$ |
| Polynomial $P(n)$ of degree $d$ | $r = 1$ is a root of multiplicity $m$ | $n^m(A_d n^d + \dots + A_1 n + A_0)$ |
| $c \cdot s^n$ | $s$ is not a root of char. eq. | $C \cdot s^n$ |
| $c \cdot s^n$ | $s$ is a root of multiplicity $m$ | $C n^m \cdot s^n$ |

**Worked example 3 (non-homogeneous with exponential driving term).** Solve $a_n - 3a_{n-1} = 2^n$ for $n \ge 1$ with $a_0 = 3$.

1. *Homogeneous solution:*
   $r - 3 = 0 \implies r = 3 \implies a_n^{(h)} = A \cdot 3^n$.
2. *Particular solution:*
   $f(n) = 2^n$. The base $s = 2$ is not a characteristic root ($2 \ne 3$). Try $a_n^{(p)} = C \cdot 2^n$.
   Substitute into recurrence:
   $$C \cdot 2^n - 3(C \cdot 2^{n-1}) = 2^n$$
   Divide through by $2^{n-1}$:
   $$2C - 3C = 2 \implies -C = 2 \implies C = -2$$
   Hence, $a_n^{(p)} = -2 \cdot 2^n = -2^{n+1}$.
3. *General solution:*
   $$a_n = a_n^{(h)} + a_n^{(p)} = A \cdot 3^n - 2^{n+1}$$
4. *Initial condition:*
   $a_0 = 3 \implies A \cdot 3^0 - 2^1 = 3 \implies A - 2 = 3 \implies A = 5$.
5. *Final closed form:*
   $$a_n = 5 \cdot 3^n - 2^{n+1}$$
   *Check:* $a_1 = 5(3) - 2^2 = 15 - 4 = 11$. From recurrence: $a_1 = 3(3) + 2^1 = 11$ ✓.

## 3. Divide-and-conquer recurrences and unrolling [PYQ: High]

Divide-and-conquer recurrences express runtime or step counts in algorithms where an input of size $n$ is divided into $a$ subproblems of size $n/b$, with $f(n)$ non-recursive work.

### Master Theorem (Divide-and-Conquer)
For $T(n) = a T(n/b) + f(n)$ where $a \ge 1, b > 1$:
Let the watershed exponent be $c_{\text{crit}} = \log_b a$.
1. **Leaf-heavy:** If $f(n) = O(n^{c})$ with $c < \log_b a$, then $T(n) = \Theta(n^{\log_b a})$.
2. **Balanced:** If $f(n) = \Theta(n^{\log_b a} \log^k n)$ for $k \ge 0$, then $T(n) = \Theta(n^{\log_b a} \log^{k+1} n)$.
3. **Root-heavy:** If $f(n) = \Omega(n^{c})$ with $c > \log_b a$, and regularity condition $a f(n/b) \le c' f(n)$ holds for some $c' < 1$, then $T(n) = \Theta(f(n))$.

### Unrolling (Repeated substitution)
When the Master theorem does not apply (e.g., unequal partition sizes, non-polynomial forcing functions, or step sizes $n - 1$), expand the recurrence by repeated substitution:

**Worked example 4 (unrolling non-standard recurrence).** Solve $T(n) = 2T(n-1) + 1$ with $T(0) = 0$.
- Step 1: $T(n) = 2T(n-1) + 1$
- Step 2: $T(n) = 2[2T(n-2) + 1] + 1 = 2^2 T(n-2) + 2 + 1$
- Step 3: $T(n) = 2^3 T(n-3) + 2^2 + 2 + 1$
- Pattern after $k$ steps: $T(n) = 2^k T(n-k) + \sum_{i=0}^{k-1} 2^i$
- Base case reached when $n - k = 0 \implies k = n$:
  $$T(n) = 2^n T(0) + \sum_{i=0}^{n-1} 2^i = 2^n(0) + (2^n - 1) = 2^n - 1$$

## 4. Ordinary generating functions (OGF) and dictionary of closed forms [PYQ: Medium]

An **ordinary generating function (OGF)** transforms a discrete sequence $\langle a_0, a_1, a_2, \dots \rangle$ into a continuous algebraic function $G(x)$ of a dummy variable $x$:
$$G(x) = \sum_{n=0}^{\infty} a_n x^n = a_0 + a_1 x + a_2 x^2 + a_3 x^3 + \dots$$

The variable $x$ acts purely as a placeholder ("clothesline" on which sequence values hang). Convergence is treated formally; algebraic operations on power series mirror operations on sequences.

### Standard generating function dictionary

| Sequence $\langle a_n \rangle_{n \ge 0}$ | Values $(a_0, a_1, a_2, \dots)$ | Closed form $G(x)$ |
|---|---|---|
| Constant 1 | $1, 1, 1, 1, \dots$ | $\frac{1}{1-x}$ |
| Alternating | $1, -1, 1, -1, \dots$ | $\frac{1}{1+x}$ |
| Geometric progression $a^n$ | $1, a, a^2, a^3, \dots$ | $\frac{1}{1-ax}$ |
| Linear terms $n + 1$ | $1, 2, 3, 4, \dots$ | $\frac{1}{(1-x)^2}$ |
| Index $n$ | $0, 1, 2, 3, \dots$ | $\frac{x}{(1-x)^2}$ |
| Squares $n^2$ | $0, 1, 4, 9, \dots$ | $\frac{x(1+x)}{(1-x)^3}$ |
| Repeated combinations $\binom{n+k-1}{k-1}$ | $1, k, \binom{k+1}{2}, \dots$ | $\frac{1}{(1-x)^k}$ |
| Finite binomial $\binom{m}{n}$ | $\binom{m}{0}, \binom{m}{1}, \dots, \binom{m}{m}, 0, \dots$ | $(1+x)^m$ |

### Operations on generating functions
1. **Linearity:** If $a_n = c_1 b_n + c_2 c_n$, then $G_a(x) = c_1 G_b(x) + c_2 G_c(x)$.
2. **Right shift (delay):** Multiplying by $x^k$ shifts sequence right by $k$ positions:
   $$x^k G(x) = \sum_{n=k}^\infty a_{n-k} x^n \longleftrightarrow \langle \underbrace{0, \dots, 0}_{k \text{ zeros}}, a_0, a_1, \dots \rangle$$
3. **Left shift (advance):**
   $$\frac{G(x) - a_0}{x} = \sum_{n=0}^\infty a_{n+1} x^n \longleftrightarrow \langle a_1, a_2, a_3, \dots \rangle$$
4. **Differentiation:**
   $$x \frac{d}{dx} G(x) = \sum_{n=0}^\infty n a_n x^n \longleftrightarrow \langle 0, a_1, 2a_2, 3a_3, \dots \rangle$$
5. **Convolution (multiplication of series):**
   $$A(x) \cdot B(x) = \sum_{n=0}^\infty c_n x^n \quad \text{where } c_n = \sum_{k=0}^n a_k b_{n-k}$$

**Worked example 5 (deriving $n^2$ closed form).**
Start with $\sum_{n=0}^\infty x^n = \frac{1}{1-x}$. Differentiate with respect to $x$:
$$\sum_{n=1}^\infty n x^{n-1} = \frac{1}{(1-x)^2} \implies \sum_{n=0}^\infty n x^n = \frac{x}{(1-x)^2}$$
Differentiate once more:
$$\frac{d}{dx}\left[\frac{x}{(1-x)^2}\right] = \frac{(1-x)^2(1) - x \cdot 2(1-x)(-1)}{(1-x)^4} = \frac{(1-x) + 2x}{(1-x)^3} = \frac{1+x}{(1-x)^3}$$
Multiply by $x$:
$$\sum_{n=0}^\infty n^2 x^n = x \cdot \frac{1+x}{(1-x)^3} = \frac{x(1+x)}{(1-x)^3}$$

## 5. Coefficient extraction and combinatorics via generating functions [PYQ: Medium]

The notation $[x^n] G(x)$ denotes the coefficient of $x^n$ in the power series expansion of $G(x)$.

The fundamental combinatorial extraction formula is:
$$[x^n] \frac{1}{(1-x)^k} = [x^n] (1-x)^{-k} = \binom{n + k - 1}{k - 1} = \binom{n + k - 1}{n}$$

This corresponds to the number of non-negative integer solutions to $x_1 + x_2 + \dots + x_k = n$.

**Worked example 6 (GATE style coefficient extraction).** Find the coefficient of $x^{12}$ in $(x^2 + x^3 + x^4 + \dots)^3$.

1. *Factor out common powers of $x$:*
   $$x^2 + x^3 + x^4 + \dots = x^2 (1 + x + x^2 + \dots) = x^2 \cdot \frac{1}{1-x}$$
2. *Cube the expression:*
   $$\left(x^2 \cdot \frac{1}{1-x}\right)^3 = x^6 \cdot \frac{1}{(1-x)^3}$$
3. *Extract coefficient:*
   $$[x^{12}] \left[ x^6 \cdot \frac{1}{(1-x)^3} \right] = [x^{12 - 6}] \frac{1}{(1-x)^3} = [x^6] \frac{1}{(1-x)^3}$$
4. *Apply the negative binomial formula:*
   Here $n = 6$ and $k = 3$:
   $$[x^6] \frac{1}{(1-x)^3} = \binom{6 + 3 - 1}{3 - 1} = \binom{8}{2} = \frac{8 \times 7}{2 \times 1} = 28$$
   *Answer:* **28**.

## 6. Solving recurrences using generating functions [PYQ: Low]

Generating functions provide a mechanical 5-step method to solve any linear recurrence:
1. Multiply the recurrence equation by $x^n$.
2. Sum over all indices $n$ for which the recurrence holds (typically $n \ge k$).
3. Rewrite the sums in terms of the generating function $G(x) = \sum_{n=0}^\infty a_n x^n$.
4. Solve the algebraic equation for $G(x)$.
5. Decompose $G(x)$ into partial fractions and extract $[x^n] G(x)$ to get the closed form for $a_n$.

**Worked example 7 (solving recurrence via OGF).** Solve $a_n = 2a_{n-1} + 1$ for $n \ge 1$, with initial condition $a_0 = 0$.

1. *Multiply by $x^n$ and sum from $n=1$ to $\infty$:*
   $$\sum_{n=1}^\infty a_n x^n = 2 \sum_{n=1}^\infty a_{n-1} x^n + \sum_{n=1}^\infty x^n$$
2. *Substitute generating function expressions:*
   - Left side: $\sum_{n=1}^\infty a_n x^n = G(x) - a_0 = G(x)$ (since $a_0 = 0$).
   - First term right: $2 \sum_{n=1}^\infty a_{n-1} x^n = 2x \sum_{n=1}^\infty a_{n-1} x^{n-1} = 2x G(x)$.
   - Second term right: $\sum_{n=1}^\infty x^n = \frac{x}{1-x}$.
3. *Algebraic equation for $G(x)$:*
   $$G(x) = 2x G(x) + \frac{x}{1-x} \implies G(x)(1 - 2x) = \frac{x}{1-x}$$
   $$G(x) = \frac{x}{(1-x)(1-2x)}$$
4. *Partial fraction decomposition:*
   $$\frac{x}{(1-x)(1-2x)} = \frac{A}{1-x} + \frac{B}{1-2x}$$
   $$x = A(1 - 2x) + B(1 - x)$$
   - Let $x = 1$: $1 = A(1 - 2) \implies A = -1$.
   - Let $x = 1/2$: $1/2 = B(1 - 1/2) \implies 1/2 = B/2 \implies B = 1$.
   $$G(x) = \frac{1}{1-2x} - \frac{1}{1-x}$$
5. *Extract general term:*
   $$a_n = [x^n] G(x) = [x^n] \sum_{n=0}^\infty 2^n x^n - [x^n] \sum_{n=0}^\infty x^n = 2^n - 1$$
   *Check:* $a_0 = 2^0 - 1 = 0$; $a_1 = 2^1 - 1 = 1$; $a_2 = 2^2 - 1 = 3 = 2(1) + 1$ ✓.

## Formulas and facts to memorise

| Item | Formula / fact | When to use |
|---|---|---|
| Characteristic equation | $r^k + c_1 r^{k-1} + \dots + c_k = 0$ | Finding roots of homogeneous linear recurrence |
| Distinct roots | $a_n = \sum_{i=1}^k A_i r_i^n$ | Linear recurrence when all characteristic roots are distinct |
| Root of multiplicity $m$ | $(A_0 + A_1 n + \dots + A_{m-1} n^{m-1}) r^n$ | Linear recurrence when root $r$ repeats $m$ times |
| Complete solution | $a_n = a_n^{(h)} + a_n^{(p)}$ | Non-homogeneous linear recurrences |
| Resonant particular ansatz | $C n^m s^n$ | Driving term $s^n$ where $s$ is char root of multiplicity $m$ |
| Master theorem watershed | Compare $f(n)$ with $n^{\log_b a}$ | Asymptotics of $T(n) = aT(n/b) + f(n)$ |
| Standard geometric OGF | $\frac{1}{1-ax} = \sum_{n=0}^\infty a^n x^n$ | Sequence $a_n = a^n$ |
| Negative binomial extraction | $[x^n] \frac{1}{(1-x)^k} = \binom{n+k-1}{k-1}$ | Extracting coefficients from powers of $(1-x)^{-1}$ |
| Index shift relation | $\sum_{n=1}^\infty a_{n-1} x^n = x G(x)$ | Setting up generating function equations from recurrences |
| Sequence evaluation at $x=1/2$ | $G(1/2) = \sum_{n=0}^\infty \frac{a_n}{2^n}$ | GATE questions asking for sum of series $\sum a_n / 2^n$ |

## GATE traps

- **Applying initial conditions too early:** Substituting $a_0, a_1$ into the homogeneous solution $a_n^{(h)}$ *before* adding the particular solution $a_n^{(p)}$ will produce completely wrong constants. Always write $a_n = a_n^{(h)} + a_n^{(p)}$ first.
- **Root resonance oversight:** If the non-homogeneous driving term is $f(n) = 3^n$, and $3$ is also a root of the characteristic equation with multiplicity 1, testing $C \cdot 3^n$ will yield $0 = 3^n$ (collapse). The correct guess must be $C n \cdot 3^n$.
- **Shifting mismatch:** Remember that $\sum_{n=1}^\infty a_n x^n = G(x) - a_0$, but $\sum_{n=1}^\infty a_{n-1} x^n = x G(x)$. Confusing these indices introduces erroneous constant offsets into $G(x)$.
- **Binomial coefficient bounds:** In $\binom{n+k-1}{k-1}$, $n$ is the power of $x$ being extracted and $k$ is the exponent of $(1-x)$ in the denominator. Writing $\binom{n+k-1}{k}$ instead of $\binom{n+k-1}{k-1}$ produces an off-by-one combination error.
- **Divide-and-conquer applicability:** Master theorem cannot be applied if the subproblem size reduction is subtractive ($T(n-1)$) or if $f(n)$ is not polynomially bounded relative to $n^{\log_b a}$ (e.g., $T(n) = 2T(n/2) + n/\log n$).

## Connections

- [Combinatorics](combinatorics.md) — Generating functions algebraically model combinations with repetition and integer partitions.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — Master theorem and characteristic roots determine algorithm time-complexity growth rates.
- [Divide-and-conquer](../08-algorithms/divide-and-conquer.md) — Solves execution time recurrences for algorithms such as Merge Sort, Binary Search, and Strassen's algorithm.
- [Dynamic programming](../08-algorithms/dynamic-programming.md) — Recurrence relations define overlapping subproblems, memoised to avoid exponential recomputation.

## Practice

**Q1 (MCQ, Easy).** Consider the recurrence relation $a_n - 4a_{n-1} + 4a_{n-2} = 0$ for $n \ge 2$, with initial conditions $a_0 = 1$ and $a_1 = 4$. The value of $a_4$ is:
(A) 32  
(B) 48  
(C) 80  
(D) 64  

<details><summary>Answer</summary>

**Answer:** C  
**Solution:**
The characteristic equation is:
$$r^2 - 4r + 4 = 0 \implies (r - 2)^2 = 0$$
There is a repeated root $r = 2$ with multiplicity 2.
The general solution has the form:
$$a_n = (A + B n) 2^n$$
Using initial conditions:
- $n = 0$: $a_0 = (A + 0) 2^0 = A = 1$.
- $n = 1$: $a_1 = (1 + B) 2^1 = 2(1 + B) = 4 \implies 1 + B = 2 \implies B = 1$.
Thus, the closed form is:
$$a_n = (1 + n) 2^n$$
For $n = 4$:
$$a_4 = (1 + 4) 2^4 = 5 \times 16 = 80$$
</details>

**Q2 (NAT, Easy).** The coefficient of $x^5$ in the power series expansion of $(1 - x)^{-3}$ is ___________.

<details><summary>Answer</summary>

**Answer:** 21  
**Solution:**
Using the negative binomial expansion:
$$[x^n] (1 - x)^{-k} = \binom{n + k - 1}{k - 1}$$
Here $n = 5$ and $k = 3$:
$$[x^5] (1 - x)^{-3} = \binom{5 + 3 - 1}{3 - 1} = \binom{7}{2} = \frac{7 \times 6}{2 \times 1} = 21$$
</details>

**Q3 (MCQ, Medium).** Which one of the following is the closed form expression for the ordinary generating function of the sequence $a_n = 3^n + 2^n$ for $n \ge 0$?
(A) $\frac{2 - 5x}{(1 - 3x)(1 - 2x)}$  
(B) $\frac{1}{(1 - 3x)(1 - 2x)}$  
(C) $\frac{2 - 5x}{(1 + 3x)(1 + 2x)}$  
(D) $\frac{5x}{(1 - 3x)(1 - 2x)}$  

<details><summary>Answer</summary>

**Answer:** A  
**Solution:**
By linearity of ordinary generating functions:
$$G(x) = \sum_{n=0}^\infty a_n x^n = \sum_{n=0}^\infty 3^n x^n + \sum_{n=0}^\infty 2^n x^n$$
Using $\sum_{n=0}^\infty a^n x^n = \frac{1}{1 - ax}$:
$$G(x) = \frac{1}{1 - 3x} + \frac{1}{1 - 2x} = \frac{(1 - 2x) + (1 - 3x)}{(1 - 3x)(1 - 2x)} = \frac{2 - 5x}{(1 - 3x)(1 - 2x)}$$
</details>

**Q4 (NAT, Medium).** Consider the recurrence relation $a_n = 2a_{n-1} + 3$ for $n \ge 1$ with $a_0 = 2$. The value of $a_6$ is ___________.

<details><summary>Answer</summary>

**Answer:** 317  
**Solution:**
1. *Homogeneous part:* $r - 2 = 0 \implies a_n^{(h)} = A \cdot 2^n$.
2. *Particular part:* $f(n) = 3$ (constant). Since $r = 1$ is not a root of $r - 2 = 0$, try constant ansatz $a_n^{(p)} = C$:
   $$C = 2C + 3 \implies -C = 3 \implies C = -3$$
3. *General solution:* $a_n = A \cdot 2^n - 3$.
4. *Initial condition:* $a_0 = 2 \implies A \cdot 2^0 - 3 = 2 \implies A - 3 = 2 \implies A = 5$.
   Thus, $a_n = 5 \cdot 2^n - 3$.
5. For $n = 6$:
   $$a_6 = 5 \cdot 2^6 - 3 = 5(64) - 3 = 320 - 3 = 317$$
*Verification step-by-step:*
$a_0 = 2$  
$a_1 = 2(2) + 3 = 7$  
$a_2 = 2(7) + 3 = 17$  
$a_3 = 2(17) + 3 = 37$  
$a_4 = 2(37) + 3 = 77$  
$a_5 = 2(77) + 3 = 157$  
$a_6 = 2(157) + 3 = 317$ ✓.
</details>

**Q5 (MSQ, Medium).** Let $F_n$ denote the $n$-th Fibonacci number ($F_0 = 0, F_1 = 1, F_n = F_{n-1} + F_{n-2}$) and $L_n$ denote the $n$-th Lucas number ($L_0 = 2, L_1 = 1, L_n = L_{n-1} + L_{n-2}$). Which of the following statements is/are TRUE?
(A) The characteristic equation for both $F_n$ and $L_n$ is $r^2 - r - 1 = 0$  
(B) For all $n \ge 1$, $L_n = F_{n-1} + F_{n+1}$  
(C) $L_4 = 7$  
(D) Both sequences grow asymptotically as $\Theta(\phi^n)$ where $\phi = \frac{1 + \sqrt{5}}{2}$  

<details><summary>Answer</summary>

**Answer:** A, B, C, D  
**Solution:**
- (A) is TRUE: Both sequences share the exact same recurrence relation $x_n = x_{n-1} + x_{n-2}$, so their characteristic equation is $r^2 - r - 1 = 0$.
- (B) is TRUE: For $n=1$, $L_1 = 1$ and $F_0 + F_2 = 0 + 1 = 1$. For $n=2$, $L_2 = 3$ and $F_1 + F_3 = 1 + 2 = 3$. By induction, $L_n = F_{n-1} + F_{n+1}$ holds for all $n \ge 1$.
- (C) is TRUE: $L_0 = 2, L_1 = 1, L_2 = 3, L_3 = 4, L_4 = 7$.
- (D) is TRUE: The dominant characteristic root for both sequences is $\phi = \frac{1 + \sqrt{5}}{2} \approx 1.618$. Because the constant coefficient corresponding to $\phi^n$ is non-zero in both cases, both grow as $\Theta(\phi^n)$.
</details>

**Q6 (NAT, Hard).** The number of non-negative integer solutions to the equation $x_1 + x_2 + x_3 = 10$ subject to the constraints $x_1 \ge 1$, $x_2 \ge 2$, and $x_3 \ge 0$ is ___________.

<details><summary>Answer</summary>

**Answer:** 36  
**Solution:**
Using generating functions:
- $x_1 \ge 1 \implies (x^1 + x^2 + x^3 + \dots) = x(1 + x + x^2 + \dots) = \frac{x}{1-x}$
- $x_2 \ge 2 \implies (x^2 + x^3 + x^4 + \dots) = x^2(1 + x + x^2 + \dots) = \frac{x^2}{1-x}$
- $x_3 \ge 0 \implies (1 + x + x^2 + \dots) = \frac{1}{1-x}$

The product generating function is:
$$G(x) = \left(\frac{x}{1-x}\right) \left(\frac{x^2}{1-x}\right) \left(\frac{1}{1-x}\right) = \frac{x^3}{(1-x)^3}$$
We need $[x^{10}] G(x)$:
$$[x^{10}] \frac{x^3}{(1-x)^3} = [x^{10 - 3}] \frac{1}{(1-x)^3} = [x^7] \frac{1}{(1-x)^3}$$
Using $[x^n] (1-x)^{-k} = \binom{n + k - 1}{k - 1}$ with $n = 7$ and $k = 3$:
$$\binom{7 + 3 - 1}{3 - 1} = \binom{9}{2} = \frac{9 \times 8}{2 \times 1} = 36$$

*(Equivalently by substitution: let $y_1 = x_1 - 1 \ge 0, y_2 = x_2 - 2 \ge 0, y_3 = x_3 \ge 0$. Then $y_1 + y_2 + y_3 = 10 - 1 - 2 = 7$. Stars and bars gives $\binom{7 + 3 - 1}{3 - 1} = \binom{9}{2} = 36$.)*
</details>

**Q7 (NAT, Hard).** Consider the recurrence relation $a_n - 2a_{n-1} = 2^n$ for $n \ge 1$, with $a_0 = 1$. The value of $a_5$ is ___________.

<details><summary>Answer</summary>

**Answer:** 192  
**Solution:**
1. *Homogeneous part:* $r - 2 = 0 \implies r = 2 \implies a_n^{(h)} = A \cdot 2^n$.
2. *Particular part:* Driving term is $f(n) = 2^n = 1 \cdot 2^n$. Notice that $s = 2$ is already a root of the homogeneous equation with multiplicity 1!
   Therefore, trial ansatz must be:
   $$a_n^{(p)} = C \cdot n \cdot 2^n$$
   Substitute into recurrence:
   $$C n \cdot 2^n - 2 \cdot C (n - 1) \cdot 2^{n-1} = 2^n$$
   $$C n \cdot 2^n - C (n - 1) \cdot 2^n = 2^n$$
   Divide through by $2^n$:
   $$C n - C(n - 1) = 1 \implies C = 1$$
   Thus, $a_n^{(p)} = n \cdot 2^n$.
3. *General solution:*
   $$a_n = A \cdot 2^n + n \cdot 2^n = (A + n) 2^n$$
4. *Initial condition:*
   $a_0 = 1 \implies (A + 0) 2^0 = 1 \implies A = 1$.
   Hence:
   $$a_n = (1 + n) 2^n$$
5. For $n = 5$:
   $$a_5 = (1 + 5) 2^5 = 6 \times 32 = 192$$
*Verification by iteration:*
$a_0 = 1$  
$a_1 = 2(1) + 2^1 = 2 + 2 = 4 = (1 + 1)2^1$  
$a_2 = 2(4) + 2^2 = 8 + 4 = 12 = (1 + 2)2^2$  
$a_3 = 2(12) + 2^3 = 24 + 8 = 32 = (1 + 3)2^3$  
$a_4 = 2(32) + 2^4 = 64 + 16 = 80 = (1 + 4)2^4$  
$a_5 = 2(80) + 2^5 = 160 + 32 = 192$ ✓.
</details>
