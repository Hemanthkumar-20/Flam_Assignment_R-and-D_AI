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

### 3️. Mathematical Model

For each $t_i$, we compute predicted values $\hat{x}_i$ and $\hat{y}_i$ as:

$$
\hat{x}_i = t_i \cos(\theta) - e^{M|t_i|}\sin(0.3t_i)\sin(\theta) + X,
$$

$$
\hat{y}_i = 42 + t_i \sin(\theta) + e^{M|t_i|}\sin(0.3t_i)\cos(\theta).
$$

The goal is to find $\theta, M, X$ that minimize the difference between predicted and observed points.

---

### 4. L1 Cost Function (Loss)

**Code purpose:** implement the objective that the optimizer will minimize.

**L1 definition used**
$
L1_{sum}(\theta,M,X) = \sum_{i=1}^{1500} \big(|x_i^{pred}-x_i^{obs}| + |y_i^{pred}-y_i^{obs}|\big)
\$

**Implementation details**
- The function checks bounds internally. If a parameter leaves the valid region, it returns a very large penalty. This enforces constraints even when using optimizers that ignore bounds.
- We sum absolute differences across x and y. We return the scalar L1 sum (not normalized). For reporting we also compute mean L1 by dividing by `n`.

**Why choose L1 here:**
- L1 is robust to outliers and is exactly the grading metric. Minimizing it directly typically yields the best score for the assignment.

---

### 5. Parameter Bounds

**Code purpose:** set the search domain for the optimizer.

- $\theta \in (0^\circ, 50^\circ)$ - we convert to radians for computation:  
  `(deg2rad(0.001), deg2rad(50))`
- $M \in [-0.05, 0.05]$
- $X \in [0, 100]$

**Why bounds matter**
- They constrain the search to physically/assignment-allowed values.
- They speed optimization by reducing search volume.
- They prevent the optimizer from wandering into unmeaningful parameter zones that could produce numerical or modeling artefacts.

**Practical note:**  
We maintain both:
1. A `bounds` structure for optimizers that support bounds (like Differential Evolution), and  
2. An internal check inside `l1_cost` to return a large penalty if the parameters violate bounds - this ensures consistent constraint handling across all optimization methods.

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

**Code purpose:** refine the DE candidate to reduce L1 further.

**Why a local method next?**
- DE is good for global exploration but not for high-precision local convergence.
- Powell’s method (derivative-free line-search) is robust with non-smooth objectives like L1: it refines the solution without needing derivatives.

**Implementation notes**
- We call `scipy.optimize.minimize(..., method='Powell')` starting from `res_de.x`.
- Some local solvers accept bounds directly; Powell does not strictly enforce bounds in SciPy, so our `l1_cost` checks bounds and returns a large penalty if a candidate violates them.
- After local refinement, `res_local.x` is the final parameter vector and `res_local.fun` is the L1 at that point.

**Why this two-step strategy is effective**
- Global search finds a good region; local refinement squeezes the final percentage points from the objective, giving a better L1 for grading.

### 8. Final Best-Fit Parameters

After running Differential Evolution (global) followed by Powell (local) optimization  
with constraints $-0.05 < M < 0.05$, the best-fit parameters obtained are:

| Parameter | Symbol | Value | Unit |
|------------|---------|--------|------|
| Theta (radians) | $\theta$ | **0.490780889103** | rad |
| Theta (degrees) | $\theta$ | **28.119674** | ° |
| Exponential rate | $M$ | **0.021395863861** | — |
| Horizontal shift | $X$ | **54.898439840215** | — |

**L1 metrics:**

\$[
L1_{\text{sum}} = 37865.095792, \qquad
L1_{\text{mean}} = 25.243397
\]$

---

#### Final Fitted Parametric Equation

The optimized parametric curve is given by:

<div align="center">

$$
\begin{aligned}
x(t) &= t\cos(0.490780889103)
- e^{0.021395863861|t|}\sin(0.3t)\sin(0.490780889103)
+ 54.898439840215, \\[6pt]
y(t) &= 42 + t\sin(0.490780889103)
+ e^{0.021395863861|t|}\sin(0.3t)\cos(0.490780889103)
\end{aligned}
$$

</div>

---

#### Notes

- $\theta = 28.119674^\circ$ controls the **rotation/tilt** of the curve.  
- $M = 0.021395863861$ defines the **exponential scaling** of oscillations with $|t|$.  
- $X = 54.898439840215$ gives the **horizontal offset**.  
- The **L1 metric** (sum of absolute deviations) quantifies total fitting error between predicted and observed points smaller is better.

