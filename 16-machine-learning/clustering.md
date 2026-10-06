# Clustering: k-means, k-medoids and hierarchical clustering

> **Paper:** DA · **Priority:** P0 · **Plan topics:** Clustering; k-means; k-medoids; Hierarchical clustering: top-down; Hierarchical clustering: bottom-up; Single-linkage clustering; Multiple-linkage clustering
> **Prerequisites:** [ML foundations](ml-foundations.md) · [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) · **Leads to:** [Dimensionality reduction and PCA](dimensionality-reduction-pca.md)

## Quick glance

- **Clustering** = unsupervised grouping of unlabelled points so that points in a cluster are similar and clusters differ.
- **k-means** (Lloyd): repeat (1) assign each point to the nearest centroid, (2) set each centroid to the mean of its points, until assignments stop changing. Minimises $\sum_k\sum_{x\in C_k}\|x-\mu_k\|^2$ (within-cluster SSE). Each step never increases SSE, so it converges, but only to a **local** optimum that depends on the initialisation. Cost per iteration $O(nkd)$.
- **k-medoids (PAM)**: like k-means but each centre is an actual **data point** (the medoid); works with any dissimilarity and is more robust to outliers; costlier ($O(k(n-k)^2)$ per iteration).
- **Hierarchical**: *agglomerative (bottom-up)* starts with $n$ singleton clusters and repeatedly merges the closest pair; *divisive (top-down)* starts with one cluster and recursively splits. Output = **dendrogram**; cut at a height to get $k$ clusters.
- **Linkage** (distance between clusters): single = min pair, complete = max pair, average = mean of all pairs; centroid and Ward also exist. **Single linkage = MST-based (chaining)**; complete gives compact clusters.
- Agglomerative is $O(n^3)$ naively ($O(n^2\log n)$ with a heap), memory $O(n^2)$ for the distance matrix.
- #1 trap: "multiple linkage" in the syllabus means **the different linkage criteria** (complete, average, ...), not a separate algorithm.

## 1. What clustering is

No labels, no "right answer": we only have a notion of similarity (distance). The number of clusters $k$ is usually a choice. Uses: customer segmentation, image compression (colour quantisation), initial exploration, pre-processing.

| Family | Idea | $k$ given? | Shape |
| --- | --- | --- | --- |
| Partitional (k-means, k-medoids) | optimise an objective over a flat partition | yes | roughly spherical, similar size |
| Hierarchical | build a tree of nested clusters | no (cut the tree) | depends on linkage |

## 2. k-means

**Intuition.** Place $k$ flags; every point joins the nearest flag; move each flag to the centre of its crowd; repeat until nobody switches.

**Algorithm (Lloyd).**
1. Choose $k$ initial centroids.
2. **Assign**: $c_i=\arg\min_k\|x_i-\mu_k\|$.
3. **Update**: $\mu_k=\frac1{|C_k|}\sum_{x\in C_k}x$.
4. Stop when assignments (hence centroids) do not change.

**Worked example 1 (1-D).** Data $\{2,4,10,12,3,20,30,11,25\}$, $k=2$, initial centroids $2$ and $4$.

| Iter | Centroids | Cluster 1 | Cluster 2 | New centroids |
| --- | --- | --- | --- | --- |
| 1 | 2, 4 | $\{2,3\}$ | $\{4,10,12,20,30,11,25\}$ | $2.5,\ 16$ |
| 2 | 2.5, 16 | $\{2,4,3\}$ | $\{10,12,20,30,11,25\}$ | $3,\ 18$ |
| 3 | 3, 18 | $\{2,4,10,3\}$ | $\{12,20,30,11,25\}$ | $4.75,\ 19.6$ |
| 4 | 4.75, 19.6 | $\{2,4,10,12,3,11\}$ | $\{20,30,25\}$ | $7,\ 25$ |
| 5 | 7, 25 | same | same | $7,\ 25$ — **converged** |

