# iRT Coupled PINN

## Initial Conditions

The PINN declares the initial conditions for the 5 vairables below. These values were developed with the idea that one single dose was administered previously, but i am working to develop a fractionation schedule, which would change the initial conditions of the damaged tumors, irradiated tumors, and the lymphocytes. For the integration of the primary immune response present in the secondary immune response equation, I created a new differential equation in order to solve for the desired value.

<img width="411" height="135" alt="Screenshot 2026-07-15 at 12 05 14 PM" src="https://github.com/user-attachments/assets/064a29dc-f61f-42c5-a143-0f1847e808ee" />

## Parameters

These parameters are used exclusively in the loss function of the PINN. Separately, the constant, L_scale = 1e9, is used to scale the lymphocyte count and the other lymphocyte parameters because the scaling keeps the network's output weights in a trainable range and prevents the L residualfrom dominating the total loss. Rate constants. such as lamda and delta_L are left unscaled, since they multiply L linearly and scale correctly on their own.

<img width="504" height="346" alt="image" src="https://github.com/user-attachments/assets/9ee15fe5-78bc-44b5-8bf1-11d33bd99b8c" />

## Loss Functions

Each residual is the difference between the left-hand and right-hand sides of the corresponding tumor-immune dynamics equations, evaluated at the collocation points, and its loss is the mean squared residual. The total loss function comprises of both, the physics loss for each equation, along with the initial loss for each initial condition.

<img width="737" height="498" alt="image" src="https://github.com/user-attachments/assets/92b205f6-792d-4fcd-b031-78da87cb96f3" />


## Network Architecture

The physics informed neural network is a fully connected network mapping time to all 5 states at once. It represents a continuous function of t and then trains until the functions satsifies the given ODEs. THis comprises of seven fully connected layers, which all pass its result through the tanh activation with the exception of the last layer. The tanh was chosen as it is a smooth curve, making it simpler to compute the derivatives. Within the forward function, time is divided by 100 in order to keep the input value within the range [0,1], which is where the tanh will still respond. The singular network contains of states that depend on each other, meaning that the variable values of Zp and L change as the tumor cells change.

For the collocation of points, I used 1000 values of t uniformly spcaed on [0,100], with requires_grad = true, so that the d/dt can be taken by autograd.For the training loop, there are 20,000 epochs, or training rounds, with a learning rate of $10^(-3)$. Each epoch performs a forward pass, then computes the five derivates, and takes a single gradient step. During these epochs, the total loss is calculated, and overtime, that loss gets smaller and smaller, demonstrating that the network is actually learning based on the governing equations

