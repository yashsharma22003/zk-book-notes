# The Schwartz-Zippel Lemma

**Source:** [RareSkills](https://rareskills.io/post/schwartz-zippel-lemma)

## 🦁 The Core Theorem
The Schwartz-Zippel Lemma is the mathematical foundation that allows Zero Knowledge Proofs to be **succinct** (short and fast to verify).

**The Lemma states:**
If you have two non-equal polynomials $P(x)$ and $Q(x)$ with degrees $d_1$ and $d_2$ respectively, the number of points where they intersect ($P(x) = Q(x)$) is at most $\max(d_1, d_2)$.

* **In simple terms:** Two different curves can only cross each other a few times.
    * Two lines (degree 1) cross at most **1** time.
    * A line and a parabola (degree 2) cross at most **2** times.
    * Two cubic curves (degree 3) cross at most **3** times.

---

## 🎲 Probabilistic Equality Testing
This lemma gives us a powerful way to check if two massive polynomials are identical without checking every single coefficient.

### The Problem
Comparing two polynomials coefficient-by-coefficient takes $O(d)$ time. If the degree $d$ is 1,000,000, this is slow.

### The Solution (The "Random Point" Trick)
1.  **Pick a random point** $r$ from a very large finite field $\mathbb{F}_p$.
2.  **Evaluate** both polynomials at that point: $y_1 = P(r)$ and $y_2 = Q(r)$.
3.  **Check:** If $y_1 = y_2$, then $P(x)$ and $Q(x)$ are almost certainly identical.

### Why it works
If $P(x) \neq Q(x)$, the chance of picking a random $r$ where they "accidentally" intersect is:
$$\text{Probability} \le \frac{\text{Degree}}{\text{Size of Field}}$$

* **Example:**
    * Degree $d = 10^6$ (1 million).
    * Field Size $p \approx 2^{256}$ (Number of atoms in the universe is $\approx 2^{266}$).
    * Probability of error $\approx \frac{10^6}{2^{256}}$.
    * This probability is so infinitesimally small it is considered impossible in cryptography.

---

## 📉 Comparing Vectors with Polynomials
We can combine **Lagrange Interpolation** with **Schwartz-Zippel** to compress vector comparisons.

**Goal:** Check if Vector $A$ == Vector $B$.
* **Naive Method:** Loop through index $i$ and check $A[i] == B[i]$. (Linear time, $O(n)$).
* **Polynomial Method:**
    1.  Convert Vector $A$ to Polynomial $P_A(x)$.
    2.  Convert Vector $B$ to Polynomial $P_B(x)$.
    3.  Pick **one** random point $r$.
    4.  Check if $P_A(r) == P_B(r)$.

If the polynomials match at one random point, the vectors are identical with overwhelming probability.

### Python Example
```python
import galois
import numpy as np
import random

p = 103 # Small prime for example
GF = galois.GF(p)

# Vectors to compare
v1 = GF(np.array([4, 8, 19]))
v2 = GF(np.array([4, 8, 19]))

# Interpolation domain (x=1, x=2, x=3)
xs = GF(np.array([1, 2, 3]))

# 1. Turn vectors into polynomials
p1 = galois.lagrange_poly(xs, v1)
p2 = galois.lagrange_poly(xs, v2)

# 2. Pick ONE random point
u = random.randint(0, p)

# 3. Check equality at that single point
lhs = p1(u)
rhs = p2(u)

assert lhs == rhs
# We checked the whole vector by checking just one number!