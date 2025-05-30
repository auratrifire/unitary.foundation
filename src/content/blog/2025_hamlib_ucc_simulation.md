---
title: Extending UCC simulation benchmarks with Hamlib
author: Misty Wahl
day: 30
month: 5
year: 2025
tags: 
  - open-source
  - ucc
  - hamlib
---

Unitary Compiler Collection (UCC) is an open source software toolkit for quantum circuit compilation recently developed by the Unitary Foundation team, in collaboration with the quantum open source community. 
UCC combines a platform-agnostic interface with circuit optimization techniques that are competitive in compile time and gate reduction efficiency, on a suite of representative benchmarking circuits. 
The UCC team actively surveys and benchmarks compiler toolchains and techniques across the ecosystem and continuously improves UCC’s default compilation sequence, integrating passes from existing tools and developing custom passes.
UCC also accepts a wide array of input circuit formats and seamlessly compiles and converts between them by simple keyword arguments, affording cross-platform flexibility in compilation workflows.
More information about the UCC package can be found in the [launch announcement](blog/2025_UCC_launch_blog.md).
 
## UCC simulation benchmarks
In addition to compile time and gate reduction benchmarks, simulation benchmarks have been performed on the circuits output by UCC and other leading quantum compilers, to compare resilience to the effects of noise.
The primary metric of interest for the simulation benchmarks is the error in the expectation value of a problem-relevant observable or similar figure of merit.
For the square Heisenberg circuits and QAOA, the chosen observable of interest is the problem Hamiltonian, while for quantum volume (QV) and quantum Fourier transform (QFT) circuits computational basis measurements on all qubits are sufficient for benchmarking purposes.

## Hamlib Hamiltonian Library
In the case of the square Heisenberg simulation benchmarks, the UCC team was able to leveraging an existing implementation within the Hamlib (Hamiltonian Library) dataset [^1].
Hamlib contains a large collection of qubit-based quantum Hamiltonians to support the evaluation and development of quantum computing systems, across a diverse set of problem instances. 
The dataset is freely and publicly available online at https://portal.nersc.gov/cfs/m888/dcamps/hamlib. 
In addition to the Heisenberg model used for the UCC benchmarks, HamLib features models such as Fermi-Hubbard, Bose-Hubbard, various molecular structures, and optimization problems such as MaxCut and the traveling salesperson problem.
Supported problem sizes within Hamlib range from 2 to 1000 qubits.
The Hamlib dataset was developed and published to streamline research by reducing the need for manual problem preparation, enabling robust benchmarking, and promoting reproducibility and standardization in quantum studies, all shared goals of the UCC project and Unitary Foundation.

## UCC is a community driven project
UCC continues to collect and integrate performant and user-friendly tools for quantum compilation and compiler benchmarking. You can help by:

* [**Creating Custom Compiler Passes**](https://ucc.readthedocs.io/en/latest/contributing.html#proposing-a-new-transpiler-pass)  
* [**Reporting Bugs & Requesting Features**](https://github.com/unitaryfoundation/ucc/issues)  
* [**Contributing Code**](https://ucc.readthedocs.io/en/latest/contributing.html#contributing-guide)  
* [**Joining the Discussion**](https://discord.com/channels/764231928676089909/1346546840526524427)

**Connect with us\!**

GitHub: [**unitaryfoundation/ucc**](https://github.com/unitaryfoundation/ucc)  
Docs: [**UCC Documentation**](https://ucc.readthedocs.io/)  
Stay Updated: [**Mailing List**](https://bit.ly/uf-signup)


------------------------------------------------------

[^1]: Nicolas PD Sawaya, Daniel Marti-Dafcik, Yang Ho, Daniel P Tabor, David E Bernal Neira, Alicia B Magann, Shavindra Premaratne, Pradeep Dubey, Anne Matsuura, Nathan Bishop, Wibe A de Jong, Simon Benjamin, Ojas Parekh, Norm Tubman, Katherine Klymko, Daan Camps.
HamLib: A library of Hamiltonians for benchmarking quantum algorithms and hardware. _Quantum_ 8, 1559 (2024) [online] (https://dx.doi.org/10.22331/q-2024-12-11-1559).
