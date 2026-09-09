---
title: "Understanding the Kalman Filter: A Step-by-Step 1D Motion Example"
date: 2026-09-09
tags:
  - odometry
  - Sensors
---

In any autonomous system, you rarely have perfect information. Your internal physics models naturally drift over time, and your hardware sensors always come with a degree of noise. The Kalman filter is a mathematical algorithm that bridges this gap, fusing these two imperfect sources of information to estimate a system's true state.

This post demystifies the standard linear Kalman filter from first principles. Using a simple 1D motion model, we will walk step-by-step through the matrix algebra behind the recursive Predict and Update loops. We will translate basic Newtonian kinematics into state vectors, track how uncertainty grows and shrinks using covariance matrices, and mathematically show how the Kalman Gain uses a simple position measurement to automatically correct an unmeasured velocity state.
### Introduction
Kalman filter at its core is a recursive bayes estimator ([[Bayes Filter]]). This means that the estimation of the state from previous time steps and the current measurements are required to calculate the current state.
One of the assumptions while using Kalman Filter are that all the involved probablity distribution are required to be Gaussian Distributions. 
(For cases following other probablisitic distribution, other filters like Particle Filter, etc can be used.)

At its core the kalman filter has 2 steps ie.
- Prediction
- Correction

Also it can be noted that the Kalman Filter is the optimal solution for linear models and Gaussian distributions.

### Perdiction Step
This state involves prediction of the future state using the current state and the control input sent. It projects the state forward in time. The prediction phase asks the question "Based on the last state and the control input recieved, what should be the current state of the system?".

Therefore we can map it to:

#### Prediction of the state
$$\overline{\mu}_t = A_t \mu_{t-1} + B_t u_t$$

where:
- $\overline{\mu}_t$: The mean of the predicted belief $\overline{bel}(x_t)$ one time step later, before incorporating the measurement.
- $A_t$: A square matrix of size $n \times n$ representing the linear state transition. It applies the deterministic version of the state transition function. This can also be described as how will the state change without any control input.
- $\mu_{t-1}$: The mean of the belief at time $t-1$.
- $B_t$: A matrix of size $n \times m$ that is multiplied by the control vector. This describes how the control input will change the state from $t - 1$ to $t$.
- $u_t$: The control vector at time $t$.

This equation describes the physics and the control input of the system to predict the next state. Lets assume the exmaple of a 1D car where, each state using 2 variables $[position(p), velocity(v)]$.

From basic Newtonian physics, assuming constant acceleration over a tiny time step ($\Delta t$), the equations of motion are:

1. **Position:** $p_{new} = p_{old} + (v_{old} \times \Delta t) + (\frac{1}{2} \times a \times \Delta t^2)$
2. **Velocity:** $v_{new} = 0 + v_{old} + (a \times \Delta t)$

To make this computable for a Kalman filter, we pack these two separate equations into a single matrix operation: $\overline{\mu}_t = A_t \mu_{t-1} + B_t u_t$.

- **$\mu$ (State):** We stack position and velocity into a vector: $\begin{bmatrix} p \\ v \end{bmatrix}$
- **$u_t$ (Control Input):** The known acceleration commanded to the motors: $\begin{bmatrix} a \end{bmatrix}$

By mapping the physics to matrices, we extract $F$ and $B$:

$$\begin{bmatrix} p_{new} \\ v_{new} \end{bmatrix} = \underbrace{\begin{bmatrix} 1 & \Delta t \\ 0 & 1 \end{bmatrix}}_{A_t} \begin{bmatrix} p_{old} \\ v_{old} \end{bmatrix} + \underbrace{\begin{bmatrix} \frac{1}{2}\Delta t^2 \\ \Delta t \end{bmatrix}}_{B_t} \underbrace{\begin{bmatrix} a \end{bmatrix}}_{u_t}$$

#### Predict the Covariance
$$\overline{\Sigma}_t = A_t \Sigma_{t-1} A_t^T + R_t$$where
- $\overline{\Sigma}_t$: The predicted uncertainty. Note that uncertainty grows during the predict step because you are projecting forward without new ground truth.
- $R_t$: The Process Noise Covariance. This is a crucial tuning parameter. It represents the uncertainty in your physics model.

Continuing the motion example, since the previous state estimate wasnt perfect and the real world doesn't always obey basic laws of motion.
The uncertainty will increase when you predict the future state without looking at the sensors.

The Covariance Matrix ($\Sigma$) tracks the variance of your state variables. 
So in this case for a 1D position and velocity state, $\Sigma$ would be a $2 \times 2$ matrix:

$$\Sigma = \begin{bmatrix} Var(position) & Cov(pos, vel) \\ Cov(vel, pos) & Var(velocity) \end{bmatrix}$$

This matrix represents how the errors across velocity and position correlate with each other (represented in the off-diagonals) and how unsure the estimation of the position and velocity are (represented in the diagonals).

### Correction Step
In this step, the filter transforms the predicted belief into the desired posterior belief by incorporating the new sensor measurement $z_t$.

#### Kalman Gain Computation
$$K_t = \overline{\Sigma}_t C_t^T (C_t \overline{\Sigma}_t C_t^T + Q_t)^{-1}$$
where
- **$K_t$**: The Kalman gain. It specifies the degree to which the measurement is incorporated into the new state estimate.
- **$C_t$**: The measurement matrix. It maps the internal state space to the measurement space.
- **$Q_t$**: The measurement noise covariance. The distribution of the measurement noise is a multivariate Gaussian with zero mean and covariance $Q_t$.

