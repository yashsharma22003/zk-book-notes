# Building a Zero Knowledge Proof from an R1CS

**Source:** [RareSkills](https://rareskills.io/post/r1cs-zkp)

## 🦁 Overview
Before diving into complex optimizations like QAP (Quadratic Arithmetic Programs) or Groth16, it is possible to build a "naive" Zero Knowledge Proof directly from the R1CS matrices.

While this method is **inefficient** (the proof size is large and the verifier does a lot of work), it demonstrates the fundamental mechanics of using Elliptic Curve points to hide the values of a computation while still proving the math is correct.

---

## 1. The R1CS Equation Recap
Recall that an R1CS system consists of three matrices ($\mathbf{L}, \mathbf{R}, \mathbf{O}$) and a witness vector $\mathbf{a}$. The system is satisfied if:

$$\mathbf{L}\mathbf{a} \circ \mathbf{R}\mathbf{a} = \mathbf{O}\mathbf{a}$$

* $\mathbf{a}$: The witness vector ($m \times 1$).
* $\circ$: Hadamard Product (element-wise multiplication).
* The result is that for every row $i$: $(\text{Row}_L \cdot \mathbf{a}) \times (\text{Row}_R \cdot \mathbf{a}) = (\text{Row}_O \cdot \mathbf{a})$.

---

## 2. "Encrypting" the Witness
To make this Zero Knowledge, we cannot send the vector $\mathbf{a}$ in plain text. Instead, we "encrypt" the values by mapping them to Elliptic Curve points.

We use **two groups** ($G_1$ and $G_2$) to facilitate Bilinear Pairings later.

* **$G_1$ Witness:** Each element $a_i$ becomes $[a_i G_1]_1$.
* **$G_2$ Witness:** Each element $a_i$ becomes $[a_i G_2]_2$.

*Note: Due to the Discrete Logarithm Problem, no one can recover $a_i$ from $[a_i G_1]_1$.*

---

## 3. The Protocol

### Step A: Prover Computations
The prover performs the matrix multiplication **on the encrypted points**.
Since matrix multiplication is just a series of additions and scalar multiplications, this works on Elliptic Curves (Homomorphic property).

**For Matrix L:**
Instead of computing $\sum l_{i,j} \cdot a_j$ (scalar math), the prover computes:
$$\sum l_{i,j} \cdot [a_j G_1]_1$$
*Result:* A vector of $G_1$ points representing $\mathbf{L}\mathbf{a}$.

**For Matrix R:**
The prover computes $\mathbf{R}\mathbf{a}$ using the $G_2$ encrypted witness:
$$\sum r_{i,j} \cdot [a_j G_2]_2$$
*Result:* A vector of $G_2$ points representing $\mathbf{R}\mathbf{a}$.

**For Matrix O:**
The prover computes $\mathbf{O}\mathbf{a}$ using the $G_1$ witness (similar to $\mathbf{L}$).

### Step B: Verifier Checks (Pairings)
The verifier receives three vectors of points. They need to check if:
$$(\mathbf{L}\mathbf{a}) \times (\mathbf{R}\mathbf{a}) = \mathbf{O}\mathbf{a}$$

Since we cannot multiply points directly, we use **Bilinear Pairings**. For every row $i$, the verifier checks:

$$e(\text{Result}_L[i], \text{Result}_R[i]) \stackrel{?}{=} e(\text{Result}_O[i], G_2)$$

* **Left Side:** Pairs the $L$ result ($G_1$) with the $R$ result ($G_2$). This represents $(L \cdot a) \times (R \cdot a)$ in the target group.
* **Right Side:** Pairs the $O$ result ($G_1$) with the generator $G_2$. This represents $(O \cdot a) \times 1$.

If these match for every row, the constraints are satisfied.

---

## 4. Handling Public Inputs
Usually, a ZK proof proves: "I know *some* secret inputs that match these *public* inputs and result."
* **Example:** Witness $\mathbf{a} = [1, \text{out}, x, \text{secret}]$.
* **Method:** The Prover does **not** encrypt the public elements (1, out, x). They send them as plain scalars.
* **Verifier:** The Verifier encrypts these scalars themselves and adds them to the encrypted vectors provided by the prover.

---

## 5. Security & Limitations

### The Malicious Prover Problem
A prover could cheat by using different values for the $G_1$ witness and the $G_2$ witness (e.g., $a_1=5$ in $G_1$ but $a_1=9$ in $G_2$).
**Fix:** The verifier runs a consistency check using pairings:
$$e([a_i G_1]_1, G_2) \stackrel{?}{=} e(G_1, [a_i G_2]_2)$$
If this holds, the underlying scalar $a_i$ is identical in both groups.

### Witness Guessing
This protocol is not *perfectly* Zero Knowledge. If the witness values are from a small range (e.g., 1 to 10), an attacker can brute-force guess values, encrypt them, and compare them to the prover's points to reveal the secret. (Real protocols like Groth16 add "random shifts" to prevent this).

### Efficiency (Why we don't use this)
* **Verifier Load:** The verifier must compute $O(n)$ pairings, where $n$ is the number of constraints. Pairings are computationally expensive.
* **Proof Size:** The proof size scales linearly with the circuit size.
* **Solution:** This is why we use **QAP (Quadratic Arithmetic Programs)**—it compresses all these constraints into polynomials so the verifier only needs to check *one* equation, regardless of circuit size.

---

## 🧪 Summary Table

| Concept | Implementation |
| :--- | :--- |
| **Witness** | Encrypted as $[a_i G_1]$ and $[a_i G_2]$ |
| **Matrix Math** | Prover does Scalar Mul on Elliptic Curve points |
| **Multiplication Check** | Verifier uses Bilinear Pairing $e(L, R) = e(O, G_2)$ |
| **Consistency** | Verifier checks $e(W_1, G_2) = e(G_1, W_2)$ |
| **Bottleneck** | Verifier does too many pairings (Linear cost) |