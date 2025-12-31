# Notes on "Evaluating a Quadratic Arithmetic Program on a Trusted Setup" (Elliptic Curve QAPs)

## Overview
This chapter explains how to evaluate a Quadratic Arithmetic Program (QAP) using a trusted setup, allowing a prover to demonstrate satisfaction of the QAP without revealing the witness. The proof is constant-sized, but the scheme is not yet a full zero-knowledge (ZK) proof—it's a stepping stone to Groth16.

Key QAP Equation (Evaluated at Unknown Point $\tau$):
$$
\left( \sum_{i=1}^m a_i u_i(x) \right) \left( \sum_{i=1}^m b_i v_i(x) \right) = \sum_{i=1}^m c_i w_i(x) + h(x) t(x)
$$
- If witness vector $\mathbf{a}$ satisfies the R1CS, the equation balances at $x = \tau$.
- Otherwise, it is unbalanced with overwhelming probability due to the randomness of $\tau$.

**Notation Preliminaries:**
- Elliptic curve groups: $\mathcal{G}_1$ and $\mathcal{G}_2$ with generators $G_1$ and $G_2$.
- Elements: $[X]_1 \in \mathcal{G}_1$, $[X]_2 \in \mathcal{G}_2$.
- Pairing: $[X]_1 \cdot [Y]_2$.
- $t(x) = (x-1)(x-2)\cdots(x-n)$, where $n =$ number of R1CS rows (degree $n$).
- Polynomials $u_i(x)$, $v_i(x)$, $w_i(x)$ from Lagrange interpolation on R1CS matrices $A$, $B$, $C$ columns.

## Concrete Example
Consider an R1CS with 3 rows ($n=3$, polynomials degree $\leq 2$) and 4 columns ($m=4$, 12 polynomials total).

QAP for Example:
$$
\left( \sum_{i=1}^4 a_i u_i(x) \right) \left( \sum_{i=1}^4 a_i v_i(x) \right) = \sum_{i=1}^4 a_i w_i(x) + h(x) t(x)
$$
- $t(x) = (x-1)(x-2)(x-3)$ (degree 3).
- $h(x) = \frac{\text{left} - \text{right}}{t(x)}$ (degree $\leq 1$).

**Lagrange Interpolation Polynomials:**
For $A$ matrix columns $j=1$ to $4$, interpolate over $x=\{1,2,3\}$:
$$
u_j(x) = u_{j2} x^2 + u_{j1} x + u_{j0} = \mathcal{L}(\mathbf{A}_{(*,j)})
$$
Similarly for $v_j(x)$ from $B$ and $w_j(x)$ from $C$.

**Polynomial Degrees in General R1CS ($n$ rows, $m$ columns):**
- $\deg(u(x))$, $\deg(v(x)) \leq n-1$ (interpolate $n$ points).
- $\deg(w(x)) \geq 0$ (could be 0 if coefficients cancel).
- $\deg(t(x)) = n$.
- $\deg(h(x)) \leq n-2$, since $\deg(\text{left}) \leq 2(n-1)$, $\deg(\text{right} - h t)$ balances, so $\deg(h t) \leq 2(n-1) \implies \deg(h) + n \leq 2n-2 \implies \deg(h) \leq n-2$.

## Expanding the QAP Terms
For the example (degree 2 polynomials):
$$
\sum_{i=1}^4 a_i u_i(x) = u_{2a} x^2 + u_{1a} x + u_{0a}
$$
where $u_{ka} = \sum_{i=1}^4 a_i u_{i k}$ ($k=0,1,2$).

Similarly:
$$
\sum_{i=1}^4 a_i v_i(x) = v_{2a} x^2 + v_{1a} x + v_{0a}, \quad \sum_{i=1}^4 a_i w_i(x) = w_{2a} x^2 + w_{1a} x + w_{0a}
$$
In general, $\sum a_i p_i(x)$ yields a polynomial of degree $\leq \max \deg(p_i)$.

## Combining with Trusted Setup (Structured Reference String - SRS)
SRS from Powers of Tau Ceremony ($\tau$ destroyed post-setup):
- In $\mathcal{G}_1$: $[\Omega_2, \Omega_1, G_1] = [\tau^2 G_1, \tau G_1, G_1]$ (extend to $[\Omega_{n-1}, \dots, G_1] = [\tau^{n-1} G_1, \dots, G_1]$).
- In $\mathcal{G}_2$: $[\Theta_2, \Theta_1, G_2] = [\tau^2 G_2, \tau G_2, G_2]$ (extend similarly).

