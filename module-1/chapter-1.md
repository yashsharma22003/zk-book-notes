# P vs NP and its Application to Zero Knowledge Proofs

**Source:** [rareskills.io/post/p-vs-np](https://rareskills.io/post/p-vs-np)

## 🧠 1. Complexity Classes

The core of Zero Knowledge Proofs (ZKPs) relies on the distinction between *computing* a solution and *verifying* a solution.

### **P (Polynomial Time)**
Problems that are both **easy to solve** and **easy to verify**.
* **Definition:** Solvable in $O(n^k)$ time.
* **Examples:**
    * **Sorting a list:** Easy to sort, easy to check if sorted.
    * **Graph Connectivity:** Easy to find if two nodes are connected (BFS), easy to verify a path.
    * **Index Search:** Finding an item in a list (if location is known).
* **ZK Context:** ZK proofs are generally **unnecessary** for P problems because the verifier can just compute the solution themselves efficiently.

### **NP (Nondeterministic Polynomial Time)**
Problems that are **hard to solve** (often exponential time) but **easy to verify**.
* **Definition:** If given a "witness" (solution), verification takes polynomial time.
* **Examples:**
    * **Sudoku:** Hard to solve (exponential search), but checking a completed grid is instant.
    * **3-Coloring:** Hard to find a valid coloring for a map, but easy to check that no neighbors share a color.
    * **Boolean Satisfiability:** Hard to find inputs that make a formula `True`, easy to plug them in and check.
* **ZK Context:** This is the domain of ZKPs. A prover can convince a verifier they have the "witness" (solution) without revealing it.

### **PSPACE**
Problems that are **hard to solve** and **hard to verify**.
* **Definition:** Solvable with polynomial *memory*, but potentially infinite *time*. Verification is not efficient.
* **Examples:**
    * **Optimal Chess Move:** To verify a move is truly "optimal," one must simulate the entire game tree (exponential time).
    * **Regex Equivalence:** Checking if two regular expressions match the exact same set of strings.
* **ZK Context:** ZKPs are **not feasible** for PSPACE problems because the verifier cannot efficiently check the proof.

---

## 🔑 2. The Witness
A **witness** is the data that proves the solution is correct.
* **In P:** The witness is often just the solution itself (e.g., the sorted list).
* **In NP:** The witness is the secret data (e.g., the filled Sudoku grid) that the prover claims to possess.
* **Note:** A witness doesn't always have to be the direct solution; it can be a solution to an equivalent representation of the problem.

---

## 🔌 3. Boolean Formulas (The Common Language)
To create a general ZK system, we need a standard way to represent *any* NP problem.
* **Cook-Levin Theorem:** Any NP problem (Sudoku, Coloring, etc.) can be translated into a **Boolean Formula**.
* **Structure:** Uses `AND` ($\land$), `OR` ($\lor$), and `NOT` ($\neg$) gates.
* **The Task:** Find input variables ($x_1, x_2, ...$) such that the final output is `True`.
* **ZK Application:** instead of building a specific ZK protocol for Sudoku and another for 3-Coloring, we translate both to a Boolean Circuit (or Arithmetic Circuit) and use a standard proof system for that circuit.

---

## ⚠️ 4. Technical Nuances
* **Chess Complexity:** Chess is technically **PSPACE** (due to the 50-move rule preventing infinite games). Without that rule, it would be **EXPSPACE** (exponential time and memory).
* **Heuristics:** While NP problems are theoretically "hard" (exponential) in the worst case, heuristics often solve average cases quickly. However, "pathological" cases always exist where the heuristic fails.
* **Assumption:** We assume $P \neq NP$ and $NP \neq PSPACE$. If $P = NP$, modern cryptography (and ZKPs) would break because keys could be derived efficiently.