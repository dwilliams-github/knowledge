---
title: "Sequential Monte Carlo: History, Concepts, and Applications to Generative Models"
topic: generative-models
date: 2026-04-25
tags: [SMC, particle-filter, inference, diffusion, flow-matching, reward-steering, Feynman-Kac]
source: agent session 2026-04-25
papers:
  - "Gordon, Salmond & Smith (1993) — bootstrap particle filter"
  - "Del Moral (2004) — Feynman-Kac Formulae"
  - "Doucet, de Freitas & Gordon (2001) — Sequential Monte Carlo Methods in Practice"
  - "Le et al. (ICLR 2018) https://arxiv.org/abs/1705.10306"
  - "Singhal et al. (2025) https://arxiv.org/abs/2501.06848"
---

# Sequential Monte Carlo: History, Concepts, and Applications to Generative Models

## What SMC Is

Sequential Monte Carlo is importance sampling applied sequentially, with a resampling step to prevent weight degeneracy. It approximates a sequence of probability distributions $\pi_0, \pi_1, \ldots, \pi_T$ using a population of $N$ weighted samples called **particles**.

The core algorithm (bootstrap particle filter):

1. **Initialize**: draw $N$ particles $\{z_0^{(i)}\}_{i=1}^N$ from a prior $p(z_0)$, each with weight $w_0^{(i)} = 1/N$.
2. **At each time step $t$**:
   - **Propagate**: move each particle forward using the dynamics, $`\tilde{z}_t^{(i)} \sim p(z_t \mid z_{t-1}^{(i)})`$.
   - **Weight**: assign each particle a weight based on how well it explains the new observation, $w_t^{(i)} \propto p(y_t \mid \tilde{z}_t^{(i)})$.
   - **Resample**: draw $N$ new particles from the current set with probability proportional to the weights, resetting all weights to $1/N$.

The weighted particle set approximates the posterior:

$$p(z_t \mid y_{1:t}) \approx \sum_{i=1}^N w_t^{(i)}\, \delta(z_t - z_t^{(i)})$$

The fundamental algorithmic move is simple: **kill unpromising particles, duplicate promising ones**. This is natural selection applied to hypothesis tracking.

## The Resampling Step

Resampling is what distinguishes SMC from naive importance sampling. Without it, after many sequential steps, one particle accumulates nearly all the weight ($w \approx 1$) while the rest become negligible ($w \approx 0$). The effective sample size collapses to $N_{\text{eff}} \approx 1$ even though $N$ particles are being maintained — a phenomenon called **weight degeneracy**.

Resampling prevents this divergence by resetting weights to uniformity at each step, at the cost of reducing **diversity** — duplicated particles share the same ancestral history. The central tension of SMC is managing this tradeoff between weight degeneracy and particle impoverishment.

Practical refinements include:

- **Adaptive resampling**: only resample when $N_{\text{eff}} = 1/\sum_i (w^{(i)})^2$ drops below a threshold, avoiding unnecessary variance injection when weights are already near-uniform.
- **Proposal design**: the bootstrap filter propagates particles using the prior dynamics $p(z_t \mid z_{t-1})$, ignoring the current observation entirely. Better proposals $q(z_t \mid z_{t-1}, y_t)$ that incorporate observation information reduce the number of particles needed for a good approximation.

## Historical Origins

The terminology "particle" and the bootstrap particle filter were introduced by Gordon, Salmond, and Smith (1993), though related ideas existed earlier. The theoretical foundations — connecting SMC to **Feynman-Kac formulae** and establishing convergence guarantees — were developed primarily by Pierre Del Moral, culminating in his book "Feynman-Kac Formulae: Genealogical and Interacting Particle Systems with Applications" (2004).

The standard collected reference for the ML and signal processing communities is "Sequential Monte Carlo Methods in Practice" edited by Doucet, de Freitas, and Gordon (2001).

SMC has deep connections to computational physics. The propagate-weight-resample loop is closely related to **diffusion Monte Carlo** and **quantum Monte Carlo**, where ensembles of walkers evolve under stochastic dynamics with branching (birth/death) to sample from target distributions. The Feynman-Kac formula provides the mathematical justification in both settings.

## Monte Carlo in High Dimensions

Monte Carlo methods are among the few approaches that remain viable in high dimensions — grid-based methods scale exponentially, while MC convergence rates ($O(1/\sqrt{N})$) are dimension-independent in principle. However, the constant hiding in that rate can grow catastrophically with dimension. Weight degeneracy, particle collapse, and the concentration of measure making most proposals land in low-probability regions are all manifestations of the curse of dimensionality.

Furthermore, Monte Carlo is fundamentally an **integration** tool — it estimates expectations. Using it for **optimization** (finding modes, maximizing likelihoods) is indirect and often inefficient compared to gradient-based methods.

