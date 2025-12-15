# Abstract Algebra for Programmers

**Source:** [rareskills.io/post/abstract-algebra](https://rareskills.io/post/abstract-algebra)

## 🦁 Introduction
Abstract Algebra is the study of sets equipped with one or more operators. In the context of Zero Knowledge Proofs (ZK), we primarily care about **sets with a single binary operator**.

Rather than jumping straight to "Groups" (which are central to ZK), it is helpful to understand the hierarchy of algebraic structures based on how restrictive their rules are.

---

## 🏗 The Hierarchy of Algebraic Structures

We can classify a set $S$ with a binary operator $\cdot$ into the following categories, from least restrictive to most restrictive.

### 1. Magma
**Definition:** A set with a **closed binary operator**.
* **Rule:** For all $a, b \in S$, $a \cdot b \in S$.
* **Properties:** No other rules (associativity not required).
* **Example:**
    * **Positive Integers with Exponentiation ($a^b$):**
        * Closed? Yes (Integer^Integer = Integer).
        * Associative? No ($a^{(b^c)} \neq (a^b)^c$).

### 2. Semigroup
**Definition:** A Magma where the operator is **associative**.
* **Rule:** $(a \cdot b) \cdot c = a \cdot (b \cdot c)$.
* **Example:**
    * **Strings with Concatenation:**
        * Set: Non-empty strings.
        * Operator: `+` ("foo" + "bar").
        * Associative? Yes ("a" + "b") + "c" is the same as "a" + ("b" + "c").
        * **Note:** It is *not* a Monoid if the empty string `""` is excluded.

### 3. Monoid
**Definition:** A Semigroup with an **identity element**.
* **Rule:** There exists an element $e \in S$ such that for all $a \in S$, $a \cdot e = a$ and $e \cdot a = a$.
* **Examples:**
    * **Strings (including `""`):** The empty string is the identity.
    * **Integers with Addition:** The identity is $0$.
    * **Integers with Multiplication:** The identity is $1$.
    * **2x2 Matrices:** The identity is the Identity Matrix ($I$).
    * **Set Union:** Identity is the Empty Set ($\emptyset$).

### 4. Group (The "Star of the Show")
**Definition:** A Monoid where every element has an **inverse**.
* **Rule:** For every $a \in S$, there exists $a^{-1} \in S$ such that $a \cdot a^{-1} = e$ (where $e$ is the identity).
* **Examples:**
    * **Integers with Addition ($\mathbb{Z}, +$):**
        * Identity: 0.
        * Inverse: For any $x$, the inverse is $-x$.
    * **Why Strings are NOT a Group:** You cannot "un-concatenate" a string to get back to the empty string. There is no string $s$ such that "foo" + $s$ = "".

### 5. Abelian Group
**Definition:** A Group where the operator is **commutative**.
* **Rule:** $a \cdot b = b \cdot a$.
* **Examples:**
    * **Integers (+):** Abelian ($1+2 = 2+1$).
    * **Matrix Multiplication:** **Not** Abelian ($A \times B \neq B \times A$ generally).

---

## 📊 Summary Table

| Set Domain | Binary Operator | Structure | Reason |
| :--- | :--- | :--- | :--- |
| Non-zero positive integers | Addition | **Semigroup** | Closed & Associative, but no Identity (0). |
| Positive integers (with 0) | Addition | **Monoid** | Has Identity (0), but no Inverses (-numbers). |
| All Integers ($\mathbb{Z}$) | Addition | **Group** | Has Identity (0) and Inverses (Negatives). |

---

## 🧠 Key Takeaways for ZK

1.  **Abstraction Power:** Theorems proven for "Groups" apply to *any* Group (integers, elliptic curves, matrices). You don't need to know the implementation details of the operator, just that it satisfies the Group axioms.
2.  **Domain Sensitivity:** The "Category" of a structure depends heavily on the set domain.
    * *Example:* $\{Integers\} + \{Min(a, b)\}$ is a **Semigroup**.
    * *Example:* $\{Positive Integers\} + \{Min(a, b)\}$ is still a Semigroup (no Identity element acts as "infinity").
    * *Example:* $\{Integers\} + \{Intersection\}$ is a Semigroup, but if you add $\mathbb{Z}$ to the set, it becomes a Monoid (Identity = $\mathbb{Z}$).
3.  **Terminology:**
    * **Closed:** Result stays in the set.
    * **Associative:** Order of operations doesn't matter (parentheses).
    * **Identity:** "Zero" element.
    * **Inverse:** "Negative" element.
    * **Abelian:** Order of operands doesn't matter ($a+b=b+a$). 