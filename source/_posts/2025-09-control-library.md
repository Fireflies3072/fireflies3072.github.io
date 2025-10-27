---
title: Mastering Control with PyController Library
date: 2025-09-20 15:30:09
tags: [control, programming]
categories: [Projects]
cover: https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/cover.jpg
mathjax: true
excerpt: The world of robotics and automation hinges on effective control systems. Whether it's a drone landing precisely on a moving platform, a chemical plant maintaining a perfect reaction temperature, or a complex spacecraft navigating the cosmos, controllers are the brains that manage system dynamics. PyController is an open-source Python library designed to provide a cohesive framework for implementing, testing, and comparing cutting-edge control algorithms across diverse simulation environments.
---

## Introduction

The world of robotics and automation hinges on effective **control systems**. Whether it's a drone landing precisely on a moving platform, a chemical plant maintaining a perfect reaction temperature, or a complex spacecraft navigating the cosmos, controllers are the brains that manage system dynamics. **PyController** is an open-source Python library designed to provide a cohesive framework for implementing, testing, and comparing cutting-edge control algorithms across diverse simulation environments.

PyController makes advanced control techniques accessible, acting as a laboratory where theory meets practice. It’s built on the **Gymnasium** standard, ensuring compatibility with a wide range of popular simulation environments. It is more than just a collection of algorithms; it's a framework for competitive analysis. It offers a standardized interface for various controllers and environments, allowing researchers and engineers to quickly benchmark the performance of different strategies, from classic proportional control to modern data-driven methods. This blog post explores the algorithms and environments currently supported by the library, showcasing its utility for both education and development.

Interested in diving in? You can find the source code and contribute here: https://github.com/Fireflies3072/PyController .



## Algorithms

PyController provides robust implementations of several high-impact control strategies, each suited to different challenges and system complexities.



### PID Controller

The **Proportional-Integral-Derivative (PID) controller** is the workhorse of industrial control. It is arguably the most common control loop feedback mechanism in use today. A PID controller continuously calculates an error value as the difference between a desired setpoint and a measured process variable, and then applies a correction based on three terms:

- **Proportional (P) term:** Responds to the current error.
- **Integral (I) term:** Accounts for past errors, eliminating steady-state offset.
- **Derivative (D) term:** Predicts future errors based on the rate of change.

PID controllers are simple, effective, and require no explicit model of the system, making them incredibly versatile. The library often provides specialized wrappers, like `PID_RocketLander`, to coordinate multiple PID loops for complex systems.



### Model Predictive Control (MPC)

**Model Predictive Control (MPC)** is a sophisticated control method that explicitly uses a system's model to predict future outcomes and optimize current actions. At each time step, MPC solves an online optimization problem over a finite, receding prediction horizon. The core process involves:

1. **Predicting:** Using a mathematical model to forecast the system's behavior for a set number of future steps.
2. **Optimizing:** Calculating the sequence of control inputs that minimizes a cost function (e.g., tracking error, control effort) while satisfying constraints.
3. **Executing:** Applying only the first control action from the optimal sequence.

MPC is renowned for its ability to handle system constraints (like actuator limits) and achieve excellent performance, but it requires an accurate system model and significant computational power.



### Data-Enabled Predictive Control (DeePC)

**Data-enabled Predictive Control (DeePC)** represents a paradigm shift from model-based control (like MPC) to **data-driven control**. Instead of relying on a complex, identified mathematical model, DeePC uses only input-output data collected from the system. Its theoretical foundation is the **fundamental lemma of Willems**, which states that any future trajectory of a linear system can be represented as a linear combination of its past trajectories.

The controller formulates an online **Quadratic Program (QP)** where the decision variable is a vector of weights ($g$) that combines past system trajectories from a large data matrix (**Hankel matrix**). This optimization finds the control inputs that drive the system to the desired state while being consistent with observed system dynamics. DeePC is a powerful technique for systems where developing an accurate physical model is difficult or impossible.



## Environments

PyController leverages the Gymnasium standard to support complex, simulation-based environments for control testing.



### Rocket Lander

