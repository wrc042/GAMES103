# Lecture VIII - Linear Finite Element Method II

## Nonlinear optimization

Gradient Descent

- Exact Line Search
  - Fast convergence
  - Large Overhead
  - Complicated
- Backtracking Line Search
  - Simple
  - Low Overhead

Any method using a positive definite matrix P to modify the gradient yields a descent method

$d(x)=-P^{-1}\nabla F(x)$

A unified descent framework

- $P=diag(\frac{\partial^2 F(x)}{\partial x^2})$
- $P=lower(\frac{\partial^2 F(x)}{\partial x^2})$
- $P=const$

Reading: Descent Methods for Elastic Body Simulation 
on the GPU
