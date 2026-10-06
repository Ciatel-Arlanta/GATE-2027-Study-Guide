# Algebraic structures: monoids and groups

> **Paper:** CS · **Priority:** P1 · **Plan topics:** binary operations, semigroups, monoids, groups, subgroups
> **Prerequisites:** [Sets, relations, functions](sets-relations-functions.md) · **Leads to:** [Propositional logic](propositional-logic.md)

## Quick glance
- A binary operation on S maps every pair in $S\times S$ back into S (closure).
- Semigroup: associative operation. Monoid: semigroup plus identity. Group: monoid plus inverse for every element.
- Commutativity is an extra property; groups need not be abelian.
- Under addition, integers form a group with identity 0 and inverse −a; positive integers do not.

## 1. Properties of an operation
For operation $*$, test closure, associativity $(a*b)*c=a*(b*c)$, identity $e*a=a*e=a$, inverse $a^{-1}*a=e$, and commutativity $a*b=b*a$ separately.

**Worked example.** Natural numbers including 0 under addition are closed and associative; 0 is identity. But 3 has no natural additive inverse. Therefore this is a monoid, not a group.

## 2. Groups and examples
Integers modulo n under addition form an abelian group: identity is 0, and inverse of $a$ is $(-a)\bmod n$. Nonzero residues modulo prime p under multiplication also form a group because every nonzero residue has a multiplicative inverse.

Under matrix multiplication, invertible $n\times n$ matrices form a group: identity is $I$, inverse exists by definition, and multiplication is associative. It is generally non-abelian: $AB$ may differ from $BA$.

## GATE traps
- Associativity does not imply commutativity.
- Identity must work on both sides unless one-sided identity is explicitly asked.
- A group under multiplication must exclude 0 in ordinary number examples.
- Closure is about staying inside the given set, not merely being defined numerically.

## Connections
- [Posets and lattices](posets-and-lattices.md) — algebraic laws also describe meet/join operations.
- [Boolean algebra](../09-digital-logic/boolean-algebra-and-minimization.md) — logic operations obey algebraic identities.

## Practice
**Q1.** Is $(\mathbb Z,+)$ a group? Give identity and inverse.
<details><summary>Answer</summary> Yes; identity 0, inverse of a is −a.</details>

**Q2.** Is positive integers under addition a monoid?
<details><summary>Answer</summary> No identity exists within positive integers (0 is absent).</details>

**Q3.** Is matrix multiplication commutative in general?
<details><summary>Answer</summary> No. For example, $A=\begin{bmatrix}1&1\\0&1\end{bmatrix}$ and $B=\begin{bmatrix}1&0\\1&1\end{bmatrix}$ yield $AB\ne BA$.</details>