**Prover Evaluations (Without Knowing $\tau$):**
$$
[A]_1 = \sum a_i u_i(\tau) = \langle [u_{2a}, u_{1a}, u_{0a}], [\Omega_2, \Omega_1, G_1] \rangle \in \mathcal{G}_1
$$
$$
[B]_2 = \sum a_i v_i(\tau) = \langle [v_{2a}, v_{1a}, v_{0a}], [\Theta_2, \Theta_1, G_2] \rangle \in \mathcal{G}_2
$$
$$
[C']_1 = \sum a_i w_i(\tau) = \langle [w_{2a}, w_{1a}, w_{0a}], [\Omega_2, \Omega_1, G_1] \rangle \in \mathcal{G}_1
$$
QAP becomes:
$$
[A]_1 \cdot [B]_2 = [C']_1 + [h(\tau) t(\tau)]_1
$$
(But $[h(\tau) t(\tau)]_1$ still needs computation.)

## Computing $h(x) t(x)$
$h(x) = \frac{\sum a_i u_i(x) \sum a_i v_i(x) - \sum a_i w_i(x)}{t(x)}$ (degree $\leq n-2$).

Direct eval at $\tau$ impossible without $\tau$. Instead:
- $t(\tau)$ scalar (known in setup).
- $h(x) t(\tau)$ is polynomial degree $\leq n-2$: $\sum_{i=0}^{n-2} h_i x^i \cdot t(\tau)$.
- Eval at $\tau$: $h(\tau) t(\tau) = \sum h_i \tau^i \cdot t(\tau)$.

**Extended SRS for $h t$:**
Add to $\mathcal{G}_1$: $[\Upsilon_{n-2}, \dots, \Upsilon_0] = [\tau^{n-2} t(\tau) G_1, \dots, t(\tau) G_1]$.

Prover computes:
$$
[H]_1 = h(\tau) t(\tau) = \langle [h_{n-2}, \dots, h_0], [\Upsilon_{n-2}, \dots, \Upsilon_0] \rangle \in \mathcal{G}_1
$$
Then:
$$
[C]_1 = [C']_1 + [H]_1
$$

**Why It Works:**
$$
\langle [h_{n-2}, \dots, h_0], [\Upsilon_{n-2}, \dots, \Upsilon_0] \rangle = \sum_{i=0}^{n-2} h_i (\tau^i t(\tau) G_1) = t(\tau) G_1 \sum h_i \tau^i = [t(\tau) h(\tau)]_1
$$

**Full SRS Summary:**
- $\mathcal{G}_1$: $[\Omega_{n-1}, \dots, \Omega_0 = G_1]$ for $u/w$ polys (degree $n-1$).
- $\mathcal{G}_2$: $[\Theta_{n-1}, \dots, \Theta_0 = G_2]$ for $v$ polys.
- $\mathcal{G}_1$: $[\Upsilon_{n-2}, \dots, \Upsilon_0]$ for $h t$ (degree $n-1$ total).

## Full QAP Evaluation Protocol
**Prover Steps:**
1. Compute coefficients of $\sum a_i u_i(x)$, $\sum a_i v_i(x)$, $\sum a_i w_i(x)$ (degree $\leq n-1$).
2. Compute $h(x)$ coefficients (divide poly difference by $t(x)$).
3. $[A]_1 =$ inner product with $\Omega$ vector.
4. $[B]_2 =$ inner product with $\Theta$ vector.
5. $[C]_1 =$ inner product $w$ with $\Omega +$ inner product $h$ with $\Upsilon$.
6. Proof $\pi = ([A]_1, [B]_2, [C]_1)$.

**Verifier Check:**
$$
[A]_1 \cdot [B]_2 \stackrel{?}{=} [C]_1 \cdot G_2
$$
- Holds if QAP satisfied (pairing linearity).
- Fails overwhelmingly otherwise (pairing security).

**Caveat:** Not ZK—verifier can't confirm prover used valid $\mathbf{a}$ (prover could fake points). No soundness against cheating without commitments.

## Proof Size and Efficiency
- $[A]_1$, $[C]_1$: 64 bytes each ($\mathcal{G}_1$, 256-bit curve).
- $[B]_2$: 128 bytes ($\mathcal{G}_2$).
- Total: 256 bytes (constant, independent of $n$, $m$).

- Prover time: $O(n m + n^2)$ (interpolation, poly ops).
- Verifier time: $O(1)$ (one pairing).

## Limitations and Next Steps
- **Not ZK:** Reveals nothing about $\mathbf{a}$, but no guarantee of correct computation.
- **Not Sound:** Prover can cheat with invalid points.
- Foundation for **Groth16:** Add Pedersen commitments to inputs, knowledge-of-exponent assumptions for soundness/ZK.

**Key Insights:**
- Trusted setup hides $\tau$, enables efficient poly eval via inner products.
- $h t$ term uses shifted powers ($\Upsilon$) to handle division by $t(x)$.
- Constant proof size from bilinear pairings and low-degree structure.