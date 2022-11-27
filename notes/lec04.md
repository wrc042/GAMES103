# Lecture IV - Rigid Contacts

## Particle Collision Detection and Response

- Penalty methods
- Impulse methods

A signed distance function $\phi(x)$

### Penalty Methods

Quadratic Penalty Method

Apply a force in the next update: $f=-k\phi(x)N$

Quadratic Penalty Method with a buffer

$f=\epsilon-k\phi(x)N$

Log-Barrier Penalty Method

$f=\rho\frac{1}{\phi(x)}N$

$\log\phi<0$ never happen

Summary

- The use of step size adjustment is a must.
  - To avoid overshooting.
  - To avoid penetration in log-barrier methods.
- Log-barrier method can be limited within a buffer as well.
- Frictional contacts are difficult to handle.

### Impulse Method

$x^{new}=x+|\phi(x)|N=x-\phi(x)\nabla\phi(x)$

$v_N^{new}=-\mu_Nv_N$

$v_T^{new}=-av_T$

$a=\max(1-\mu_T(1+\mu_N)||v_N||/||v_T||,0)$

can control friction and collision.

rigid: impulse
cloth: penalty

## Rigid Body Collision Detection and Response

vertex i:

$x_i=x+Rr_i$

$v_i=v+\omega\times Rr_i$

just by impulse

modify v and $\omega$

$v_i^{new}=v_i+\frac{1}{M}j-(Rr_i)\times(I^{-1}(Rr_i\times j))=Kj$

$K=\frac{1}{M}1-[Rr_i]_\times I^{-1}[Rr_i]_\times$

$j=K^{-1}\Delta v$

- If there are many vertices in collision, we use their average.
- We can decrease the restitution 𝜇𝐍 to reduce oscillation.
- We don’t update the position here.
  - non-linear

## Shape Matching

keypoint: enforce the rigidity constraint

$c=\frac{1}{N}\sum_iy_i$

$A=(\sum_i(y_i-c)r_i^T)(\sum_ir_ir_i^T)^{-1}$

Polar Decomposition

$A=RS$

$A=UDV^T=(UV^T)(VDV^T)=RS$

- Easy to implement and compatible with other nodal systems, i.e., cloth, soft 
bodies and even particle fluids.
- Difficult to strictly enforce friction and other goals.
  - The rigidification process will destroy them.