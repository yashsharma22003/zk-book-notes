# Elementary Set Theory for Programmers

**Source:** [rareskills.io/post/set-theory](https://rareskills.io/post/set-theory)

## 📦 1. Sets & Relations
A **set** is a collection of distinct, unordered objects.
* **Notation:** $A = \{1, 2, 3\}$.
* **Cardinality:** The number of elements in a set, denoted as $|A|$.
* **Subsets:**
    * $A \subseteq B$: Every element of $A$ is in $B$.
    * **Proper Subset ($A \subset B$):** $A$ is in $B$, but $A \neq B$.

### Standard Number Sets
* $\mathbb{N}$: **Natural Numbers** (0, 1, 2...).
* $\mathbb{Z}$: **Integers** (..., -1, 0, 1...).
* $\mathbb{Q}$: **Rational Numbers** (Fractions).
* $\mathbb{R}$: **Real Numbers**.
* $\mathbb{C}$: **Complex Numbers**.

---

## 🔗 2. Cartesian Products & Tuples
While sets are unordered, we can create **order** using sets of sets.

### Ordered Pairs
* **Definition:** An ordered pair $(a, b)$ is technically defined as the set $\{\{a\}, \{a, b\}\}$.
* **Usage:** Programmers know these as **Tuples**. $(1, 2) \neq (2, 1)$.

### Cartesian Product ($\times$)
The set of all possible ordered pairs between two sets.
$$A \times B = \{(a, b) \mid a \in A, b \in B\}$$

**Example:**
If $A = \{1, 2\}$ and $B = \{x, y\}$:
$$A \times B = \{(1, x), (1, y), (2, x), (2, y)\}$$

---

## 𝑓 3. Functions (The Set Theoretic View)
This is the most critical concept for ZK math.
**A function is not just "code"; it is a subset of a Cartesian Product.**

To define a function $f: A \to B$:
1.  Take the Cartesian Product $A \times B$.
2.  Select a **valid subset** of pairs such that every $a \in A$ maps to exactly one $b \in B$.

* **Example:**
    * Domain $A = \{1, 2, 3\}$, Codomain $B = \{10, 20\}$.
    * Cartesian Product has $3 \times 2 = 6$ pairs.
    * Function Subset: $\{(1, 10), (2, 20), (3, 10)\}$.
    * *Invalid Subset:* $\{(1, 10), (1, 20)\}$ (One input maps to two outputs).

---

## ⚙️ 4. Binary Operators
A **Binary Operator** is a specific type of function used to combine two elements from the same set.

* **Definition:** A function mapping $(S \times S) \to S$.
* **Process:**
    1.  Take every pair of elements in Set $S$ ($S \times S$).
    2.  Map each pair to a result that is *also* in Set $S$.

### Closure (Important)
A binary operator is **closed** if the result is always in the original set.
* **Closed Example:** Addition on Integers ($\mathbb{Z}$). $1 + 2 = 3$ (3 is an integer).
* **Not Closed Example:** Division on Integers. $1 / 2 = 0.5$ (0.5 is not an integer).

**Why this matters for ZK:**
In Elliptic Curve Cryptography and ZK proofs, we rely on **Groups**. A Group is simply a Set combined with a **Closed Binary Operator**. If you understand that operators are just "maps from pairs to single elements," the complex math of elliptic curve addition becomes much easier to grasp conceptually.