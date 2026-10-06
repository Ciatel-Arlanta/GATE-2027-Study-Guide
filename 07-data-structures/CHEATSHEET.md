# Data Structures: Cheat sheet

Chapters: [arrays/stacks/queues](arrays-stacks-queues.md) · [linked lists](linked-lists.md) · [trees/BST/AVL](trees-and-bst.md) · [heaps](heaps.md) · [graphs](graphs.md) · [complexity](complexity-reference.md).

## Arrays and special storage

| Item | Formula |
|---|---|
| 1-D | `base + (i-L)·w` |
| Row-major | `base + ((i-L1)·N2 + (j-L2))·w`, `N2` = #columns |
| Column-major | `base + ((j-L2)·N1 + (i-L1))·w`, `N1` = #rows |
| Lower-triangular (1-based, row-major) | offset `i(i-1)/2 + j - 1`; size `n(n+1)/2` |
| Tridiagonal | `3n-2` cells; offset `2i + j - 3` (1-based, row-major) |
| Elements in `A[L..U]` | `U - L + 1` |

## Stacks and queues

| Item | Fact |
|---|---|
| Stack | LIFO; push/pop/peek O(1) |
| Infix to postfix | operand to output; operator pops higher or equal precedence (equal only if left-assoc.; `^` is right-assoc.); `(` push, `)` pop to `(` |
| Postfix evaluation | first-popped is the right operand |
| Valid stack permutations of `1..n` | Catalan: 1, 2, 5, 14, 42; invalid iff a 312 pattern (c,a,b with a<b<c) exists |
| Circular queue (size N) | full `(rear+1)%N == front`; empty `front == rear`; count `(rear-front+N)%N`; capacity N-1 |
| Queue from 2 stacks | amortised O(1); one dequeue worst O(n) |
| Stack from queues | push or pop must be O(n) |
| Deque | insert/delete both ends O(1) |

## Linked lists

| Item | Fact |
|---|---|
| Head insert/delete | O(1) |
| Tail insert | O(n) (O(1) with tail pointer); tail delete O(n) singly even with tail |
| Delete given node | doubly O(1); singly: copy next's data (not for last node) |
| Reverse | `next=curr->next; curr->next=prev; prev=curr; curr=next;` O(n), O(1) space |
| Middle | slow/fast; index `floor(n/2)` (0-based) |
| Floyd | meet inside cycle; head and meeting point step by 1: meet at cycle start |
| Merge sorted | O(m+n), O(1) space, dummy node |
| Order rule | link new node's `next` before changing the predecessor's `next` |

## Trees

| Item | Fact |
|---|---|
| Edges | `n-1` |
| Binary tree | `n0 = n2 + 1`; NULL pointers `n+1` |
| Height h (edges): nodes | between `h+1` and `2^(h+1)-1`; level `l` has ≤ `2^l` |
| Complete tree height | `floor(log2 n)` |
| k-ary full, i internal | `n = ki+1`, leaves `(k-1)i+1` |
| Traversals | pre NLR, in LNR, post LRN, level = queue |
| Reconstruct | in + pre or in + post unique; pre + post unique only if full; `2^k` trees (k = single-child nodes) |
| Count of trees/BSTs, n nodes | Catalan `C(2n,n)/(n+1)`: 1,2,5,14,42 |
| BST | inorder sorted; ops O(h): avg log n, worst n |
| BST delete | leaf; one child; two children: inorder successor (min of right) |
| AVL | `|BF| ≤ 1`; LL→right rot, RR→left rot, LR→left at child then right, RL→right at child then left |
| AVL min nodes | `N(h) = N(h-1)+N(h-2)+1`: 1, 2, 4, 7, 12, 20, 33, 54 (h = 0..7) |
| AVL height | `≤ 1.44 log2(n+2)` |
| Insertion rotations | at most one (single or double) |

## Heaps

| Item | Fact |
|---|---|
| Indices 1-based / 0-based | parent `i/2`, children `2i, 2i+1` / parent `(i-1)/2`, children `2i+1, 2i+2` |
| Height / leaves | `floor(log2 n)` / positions `floor(n/2)+1..n` |
| Insert (sift-up) / extract (sift-down) | O(log n) |
| Find max | O(1); search O(n); min of max-heap in a leaf |
| Build-heap | O(n) (sift down from `n/2` to 1); n inserts O(n log n) |
| Heap sort | Θ(n log n), in place, not stable |
| Distinct max-heaps n = 1..7 | 1, 1, 2, 3, 8, 20, 80 |
| d-ary | height `log_d n`; insert O(log_d n); extract O(d log_d n) |

## Graphs

| Item | Fact |
|---|---|
| Handshake | `Σ deg = 2e`; directed `Σ in = Σ out = e` |
| Max edges | undirected `n(n-1)/2`; directed `n(n-1)` |
| Tree/connected | connected needs `≥ n-1` edges |
| Matrix | Θ(n²) space; edge test O(1); neighbours Θ(n) |
| List | Θ(n+e) space (2e nodes undirected); edge test O(deg) |
| BFS/DFS | O(n²) matrix; O(n+e) list |
| Dijkstra / Prim | O(n²) array; O((n+e) log n) heap |
| `(A^k)[i][j]` | number of walks of length k; `trace(A³)/6` = triangles |
| Spanning trees of K_n | `n^(n-2)` |

## Complexity master table (worst case unless noted)

| Structure | Access | Search | Insert | Delete | Min |
|---|---|---|---|---|---|
| Unsorted array | 1 | n | 1 (end) | n | n |
| Sorted array | 1 | log n | n | n | 1 |
| Singly list | n | n | 1 (head) | n | n |
| Doubly list | n | n | 1 | 1 (given node) | n |
| Stack/queue | n/a | n | 1 | 1 | n |
| BST | h | h (avg log n, worst n) | h | h | h |
| AVL | log n | log n | log n | log n | log n |
| Min-heap | n/a | n | log n | log n | 1 |
| Hash (chaining) | n/a | 1 avg / n worst | 1 avg | 1 avg | n |

Other: build-heap O(n); BST from sorted input O(n²); dynamic array append amortised O(1); chained hash expected chain `α = n/M`; open addressing unsuccessful probes `1/(1-α)`.