(In iteration 1 the point $3$ is a tie, $|3-2|=|3-4|=1$; the table breaks it toward the first centroid. Later iterations have no ties.) Final SSE: cluster 1 (mean 7): $25+9+9+25+16+16=100$ for $\{2,4,10,12,3,11\}$ in order of deviations $-5,-3,3,5,-4,4$; cluster 2 (mean 25): $25+0+25=50$. **SSE $=150$.**

**Worked example 2 (2-D).** Points A(1,1), B(1,2), C(2,1), D(6,5), E(7,5), F(6,6); $k=2$, initial centroids A and D.
1. Distances to $(1,1)$: A 0, B 1, C 1, D 6.40, E 7.21, F 7.07. To $(6,5)$: A 6.40, B 5.83, C 5.66, D 0, E 1, F 1. Assign: $\{A,B,C\}$, $\{D,E,F\}$.
2. New centroids: $\mu_1=(\tfrac43,\tfrac43)$, $\mu_2=(\tfrac{19}3,\tfrac{16}3)$.
3. Re-assign: nothing changes (each point is still nearest its own centroid). **Converged** after 2 iterations.
4. SSE: cluster 1: $\|A-\mu_1\|^2=\tfrac19+\tfrac19=\tfrac29$; $B$: $\tfrac19+\tfrac49=\tfrac59$; $C$: $\tfrac49+\tfrac19=\tfrac59$; total $\tfrac{12}9$. Cluster 2 likewise $\tfrac{12}9$. **SSE $=2.667$.**

A different start (A and B as centroids) goes $\{A,C\},\{B,D,E,F\}\to\{A,B,C\},\{D,E,F\}$ and reaches the same answer here; with unlucky starts on harder data k-means can end in a worse local minimum.

**Properties.**
- Objective decreases monotonically; finite states $\Rightarrow$ terminates. Result is a **local** minimum; run several random restarts or use **k-means++** seeding (spread-out initial centroids).
- Sensitive to **outliers** (mean is not robust), to feature scaling, and to the choice of $k$. Assumes convex, similarly sized clusters.
- **Choosing $k$**: **elbow method** — plot SSE vs $k$ (SSE always decreases as $k$ grows; $k=n$ gives 0); pick the bend. Silhouette score is another criterion.
- Cost $O(nkd)$ per iteration; $k=1$ gives centroid = global mean.
- Every cluster boundary is a straight line/hyperplane (Voronoi cells).

## 3. k-medoids (PAM)

**Intuition.** Same as k-means, but each cluster is represented by its most central **actual data point**, the **medoid**, which minimises the sum of dissimilarities to the other members. Only pairwise dissimilarities are needed (any distance, even non-Euclidean, categorical data).

**PAM algorithm.**
1. Pick $k$ points as medoids; assign each point to its nearest medoid; cost $=\sum_i d(x_i,\text{medoid}(i))$.
2. For each (medoid $m$, non-medoid $o$) pair, compute the cost of swapping; apply the swap that reduces cost most.
3. Repeat until no swap lowers the cost.

**Worked example.** Points $\{1,2,3,10,11,12\}$ on a line, $k=2$, $d=|x-y|$, initial medoids $\{1,10\}$.
- Cost: $1\to0$, $2\to1$, $3\to2$, $10\to0$, $11\to1$, $12\to2$: total $6$.
- Swap $1\to2$ (medoids $\{2,10\}$): $1,0,1,0,1,2=5$. Better; accept.
- Swap $10\to11$ (medoids $\{2,11\}$): $1,0,1,1,0,1=4$. Better; accept.
- No further swap helps ($\{2,11\}$ is the optimum, cost 4). Clusters $\{1,2,3\}$, $\{10,11,12\}$.

**Robustness.** $\{1,2,3,4,100\}$ with $k=1$: the mean is $22$ (no data point near it; dragged by the outlier); the medoid is $3$ (cost $2+1+0+1+97=101$; medoid $4$ would cost $102$). With $k\ge2$ PAM may spend a medoid on the outlier itself, so "robust" means the *centre* is not distorted, not that outliers are ignored.

