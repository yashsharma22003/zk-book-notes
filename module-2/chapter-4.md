# Lagrange Interpolation with Python

**Source:** [RareSkills](https://rareskills.io/post/python-lagrange-interpolation)

## 📉 What is Interpolation?
Interpolation is the process of finding a polynomial that passes through a specific set of points (data).
* **Goal:** Given a set of $x$ and $y$ values, find a polynomial $P(x)$ such that for every pair, $P(x_i) = y_i$.
* **Degree Rule:** For $n$ points, you generally need a polynomial of degree at most $n-1$.
    * 1 Point $\to$ Degree 0 (Constant line)
    * 2 Points $\to$ Degree 1 (Straight line)
    * 3 Points $\to$ Degree 2 (Parabola)


---

## 🐍 Python Implementation

We don't need to solve the math manually; we can use standard libraries.

### 1. Floating Point Example (`scipy`)
Used for standard arithmetic (real numbers).

```python
from scipy.interpolate import lagrange

# Points: (1, 4), (2, 8), (3, 2), (4, 1)
x_values = [1, 2, 3, 4]
y_values = [4, 8, 2, 1]

# Compute Polynomial
poly = lagrange(x_values, y_values)

print(poly)
# Output: A cubic polynomial (approx)
# 2.5 x^3 - 20 x^2 + 46.5 x - 25