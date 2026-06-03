# Deutsch-Jozsa Algorithm — Quantum Computing

A quantum computing project implementing the **Deutsch-Jozsa algorithm** using IBM's **Qiskit** framework, demonstrating an exponential speedup over classical computation by classifying Boolean oracle functions as constant or balanced in a single query.

---

## Project Overview

| | |
|---|---|
| **Goal** | Classify a hidden Boolean function as constant or balanced using quantum parallelism |
| **Algorithm** | Deutsch-Jozsa |
| **Classical Complexity** | O(2ⁿ⁻¹ + 1) queries (worst case) |
| **Quantum Complexity** | O(1) — exactly 1 oracle query |
| **Tools** | Python, Qiskit 2.4.1, qiskit-ibm-runtime, Matplotlib |
| **Simulator** | `FakeAlmadenV2` (noise-aware fake backend) |

---

## The Problem

Given a black-box function f: {0,1}ⁿ → {0,1}, determine whether f is:
- **Constant** — returns the same value (all 0s or all 1s) for every input
- **Balanced** — returns 0 for exactly half of inputs and 1 for the other half

Classically, this requires up to **2ⁿ⁻¹ + 1 queries** in the worst case. The Deutsch-Jozsa algorithm solves it in **exactly 1 query**, regardless of n.

---

## Circuit Architecture

### Step-by-Step Protocol

```
1. Initialize n input qubits in |0⟩ and 1 ancilla qubit in |1⟩
2. Apply Hadamard (H) to all n+1 qubits  →  creates uniform superposition
3. Apply oracle U_f                        →  encodes f as phase kickback
4. Apply Hadamard (H) to all n input qubits
5. Measure all n input qubits
   → All |0...0⟩  =  CONSTANT
   → Any |1⟩      =  BALANCED
```

### Why This Works — Phase Kickback

The ancilla qubit is prepared in the |−⟩ state (via X then H), which causes the oracle to imprint f's structure as a **phase** on the input register rather than flipping bits. After the second Hadamard layer, constructive interference collapses constant functions to the all-zeros state and destructive interference guarantees at least one non-zero qubit for balanced functions.

---

## Implementations

### 1. Single-Qubit Deutsch (2-qubit circuit)
A 2-qubit circuit (1 input + 1 ancilla) implementing all **4 possible 1-bit Boolean functions**:

| f value (input) | Function Type | Oracle Gate(s) Applied |
|---|---|---|
| f = 1 | Constant-0 | None |
| f = 2 | Balanced-identity | CNOT(q0 → q1) |
| f = 3 | Balanced-NOT | X(q0), CNOT(q0 → q1), X(q0) |
| f = 4 | Constant-1 | X(q1) |

User selects f at runtime; circuit is built dynamically and classified by comparing counts of measured `0` vs. `1`.

### 2. Generalized n-Qubit Deutsch-Jozsa
A parameterized **(n+1)-qubit circuit** (n input + 1 ancilla) that scales to any input size:

```python
n = int(input("Enter number of qubits: "))
# Builds QuantumRegister(n+1) and ClassicalRegister(n)
# Applies parallel H across all n+1 qubits
# Applies oracle U_f
# Applies parallel H across n input qubits
# Measures all n input qubits
```

Tested at **n = 5** (6-qubit circuit total); classified output as **Constant** when all measured bits were 0.

---

## Bonus: n-Qubit GHZ State

As a foundational warm-up, the notebook also implements a general **n-qubit GHZ state** builder:

```python
def get_qc_for_n_qubit_GHZ_state(n: int) -> QuantumCircuit:
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n - 1):
        qc.cx(i, i + 1)
    return qc
```

Demonstrated on a **7-qubit GHZ state** — a maximally entangled state where all qubits are correlated: |ψ⟩ = (|000...0⟩ + |111...1⟩) / √2.

---

## Simulation & Visualization

- Executed on **`FakeAlmadenV2`**, a noise-aware fake backend that mimics real IBM quantum hardware characteristics
- Measurement outcomes plotted as **bar charts** (counts of `0` vs. `1` for the 1-qubit case; `all-zeros` vs. `not all-zeros` for the n-qubit case)
- Circuit diagrams rendered via `qc.draw("mpl")` for visual verification of gate structure

---

## Installation & Usage

```bash
# Clone the repo
git clone https://github.com/your-username/deutsch-jozsa-qiskit.git
cd deutsch-jozsa-qiskit

# Install dependencies
pip install 'qiskit[visualization]' qiskit-ibm-runtime

# Launch the notebook
jupyter notebook Deutsch-Jozsa.ipynb
```

---

## Repository Structure

```
deutsch-jozsa-qiskit/
│
├── Deutsch-Jozsa.ipynb   # Full implementation notebook
└── README.md             # Project documentation
```

---

## Key Takeaways

- The Deutsch-Jozsa algorithm is one of the first proofs that quantum computers can solve certain problems **exponentially faster** than any classical algorithm
- The power comes from **quantum parallelism** (superposition) and **phase kickback** — not from running multiple computations sequentially
- The ancilla qubit in the |−⟩ state is what enables phase kickback; without it, the oracle would simply flip bits with no useful interference pattern
- The algorithm is exact and deterministic — unlike many quantum algorithms, it requires **no repetition** and produces the correct answer with probability 1
