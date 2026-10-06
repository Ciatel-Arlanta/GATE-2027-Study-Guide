# Trees, Binary Search Trees and AVL Trees

> **Paper:** CS+DA · **Priority:** P0 · **Plan topics:** Trees; Binary search trees; Data-structure operations and complexity
> **Prerequisites:** [Linked lists](linked-lists.md), [Recursion in C](../05-c-programming/functions-recursion-structures.md), [Stacks and queues](arrays-stacks-queues.md) · **Leads to:** [Heaps](heaps.md), [Graph traversals](../08-algorithms/graph-traversals.md), [B and B+ trees](../14-databases/file-organization-and-indexing.md)

**Height convention used here:** the height of a tree is the number of **edges** on the longest root-to-leaf path, so a single node has height 0 and the empty tree has height -1. Many books (and some GATE questions) count **nodes** instead (single node = height 1); then add 1 to every height below and use `2^h - 1` for the maximum node count. Always read the question's convention.

## Quick glance

- **Tree** = connected acyclic graph with a designated root; `n` nodes ⇒ `n - 1` edges.
- **Binary tree:** each node has ≤ 2 children (left, right). Height `h` (edges): nodes between `h + 1` and `2^(h+1) - 1`; at most `2^l` nodes on level `l`.
- **Leaves = nodes of degree 2 + 1:** `n0 = n2 + 1`. For a full k-ary tree with `i` internal nodes: leaves = `(k-1)i + 1`.
- **Traversals:** preorder (N L R), inorder (L N R), postorder (L R N), level order (queue). **Inorder + preorder** or **inorder + postorder** determine a tree uniquely; **preorder + postorder do not** (unless every internal node has 2 children).
- **Number of binary tree shapes / BSTs with n nodes:** Catalan `C(2n,n)/(n+1)` = 1, 2, 5, 14, 42.
- **BST:** left subtree < node < right subtree; **inorder traversal is sorted**; search/insert/delete cost O(height): O(log n) average, **O(n) worst (skewed)**.
- **BST delete:** leaf → remove; one child → splice; two children → replace by inorder successor (min of right subtree) or predecessor, then delete that node.
- **AVL:** BST with `|height(L) - height(R)| ≤ 1` at every node; fixed by four rotations (LL, RR, LR, RL); height ≤ ~1.44 log₂ n; **min nodes `N(h) = N(h-1) + N(h-2) + 1`** with `N(0) = 1, N(1) = 2`: 1, 2, 4, 7, 12, 20, 33.
- Complete binary tree with `n` nodes has height `floor(log₂ n)`.

## 1. Terminology

| Term | Meaning |
|---|---|
| Root | unique node with no parent |
| Parent / child / sibling | direct neighbours one level up / down / same parent |
| Leaf (external) | node with no children; others are internal |
| Degree of a node | number of children; degree of the tree = max over nodes |
| Depth (level) of a node | edges from the root (root depth 0) |
| Height of a node | edges on the longest path down to a leaf; height of tree = height of root |
| Subtree | a node together with all its descendants |
| Ancestor / descendant | on the path to / from the root |
| Path length | edges on a path |

```text
        A            depth 0
       / \
      B   C          depth 1
     / \   \
    D   E   F        depth 2        height of tree = 2, n = 6, edges = 5
```

**Types of binary trees**

| Type | Definition | Nodes for height h |
|---|---|---|
| Full (strict) | every node has 0 or 2 children | at least `2h + 1`, at most `2^(h+1) - 1` |
| Perfect | full and all leaves at the same depth | exactly `2^(h+1) - 1` |
| Complete | all levels full except possibly the last, which is filled **left to right** | `2^h` to `2^(h+1) - 1` |
| Skewed (degenerate) | every node has one child | `h + 1` |

Heaps use complete trees ([heaps](heaps.md)). A complete tree with `n` nodes has height `floor(log₂ n)`: n = 1000 gives 9 (since 2^9 = 512 ≤ 1000 < 1024).

### 1.1 Counting properties

**Property 1.** A tree with `n` nodes has `n - 1` edges (every non-root node has exactly one parent edge).