| | k-means | k-medoids |
| --- | --- | --- |
| Centre | mean (need not be a data point) | a data point |
| Distance | squared Euclidean | any dissimilarity |
| Outliers | sensitive | more robust |
| Cost / iteration | $O(nkd)$ | $O(k(n-k)^2)$ |

## 4. Hierarchical clustering

### 4.1 Bottom-up (agglomerative) and top-down (divisive)

**Agglomerative (bottom-up).**
1. Start with $n$ singleton clusters; compute the $n\times n$ distance matrix.
2. Merge the two closest clusters (by the chosen linkage).
3. Update the distances from the new cluster to every other cluster.
4. Repeat until one cluster remains ($n-1$ merges). Record the merge distance for the dendrogram.

**Divisive (top-down).** Start with all points in one cluster; repeatedly split a cluster into two (e.g. by running 2-means on it, or splitting off the most dissimilar point) until singletons or the desired $k$. Optimal divisive search is exponential ($2^{n-1}-1$ ways to split), so heuristics are used. Agglomerative is far more common.

**Dendrogram.** A tree whose leaves are points; the height at which two branches join is the merge distance. **Cutting at height $h$** gives the clusters formed by merges with distance $\le h$; the number of clusters equals $n-$(number of merges below $h$). Lance-Williams updates make linkage updates cheap.

### 4.2 Linkage criteria ("multiple-linkage")

The Plan's *single-linkage* and *multiple-linkage* topics together mean: **the way we define the distance between two clusters $A,B$**.

| Linkage | $d(A,B)$ | Behaviour |
| --- | --- | --- |
| **Single** | $\min_{a\in A,b\in B}d(a,b)$ | chaining: clusters stretch into long chains; can handle non-spherical shapes; sensitive to noise bridges |
| **Complete** | $\max_{a,b}d(a,b)$ | compact, similar-diameter clusters; sensitive to outliers |
| **Average** (UPGMA) | $\frac1{|A||B|}\sum_{a,b}d(a,b)$ | compromise |
| Centroid | $d(\mu_A,\mu_B)$ | can produce inversions (merge heights not monotone) |
| Ward | merge with the smallest increase in total within-cluster SSE | compact, k-means-like |

**Worked example.** Five points on a line at $a=0,\ b=1,\ c=3,\ d=7,\ e=8$. Distance matrix:

| | a | b | c | d | e |
| --- | --- | --- | --- | --- | --- |
| a | 0 | 1 | 3 | 7 | 8 |
| b | 1 | 0 | 2 | 6 | 7 |
| c | 3 | 2 | 0 | 4 | 5 |
| d | 7 | 6 | 4 | 0 | 1 |
| e | 8 | 7 | 5 | 1 | 0 |

*Single linkage.*
1. Smallest is $1$ ($a$-$b$ and $d$-$e$; tie, take $a$-$b$): merge $\{a,b\}$ at height 1. Distances: to $c$: $\min(3,2)=2$; to $d$: $\min(7,6)=6$; to $e$: $\min(8,7)=7$.
2. Smallest now is $d$-$e=1$: merge $\{d,e\}$ at 1. To $c$: $\min(4,5)=4$; to $\{a,b\}$: $\min(6,7)=6$.
3. Smallest: $c$-$\{a,b\}=2$: merge $\{a,b,c\}$ at 2. To $\{d,e\}$: $\min(4,6)=4$.
4. Merge $\{a,b,c\}$ and $\{d,e\}$ at 4.
Heights: $1,1,2,4$.

*Complete linkage.* Steps 1, 2 same (height 1, 1). Then $c$ to $\{a,b\}$: $\max(3,2)=3$; $c$ to $\{d,e\}$: $\max(4,5)=5$; $\{a,b\}$-$\{d,e\}$: $\max=8$. Merge $\{a,b,c\}$ at **3**. Final merge at $\max(7,8,4,5,\dots)=\mathbf8$ (farthest pair $a$-$e$). Heights: $1,1,3,8$.

