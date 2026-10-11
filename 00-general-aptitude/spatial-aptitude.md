# Spatial Aptitude: 2D/3D Transformations, Patterns, and Visual Reasoning

> **Paper:** CS+DA · **Priority:** P2 · **Plan topics:** translation, rotation, scaling, mirroring, assembling, paper folding/cutting, 2D/3D patterns
> **Prerequisites:** none · **Leads to:** [Quantitative aptitude](quantitative-aptitude.md), [Matrices and determinants](../03-linear-algebra/matrices-and-determinants.md)
> **PYQ weightage:** High · 19 Qs since 2015 (19 since 2021) · DA: 3 · Trend: rising (new in the 2021 syllabus) · Most asked: Paper Folding & Cutting, 2D Patterns & Mirror/Rotation, 3D Patterns & Cube Nets, Assembling Pieces

## Quick glance

- **Exam focus:** Introduced into GATE in 2021, spatial aptitude contributes 1–3 steady questions (1-mark and 2-mark MCQs/NATs) per paper across CS and DA; questions test paper folding/cutting, cube net folding, 2D/3D rotations, mirror reflection symmetry, and counting figures.
- **2D rotations about origin:** Counter-clockwise by $90^\circ$: $(x, y) \mapsto (-y, x)$; by $180^\circ$: $(x, y) \mapsto (-x, -y)$; by $270^\circ$ (or $90^\circ$ clockwise): $(x, y) \mapsto (y, -x)$.
- **Reflections (mirroring):** Across $x$-axis: $(x, -y)$; across $y$-axis: $(-x, y)$; across line $y = x$: $(y, x)$; across line $y = -x$: $(-y, -x)$.
- **Chirality / Handedness:** Pure in-plane rotations preserve cyclic vertex order (clockwise remains clockwise); reflections **reverse handedness**. An asymmetric shape cannot be matched to its mirror reflection by any 2D rotation.
- **Paper unfolding rule:** Unfold in reverse chronological order (LIFO). Each crease line acts as a mirror line of reflection for all interior cuts and holes.
- **Holes on fold lines:** An interior punch away from creases yields $2^k$ holes after $k$ independent folds. A punch centered directly on a fold line merges across the seam and yields half as many separate openings.
- **Cube nets:** 6 square faces fold into a closed cube. In any valid net, **opposite faces never share a common edge or vertex**. In a 4-square strip, alternate squares are opposite.
- **Painted cube of side $n$:** Total unit cubes $= n^3$. 3 faces painted $= 8$ (corners); 2 faces painted $= 12(n - 2)$ (edges); 1 face painted $= 6(n - 2)^2$ (face centers); unpainted $= (n - 2)^3$ (interior core).
- **Grid shape formulas:** In an $m \times n$ grid of unit squares, total squares $= \sum_{k=1}^{\min(m,n)} (m - k + 1)(n - k + 1)$; total rectangles $= \binom{m+1}{2}\binom{n+1}{2}$.
- **#1 trap:** Assuming rotation and reflection commute. In general, $R_\theta \circ M \ne M \circ R_\theta$. Applying a reflection after a rotation yields a different orientation than reflecting first.

## 1. 2D Coordinate Transformations: Rotation, Reflection, and Scaling [PYQ: High]

Spatial transformations manipulate 2D shapes on a Cartesian plane through rigid motions (distance-preserving) and scaling.

```text
       Original Shape           90° CCW Rotation            Mirror Across y-axis
            ▲                         ◄──┐                        ▲
            │                            │                        │
            └──►                         ▼                        ◄──┘
     Handedness: Right            Handedness: Right          Handedness: Left (Flipped)
```

### 1.1 Transformation Matrices and Mappings
Let $(x, y)$ denote the coordinates of a vertex before transformation:

