# Lecture III - Rigid Body Dynamics

## Translational Motion

Explicit Euler

$\int^{t_1}_{t_0}v(t)dt=\Delta tv(t_0)+O(\Delta t^2)$

Implicit Euler

$\int^{t_1}_{t_0}v(t)dt=\Delta tv(t_1)+O(\Delta t^2)$

Mid-point

$\int^{t_1}_{t_0}v(t)dt=\Delta tv(t_{0.5})+O(\Delta t^3)$

explicit+implicit

$v_{1}=v_{0}+\Delta tM^{-1}f_{0}$  
$x_1=x_0+\Delta tv_{1}$

Leapfrog Integration

semi-implicit

mid-point+mid-point

$v_{0.5}=v_{-0.5}+\Delta tM^{-1}f_{0}$  
$x_1=x_0+\Delta tv_{0.5}$

## Rotational Motion

Matrix, Euler Angles, Quaternion

$q=[s\ v]$  
$q_1\times q_2=[s_1s_2-v_1\cdot v_2\ s_1v_2+s_2v_1+v_1\times v_2]$  
$q=[\cos\frac{\theta}{2}\ v]$

Torque and Inertia

$\tau_i=(Rr_i)\times f_i$  
$\tau=\sum\tau_i$

$I_{ref}=\sum m_i(r_i^Tr_i1-r_ir_i^T)$  
$I=RI_{ref}R^T$

$\omega_1=\omega_0+\Delta t(I_0)^{-1\tau_0}$  
$q_1=q_0+[0\ \frac{\Delta t}{2}\omega_1]\times q_0$ (why?)

need to be normalized

$s=\{v,x,w,q\}$
