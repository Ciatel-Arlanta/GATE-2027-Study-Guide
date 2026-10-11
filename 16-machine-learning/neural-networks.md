# Neural networks: perceptron, MLP and feed-forward networks

> **Paper:** DA · **Priority:** P1 · **Plan topics:** Multi-layer perceptron; Feed-forward neural networks
> **Prerequisites:** [Linear and logistic regression](linear-and-logistic-regression.md) · [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) (chain rule, gradients) · **Leads to:** [Clustering](clustering.md) (end of supervised methods) · [CHECKPOINT](CHECKPOINT.md)
> **PYQ weightage:** High · 4 Qs (DA 2024–2026 only) · Trend: steady · Most asked: Activation functions (ReLU, Sigmoid properties & gradients), Feedforward propagation numericals

## Quick glance

- **Exam focus:** In GATE DA, neural networks questions test properties of activation functions (ReLU derivative and non-differentiability at 0, sigmoid derivative $\sigma(1-\sigma)$, vanishing gradients) and step-by-step forward pass evaluation with ReLU/Sigmoid units, as MSQ and NAT questions.
- **Neuron**: $a = \phi(w^Tx+b)$ — a weighted sum then a nonlinearity $\phi$. A **perceptron** uses a step $\phi$.
- **Perceptron rule**: if prediction $\hat y\ne y$, $w\leftarrow w+\eta(y-\hat y)x$, $b\leftarrow b+\eta(y-\hat y)$. Converges **iff data are linearly separable**. A single perceptron **cannot learn XOR**.
- **MLP / feed-forward network**: layers of neurons, signals flow input $\to$ hidden $\to$ output with no cycles. Forward pass: $a^{(l)}=\phi(W^{(l)}a^{(l-1)}+b^{(l)})$.
- **Parameters** of a layer with $m$ inputs and $n$ units: $mn$ weights $+\,n$ biases $=(m+1)n$.
- **Backpropagation** = chain rule applied layer by layer; gradient descent step $\theta\leftarrow\theta-\eta\nabla L$.
- Activations: sigmoid $\sigma'=\sigma(1-\sigma)$, tanh $'=1-\tanh^2$, ReLU $'=1[z>0]$, softmax for multiclass outputs (with cross-entropy: output error $=\hat y-y$).
- With **linear** activations any depth collapses to one linear model: nonlinearity is essential.
- Universal approximation: one hidden layer with enough nonlinear units approximates any continuous function on a compact set.
- #1 trap: confusing the number of *layers* (does the input count?) with the number of *weight matrices*; count parameters as $(\text{inputs}+1)\times\text{units}$ per layer.

## 1. The perceptron [PYQ: Medium]

**Intuition.** A perceptron draws one hyperplane and says "positive side / negative side".

**Model.** $\hat y = 1$ if $w^Tx+b\ge0$, else $0$ (labels in $\{0,1\}$).

**Learning rule** (per example, learning rate $\eta$; we use $\eta=1$):

$$w\leftarrow w+\eta\,(y-\hat y)\,x,\qquad b\leftarrow b+\eta\,(y-\hat y)$$

No change when correct; add $x$ when it wrongly predicted 0; subtract $x$ when it wrongly predicted 1.

**Convergence theorem.** If the data are linearly separable, the perceptron algorithm finds a separating hyperplane in a finite number of updates. If not, it never converges (it cycles).

**Worked example: learning OR**, starting $w=(0,0)$, $b=0$, $\eta=1$, visiting $(0,0),(0,1),(1,0),(1,1)$ in order.

| Epoch | Input | $(w_1,w_2,b)$ before | $w^Tx+b$ | $\hat y$ | $y$ | Update |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | (0,0) | (0,0,0) | 0 | 1 | 0 | $e=-1$: $b=-1$ |
| 1 | (0,1) | (0,0,-1) | -1 | 0 | 1 | $e=+1$: $w=(0,1)$, $b=0$ |
| 1 | (1,0) | (0,1,0) | 0 | 1 | 1 | none |
| 1 | (1,1) | (0,1,0) | 1 | 1 | 1 | none |
| 2 | (0,0) | (0,1,0) | 0 | 1 | 0 | $b=-1$ |
| 2 | (0,1) | (0,1,-1) | 0 | 1 | 1 | none |
| 2 | (1,0) | (0,1,-1) | -1 | 0 | 1 | $w=(1,1)$, $b=0$ |
| 2 | (1,1) | (1,1,0) | 2 | 1 | 1 | none |
| 3 | (0,0) | (1,1,0) | 0 | 1 | 0 | $b=-1$ |
| 3 | (0,1),(1,0),(1,1) | (1,1,-1) | 0,0,1 | 1,1,1 | 1 | none |

