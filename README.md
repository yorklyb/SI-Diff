# SI-Diff: A Framework for Learning Search and High-Precision Insertion with a Force-Domain Diffusion Policy (RA-L'26 & ICRA'27)
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
<br>
<br>
Once the teacher policy is ready, the robot can start searching. In the early stages, we manually created misalignments to collect data for the teacher policy. Later, we developed an automated data collection pipeline. It mirrors the evaluation process of the teacher policy, but only records successful demonstrations that meet our efficiency criteria (completed within 2 seconds). We kept running this until a sufficient number of expert demonstrations are collected.


https://github.com/user-attachments/assets/f925b36f-ab02-45b1-942f-90a4ffa58dff



## Step 4: Diffusion Policy
Our diffusion policy is built upon [Imitating-Human-Behaviour-w-Diffusion](https://github.com/microsoft/Imitating-Human-Behaviour-w-Diffusion) and [TacDiffusion](https://github.com/popnut123/TacDiffusion). We recommend first becoming familiar with these two works, then following the instructions in our paper to add the mode embedding layers.
<br>
<img width="1280" height="675" alt="network" src="https://github.com/user-attachments/assets/41b41210-d46c-482a-a2fb-d030394abb72" />
<br>
## Step 5: Model Training
Since the model needs to learn two modes simultaneously, and the data distribution between the two modes is imbalanced, we recommend using the BBS technique. The following code briefly illustrates one training iteration process.
```python
for ep in range(n_epoch):
    dataload_train_0.sampler.set_epoch(ep)
    dataload_train_1.sampler.set_epoch(ep)

    model.train()
    optim.param_groups[0]["lr"] = lrate * ((np.cos((ep / n_epoch) * np.pi) + 1) / 2)

    pbar = zip(dataload_train_0, dataload_train_1)
    if rank == 0:
        pbar = tqdm(pbar, total=min(len(dataload_train_0), len(dataload_train_1)), desc=f"Epoch {ep}")

    for (x0, y0), (x1, y1) in pbar:
        # 1. Move tensors to the configured device asynchronously
        x0 = x0.to(device, non_blocking=True).float()
        y0 = y0.to(device, non_blocking=True).float()
        x1 = x1.to(device, non_blocking=True).float()
        y1 = y1.to(device, non_blocking=True).float()

        # 2. Extract the mode prompt from the first dimension (index 0)
        # Input shape: [B, 37] -> mode shape: [B], feature shape: [B, 36]
        mode0 = x0[:, 0].long()       # Cast to long for the embedding layer
        x0_feature = x0[:, 1:]        # Slice the remaining 36 dimensions for observations

        mode1 = x1[:, 0].long()
        x1_feature = x1[:, 1:]

        # 3. Concatenate the dual-source data into a single balanced batch
        x_batch = torch.cat([x0_feature, x1_feature], dim=0)  # Pure 36-dim observations
        y_batch = torch.cat([y0, y1], dim=0)
        mode_batch = torch.cat([mode0, mode1], dim=0)          # Combined mode prompts

        # 4. Forward pass and loss computation
        loss = model.module.loss_on_batch(x_batch, y_batch, mode_batch)
        
        # 5. Backward pass and optimization step
        optim.zero_grad()
        loss.backward()
        optim.step()

        if rank == 0:
            pbar.set_description(f"train loss: {loss.item():.4f}")
            writer.add_scalar('training_loss', loss.item(), global_step)
            global_step += 1
```

## Acknowledgments
Parts of this project page were adopted from the [Nerfies](https://nerfies.github.io/) page. 
We would like to thank the authors of [Imitating-Human-Behaviour-w-Diffusion](https://github.com/microsoft/Imitating-Human-Behaviour-w-Diffusion) and [TacDiffusion](https://github.com/popnut123/TacDiffusion) for their open-source contributions.

