# DMP
## Inputs
### For Learning
- $y, \dot{y}, \ddot{y}$
### At Runtime
- $y_0, z_0$: initial system state
- g: target (attractive) point
- $\tau$: time scaling constant
- $f(x) = \frac{\Sigma w_i \Psi_i(x)}{\Sigma \Psi_i(x)}\cdot x$ learned forcing function of phase variable $x$ consisting of:
  - $\Psi_i(x) = \exp(-h_i(x-c_i)^2)$: Gaussian nonlinear radial basis functions (RBF) with height $h_i$ and center $c_i$
  - $w_i$: weights learned from demonstration 
## Outputs
- $y$: current system state
- $\dot{y} = z/\tau$: current system velocity
- $\ddot{y} = \dot{z}/\tau$: current system acceleration
## Parameters
- $\alpha_z$, $\beta_z$: parameters determining convergence of the DMP ($\alpha_z = 4\beta_z$ $\rightarrow$ critically damped)
- $\alpha_x$: phase decay rate (when standard formulation is used)
- Number of basis functions determining trajectory resolution
## Constraint Handling
## Applications

## Sources
- 

# ProMP
## Inputs
### For Learning
- $\tau = \{y_t\}_{t=1}^T$: several of those demos required, optionally including corresponding quaternion demos $y_t \in S^3$ for full pose. When using Euler angles they must stay clear of gimbal lock configurations
### At runtime
- $z(t)$ or $\dot(z)(t)$: phase or phase rate
- $y^*_t$: via-/start-/end-point at time t
- $\Sigma^*_y$: precision for hitting y^*_t
- $\alpha_t[i]\in [0,1]$ (optional): activation/blending weights when blending primitives
- $\hat{s}$ (optional): task parameter, e.g. object position

## Outputs
- $y_d(t)$: desired state (mean trajectory point)
- $\dot{y}_d(t)$: desired velocity
- $\Sigma_t$: covariance at current time step

## Parameters
### Hyperparameters
- $N_\phi$: number of basis functions
- basis type: Gaussian for strokes (point to point), Von Mises for rhythmic movements
- $c_i, h$: centers and heights for each basis function (usually evenly spaced with same width)
- basis normalization: $φ_i(z) = b_i(z) / \Sigma_j b_j(z)$ for enhanced coverage and interpretability
- $\lambda$: regularization to penalize overly high weights (should be chosen low relative to data scale)
- $\Sigma_y$: observation noise assumed on each trajectory point, adding to the covariance at the output
- $p$ (orientation only): origin point of the tangent space in which the orientation weights live (shared by all demos)
### Learned Parameters
- $w_n$: per-demo weight (demo compressed to weight vector)
- $\theta = \{\mu_w, \Sigma_w\}$: mean and covariance over weights, representing the actual skill ($\Sigma_w$ also contains information about the coupling between DoFs)

## Constraint Handling
## Applications

# ProDMP
## Inputs
## Outputs
## Parameters
## Constraint Handling
## Applications

# KMP
## Inputs
## Outputs
## Parameters
## Constraint Handling
## Applications

# Sources
- https://arxiv.org/pdf/2102.03861
- https://proceedings.mlr.press/v164/rozo22a/rozo22a.pdf
- https://arxiv.org/pdf/2210.01531