Final $w=(1,1)$, $b=-1$: check $(0,0)\to-1\to0$ ✓, $(0,1)\to0\to1$ ✓, $(1,0)\to1$ ✓, $(1,1)\to1$ ✓. Epoch 4 would make no updates: converged. (The step at exactly $0$ is classified as 1; a different tie convention changes the trace but not the convergence.)

**XOR is not linearly separable**: $(0,0),(1,1)\to0$ and $(0,1),(1,0)\to1$. No single line separates them, so no perceptron works. A hidden layer fixes it (next section).

## 2. Multi-layer perceptron (MLP) and the feed-forward architecture [PYQ: High]

**Intuition.** Hidden units compute new features of the input; the next layer draws a line in that *feature space*. Stacking nonlinear layers lets the network carve curved regions.

**Architecture.** Layer $0$ = inputs; hidden layers $1..L-1$; output layer $L$. Fully connected: every unit connects to every unit of the previous layer; **feed-forward**: no cycles or recurrent loops.

$$z^{(l)} = W^{(l)}a^{(l-1)} + b^{(l)},\qquad a^{(l)} = \phi(z^{(l)}),\qquad a^{(0)}=x,\; \hat y = a^{(L)}$$

$W^{(l)}$ has shape (units in $l$) $\times$ (units in $l-1$).

**XOR by hand** (step activations): $h_1=\text{step}(x_1+x_2-0.5)$ (OR), $h_2=\text{step}(x_1+x_2-1.5)$ (AND), output $=\text{step}(h_1-h_2-0.5)$.
- $(0,0)$: $h=(0,0)$, out $=\text{step}(-0.5)=0$ ✓
- $(0,1)$ or $(1,0)$: $h=(1,0)$, out $=\text{step}(0.5)=1$ ✓
- $(1,1)$: $h=(1,1)$, out $=\text{step}(-0.5)=0$ ✓
So 2 hidden units solve XOR: "OR but not AND".

**Output layer and loss.**

| Task | Output activation | Loss |
| --- | --- | --- |
| Regression | identity | squared error |
| Binary classification | sigmoid | binary cross-entropy |
| Multiclass | softmax $\;\dfrac{e^{z_k}}{\sum_j e^{z_j}}$ | categorical cross-entropy |

**Counting parameters.** Layer with $m$ inputs, $n$ units: $(m+1)n$.
- Architecture $4\to5\to3\to2$: $(4+1)5 + (5+1)3 + (3+1)2 = 25+18+8=\mathbf{51}$.
- $784\to128\to64\to10$: $785\cdot128 + 129\cdot64 + 65\cdot10 = 100480+8256+650 = 109386$.
- Without biases it would be $mn$ per layer. Check whether a question says "weights" or "parameters/weights and biases".

**Linear activations collapse.** If $\phi$ is the identity, $a^{(2)}=W^{(2)}(W^{(1)}x+b^{(1)})+b^{(2)} = (W^{(2)}W^{(1)})x + \text{const}$: a single linear map. Depth without nonlinearity adds nothing; the network equals linear (or logistic, if only the output is sigmoid) regression.

**Universal approximation theorem.** A feed-forward network with one hidden layer of enough units and a non-polynomial (e.g. sigmoid) activation can approximate any continuous function on a compact set arbitrarily well. It says nothing about how many units are needed or whether gradient descent will find the weights.

## 3. Activation functions [PYQ: High]

**Intuition.** Without nonlinear activation functions, composing multiple layers collapses into a single affine transformation:
$$a^{(2)} = W^{(2)}(W^{(1)}x + b^{(1)}) + b^{(2)} = (W^{(2)}W^{(1)})x + (W^{(2)}b^{(1)} + b^{(2)})$$
Depth without nonlinearity cannot compute non-linear decision boundaries (such as XOR). Activation functions introduce the curvature required to approximate complex functions.

