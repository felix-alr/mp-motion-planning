# DMP
## Inputs
### For Learning
- $y, \dot{y}, \ddot{y}$ of a single demonstration (velocity and acceleration usually obtained by numerical differentiation of $y$) [Ijspeert 2013]
- $y_0 = y(0)$, $g = y(T)$, $\tau$ = demo duration: needed to compute the target forcing term $f_{target} = \tau^2\ddot{y} - \alpha_z(\beta_z(g-y) - \tau\dot{y})$, which the weights are regressed on (locally weighted regression or batch least squares) [Ijspeert 2013; Saveriano 2023]
### At Runtime
- $y_0, z_0$: initial system state ($z = \tau\dot{y}$ is the scaled velocity)
- g: target (attractive) point
- $\tau$: time scaling constant
- $f(x) = \frac{\Sigma w_i \Psi_i(x)}{\Sigma \Psi_i(x)}\cdot x\,(g - y_0)$ learned forcing function of phase variable $x$ consisting of [Ijspeert 2013]:
  - $\Psi_i(x) = \exp(-h_i(x-c_i)^2)$: Gaussian nonlinear radial basis functions (RBF) with width $h_i$ (inverse width; the height is fixed at 1) and center $c_i$
  - $w_i$: weights learned from demonstration 
  - $(g - y_0)$: spatial scaling term, which causes the "goal $\approx$ start" failure and the frame dependence noted in the summary. Pastor 2009 replaces it with a formulation that avoids this; Koutras 2020 gives a frame-independent version
- Dynamics: $\tau\dot{z} = \alpha_z(\beta_z(g-y) - z) + f(x)$, $\tau\dot{y} = z$, canonical system $\tau\dot{x} = -\alpha_x x$ with $x_0 = 1$ [Ijspeert 2013]
- Multi-DoF: one transformation system per DoF, all driven by one shared canonical system (keeps the DoFs synchronized) [Ijspeert 2013]
## Outputs
- $y$: current system state
- $\dot{y} = z/\tau$: current system velocity
- $\ddot{y} = \dot{z}/\tau$: current system acceleration
## Parameters
- $\alpha_z$, $\beta_z$: parameters determining convergence of the DMP ($\alpha_z = 4\beta_z$ $\rightarrow$ critically damped; common choice $\alpha_z = 25$) [Ijspeert 2013]
- $\alpha_x$: phase decay rate (when standard formulation is used)
- Number of basis functions determining trajectory resolution
- $c_i$, $h_i$: centers are usually spaced so that they are equidistant in *time* (i.e. exponentially spaced in $x$), with widths set heuristically from the distance to the neighbouring center [Saveriano 2023]
- Orientation: a DMP on Euler angles is not singularity-free. The quaternion / rotation-matrix formulation is in Ude 2014, and Koutras 2019 corrects oscillations in it. Relevant because the existing controller takes $(\phi_x, \phi_y, \phi_z)$, so a conversion step is needed
## Constraint Handling
## Applications

## Sources
- [Ijspeert 2013] Ijspeert, Nakanishi, Hoffmann, Pastor, Schaal: *Dynamical Movement Primitives: Learning Attractor Models for Motor Behaviors*, Neural Computation 25(2), 2013. DOI `10.1162/NECO_a_00393` (not in Zotero yet). Canonical reference for the formulation above
- [Saveriano 2023] Saveriano, Abu-Dakka, Kramberger, Peternel: *Dynamic Movement Primitives in Robotics: A Tutorial Survey*, IJRR 42(13), 2023. Zotero `Saveriano2023_9SZ3ZNA3`, DOI `10.1177/02783649231201196`, arXiv `2102.03861`. Implementation details, variants, released code
- [Pastor 2009] Pastor, Hoffmann, Asfour, Schaal: *Learning and generalization of motor skills by learning from demonstration*, ICRA 2009. DOI `10.1109/ROBOT.2009.5152385` (not in Zotero yet). Modified transformation system fixing the $g \approx y_0$ problem
- [Hoffmann 2009] Hoffmann, Pastor, Park, Schaal: *Biologically-inspired dynamical systems for movement generation: automatic real-time goal adaptation and obstacle avoidance*, ICRA 2009. DOI `10.1109/ROBOT.2009.5152423` (not in Zotero yet). Online goal change plus obstacle coupling term, for the Constraint Handling section
- [Ude 2014] Ude, Nemec, Petrič, Morimoto: *Orientation in Cartesian space dynamic movement primitives*, ICRA 2014. DOI `10.1109/ICRA.2014.6907291` (not in Zotero yet)
- [Koutras 2019] Koutras, Doulgeri: *A correct formulation for the Orientation Dynamic Movement Primitives for robot control in the Cartesian space*, CoRL 2019, PMLR 100:293–302. DOI `10.5281/zenodo.3537791` (Zenodo) (not in Zotero yet)
- [Koutras 2020] Koutras, Doulgeri: *A novel DMP formulation for global and frame independent spatial scaling in the task space*, RO-MAN 2020. Zenodo record 3955889; the IEEE DOI is still unverified, so check IEEE Xplore (not in Zotero yet)

