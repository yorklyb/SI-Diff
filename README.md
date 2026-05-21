## From the Author
Due to IP policies, we do not release a click-and-run version of SI-Diff. We will provide more supplementary details to the paper to help you reproduce the work. <br>
<br>
We first provide a straightforward introduction to the fundamentals of robot control to help readers avoid confusion. The **second-order dynamic
model** of an n-Degree-of-Freedom torque-controlled robot is as follows: <br>
<img width="383" height="45" alt="image" src="https://github.com/user-attachments/assets/185935f6-f8be-4706-8fdd-37e8ada05b4a" /> <br>
Among these terms, we control the robot by changing $\boldsymbol{\tau}_m$, the joint torque. If the following algorithm is used, the robot is controlled by an **impedance controller**. <br>
<img width="411" height="44" alt="image" src="https://github.com/user-attachments/assets/f1e9c4dc-9249-4717-a752-ffd7d0ecec00" /> <br>
Based on this, if a feedforward force term is added, the controller becomes a feedforward force-based impedance controller, which is the controller used in this work. <br>
<img width="475" height="38" alt="image" src="https://github.com/user-attachments/assets/7b46c52d-1ef3-459a-9022-2ef0053e1b6f" /> <br>
Our force diffusion policy learns how to predict the feedforward force. <br> 
<br>
Note that we rely on the error term e to drive the end effector (EE) to the desired position. In other words, we need to first define a desired position or trajectory. Although the feedforward force can also influence the motion of the EE, we only rely on it to handle misalignment or sticking situations.

## Step 1: Impedance Controller
First, you need to build an impedance controller for your robot. If you are using a Franka Robotics robot, you can follow this [demo](https://frankarobotics.github.io/libfranka/0.15.0/cartesian_impedance_control_8cpp-example.html). Once this step is completed, your robot should behave like the one shown in the following video.


https://github.com/user-attachments/assets/2835e665-05a5-4413-93c1-9be582d8a343



## Step 2: Feedforward-based Impedance Controller
On top of the impedance controller, you need to further add a feedforward force term to the controller. You can start by designing the feedforward force using a simple pattern. For example, you can set fz as a sinusoidal signal and set fx, fy, mx, my, and mz to zero. Then, your robot should behave as shown in the following video. 




https://github.com/user-attachments/assets/cad28686-fe71-4c0c-8537-d256b0264181

## Step 3: Teacher Policy
Follow Algorithm 1 in our paper to design the teacher policy and collect training data. We provide one demonstration ([robot_action.pkl](https://github.com/yorklyb/SI-Diff/blob/master/robot_action_train_demo.pkl) & [robot_state.pkl](https://github.com/yorklyb/SI-Diff/blob/master/robot_state_train_demo.pkl)) in this repository to show what the training data look like. 
<br>
<br>
Our diffusion policy learns to predict robot action (**output**) from robot states (**input**). The action is the 6 DoF feedforward force (fx, fy, fz, mx, my, and mz). The robot state is 37-dimensional: the first value is the mode prompt, and the following 36 dimensions are identical to the observations in TacDiffusion. You can refer to the discussion [here](https://github.com/popnut123/TacDiffusion/issues/1) for details regarding the 36 dimension values.

## Step 4: Diffusion Policy
Our diffusion policy is built upon [Imitating-Human-Behaviour-w-Diffusion](https://github.com/microsoft/Imitating-Human-Behaviour-w-Diffusion) and [TacDiffusion](https://github.com/popnut123/TacDiffusion). We recommend first becoming familiar with these two works, then following the instructions in our paper to add the mode embedding layers.
<br>
<img width="1280" height="675" alt="network" src="https://github.com/user-attachments/assets/41b41210-d46c-482a-a2fb-d030394abb72" />
<br>


## Acknowledgments
Parts of this project page were adopted from the [Nerfies](https://nerfies.github.io/) page.