| Name | Formula $\phi(z)$ | Derivative $\phi'(z)$ | Output Range | Key properties & GATE points |
| --- | --- | --- | --- | --- |
| **Sigmoid** | $\dfrac{1}{1 + e^{-z}}$ | $\sigma(z)(1 - \sigma(z))$ | $(0, 1)$ | Smooth, monotonic; max derivative is $0.25$ at $z = 0$; saturates for large $\vert z\vert$ causing **vanishing gradients**; output not zero-centered. |
| **tanh** | $\dfrac{e^z - e^{-z}}{e^z + e^{-z}}$ | $1 - \tanh^2(z)$ | $(-1, 1)$ | Smooth, monotonic, **zero-centered**; max derivative is $1.0$ at $z = 0$; still suffers from vanishing gradients for $\vert z\vert \gg 0$. |
| **ReLU** | $\max(0, z)$ | $\begin{cases} 1 & z > 0 \\ 0 & z < 0 \\ \text{undefined} & z = 0 \end{cases}$ | $[0, \infty)$ | Piecewise linear; non-saturating for $z > 0$ (prevents vanishing gradient); non-differentiable at $z = 0$ (subgradient $[0, 1]$, conventionally 0); vulnerable to **dying ReLU**. |
| **Leaky ReLU** | $\max(\alpha z, z)$, $\alpha \approx 0.01$ | $\begin{cases} 1 & z > 0 \\ \alpha & z < 0 \end{cases}$ | $(-\infty, \infty)$ | Avoids dying units: small non-zero slope $\alpha$ on negative inputs keeps gradient flowing. |
| **Softmax** | $\dfrac{e^{z_k}}{\sum_{j=1}^K e^{z_j}}$ | $s_k(\delta_{kj} - s_j)$ | $(0, 1), \sum s_k = 1$ | Multi-class generalization of sigmoid; turns logits into a valid probability distribution. |

### 3.1 Mathematical analysis of core activation functions

**Sigmoid derivative derivation:**
$$\sigma(z) = (1 + e^{-z})^{-1} \implies \frac{d\sigma}{dz} = -(1 + e^{-z})^{-2}(-e^{-z}) = \frac{1}{1 + e^{-z}} \cdot \frac{e^{-z}}{1 + e^{-z}} = \sigma(z)(1 - \sigma(z))$$
- At $z = 0$: $\sigma(0) = 0.5$, $\sigma'(0) = 0.5(1 - 0.5) = \mathbf{0.25}$ (the global maximum of $\sigma'$).
- For large positive or negative $z$: $\sigma'(z) \to 0$ (saturation). In deep networks, multiplying $L$ such derivatives causes gradients to scale as $(0.25)^L \to 0$ (**vanishing gradient problem**).

**ReLU and the dying ReLU problem:**
- $\phi(z) = \max(0, z)$.
- For $z > 0$, the derivative is strictly $1$, so gradient signals pass through without shrinkage regardless of network depth.
- At $z = 0$, the function has a sharp corner and is technically non-differentiable; optimization algorithms use a subgradient $\in [0, 1]$ (by convention set to $0$ in standard libraries).
- If large gradient updates cause a neuron's weights to update such that $w^T x + b < 0$ for **all** data points in the training set, then $\phi'(z) = 0$ permanently: the neuron never activates and its weights never change again (**dead neuron**). Leaky ReLU and ELU resolve this by ensuring $\phi'(z) > 0$ for $z < 0$.

### 3.2 Worked example: activation and derivative calculations

A single neuron has weight vector $w = [-2, 3]^T$, bias $b = -1$. Consider inputs $x_1 = [1, 1]^T$ and $x_2 = [2, 1]^T$.

1. **Pre-activations ($z = w^T x + b$):**
   - For $x_1$: $z_1 = (-2)(1) + (3)(1) - 1 = 0$.
   - For $x_2$: $z_2 = (-2)(2) + (3)(1) - 1 = -2$.

2. **Under Sigmoid activation $\sigma(z)$:**
   - At $x_1$: $a_1 = \sigma(0) = \mathbf{0.5000}$; derivative $\sigma'(0) = 0.5(1 - 0.5) = \mathbf{0.2500}$.
   - At $x_2$: $a_2 = \sigma(-2) = \frac{1}{1 + e^2} \approx \frac{1}{1 + 7.3891} = \mathbf{0.1192}$; derivative $\sigma'(-2) = 0.1192(1 - 0.1192) = \mathbf{0.1050}$.

3. **Under ReLU activation $\phi(z) = \max(0, z)$:**
   - At $x_1$: $a_1 = \max(0, 0) = \mathbf{0}$; derivative at $0$ is taken as $\mathbf{0}$ (subgradient).
   - At $x_2$: $a_2 = \max(0, -2) = \mathbf{0}$; derivative is strictly $\mathbf{0}$. The unit is inactive.

