# Constraint Approaches

## Strain Limiting and Position Based Dynamics

increasing the stiffness can cause problems.

-  Explicit integrators will be unstable
   -  Solution: smaller time steps and more computational time.
-  The linear systems involved in Implicit integrators will be ill-conditioned.
   -  Solution: more iterations and computational time.

optimization with constrain

Gauss-Seidel: more like SGD

- The order matters. The order can cause bias and affect convergence behavior.

Jacobi Approach

average update

lower convergence rate

### PBD

dynamic-projection(constrain)-update

not physics based

stiffness:

- number of iteration
- mesh resolution
- update velocity after projection
- applied to other constrains

- Pros
- Parallelable on GPUs (PhysX) 
- Easy to implement 
- Fast in low resolutions 
- Generic, can handle other coupling and constraints, including fluids

Cons
- Not physically correct 
- Low performance in high resolutions 
  - Hierarchical approaches (can cause oscillation and other issues…)
  - Acceleration approaches, like Chebyshev

Reading: Hierarchical Position Based Dynamics.

Strain Limiting

set strain in projection step

Triangle Area Limit

Strain limiting is widely used in physics based simulation, typically for avoiding instability and artifacts due to large deformation.

## Projective Dynamics

projective dynamics uses projection to define a quadratic energy.

consider new position as independent?

We can use a direct solver with only one factorization

$(\frac{1}{\Delta t^2}M+H)\Delta x=-\frac{1}{\Delta t^2}M(x^k-x^0-\Delta tv^0)+f(x^k)$

(Newton's)

PBD: small memory visit

pros:

- By building constraints into energy, the simulation now has a theoretical solution with physical meaning.
- Fast on CPUs with a direct solver. No more factorization!
- Fast convergence in the first few iterations.

cons:

- Slow on GPUs. (GPUs don’t support direct solver wells.)
- Slow convergence over time, as it fails to consider Hessian caused by projection.
- Still suffering from high stiffness
- Cannot easily handle constraint changes.
- Contacts
- Remeshing due to fracture, etc

reading: Projective Dynamics

## Constrained Dynamics

for strong stiff

$E=\frac{1}{2}\Phi^T(x)C^{-1}\Phi(x)$

$f(x)=-\nabla E=-J^TC^{-1}\Phi=J^T\lambda$

$\lambda$: Lagrangian multipliers

$Mv'-\Delta J^T\lambda'=Mv$

$C\lambda'\approx -\Phi-\Delta tJv'$

- direct solver: promal dual
- solve $\lambda$, then v
- Infinite stiffness: $C\to 0$

Articulated Rigid Bodies!

Reading: Stable Constrained Dynamics.
