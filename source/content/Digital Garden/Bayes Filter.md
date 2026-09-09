---
title: Bayes Filter
date: 2026-09-09
tags:
  - Concept
---
A Bayes filter abandons the idea of knowing a single, exact state. Instead, it tracks a **Belief distribution**—a probability density function representing how likely every possible state is, given all past sensor readings and control inputs.

**1. Predict (The Prior):**

You integrate across all possible previous states, applying your physical motion model ($p(x_t \mid x_{t-1}, u_t)$).

$$\overline{bel}(x_t) = \int p(x_t \mid x_{t-1}, u_t) \space bel(x_{t-1}) \space dx_{t-1}$$

**2. Update (The Posterior):**

You multiply that predicted belief by the likelihood of getting your current sensor reading ($p(z_t \mid x_t)$), and normalize it with a constant $\eta$.

$$bel(x_t) = \eta \space p(z_t \mid x_t) \space \overline{bel}(x_t)$$