The **Rocket Lander** environment simulates the challenging task of landing a reusable rocket stage, akin to a SpaceX Falcon 9, onto an ocean barge. The physics are based on the Box2D engine, offering a realistic, planar (2D) simulation. The goal is to safely bring the lander to a specific target position with minimal velocity and fuel consumption. This scenario is a classic challenge for control algorithms due to the inherent instability, non-linear dynamics, and external disturbances.

**PID**

The specialized `PID_RocketLander` handles the multi-input/multi-output (MIMO) nature of the problem, controlling main engine thrust, side-engine forces, and nozzle vectoring.

<video width="500" height="400" controls>   <source src="https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rl_pid.mp4" type="video/mp4">   Not support video </video>

**MPC**

To use MPC, a calculated system matrix is needed for the controller to optimize the multi-dimensional thrust and vectoring inputs over a prediction horizon to achieve a soft and accurate touchdown.

<video width="500" height="400" controls>   <source src="https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rl_mpc.mp4" type="video/mp4">   Not support video </video>

**DeePC with data from PID controller**

Data is collected from the rocket's flight and used to predict the required thrust and vectoring for a stable descent.

<video width="500" height="400" controls>   <source src="https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rl_deepc_pid.mp4" type="video/mp4">   Not support video </video>

![](https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rl_deepc_pid.png)

**DeePC with data from random actions**

<video width="500" height="400" controls>   <source src="https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rl_deepc_random.mp4" type="video/mp4">   Not support video </video>

![](https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rl_deepc_random.png)



### Room Temperature

The **Room Temperature** environment models a simple, yet fundamental, problem in process control: maintaining a desired interior temperature against external thermal dynamics. This is a **Single-Input, Single-Output (SISO)** system. The state is the room's temperature ($T_{\text{current}}$), and the input is the electric **power supplied by a heater (**$P_{\text{heater}}$**)**.

The simulation is based on **Newton's Law of Cooling**, where the rate of temperature change is proportional to the difference between the room temperature and the outside temperature ($T_{\text{out}}$), plus the heat added by the controller. Key system parameters include the **cooling constant**, the **initial room temperature**, and the constant **outside temperature**. The primary objective is to drive and maintain the room temperature exactly at the comfortable setpoint of **22°C**. A control run is considered **successful** if the temperature remains within $\mathbf{0.1^\circ C}$ of the $22^\circ C$ setpoint for **20 continuous samples**.

**PID**

![](https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rt_deepc_random.png)

**DeePC with data from random actions**

![](https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/rt_deepc_random.png)



### Inverted Pendulum

The **Inverted Pendulum** is a canonical problem in control theory. It involves a pendulum mounted on a cart, where the goal is to keep the pendulum balanced vertically upright by applying forces to the cart. This environment is inherently unstable and non-linear, making it a perfect testbed for evaluating the stability and response of control algorithms. The complexity can vary from a simple one-dimensional setup to a full-fledged cart-pole system.

**PID**

Although a standard PID can be respond to position and angle at the same time with a weight, its performance is very poor. A cascaded PID is a applied here. The first PID takes position input and gives a target angle output. The second PID takes the actual angle and target angle and gives force output.

<video width="480" height="480" controls>   <source src="https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/ip_pid.mp4" type="video/mp4">   Not support video </video>

**DeePC with data from PID controller**

<video width="480" height="480" controls>   <source src="https://fireflies3072.blob.core.windows.net/blog/images/2025-09-control-library/ip_deepc_pid.mp4" type="video/mp4">   Not support video </video>



## Future Updates

The PyController library is committed to continuous growth. Our immediate focus is on expanding the stability and examples for existing environments. Specifically, we plan to address the current difficulties with the **Inverted Pendulum** examples, tuning parameters, and exploring new control variants to ensure robust and illustrative performance. Additionally, we aim to integrate more complex and challenging Gymnasium environments in the near future.



## Acknowledgement

The PyController project benefits greatly from prior foundational work. The **Rocket Lander** environment, a Box2D Gymnasium simulation of a Falcon 9 ocean barge landing, was originally created by Reuben Ferrante and later modified by Dylan Vogel and Gerasimos Maltezos for the 2023 Computation Control course at ETH Zurich. Their work provides a rich and realistic simulation for testing advanced control systems.