**Property 2 (binary tree): `n0 = n2 + 1`.** Let `n0, n1, n2` be the numbers of nodes with 0, 1, 2 children. Edges counted by children: `n1 + 2·n2`. Edges counted by nodes: `n - 1 = n0 + n1 + n2 - 1`. Equate: `n1 + 2n2 = n0 + n1 + n2 - 1` ⇒ **n0 = n2 + 1**.

**Property 3 (k-ary).** If every internal node has exactly `k` children and there are `i` internal nodes: `n = k·i + 1`, leaves `= (k - 1)·i + 1`. Example: a 3-ary tree with 10 internal nodes has 31 nodes and 21 leaves.

**Worked example 1.** A binary tree has 20 leaves and 10 nodes with exactly one child. Then `n2 = 19`, `n = 20 + 19 + 10 = 49`, edges 48.

**Worked example 2 (height bounds).** A binary tree with 100 nodes: minimum height `ceil(log₂ 101) - 1 = 6` (a complete tree: levels 0..5 hold 63 nodes, level 6 holds the remaining 37), maximum height 99 (skewed).

**Level / index relations in a complete tree** (1-based array representation): node `i` has children `2i`, `2i + 1`, parent `floor(i/2)`.

## 2. Traversals

Depth-first orders differ in when the node `N` is visited relative to the left (`L`) and right (`R`) subtrees.

```c
struct node { int data; struct node *left, *right; };
void preorder (struct node *t) { if (t) { visit(t); preorder(t->left);  preorder(t->right); } }
void inorder  (struct node *t) { if (t) { inorder(t->left);  visit(t);  inorder(t->right); } }
void postorder(struct node *t) { if (t) { postorder(t->left); postorder(t->right); visit(t); } }
```

**Worked example 3.** Tree: root A; left child B (children D, E); right child C (children F, G).

```text
          A
        /   \
       B     C
      / \   / \
     D   E F   G
```
| Traversal | Output |
|---|---|
| Preorder | A B D E C F G |
| Inorder | D B E A F C G |
| Postorder | D E B F G C A |
| Level order | A B C D E F G |

**Level order** uses a queue: enqueue the root; repeatedly dequeue a node, visit it, enqueue its non-null children. O(n) time, O(width) space.

**Iterative inorder** with an explicit stack: go left pushing nodes; pop, visit, then go to the right child. Stack depth = height (O(n) worst, O(log n) balanced). Recursion uses the call stack in the same way ([recursion](../05-c-programming/functions-recursion-structures.md)).

All depth-first traversals take **O(n)** time and **O(h)** extra space.

**Worked example 4 (expression tree).** For `(a + b) * (c - d) / e`:
```text
        /
       / \
      *   e
     / \
    +   -
   / \ / \
  a  b c  d
```
Postorder gives postfix `a b + c d - * e /`; preorder gives prefix `/ * + a b - c d e`; inorder (with parentheses) gives the infix. Evaluating = postorder traversal. See [infix to postfix](arrays-stacks-queues.md).

### 2.1 Reconstruction from two traversals