# ProMP
## Inputs
### For Learning
- $\tau = \{y_t\}_{t=1}^T$: several of those demos required, optionally including corresponding quaternion demos $y_t \in S^3$ for full pose [Rozo 2022]. When using Euler angles they must stay clear of gimbal lock configurations (note: $\tau$ here is a demo, not the DMP time constant)
- Each demo is mapped onto a common phase $z \in [0,1]$ (time normalization or temporal alignment, e.g. DTW) [Paraschos 2018]
### At runtime
- $z(t)$ or $\dot{z}(t)$: phase or phase rate (typically $z$ linear in $t$; changing $\dot{z}$ changes the execution speed) [Paraschos 2013]
- $y^*_t$: via-/start-/end-point at time t [Paraschos 2013]
- $\Sigma^*_y$: desired *accuracy* for hitting $y^*_t$. It is a covariance, so smaller means tighter (not a precision matrix); $\Sigma^*_y \to 0$ approaches a hard via-point [Paraschos 2013]
- $\alpha_t[i]\in [0,1]$ (optional): activation/blending weights when blending primitives (co-activation as a product of distributions) [Paraschos 2013]
- $\hat{s}$ (optional): task parameter, e.g. object position [Paraschos 2018]

## Outputs
- $y_d(t)$: desired state (mean trajectory point)
- $\dot{y}_d(t)$: desired velocity
- $\Sigma_t$: covariance at current time step
- (optional) stochastic time-varying feedback controller $u_t = K_t y_t + k_t + \epsilon_u$ that reproduces the distribution [Paraschos 2013; Paraschos 2018]. It needs a (locally linearized) system model. With the existing Cartesian controller, the mean and covariance are probably enough

## Parameters
### Hyperparameters
- $N_\phi$: number of basis functions
- basis type: Gaussian for strokes (point to point), Von Mises for rhythmic movements
- $c_i, h$: centers and widths for each basis function (usually evenly spaced with same width) [Paraschos 2018]
- basis normalization: $φ_i(z) = b_i(z) / \Sigma_j b_j(z)$ for enhanced coverage and interpretability
- $\lambda$: regularization to penalize overly high weights (should be chosen low relative to data scale). Per-demo ridge regression: $w_n = (\Phi^T\Phi + \lambda I)^{-1}\Phi^T Y_n$ [Paraschos 2018]
- $\Sigma_y$: observation noise assumed on each trajectory point, adding to the covariance at the output
- $p$ (orientation only): origin point of the tangent space in which the orientation weights live (shared by all demos) [Rozo 2022]
### Learned Parameters
- $w_n$: per-demo weight (demo compressed to weight vector)
- $\theta = \{\mu_w, \Sigma_w\}$: mean and covariance over weights, representing the actual skill ($\Sigma_w$ also contains information about the coupling between DoFs) [Paraschos 2013]
  - Plain MLE (sample mean and covariance of the $w_n$) gives a rank-deficient $\Sigma_w$ when there are fewer demos than weight dimensions ($N_\phi \cdot$ DoF). The fix is a prior over $\theta$ fitted by MAP/EM [Gomez-Gonzalez 2020]

## Constraint Handling
## Applications

## Sources
- [Paraschos 2013] Paraschos, Daniel, Peters, Neumann: *Probabilistic Movement Primitives*, NeurIPS 26, 2013. https://papers.nips.cc/paper/5177-probabilistic-movement-primitives (no DOI) (not in Zotero yet). Original formulation: conditioning, blending, co-activation, controller derivation
- [Paraschos 2018] Paraschos, Daniel, Peters, Neumann: *Using probabilistic movement primitives in robotics*, Autonomous Robots 42(3), 2018. DOI `10.1007/s10514-017-9648-7` (not in Zotero yet). Extended journal version and the main reference for learning details (ridge regression, basis normalization, phase, task parameters)
- [Rozo 2022] Rozo, Dave: *Orientation Probabilistic Movement Primitives on Riemannian Manifolds*, CoRL 2021, PMLR 164:373–383. arXiv `2110.15036` (not in Zotero yet). Quaternion ProMPs and the tangent-space origin $p$
- [Gomez-Gonzalez 2020] Gomez-Gonzalez, Neumann, Schölkopf, Peters: *Adaptation and Robust Learning of Probabilistic Movement Primitives*, IEEE T-RO 36(2), 2020. DOI `10.1109/TRO.2019.2937010`, arXiv `1808.10648` (not in Zotero yet). Prior and EM learning from few demos
- See also the general MP survey: Zotero `Gutierrez2026_SUMP2MUE`, arXiv `2601.02379`

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
- https://arxiv.org/pdf/2102.03861 = [Saveriano 2023], Zotero `Saveriano2023_9SZ3ZNA3`
- https://proceedings.mlr.press/v164/rozo22a/rozo22a.pdf = [Rozo 2022] (not in Zotero yet)
- https://arxiv.org/pdf/2210.01531 = Li et al., ProDMP, Zotero `Li2022_4C3BJJMS`. Published version: IEEE RA-L 8(4), 2023, DOI `10.1109/LRA.2023.3248443` (the Zotero item is saved as a webpage pointing to arXiv v1, so it is worth updating)