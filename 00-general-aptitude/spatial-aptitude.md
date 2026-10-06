# Spatial aptitude: transformations and visual patterns

> **Paper:** CS+DA · **Priority:** P2 · **Plan topics:** translation, rotation, scaling, mirroring, assembling, paper folding/cutting, 2D/3D patterns
> **Prerequisites:** none · **Leads to:** [Quantitative aptitude](quantitative-aptitude.md)

## Quick glance
- Translation moves every point by the same vector; orientation stays fixed.
- Rotation turns around a centre; reflection reverses handedness.
- Scaling changes lengths by $k$ and areas by $k^2$.
- Track an asymmetric feature (one notch, dot, or unequal side) to distinguish rotation from reflection.
- For folding, undo folds in reverse order and mirror each cut across every fold line.

## 1. Transformations
Represent a point $(x,y)$ on a square grid. A 90° counter-clockwise rotation about the origin maps $(x,y)\mapsto(-y,x)$; reflection in the vertical axis maps $(x,y)\mapsto(-x,y)$. Translation by $(a,b)$ maps $(x,y)\mapsto(x+a,y+b)$.

**Worked example.** Point $(2,1)$ rotated 90° counter-clockwise becomes $(-1,2)$. Reflecting the result in the vertical axis gives $(1,2)$. Applying transformations in the opposite order can produce a different answer; preserve the stated order.

## 2. Rotation versus mirror image
A square may look unchanged after several rotations, so use a directional marker. A right-pointing arrow rotated 90° counter-clockwise points up; its mirror across a vertical axis points left. Rotation preserves clockwise order of vertices; reflection reverses it.

## 3. Paper folding and 3D views
When a sheet is folded, each cut or hole is copied by reflection across each fold when the sheet is reopened. If two folds are independent, a single cut can yield four symmetric marks. For cube nets, opposite faces cannot share an edge in the folded cube; label one face and propagate orientations rather than relying on visual intuition.

## GATE traps
- “Rotate then reflect” is not generally the same as “reflect then rotate.”
- A shape's outline may hide orientation; follow a marked corner.
- Distinguish scale factor from area factor.
- Count all copies created when undoing multiple folds.

## Connections
- [Linear algebra: matrices](../03-linear-algebra/matrices-and-determinants.md) — rotations and reflections are linear transformations.
- [Quantitative aptitude](quantitative-aptitude.md) — scale and area relations.

## Practice
**Q1.** Rotate $(3,-2)$ by 90° counter-clockwise about origin.
<details><summary>Answer</summary> $(2,3)$ using $(x,y)\mapsto(-y,x)$.</details>

**Q2 (MCQ).** A length scale doubles. The area scale is (A) 2 (B) 4 (C) 6 (D) 8.
<details><summary>Answer</summary> **B.** Area scales by the square.</details>

**Q3.** A paper is folded once in half and one interior hole is punched away from the fold. How many holes after opening?
<details><summary>Answer</summary> **2**, symmetric across the fold line.</details>
