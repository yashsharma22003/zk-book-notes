# Evaluating QAP on a Trusted Setup

**Source:** [RareSkills](https://rareskills.io/post/elliptic-curve-qap)

## 🦁 The Goal
We want to prove that we know a witness that satisfies a **QAP** (Quadratic Arithmetic Program) without revealing the witness.
* **Previous Step:** We converted R1CS matrices into polynomials $A(x), B(x), C(x)$.
* **This Step:** We evaluate these polynomials at a secret point $\tau$ using the **Trusted Setup (SRS)** and check the result using **Bilinear Pairings**.

**Key Feature:** The proof size is **constant** (just a few Elliptic Curve points), regardless of how large the circuit is (millions of constraints).

---

## 🛠 The Setup (SRS)
The Trusted Setup must provide the "encrypted" powers of the secret $\tau$ so the prover can evaluate their polynomials blindly.

### 1. Standard Powers (for A, B, C)
The prover needs to compute $A(\tau), B(\tau), C(\tau)$. Since these are constructed from the witness and Lagrange polynomials, the prover needs:
$$[G_1, \tau G_1, \tau^2 G_1, \dots, \tau^n G_1]$$
$$[G_2, \tau G_2, \tau^2 G_2, \dots, \tau^n G_2]$$

### 2. Special Powers (for H)
The prover also needs to compute the term $H(\tau)Z(\tau)$.
* $Z(x)$ is the **Target Polynomial** (vanishing at all roots), known publicly at setup time.
* $H(x)$ is the **Quotient Polynomial**, computed by the prover ($P(x) / Z(x)$).

To evaluate $H(\tau)Z(\tau)$, we don't calculate $H(\tau)$ and $Z(\tau)$ separately (that would require a pairing multiplication, which lands us in a group we can't use easily).
Instead, the Trusted Setup pre-computes $Z(\tau)$ multiplied by powers of $\tau$:

**The H-SRS:**
$$[Z(\tau)G_1, \tau Z(\tau)G_1, \tau^2 Z(\tau)G_1, \dots, \tau^{n-2} Z(\tau)G_1]$$

This allows the Prover to compute $[H(\tau)Z(\tau)]_1$ directly by taking the inner product of the coefficients of $H(x)$ with this special SRS.

---

## 🎭 The Prover's Work
The prover has the witness $\mathbf{w}$ and the QAP polynomials.

1.  **Compute Aggregate Polynomials:**
    Combine the witness with the column polynomials to get $A(x), B(x), C(x)$.
    $$A(x) = \sum w_i L_i(x)$$
2.  **Compute Quotient $H(x)$:**
    $$P(x) = A(x)B(x) - C(x)$$
    $$H(x) = P(x) / Z(x)$$
3.  **Evaluate on Curve (Encryption):**
    Using the SRS, calculate the EC points:
    * $[A]_1 = Evaluate(A(x), SRS_{G1})$
    * $[B]_2 = Evaluate(B(x), SRS_{G2})$
    * $[C]_1 = Evaluate(C(x), SRS_{G1})$
    * $[HZ]_1 = Evaluate(H(x), SRS_{H})$

---

## 🕵️ The Verifier's Check
The prover sends the points: $[A]_1, [B]_2, [C]_1, [HZ]_1$.
The verifier checks if the polynomial identity holds:
$$A(\tau)B(\tau) - C(\tau) = H(\tau)Z(\tau)$$

Using **Bilinear Pairings**, this check becomes:
$$e([A]_1, [B]_2) \stackrel{?}{=} e([C]_1, G_2) + e([HZ]_1, G_2)$$
*(Note: Depending on implementation, $[C]$ and $[HZ]$ might be combined into a single point to save space).*

---

## 📏 Constant Proof Size
Notice what happened here:
* If the circuit has **1 constraint**, the proof is ~3-4 points.
* If the circuit has **1,000,000 constraints**, the proof is **still ~3-4 points**.

The complexity is shifted to the **Prover** (who has to do 1M evaluations) and the **Setup** (who generates a large SRS), but the **Verifier** always does the same constant number of pairings.

---

## ⚠️ Security Note
This specific scheme is **not yet Zero Knowledge**.
* The prover is hiding the *values* (witness) by encrypting them on the curve.
* However, the prover is providing definite points $[A], [B]$. In a real ZK-SNARK (like Groth16), we add "random shifts" (multiplying by random scalars $\delta, \alpha, \beta$) to these points so that they look completely random to an observer, while still satisfying the pairing equation.