| Transformation | Mathematical Mapping | Matrix Operator | Determinant |
|---|---|---|---|
| **Translation** by $(t_x, t_y)$ | $(x + t_x, y + t_y)$ | Affine vector addition | $+1$ |
| **Scaling** by factor $k$ | $(kx, ky)$ | $\begin{bmatrix} k & 0 \\ 0 & k \end{bmatrix}$ | $k^2$ |
| **Reflection** across $y$-axis | $(-x, y)$ | $\begin{bmatrix} -1 & 0 \\ 0 & 1 \end{bmatrix}$ | $-1$ |
| **Reflection** across $x$-axis | $(x, -y)$ | $\begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$ | $-1$ |
| **Reflection** across line $y = x$ | $(y, x)$ | $\begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}$ | $-1$ |
| **Rotation** by $\theta$ (CCW) | $(x\cos\theta - y\sin\theta, x\sin\theta + y\cos\theta)$ | $\begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$ | $+1$ |

### 1.2 Handedness and Invariance
- Proper rigid motions (translations and rotations) have determinant $+1$ and **preserve chirality**.
- Reflections have determinant $-1$ and **reverse chirality**.
- If a 2D figure has an axis of symmetry, reflecting across that axis leaves the figure invariant.

**Worked example 1 (Composition of rotation and reflection).** Consider the triangle with vertices $A(1, 2), B(4, 2), C(1, 5)$.
1. Apply a $90^\circ$ counter-clockwise rotation about the origin to obtain $A_1, B_1, C_1$.
2. Then reflect the resulting triangle across the $y$-axis to obtain $A_2, B_2, C_2$.
3. Compare the result with reflecting first across the $y$-axis and then rotating $90^\circ$ counter-clockwise.

*Step 1: Rotate $90^\circ$ CCW using $(x, y) \mapsto (-y, x)$.*
- $A_1 = (-2, 1)$
- $B_1 = (-2, 4)$
- $C_1 = (-5, 1)$

*Step 2: Reflect across the $y$-axis using $(x, y) \mapsto (-x, y)$.*
- $A_2 = (-(-2), 1) = (2, 1)$
- $B_2 = (-(-2), 4) = (2, 4)$
- $C_2 = (-(-5), 1) = (5, 1)$

*Step 3: Reverse the operation order (Reflect first, then Rotate).*
- Reflect $A, B, C$ across $y$-axis:
  $A' = (-1, 2), B' = (-4, 2), C' = (-1, 5)$
- Rotate $A', B', C'$ by $90^\circ$ CCW:
  $A'' = (-2, -1), B'' = (-2, -4), C'' = (-5, -1)$

*Observation:* Notice that $A_2(2, 1) \ne A''(-2, -1)$. The two operations **do not commute**: order of application is critical.

## 2. Paper Folding and Paper Cutting [PYQ: Medium]

Paper folding problems test mental visualization of reflection symmetry across folding axes.

```text
       Step 1: Unfolded Sheet      Step 2: Fold Along Crease      Step 3: Punched Hole
          ┌────────┬────────┐             ┌────────┐                   ┌────────┐
          │        │        │    Fold     │        │      Punch        │  ●     │
          │        │        │   ─────►    │        │     ──────►       │        │
          │        │        │   Right     │        │                   │        │
          └────────┴────────┘             └────────┘                   └────────┘
                                            Crease
```

### 2.1 Principles of Unfolding
1. **LIFO Stack Inversion:** If paper was folded along crease 1, then crease 2, unfold crease 2 first, then crease 1.
2. **Mirror Reflection across Crease:** When unfolding along line $L$, duplicate every hole and shape by reflecting across $L$.
3. **Crease Overlap Rule:**
   - A hole in the interior of a $k$-folded sheet yields $2^k$ distinct holes.
   - A punch lying centered on 1 fold line yields $2^{k-1}$ holes.
   - A punch on the corner intersection of 2 perpendicular fold lines yields 1 single hole at the center of the unfolded paper.

### 2.2 Sheet Dimensions and Perimeters
When an $L \times W$ rectangular paper is folded along its line of symmetry perpendicular to the longer edge:
- The longer edge is halved: $\max(L, W) \to \frac{\max(L, W)}{2}$.
- The other edge remains unchanged.

