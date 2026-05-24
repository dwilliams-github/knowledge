---
title: "Feynman-Kac Steering: Inference-Time Reward Alignment for Diffusion Models"
topic: generative-models
date: 2026-04-25
tags: [Feynman-Kac, diffusion, reward-steering, SMC, particle-methods, inference-time-scaling]
source: agent session 2026-04-25
papers:
  - "Singhal, Horvitz, Teehan, Ren, Yu, McKeown, Ranganath (2025) — [arXiv:2501.06848](https://arxiv.org/abs/2501.06848)"
  - "Del Moral (2004) — Feynman-Kac Formulae: Genealogical and Interacting Particle Systems with Applications. DOI: 10.1007/978-1-4684-9393-1"
  - "Hairer & Weare (2014) — Improved diffusion Monte Carlo — [arXiv:1207.2866](https://arxiv.org/abs/1207.2866)"
---

# Feynman-Kac Steering: Summary and Key Insights

## Paper Reference

**"A General Framework for Inference-time Scaling and Steering of Diffusion Models"**
Singhal, Horvitz, Teehan, Ren, Yu, McKeown, Ranganath ([arXiv:2501.06848](https://arxiv.org/abs/2501.06848), January 2025)

---

## Core Idea

FK steering is an inference-time method for steering pre-trained diffusion models toward high-reward samples, without fine-tuning. The algorithm:

1. Run $N$ diffusion trajectories (particles) in parallel
2. At intermediate times, score each trajectory using a potential function
3. Resample: duplicate high-scoring particles, kill low-scoring ones
4. Continue the diffusion process from the resampled particles

The framework requires no additional training and works with arbitrary reward functions, including non-differentiable ones.

## Theoretical Foundations

### The Feynman-Kac Formula

The name derives from the Feynman-Kac formula in stochastic analysis, which connects the solution of a PDE with a potential term:

$$\frac{\partial u}{\partial t} + \frac{1}{2}\sigma^2 \nabla^2 u + V(x)\thinspace u = 0$$

to an expectation over paths of a stochastic process, weighted by a multiplicative functional:

$$u(x, t) = \mathbb{E}\left[f(X_T)\thinspace \exp\!\left(-\int_t^T V(X_s)\thinspace ds\right) \;\bigg|\; X_t = x\right]$$

This is essentially a path integral formulation — sampling trajectories weighted by a potential.

### FK Interacting Particle Systems (FK-IPS)

FK steering builds on Del Moral's framework of Feynman-Kac interacting particle systems (FK-IPS), originally developed for rare event simulation. The connection is direct: generating a high-reward sample from a diffusion model is a rare event problem. Most samples from the base model have mediocre reward; the goal is to find trajectories that land in the high-reward tail.

The "interacting" terminology refers to the fact that each particle's survival probability during resampling depends on the scores of all other particles (since resampling weights are relative). However, between resampling steps, particles evolve independently — there is no pairwise coupling during propagation.

### Key FK-IPS References

- **Del Moral, P.** (2004). *Feynman-Kac Formulae: Genealogical and Interacting Particle Systems with Applications*. Probability and Its Applications. Springer, New York. ISBN: 978-0-387-20268-6. DOI: 10.1007/978-1-4684-9393-1

- **Hairer, M. and Weare, J.** (2014). Improved diffusion Monte Carlo. *Communications on Pure and Applied Mathematics*, 67(12):1995–2021. DOI: 10.1002/cpa.21526. [arXiv:1207.2866](https://arxiv.org/abs/1207.2866)

## Connection to Sequential Monte Carlo (SMC)

FK steering applies SMC-like particle methods to diffusion model generation. However, the transfer from classical SMC to diffusion steering is partial — most of SMC's machinery becomes irrelevant because the diffusion model provides the dynamics.

### What Transfers from SMC

- **Resampling**: the core algorithmic move — duplicate promising particles, kill unpromising ones. This prevents weight degeneracy, where one particle would otherwise accumulate nearly all the weight.
- **Potential design**: choosing good scoring functions for intermediate states, analogous to proposal design in SMC.
- **Theoretical guarantees**: consistency and convergence results from the SMC/Feynman-Kac literature.

### What Does Not Transfer

- **Sequential Bayesian inference**: in classical SMC, particles track a posterior distribution given streaming observations. In FK steering, there are no observations — the diffusion model provides the full generative process.
- **Proposal learning**: the dynamics are fixed by the pre-trained diffusion model (though advanced variants can modify the drift).

In FK steering, each particle follows an explicit trajectory defined by the generative SDE/ODE. This is fundamentally different from classical SMC (and AESMC), where particles are sampled from learned proposals and do not have deterministic trajectories.

## Three Independent Design Choices

The paper's methodological contribution centers on identifying three independent design axes for FK steering:

### 1. Proposal Generator

How particles propagate between resampling steps.

- **Simplest choice**: the pre-trained diffusion model itself — run the base SDE/ODE unmodified.
- **Advanced choice**: modify the drift to bias trajectories toward high-reward regions (twisted proposals), analogous to using informed proposals rather than prior dynamics in classical SMC.

### 2. Intermediate Rewards

How to define a meaningful reward signal at intermediate noise levels $t < 1$, given that the true reward $r(x)$ is only defined on final clean samples.

- **Denoiser-based (standard approach)**: compute the denoiser prediction $`\hat{x}_0 = \mathbb{E}[x_0 \mid x_t]`$ at intermediate state $`x_t`$, then evaluate the reward on this prediction: $`r(\hat{x}_0(x_t))`$. This is cheap (single forward pass, no gradients required) and always available, but the denoiser prediction is inaccurate at early times when $`x_t`$ is mostly noise — the posterior mean is a blurry average over many possible outcomes.
- **Physics-based (for scientific applications)**: evaluate a physical model (energy function, conservation law, structural constraint) directly on the intermediate state $`x_t`$, without needing to predict the final sample. This can be more reliable than denoiser-based rewards when the physical constraint applies at all noise levels, though domain-specific evaluation is needed to confirm the physical model remains meaningful under noise.

The paper proposes several novel intermediate reward strategies and demonstrates empirically that these improve over the standard denoiser-based approach.

### 3. Potential

How intermediate rewards are converted into resampling weights. Different mappings from reward scores to particle weights (multiplicative, additive, tempered, etc.) constitute different potential designs.

The paper claims these three choices are approximately independent — a reasonable proposal generator is amenable to any good potential — allowing them to be optimized separately. This independence likely holds when the proposal is adequate (particles have non-negligible probability of reaching high-reward regions) but could break down in genuinely rare-event regimes where the target is far in the tails.

## Key Empirical Results

The paper demonstrates FK steering across multiple modalities:

- **Image generation**: FK steering with $k=4$ particles outperforms fine-tuning on prompt fidelity and aesthetic quality for text-to-image latent diffusion models, without requiring reward gradients. Even $k=2$ particles outperforms fine-tuned models on prompt fidelity.
- **Model efficiency**: smaller models (0.8B parameters) with FK steering can outperform larger models (2.6B parameters) on prompt fidelity using fewer FLOPs, suggesting inference-time compute (more particles) can substitute for model-size compute.
- **Text generation**: FK steering steers text diffusion models to generate samples with competitive linguistic acceptability and perplexity relative to autoregressive models.
- **Rare-attribute generation**: FK steering increases the toxicity rate of a text diffusion model from 0.3% to 64.7% with $k=8$ particles, outperforming both gradient guidance and best-of-$n$. This has applications in responsible AI and red-teaming.
- **Composability with fine-tuning**: FK steering applied to already fine-tuned models provides additional performance benefits.

## Advantages Over Alternative Approaches

FK steering occupies a specific point in the design space of reward-aligned generation:

| Method | Training Cost | Inference Cost | Reward Requirements |
|--------|--------------|----------------|-------------------|
| **Fine-tuning** (RLHF) | High | Amortized (1x) | Differentiable (for gradients) |
| **Classifier/reward guidance** | None | ~1x + gradient cost | Differentiable |
| **FK steering** | None | $N\times$ trajectories | Evaluable only (black-box) |
| **Meta Flow Maps** | Moderate | ~1x | Differentiable |

Key advantages of FK steering:

- **No additional training**: works with any pre-trained diffusion model
- **Black-box rewards**: only requires reward evaluation, not gradients — compatible with non-differentiable rewards, human preference models, or physics-based constraints
- **No mode collapse risk**: unlike fine-tuning approaches

Key limitations:

- **Inference cost**: scales linearly with particle count $N$
- **Particle collapse**: in high dimensions, resampling can cause all particles to converge to copies of a single ancestor, destroying diversity
- **Potential design**: the optimal potential (the value function $`V_t(x) = \log \mathbb{E}[e^{r(X_1)} \mid X_t = x]`$) is intractable; practical potentials are approximations whose quality limits overall performance

## Unification of Prior Work

The paper shows that existing particle-based sampling methods, including twisted diffusion sampler (TDS) and the method of Li et al. (2024), are specific instances of the FK-IPS framework. FK steering generalizes these approaches and enables new combinations of proposals, potentials, and reward models.

## Connections to Related Frameworks

- **[Sequential Monte Carlo](sequential-monte-carlo.md)**: FK steering draws on SMC's resampling machinery and Feynman-Kac theoretical foundations, while discarding the sequential Bayesian inference components that are unnecessary given the diffusion model's generative process.
- **Meta Flow Maps** (Potaptchik et al., 2026): solves the same reward-alignment problem but amortizes the posterior sampling into a learned stochastic flow map, trading training cost for reduced inference cost. A compute-at-training vs compute-at-inference tradeoff.
- **[Stochastic interpolants / flow matching](stochastic-interpolants.md)**: FK steering operates on top of models trained with any of these frameworks — it is agnostic to the training procedure and only requires the ability to run the generative process.
- **Diffusion Monte Carlo / Quantum Monte Carlo**: the propagate-weight-resample loop in FK steering is closely related to walker-based methods in computational physics, where ensembles evolve under stochastic dynamics with branching to sample target distributions.