*Average linkage.* $c$ to $\{a,b\}$: $(3+2)/2=2.5$ (merge at 2.5); $c$ to $\{d,e\}$: $4.5$. Final: mean of six cross distances $(7+8+6+7+4+5)/6 = 37/6=6.17$. Heights: $1,1,2.5,6.17$.

The 2-cluster cut is $\{a,b,c\},\{d,e\}$ for all three here. A case where linkages **disagree**: points $0,2,5,9$.
- Single: merges $(0,2)$ at 2; $5$ joins $\{0,2\}$ at $\min(5,3)=3$; $9$ joins at $\min(9,7,4)=4$. Two clusters: $\{0,2,5\},\{9\}$.
- Complete: $(0,2)$ at 2; then $d(5,\{0,2\})=5$, $d(5,9)=4$ $\Rightarrow$ merge $(5,9)$ at 4. Two clusters: $\{0,2\},\{5,9\}$. Final merge at $\max=9$.

**Single linkage = MST.** The merges of single-linkage agglomerative clustering are exactly the edges chosen by **Kruskal's algorithm** on the complete graph weighted by distance, in increasing order of weight. Cutting the dendrogram into $k$ clusters = deleting the $k-1$ heaviest edges of the MST. See [minimum spanning trees](../08-algorithms/minimum-spanning-trees.md). In the example the MST edges are $a$-$b$ (1), $d$-$e$ (1), $b$-$c$ (2), $c$-$d$ (4): merge heights $1,1,2,4$ ✓.

**Chaining.** Single linkage joins clusters through one close pair, so a thin line of points bridges two blobs and they merge into one elongated cluster. Complete linkage avoids this but breaks large clusters and over-reacts to outliers.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| k-means objective | $\sum_k\sum_{x\in C_k}\|x-\mu_k\|^2$ | SSE questions |
| Centroid update | mean of members | iteration trace |
| k-means cost | $O(nkd)$ per iteration | complexity |
| k-medoids cost | $O(k(n-k)^2)$ per iteration | complexity |
| Merges in agglomerative | $n-1$ | count questions |
| Single | min pair distance | MST / chaining |
| Complete | max pair distance | compact clusters |
| Average | mean of all pairs | UPGMA |
| Divisive splits | $2^{n-1}-1$ possible first splits | why heuristics |
| Agglomerative cost | $O(n^3)$ naive, $O(n^2\log n)$ with heap, memory $O(n^2)$ | complexity |

## GATE traps

