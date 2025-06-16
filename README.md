# Quantum SAT Solver - QCIC 766 Project

A quantum computing implementation for solving Boolean Satisfiability (SAT) problems using Grover's algorithm and quantum counting techniques.

## Overview

The Boolean Satisfiability Problem (SAT) is one of the most fundamental problems in computer science and the first problem proven to be NP-complete. This project implements a quantum approach to solving SAT problems, potentially offering quadratic speedup over classical algorithms for certain instances.

### What is the SAT Problem?

The SAT problem asks whether there exists an assignment of boolean values (true/false) to variables that makes a given boolean formula evaluate to true. For example, given the formula `(x1 OR x2) AND (NOT x1 OR x3)`, we need to find values for x1, x2, and x3 that make the entire expression true.

### Why Quantum Computing for SAT?

Classical SAT solvers can take exponential time in the worst case. Quantum algorithms like Grover's search can provide a quadratic speedup, reducing the search complexity from O(2^n) to O(2^(n/2)) for n variables.

## Key Components

### 1. DIMACS CNF Parser (`dimacs_cnf_files_parser`)

The parser reads SAT problems in the standard DIMACS CNF (Conjunctive Normal Form) format:

**Understanding CNF Format:**
- **CNF** means the formula is a conjunction (AND) of clauses, where each clause is a disjunction (OR) of literals
- **Literals** are either variables (positive) or their negations (negative)
- **Example**: `(x1 OR NOT x2) AND (x2 OR x3)` is in CNF form

**DIMACS Format Structure:**
```
c This is a comment line
p cnf 3 2          # Problem line: 3 variables, 2 clauses
1 -2 0             # Clause 1: (x1 OR NOT x2)
2 3 0              # Clause 2: (x2 OR x3)
```

The parser extracts:
- Number of variables and clauses
- List of clauses as integer arrays
- Generates human-readable boolean expressions

### 2. Quantum Counting (`quantum_counting`)

Before solving the SAT problem, we need to estimate how many solutions exist. This is crucial for optimizing Grover's algorithm.

**How Quantum Counting Works:**

Think of quantum counting like estimating the number of marked items in a database without examining each item individually. The algorithm uses:

1. **Quantum Phase Estimation**: Creates superposition states and applies controlled operations
2. **Grover Operator**: Amplifies the amplitude of satisfying assignments
3. **Inverse Quantum Fourier Transform**: Extracts phase information to estimate solution count

**Mathematical Foundation:**
- If M solutions exist out of N total possibilities, Grover's operator rotates the state vector by angle θ where sin²(θ) = M/N
- By measuring the phase, we can estimate θ and hence M

### 3. Partial Diffusion SAT Solver (`pd_sat_solver`)

The main solver uses a modified Grover's algorithm with partial diffusion operator.

**Algorithm Steps:**

1. **Initialization**: Create uniform superposition of all possible variable assignments
2. **Oracle Construction**: Build quantum circuit that marks satisfying assignments
3. **Amplitude Amplification**: Use partial diffusion to amplify probability of correct solutions
4. **Measurement**: Extract the most probable solutions

**Understanding the Oracle:**

The oracle is the heart of the quantum SAT solver. It works by:
- Using ancillary qubits to evaluate each clause
- Applying De Morgan's laws to convert OR operations to AND operations (easier to implement the clause by mcx gate)
- Marking states where all clauses are satisfied

**Why Partial Diffusion?**

Traditional Grover's algorithm assumes we know exactly how many solutions exist. Partial diffusion is more robust when our estimate might be imperfect, preventing the algorithm from "overshooting" and reducing success probability.

## Circuit Architecture

### Quantum Registers Used:

1. **Literal Register**: Stores the boolean values of variables (x1, x2, ..., xn)
2. **Clause Register**: Auxiliary qubits for evaluating individual clauses
3. **Ancillary Register**: Workspace for final SAT evaluation
4. **Phase Register**: Used in quantum counting for phase estimation
5. **Classical Register**: Stores measurement results

### Key Quantum Operations:

- **Multi-Controlled X Gates**: Implement logical AND operations
- **Hadamard Gates**: Create and manipulate superposition states
- **Controlled Unitary Operations**: Apply Grover operators conditionally
- **Quantum Fourier Transform**: Extract phase information in counting

## Usage Example

```python
# 1. Parse DIMACS CNF file
variables_number, clauses_number, cnf_clauses_list = dimacs_cnf_files_parser()

# 2. Estimate number of solutions using quantum counting
estimated_solutions_number = quantum_counting(variables_number, clauses_number, cnf_clauses_list)

# 3. Solve SAT problem using partial diffusion
pd_sat_solver(variables_number, clauses_number, cnf_clauses_list, estimated_solutions_number)
```

## Implementation Details

### De Morgan's Laws in Quantum Circuits

Converting CNF clauses to quantum circuits requires careful application of De Morgan's laws:
- `(A OR B) = NOT(NOT A AND NOT B)`
- This allows us to use multi-controlled X gates (which implement AND) to evaluate OR clauses

### Uncomputation Strategy

Quantum circuits must be reversible, so intermediate computations need to be "uncomputed" to avoid entanglement with ancillary qubits. The solver carefully reverses operations to clean up workspace qubits.

### Iteration Count Optimization

The number of Grover iterations is calculated as: `π/4 × √(N/M)`
where N is the total search space size and M is the estimated number of solutions.

## Sample Test Files

The project includes various DIMACS CNF test files:
- `2vsat.cnf`, `3vsat.cnf`: Simple satisfiable instances
- `u3v6c.cnf`, `u4v8c.cnf`: Unsatisfiable instances for testing
- `max.cnf`, `max4.cnf`: Maximum satisfiability problems

## Dependencies

```python
import os, sys, math, itertools
from qiskit import QuantumRegister, ClassicalRegister, QuantumCircuit, transpile
from qiskit.circuit.library import QFTGate, UnitaryGate
from qiskit_aer import AerSimulator
from IPython.display import display
from qiskit.visualization import plot_histogram
```

## Output Interpretation

The solver provides:
1. **Circuit Visualization**: Shows the quantum circuit structure
2. **Histogram Plots**: Display measurement probability distributions
3. **Solution Assignments**: Boolean variable assignments that satisfy the formula
4. **Confidence Percentages**: How often each solution appeared in measurements

## Educational Value

This implementation demonstrates several important quantum computing concepts:

**Quantum Parallelism**: How superposition allows simultaneous evaluation of all possible variable assignments

**Amplitude Amplification**: The principle behind Grover's algorithm and how it provides quadratic speedup

**Quantum Phase Estimation**: A fundamental subroutine used in many quantum algorithms

**Reversible Computing**: How quantum circuits maintain unitarity through careful uncomputation

## Theoretical Background

The quantum SAT solver builds upon several key theoretical results:

**Grover's Search Algorithm**: Provides quadratic speedup for unstructured search problems

**Quantum Counting**: Allows estimation of solution space size without exhaustive search

**Amplitude Amplification**: Generalizes Grover's algorithm to arbitrary initial states and success probabilities

This implementation serves as both a practical tool for small SAT instances and an educational demonstration of how quantum algorithms can be applied to classical computational problems.

---

*This project was developed for QCIC 766 - Research Project course, Professional Master in Quantum Computing and Quantum Information, demonstrating the application of quantum algorithms to NP-complete problems.*
