# Lecture VII - Linear Finite Element Method

## Linear Finite Element Method(FEM)

$x=FX+c$

$F=\frac{\partial x}{\partial X}$, deformation gradient.

Green Strain

get rid of U

$G=\frac{1}{2}(F^TF-I)=\frac{1}{2}(VD^2V^T-I)$

- rotation invariant
- $G=0$, not deformation

energy:$E=\int W(G)dA=A^{ref}W(G)$

The Saint Venant-Kirchhoff Model (StVK)

$\frac{\partial W}{\partial G}=2\mu G+\lambda tr(G)I=S$

stress tensor

$f_i=-A^{ref}(\frac{\partial W}{\partial x_i})^T$

$[f_1\ f_2]=-A^{ref}FS[X_{10}\ X_{20}]^{-T}$

Reading: A simple approach to nonlinear tensile stiffness for accurate cloth simulation.

## Finite Volume Method

interface force: $f=\oint_Ltdl$

stress tensor: $t=\sigma n$

$\sigma$ is constant in the triangle

Divergence Theorem

$f_0=-\oint_{L_{20}}\sigma n_{20}dl-\oint_{L_{10}}\sigma n_{10}dl=-\sigma(\frac{||x_{20}||}{2}n_{20}+\frac{||x_{10}||}{2}n_{10})$

$f_0=\frac{-\sigma}{6}(x_{10}\times x_{20}+x_{20}\times x_{30}+x_{30}\times x_{10})$

defferent stress

|   i/o   |  ref  | current  |
| :-----: | :---: | :------: |
|   ref   |   S   |          |
| current |   P   | $\sigma$ |

$P=FS$

$\sigma=\det^{-1}(F)PF^T$

$f_0=-\frac{FS}{6}b_0$

Reading: Finite Volume Methods for the Simulation of Skeleton Muscles.

Reading: A Simple Approach to Nonlinear Tensile Stiffness for Accurate Cloth Simulation.

A FEM/FVM Framework

- $D_m=[X_{10}\ X_{20}\ X_{30}]$
- $F=[x_{10}\ x_{20}\ x_{30}]D^{-1}_m$
- $G=\frac{1}{2}(F^TF-I)$
- $P=F\frac{\partial W}{\partial G}$
- $[f_1\ f_2\ f_3]=\frac{1}{6\det(D^{-1}_m)}PD^{-T}_m$
- $f_0=-f_1-f_2-f_3$

## Hyperelastic Models

Isotropic Materials

$P(F)=UP(\lambda_0,\lambda_1,\lambda_2)V^T$

StVK, neo-Hookean, Mooney-Rivlin model, Fung model.

- FEM uses the derivates of the strain energy function to obtain the force.
- FVM uses the integral of the interface traction to obtain the force.
- The two approaches lead to the identical outcome, in different formulations