## AESMC: Auto-Encoding Sequential Monte Carlo

**Reference**: Le, Igl, Rainforth, Jin, Wood (ICLR 2018, [arXiv:1705.10306](https://arxiv.org/abs/1705.10306))

AESMC combines SMC with amortized inference (auto-encoding), using deep neural networks to learn proposal distributions for SMC while simultaneously learning the generative model. The key ideas:

- Use SMC (rather than importance sampling as in VAEs/IWAEs) as the marginal likelihood estimator for structured sequential models.
- Learn the proposal distribution $q_\phi(z_t \mid z_{t-1}, y_t)$ using a neural network, amortizing the cost of proposal adaptation.
- The SMC marginal likelihood estimator provides a tighter lower bound (ELBO) than importance sampling, enabling more effective model learning.

AESMC targets **probabilistic state-space models** — latent states evolving with noisy observations:

$$z_t \sim p(z_t \mid z_{t-1}), \quad y_t \sim p(y_t \mid z_t)$$

Note: "state-space model" here refers to the classical probabilistic formulation (Kalman filters, hidden Markov models, particle filtering), **not** the linear dynamical systems used in the deep learning sequence modeling literature (HiPPO, S4, Mamba). These are different uses of the same term.

In AESMC, particles do not follow explicit trajectories. At each step, each particle is sampled from a learned proposal, and after resampling, multiple particles may share the same ancestor. The "trajectory" is a genealogical tree, not a fixed path.

## FK Steering: Feynman-Kac Steering of Diffusion Models

**Reference**: Singhal, Horvitz, Teehan, Ren, Yu, McKeown, Ranganath ([arXiv:2501.06848](https://arxiv.org/abs/2501.06848), January 2025)

FK steering applies SMC-like particle methods to steer pre-trained diffusion models toward high-reward samples at inference time, without fine-tuning. The algorithm:

1. Run $N$ diffusion trajectories (particles) in parallel.
2. At intermediate times, evaluate a **potential function** on each particle — a proxy for how likely that trajectory is to produce a high-reward final sample.
3. **Resample**: duplicate promising particles, kill unpromising ones.
4. Continue the diffusion process from the resampled particles.

The theoretical foundation is the **Feynman-Kac formula**, which connects the value function:

$$V_t(x) = \log \mathbb{E}\left[e^{r(X_1)} \mid X_t = x\right]$$

to a path integral weighted by a potential. The optimal steering modifies the SDE drift by $\sigma_t^2 \nabla V_t(x)$, but computing $\nabla V_t$ exactly is intractable. FK steering approximates it empirically using the particle population.

### What Transfers from SMC and What Doesn't

The connection to SMC is primarily in the resampling strategy. Unlike classical SMC or AESMC, the diffusion model provides the dynamics — there is no need for the sequential inference machinery that is SMC's raison d'être. Each particle follows an explicit trajectory defined by the generative SDE/ODE, which is fundamentally different from the stochastic proposal-based propagation in classical SMC.

What survives the transfer to diffusion models:

- **Resampling**: duplicate promising trajectories, kill unpromising ones — the simple core of SMC.
- **Potential design**: choosing good scoring functions for intermediate states, analogous to proposal design in SMC.
- **Theoretical guarantees**: consistency and convergence results from the SMC/Feynman-Kac literature.

What becomes irrelevant:

- **Sequential Bayesian inference**: there are no observations to incorporate; the diffusion model provides the full generative process.
- **Proposal learning**: the dynamics are fixed by the pre-trained diffusion model (though advanced variants can modify the drift using twisted proposals).

### Practical Limitations

The fundamental constraint is that each particle is a full diffusion trajectory in a high-dimensional space. The particle budget $N$ is limited by compute, and in high dimensions, even moderate $N$ may be insufficient to adequately cover the relevant regions of space. Resampling can lead to **particle collapse** — all particles converging to copies of a single ancestor — destroying the diversity needed for exploration.

This is the same curse of dimensionality that affects all Monte Carlo methods, repackaged in the diffusion model setting.

## Relationship to Other Reward-Steering Approaches

FK steering occupies a specific point in the design space of reward-aligned generation:

- **Fine-tuning** (RLHF, reward-weighted training): modifies the model weights. Expensive training, risk of mode collapse, but amortized at inference time.
- **Classifier/reward guidance**: adds $\nabla_x r(x_t)$ to the diffusion drift. Simple but requires differentiable rewards and can produce artifacts.
- **Meta Flow Maps**: learns a stochastic flow map that amortizes posterior sampling, enabling efficient reward alignment without particles. Requires additional training but avoids the per-sample cost of running $N$ trajectories.
- **FK steering**: no additional training, works with any reward function, but pays with $N\times$ inference cost. A classic **compute-at-inference** vs **compute-at-training** tradeoff.