So for the 1D car, we can assume the sensor to be a GPS sensor (for simplicity we will assume only one sensor). Therefore, we can say that
- Predicted Covariance ($\overline{\Sigma}_t$): $\begin{bmatrix} Var(p) & Cov(p,v) \\ Cov(v,p) & Var(v) \end{bmatrix}$
- Measurement Matrix ($C_t$): $\begin{bmatrix} 1 & 0 \end{bmatrix}$
- Measurement Noise ($Q_t$): A scalar value representing the variance of the GPS error, which we will call $Var(sensor)$.

##### Note:
Here you can see that we have set $C_t$ as $[1, 0]$ since we are using a position sensor ie. GPS. So we can get 

$$C_t \overline{\mu}_t = \begin{bmatrix} 1 & 0 \end{bmatrix} \begin{bmatrix} p \\ v \end{bmatrix} = (1 \times p) + (0 \times v) = p .
$$

If we use an additional sensor for velocity like a speedometer, we can set $C_t$ as $2 \times 2$ identity matrix.

So now solving for the current scenario we will get 

$$\overline{\Sigma}_t C_t^T = \begin{bmatrix} Var(p) & Cov(p,v) \\ Cov(v,p) & Var(v) \end{bmatrix} \begin{bmatrix} 1 \\ 0 \end{bmatrix} = \begin{bmatrix} Var(p) \\ Cov(v,p) \end{bmatrix}$$

and therefore we can calculate the innovation covariance ie. $(C_t \overline{\Sigma}_t C_t^T)$
$$C_t \overline{\Sigma}_t C_t^T = \begin{bmatrix} 1 & 0 \end{bmatrix} \begin{bmatrix} Var(p) \\ Cov(v,p) \end{bmatrix} = Var(p)$$

Here we can also see that the predicted uncertainty of the GPS is the predicted uncertainty of the position.
So now we can just add the sensor noise ie $Q_t = Var(Sensor)$ and calculate the Kalman Gain as 

$$K_t = \begin{bmatrix} Var(p) \\ Cov(v,p) \end{bmatrix} \frac{1}{Var(p) + Var(sensor)}$$

$$K_t = \begin{bmatrix} \frac{Var(p)}{Var(p) + Var(sensor)} \\ \frac{Cov(v,p)}{Var(p) + Var(sensor)} \end{bmatrix} = \begin{bmatrix} K_{position} \\ K_{velocity} \end{bmatrix}$$

(since the innovation covariance is scalar we can just transfer the power to division)


#### Update the State Estimation
$$\mu_t = \overline{\mu}_t + K_t(z_t - C_t \overline{\mu}_t)$$

where:
- **$\mu_t$**: The updated mean of the posterior state.
- **$z_t$**: The actual measurement received from the sensor.
- **$C_t \overline{\mu}_t$**: The measurement predicted according to the measurement probability.
- **$(z_t - C_t \overline{\mu}_t)$**: The deviation of the actual measurement from the predicted measurement. This line adjusts the mean in proportion to both this deviation and the Kalman gain $K_t$.

So now assume that teh predicted estimate before the measurement was $\overline{\mu}_t = \begin{bmatrix} p_{pred} \\ v_{pred} \end{bmatrix}$, and the GPS measurement gave us a raw positional reading ie. $z_{pos}$

Now we can appy teh measurement matrix to our predicted state: 

$$\begin{bmatrix} 1 & 0 \end{bmatrix} \begin{bmatrix} p_{pred} \\ v_{pred} \end{bmatrix} = p_{pred}$$

Now we can apply the Kalman Gain and Update

$$\begin{bmatrix} p_{new} \\ v_{new} \end{bmatrix} = \begin{bmatrix} p_{pred} \\ v_{pred} \end{bmatrix} + \begin{bmatrix} K_p \\ K_v \end{bmatrix} (z_{pos} - p_{pred})$$

$$\begin{bmatrix} p_{new} \\ v_{new} \end{bmatrix} = \begin{bmatrix} p_{pred} + K_p(z_{pos} - p_{pred}) \\ v_{pred} + K_v(z_{pos} - p_{pred}) \end{bmatrix}$$

##### Note
Look at the velocity update equation: $v_{new} = v_{pred} + K_v(z_{pos} - p_{pred})$. The GPS did not measure velocity. Yet, the filter updates the velocity by taking the _position error_ and scaling it by the _velocity gain_ ($K_v$). If you were further ahead of your predicted position than expected, the math correctly deduces you must have been moving faster than expected, and fixes the velocity state automatically.

#### Update the Covariance
$$\Sigma_t = (I - K_t C_t) \overline{\Sigma}_t$$

Now we can calculate the update as 

$$\Sigma_t = \begin{bmatrix} (1 - K_p) & 0 \\ -K_v & 1 \end{bmatrix} \begin{bmatrix} Var(p) & Cov(p,v) \\ Cov(v,p) & Var(v) \end{bmatrix}$$

So the new pose variance becomes 

$$Var(p)_{new} = (1 - K_p) Var(p)_{old}$$
##### Note
If your GPS is perfect, $K_p$ approaches $1$. Therefore, $(1 - 1) = 0$. Your position variance drops to exactly zero. You are 100% certain of where you are. If your GPS is pure noise, $K_p$ approaches $0$. Therefore, $(1 - 0) = 1$. Your variance remains unchanged ($1 \times Var(p)_{old}$). The filter ignores the bad sensor and retains its original uncertainty.