**Worked example 2 (Three successive folds and perimeter).** A rectangular sheet of paper measuring $32\text{ cm} \times 20\text{ cm}$ is folded in half three times. Each fold is made along the axis of symmetry perpendicular to the longer edge of the current rectangular shape. Find the perimeter of the final folded rectangular sheet.

*Step 1: First fold.*
- Current dimensions: $32\text{ cm} \times 20\text{ cm}$. Longer edge is 32 cm.
- Crease is perpendicular to the 32 cm edge.
- New dimensions: $\frac{32}{2} = 16\text{ cm}$ by $20\text{ cm}$.

*Step 2: Second fold.*
- Current dimensions: $16\text{ cm} \times 20\text{ cm}$. Longer edge is 20 cm.
- Crease is perpendicular to the 20 cm edge.
- New dimensions: $16\text{ cm}$ by $\frac{20}{2} = 10\text{ cm}$.

*Step 3: Third fold.*
- Current dimensions: $16\text{ cm} \times 10\text{ cm}$. Longer edge is 16 cm.
- Crease is perpendicular to the 16 cm edge.
- New dimensions: $\frac{16}{2} = 8\text{ cm}$ by $10\text{ cm}$.

*Step 4: Compute final perimeter.*
$$\text{Perimeter} = 2 \times (\text{Length} + \text{Width}) = 2 \times (8 + 10) = 2 \times 18 = 36\text{ cm}$$

## 3. 3D Structures, Cube Nets, and Painted Cubes [PYQ: Medium]

### 3.1 Cube Nets
A cube net consists of 6 connected squares in a plane that fold into a 3D cube.

```text
             Standard 1-4-1 Cube Net:
                     ┌───┐
                     │ 1 │               Opposite face pairs:
                 ┌───┼───┼───┬───┐       • 2 is opposite 4
                 │ 2 │ 3 │ 4 │ 5 │       • 3 is opposite 5
                 └───┼───┼───┴───┘       • 1 is opposite 6
                     │ 6 │
                     └───┘
```

**Key rules of cube nets:**
- **Alternate Face Rule:** In any continuous straight row or column of squares, alternate squares (separated by exactly one square) fold into opposite faces (e.g., face 2 and face 4; face 3 and face 5).
- **No Shared Edge:** Opposite faces can never share an edge or vertex in the net or in the folded 3D cube.
- **Corner Joint Rule:** Edges meeting at an internal $90^\circ$ reflex angle zip together to form an edge in the 3D cube.

### 3.2 Painted Cubes Cut into Unit Cubes
A large solid cube of side $n$ painted on all six faces and cut into $n^3$ identical unit cubes ($n \ge 2$):

| Painted Faces | Location in Cube | Formula | For $n = 4$ | For $n = 5$ |
|---|---|---|---|---|
| **3 faces painted** | 8 corner vertices | $8$ (constant) | 8 | 8 |
| **2 faces painted** | Interior of 12 edges | $12(n - 2)$ | 24 | 36 |
| **1 face painted** | Interior of 6 faces | $6(n - 2)^2$ | 24 | 54 |
| **0 faces painted** | Inner core | $(n - 2)^3$ | 8 | 27 |
| **Total cubes** | Entire cube | $n^3$ | 64 | 125 |

**Worked example 3 (Cube with alternating painted faces).** A solid wooden cube of side 4 cm is painted: the top and bottom faces are painted Red, while all four side faces are painted Green. It is sliced into 64 unit cubes of side 1 cm. Find the number of unit cubes that have:
1. At least one Red face and at least one Green face painted.
2. Only Green paint.

*Step 1: Identify geometry for $n = 4$.*
- Total unit cubes: $4^3 = 64$.
- Red faces: top layer ($z = 4$) and bottom layer ($z = 1$).
- Green faces: side faces ($x = 1, x = 4, y = 1, y = 4$).

*Step 2: Cubes with both Red and Green paint.*
These cubes must lie on the boundary of the top or bottom face (where Red meets Green):
- 8 corner cubes: each has 1 Red face + 2 Green faces. (8 cubes)
- Edge cubes on top and bottom faces: each edge has $n - 2 = 2$ cubes. There are 4 top edges and 4 bottom edges $= 8$ edges $\times 2 = 16$ cubes. Each has 1 Red face + 1 Green face.
- Total cubes with $\ge 1$ Red and $\ge 1$ Green:
  $$8 + 16 = 24\text{ cubes}$$

