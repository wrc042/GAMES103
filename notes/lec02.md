# Lecture II - Math Background

## Vector

left/right hand system

- left: OpenGL, Research
- right: Unity, DirectX

Vector Norm

### Dot Prodoct

$p^Tq$, $<p,q>$

particle-line projection

$s=(q-o)^T\={v}$

plane representation

$(p-c)^Tn=0$

### Cross Product

triangle normal and area

$A=||x_{10}\times x_{20}||_2$

Triangle inside/outside test

$(x_0\times p)\times(x_1-p)\cdot n$

barycentric coordinates

tetrahedral volume

$V=\frac{1}{3}hA=\frac{1}{6}x_{30}\cdot x_{10}\times x_{20}=\frac{1}{6}\begin{vmatrix}
    x_1 & x_2 & x_3 & x_0 \\
    1 & 1 & 1 & 1 \\
\end{vmatrix}$

this volume is signed

barycentric weight: tetrahedral

particle-triangle intersection

$(p-x_0+tv)\cdot x_{10}\times x_{20}=0$

$\implies t=\frac{(p-x_0)\cdot x_{10}\times x_{20}}{v\cdot x_{10}\times x_{20}}$

## Matrices

Transpose, Diagnal, Identity, Symmetry

$A^TA$: symmetry  
$A+A^T$: symmetry

Orthognality: $A^TA=I$

SVD

$A=UDV^T$: D diagnal, UV orthogonal

rotation - scale - rotation

Eigenvalue for **symmetry matrix**

$A=UDU^{-1}$: D diagnal, U orthogonal

$Au_i=d_iu_i,U=[\dots\ u_i \dots]$

s.p.d Symmetric Positive Definiteness

$v^TAv>0$

$v^T(UDU^T)v=(U^Tv)^TD(U^Tv)>0$

A is sy.p.d. if only if all of its eigenvalues are positive.

In practice, people often choose other ways to check if A is sy.p.d.

$a_{ii}>\sum_{i\neq j}|a_{ij}|$ for all i  
A diagonally dominant matrix is p.d.

diagonally dominant matrix $\implies$ p.d.

A sy.p.d. matrix must be invertible.

if A is sy.p.d., $B=\begin{bmatrix}
    A & -A \\
    -A & A \\
\end{bmatrix}$ is symmetric semi-definite.

### Linear Solver

Ax=b

direct solver: A=LU

- Ly=b
- Ux=y

If we must solve many linear systems with the same 𝐀, we can factorize it only once.

iterative solver

$x^{[k+1]}=x^{[k]}+\alpha M^{-1}(b-Ax^{[k]})$

$b-Ax^{[k]}=(I-\alpha AM^{-1})^{k}(b-Ax^{[0]})$

spectral radius(the largest absolute value of the eigenvalues)

M always be diag(A)(Jacobi Method) or lower(A)(Gauss-Seidel Method)

The convergence can be accelerated: Chebyshev, Conjugate Gradient.

- pros: simple, fast, parallelable
- cons: converge condition, slow

## Tensor Calculus

gradient, Jacobian, Divergence, Curl

$\nabla\times f=\begin{bmatrix}
    \frac{\partial h}{\partial y} - \frac{\partial g}{\partial z} \\
    \frac{\partial f}{\partial z} - \frac{\partial h}{\partial x} \\
    \frac{\partial g}{\partial x} - \frac{\partial f}{\partial y} \\
\end{bmatrix}$

Hessian, Laplacian

$\frac{\partial||x||}{\partial x}=\frac{x^T}{||x||}$