**Inorder + preorder.** The first preorder element is the root; find it in the inorder sequence: everything to its left is the left subtree, to its right the right subtree; recurse (the left subtree's size tells you how many preorder elements belong to it). **Inorder + postorder:** the **last** postorder element is the root; same recursion.

**Worked example 5.** Inorder `4 2 5 1 6 3`, preorder `1 2 4 5 3 6`.
- Root = 1. Inorder splits into left `{4 2 5}` and right `{6 3}`.
- Preorder remainder `2 4 5` (left, size 3) and `3 6` (right, size 2).
- Left: root 2; inorder `4 | 2 | 5` ⇒ left child 4, right child 5.
- Right: root 3; inorder `6 | 3` ⇒ left child 6, no right child.
- Tree: `1(2(4,5), 3(6,-))`. Postorder: `4 5 2 6 3 1`.

**Why preorder + postorder is ambiguous.** A node with only one child can have it on either side without changing either sequence. Preorder `A B`, postorder `B A` fits both "B is the left child of A" and "B is the right child of A". **If the number of nodes with exactly one child is `k`, there are `2^k` binary trees with the given preorder and postorder** (k = 0, a full tree, gives a unique tree). Preorder `A B C` with postorder `C B A` (two single-child nodes) gives 4 trees.

**Counting trees.** Number of structurally distinct binary trees with `n` nodes `= C(n)` (Catalan): 1, 1, 2, 5, 14, 42 for n = 0..5. Since a BST on keys `1..n` is determined by its shape, the number of BSTs on `n` distinct keys is also `C(n)` (n = 3: 5). The number of binary trees on `n` *labelled* nodes is `n!·C(n)`. Given a single traversal sequence (say preorder `1..n`) there are `C(n)` trees that can produce it.

### 2.2 Threaded binary trees (brief)

A binary tree with `n` nodes has `2n` child pointers, of which `n - 1` are used, so **`n + 1` are NULL**. A **threaded tree** reuses NULL right pointers to point to the inorder **successor** (and NULL left pointers to the inorder predecessor in a double-threaded tree), plus a flag per pointer to tell "child" from "thread". Inorder traversal then needs no stack and no recursion: O(n) time, O(1) space.

## 3. Binary search trees (BST)

**Definition.** For every node `x`: all keys in the left subtree are `< x.key` and all keys in the right subtree are `> x.key` (duplicates are excluded or placed consistently on one side). **The inorder traversal of a BST visits keys in sorted order**; this is the property most GATE questions use.

### 3.1 Search, insert

```c
struct node *search(struct node *t, int k) {
    while (t && t->data != k) t = (k < t->data) ? t->left : t->right;
    return t;
}
struct node *insert(struct node *t, int k) {
    if (t == NULL) return newnode(k);
    if (k < t->data) t->left  = insert(t->left, k);
    else if (k > t->data) t->right = insert(t->right, k);
    return t;
}
```
Both walk one root-to-leaf path: **O(h)**. Minimum = leftmost node, maximum = rightmost node, also O(h). Inorder successor of a node with a right child = minimum of its right subtree; otherwise the lowest ancestor whose left subtree contains the node.

**Insert order matters.** The same keys in different orders give different shapes. Inserting 1, 2, 3, 4, 5 (sorted) creates a right-skewed chain of height 4 (O(n) search); inserting 3, 1, 5, 2, 4 gives a balanced-looking tree of height 2. The average height over random insertion orders is O(log n) (about 1.39 log₂ n for the average search depth).

**Worked example 6 (build, then delete).** Insert `50, 30, 70, 20, 40, 60, 80, 35, 45, 65` in order:
```text
            50
          /    \
        30      70
       /  \    /  \
     20   40  60   80
         /  \   \
        35  45   65
```
Inorder: `20 30 35 40 45 50 60 65 70 80` (sorted ✓). Preorder: `50 30 20 40 35 45 70 60 65 80`.

### 3.2 Deletion: three cases

1. **Leaf:** remove it.
2. **One child:** replace the node by its child.
3. **Two children:** replace the node's key by its **inorder successor** (smallest key in the right subtree) [or inorder predecessor], then delete that successor node from the right subtree (it has no left child, so cases 1 or 2 apply).

**Worked example 7.** Continue with the tree above.
- **Delete 20** (leaf): 30 loses its left child. Tree: `50(30(-,40(35,45)), 70(60(-,65), 80))`.
- **Delete 70** (two children 60 and 80): successor = min of right subtree `{80}` = 80. Replace 70 by 80 and remove the old 80 leaf: `50(30(-,40(35,45)), 80(60(-,65), -))`.
- **Delete 50** (two children): successor = min of the right subtree rooted at 80 = go left to 60. Replace 50 by 60; delete the old node 60, which has only a right child 65, so 65 takes its place: `60(30(-,40(35,45)), 80(65,-))`.
- Final preorder: **60 30 40 35 45 80 65**; inorder `30 35 40 45 60 65 80` still sorted.

### 3.3 Cost and height

| Case | Height | Search/insert/delete |
|---|---|---|
| Best/balanced | `floor(log₂ n)` | O(log n) |
| Worst (sorted input, skewed) | `n - 1` | **O(n)** |
| Average (random insertions) | height O(log n); average node depth ≈ 2 ln n ≈ 1.39 log₂ n | O(log n) |

For `n` nodes the BST height lies in `[floor(log₂ n), n - 1]`.

**Valid search sequences.** In a BST, when searching for key `k`, the keys examined form a sequence in which each key lies strictly inside the interval created by all the earlier comparisons (after comparing with `x` and going right, all later keys exceed `x`; after going left, all are below `x`).

**Worked example 8.** Keys 1..100 in a BST, search for 55. Sequence `10, 90, 20, 80, 50, 60, 55`: 10 <55 (lo=10), 90 >55 (hi=90), 20 in (10,90) → lo=20, 80 → hi=80, 50 → lo=50, 60 → hi=60, 55 in (50,60) ✓ **valid**. Sequence `10, 90, 20, 80, 50, 60, 30, 55` has `30 ∉ (50, 60)` ⇒ **invalid**.

**BST from preorder.** Preorder `50 30 20 40 70 60 80` determines the BST: the first key is the root, the next smaller keys form the left subtree, the rest the right. Postorder: `20 40 30 60 80 70 50`. Given only a **BST's** preorder (or postorder), the tree is unique (sorting the preorder gives the inorder).

## 4. AVL trees

A BST can degenerate. An **AVL tree** (Adelson-Velsky and Landis) keeps it balanced: for every node, **balance factor** `BF = height(left) - height(right) ∈ {-1, 0, +1}`. Then height is O(log n) and all operations are **O(log n) worst case**.

**Height bounds.** Let `N(h)` be the minimum number of nodes of an AVL tree of height `h` (edges). The sparsest tree has one subtree of height `h - 1` and the other of height `h - 2`:

**N(h) = N(h-1) + N(h-2) + 1, N(0) = 1, N(1) = 2.**

| h | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|
| N(h) | 1 | 2 | 4 | 7 | 12 | 20 | 33 | 54 |

(`N(h) = Fib(h+3) - 1`.) Hence `n ≥ N(h) ≈ φ^h` and **h ≤ 1.44 log₂(n + 2)**. Maximum nodes at height `h` is `2^(h+1) - 1` as for any binary tree. With the node-counting convention (single node = height 1) the table shifts: 1, 2, 4, 7, 12 for heights 1, 2, 3, 4, 5.

### 4.1 Rotations

After inserting into a subtree, walk back up; at the **first (lowest) unbalanced node** `z` (|BF| = 2) apply one of four fixes. Let `y` be the child of `z` on the heavy side and `x` the heavy child of `y`.

| Case | Where inserted | Fix |
|---|---|---|
| **LL** (left-left) | left subtree of left child | single **right rotation** at `z` |
| **RR** | right subtree of right child | single **left rotation** at `z` |
| **LR** | right subtree of left child | left rotation at `y`, then right rotation at `z` |
| **RL** | left subtree of right child | right rotation at `y`, then left rotation at `z` |

```text
 LL (right rotation at z)                  RR (left rotation at z)
        z                y                    z                    y
       / \              / \                  / \                  / \
      y   T4    ==>    x   z                T1  y        ==>     z   x
     / \              /\  / \                  / \              / \  /\
    x   T3           T1 T2 T3 T4              T2  x            T1 T2 T3 T4
   /\                                             /\
  T1 T2                                          T3 T4
```
LR: `z(y(T1, x(T2,T3)), T4)` becomes `x(y(T1,T2), z(T3,T4))`. RL: `z(T1, y(x(T2,T3), T4))` becomes `x(z(T1,T2), y(T3,T4))`. A rotation preserves the inorder sequence. **Insertion needs at most one (single or double) rotation; deletion may need O(log n) rotations.**

**Worked example 9.** Insert `10, 20, 30, 40, 50, 25` into an empty AVL tree.

| Insert | Tree after (before fixing shown as note) | Action |
|---|---|---|
| 10 | `10` | |
| 20 | `10(-,20)` | BF(10) = -1 |
| 30 | `10(-,20(-,30))` BF(10) = -2 | **RR at 10** → `20(10,30)` |
| 40 | `20(10,30(-,40))` | balanced |
| 50 | `20(10,30(-,40(-,50)))` BF(30) = -2 | **RR at 30** → `20(10,40(30,50))` |
| 25 | `20(10,40(30(25,-),50))`: heights: left of 20 = 0, right = 2, BF(20) = -2; its right child 40 is left-heavy | **RL at 20**: right rotation at 40, then left rotation at 20 |

Result: `30(20(10,25), 40(-,50))`. Preorder `30 20 10 25 40 50`; inorder `10 20 25 30 40 50` ✓ (rotations preserve it).

**Worked example 10 (LR).** Insert `30, 10, 20`: `30(10(-,20),-)` has BF(30) = +2 and the heavy child 10 is right-heavy ⇒ **LR**: left-rotate 10 (→ `30(20(10,-),-)`), right-rotate 30 (→ `20(10,30)`).

**Worked example 11 (sequential insertion of sorted keys).** Inserting `1, 2, ..., 7` into an AVL tree yields the perfect tree rooted at 4 (`4(2(1,3),6(5,7))`), height 2, instead of a chain of height 6. Exactly 4 rotations occur, all single left (RR) rotations, triggered by the insertions of 3, 5, 6 and 7 (at nodes 1, 3, 2 and 5 respectively).

**Complexity.** Search, insert, delete: O(log n) worst case. Storage: one height (or BF) per node. Compare with unbalanced BST: O(n) worst. Related balanced structures: red-black trees (height ≤ 2 log₂(n+1)), [B and B+ trees](../14-databases/file-organization-and-indexing.md) for disks.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
|---|---|---|
| Edges | n - 1 | any tree |
| Binary tree leaves | n0 = n2 + 1 | degree counting |
| k-ary, i internal | n = ki + 1, leaves = (k-1)i + 1 | full k-ary |
| Max nodes, height h (edges) | 2^(h+1) - 1 | upper bounds |
| Max nodes at level l | 2^l | level questions |
| Min height with n nodes | floor(log₂ n) | complete tree |
| Binary trees / BSTs with n nodes | C(2n,n)/(n+1) | counting |
| Trees from pre+post | 2^k (k = single-child nodes) | ambiguity |
| NULL pointers in n-node binary tree | n + 1 | threading |
| BST inorder | sorted | validity, k-th smallest |
| BST ops | O(h): log n avg, n worst | complexity |
| AVL min nodes | N(h) = N(h-1) + N(h-2) + 1 | height bound |
| AVL height | ≤ 1.44 log₂(n+2) | complexity |
| Rotations | LL→R, RR→L, LR→L then R, RL→R then L | AVL insertion |
| Traversal cost | O(n) time, O(h) space | all DFS orders |

## GATE traps

- **Height convention:** edges vs nodes changes `2^(h+1) - 1` to `2^h - 1` and N(h) tables; use the question's definition.
- **Preorder + postorder** do not identify a tree uniquely; inorder + (pre or post) do.
- **Number of BSTs** on n keys is Catalan, not `n!` (many insertion orders give the same shape).
- **"BST is sorted in preorder"** is false; only inorder is sorted.
- **Deleting a node with two children:** use the inorder successor/predecessor, then delete *that* node; the successor may have a right child.
- **Searching a BST is O(n) worst case**, not O(log n), unless balanced.
- **A node is unbalanced** in AVL only if |BF| ≥ 2; check heights of both subtrees, not just node counts.
- **The rotation type is determined by the path** from the unbalanced node: left-right = LR (double), even when the middle node looks "straight".
- **Valid BST check:** every node must satisfy the bounds of all its ancestors, not only its parent.
- **Complete ≠ full ≠ perfect.** Heaps are complete; a full tree need not be complete.
- **Inorder successor** of a node without a right child is an *ancestor*.
- **Stack usage of recursion** on a skewed tree is O(n).

## Connections

- [Linked lists](linked-lists.md) — a tree node is a list node with two links.
- [Heaps](heaps.md) — complete binary trees stored in arrays; the same height formulas.
- [Arrays, stacks, queues](arrays-stacks-queues.md) — traversals use a stack (DFS) or queue (BFS); expression trees ↔ postfix.
- [Graphs](graphs.md), [Graph traversals](../08-algorithms/graph-traversals.md) — a tree is a connected acyclic graph; DFS/BFS generalise tree traversals.
- [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) — spanning trees have `n - 1` edges.
- [Searching and sorting](../08-algorithms/searching-and-sorting.md) — BST sort = inorder after n inserts; relation to quicksort's recursion tree.
- [Asymptotic analysis](../08-algorithms/asymptotic-analysis.md) — recursion trees and height of decision trees (`Ω(n log n)` comparison bound).
- [Combinatorics](../01-discrete-mathematics/combinatorics.md) — Catalan numbers count binary trees, stack permutations, balanced parentheses.
- [Graph theory](../01-discrete-mathematics/graph-theory.md) — tree properties and counting.
- [File organization and indexing](../14-databases/file-organization-and-indexing.md) — B/B+ trees are multi-way balanced search trees.
- [Syntax-directed translation](../12-compiler-design/syntax-directed-translation.md) and [parsing](../12-compiler-design/parsing.md) — parse trees and expression trees; postorder evaluation.
- [Search (AI)](../17-artificial-intelligence/search.md) — search trees and game trees.
- [Decision trees](../16-machine-learning/classification-methods.md) — a tree-structured classifier.

## Practice

**Q1 (NAT, easy).** The maximum number of nodes in a binary tree of height 4 (height counted in edges, single node has height 0) is?

<details><summary>Answer</summary>

**Answer:** 31  
**Solution:** `2^(4+1) - 1 = 31`.

</details>

**Q2 (NAT).** A binary tree has 20 leaves and 10 nodes of degree 1. How many nodes does it have?

<details><summary>Answer</summary>

**Answer:** 49  
**Solution:** `n2 = n0 - 1 = 19`; `n = 20 + 10 + 19 = 49`.

</details>

**Q3 (MCQ).** Inorder is `4 2 5 1 6 3` and preorder is `1 2 4 5 3 6`. The postorder is  (A) 4 5 2 6 3 1  (B) 4 5 6 2 3 1  (C) 5 4 2 6 3 1  (D) 4 2 5 6 3 1

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** See Worked example 5: tree `1(2(4,5),3(6,-))`; postorder `4 5 2 6 3 1`.

</details>

**Q4 (NAT).** Minimum number of nodes in an AVL tree of height 5 (edges, single node height 0)?

<details><summary>Answer</summary>

**Answer:** 20  
**Solution:** N(0)=1, N(1)=2, N(2)=4, N(3)=7, N(4)=12, N(5)=12+7+1=20.

</details>

**Q5 (MCQ).** Keys `10, 20, 30, 40, 50, 25` are inserted in this order into an initially empty AVL tree. The preorder traversal of the final tree is  (A) 30 20 10 25 40 50  (B) 20 10 40 30 25 50  (C) 30 20 25 10 40 50  (D) 25 20 10 30 40 50

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** See Worked example 9: final tree `30(20(10,25),40(-,50))`.

</details>

**Q6 (NAT).** Keys `50, 30, 70, 20, 40, 60, 80, 35, 45, 65` are inserted into an empty BST; then 20, 70 and 50 are deleted (two-children deletions use the inorder successor). How many nodes are on the longest root-to-leaf path in the resulting tree (counting nodes)?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** The result is `60(30(-,40(35,45)), 80(65,-))`; the longest path 60 → 30 → 40 → 35 has 4 nodes. (Preorder 60 30 40 35 45 80 65.)

</details>

**Q7 (MSQ).** Which statements are true? (A) The number of distinct BSTs on keys 1..4 is 14. (B) Preorder and postorder together always determine a binary tree uniquely. (C) A binary tree with n nodes has n + 1 NULL child pointers. (D) The inorder traversal of a BST is sorted. (E) Insertion into an AVL tree needs at most 2 rotations (counting a double rotation as two).

<details><summary>Answer</summary>

**Answer:** A, C, D, E  
**Solution:** (A) Catalan C(4) = 14. (B) false: ambiguity when nodes have one child. (C) 2n pointers, n - 1 used, so n + 1 NULL. (D) true. (E) a single rotation or one double rotation (two single rotations) restores balance after an insertion.

</details>

**Q8 (NAT, harder).** The preorder of a binary tree is `A B C` and its postorder is `C B A`. How many different binary trees have these traversals?

<details><summary>Answer</summary>

**Answer:** 4  
**Solution:** A has only child B and B has only child C (nodes with exactly one child: A and B, so k = 2). Each such node can have its child on the left or right: `2^2 = 4` trees.

</details>
