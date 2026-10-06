# Data Structures: Checkpoint

12 mixed questions across the section, easy to hard. Chapters: [arrays/stacks/queues](arrays-stacks-queues.md) · [linked lists](linked-lists.md) · [trees/BST/AVL](trees-and-bst.md) · [heaps](heaps.md) · [graphs](graphs.md) · [complexity](complexity-reference.md).

**Q1 (NAT).** `A[-3..4][2..6]` is stored in row-major order, base address 500, 2 bytes per element. Address of `A[2][5]`?

<details><summary>Answer</summary>

**Answer:** 556  
**Solution:** Columns `N2 = 6 - 2 + 1 = 5`. Offset `((2-(-3))·5 + (5-2)) = 25 + 3 = 28` elements; 500 + 28·2 = 556.

</details>

**Q2 (NAT).** Evaluate the postfix expression `6 2 3 + - 3 8 2 / + * 2 ^ 3 +` (`^` is exponentiation).

<details><summary>Answer</summary>

**Answer:** 52  
**Solution:** `2 3 +` = 5; `6 5 -` = 1; `8 2 /` = 4; `3 4 +` = 7; `1 7 *` = 7; `7 2 ^` = 49; `49 3 +` = 52.

</details>

**Q3 (NAT).** Numbers 1..4 are pushed in order on a stack (pops allowed at any time). How many different output sequences are possible?

<details><summary>Answer</summary>

**Answer:** 14  
**Solution:** Catalan number `C(8,4)/5 = 70/5 = 14`.

</details>

**Q4 (NAT).** A circular queue is implemented in an array of size 10 with empty `front == rear` and full `(rear+1) % 10 == front`. If `front = 7` and `rear = 3`, how many elements does it hold?

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:** `(3 - 7 + 10) % 10 = 6` (indices 7, 8, 9, 0, 1, 2).

</details>

**Q5 (NAT).** A singly linked list has a tail of 5 nodes followed by a cycle of 3 nodes. `slow` moves one node and `fast` two nodes per step, both starting at the head. After how many steps do they first meet?

<details><summary>Answer</summary>

**Answer:** 6  
**Solution:** They can only meet inside the cycle after `k` steps with `k ≡ 0 (mod 3)` (slow at offset `k - 5`, fast at offset `2k - 5` in the cycle, equal mod 3 iff `k ≡ 0`) and `k ≥ 5`. The smallest such `k` is 6.

</details>

**Q6 (NAT).** A binary tree has 100 nodes, of which 29 have exactly one child. How many leaves does it have?

<details><summary>Answer</summary>

**Answer:** 36  
**Solution:** `n0 + n2 = 100 - 29 = 71` and `n0 = n2 + 1`, so `2n2 + 1 = 71`, `n2 = 35`, `n0 = 36`.

</details>

**Q7 (MCQ).** A tree has preorder `A B D C` and inorder `D B A C`. Its postorder is  (A) D B C A  (B) D C B A  (C) B D C A  (D) D B A C

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** Root A; inorder left part `D B`, right part `C`. Preorder after A: `B D` (left), `C` (right). Left subtree: root B, left child D. Tree `A(B(D,-), C)`; postorder `D B C A`.

</details>

**Q8 (MCQ).** Keys `14, 17, 11, 7, 53, 4, 13, 12, 8` are inserted in order into an initially empty AVL tree. The preorder traversal of the result is  (A) 14 11 7 4 8 12 13 17 53  (B) 14 7 4 11 8 12 13 17 53  (C) 11 7 4 8 14 12 13 17 53  (D) 14 11 7 4 8 13 12 17 53

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** After 14, 17, 11, 7, 53: `14(11(7,-),17(-,53))`, balanced. Insert 4 (left of 7): node 11 has BF +2 (LL) → right rotation: `14(7(4,11),17(-,53))`. Insert 13 (right of 11): balanced. Insert 12 (left of 13): node 11 becomes BF -2 with a left-heavy right child (RL) → `12(11,13)` replaces 11: `14(7(4,12(11,13)),17(-,53))`. Insert 8: it goes left of 11. Now node 7 has left height 0 and right height 2 (BF -2), and its right child 12 is left-heavy: **RL at 7**: right-rotate 12 → `11(8,12(-,13))`, then left-rotate 7 → `11(7(4,8),12(-,13))`. Final tree `14(11(7(4,8),12(-,13)),17(-,53))`; preorder **14 11 7 4 8 12 13 17 53**.

</details>

**Q9 (MCQ).** The array `[3, 1, 6, 5, 2, 4]` is converted to a max-heap by bottom-up heapify. The result is  (A) 6 5 4 1 2 3  (B) 6 5 3 1 2 4  (C) 6 4 5 1 2 3  (D) 6 5 4 3 2 1

<details><summary>Answer</summary>

**Answer:** (A)  
**Solution:** n = 6; sift down positions 3, 2, 1. Position 3 (value 6; child 4): fine. Position 2 (value 1; children 5, 2): swap with 5 → `3 5 6 1 2 4`. Position 1 (value 3; children 5, 6): swap with 6 → `6 5 3 1 2 4`; position 3 now holds 3 with child 4: swap → `6 5 4 1 2 3`.

</details>

**Q10 (NAT).** The adjacency matrix `A` of the complete graph `K₄` satisfies `trace(A³) = 24`. How many triangles does `K₄` have, and how many nodes does its adjacency-list representation use in total (list nodes, not counting the 4 array heads)? Give "triangles,nodes".

<details><summary>Answer</summary>

**Answer:** 4,12  
**Solution:** Triangles `= trace(A³)/6 = 4` (also `C(4,3) = 4`). `K₄` has 6 edges; each is stored twice: 12 list nodes.

</details>

**Q11 (NAT, C + DS).** For the BST `50(30(20,40),70)` (50 root; 30 with children 20 and 40; 70 right child of 50), what does `f(root)` return?
```c
int f(struct node *t) {
    if (t == NULL) return 0;
    int a = f(t->left), b = f(t->right);
    return 1 + (a > b ? a : b);
}
```

<details><summary>Answer</summary>

**Answer:** 3  
**Solution:** `f` returns the height counted in **nodes**: leaves give 1; node 30 gives `1 + max(1,1) = 2`; node 70 gives 1; root gives `1 + max(2,1) = 3`. (In edges the height is 2.) The recursion makes one call per node plus one per NULL pointer: 5 + 6 = 11 calls.

</details>

**Q12 (MSQ).** Which statements are correct? (A) Inserting `1..n` in sorted order into an AVL tree gives height O(log n). (B) Build-heap on n elements takes Θ(n log n) in the worst case. (C) Binary search on a sorted singly linked list takes O(n). (D) Reaching the second largest key in a max-heap needs O(1). (E) Deleting the last node of a singly linked list with a tail pointer is O(1).

<details><summary>Answer</summary>

**Answer:** A, C, D  
**Solution:** (A) AVL stays balanced. (B) false, bottom-up build is O(n). (C) reaching the middle already costs O(n). (D) it is at index 2 or 3: two comparisons. (E) false: the predecessor of the tail must be found: O(n).

</details>

**Scoring.** ≥ 80% (10+ correct) → move on to [Algorithms](../08-algorithms/README.md). 60-80% → redo the traces you missed on paper. < 60% → re-read: Q1-Q4 → [arrays, stacks, queues](arrays-stacks-queues.md); Q5 → [linked lists](linked-lists.md); Q6-Q8, Q11 → [trees and BST](trees-and-bst.md); Q9 → [heaps](heaps.md); Q10 → [graphs](graphs.md); Q12 → [complexity reference](complexity-reference.md).
