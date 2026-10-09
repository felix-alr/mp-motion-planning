# Research Summary On Safe Trajectory Generation Using Motion Primitives To Imitate Pre-Recorded Trajectories
This research task serves the purpose of yielding a list of relevant approaches for generating safe trajectories that imitate a pre-recorded trajectory. For comparing different approaches, the following metrics are used:

## Requirements / Metrics
Each approach is graded based on how well
1. it can imitate a pre-recorded trajectory, shaping the spatial and temporal progression
2. variable start and end-points can be set
3. the time of the movement can be altered
4. timed via points can be used for precise control of temporal progression
5. spatial safety constraints can be introduced
 - e.g. minimum distance from object (e.g. point, trajectory)
 - e.g. forbidden coordinate values to ensure that multiple arms do not collide
6. stability constraints can be introduced
 - e.g. maximum allowable deviation from optimal (scaled) pre-recorded trajectory progression to ensure the generated trajectory is precise where it needs to be
7. online changes to constraints may be handled

## Comparing Approaches

Scale: `++` native/strong · `+` supported with caveats · `~` only via a specific published extension · `−` weak or absent

Rated on **base formulations**; published extensions are noted separately in the limitations above. ProDMP ratings draw partly on Li et al. (2023a) rather than the survey, which covers ProDMP in a single paragraph. Metric 6 is not addressed directly by the survey and is inferred throughout.

| # | Metric | DMP | ProMP | ProDMP | KMP | CNMP | FMP |
|---|---|---|---|---|---|---|---|
| 1 | Imitate recorded trajectory (spatial + temporal shaping) | ++ | ++ | ++ | ++ | + | + (rhythmic only) |
| 2 | Variable start/end-points | + | ~ | + | ++ | ~ | − |
| 3 | Variable movement duration | ++ | ++ | ++ | + | ~ | − |
| 4 | Timed via-points | ~ | ++ | ++ | ++ | ~ | − |
| 5 | Spatial safety constraints | ~ | ~ | ~ | + | ~ | − |
| 6 | Stability / deviation constraints | ~ | + | + | + | − | − |
| 7 | Online constraint changes | + | ~ | + | ~ | ~ | − |

### Notes on individual ratings

- **Metric 1** — CNMP rated `+` because reproduction quality depends on training-data volume and only isotropic per-step variance is modelled. FMP is capable but confined to rhythmic motion.
- **Metric 2** — DMP and ProDMP rated `+`: the goal is an explicit parameter, but spatial scaling is frame-dependent and shape generalization to rotated or distant targets requires a reformulation (Hoffmann et al. 2009; Koutras and Doulgeri 2020c; Sidiropoulos and Doulgeri 2024). KMP rated `++` as extrapolation is its stated advantage over ProMPs. ProMP rated `~`: endpoint conditioning works, but the distribution is a global frame of reference and does not extrapolate outside the demonstrated region.
- **Metric 3** — KMP rated `+` rather than `++` because it is driven by a general input variable rather than a phase variable; temporal scaling must be implemented by normalizing the input axis.
- **Metric 4** — DMP rated `~` via dynamic via-points (Sidiropoulos and Doulgeri 2024). ProMP hits via-points with high probability rather than deterministically; KMP's desired-point handling is explicit but the choice of points is empirical.
- **Metric 5** — KMP rated `+` because extended KMPs (Huang 2021) place hard linear and nonlinear constraints and link-level obstacle avoidance *inside* the primitive. DMP's `~` covers a large extension family: barrier Lyapunov functions (Lu et al. 2021b), zeroing barrier functions (Shaw et al. 2023), kinematic constraints with an MPC extension (Sidiropoulos et al. 2023), and CLF-CBF control (Theofanidis et al. 2024).
- **Metric 6** — Own metric; not addressed by the survey. ProMP, ProDMP and KMP rated `+` because the learned covariance constitutes a deviation bound: Davoodi et al. (2021b) construct CBFs from ProMP distribution moments guaranteeing the state never leaves a set distance from the mean. DMP has no variance notion in its base form; probabilistic DMP variants (Calinon et al. 2012; Meier and Schaal 2016) would be the route. CNMP rated `−` as its isotropic per-step variance models no correlation across time steps or dimensions.
- **Metric 7** — DMP and ProDMP rated `+`: a DMP is an ODE integrated online, so coupling terms can be injected at runtime; ProDMP additionally replans from the current position and velocity with guaranteed initial conditions. The remaining frameworks are rated `~` — possible via extensions, not a property of the base formulation.

