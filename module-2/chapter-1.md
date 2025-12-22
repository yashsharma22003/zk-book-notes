# Bilinear Pairings in Python, Solidity, and the EVM

**Source:** [RareSkills](https://rareskills.io/zk-book)

## 🦁 High-Level Concept
Bilinear Pairings (or mappings) are the cryptographic "magic" that allow us to verify relationships between numbers without revealing the numbers themselves.

* **Goal:** Prove that $a \times b = c$ without revealing $a$ or $b$.
* **Mechanism:** We take encrypted values (Elliptic Curve points $A=aG$, $B=bG$) and use a pairing function to check if their "product" matches the encrypted result $C=cG$.
* **Application:** This is the core engine behind **ZK-SNARKs** (Groth16, Tornado Cash) and **BLS Signatures**.

---

## 📐 Mathematical Theory

### The Function Definition
A bilinear pairing is a function $e$ that maps two points from source groups ($G_1$ and $G_2$) to a target group ($G_T$).

$$e: G_1 \times G_2 \rightarrow G_T$$

### Bilinearity Property
The defining property is that the function preserves the linear relationship of scalars "inside" the encryption.
If $P \in G_1, Q \in G_2$ and $a, b$ are scalars:

$$e(aP, bQ) = e(P, Q)^{ab}$$

This implies we can "pull out" the scalars. If $P$ and $Q$ are generators ($G_1, G_2$), then:
$$e(aG_1, bG_2) \rightarrow \text{Output represents } a \times b$$

### Symmetric vs. Asymmetric
* **Symmetric Pairing:** Inputs come from the same group ($G_1 \times G_1$).
* **Asymmetric Pairing:** Inputs come from distinct groups ($G_1 \times G_2$). **Ethereum uses this.**
    * **G1:** Standard Elliptic Curve points $(x, y)$. Small and cheap.
    * **G2:** Points over a **Field Extension** (Complex numbers). Coordinates are pairs $((x_1, x_2), (y_1, y_2))$. Larger and more expensive.

---

## 🐍 Python Implementation (`py_ecc`)

We use the Ethereum Foundation's `py_ecc` library to perform pairings on the BN128 curve.

### Verifying Multiplication ($3 \times 8 = 24$)
We encrypt the scalars 3, 8, and 24 into points and check if the pairing holds.

```python
from py_ecc.bn128 import G1, G2, pairing, multiply, eq

# Prover computes encrypted values
P = multiply(G1, 3)   # Encrypted 3
Q = multiply(G2, 8)   # Encrypted 8
R = multiply(G1, 24)  # Encrypted 24

# Verifier checks: e(Q, P) == e(G2, R)
# Note: py_ecc often expects G2 as the first argument
assert eq(pairing(Q, P), pairing(G2, R))