# Quadratic Arithmetic Programs (QAP)

**Source:** [RareSkills](https://rareskills.io/post/quadratic-arithmetic-program)

## 🦁 Motivation: Why QAP?
We previously learned about **Rank 1 Constraint Systems (R1CS)**, which represent a computation as a system of vector equations:
$$(\mathbf{L} \cdot \mathbf{s}) \times (\mathbf{R} \cdot \mathbf{s}) = (\mathbf{O} \cdot \mathbf{s})$$

**The Problem:**
To verify an R1CS, the verifier must check this equation for **every single row** (constraint). If a circuit has 1,000,000 constraints, the verifier does 1,000,000 checks. This is too slow for blockchain/ZK applications.

**The Solution (QAP):**
We convert these thousands of constraints into **Polynomials**.
* Vectors $\rightarrow$ Polynomials
* Checking 1,000,000 constraints $\rightarrow$ Checking **1 Polynomial Identity**.

Using the **Schwartz-Zippel Lemma**, if the polynomial identity holds at **one random point**, it holds everywhere (with overwhelming probability).

---

## 🔄 The Transformation: R1CS $\rightarrow$ QAP

The core trick is using **Lagrange Interpolation** to transform the *columns* of the R1CS matrices into polynomials.

### 1. Matrix to Polynomials
Imagine our R1CS matrices $L, R, O$ are size $n \times m$ ($n$ constraints, $m$ variables).
Instead of looking at the **rows** (constraints), we look at the **columns**.

For each column $i$ (associated with variable $w_i$), we create a polynomial $L_i(x)$ such that:
* At $x=1$, $L_i(1)$ equals the value in row 1, column $i$.
* At $x=2$, $L_i(2)$ equals the value in row 2, column $i$.
* At $x=n$, $L_i(n)$ equals the value in row $n$, column $i$.

We do this for all matrices to get sets of polynomials: $\{L_i(x)\}, \{R_i(x)\}, \{O_i(x)\}$.

### 2. The New Equation
In R1CS, we computed a dot product of the witness $\mathbf{s}$ with a row vector.
In QAP, we compute a **linear combination** of polynomials weighted by the witness $\mathbf{s}$.

$$P(x) = \left( \sum_{i=0}^m s_i \cdot L_i(x) \right) \cdot \left( \sum_{i=0}^m s_i \cdot R_i(x) \right) - \left( \sum_{i=0}^m s_i \cdot O_i(x) \right)$$

* If we evaluate $P(x)$ at $x=1$, we get exactly the first constraint of the R1CS.
* If we evaluate $P(x)$ at $x=2$, we get exactly the second constraint.
* Therefore, if the R1CS is satisfied, $P(x)$ must be **zero** at all points $x = 1, 2, \dots, n$.

---

## 🎯 The Target Polynomial $Z(x)$
We need to prove that $P(x)$ is zero at all valid constraint indices ($1, 2, \dots, n$).
We define a **Target Polynomial** (or Vanishing Polynomial) $Z(x)$ that is zero *only* at these points:
$$Z(x) = (x-1)(x-2)\dots(x-n)$$

### The Divisibility Check
If $P(x)$ is truly zero at all these points, then $P(x)$ must be perfectly divisible by $Z(x)$ without a remainder.
$$P(x) = H(x) \cdot Z(x)$$
* $H(x)$ is the quotient polynomial.

---

## ✅ The Verification Process
Instead of checking dot products, the Verifier now checks:
**Does $H(x) \cdot Z(x)$ equal $L(x)R(x) - O(x)$?**

1.  **Prover** computes $P(x)$ and divides it by $Z(x)$ to find $H(x)$.
2.  **Verifier** picks a random point $\tau$ (tau).
3.  **Verifier** evaluates:
    $$L(\tau) \cdot R(\tau) - O(\tau) \stackrel{?}{=} H(\tau) \cdot Z(\tau)$$

If this equation balances at a single random point $\tau$, then the prover almost certainly holds a valid witness for the entire circuit.

---

## 🐍 Python Implementation Steps

1.  **Setup Field:** Use `galois` library for Finite Field arithmetic.
2.  **Interpolate:** Turn R1CS columns into polynomials.
    ```python
    # Example: Interpolating column 0 of Matrix L
    # values at x=1, x=2, x=3 might be [0, 5, 0]
    L_poly_0 = galois.lagrange_poly([1, 2, 3], [0, 5, 0])
    ```
3.  **Compute Aggregate Polynomials:**
    Multiply each polynomial by its corresponding witness element ($s_i$) and sum them up.
    ```python
    L_sum = s[0]*L_poly_0 + s[1]*L_poly_1 + ...
    R_sum = s[0]*R_poly_0 + ...
    O_sum = s[0]*O_poly_0 + ...
    ```
4.  **Compute P(x):**
    ```python
    P = L_sum * R_sum - O_sum
    ```
5.  **Compute Z(x):**
    ```python
    Z = galois.Poly([1, -1]) * galois.Poly([1, -2]) * ... 
    # Roots at 1, 2, etc.
    ```
6.  **Verify Divisibility:**
    ```python
    H, remainder = divmod(P, Z)
    assert remainder == 0
    ```