## Approaches - Summary
- Dynamic Movement Primitives
  - They use a stable (critically damped) spring damper system with a target and a forcing term $f$ that shapes the trajectory and consists of learned weights of Gaussian basis functions.
  - Potential:
    - Robot arm states can be inferred from the DMP state (x,u)
    - Highly extensible by means of the forcing term $f$
    - Learns from a single demonstration
  - Limitations:
    - Generalization to new targets (solvable using different formulation)
    - Reliance on Gaussian basis functions
    - Assumes one demonstration is perfect
    - Fails when goal $\approx$ start

- Probabilistic Movement Primitives 
  - They encode a distribution over trajectories using the linear basis-function model $y_{i,j} = \Phi(i)w_j+\xi$, where $\Phi(i)$ is a vector stacking the basis functions and their temporal derivative and the weight matrix $w_j$ is modelled as a Gaussian $\mathcal{N}(\mu_{w_j}, \Sigma_{w_j})$ fitted over many demonstrations, e.g. using maximum likelihood estimation (MLE).
  - Potential:
    - Time abstracted into phase variable -> temporal scaling
    - Primitives can be blended (better generalization as opposed to DMP)
    - Covariance across DoFs captures joint coupling automatically
    - Smooth closed form Gaussian conditioning for timed via points
  - Limitations:
    - Requires temporally aligned demonstrations of similar length (preprocessing)
    - Zero final velocity is assumed
    - Poor in high-dimensional input spaces
    - Cannot extrapolate outside the demonstrated region of the learned distribution
    - Via-points are hit with high probability, not deterministically
  
- Probabilistic Dynamic Movement Primitives
  - A unified perspective on DMPs and ProMPs: the DMP solution is rewritten as a linear basis-function model whose basis functions depend on the DMP parameters (forcing-term weights and goal attractor). This removes the numerical integration from acceleration to position, which is required with plain DMPs, so trajectory statistics can be formed as in ProMPs while the underlying dynamical-system properties are retained.
  - Potential:
    - Combines ProMP-style trajectory distributions and conditioning with DMP dynamics
    - No numerical integration needed -> trajectory statistics are directly accessible
    - Replanning starts from the current position and velocity, enabling smooth online reconditioning (relevant for requirement 7)
    - Guaranteed initial conditions when generating action sequences
  - Limitations:
    - Inherits DMP-style spatial scaling, so generalization to new targets is not automatically improved
    - Basis parameter tuning still required
    - Thin coverage in the survey (single paragraph, no dedicated section) and fewer follow-up works than DMP/ProMP

  - Kernelized Movement Primitives
    - Non-parametric model of the trajectory. Firstly, a reference database $\{s_n, \hat{\mu}_n, \hat{\Sigma}_n\}$ is extracted from the demonstrations. Then a kernel is used to predict the output for a given query input.
    - Potential:
      - The kernel approach is better suited for high-dimensional inputs
      - Input can be chosen freely and does not necessarily have to be time
      - Extensions available to handle constraints within the primitive
      - Keeps probabilistic approach of ProMP but is better at extrapolating
    - Limitations:
      - Computational complexity grows quickly with training-set size (costly kernel calculations)
      - Kernel choice is crucial and non-trivial
      - Choice of via points is empirical
      - Multiple demonstrations required
    
  - Conditional Neural Movement Primitives
    - An encoder network E maps observation pairs to latent representations which are averaged into a context vector and fed into a query network Q along with a query. The query network outputs distribution parameters $(\mu_q, \sigma_q)$ for a queried input, determining the trajectory.
    - Potential:
      - Context dependent adaptation of trajectories
    - Limitations:
      - Requires large amounts of data
      - Sensitive to choice of hyperparameters
      - One model per skill
      - Via-points may be missed
      - No correlation across time or dimensions

  - Fourier Movement Primitives
    - They follow the ProMP learning process just in the frequency domain. Movements are expressed through a sum of sinusoidal basis functions obtained by means of the Fourier transform. Well suited for tasks such as wiping, polishing, stirring, or shaking.
    - Potential:
      - Basis functions are automatically extracted from the frequency domain data.
      - No manual alignment of the demonstrations required (less preprocessing)
      - Captures magnitude and phase statistics (robust to noise)
    - Limitations:
      - Restricted to rhythmic tasks
      - Signal length T must be chosen manually and must cover at least one period
      - Multiple low-noise demonstrations required
      - No direct extensions or improvements to the original framework
