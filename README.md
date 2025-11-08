# FLAM Placement Assignment - Research & Development / AI

---

### 1. Problem Statement

We are given **1500 points** $(x_i, y_i)$ that lie on a **parametric curve** defined as:

$$
x(t) = t \cos(\theta) - e^{M|t|} \sin(0.3t) \sin(\theta) + X,
$$

$$
y(t) = 42 + t \sin(\theta) + e^{M|t|} \sin(0.3t) \cos(\theta)
$$

with unknown constants **$(\theta, M, X)$**
.

The given parameter ranges are:

$$
0^\circ < \theta < 50^\circ, \quad -0.05 < M < 0.05, \quad 0 < X < 100, \quad 6 < t < 60.
$$


**Goal:**  
Estimate the parameters $(\theta, M, X)$ such that the model curve matches the given dataset $(x, y)$.  

The model performance is evaluated using the **L1 distance** (sum of absolute deviations) between the predicted and observed points.

---

### 2. Implementation Tools

| Task | Library / Function Used |
|------|--------------------------|
| Data handling | `pandas` |
| Numerical computation | `numpy` |
| Optimization | `scipy.optimize.minimize`, `differential_evolution` |
| Visualization | `matplotlib` |

---

### 3. Mathematical Model

For each $t_i$, we compute predicted values $\hat{x}_i$ and $\hat{y}_i$ as:

\[
\hat{x}_i = t_i \cos(\theta) - e^{M|t_i|}\sin(0.3t_i)\sin(\theta) + X,
\]

\[
\hat{y}_i = 42 + t_i \sin(\theta) + e^{M|t_i|}\sin(0.3t_i)\cos(\theta)
\]

The goal is to find $\theta, M, X$ that minimize the difference between predicted and observed points.

---

### 4. L1 Cost Function (Loss)

**Purpose:**  
Implement the objective that the optimizer will minimize.

**Mathematical Definition:**

\[
L1_{sum}(\theta,M,X) = \sum_{i=1}^{1500} \big(|x_i^{pred}-x_i^{obs}| + |y_i^{pred}-y_i^{obs}|\big)
\]

**Implementation Details:**
- The function checks bounds internally. If a parameter leaves the valid region, it returns a very large penalty.  
  This enforces constraints even when using optimizers that ignore bounds.
- The sum of absolute differences is used (not squared errors).  
- For reporting, the mean L1 is also computed as $L1_{mean} = L1_{sum} / n$.

**Why Choose L1:**
- L1 is robust to outliers.
- It directly corresponds to the **assignment scoring metric** (max score = 100).

---

### 5. Parameter Bounds

**Purpose:**  
Set the valid search domain for the optimizer.

**Defined Bounds:**

- $\theta \in (0^\circ, 50^\circ)$ — converted to radians in code:  
  `(deg2rad(0.001), deg2rad(50))`
- $M \in [-0.05, 0.05]$
- $X \in [0, 100]$

**Why Bounds Are Important:**
- They constrain the search to physically and assignment-allowed ranges.
- They accelerate convergence by reducing the optimization space.
- They prevent unrealistic or numerically unstable parameter values.

**Practical Note:**  
Two types of bound enforcement are used:
1. A `bounds` structure for optimizers that support bounded search (e.g., Differential Evolution).  
2. An internal penalty in `l1_cost` for invalid values — ensures consistent constraint handling across methods.

---

### 6. Optimization - Global Search (Differential Evolution)

**Purpose:**  
Find a robust initial solution across the entire parameter space.

**Why Differential Evolution (DE)?**
- The L1 objective is **nonlinear** and **non-convex** due to trigonometric and exponential terms.  
- DE is a **population-based global optimizer** that explores the parameter space broadly and avoids local minima.  
- It does not require derivatives and handles non-smooth objectives like L1.

**Key Settings Used:**
- `maxiter=300`, `popsize=15` — trade-off between reliability and computation time.  
- `polish=True` — applies a local optimizer automatically after convergence.  
- `seed` — ensures reproducibility.

**Output:**
- `res_de.x` → best parameter candidate.  
- `res_de.fun` → corresponding L1 value.

**Reason for Global Start:**
Local methods (e.g., Powell) can get stuck in local minima.  
Differential Evolution provides a solid starting point for local refinement.

---

### 7. Optimization - Local Refinement (Powell)

**Purpose:**  
Refine the solution obtained from DE for better precision.

**Why Powell’s Method?**
- Works well for non-smooth, derivative-free functions (like L1).  
- Efficiently refines results obtained from global optimizers.  

**Implementation Notes:**
- Run after DE using:
  ```python
  res_local = minimize(l1_cost, res_de.x, method='Powell')
