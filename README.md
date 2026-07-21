# iRT Coupled PINN

This is a physics-informed neural network that models tumor and immune response to radiotherapy combined with immunotherapy. The network solves a coupled system of five ODEs describing irradiated tumor, non-irradiated tumors damaged tumor, and circulating lymphocyte populations over a 1window of 100 days.

## Initial Conditions

The PINN declares the initial conditions for the 5 variables below. These values were developed with the idea that one single dose was administered previously, but I am working to develop a fractionation schedule, which would change the initial conditions of the damaged tumors, irradiated tumors, and the lymphocytes. For the integration of the primary immune response present in the secondary immune response equation, I created a new differential equation in order to solve for the desired value.

<img width="411" height="135" alt="Screenshot 2026-07-15 at 12 05 14 PM" src="https://github.com/user-attachments/assets/064a29dc-f61f-42c5-a143-0f1847e808ee" />

## Parameters

These parameters are used exclusively in the loss function of the PINN. Separately, the constant, L_scale = 1e9, is used to scale the lymphocyte count and the other lymphocyte parameters because the scaling keeps the network's output weights in a trainable range and prevents the L residual from dominating the total loss. Rate constants, such as lamda and delta_L, are left unscaled, since they multiply L linearly and scale correctly on their own.

<img width="504" height="346" alt="image" src="https://github.com/user-attachments/assets/9ee15fe5-78bc-44b5-8bf1-11d33bd99b8c" />

## Loss Function

Each residual is the difference between the left-hand and right-hand sides of the corresponding tumor-immune dynamics equations, evaluated at the collocation points, and its loss is the mean squared residual. The total loss function comprises both the physics loss for each equation and the initial loss for each initial condition.

<img width="737" height="498" alt="image" src="https://github.com/user-attachments/assets/92b205f6-792d-4fcd-b031-78da87cb96f3" />


## Network Architecture

The physics informed neural network is a fully connected network mapping time to all 5 states at once. It represents a continuous function of t and then trains until the function satisfies the given ODEs. This comprises seven fully connected layers, each of which passes its result through the tanh activation with the exception of the last layer. Tanh was chosen because it is smooth and can be differentiated multiple times. The PINN needs to calculate derivatives of the outputs and then differentiate them again during training. ReLU would not work well because its second derivative is zero, making it impossible for the network to learn the physics in the equations. Within the forward function, time is divided by 100 in order to keep the input value within the range [0,1], which is where the tanh will still respond best. The singular network serves all 5 states rather than having 5 different networks because the states are coupled, meaning that the hidden layers learn representations of the system while the output layer outputs five projections of that system. 

For the collocation of points, I used 1000 values of t uniformly spaced on [0,100], with requires_grad = True, so that the d/dt can be taken by autograd. For the training loop, there are 20,000 epochs, or training rounds, with a learning rate of $10^{-3}$. Each epoch performs a forward pass, then computes the five derivatives, evaluates the initial condition loss in a separate forward pass, and takes a single gradient step. During these epochs, the total loss is calculated, and over time, that loss gets smaller and smaller, demonstrating that the network is actually learning based on the governing equations.

## Fractionation Scheme

Now, the PINN is able to predict the fractionation and scheduling of the radioation as well. The spikes throughout the graph show the instantaneous jumps caused by the administration of the radiation dose. The graph also shows the two-week follow up after the treatment and how the tumor cells and lymphocyte populations change during those weeks following. This was achieved by creating a scheduling and radiation dosage function, whcih are then utilized in the for loop. The scheduling function takes in the number of doses, the start day, and whether or not the dose is administered on weekends. In order to achieve this shape in the graph, the total timeline is cut into segments, where the PINN is now training for each segment, which can be seen now by the new for loop that was implemented. 
<img width="989" height="490" alt="image" src="https://github.com/user-attachments/assets/9287e064-47f6-4dca-bcec-893f87b7b557" />


### Current Files

The files in the repository show my gradual journey to developing the 5 ODE coupled PINN. I started off with just the one Damaged Tumors equation and then slowly added the rest of the tumor-immune dynamic equations in order to develop the full PINN. Next, I am working on how to incorporate a fractionstion schedule into my PINN.
