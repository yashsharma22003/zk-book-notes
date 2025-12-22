# Converting Algebraic Circuits to R1CS (Rank One Constraint System)

**Source:** [RareSkills](https://rareskills.io/zk-book)

## 🏗 What is R1CS?
A **Rank 1 Constraint System (R1CS)** is a specific standard for formatting arithmetic circuits. It is the bridge between human-readable algebraic constraints and the mathematical format required by Zero Knowledge Proof systems (like Groth16) and Bilinear Pairings.

### The "One Multiplication" Rule
The defining characteristic of R1CS is that **every equality constraint must contain exactly one multiplication**.
* Allowed: $a \times b = c$
* Allowed: $(a + b) \times (c + d) = e$
* **Not** Allowed: $a \times b \times c = d$ (Must be broken into two steps)

Formally, every constraint is written as the product of two linear combinations equaling a third linear combination:
$$(A \cdot \mathbf{w}) \times (B \cdot \mathbf{w}) = (C \cdot \mathbf{w})$$
*Where $\mathbf{w}$ is the "witness vector" and $A, B, C$ are coefficient vectors (rows in a matrix).*

---

## 🔑 The Witness Vector ($\mathbf{w}$)
In a standard algebraic circuit, the "witness" is just the solution values. In R1CS, the witness vector $\mathbf{w}$ must contain **every value** in the system: the inputs, the output, the intermediate variables, and a mandatory constant.



**Structure:**
$$\mathbf{w} = [1, \text{output}, \text{inputs}, \text{intermediate\_vars}]$$

* **The '1' (Index 0):** The first element is always 1. This allows us to represent constants in our constraints (e.g., adding 5 is represented as $5 \times \mathbf{w}[0]$).
* **Completeness:** To generate a valid proof, the prover must know every single value in this vector.

---

## 🔄 The Transformation Process

We transform the circuit into three matrices:
1.  **$\mathbf{L}$ (Left input):** The variables entering the multiplication from the left.
2.  **$\mathbf{R}$ (Right input):** The variables entering the multiplication from the right.
3.  **$\mathbf{O}$ (Output):** The result of the multiplication.

For a system with $m$ constraints and a witness vector of size $n$, these are $m \times n$ matrices.

### Example 1: Simple Multiplication
**Equation:** $r = x \times y$
**Witness:** $\mathbf{w} = [1, r, x, y]$

We map the equation $x \times y = r$:
* **Left:** $x$ is at index 2. Vector: $[0, 0, 1, 0]$
* **Right:** $y$ is at index 3. Vector: $[0, 0, 0, 1]$
* **Output:** $r$ is at index 1. Vector: $[0, 1, 0, 0]$

**Verification (Dot Product):**
$$(\mathbf{L} \cdot \mathbf{w}) \times (\mathbf{R} \cdot \mathbf{w}) = (\mathbf{O} \cdot \mathbf{w})$$
$$(1 \cdot x) \times (1 \cdot y) = (1 \cdot r)$$

---

### Example 2: Multiple Multiplications (Intermediates)
**Equation:** $r = x \times y \times z \times u$
**Issue:** This constraint has 3 multiplications. R1CS allows only one per row.
**Solution:** Break it down using **intermediate variables**.

1.  $v_1 = x \times y$
2.  $v_2 = z \times u$
3.  $r = v_1 \times v_2$



**Witness:** $\mathbf{w} = [1, r, x, y, z, u, v_1, v_2]$

**The Matrices (3 Rows):**

**Constraint 1 ($v_1 = x \cdot y$):**
* $\mathbf{L}$: Select $x$.
* $\mathbf{R}$: Select $y$.
* $\mathbf{O}$: Select $v_1$.

**Constraint 2 ($v_2 = z \cdot u$):**
* $\mathbf{L}$: Select $z$.
* $\mathbf{R}$: Select $u$.
* $\mathbf{O}$: Select $v_2$.

**Constraint 3 ($r = v_1 \cdot v_2$):**
* $\mathbf{L}$: Select $v_1$.
* $\mathbf{R}$: Select $v_2$.
* $\mathbf{O}$: Select $r$.

---

### Example 3: Addition and Constants ("Addition is Free")
**Equation:** $z = x \times y + 2$
**Rearranged:** $z - 2 = x \times y$

**Witness:** $\mathbf{w} = [1, z, x, y]$

**The Matrices:**
* $\mathbf{L}$: Select $x$ ($[0, 0, 1, 0]$).
* $\mathbf{R}$: Select $y$ ($[0, 0, 0, 1]$).
* **$\mathbf{O}$**: This must represent $z - 2$.
    * Coefficient for $z$ (Index 1) is $1$.
    * Coefficient for "Constant 1" (Index 0) is $-2$.
    * Vector: $[-2, 1, 0, 0]$.

**Key Insight:** We didn't create a new row for addition. We just combined coefficients in the existing row. This is why "addition is free" in R1CS.

---

### Example 4: Complex Polynomials
**Equation:** $z = 3x^2y + 5xy - x - 2y + 3$

We must break this down step-by-step to isolate multiplications.
1.  **$v_1 = x \times x$** ($x^2$)
2.  **$v_2 = v_1 \times y$** ($x^2y$)
3.  **Final Linear Combination:**
    We need to represent the remaining logic. We can rearrange the equation to isolate the term $5xy$:
    $$5xy = z - 3v_2 + x + 2y - 3$$
    
    This fits the form $L \times R = O$:
    * **Left:** $5x$ (Row: $[0, 0, 5, 0, \dots]$)
    * **Right:** $y$ (Row: $[0, 0, 0, 1, \dots]$)
    * **Output:** $z - 3v_2 + x + 2y - 3$ (Row combines coefficients for $z, v_2, x, y$ and constant).

---

## ⚡ Special Case: Pure Addition
What if we have a constraint with **no** multiplication, like $z = x + y$?
R1CS *requires* a multiplication.
**Trick:** Multiply by 1.
$$z = (x + y) \times 1$$

* $\mathbf{L}$: Coefficients for $x$ and $y$ ($[0, 0, 1, 1]$).
* $\mathbf{R}$: Select the constant 1 ($[1, 0, 0, 0]$).
* $\mathbf{O}$: Select $z$ ($[0, 1, 0, 0]$).

---

## 📝 Summary of Transformation Rules

| Algebra | R1CS Strategy | Cost |
| :--- | :--- | :--- |
| **$A \times B = C$** | Standard row. | 1 Constraint |
| **$A \times B \times C = D$** | Intermediate $v_1 = A \times B$. | 2 Constraints |
| **$A + B = C$** | $(A + B) \times 1 = C$. | 1 Constraint |
| **$C = A \times B + 5$** | Combine const in $\mathbf{O}$ matrix. | 1 Constraint |
| **$C = 5A$** | $(5A) \times 1 = C$ (or fold into other math). | 1 Constraint |

## 💻 Python Verification Code
```python
import numpy as np

# Example: r = x * y
# Witness w = [1, r, x, y]
w = np.array([1, 12, 3, 4]) 

# Define Matrices
L = np.matrix([[0, 0, 1, 0]]) # x
R = np.matrix([[0, 0, 0, 1]]) # y
O = np.matrix([[0, 1, 0, 0]]) # r

# Verify
lhs = np.matmul(L, w)
rhs = np.matmul(R, w)
output = np.matmul(O, w)

# Element-wise multiplication check
assert output == lhs * rhs