4. **Under Leaky ReLU ($\alpha = 0.1$):**
   - At $x_2$: $a_2 = \max(0.1(-2), -2) = \mathbf{-0.2}$; derivative is $\mathbf{0.1000}$ (gradient flows back).

## 4. Forward pass and backpropagation worked by hand [PYQ: Medium]

**Network.** $2\to2\to1$, sigmoid everywhere, squared loss $L=\tfrac12(\hat y-t)^2$. Input $x=(1,\,0.5)$, target $t=1$.

Weights: hidden unit 1: $w=(0.1,0.3)$, $b=0$; hidden unit 2: $w=(0.2,0.4)$, $b=0.1$; output: $v=(0.5,0.6)$, $c=0.1$.

**Forward pass.**
1. $z_1 = 0.1(1)+0.3(0.5)+0 = 0.25$; $h_1=\sigma(0.25)=0.5622$.
2. $z_2 = 0.2(1)+0.4(0.5)+0.1 = 0.5$; $h_2=\sigma(0.5)=0.6225$.
3. $z_o = 0.5(0.5622)+0.6(0.6225)+0.1 = 0.7546$; $\hat y=\sigma(0.7546)=0.6802$.
4. $L=\tfrac12(0.6802-1)^2 = 0.05114$.

**Backward pass** (chain rule; $\delta$ = $\partial L/\partial z$ at that unit).
1. Output: $\delta_o = (\hat y-t)\,\hat y(1-\hat y) = (-0.3198)(0.6802)(0.3198)=-0.06957$.
2. Output weights: $\partial L/\partial v_j=\delta_o h_j$: $(-0.03911,\,-0.04331)$; $\partial L/\partial c=-0.06957$.
3. Hidden deltas: $\delta_j=\delta_o\,v_j\,h_j(1-h_j)$: $\delta_1=(-0.06957)(0.5)(0.2461)=-0.008562$; $\delta_2=(-0.06957)(0.6)(0.2350)=-0.009810$.
4. Hidden weights: $\partial L/\partial w_{jk}=\delta_jx_k$: unit 1 $(-0.008562,\,-0.004281)$; unit 2 $(-0.009810,\,-0.004905)$; biases $=\delta_j$.

**Gradient-descent step** with $\eta=0.5$ ($\theta\leftarrow\theta-\eta\,\partial L/\partial\theta$):
$v=(0.5196,\,0.6217)$, $c=0.1348$, unit 1 $w=(0.1043,\,0.3021)$, $b_1=0.0043$, unit 2 $w=(0.2049,\,0.4025)$, $b_2=0.1049$.

New forward pass: $\hat y=0.6935$, $L=0.04696$: the loss dropped from $0.05114$. (A numerical finite-difference check of $\partial L/\partial w_{11}$ gives $-0.008562$, matching.)

**Pattern to memorise.** Each hidden delta = (downstream delta) $\times$ (connecting weight) $\times$ (local activation derivative); each weight gradient = (delta of the unit it feeds) $\times$ (activation coming in). With softmax + cross-entropy the output delta is simply $\hat y-y$ (and for sigmoid + cross-entropy too: no extra $\hat y(1-\hat y)$ factor, which is why cross-entropy avoids the saturation slow-down of squared loss).

**Training loops.** *Batch* GD uses all examples per step; *stochastic* GD one example; *mini-batch* a few. An *epoch* = one pass over the data.

## Formulas and facts to memorise

| Item | Formula/fact | When to use |
| --- | --- | --- |
| Perceptron update | $w\leftarrow w+\eta(y-\hat y)x$ | trace questions |
| Parameters of a layer | $(m+1)n$ | architecture counting |
| Forward | $a^{(l)}=\phi(W^{(l)}a^{(l-1)}+b^{(l)})$ | evaluation |
| Sigmoid derivative | $\sigma(1-\sigma)$, max $0.25$ | backprop numeric |
| tanh derivative | $1-\tanh^2$ | backprop numeric |
| ReLU derivative | $1[z>0]$ | backprop numeric |
| Softmax + CE output delta | $\hat y-y$ | gradients |
| Output delta, sigmoid + squared | $(\hat y-t)\hat y(1-\hat y)$ | as worked above |
| Linear activations | collapse to one linear map | conceptual |
| XOR | needs $\ge1$ hidden layer (2 units) | separability |

## GATE traps

