# 🧠 FLAM Assignment — Parametric Curve Fitting using L1 Optimization

## 🎯 Objective

The goal of this assignment is to determine the **unknown parameters** 
$\theta$, $M$, and $X$ in the given **parametric equation of a curve** 
so that the predicted curve best matches the provided dataset.

---

## 🧩 Problem Definition

The given equations are:

\[
\begin{aligned}
x(t) &= t\cos(\theta) - e^{M|t|}\sin(0.3t)\sin(\theta) + X, \\[6pt]
y(t) &= 42 + t\sin(\theta) + e^{M|t|}\sin(0.3t)\cos(\theta)
\end{aligned}
\]

with the following parameter constraints:

\[
0^\circ < \theta < 50^\circ, \quad
-0.05 \le M \le 0.05, \quad
0 < X < 100, \quad
6 < t < 60
\]

The dataset contains **1500 observed points** $(x_i, y_i)$ for values of $t$ within this range.

Our task is to **estimate $\theta$, $M$, and $X$** so that the predicted curve 
minimizes the total **L1 distance** from the observed points.

---

## 🧮 L1 Distance Metric

The **L1 distance** (sum of absolute differences) is defined as:

\[
L1_{\text{sum}} = \sum_{i=1}^{1500} 
\left( 
|x_i^{pred} - x_i^{obs}| + |y_i^{pred} - y_i^{obs}|
\right)
\]

\[
L1_{\text{mean}} = \frac{L1_{\text{sum}}}{1500}
\]

- Lower $L1_{\text{sum}}$ means the model fits the data better.  
- This metric matches the assignment’s evaluation criterion (out of 100 marks).

---

## ⚙️ Step-by-Step Approach

### **Step 1 — Data Loading**

We first read the file `xy_data.csv` which contains the $(x, y)$ coordinates of points on the curve.  
A parameter vector `t` is generated using a **uniform spacing** between 6 and 60:

```python
df = pd.read_csv("xy_data.csv")
x_obs, y_obs = df['x'].values, df['y'].values
t = np.linspace(6, 60, len(df))
