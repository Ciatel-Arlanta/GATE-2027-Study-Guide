# Brief: 05-c-programming + 07-data-structures

Compile and run every C example with gcc or clang if one is installed (check `gcc --version` / `clang --version`). If no compiler is available, trace each program by hand twice and be certain of the output. Avoid examples whose output depends on undefined behaviour unless the point is to teach that it is undefined.

## 1) 05-c-programming/ (CS; part of "Programming and Data Structures", ~10-12 marks together with DS)
Plan topics: C syntax and semantics; Pointers; Arrays in C; Structures; Functions; Recursion in C; Memory and pointer reasoning; Tracing C programs and expressions.
Files: README.md, c-basics-and-expressions.md, pointers-arrays-strings.md, functions-recursion-structures.md, CHEATSHEET.md, CHECKPOINT.md.
Must cover:
- Data types and sizes (assume a typical 64-bit model and state it), integer promotion, signed/unsigned conversion traps (e.g. `-1 > 1u`), overflow, implicit/explicit casts, char arithmetic.
- Operator precedence and associativity table; short-circuit evaluation; pre/post increment; comma operator; ternary; bitwise ops and shifts; sequence points and undefined behaviour (`i = i++`) — GATE avoids UB, so teach how to recognise it.
- Control flow (switch fall-through, break/continue, loop tracing), printf format specifiers, octal/hex literals.
- Storage classes (auto, static, extern, register), scope and lifetime, static local variables across calls (a GATE favourite), global vs local shadowing, static vs dynamic scoping (GATE asks this — show the same program under both).
- Pointers: declaration, `&`/`*`, pointer arithmetic scaled by type size, pointer to pointer, pointers and arrays (`a[i] == *(a+i) == i[a]`), array decay, `sizeof` on arrays vs pointers, 2D arrays and row-major address calculation, array of pointers vs pointer to array, `const` with pointers, function pointers, void pointers, NULL, dangling pointers, memory leaks.
- Strings: char arrays vs string literals, `'\0'`, strlen vs sizeof, common string-function tracing.
- Functions: call by value vs simulated call by reference, swap examples, returning pointers to locals (bug), activation records and the call stack (link 12-compiler-design/runtime-environments.md), parameter-passing modes GATE asks about (value, reference, value-result, name) with one program traced under each.
- Recursion: tracing trees, counting calls, printing order (head vs tail recursion), recursion that returns values (e.g. f(n) = f(n-1) + f(n-2) call count), static variables inside recursion, converting recursion to recurrence (link 08-algorithms).
- Structures and unions: layout, padding/alignment (state assumptions), `->` vs `.`, self-referential structs (linked-list node), typedef; dynamic memory: malloc/calloc/realloc/free.

## 2) 07-data-structures/ (CS+DA)
Plan topics: Arrays; Stacks; Queues; Linked lists; Trees; Binary search trees; Binary heaps; Graphs; Data-structure operations and complexity. (DA also lists hash tables — hashing itself lives in 08-algorithms/hashing.md; link to it.)
Files: README.md, arrays-stacks-queues.md, linked-lists.md, trees-and-bst.md, heaps.md, graphs.md, complexity-reference.md, CHEATSHEET.md, CHECKPOINT.md.
Must cover:
- Arrays: row-major/column-major address formulas (incl. non-zero lower bounds), triangular/sparse matrix storage.
- Stacks: array and linked implementation, infix->postfix/prefix conversion with an operator-stack trace table, postfix evaluation trace, balanced parentheses, recursion via stack, number of valid stack permutations (Catalan), stack-permutation checks.
- Queues: linear vs circular (full/empty conditions with front/rear), deque, priority queue (link heaps), queue using two stacks (amortised cost), stack using queues.
- Linked lists: singly/doubly/circular, insertion/deletion code in C, reversal (iterative and recursive), finding middle, cycle detection (Floyd), merging sorted lists, typical GATE code-completion questions.
- Trees: terminology (state the height convention and the alternative), binary-tree properties (max nodes 2^(h+1)-1, leaves = internal nodes of degree 2 + 1, n0 = n2 + 1), full/complete/perfect, number of distinct binary trees / BSTs with n nodes (Catalan), traversals (pre/in/post/level) with reconstruction from inorder+preorder or inorder+postorder (and why preorder+postorder is ambiguous), expression trees, threaded trees (brief).
- BST: search/insert/delete (all three delete cases), inorder = sorted, worst case skewed, range of heights, AVL trees (balance factor, four rotations worked, min nodes in AVL of height h: N(h) = N(h-1) + N(h-2) + 1).
- Heaps: array representation and index formulas (0- and 1-based), heapify (sift-down), build-heap in O(n) and why, insert/extract/decrease-key, heap sort link, k-th smallest, min/max element positions, number of distinct heaps for small n, d-ary heaps.
- Graphs: terminology, adjacency matrix vs list (space and operation costs), incidence matrix, edge-list; how representation changes BFS/DFS/Dijkstra complexity.
- Complexity reference: one master table of every operation (access/search/insert/delete/min/merge) for arrays, sorted arrays, linked lists, stacks, queues, BST (avg/worst), AVL, binary heap, hash table (avg/worst), adjacency matrix/list.