*Step 3: Cubes with ONLY Green paint.*
These cubes must have Green paint but NO Red paint, so they cannot belong to the top or bottom layer ($z = 1$ or $z = 4$):
- They must lie in the middle 2 layers ($z = 2, 3$).
- In each middle layer ($4 \times 4$ slice), the cubes on the perimeter have Green paint:
  - 4 corners of the slice: have 2 Green faces. (4 cubes)
  - 8 edge cubes of the slice (2 per side): have 1 Green face. (8 cubes)
  - Interior core cubes ($2 \times 2 = 4$): have 0 paint.
  - Cubes with Green paint in one middle layer: $4 + 8 = 12$ cubes.
- Across both middle layers: $12 \times 2 = 24\text{ cubes}$.

## 4. 2D and 3D Shape Counting and Decomposition [PYQ: Medium]

Systematic counting prevents duplicate counting and omissions in overlapping figures.

```text
       Counting Rectangles and Squares in a Grid (4 x 3):
       ┌───┬───┬───┬───┐
       │   │   │   │   │   • Squares of size 1x1: 4 × 3 = 12
       ├───┼───┼───┼───┤   • Squares of size 2x2: 3 × 2 = 6
       │   │   │   │   │   • Squares of size 3x3: 2 × 1 = 2
       ├───┼───┼───┼───┤   • Total Squares = 12 + 6 + 2 = 20
       │   │   │   │   │   • Total Rectangles = (5 × 4 / 2) × (4 × 3 / 2) = 10 × 6 = 60
       └───┴───┴───┴───┘
```

### 4.1 Grid Formulas
- **Squares in $m \times n$ grid ($m \ge n$):**
  $$N_{\text{sq}} = \sum_{k=1}^n (m - k + 1)(n - k + 1) = mn + (m-1)(n-1) + \dots + (m-n+1)(1)$$
- **Rectangles in $m \times n$ grid:**
  Any rectangle is formed by choosing 2 horizontal lines from $m + 1$ lines and 2 vertical lines from $n + 1$ lines:
  $$N_{\text{rect}} = \binom{m + 1}{2} \times \binom{n + 1}{2} = \frac{m(m + 1)}{2} \times \frac{n(n + 1)}{2}$$

### 4.2 Triangle Counting in Decomposed Geometries
- **Triangles with common vertex and base divided into $k$ segments:**
  $$N_{\text{tri}} = \frac{k(k + 1)}{2}$$
     If intersected by $h$ horizontal parallel dividing lines (creating $h + 1$ strips), total triangles $= (h + 1) \times \frac{k(k + 1)}{2}$.

**Worked example 4 (Systematic shape counting).** Find the total number of squares in a $4 \times 6$ rectangular grid of unit cells.

*Step 1: Set grid dimensions.*
Let $m = 6$ (columns) and $n = 4$ (rows). Size of squares $k$ ranges from $1$ to $\min(6, 4) = 4$.

*Step 2: Enumerate by square dimension $k \times k$.*
- $1 \times 1$ squares: $(6 - 1 + 1)(4 - 1 + 1) = 6 \times 4 = 24$
- $2 \times 2$ squares: $(6 - 2 + 1)(4 - 2 + 1) = 5 \times 3 = 15$
- $3 \times 3$ squares: $(6 - 3 + 1)(4 - 3 + 1) = 4 \times 2 = 8$
- $4 \times 4$ squares: $(6 - 4 + 1)(4 - 4 + 1) = 3 \times 1 = 3$

*Step 3: Sum the categories.*
$$\text{Total squares} = 24 + 15 + 8 + 3 = 50\text{ squares}$$

## 5. Shape Assembly, Grouping, and Visual Symmetry [PYQ: Medium]

### 5.1 Assembling Pieces (Tessellation & Polyominoes)
When assembling a target geometric boundary from a set of pieces:
1. **Area Conservation:** The total area of pieces must exactly equal the area of the target outline:
   $$\text{Area}(\text{Target}) = \sum \text{Area}(\text{Piece}_i)$$