- k-means centroids are **means** and need not be data points; k-medoids centres must be data points.
- k-means finds a **local** minimum; different initialisations can give different answers. Convergence is guaranteed, global optimality is not.
- SSE decreases as $k$ increases; "minimum SSE" is not a criterion for picking $k$ (use elbow/silhouette).
- Ties in distance: state the tie-breaking rule used; GATE usually avoids ties.
- Single linkage uses the **minimum**, complete the **maximum** — easy to swap. After a merge, update the matrix with $\min$ (single) or $\max$ (complete) of the old entries, not with recomputed centroids.
- The number of clusters after cutting an agglomerative dendrogram: $n-\#\{\text{merges done}\}$.
- Single-linkage merge heights equal the sorted MST edge weights (for Kruskal's order).
- "Multiple linkage" is not an algorithm: it refers to the family of linkage criteria.
- Hierarchical clustering is deterministic (no initialisation), but not scalable: $O(n^2)$ memory.

## Connections

- [ML foundations](ml-foundations.md) — unsupervised vs supervised; no labels means no test error, only internal measures like SSE.
- [Minimum spanning trees](../08-algorithms/minimum-spanning-trees.md) — single-linkage dendrogram = Kruskal's merge order.
- [Graph traversals](../08-algorithms/graph-traversals.md) — connected components of the threshold graph are the single-linkage clusters at that height.
- [Greedy algorithms](../08-algorithms/greedy-algorithms.md) — agglomerative clustering is a greedy closest-pair merge.
- [Classification methods](classification-methods.md) — k-NN also uses distances; k-means centroids with nearest-centroid assignment resemble a nearest-prototype classifier.
- [Dimensionality reduction and PCA](dimensionality-reduction-pca.md) — cluster in a PCA-reduced space to beat the curse of dimensionality.
- [Statistical inference](../02-probability-statistics/statistical-inference.md) — k-means is the hard-assignment limit of fitting a Gaussian mixture by maximum likelihood.

## Practice

**Q1 (NAT).** k-means, $k=2$, 1-D data $\{1,2,9,10\}$, initial centroids $1$ and $2$. What is the final SSE?

<details><summary>Answer</summary>

**Answer:** 1. **Solution:** Iter 1: $1\to c_1$; $2\to c_2$ (tie-free: distance 1 vs 0); $9,10\to c_2$. Clusters $\{1\},\{2,9,10\}$; centroids $1$, $7$. Iter 2: $2$: $|2-1|=1<|2-7|=5\to c_1$; $9,10\to c_2$. Clusters $\{1,2\},\{9,10\}$, centroids $1.5,\ 9.5$. Iter 3: no change. SSE $=0.25+0.25+0.25+0.25=1$.

</details>

**Q2 (MCQ).** Which clustering method needs the number of clusters in advance? (a) agglomerative single linkage (b) k-means (c) divisive (d) all.

<details><summary>Answer</summary>

**Answer:** (b). **Solution:** hierarchical methods produce the whole dendrogram; $k$ is chosen later by cutting.

</details>

**Q3 (NAT).** Single-linkage agglomerative clustering on points $1,2,4,8,16$ (1-D). At what distance do the last two clusters merge?

<details><summary>Answer</summary>

**Answer:** 8. **Solution:** gaps are $1,2,4,8$; single linkage merges in order of MST edges (consecutive gaps): $1,2,4,8$. The last merge joins $\{1,2,4,8\}$ and $\{16\}$ at $16-8=8$.

</details>

**Q4 (NAT).** Same points $1,2,4,8,16$ with complete linkage. Last merge distance?

<details><summary>Answer</summary>

**Answer:** 15. **Solution:** merges: $(1,2)$ at 1; then $d(\{1,2\},4)=3$, $d(4,8)=4$, so $\{1,2,4\}$ at 3; then $d(\{1,2,4\},8)=7$, $d(8,16)=8$: merge $\{1,2,4,8\}$ at 7; finally $\{1,2,4,8\}$ with $16$: $\max=16-1=15$.

</details>

**Q5 (MSQ).** Which are true? (a) k-means always converges to the global optimum. (b) k-medoids centres are data points. (c) Single linkage can suffer chaining. (d) Agglomerative clustering on $n$ points performs $n-1$ merges.

<details><summary>Answer</summary>

**Answer:** (b), (c), (d). **Solution:** (a) false: only local optimum.

</details>

**Q6 (NAT).** Complete linkage cluster distances: after merging, clusters $A=\{p,q\}$ and $B=\{r\}$ have $d(p,r)=4$, $d(q,r)=9$. Distance $d(A,B)$ under single, complete and average linkage? Give the sum of the three.

<details><summary>Answer</summary>

**Answer:** 19.5. **Solution:** single $=4$, complete $=9$, average $=6.5$; sum $=19.5$.

</details>

**Q7 (MCQ).** Cutting a dendrogram of 20 points just above the 17th merge (merges counted from the bottom) yields how many clusters? (a) 3 (b) 17 (c) 2 (d) 20.

<details><summary>Answer</summary>

**Answer:** (a). **Solution:** after 17 merges, $20-17=3$ clusters remain.

</details>

**Q8 (NAT).** Points in 2-D: $(0,0),(0,2),(5,0)$. One k-means iteration with initial centroids $(0,0)$ and $(5,0)$. What is the new first centroid?

<details><summary>Answer</summary>

**Answer:** $(0,1)$. **Solution:** $(0,2)$ is at distance 2 from $(0,0)$ and $\sqrt{29}$ from $(5,0)$, so cluster 1 $=\{(0,0),(0,2)\}$ with mean $(0,1)$; cluster 2 $=\{(5,0)\}$.

</details>