- A perceptron converges only for linearly separable data; for XOR it loops forever. Learning rate does not affect convergence for separable data when starting from $w=0$ (it only scales $w$).
- Parameter counts: don't forget biases; don't count the input layer as having parameters; $(m+1)n$ per layer.
- "Number of layers" conventions vary (input counted or not): count weight matrices to be safe.
- Linear (identity) hidden activations make the network a linear model regardless of depth.
- Sigmoid derivative is at most $0.25$ (not $1$); the gradient through a deep sigmoid net vanishes.
- Softmax outputs sum to 1; sigmoid outputs of separate units need not.
- Backprop gives gradients; it is **not** the optimiser. Weights initialised to the same value (e.g. all 0) in a hidden layer stay symmetric and learn identical features.
- Universal approximation does not guarantee training finds the solution or that the network generalises.

## Connections

- [Linear and logistic regression](linear-and-logistic-regression.md) — a single sigmoid neuron *is* logistic regression; a linear neuron is linear regression; same gradient $(\hat y-y)x$.
- [Classification methods](classification-methods.md) — perceptron vs SVM (any separating line vs the max-margin one); kernels vs learned hidden features.
- [ML foundations](ml-foundations.md) — more hidden units = more variance; regularise with weight decay (ridge on weights), early stopping, validation sets.
- [Maxima and minima](../04-calculus-optimization/maxima-minima-optimization.md) — gradient descent, chain rule.
- [Matrices and determinants](../03-linear-algebra/matrices-and-determinants.md) — layers are matrix multiplications; collapse of linear layers is $W_2W_1$.
- [Boolean algebra](../09-digital-logic/boolean-algebra-and-minimization.md) — threshold units implement AND/OR; XOR needs two levels, like a two-level logic circuit.

## Practice

**Q1 (NAT).** A fully connected network has 10 inputs, hidden layers of 20 and 15 units, and 3 outputs. How many parameters (weights and biases)?

<details><summary>Answer</summary>

**Answer:** 583. **Solution:** $(10+1)20 + (20+1)15 + (15+1)3 = 220+315+48=583$.

</details>

**Q2 (MCQ).** A network with 3 hidden layers, all using the identity activation, is equivalent to: (a) a deep nonlinear model (b) a single linear model (c) a decision tree (d) logistic regression always.

<details><summary>Answer</summary>

**Answer:** (b). **Solution:** composition of affine maps is affine.

</details>

**Q3 (MCQ).** Which function can a single perceptron NOT compute? (a) AND (b) OR (c) NAND (d) XOR.

<details><summary>Answer</summary>

**Answer:** (d). **Solution:** AND, OR, NAND are linearly separable; XOR is not.

</details>

**Q4 (NAT).** A sigmoid neuron has $z=0$. What is its derivative $\sigma'(z)$?

<details><summary>Answer</summary>

**Answer:** 0.25. **Solution:** $\sigma(0)=0.5$, $\sigma'=0.5(1-0.5)=0.25$.

</details>

**Q5 (NAT).** A perceptron (threshold $\ge0\to1$) has $w=(1,-1)$, $b=0$, $\eta=1$. It sees $x=(1,2)$ with $y=1$. After the update, what is $w_2$?

<details><summary>Answer</summary>

**Answer:** 1. **Solution:** $w^Tx+b=1-2=-1\Rightarrow\hat y=0$, error $=y-\hat y=1$. $w\leftarrow(1,-1)+1\cdot(1,2)=(2,1)$. So $w_2=1$.

</details>

**Q6 (NAT).** Softmax over logits $(0,\ \ln2,\ \ln 5)$. Probability of class 3? (3 decimals)

<details><summary>Answer</summary>

**Answer:** 0.625. **Solution:** $e^z = (1,2,5)$, sum $=8$; $5/8=0.625$.

</details>

**Q7 (MSQ).** Which statements are true? (a) ReLU units can be "dead". (b) Sigmoid hidden units in a deep network risk vanishing gradients. (c) Initialising all weights of a hidden layer to the same value is fine because gradients differ. (d) Backpropagation is an application of the chain rule.

<details><summary>Answer</summary>

**Answer:** (a), (b), (d). **Solution:** (c) is false: identical weights receive identical gradients (symmetry), so the units stay identical.

</details>

**Q8 (NAT).** One sigmoid output neuron, cross-entropy loss, $\hat y=0.8$, target $y=1$, incoming activation (a single input feature) $a=2$. Compute $\partial L/\partial w$ for the weight on that input.

<details><summary>Answer</summary>

**Answer:** $-0.4$. **Solution:** with sigmoid + cross-entropy, $\partial L/\partial z=\hat y-y=-0.2$; $\partial L/\partial w=(-0.2)(2)=-0.4$.

</details>
