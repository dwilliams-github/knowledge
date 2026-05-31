---
title: "Flow Map Matching"
topic: generative-models
date: 2024-06-11
tags: [flow-matching, diffusion, fast-sampling, consistency-models, distillation, ODE, stochastic-interpolants]
papers:
  - "Boffi, Albergo & Vanden-Eijnden (2024) — [arXiv:2406.07507](https://arxiv.org/abs/2406.07507)"
---

# Flow Map Matching: Summary and Key Insights

## Paper Reference

**"Flow Map Matching with Stochastic Interpolants: A Mathematical Framework for Consistency Models"**
Nicholas M. Boffi, Michael S. Albergo, Eric Vanden-Eijnden ([arXiv:2406.07507](https://arxiv.org/abs/2406.07507), June 2024)
Published in Transactions on Machine Learning Research, 2025.

---

## Motivation

Fast inference methods for diffusion models — [progressive distillation](progressive-distillation.md), consistency models, consistency trajectory models — were developed empirically and achieved practical success, but lacked a unifying theoretical framework. As the authors state: "a systematic understanding of their design has been hindered by the lack of a comprehensive theoretical framework." FMM provides this missing foundation.

## The Central Object: The Two-Time Flow Map

Given a velocity field $`b_t(x)`$ (from flow matching, [stochastic interpolants](stochastic-interpolants.md), or the probability flow ODE of a diffusion model), the ODE:

$$\frac{dx}{dt} = b_t(x)$$

defines a **flow map** $`F_{s,t}`$ that maps any state at time $s$ to the corresponding state at time $t$:

$$F_{s,t}(x_s) = x_s + \int_s^t b_\tau(F_{s,\tau}(x_s))\thinspace d\tau$$

The flow map satisfies the **semigroup property**:

$$F_{s,t} = F_{u,t} \circ F_{s,u} \quad \text{for } s \leq u \leq t$$

Flowing from $s$ to $u$ and then from $u$ to $t$ is the same as flowing directly from $s$ to $t$. This is the composition law expected from time evolution operators.

### Invertibility

The flow map is invertible ($`F_{s,t}^{-1} = F_{t,s}`$), guaranteed by uniqueness of ODE solutions under the Lipschitz condition:

$$\|b_t(x) - b_t(y)\| \leq L\|x - y\|$$

for finite $L$. If two distinct initial conditions could map to the same endpoint, running the ODE backward would produce two trajectories from one point, violating uniqueness. For neural networks with smooth activations (SiLU, GELU) composed with linear layers, the Lipschitz condition is satisfied with $L$ bounded by the product of spectral norms across layers.

### Lipschitz Condition

The Lipschitz condition states that for all $x, y$:

$$\|b(x) - b(y)\| \leq L\|x - y\|$$

For differentiable functions, this is equivalent to requiring the derivative to be bounded everywhere: $\|b'(x)\| \leq L$. The standard counterexample showing why this matters is $\dot{x} = \sqrt{|x|}$, which is continuous but not Lipschitz at $x = 0$ (since $\sqrt{h}/h = 1/\sqrt{h} \to \infty$), and admits non-unique solutions.

The **one-sided Lipschitz condition** is a weaker variant:

$$\langle b_t(x) - b_t(y),\thinspace x - y \rangle \leq L\|x - y\|^2$$

This only constrains expansion (divergence of trajectories), allowing arbitrary contraction. It is more natural for generative ODEs, which typically contract — mapping diffuse noise to concentrated data.

## Eulerian vs Lagrangian Perspective

The paper frames the distinction between flow matching and FMM using fluid mechanics terminology:

- **Eulerian** (flow matching): learn the velocity field $`b_t(x)`$ at fixed spatial points. At inference, numerically integrate the ODE through the field. This is the "field theory" description.
- **Lagrangian** (FMM): learn the flow map $`F_{s,t}(x)`$ directly — where each particle ends up. At inference, evaluate the network once per step with no numerical integration within each step.

These are mathematically equivalent — the velocity field determines the flow map and vice versa. The distinction is purely about what the network outputs and how inference is performed.

Note: "Lagrangian" here refers to the fluid mechanics convention (particle tracking), not the variational mechanics convention (action minimization via $\mathcal{L} = T - V$), though the Benamou-Brenier formulation of optimal transport connects the two.

## Connection to Classical Numerical Integration

The flow map $`F_{s,t}`$ is the object that classical ODE integrators approximate:

- **Euler**: first-order polynomial approximation to $`F_{s,t}`$, one velocity evaluation
- **RK4**: fourth-order polynomial approximation, four velocity evaluations
- **FMM**: learned neural network approximation, one network evaluation, potentially accurate over arbitrarily large intervals

Karras (2022) optimized the classical integrator approach — choosing the best step placement and integrator order for diffusion ODEs. FMM replaces the analytical approximation with a learned one, trading known error bounds for greater flexibility over large time intervals.

## Two Training Modes

### Distillation Training (LMD — Lagrangian Matching by Differentiation)

Requires a pre-trained velocity field $`b_t(x)`$. The loss enforces the Lagrangian equation at sampled points:

$$\mathcal{L}_{\text{LMD}}(\hat{X}) = \int_{[0,1]^2} w_{s,t}\thinspace \mathbb{E}\left[|\partial_t \hat{X}_{s,t}(I_s) - b_t(\hat{X}_{s,t}(I_s))|^2\right] ds\thinspace dt$$

The training loop:

1. Sample $`x_0 \sim \rho_0`$ (noise), $`x_1 \sim \rho_1`$ (data)
2. Sample $s, t$ from $[0,1]$ with $s \leq t$
3. Compute $`I_s = \alpha_s x_0 + \beta_s x_1`$ (interpolant provides a correctly distributed point at time $s$)
4. Forward pass: compute $`\hat{X}_{s,t}(I_s)`$ — the network's predicted endpoint
5. Compute $`\partial_t \hat{X}_{s,t}(I_s)`$ via automatic differentiation with respect to the scalar input $t$
6. Evaluate $`b_t(\hat{X}_{s,t}(I_s))`$ using the pre-trained velocity model
7. Penalize the ODE residual $`|\partial_t \hat{X} - b_t(\hat{X})|^2`$

The derivative $`\partial_t \hat{X}`$ is computed using the same automatic differentiation infrastructure built for backpropagation — repurposed to evaluate a derivative that appears in the loss function itself, rather than for parameter optimization. This is structurally identical to physics-informed neural networks (PINNs), where differential equations are embedded in the loss and autodiff enforces them.

The interpolation coefficients $`\alpha_t, \beta_t`$ (e.g., linear: $`\alpha_t = 1-t`$, $`\beta_t = t`$; or trigonometric: $`\alpha_t = \cos(\frac{\pi}{2}t)`$, $`\beta_t = \sin(\frac{\pi}{2}t)`$) are chosen by the practitioner, not learned. Their only role in distillation is generating correctly distributed sample points $`I_s`$ at time $s$.

### Direct Training (FMM Loss)

No pre-trained model needed. The loss combines two terms:

$$\mathcal{L}_{\text{FMM}}(\hat{X}) = \int_{[0,1]^2} w_{s,t}\left(\mathbb{E}\left[|\partial_t \hat{X}_{s,t}(\hat{X}_{t,s}(I_t)) - \dot{I}_t|^2\right] + \mathbb{E}\left[|\hat{X}_{s,t}(\hat{X}_{t,s}(I_t)) - I_t|^2\right]\right) ds\thinspace dt$$

**Derivative matching**: $`|\partial_t \hat{X}_{s,t}(\hat{X}_{t,s}(I_t)) - \dot{I}_t|^2`$ — the time derivative of the flow map must match the interpolant velocity $`\dot{I}_t`$. This enforces the ODE constraint.

**Value matching**: $`|\hat{X}_{s,t}(\hat{X}_{t,s}(I_t)) - I_t|^2`$ — the round trip $`\hat{X}_{s,t} \circ \hat{X}_{t,s}`$ must recover $`I_t`$. This enforces invertibility.

The composition $`\hat{X}_{t,s}(I_t)`$ maps backward from $`I_t`$ (correctly distributed at time $t$ via the interpolant) to time $s$ using the flow map itself. The loss is self-referential — the flow map is trained against its own inverse, anchored to the interpolant.

The regression target $`\dot{I}_t = \dot{\alpha}_t x_0 + \dot{\beta}_t x_1`$ is a per-sample quantity that is noisy (different $(x_0, x_1)$ pairs give different targets at the same point). But the $L^2$ regression automatically recovers the conditional expectation:

$$b_t(x) = \mathbb{E}[\dot{I}_t \mid I_t = x]$$

This is the marginal velocity field — the average of $`\dot{I}_t`$ over all $(x_0, x_1)$ pairs that produce $`I_t = x`$ at time $t$. The same principle that makes flow matching simulation-free extends to FMM direct training.

### Design Choice: Equal Weighting

The two terms in the FMM loss are given equal weight. This is not derived from a theoretical principle — it is the simplest choice that enforces both constraints (invertibility and ODE consistency). A more general formulation could use separate coefficients, analogous to the $\beta$ weighting in VAE losses. The relative weighting is an additional design degree of freedom that could be optimized, in the spirit of Karras's loss weighting analysis.

## Inference: Flexible Step Count

Since the network takes $(x, s, t)$ as inputs, the step count is chosen **post-training**:

- **One step**: $`x_1 = F_{0,1}(x_0)`$ with $`x_0 \sim \mathcal{N}(0, I)`$
- **$K$ steps**: partition $[0, 1]$ into subintervals and compose:

$$x_1 = F_{t_{K-1}, 1} \circ \cdots \circ F_{t_1, t_2} \circ F_{0, t_1}(x_0)$$

No retraining needed for different step counts. This is the key advantage over [progressive distillation](progressive-distillation.md), which requires committing to a step count at training time (specifically, powers of 2 from the halving schedule) and produces a separate model checkpoint for each.

## Unification of Prior Work

FMM shows that existing fast-sampling methods are special cases of learning the two-time flow map:

| Method | What it learns | Constraints |
|--------|---------------|-------------|
| **Progressive distillation** (Salimans & Ho, 2022) | $`F_{s,t}`$ for specific halved schedule pairs | Fixed step count (powers of 2), iterative retraining |
| **Consistency models** (Song et al., 2023) | $`F_{t, 0}`$ — map any $t$ to clean endpoint | One endpoint fixed at $t=0$, flexible steps |
| **Consistency trajectory models** | $`F_{s, t}`$ for arbitrary pairs | Full two-time map, but adversarial loss |
| **FMM** | $`F_{s, t}`$ for arbitrary pairs | Full two-time map, principled non-adversarial loss |

All learn the same underlying object — the flow map — with different restrictions on which $(s, t)$ pairs are supported and different training objectives.

### Connection to Progressive Distillation

Each round of [progressive distillation](progressive-distillation.md) implicitly learns a flow map over a doubled time interval. The semigroup property $`F_{s,t} = F_{u,t} \circ F_{s,u}`$ is exactly the composition that the teacher provides — the student learns to collapse two teacher steps into one. FMM formalizes this and shows the flow map can be trained directly without iterative halving.

Progressive distillation also introduces a subtle shift in what the network represents. In the original diffusion model, the network at each $t$ predicts a local denoising quantity. After distillation, the output at the same $t$ has been deliberately biased to compensate for discretization error over the larger stride. After several halving rounds, $t$ no longer has its original interpretation as "noise level" — the network is implementing a learned transport map whose relationship to Gaussian denoising has drifted. FMM avoids this by making the stride explicit via the $(s, t)$ parameterization.

### Connection to MeanFlow

The average velocity:

$$u_{s,t}(x_s) = \frac{F_{s,t}(x_s) - x_s}{t - s}$$

is a reparameterization of the flow map — knowing one gives you the other. MeanFlow derives a self-consistency relation for training $`u_{s,t}`$; FMM derives training objectives for $`F_{s,t}`$ directly. Different parameterizations of the same object, with different training procedures.

### What FMM Is, Operationally

Under Karras's reduction that a diffusion model is a denoiser with preconditioning, FMM is a **generalized denoiser** with two time parameters. A standard denoiser takes $`(x_t, t)`$ and predicts $`x_0`$ — this is $`F_{t,0}`$. The flow map takes $`(x_s, s, t)`$ and predicts $`x_t`$ — the denoiser generalized to arbitrary start and end times. The network architecture and output space are the same; only the conditioning changes. Flow matching / [stochastic interpolants](stochastic-interpolants.md) provide additional freedom in that the paths need not correspond to any Gaussian diffusion process.

## Role in the Broader Landscape

Three papers together provide a clean theoretical foundation for the diffusion model pipeline:

- **Karras (2022)**: unifying framework for **training** — shows DDPM, NCSN, score-SDE are equivalent up to noise schedule, preconditioning, and loss weighting
- **[Stochastic interpolants](stochastic-interpolants.md) (Albergo & Vanden-Eijnden, 2022)**: unifying framework for the **generative process** — derives flow matching, score-based diffusion, and probability flow ODEs from a single interpolant construction
- **FMM (Boffi, Albergo & Vanden-Eijnden, 2024)**: unifying framework for **fast inference** — shows progressive distillation, consistency models, and consistency trajectory models are all special cases of learning the two-time flow map

## Key References

- Salimans, T. and Ho, J. (2022). Progressive Distillation for Fast Sampling of Diffusion Models. ICLR 2022. [arXiv:2202.00512](https://arxiv.org/abs/2202.00512)
- Song, Y. et al. (2023). Consistency Models. ICML 2023. [arXiv:2303.01469](https://arxiv.org/abs/2303.01469)
- Albergo, M. S. and Vanden-Eijnden, E. (2022). Building Normalizing Flows with Stochastic Interpolants. [arXiv:2209.15571](https://arxiv.org/abs/2209.15571)
- Karras, T. et al. (2022). Elucidating the Design Space of Diffusion-Based Generative Models. NeurIPS 2022. [arXiv:2206.00364](https://arxiv.org/abs/2206.00364)
- Geng, Z. et al. (2025). Mean Flows for One-step Generative Modeling. NeurIPS 2025. [arXiv:2505.13447](https://arxiv.org/abs/2505.13447)
