# Lecture V - Cloth Simulation

## A Mass Spring System

Structured Spring Networks

bending edges

Topological Construction: sort and removal

Explicit Integration: overshooting when k or h is large.

Implicit Integration

$v^1=v^0+\Delta tM^{-1}f^1$  
$x^1=x^0+\Delta tv^1$

$\implies x^1=x^0+\Delta tv^0+\Delta t^2M^{-1}f^1(x^1)$

$x^1=(x^0+\Delta tv^0+\Delta t^2M^{-1}f_{ext})+\Delta t^2M^{-1}f_{int}(x^1)$

$x^1-y=\Delta t^2M^{-1}f_{int}(x^1)=-\Delta t^2M^{-1}\frac{\partial E}{\partial x}(x^1)$

$x^1=\argmin_x\frac{1}{2\Delta t^2}|x-y|_M^2+E(x)$

Newton-Raphson Method

$\Delta x=-(F'')^{-1}F'$

if $F''>0$, then $F$ has only one minimum. 

$F''=\frac{1}{\Delta t^2}M+H(x)$

$H=\sum_{e=\{i,j\}}\begin{bmatrix}
    H_e & -H_e \\
    -H_e & H_e \\
\end{bmatrix}$

$H_e=k\frac{x_{ij}x^T_{ij}}{||x_{ij}||^2}+k(1-\frac{L}{||x_{ij}||})(I-\frac{x_{ij}x^T_{ij}}{||x_{ij}||^2})$

if $||x_{ij}||<L$, the hessian may not be se.p.d.

When a spring is compressed, the spring Hessian may not be positive definite. This means there can be multiple local minima.

how to deal with non p.d.

- One solution is to simply drop the ending term.
- Stable But Responive Cloth.

Large Steps in Cloth Simulation.

## Bending and Locking Issues

### A Dihedral Angle Model

$f_i=f(\theta)u_i$

- $u_1=n_1$, $u_2=n_2$
- $(u_3-u_4)\cdot e=0$
  - in the span of $n_1$, $n_2$
- $u_1+u_2+u_3+u_4=0$

simulation of clothing with folds and wrinkles

### A Quadratic Bending Model

estimate laplacian energy

- assumptions
  - planar case
  - little streching
- Easy to implement.
- Compatible with implicit integration.
- No longer valid if cloth stretches much.
- Not suitable if the rest configuration is not planar.
  - Cubic shell model.
  - Projective dynamics model. 
  - Details skipped here.

A Quadratic Bending Model for Inextensible Surfaces

### The Locking Issue

we assumed cloth planar deformation and cloth bending deformation are independent. 

edges=3vertices-3-boundary_edges

DOF: 3 + boundary_edges.

布弯不了

- 减小弹簧压缩时的弹性系数
- 使弹簧可以在一定范围内自由移动，超出范围才开始计算弹力

## Shape Matching