2. **Boundary and Corner Matching:** Locate pieces with sharp acute angles or straight edges to fill corners of the target frame first.
3. **Parity / Checkerboard Invariant:** For polyominoes (e.g., dominoes covering a truncated grid), coloring the grid in alternating black and white cells proves whether a tiling is possible.

### 5.2 Grouping and Symmetries
- **Rotational Symmetry of Order $k$:** Figure looks identical under rotation by $360^\circ / k$.
- **Reflection Symmetry:** Number of lines of bilateral symmetry across which the shape can be reflected into itself.

**Worked example 5 (Minimum squares to achieve symmetry).** Consider a $3 \times 3$ grid with cells at $(r, c)$ for $r, c \in \{1, 2, 3\}$. Currently, cells $(1, 1), (1, 2), (2, 1)$ are shaded. What is the minimum number of additional cells that must be shaded so that the resulting grid is symmetric about the main diagonal connecting $(1, 1)$ to $(3, 3)$?

*Step 1: Identify the line of symmetry.*
The main diagonal connects $(1, 1), (2, 2), (3, 3)$.
Under reflection across this diagonal, cell $(r, c)$ is mapped to cell $(c, r)$.

*Step 2: Inspect currently shaded cells.*
- Cell $(1, 1)$ maps to $(1, 1)$ (on the diagonal, already symmetric).
- Cell $(1, 2)$ maps to $(2, 1)$, which is already shaded!
- Cell $(2, 1)$ maps to $(1, 2)$, which is already shaded!

*Conclusion:* The currently shaded pattern is **already symmetric** about the main diagonal! The minimum number of additional cells to be added is **0**.

## Formulas and facts to memorise

| Transformation / Concept | Formula / Rule | When to use in GATE |
|---|---|---|
| Rotation $90^\circ$ CCW | $(x, y) \mapsto (-y, x)$ | Pure counter-clockwise turning |
| Rotation $180^\circ$ | $(x, y) \mapsto (-x, -y)$ | Inversion through origin |
| Rotation $90^\circ$ CW | $(x, y) \mapsto (y, -x)$ | Clockwise quarter-turn |
| Reflection across $y$-axis | $(x, y) \mapsto (-x, y)$ | Horizontal flip (vertical mirror) |
| Reflection across $x$-axis | $(x, y) \mapsto (x, -y)$ | Vertical flip (horizontal mirror) |
| Reflection across $y = x$ | $(x, y) \mapsto (y, x)$ | Diagonal mirror swap |
| Painted cube 3-face | $8$ | Corner unit cubes |
| Painted cube 2-face | $12(n - 2)$ | Edge unit cubes |
| Painted cube 1-face | $6(n - 2)^2$ | Face center unit cubes |
| Painted cube 0-face | $(n - 2)^3$ | Interior unpainted unit cubes |
| Squares in $m \times n$ grid | $\sum_{k=1}^{\min(m,n)} (m - k + 1)(n - k + 1)$ | Counting squares in rectangular grid |
| Rectangles in $m \times n$ grid | $\binom{m+1}{2}\binom{n+1}{2}$ | All rectangular subgrids |

## GATE traps

- **Non-commutativity of transformations:** $R_{90} \circ M_y \ne M_y \circ R_{90}$. Do not swap the order of transformations when solving composed motion questions.
- **Handedness violation:** You cannot obtain a reflected figure by rotating it in the 2D plane. If an arrow points clockwise, in-plane rotations keep it clockwise; reflection makes it counter-clockwise.
- **Punch on a fold line:** Do not multiply by $2^k$ blindly. A hole punched right on a fold crease unfolds into fewer openings than an interior punch.
- **Long edge identification in folding:** After each fold, check which edge is now longer. The next fold along the axis of symmetry perpendicular to the longer edge may flip directions.
- **Opposite faces sharing corners:** On a cube, opposite faces share **zero** vertices and **zero** edges. If two faces touch at even a point, they cannot be opposite.

