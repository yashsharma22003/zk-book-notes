# Elliptic Curve Point Addition

**Source:** [rareskills.io/post/elliptic-curve-addition](https://rareskills.io/post/elliptic-curve-addition)

## 📉 1. The Set Theoretic View
While we visualize Elliptic Curves as lines, they are formally defined as a **Set** of points $(x, y)$ that satisfy the equation:
$$y^2 = x^3 + ax + b$$

* **The Group:** The set of valid points, plus a special "Point at Infinity" ($O$).
* **The Operator ($\oplus$):** A custom binary operator that takes two points in the set and produces a third.
    * *Note:* The article uses $\oplus$ instead of $+$ to emphasize this is not standard coordinate addition.

---

## 📐 2. Geometric Addition (Chord-and-Tangent)

We define the operator $\oplus$ using geometry:

### Addition ($A \oplus B = C$)
1.  **Draw Line:** Connect $A$ and $B$.
2.  **Intersect:** The line hits the curve at a 3rd point.
3.  **Reflect:** Flip that point over the X-axis to get $C$.

### Doubling ($A \oplus A = 2A$)

1.  **Tangent:** Draw the tangent line at $A$.
2.  **Intersect:** The line hits the curve at a 3rd point.
3.  **Reflect:** Flip over the X-axis.

### The Identity ($O$)
* **Vertical Lines:** If $A = (x, y)$ and $B = (x, -y)$, the line is vertical.
* **Definition:** Vertical lines intersect at the "Point at Infinity" ($O$).
* **Inverse:** $(x, -y)$ is the inverse of $(x, y)$.

---

## 🦁 3. Group Properties (Abelian)
The set of points under $\oplus$ forms an **Abelian Group**.

1.  **Closure:** Any operation results in a point *on the curve* (or $O$).
2.  **Identity:** $P \oplus O = P$.
3.  **Inverse:** $P \oplus (-P) = O$.
4.  **Commutativity:** $A \oplus B = B \oplus A$ (Obvious geometrically; the line $AB$ is the same as line $BA$).
5.  **Associativity:** $(A \oplus B) \oplus C = A \oplus (B \oplus C)$.
    * *Note:* Proving this algebraically is messy. The article demonstrates this using a Computer Algebra System (SageMath) rather than solving it by hand.

---

## 🧮 4. Algebraic Formulas
To compute this efficiently (without drawing lines), we use slope formulas.

### Slope ($\lambda$)
* **Adding distinct points:** $\lambda = \frac{y_2 - y_1}{x_2 - x_1}$
* **Doubling a point:** $\lambda = \frac{3x_1^2 + a}{2y_1}$ (Derivative)

### New Point $(x_3, y_3)$
$$x_3 = \lambda^2 - x_1 - x_2$$
$$y_3 = \lambda(x_1 - x_3) - y_1$$

---

## ⚡ 5. Scalar Multiplication
In crypto, we don't just add random points. We add a point $G$ to itself $n$ times.
$$n \cdot P = \underbrace{P \oplus P \oplus \dots \oplus P}_{n \text{ times}}$$

### The "Double-and-Add" Shortcut
We can compute $1000P$ in just ~14 operations (vs 1000) using the binary representation of 1000.
* Calculate $2P, 4P, 8P, \dots$ (Repeated Doubling).
* Add the necessary powers.

### Distributivity / Homomorphism
Scalar multiplication behaves like standard algebra:
$$(a + b)P = aP \oplus bP$$
* **Why?** This isn't a new "rule." It just means adding $P$ to itself $(a+b)$ times is the same as adding it $a$ times, then $b$ times, and combining the results.
* *This property allows Elliptic Curves to be used for Homomorphic Encryption and ZK proofs.*