# Artificial Intelligence: cheat sheet

Chapters: [search](search.md) · [logic and inference](logic-and-inference.md) · [probabilistic reasoning](probabilistic-reasoning.md).

## Search

| Algorithm | Frontier | Complete | Optimal | Time | Space |
| --- | --- | --- | --- | --- | --- |
| BFS | FIFO | yes | equal step costs | $b^d$ | $b^d$ |
| UCS | min $g$ | if cost $\ge\varepsilon$ | yes | $b^{1+\lfloor C^*/\varepsilon\rfloor}$ | same |
| DFS | LIFO | no (cycles / infinite); yes finite graph search | no | $b^m$ | $bm$ |
| Depth-limited | LIFO + limit $\ell$ | no if $\ell<d$ | no | $b^\ell$ | $b\ell$ |
| IDS | repeated DLS | yes | equal step costs | $b^d$ | $bd$ |
| Bidirectional | two BFS | yes | unit cost | $b^{d/2}$ | $b^{d/2}$ |
| Greedy | min $h$ | no | no | $b^m$ | $b^m$ |
| A\* | min $f=g+h$ | yes | admissible (tree) / consistent (graph) | exp. | exp. |

- IDS nodes: $(d+1)+d\,b+(d-1)b^2+\dots+b^d$. Goal test at **expansion** for UCS and A\*.
- Admissible: $h\le h^*$. Consistent: $h(n)\le c(n,n')+h(n')$; consistent $\Rightarrow$ admissible; $f$ non-decreasing along paths.
- $h=0$: A\* = UCS. Dominance $h_2\ge h_1$: fewer expansions. Inadmissible $h$ may give a suboptimal path. IDA\* = $f$-cutoff DFS.
- Local search: hill climbing (stuck at local max), simulated annealing (accept worse w.p. $e^{\Delta E/T}$), beam search.
- Minimax: MAX = max, MIN = min; $O(b^m)$ time, $O(bm)$ space. **Alpha-beta**: prune when $\alpha\ge\beta$; same root value; best case $O(b^{m/2})$, worst $O(b^m)$; first-child leaves are never pruned. Expectiminimax: chance node = expected value.

## Logic

| Item | Fact |
| --- | --- |
| Entailment | $KB\models\alpha$ iff $KB\wedge\neg\alpha$ unsat; truth table $2^n$ |
| Valid / sat / unsat | all / some / no models |
| Resolution | $\ell\vee A,\ \neg\ell\vee B\vdash A\vee B$; refutation-complete; derive $\square$ |
| CNF steps | remove $\leftrightarrow,\to$; push $\neg$ (De Morgan); distribute $\vee$ over $\wedge$ |
| Horn clause | $\le1$ positive literal; chaining linear, sound and complete |
| Forward / backward chaining | data-driven (counters + agenda) / goal-driven |
| FOL translation | $\forall\ \to$ implication; $\exists\ \wedge$ conjunction; $\forall\exists\ne\exists\forall$ |
| Unification | MGU; occurs check; $P(x,x)$ vs $P(a,b)$ fails; $x$ vs $f(x)$ fails |
| Skolemisation | $\exists y$ under $\forall x$ $\to$ $f(x)$; none $\to$ constant |
| Decidability | propositional: decidable (NP-complete SAT); FOL: semi-decidable |

## Probabilistic reasoning

| Item | Fact |
| --- | --- |
| Conditional independence | $P(x,y\mid z)=P(x\mid z)P(y\mid z)$ |
| BN joint | $\prod_iP(x_i\mid\text{pa}_i)$ |
| Parameters (binary) | $\sum2^{|\text{Pa}_i|}$ vs $2^n-1$ |
| d-separation | chain/fork blocked iff middle observed; collider blocked iff it and descendants unobserved |
| Markov blanket | parents + children + children's other parents |
| Explaining away | observing a common effect makes causes dependent |
| Enumeration | sum joint over hidden vars, normalise |
| Variable elimination | restrict evidence, multiply factors, sum out hidden; order affects cost; polytree linear, general NP-hard |
| Prior sampling | sample in topological order |
| Rejection | discard samples inconsistent with evidence; accept rate $P(e)$ |
| Likelihood weighting | fix evidence; weight $=\prod P(e\mid\text{pa}(e))$ |
| Gibbs | resample one non-evidence variable from $P(X\mid MB(X))$; burn-in; MCMC |
| Naive Bayes | one-parent BN: $1+2d$ params for binary features |

## Burglary-alarm numbers

$P(B)=0.001$, $P(E)=0.002$; $P(A\mid B,E)$: TT .95, TF .94, FT .29, FF .001; $P(J\mid A)$: .90 / .05; $P(M\mid A)$: .70 / .01. $P(b\mid j,m)=0.284$; $P(j,m)=0.00208$; $P(b\mid a)=0.374$, $P(b\mid a,e)=0.0033$.

## Remember

- BFS = shallowest, not cheapest. UCS/A\* test the goal when popped.
- Tie-breaking decides expansion order: use the stated rule.
- Negate the goal in resolution.
- Collider observed (or descendant) = dependent.
- Normalise after enumeration/VE.

[Back to section README](README.md) · [Checkpoint](CHECKPOINT.md)