## Connections

- [Matrices and determinants](../03-linear-algebra/matrices-and-determinants.md) — 2D and 3D rotation and reflection transformation matrices and determinants.
- [Quantitative aptitude](quantitative-aptitude.md) — coordinate geometry, perimeter, area, and volume formulas.
- [Analytical aptitude](analytical-aptitude.md) — pattern series, visual analogies, and structural puzzles.

## Practice

**Q1 (MCQ).** Point $P(4, -3)$ is rotated $90^\circ$ counter-clockwise about the origin. What are the coordinates of the resulting point $P'$?  
(A) $(3, 4)$  
(B) $(-3, -4)$  
(C) $(-4, 3)$  
(D) $(3, -4)$  

<details><summary>Answer</summary>

**Answer:** A ($(3, 4)$)  
**Solution:**  
A counter-clockwise rotation of $90^\circ$ about the origin transforms coordinates as:  
$$(x, y) \mapsto (-y, x)$$  
Given $(x, y) = (4, -3)$:  
$$x' = -(-3) = 3$$  
$$y' = 4$$  
Resulting point is $(3, 4)$.

</details>

**Q2 (MCQ).** A square sheet of paper is folded in half from left to right, and then folded in half from top to bottom. A single circular hole is punched in the interior of the resulting quarter-square, completely away from any folded edges. When the sheet is fully unfolded, how many circular holes are visible, and what is their arrangement?  
(A) 2 holes in a vertical line  
(B) 4 holes arranged symmetrically in a $2 \times 2$ square pattern  
(C) 4 holes arranged along a straight diagonal  
(D) 8 holes arranged in a circle  

<details><summary>Answer</summary>

**Answer:** B  
**Solution:**  
The paper undergoes two independent folds along two mutually perpendicular axes (one vertical, one horizontal).  
When unfolded, each fold line acts as an axis of mirror symmetry:  
- Unfolding the vertical crease duplicates the hole across the horizontal fold line ($1 \to 2$ holes).  
- Unfolding the horizontal crease duplicates both holes across the vertical centerline ($2 \to 4$ holes).  
Because the punch is in the interior and not on any fold line, exactly 4 distinct holes are produced, forming a symmetric $2 \times 2$ grid.

</details>

**Q3 (NAT).** A rectangular paper of dimensions $32\text{ cm} \times 20\text{ cm}$ is folded in half 3 times. Each fold is made along the line of symmetry perpendicular to its longer edge. What is the perimeter (in cm) of the final folded rectangular sheet?

<details><summary>Answer</summary>

**Answer:** 36  
**Solution:**  
Track the dimensions step by step:  
- **Initial:** $32\text{ cm} \times 20\text{ cm}$. Longer edge is 32 cm.  
- **Fold 1:** Fold perpendicular to 32 cm edge $\implies 16\text{ cm} \times 20\text{ cm}$.  
- **Fold 2:** Current dimensions are $16\text{ cm} \times 20\text{ cm}$. Longer edge is now 20 cm! Fold perpendicular to 20 cm edge $\implies 16\text{ cm} \times 10\text{ cm}$.  
- **Fold 3:** Current dimensions are $16\text{ cm} \times 10\text{ cm}$. Longer edge is 16 cm. Fold perpendicular to 16 cm edge $\implies 8\text{ cm} \times 10\text{ cm}$.  
Perimeter of final sheet:  
$$\text{Perimeter} = 2 \times (8 + 10) = 2 \times 18 = 36\text{ cm}$$

</details>

**Q4 (MCQ).** Consider the standard T-shaped cube net shown below:
```text
          ┌───┐
          │ 1 │
      ┌───┼───┼───┬───┐
      │ 2 │ 3 │ 4 │ 5 │
      └───┼───┼───┴───┘
          │ 6 │
          └───┘
```
When folded into a closed 3D cube, which face is situated directly opposite to face 3?  
(A) Face 1  
(B) Face 4  
(C) Face 5  
(D) Face 6  

<details><summary>Answer</summary>

