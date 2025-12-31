# Trusted Setup (Powers of Tau)

**Source:** [RareSkills](https://rareskills.io/post/trusted-setup)

## 🦁 The Core Concept
A **Trusted Setup** is a mechanism that allows a Prover to evaluate a polynomial at a **secret point** $\tau$ (tau), without anyone (Prover or Verifier) knowing what $\tau$ actually is.

* **Why?** In ZK-SNARKs (like Groth16), we need to ensure the Prover evaluates their polynomial at a random point chosen by the Verifier. However, if the Verifier simply sends the point $x=5$, the Prover could cheat by constructing a fake polynomial that works only at $x=5$.
* **Solution:** We encrypt powers of a secret number $\tau$ and give them to the world. This is called the **Structured Reference String (SRS)**.

---

## 🔐 The Math: Structured Reference String (SRS)

### 1. Polynomial Evaluation as Inner Product
A polynomial $P(x) = c_d x^d + \dots + c_1 x + c_0$ can be evaluated by taking the **dot product** of coefficients and powers of $x$.

Example: $P(x) = 4x^2 + 7x + 8$
$$P(x) = [4, 7, 8] \cdot [x^2, x^1, x^0]$$

### 2. Encrypting the Powers
Instead of revealing $\tau$ (which would allow cheating), a trusted entity generates $\tau$, computes its powers, encrypts them on an Elliptic Curve, and then **deletes $\tau$**.

**The SRS:**
$$[G_1, \tau G_1, \tau^2 G_1, \dots, \tau^d G_1]$$

* **Public Knowledge:** The SRS (list of points).
* **Secret Knowledge:** $\tau$ (Nobody knows this, hopefully).

### 3. Blind Evaluation
Using Homomorphic Encryption (Elliptic Curve Scalar Multiplication), anyone can now evaluate any polynomial $P(x)$ at $\tau$ without knowing $\tau$.

If $P(x) = 4x^2 + 7x + 8$:
$$P(\tau) G_1 = 4(\tau^2 G_1) + 7(\tau G_1) + 8(G_1)$$

* The result is the point $[P(\tau)]_1$.
* The Prover calculated this **without knowing $\tau$**.

---

## 🐍 Python Implementation

```python
from py_ecc.bn128 import G1, multiply, add
from functools import reduce

# 1. THE SETUP (Done once, then tau is deleted)
tau = 88
degree = 3
# SRS = [tau^3 * G, tau^2 * G, tau^1 * G, 1 * G]
srs = [multiply(G1, tau**i) for i in range(degree, -1, -1)]

# 2. THE EVALUATION (Done by Prover)
# Polynomial: P(x) = 4x^2 + 7x + 8
coeffs = [0, 4, 7, 8] # Pad with 0 for x^3 term

def inner_product(points, coeffs):
    # Sum of (coeff_i * point_i)
    return reduce(add, map(multiply, points, coeffs))

# Result is P(tau) * G encrypted
encrypted_result = inner_product(srs, coeffs)