# Converting R1CS to Quadratic Arithmetic Programs (QAP)

**Source:** [RareSkills](https://rareskills.io/post/r1cs-to-qap)

## 🦁 Motivation
R1CS (Rank 1 Constraint Systems) are efficient for *representing* a circuit, but inefficient for *verifying* it because the verifier must check thousands of constraints individually.

**QAP (Quadratic Arithmetic Programs)** solve this by bundling all constraints into a **single polynomial equation**.
* If the polynomial equation holds true, then *all* R1CS constraints are simultaneously satisfied.
* This allows the verifier to check the proof at a **single random point** (using the Schwartz-Zippel Lemma).

---

## 🛠 The Example Circuit
We are proving the validity of the equation:
$$out = x^4 - 5y^2x^2$$

This translates to the following R1CS Constraints (4 rows):
1.  $v_1 = x \cdot x$
2.  $v_2 = v_1 \cdot v_1$ (which is $x^4$)
3.  $v_3 = -5y \cdot y$
4.  $out = v_3 \cdot v_1 + v_2$ (This linear combination involves no new multiplication, just addition).

---

## 🐍 Implementation Steps

### 1. Define Matrices over a Finite Field
Unlike standard Python math, we must use a Finite Field (e.g., Prime 79).
We use the `galois` library for this.

**Note:** Negative numbers (like -5) must be converted to field elements:
$$-5 \pmod{79} = 74$$

```python
import galois
import numpy as np

GF = galois.GF(79)

# Define R1CS Matrices (L, R, O)
# Rows = Constraints, Cols = Witness Variables [1, out, x, y, v1, v2, v3]
L = GF(np.array([...])) 
R = GF(np.array([...]))
O = GF(np.array([...]))

# Compute Witness
x = GF(4)
y = GF(77) # -2
# ... compute rest of witness ...
witness = GF(np.array([1, out, x, y, v1, v2, v3]))