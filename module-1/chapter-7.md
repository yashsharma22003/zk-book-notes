# Homomorphisms

**Source:** [rareskills.io/post/homomorphisms](https://rareskills.io/post/homomorphisms)

## 🦁 What is a Homomorphism?

A **Homomorphism** is a map between two algebraic structures (like two Groups) that **preserves the structure** of the operation.

Given:
* Group $A$ with binary operator $\square$.
* Group $B$ with binary operator $\triangle$.

A function $\phi: A \to B$ is a homomorphism if for all $x, y \in A$:
$$\phi(x \square y) = \phi(x) \triangle \phi(y)$$

**In simple terms:** It doesn't matter if you perform the operation *before* or *after* the mapping; the result is the same.
* *Analogy:* Translating a sentence from English to Spanish. If the translation is perfect, the meaning of "The cat" + "sat" should be the same as "El gato" + "se sentó".

---

## 🔎 Examples

### 1. The Classic: Exponents (Logarithms)
* **Group A:** Real numbers under Addition ($\mathbb{R}, +$).
* **Group B:** Positive Real numbers under Multiplication ($\mathbb{R}^+, \times$).
* **Map:** $\phi(x) = e^x$.
* **Check:**
    * $\phi(x + y) = e^{x+y}$
    * $\phi(x) \times \phi(y) = e^x \times e^y = e^{x+y}$
    * **Result:** It holds! This is why logarithms transform multiplication into addition.

### 2. String Length
* **Group A:** Strings under Concatenation ("foo" + "bar").
* **Group B:** Integers under Addition ($3 + 3$).
* **Map:** $\phi(s) = \text{len}(s)$.
* **Check:**
    * $\text{len}(\text{"foo"} + \text{"bar"}) = \text{len}(\text{"foobar"}) = 6$
    * $\text{len}(\text{"foo"}) + \text{len}(\text{"bar"}) = 3 + 3 = 6$

### 3. Even Numbers
* **Group A:** Integers ($\mathbb{Z}$).
* **Group B:** Even Integers ($2\mathbb{Z}$).
* **Map:** $\phi(x) = 2x$.
* **Check:**
    * $\phi(x + y) = 2(x + y) = 2x + 2y$
    * $\phi(x) + \phi(y) = 2x + 2y$

---

## 🔐 Homomorphic Encryption & ZK
This concept is the "magic" behind Zero Knowledge Proofs and Homomorphic Encryption.

If the map $\phi$ is **easy to compute** but **hard to invert** (like Discrete Logarithms in Elliptic Curves), we can prove things about secret numbers without revealing them.

### Example: Proving Addition
* **Prover** has secrets $a$ and $b$.
* **Verifier** wants to know if the prover computed $c = a + b$ correctly, but Prover refuses to show $a$ or $b$.
* **Homomorphism:** $\phi(x) = g^x \pmod p$.
* **Protocol:**
    1.  Prover sends $\phi(a)$ and $\phi(b)$ and $\phi(c)$.
    2.  Verifier checks: $\phi(c) \stackrel{?}{=} \phi(a) \times \phi(b)$.
    3.  Because of the homomorphism ($g^{a+b} = g^a \times g^b$), the math holds.

---

## 🧠 Why Programmers Should Care
The article's main point is to prepare you for this statement about **Elliptic Curves**:
> "Elliptic curve points in a finite field under addition are a finite cyclic group, and **integers under addition are homomorphic to this group**."

This means even if Elliptic Curve math looks weird, you can treat it just like "adding integers" but with a different notation. You already know how it behaves because you understand homomorphisms.. 