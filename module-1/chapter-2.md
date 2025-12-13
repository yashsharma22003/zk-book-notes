# Arithmetic Circuits

**Source:** [rareskills.io/post/arithmetic-circuit](https://rareskills.io/post/arithmetic-circuit)

## ⚡ What is an Arithmetic Circuit?
An arithmetic circuit models a computation using numbers (signals) and math operations rather than bits and logical gates. It is the standard format for representing NP problems in Zero-Knowledge proofs.

| Feature | Boolean Circuit | Arithmetic Circuit |
| :--- | :--- | :--- |
| **Variables** | Bits (0, 1) | Signals (Numbers) |
| **Operations** | AND, OR, XOR, NOT | Add ($+$), Multiply ($\times$) |
| **Satisfaction** | Output is `True` | All equations hold (LHS = RHS) |
| **Witness** | Input bits | Assignment of values to signals |

## 📝 Notation & Syntax
* **Signals:** The variables in the system. All signals are treated as inputs.
* **Constraint (`===`):** Represents an assertion of equality, similar to `assertEq(lhs, rhs)`.
    * *Example:* `c === a + b` does not "assign" `a+b` to `c`. It asserts that the prover must provide `a`, `b`, and `c` such that the equation holds.
* **Satisfaction:** A circuit is "satisfied" if there is a valid assignment of signals (the witness) that makes *every* constraint true.

## 🛠 Programming Logic with Math
Since we only have `+` and `*`, we use polynomial properties to enforce logical rules ("gadgets").

### 1. Binary Constraint
To force a signal `x` to be either 0 or 1:
$$x(x - 1) === 0$$
* **Logic:** The only roots for $x^2 - x = 0$ are 0 and 1.

### 2. Logic Gates
* **AND:** `out === x * y`
* **OR:** `out === x + y - (x * y)` (De Morgan's Laws)
* **NOT:** `out === 1 - x`

### 3. Bit Decomposition (Range Check)
Arithmetic circuits handle large numbers. To restrict a number $v$ to a specific range (e.g., 4 bits, $v < 16$), we decompose it into binary signals $b_0, b_1, b_2, b_3$.
1.  **Constrain bits:** Ensure every $b_i$ is binary: $b_i(b_i - 1) === 0$.
2.  **Reconstruct sum:**
    $$v === 8b_3 + 4b_2 + 2b_1 + 1b_0$$
*This proves $v$ is a valid 4-bit number.*

---

## 🇦🇺 Case Study: 3-Coloring Australia
**Problem:** Prove a graph is 3-colored (Values 1, 2, 3) such that no connected territories share a value.

### Step 1: Color Constraints
For every territory $T$, enforce it is exactly 1, 2, or 3:
$$(1 - T)(2 - T)(3 - T) === 0$$

### Step 2: Neighbor Constraints (Edge Check)
For neighbors $A$ and $B$, their product must not imply they are the same value.
* **Valid Products:** If $A, B \in \{1, 2, 3\}$ and $A \neq B$, their product $AB$ must be $\{2, 3, 6\}$.
    * $1 \times 2 = 2$
    * $1 \times 3 = 3$
    * $2 \times 3 = 6$
* **Invalid Products:** $1 \times 1 = 1$, $2 \times 2 = 4$, $3 \times 3 = 9$.
* **Constraint:**
    $$(2 - AB)(3 - AB)(6 - AB) === 0$$
*This equation is only satisfied if $A$ and $B$ have different colors.*

---

## 🧩 Other Common Patterns

### Subset Sum Problem
**Goal:** Given a set of numbers $\{n_1, n_2, ...\}$, prove a subset sums to $Target$.
* **Technique:** Create a binary "switch" $s_i$ for each number.
* **Constraint:**
    $$Target === s_1 n_1 + s_2 n_2 + \dots$$
    *(Where every $s_i$ is constrained to be 0 or 1)*.

### "At Least One" Logic
To prove **at least one** signal in $\{x_1, x_2, x_3\}$ is 0:
$$x_1 \times x_2 \times x_3 === 0$$
*(If the product is 0, one of the terms must be 0).*