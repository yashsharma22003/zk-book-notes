# Elliptic Curve Point Addition

**Source:** [rareskills.io/post/elliptic-curve-addition](https://rareskills.io/post/elliptic-curve-addition)

## 📉 Introduction
Elliptic Curves are often taught with complex algebra, but they are easier to understand as a **Group**.
* **Equation:** $y^2 = x^3 + ax + b$
* **The Set:** All $(x, y)$ points that satisfy the equation, plus a special "Point at Infinity" ($O$).
* **The Goal:** Define a binary operator ("addition") that takes two points on the curve and produces a third point on the curve.

---

## 📐 Geometric Addition (The "Chord-and-Tangent" Rule)

How do we "add" two points $P$ and $Q$ to get $R$?

### Case 1: $P \neq Q$ (Distinct Points)
1.  Draw a straight line through $P$ and $Q$.
2.  The line will intersect the curve at a **third point** (let's call it $-R$).
3.  Reflect this point across the X-axis (flip the Y-coordinate) to get the final result $R$.
    * **Visual:** Draw line $\to$ Find intersection $\to$ Flip vertical.
    * **Formula:** $P + Q = R$.

### Case 2: $P = Q$ (Point Doubling)
1.  Since we can't draw a line through one point, we use the **tangent line** at $P$.
2.  The tangent line intersects the curve at exactly one other point ($-2P$).
3.  Reflect this point across the X-axis to get $2P$.

### Case 3: Vertical Line
If $P$ and $Q$ are vertical opposites (e.g., $(x, y)$ and $(x, -y)$):
1.  The line is vertical and never intersects the curve again in the Euclidean plane.
2.  **Definition:** The intersection is defined as the **Point at Infinity** ($O$).
3.  **Result:** $P + (-P) = O$ (The Identity).

---

## 🧮 Algebraic Formulas

We can derive exact formulas for the coordinates of the new point $P_3 = (x_3, y_3)$ given $P_1$ and $P_2$.

### 1. Calculate Slope ($\lambda$)
* **If Adding ($P_1 \neq P_2$):**
  $$\lambda = \frac{y_2 - y_1}{x_2 - x_1}$$
* **If Doubling ($P_1 = P_2$):** (Derived from derivative $dy/dx$)
  $$\lambda = \frac{3x_1^2 + a}{2y_1}$$

### 2. Calculate New Intersection ($x_3, y_3$)
Once you have the slope $\lambda$:
$$x_3 = \lambda^2 - x_1 - x_2$$
$$y_3 = \lambda(x_1 - x_3) - y_1$$

---

## 🦁 Group Properties

The set of points on an elliptic curve forms an **Abelian Group**.

1.  **Closure:** Adding any two points results in a valid third point (or Infinity).
2.  **Associativity:** $(A + B) + C = A + (B + C)$. *Note: This is hard to prove algebraically but geometrically consistent.*
3.  **Identity:** The "Point at Infinity" ($O$).
    * $P + O = P$
    * Visual: A vertical line through $P$ "hits" infinity, reflecting back to $P$.
4.  **Inverse:** For any $P(x, y)$, the inverse is $-P(x, -y)$.
    * $P + (-P) = O$.
5.  **Commutativity (Abelian):** $A + B = B + A$.
    * Visual: The line through $A$ and $B$ is the same as the line through $B$ and $A$.

---

## ⚡ Scalar Multiplication (The "Double-and-Add" Trick)
In cryptography, we often need to compute $k \cdot P$ (adding $P$ to itself $k$ times), where $k$ is huge.
* **Naive way:** $P + P + P \dots$ ($k$ times). Too slow.
* **Efficient way:** **Double and Add**.
    * To compute $1000P$:
    * $1000P = 512P + 256P + 128P + 64P + 32P + 8P$.
    * You only need to "Double" $P$ a few times to get these powers of 2, then add them up.
    * Complexity drops from $O(k)$ to $O(\log k)$.

---

## ⚠️ Real vs. Finite Fields
* **Real Numbers:** Good for visualization (continuous curves), but bad for computers (floating point errors, slow).
* **Finite Fields:** In ZK and Crypto, we do this all **modulo $p$**.
    * The geometry is less intuitive (scattered points), but the **algebraic formulas remain exactly the same**.