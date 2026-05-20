## From the Author
Due to IP policies, we do not release a click-and-run version of SI-Diff. We will provide more supplementary details to the paper to help you reproduce the work. <br>
We first provide a straightforward introduction to the fundamentals of robot control to help readers avoid confusion. The second-order dynamic
model of an n-Degree-of-Freedom torque-controlled robot is as follows: <br>
<img width="383" height="45" alt="image" src="https://github.com/user-attachments/assets/185935f6-f8be-4706-8fdd-37e8ada05b4a" /> <br>
Among these terms, we control the robot by changing $\boldsymbol{\tau}_m$, the joint torque.
## Step 1: Impedance Controller
First, you need to build an impedance controller for your robot. If you are using a Franka Robotics robot, you can follow this [demo](https://frankarobotics.github.io/libfranka/0.15.0/cartesian_impedance_control_8cpp-example.html). Once this step is completed, your robot should behave like the one shown in the following video.


https://github.com/user-attachments/assets/2835e665-05a5-4413-93c1-9be582d8a343



## Step 2: Feedforward-based Impedance Controller
On top of the impedance controller, you need to further add a feedforward force term to the controller.





## Acknowledgments
Parts of this project page were adopted from the [Nerfies](https://nerfies.github.io/) page.

