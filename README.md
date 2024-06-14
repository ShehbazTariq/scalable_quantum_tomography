
# Scalable Estimation of Pure Multi-Qubit States Using Qiskit

This repository contains a Python implementation of the scalable estimation method for pure multi-qubit states as described in the paper "Scalable estimation of pure multi-qubit states" published in npj Quantum Information. This method leverages projective measurements on separable and entangled bases and is particularly suitable for applications in noisy intermediate-scale quantum (NISQ) computers.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
- [Functions](#functions)
- [Example](#example)
- [Contributing](#contributing)
- [License](#license)

## Introduction

The estimation method introduced in the paper provides a favorable scaling in the number of qubits compared to other state estimation schemes. It uses projective measurements on \(m n + 1\) separable bases or \(m\) entangled bases plus the computational basis, with \(m \geq 2\). This implementation uses Qiskit, a Python library for quantum computing, to simulate and experimentally demonstrate the method on IBM’s quantum processors.

## Installation

To run this implementation, you need to have Python 3.7+ installed. You can install the required dependencies using pip:

```bash
pip install qiskit numpy matplotlib sympy tqdm qutip
```

## Usage

### Functions

1. **Dec2Bin(x, n)**
   Converts a decimal number `x` into a binary representation of width `n`.

   ```python
   def Dec2Bin(x, n):
       return np.array([int(i) for i in np.binary_repr(x, width=n)])
   ```

2. **get_proj(n)**
   Generates the projective measurements for `n` qubits.

   ```python
   def get_proj(n):
       circ_idx = {}    
       T_Proj = []

       states = ['0','1']
       idx_r = [0,1]
       
       for j in range(n):
           z_len = n-(j+1)
           b_len = 2**(z_len) - 1
           m_len = (j+1)-1

           if b_len != 0:
               for b in range(b_len+1):
                   proj = []
                   Bin = Dec2Bin(b, z_len)
                   for i in Bin:
                       proj.append(states[i])
                   proj.append(states[idx_r[0]])

                   for i in range(m_len):
                       proj.append(states[idx_r[1]])

                   T_Proj.append(''.join(proj))
           else:
               proj = []
               proj.append(states[idx_r[0]])
               for i in range(m_len):
                   proj.append(states[idx_r[1]])
               
               T_Proj.append(''.join(proj))
           circ_idx[j] = T_Proj
           T_Proj = []
       
       return circ_idx
   ```

3. **get_keys(n)**
   Generates binary keys for `n` qubits.

   ```python
   def get_keys(n):
       keys = []
       for i in range(2**n):
           keys.append(format(i, '0'+str(n)+'b'))
       return keys
   ```

4. **IBM_counts(n, PSI, a, shots, qcomp)**
   Performs projective measurements on IBM quantum computers.

   ```python
   def IBM_counts(n, PSI, a, shots, qcomp):
       proj  = get_proj(n)
       prob_dict = {}
       for i in range(n):
           qc = QuantumCircuit(n)
           # Prepare n qubit random state
           qc.initialize(PSI, range(n))
           for j in range(i+1):
               if a == 1:
                   qc.h(i-j)
               else:
                   qc.sdg(i-j)
                   qc.h(i-j)
           qc.measure_all()
           job = execute(transpile(qc, qcomp), qcomp, shots=shots)
           result = job.result()
           counts = result.get_counts(qc)
       
           values = list(map(counts.get, proj[i], [0]*len(proj[i])))
           prob_dict[i] = np.array(values) / shots
       
       return prob_dict
   ```

### Example

```python
import numpy as np
from qiskit import Aer, transpile, QuantumCircuit
from qiskit.providers.aer import AerSimulator
from qiskit.quantum_info import random_statevector
from your_module import Dec2Bin, get_proj, get_keys, IBM_counts

# Initialize quantum simulator
simulator = AerSimulator()

# Prepare a random n-qubit state
n = 4
PSI = random_statevector(2**n).data

# Set parameters
shots = 1024
a = 1

# Perform measurements and estimate the state
prob_dict = IBM_counts(n, PSI, a, shots, simulator)

print("Probability Dictionary:", prob_dict)
```

## Contributing

We welcome contributions to improve the implementation and extend its functionality. Please fork the repository and submit pull requests.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## References

- Pereira, L., Zambrano, L., & Delgado, A. (2022). Scalable estimation of pure multi-qubit states. npj Quantum Information, 8, 57. [Link to the paper](https://www.nature.com/articles/s41534-022-00565-9)

---

For more detailed examples and usage, please refer to the documentation in the `docs` directory.
