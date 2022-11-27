# Lecture X - Waves: An Intro to Fluid Dynamics

- Lagrangian Approach
- Eulerian Approach

## A Height Field Model

$\frac{dh(x)}{dt}+\frac{d(h(x)u(x))}{dx}=0$

$h(x+dx)u(x+dx)-h(x)u(x)$

$\frac{du(x)}{dt}=-u(x)\frac{du(x)}{dx}-\frac{1}{\rho}\frac{dP(x)}{dx}+a(x)$

advection

a(x): external force

$\frac{du(x)}{dt}=-\frac{1}{\rho}\frac{dP(x)}{dx}$

two equations:

$\frac{dh}{dt}+\frac{d(hu)}{dx}=0\implies\frac{d^2h}{dt^2}+h\frac{d^2u}{dxdt}=0$

$\frac{du}{dt}=-\frac{1}{\rho}\frac{dP}{dx}\implies\frac{d^2u}{dxdt}=-\frac{1}{\rho}\frac{d^2P}{dx^2}$

omit $u\frac{dh}{dx}$: assume height does not change rapidly.

then

$\frac{d^2h}{dt^2}=\frac{h}{\rho}\frac{d^2P}{dx^2}$

### Discretization

Finite Differencing

Central differencing: $\frac{df(t_0)}{dt}=\frac{df(t_0+\Delta t)-df(t_0-\Delta t)}{2\Delta t}$

$\frac{d^2h}{dt}=\frac{h(t_0+\Delta t)+h(t_0-\Delta t)-2h}{\Delta t^2}$

Laplacian operator

Volume Preservation

Solution 1: divide h

Solution 2: constant h

Reading: Rapid, Stable Fluid Dynamics for Computer Graphics.

Pressure: $P=\rho gh$

Viscosity

### Boundary Conditions

- Dirichlet boundary: open boundary
- Neumann boundary: zero-derivative boundary

### Two-Way Coupling

Virtual Height

Poisson’s Equation

Rigid Body Update