**Answer:** C (Face 5)  
**Solution:**  
In any continuous horizontal or vertical row of four squares, alternate faces fold into opposite faces:  
- Face 2 is opposite Face 4 (separated by 3).  
- Face 3 is opposite Face 5 (separated by 4).  
The two attached flaps (top and bottom) fold up to become opposite to each other:  
- Face 1 is opposite Face 6.  
Hence, the face directly opposite to face 3 is **Face 5**.

</details>

**Q5 (NAT).** A solid wooden cube of side length 5 cm is painted blue on all six of its outer faces. It is subsequently cut into smaller unit cubes of side length 1 cm each. How many of the resulting unit cubes have **exactly two** faces painted blue?

<details><summary>Answer</summary>

**Answer:** 36  
**Solution:**  
Here $n = 5$ (number of unit divisions along each edge).  
Unit cubes with exactly two painted faces originate exclusively from the edges of the original large cube (excluding the 8 corners, which have 3 painted faces).  
A cube has 12 edges. Along each edge, there are $n - 2$ unit cubes with 2 painted faces:  
$$\text{Number of cubes} = 12(n - 2) = 12(5 - 2) = 12 \times 3 = 36$$

</details>

**Q6 (NAT).** A rectangular grid contains 4 horizontal rows and 6 vertical columns of unit squares (dimensions $4 \times 6$). What is the total number of squares of all sizes contained within this grid?

<details><summary>Answer</summary>

**Answer:** 50  
**Solution:**  
The total number of squares in an $m \times n$ grid is given by:  
$$N_{\text{sq}} = \sum_{k=1}^{\min(m, n)} (m - k + 1)(n - k + 1)$$  
For $m = 6$ and $n = 4$:  
- $1 \times 1$ squares: $(6)(4) = 24$  
- $2 \times 2$ squares: $(5)(3) = 15$  
- $3 \times 3$ squares: $(4)(2) = 8$  
- $4 \times 4$ squares: $(3)(1) = 3$  
Summing all sizes:  
$$24 + 15 + 8 + 3 = 50\text{ squares}$$

</details>

**Q7 (MSQ).** Let $P$ be an asymmetric triangle in the 2D plane with vertices $A(1, 1), B(3, 1), C(1, 4)$.  
Let $R_{90}$ denote a $90^\circ$ counter-clockwise rotation about the origin, and let $M_y$ denote reflection across the $y$-axis.  
Which of the following statements is/are TRUE?  
(A) The image of vertex $B(3, 1)$ under $R_{90}$ is $(-1, 3)$.  
(B) The image of vertex $B(3, 1)$ under the composite transformation $M_y \circ R_{90}$ (rotation first, then reflection across $y$-axis) is $(1, 3)$.  
(C) $R_{90}$ and $M_y$ commute, meaning $R_{90} \circ M_y = M_y \circ R_{90}$.  
(D) Triangle $P$ cannot be mapped onto its reflection $M_y(P)$ by any sequence of pure in-plane rotations and translations.  

<details><summary>Answer</summary>

**Answer:** A, B, D  
**Solution:**  
- **(A) Rotation of $B(3, 1)$:** Using $(x, y) \mapsto (-y, x)$, $R_{90}(3, 1) = (-1, 3)$. **(True)**  
- **(B) Composite $M_y \circ R_{90}$:** Applying reflection $(x, y) \mapsto (-x, y)$ to $(-1, 3)$ yields $(1, 3)$. **(True)**  
- **(C) Commutativity check:** Evaluate $R_{90} \circ M_y$ on $(3, 1)$:  
  $M_y(3, 1) = (-3, 1)$. Then $R_{90}(-3, 1) = (-1, -3) \ne (1, 3)$. Since $(R_{90} \circ M_y)(B) \ne (M_y \circ R_{90})(B)$, the operations do not commute. **(False)**  
- **(D) Chirality / handedness:** Any sequence of pure in-plane translations and rotations has determinant $+1$, preserving cyclic vertex orientation (chirality). Reflection $M_y$ has determinant $-1$, which reverses chirality. Hence no 2D rigid motion can map an asymmetric figure onto its mirror reflection. **(True)**

</details>
