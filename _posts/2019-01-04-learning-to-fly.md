---
bg: "glider.jpg"
layout: post
title:  "Learning to Fly"
crawlertitle: "Learning to Fly - Tâm Le Minh"
summary: "Autonomous Glider using Reinforcement Learning"
date:   2019-01-04 20:09:47 +0700
categories: posts
author: Tâm Le Minh
type: project
github: https://github.com/SuReLI/L2F-sim
---

The idea of an autonomous glider instead of a regular engine-powered UAVs is very attractive. 
The absence of engines makes a glider more silent and more economical, but its piloting differs 
sensibly from a normal plane. As a Master student, I contributed to *Learning to Fly*, a flight dynamics 
simulator in C++ developed by the [SuReLI](https://sureli.github.io) research group at ISAE-SUPAERO. The 
simulator provides an environment to test Reinforcement Learning algorithms allowing a model of glider 
to learn the optimal guidance and controls for an autonomous flight.

* ToC
{:toc}

## Background

### Flight dynamics

In fact, any object can fly until it runs out of energy. The energy can be calculated from his 
state. A very simplifieed approach is to consider only two parameters: its speed (kinetic energy) 
and its altitude (potential energy). In aviation, there is a constant trade-off between speed and 
altitude: At constant energy, when going down, the plane would get more speed. Inversely, when 
climbing, the plane would lose speed.

Now, the energy is not constant. For example, in order to preserve its speed even when climbing, 
the plane to increase its energy by using an external source. Also, in a real scenario, the plane 
constantly loses energy because of opposing forces to its movement, namely the drag and the weight. 
Again, to preserve its speed and its altitude, the plane must use an external source. For regular 
planes, this external source is the thrust from their engines. When they run out of fuel, they cannot 
keep their energy anymore, so they will eventually slow down and fall down to the ground.

[![flightdynamics]({{ site.images | relative_url }}/flightdynamics.png)]({{ site.images | relative_url }}/flightdynamics.png)
<div style="text-align: center">
<em>Main flight dynamics forces</em>
</div>

Gliders cannot benefit from thrust, so they must rely on air currents. Glider pilots soar by taking 
advantage of columns of rising air called thermals. They fly in circles to stay in the thermal until 
they reach the desired altitude. The idea of the project is to use Reinforcement Learning techniques 
to let the glider learn the optimal way to use the thermals.

[![thermal1]({{ site.images | relative_url }}/thermal1.png)]({{ site.images | relative_url }}/thermal1.png)
<div style="text-align: center">
<em>Allen model for thermals</em>
</div>

### Reinforcement Learning

Reinforcement learning relies typically on Markov decision processes. In this framework, the time is
discrete and there is a finite set of possible states. Between steps, there are transitions that can be 
enabled through actions. 

At each time step, the object is at one particular state $$s$$, and there is a finite set of actions $$A$$. It 
chooses one particular action that will allow it to move to a state $$s'$$. This transition is associated 
with a reward $$r_a(s,s')$$. The goal of reinforcement learning algorithms is to maximize the cumulative reward, 
meaning the total reward over the succesive steps of a run.

The most basic RL algorithm is the Q-learning algorithm. At each state $$s$$, a "quality" function $$Q(s,a)$$ can 
be evaluated for each possible action. A table can be built with values of $$Q$$ with each row representing one state, 
and each column one action. 

The values in the table are arbitrarily initialized. Then at the step $$t$$, at the state $$s_t$$, the action 
$$a_t$$ is selected, which enables a transition to $$s_{t+1}$$. The obtained reward $$r_t$$ is then used to assess 
the value of $$Q(s_t,a_t)$$ is updated with:

$$Q_{t+1}(s_{t},a_{t}) = (1-\alpha_t) \cdot \underbrace{Q_{t}(s_{t},a_{t})}_{\text{old value}} + \underbrace{\alpha_t}_{\text{learning rate}} \cdot \overbrace{ {\bigg (}\underbrace{r_{t}}_{\text{reward}}+ \underbrace{\gamma_t}_{\text{discount factor}} \cdot \underbrace{\max _{a}Q_{t}(s_{t+1},a)}_{\text{estimate of optimal future value}}{\bigg )}} ^{\text{learned value}}$$

which can be also written in the form:

$$Q_{t+1}(s_{t},a_{t}) = Q_{t}(s_{t},a_{t}) + \alpha_t \cdot \delta_t$$

where $$\delta_t = r_t + \gamma_t \cdot \max_{a}Q_{t}(s_{t+1},a) - Q_{t}(s_{t},a_{t})$$ is also called 
Temporal-Difference (TD) error.

Therefore, $$Q(s_t,a_t)$$ is updated with the weighted average between the old value and the value learned from the 
new information. The learning process can be influenced by two parameters:
- the discount factor $$\gamma_t$$, setting the importance of future rewards ($$0 \lt \gamma_t \lt 1$$). A lower 
$$\gamma_t$$ means the later rewards are worth less than those received now, so the algorithm will focus on optimizing 
the near future. A $$\gamma_t$$ closer to $$1$$ would make the algorithm find better long-term rewards. Usually, 
$$\gamma_t$$ is set low at the beginning, but is increased progressively during the training.
- the learning rate $$\alpha_t$$, often fixed $$\alpha_t = \alpha$$, determining the "weights" ($$0 \lt \alpha_t \lt 1$$). 
Indeed, $$\alpha = 0$$ means the algorithm relies exclusively on his prior knowledge of $$Q$$. It doesn't learn from the 
new inputs. In constrast, $$\alpha = 1$$ makes the new information override the old values completely. Therefore, $$\alpha$$ 
sets how fast the values should be updated.

<div style="text-align: center">
<img src="/assets/images/q-learning-table.png" alt="q-learning-table" width="60%">
<em>Example of Q-learning table</em>
</div>

## Implementation

The problem of the glider is more complicated than a classic RL problem. For the implementation, some simplifications were made.
For instance, constant values $$\alpha$$ and $$\gamma$$ are chosen for $$\alpha_t$$ and $$\gamma_t$$. One example of future 
improvement can be to let these values change, for example $$\gamma$$ to decrease.

### Restrained action-space

In a real situation, the main controls consist of a stick for pitch and roll control and pedals for yaw control. The stick 
and the pedals are actuators to external control surfaces, which consist of the ailerons, the elevators and the rudder. Their 
positions are what allow the aircraft to be controlled. Let $$m_i$$ be the angle made by the actuator controlling the dimension 
$$i \in (pitch, roll, yaw)$$. Between two timesteps in the simulation, it's more realistic to restrain the movement of these 
controls to a maximum speed. For instance, if the stick is completely oriented to the right, the pilot would need several timesteps 
to put it back to the center. At each step, they can only move the stick with a certain angle $$\pm \delta m_{roll}$$. This is 
necessary to avoid unrealistic behaviors such as high-frequency piloting. 

Typically, at a certain state, an action consists of one input for each dimension. Let $$\delta m_i$$ be the maximum angle magnitude 
for the dimension $$i$$. For one timestep, these inputs can be picked in among three possibilities: 
- $$0$$, no change,
- $$-\delta m_i$$, add an angle $$m_i$$ in the (-) direction (e.g. the stick toward the left)
- $$+\delta m_i$$, add an angle $$m_i$$ in the (+) direction (e.g. the stick toward the right)

Thus with all 3 dimensions, $$[-\delta m_{pitch}, +\delta m_{roll}, 0]$$ means the pilot pushes the stick upfront and to the right 
but doesn't change the pedals position, resulting in a change of $$\delta m_{pitch}$$ for the elevators and $$\delta m_{roll}$$ 
for the ailerons.

Now if the control is already at its maximum angle, for instance the stick is pulled all the way towards the pilot, it cannot be 
pulled any further. That means the command $$+\delta m_{pitch}$$ is forbidden. At each step $$t$$, one must check if all the commands 
are available. These actions can be stored in a set $$A_t$$.

### Continuous state-space

The state-space is continuous, so it doesn't make sense to update $$Q(s,a)$$ independently and discretely in a table. A parametrized 
function approximator can be rather used, i.e. let:

$$Q(s,a) = \phi (s,a)^\intercal \theta$$ 

where $$\phi (s,a)$$ is a vector of features built from $$s$$ and $$a$$ and $$\theta$$ is a vector of parameters. Instead of 
learning values of $$Q$$, we can learn $$\theta$$. Replacing $$Q$$ with $$\theta$$ in the Q-learning update 
equation, we get:

$$\theta_{t+1} = \theta_t + \alpha \cdot \delta_t \cdot \nabla_{\theta} Q_t$$

which finally gives:

$$\theta_{t+1} = \theta_t + \alpha \cdot \delta_t \cdot \phi_t$$

Nevertheless, the way to build $$\phi$$ has an influence on how well the algorithm does. A basic feature vectors consists 
simply of the observable state variables from $$s$$ (e.g. altitude, control surfaces position, speed, etc.) and the action 
vector $$a$$ (e.g. variations in the control surfaces). In our implementation, we have enriched $$\phi$$ with cross-products 
between all these variables, such that:

$$\phi = \begin{bmatrix} z \\ \dot{z} \\ .. \\ m_{roll} \\ m_{yaw} \\ z^2 \\ z \cdot \dot{z} \\ .. \\ m_{roll} \cdot m_{yaw} \\ m_{yaw}^2 \end{bmatrix}$$

### Reward

The reward should be set to assess the quality of an action. In the glider problem, we have seen how it can be viewed as an 
energy problem. Out of the thermal, the glider should save energy and in the thermal, it should maximize it. Therefore, we 
can reasonably set the reward to be proportional to the variation of energy: 

$$r \propto \dot{z} + \frac{v \cdot \dot{v}}{g}$$

$$\dot{z}$$, $$v$$ and $$\dot{v}$$ are all three elements of the state vector $$s$$. So at the step $$t$$, $$r_t$$ can be 
calculated simply from $$s_t$$.

### Policy

Now, all the blocks are almost ready to apply the Q-learning algorithm to the glider. The only thing left to decide is how to 
decide on which action to take at each step, i.e. the policy. If the estimated best action is chosen at every step (greedy policy), 
the algorithm may converge to a local solution, but probably not the optimal one, because the action-space may not be 
sufficiently explored. Some states and actions may remain unknown or have high uncertainty. One easy way to explore is to 
introduce some random actions. At each step $$t$$, the $$\epsilon$$-greedy policy ($$0 \lt \epsilon \lt 1$$) has a probability of:
- $$\epsilon$$ to pick a random action, $$a_t \in A_t$$, 
- $$1 - \epsilon$$ to pick the estimated optimal action, $$a_t = \arg \max_{a \in A_t} Q_t(s_t,a)$$.

$$\epsilon$$ is then used to set the explore vs. exploit ratio. When $$\epsilon = 0$$, this is a greedy policy. The algorithm 
only relies on known values of $$Q$$. It exploits the prior knowledge of the state and action spaces. The exploration induced 
by selecting random actions is necessary, at least at the beginning, because $$Q$$ is initilialized arbitrarily and without 
any information. Sometimes, the actions randomly picked reveal new paths that would be otherwise ignored. As more of the 
state and the action space is known, the need to explore is less important. To speed up, the learning process, the value of 
$$\epsilon$$ can then be decreased.

### Algorithm

In summary, the algorithm consists of the following steps:

**Define:**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$\phi(s,a)$$, vector of features built with elements from $$s$$ and $$a$$  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$Q(s,a,\theta) = \phi(s,a)^\intercal \theta$$, quality function  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$\gamma =$$ cst., discount factor  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$\alpha =$$ cst., learning rate  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$T =$$ cst., number of steps of the simulation  

**Initialize:**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$a_{prev}$$, $$s_{prev}$$, $$\theta$$ to some value   

**Loop:**  
For step $$t$$ in $$[1,T]$$:  
&nbsp;&nbsp;&nbsp;&nbsp;get state from simulator $$s_t$$  
&nbsp;&nbsp;&nbsp;&nbsp;calculate reward $$r_t$$ with $$s_t$$  
&nbsp;&nbsp;&nbsp;&nbsp;find set of possible actions $$A_t$$ with $$a_{prev}$$  
&nbsp;&nbsp;&nbsp;&nbsp;find optimal action $$a_{max} := \arg \max_{a \in A_t} Q(s_t,a,\theta)$$  
&nbsp;&nbsp;&nbsp;&nbsp;calculate $$\delta_t := r_t + \gamma Q(s_t,a_{max},\theta) - Q(s_{prev},a_{prev},\theta)$$  
&nbsp;&nbsp;&nbsp;&nbsp;update parameters $$\theta := \theta + \alpha \cdot \delta_t \cdot \phi(s_t,a_{prev})$$  
&nbsp;&nbsp;&nbsp;&nbsp;pick next action $$a_t \in A_t$$ with $$\epsilon$$-greedy policy  
&nbsp;&nbsp;&nbsp;&nbsp;update $$a_{prev} := a_t$$ and $$s_{prev} := s_t$$  
Endfor   

## L2F Simulator

The simulator consists of 4 main components:
- an aerological model, representing the atmosphere and simulating thermals. Thermals have a lifetime and 
they have different properties depending on their age. Several models of thermals can be selected.

[![thermal2]({{ site.images | relative_url }}/thermal2.png)]({{ site.images | relative_url }}/thermal2.png)
<div style="text-align: center">
<em>Simulated flight zone with thermals</em>
</div>

- a flight dynamics model, with a physical model of the glider inspired by 
[Beeler et al., 2003](https://ntrs.nasa.gov/search.jsp?R=20040031358), 

- a piloting bot, where the Q-learning algorithm is implemented,

- a stepper, to manage the discrete temporal evolution of the simulation. Interpolation techniques such as 
Euler or RK4 are provided.

All these components are very modular. New models of atmosphere, thermals, aircraft, interpolation techniques, as well 
as piloting or AI algorithm can be implemented and easily plugged in, thanks to the interfaces.

Here is the official website of [Learning to Fly](https://websites.isae-supaero.fr/learning-to-fly). 
You can also check out the latest version of the environment source code in this [Github Repository](https://github.com/SuReLI/L2F-sim).

[![simu-l2f]({{ site.images | relative_url }}/simu-l2f.png)]({{ site.images | relative_url }}/simu-l2f.png)
<div style="text-align: center">
<em>Trajectory of a trained drone</em>
</div>

