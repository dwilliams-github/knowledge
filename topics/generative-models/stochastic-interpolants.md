---
title: "Stochastic Interpolants: Summary and Key Insights"
topic: generative-models
date: 2026-04-04
tags: [flow-matching, diffusion, normalizing-flows, generative-models, ODE, score-matching]
source: agent session 2026-04-04
papers:
  - "Albergo & Vanden-Eijnden (2022) https://arxiv.org/abs/2209.15571"
  - "Albergo & Vanden-Eijnden (2023) https://arxiv.org/abs/2303.08797"
---

# Stochastic Interpolants: Summary and Key Insights

## Paper Reference
**[Building Normalizing Flows with Stochastic Interpolants](https://arxiv.org/abs/2209.15571)**
Michael S. Albergo, Eric Vanden-Eijnden (2022)

Follow-up: **[Stochastic Interpolants: A Unifying Framework for Flows and Diffusions](https://arxiv.org/abs/2303.08797)** (2023)

---

## Core Construction

The framework begins with a remarkably simple object. Given samples from a noise distribution $`I_0 \sim p_0`$ (typically $\mathcal{N}(0, I)$) and a data distribution $`I_1 \sim p_1`$, define a stochastic process:

$$I_t = \alpha_t \, I_0 + \beta_t \, I_1$$

where $`\alpha_t, \beta_t`$ are smooth interpolation coefficients satisfying boundary conditions $`\alpha_0 = \beta_1 = 1`$ and $`\alpha_1 = \beta_0 = 0`$.

This interpolant is not learned — it is a deterministic, analytically defined function. A common choice is the trigonometric schedule: $`\alpha_t = \cos(\frac{\pi}{2}t)`$, $`\beta_t = \sin(\frac{\pi}{2}t)`$.

The entire framework flows from one question: **what ODE has marginals matching those of $`I_t`$?**

## The Velocity Field

The answer is an ODE with velocity field defined as the conditional expectation:

$$v_t(x) = \mathbb{E}\left[\dot{I}_t \mid I_t = x\right]$$

where $`\dot{I}_t = \dot{\alpha}_t \, I_0 + \dot{\beta}_t \, I_1`$ is the time derivative of the interpolant. The ODE:

$$\frac{dx}{dt} = v_t(x)$$

generates marginals $`\rho_t`$ matching those of $`I_t`$ at every $t$, and in particular pushes $`p_0`$ to $`p_1`$.

## Training

The velocity field is learned by a neural network $\hat{v}(t, x)$ via a regression loss:

$$\mathcal{L} = \int_0^1 \mathbb{E}\left\|\hat{v}(t, I_t) - \dot{I}_t\right\|^2 \, dt$$

The training loop is simulation-free:

1. Sample $`x_1 \sim p_1`$ (data point from the dataset)
2. Sample $`x_0 \sim p_0`$ (noise sample, typically $\mathcal{N}(0, I)$)
3. Sample $t \sim \text{Uniform}[0, 1]$
4. Compute $`I_t = \alpha_t \, x_0 + \beta_t \, x_1`$ (closed-form arithmetic)
5. Compute $`\dot{I}_t = \dot{\alpha}_t \, x_0 + \dot{\beta}_t \, x_1`$ (closed-form arithmetic)
6. Minimize $`\|\hat{v}(t, I_t) - \dot{I}_t\|^2`$

No ODE/SDE simulation is needed during training. The interpolant serves as scaffolding to generate cheap training pairs, then is discarded at inference time.

## Key Insight: Marginal vs. Conditional Velocities

The network does **not** learn the velocity of any individual interpolant trajectory. Since each training step draws a fresh random $`x_0`$, the same region of space at time $t$ is visited by many different $`(x_0, x_1)`$ pairs with different velocities $`\dot{I}_t`$. The $L^2$ regression objective forces the network to learn their **conditional mean** — the average over all pairs consistent with a given $`I_t = x`$.

This means:
- The conditional interpolant paths are straight lines (by construction)
- The marginal ODE trajectories are generally **curved**, because the velocity field averages over crossing conditional paths
- The curvature of the marginal ODE is what necessitates multi-step integration at inference time

## Score Function as a Derived Quantity

A notable result: the score function $`\nabla \log \rho_t(x)`$ is obtainable algebraically from the learned velocity field. For the trigonometric schedule:

$$\nabla \log \rho_t(x) = -x - \frac{2}{\pi}\tan\!\left(\frac{\pi}{2}t\right) v_t(x) \quad \text{for } t \in [0, 1)$$

This means the entire score-based diffusion framework is **contained within** the interpolant/flow matching framework. If you learn $`v_t`$, you get the score for free — no separate score matching training is required.

## Relationship to Other Frameworks

### Equivalence to Flow Matching
The stochastic interpolants training objective is mathematically equivalent to the flow matching objective of Lipman et al. (2022). Both learn a velocity field via simulation-free regression on interpolated pairs. The interpolant framework provides a cleaner mathematical derivation.

### Equivalence to Diffusion Models
With specific choices of $`\alpha_t`$ and $`\beta_t`$ (e.g., variance-preserving schedule), the framework recovers standard diffusion model training. The diffusion forward process (e.g., Ornstein-Uhlenbeck) is not needed — it was a historical construction that the interpolant framework bypasses entirely.

### Relationship to Rectified Flow
Liu et al.'s rectified flow (2022) uses the same linear interpolation with $`\alpha_t = 1-t`$, $`\beta_t = t`$. This is a specific instantiation of the interpolant framework. Rectified flow additionally introduces the Reflow procedure, which iteratively refines the noise-data coupling to straighten marginal ODE trajectories.

### Relationship to Flow Map Matching and Progressive Distillation
Boffi, Albergo, and Vanden-Eijnden (2024) extend the interpolant framework to learn two-time flow maps $`X_{s,t}`$ for arbitrary $(s, t)$, enabling post-training selection of step count. [Progressive distillation](progressive-distillation.md) (Salimans & Ho, 2022) is recovered as a special case: each distillation round implicitly learns the flow map composition $`X_{s,t} = X_{u,t} \circ X_{s,u}`$ via teacher-student training, but commits to a fixed step count at training time.

## Design Degrees of Freedom

The choice of $`\alpha_t`$ and $`\beta_t`$ affects:
- **The function the network must learn**: different schedules produce different $`\dot{I}_t`$ targets, changing the velocity field $`v_t`$ even though the underlying transport is the same (paths are identical, just traversed at different speeds)
- **Training efficiency**: the $t$-sampling distribution and loss weighting determine where the network allocates capacity, analogous to the noise schedule optimization in Karras et al. (2022)
- **Not the optimal solution**: the true velocity field's ODE generates the same samples regardless of parameterization; the schedule only affects approximation quality for finite-capacity networks

## Conceptual Advantages

1. **ODE-first**: the deterministic ODE is the primary object; SDEs are optional perturbations added later in the follow-up paper. This inverts the historical development where diffusion models started with SDEs.
2. **No stochastic process needed**: the framework does not require defining a forward diffusion/corruption process. The interpolant directly specifies the bridge between distributions.
3. **Simulation-free**: training requires only evaluation of closed-form expressions, no numerical integration.
4. **Probability conservation**: the ODE flow is a diffeomorphism (bijection), preserving total probability ($`\int \rho_t = 1`$ for all $t$) via the continuity equation $`\partial_t \rho_t + \nabla \cdot (\rho_t v_t) = 0`$.
5. **Clean mathematical presentation**: the authors (mathematical physicists) use precise language that bridges the physics and ML communities.

## Practical Considerations

- The entire difficulty of generative modeling reduces to whether the network architecture can capture the **correlation structure** of the data well enough to predict the conditional mean velocity accurately.
- In latent spaces where dimensions are approximately decorrelated, the velocity field factorizes more easily, explaining why latent diffusion outperforms pixel-space models.
- The starting distribution $`p_0 = \mathcal{N}(0, I)`$ is exact by construction (unlike diffusion models where the terminal distribution is only approximately Gaussian), eliminating one source of approximation error.
