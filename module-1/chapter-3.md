# Finite Fields and Modular Arithmetic for ZK Proofs

Notes and implementation details based on the [RareSkills article](https://rareskills.io/post/finite-fields). These concepts are the mathematical foundation for Zero-Knowledge Proofs (ZKPs) and Arithmetic Circuits.

## 🧮 1. What is a Finite Field?
A Finite Field is a set of integers modulo a prime number $p$. Unlike standard computer arithmetic which suffers from overflow and floating-point errors, finite fields wrap around, ensuring **exact precision**.

- **Elements:** $\{0, 1, ..., p-1\}$
- **Order:** The size of the set ($p$).
- **Notation:** $a \equiv b \pmod p$

## ➕ 2. Arithmetic Operations

### Addition & Additive Inverse
Addition wraps around $p$.
- **Additive Inverse:** The value $b$ such that $a + b \equiv 0 \pmod p$.
- **Calculation:** $b = p - a$.
- **Geometric View:** If mapped on a circle, inverses are opposite each other (except 0).

### Multiplication & Division
- **Multiplication:** Standard multiplication modulo $p$.
- **Division:** Defined as multiplication by the **multiplicative inverse**.
  $$a / b \implies a \cdot b^{-1} \pmod p$$
- **Precision:** Division is exact. $1/3$ is an integer in the field, not $0.333...$

### Multiplicative Inverse
For any non-zero $a$, there exists a $b$ such that $a \cdot b \equiv 1 \pmod p$.
- **Fermat's Little Theorem:** Used to compute the inverse.
  $$a^{p-2} \equiv a^{-1} \pmod p$$
- **Rule:** 0 has no inverse. 1 is its own inverse. $p-1$ is its own inverse.

## 🔌 3. ZK Circuit Patterns

### The "Zero Product" Trick
In ZK circuits, we often want to prove that *at least one* of several variables equals a specific value $k$, without revealing which one.
- **Formula:** $(x_1 - k)(x_2 - k)(x_3 - k) = 0$
- **Logic:** If the product is 0, one of the terms *must* be 0 (Field property: $a \cdot b = 0 \implies a=0 \text{ or } b=0$).
- **Implementation:** In a field, we use additive inverses:
  $$(x_1 + \text{inv}(k))(x_2 + \text{inv}(k)) \dots \equiv 0$$

## 📉 4. Systems of Equations & Graphs
Geometry in finite fields is unintuitive compared to real numbers.
- **Intersections:** Parallel lines in real geometry (slope $m_1 = m_2$) might intersect in a finite field if slopes differ modulo $p$. Conversely, intersecting real lines might have no solution in a field.
- **Slopes:** Slopes wrap around. A "positive" slope might wrap to look "negative".

## 📐 5. Polynomials
Polynomials in finite fields ($a_nx^n + \dots + a_0$) behave like standard algebra but coefficients and roots are field elements.
- **Roots:** Values of $x$ where $P(x) \equiv 0$.
- **Visuals:** A polynomial graph is a set of discrete points in the $p \times p$ grid, not a continuous curve.
- **Degree Limit:** A degree $d$ polynomial has at most $d$ roots.

## 6. Advanced Properties

### Square Roots (Quadratic Residues)
- Not all elements have square roots.
- Elements don't need to be "perfect squares" (e.g., 9, 16) to have roots.
  - *Example:* In Mod 11, the square root of $3$ is $5$ (because $5^2 = 25 \equiv 3$).
- If a root exists, there are usually two solutions ($x$ and $p-x$).

### No "Even or Odd"
- Concepts of Even/Odd do not exist because every number is divisible by 2 (multiplied by $2^{-1}$).
- Parity can only be checked if you treat the element as a raw integer (bit check), but this breaks field abstraction.

## 🐍 Python Implementation

### Native Python
```python
p = 7

# Modular Inverse (1/3 mod 7)
inv = pow(3, -1, p) 

# Division (5/6 mod 7)
val = (5 * pow(6, -1, p)) % p