# Lecture XII - Smoothed Particle Hydrodynamics

## SPH Model

$A_i^{smooth}=\frac{1}{n}\sum_jV_jA_jW_{ij}$

query neighbors

smoothing kernel

how to compute volume?

$V_i=\frac{m_i}{\rho_i}$

$\rho_i^{smooth}=\sum_jV_j\rho_jW_{ij}=\sum_jm_jW_{ij}$

$V_i=\frac{m_i}{\sum_jm_jW_{ij}}$

$A_i^{smooth}=\frac{1}{n}\sum_j\frac{m_j}{\sum_km_kW_{jk}}A_jW_{ij}$

$\nabla A_i=\sum_jV_jA_j\nabla W_{ij}$

$\Delta A_i=\sum_jV_jA_j\Delta W_{ij}$

## SPH-Based Fluids

applying three forces on particle

- Gravity
  - easy!
- Fluid Pressure
- Fluid Viscosity

### Pressure force

$P_i=k((\frac{\rho_i}{\rho_{const}})^7-1)$

$F_i^{pressure}=-V_i\nabla_iP^{smooth}=-V)i\sum_jV_jP_j\nabla_iW_{ij}$

### Viscosity force

particles should move together in the same velocity.

$F_i^{viscosity}=-vm_i\Delta_iv^{smooth}=-vm_i\sum_jV_jv_j\Delta W_{ij}$

bottleneck of the performance? Querying!

Spatial Partition: 

- Separate the space into cells
- Each cell stores the particles in it
- To find the neighborhood of i, just look at the surrounding cells

Octree, Binary Spatial Partitioning tree

Reconstruction

SDF?

## Ongoing Research

- efficient
- incompressble
- water-air boundary
- predict particle movement?

talk

- 效率与效果的平衡
- 不赞同追求效果
- 学术工业脱节
  - sig难以落地
- 下一步
  - 基础
  - 自学没有大局观
  - 盯着一个方向，不要三心二意
- 斯坦福
- 图形学读博：模拟 > 渲染
