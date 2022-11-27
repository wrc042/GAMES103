# Lecture XI - Eulerian Fluids

## A Grid Representation and Finite Differencing

Finite Differencing on Grid

Discretized Laplacian

$\frac{\partial f_{i+0.5,j}}{\partial x}\approx\frac{f_{i+1,j}-f_{i,j}}{h}$

A Dirichlet boundary: $f_{i-1,j}=C$

A Neumann boundary: $f_{i-1,j}=f_{i,j}$

Laplace’s Equation: Jacobi Method

$f_{i+1,j}+f_{i-1,j}+f_{i,j+1}+f_{i,j-1}-4f_{i,j}=0$

At least one condition should be Dirichlet: infinite solution.

Laplacian Smoothing

The process of applying Laplacian smoothing is called diffusion.

Staggered Grid

We define some physical quantities on faces, specifically velocities.

$u_{i+1,j}-u_{i,j}+v_{i+1,j}-v_{i,j}=0\implies\nabla\cdot u_{i,j}=0$

Bilinear Interpolation

## Navier-Stokes Equations

Incompressibility

$\nabla\cdot u=0$

$\frac{\partial u}{\partial t}=g-(u\cdot\nabla)u+\mu\Delta u-\nabla p$

solving a long partial differential equation (PDE) in steps

### External Acceleration

$\frac{\partial u}{\partial t}=g\implies v'=v+\Delta tg$

### Advection

$\frac{\partial u}{\partial t}=-(u\cdot\nabla)u$

$(u\cdot\nabla)u=u\frac{\partial u}{\partial x}+v\frac{\partial v}{\partial y}$

instability!

Solution: Semi-Lagrangian Method

$u'=u(x_0-\Delta tu(x_0))$

We could also subdivided the time step for better tracing.

### Diffusion

$\frac{\partial u}{\partial t}=\mu\Delta u$

laplacian

sub-steps

### Pressure Projection

what is p?

$\nabla u'=0$

a Poisson equation

Dirichlet boundary (open)
Neumann boundary (close)

## Air and Smoke

Air: 

- we update the flow
- density, temperature with use semi-Lagrangian.

Water:

- Volume-of-fluid
  - not precise
- A signed distance function defined over the grid.
- Semi-Lagrangian (volume loss)
- Level set method (volume loss)

