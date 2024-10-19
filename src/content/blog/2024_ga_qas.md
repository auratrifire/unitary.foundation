---
title: An Introduction to Genetic Algorithms for Quantum Architecture Search
author: Vu Tuan Hai
day: 11
month: 9
year: 2024
tags:
  - Qiskit
  - Guest Post
  - Microgrant
---

In various quantum optimization problems, choosing the right ansatz is a critical point that will affect the accuracy of the result. For example, in Variational Quantum Eigensolver, many template ansatzes have been proposed, such as Graph Ansatz and EffecientSU2, etc. An ansatz [1] is the structure of quantum gates, deciding how many and what type of gates will be used, for easier imagination, it's equivalent to the model's architecture in a deep learning model.


Yet the above example ansatzes can be limited in their applicability. Hence, we've created a search engine called **GA-QAS** (Genetic Algorithm for Quantum Architecture Search) to aid in the discovery of the right ansatz. In this post, we will guide you on how to use it efficiently.

## Now, let’s start!

# Step 0. Setup the environment

Make sure that you installed Python 3 and the following packages:

- qiskit 1.0+
- matplotlib
- tdqm

We will use a Python package named <a href="https://github.com/vutuanhai237/qoop">$\langle \mathrm{qo}|\mathrm{op} \rangle$</a>, which is a core package for developing quantum circuit optimization frameworks. You can download it via `git` as:

```
git clone https://github.com/vutuanhai237/qoop.git
```

**Note**: You should put the repo on the same level as your Python/Jupyter Notebook file.

```
folder_name
│   your_python_file.py
|   your_jupyter_file.ipynb
└───qoop
│   └───evolution
│       │   environment.py
│       │   cross_over.py
│       │   ...
```

# Step 1. Define problem (W state preparation)

<figure>
    <img src='/images/ga-qas/ga-qas_scheme.png' width="600" style="margin: auto" alt='' />
    <figcaption>
    Fig 1. (a) State preparation scheme based on quantum compilation technique, where U, V are ansatz and state that need to prepare[2][3]. (b) Some default ansatz, such as hyper-graph.
    </figcaption>
</figure>

Quantum state preparation (QSP) is a process that uses a trainable unitary operator to transform the initial state into a desired target state.
This process is crucial for quantum computation and information processing.
For example in a quantum machine learning applications when you want to work with images, you must flatten the image as a vector and then use QSP to encode the image into a quantum state!

In this example we will show how to prepare a Werner state, also known as a [W state](https://en.wikipedia.org/wiki/W_state).
For 3-qubits, the W state is defined as:

$$
|W_3\rangle = \frac{1}{\sqrt{3}}(|001\rangle + |010\rangle + |100\rangle) = V|000\rangle
$$

To prepare the W state, we will need a unitary operator $V$.
In general, $V$ requires large resources such as a large depth or number of total gates.
Therefore, we look to find another unitary $U(\theta)$ that is trainable and that approximates $V$, but requires fewer resources than $V$.

The problem comes from here: the structure of $U$, affects how "hard" and "large" this optimization process is.
Then, we need to find the "best" $U$ automatically.
This is known as the "quantum architecture search" problem.

<figure>
    <img src='/images/ga-qas/ga-qas_example_plot_preparew.png' style="margin: auto" alt='' />
    <figcaption>Fig 2. Metrics of W-state preparation process versus iteration when running the example code. Note that loss_fubini_study is overlapped by trace distances. loss_fubini_study should be near 0 as much as possible.</figcaption>
</figure>

```py
import qiskit
from qoop.compilation.qsp import QuantumStatePreparation
from qoop.core import state

def fitnessW(qc: qiskit.QuantumCircuit):
    # Create a quantum state preparation optimizer based on a given trainable unitary (u) and target unitary.
    qsp = QuantumStatePreparation(
        u = qc,
        target_state = state.w(num_qubits = 3).inverse()
    ).fit() # Begin to optimize trainable unitary
    # qsp.plot() # Plot optimization process as Fig. 2.
    return 1 - qsp.compiler.metrics['loss_fubini_study'][-1] # Fitness value
```

**Make sure** that you can run the above code. We wrap the quantum state preparation problem into a function $f: \mathsf{U}(n) \rightarrow \mathbb{R}$, the return value of this function is $(1 - \text{last loss value})$, which means a value near 1 is desired.

# Step 2. Configuration for genetic algorithm

<figure>
    <img src='/images/ga-qas/ga-qas_pipeline.png' style="margin: auto" alt='' />
    <figcaption>Fig 3. The general pipeline of GA-QAS</figcaption>
</figure>

A genetic algorithm (GA) is a heuristic algorithm based on a genetic combination process. GA processes include Fitness evaluation, Selection, Cross-over, and Mutation. We treat ansatz as an individual in the population (where gates are its genes). The overall process can be viewed in _Fig. 3_.

|        Process        |                                                                                                     Method                                                                                                      |
| :-------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Population generation |                                                A $n$ ansatz is initialized from a pool gate (Clifford$+R_i+CR_i, i\in\{x,y,z\}$) with certain provided metadata                                                 |
|  Fitness evaluation   |                                                                             Evaluate each ansatz by calling `fitness_func(ansatz)`                                                                              |
|       Selection       |                                                    Get fitness value from the fitness evaluation; simply sort from high to low and take the first half part                                                     |
|      Cross-over       | Divide two ansatz $qc_1$ and $qc_2$ into four parts $\{qc_{11}, qc_{12}, qc_{21}, qc_{22}\}$, then combine each two parts into two new ansatz $qc_1^*=qc_{11}\vert qc_{22}$ and $qc_2^{*}=qc_{12}\vert qc_{21}$ |
|       Mutation        |                                              Each gate on ansatz has a small probability of mutating to another gate (with the same number of qubits) in the pool                                               |

_**Tab 1**. Detail of each operation in GA-QAS._

In the GA, some hyper-parameters need to be considered and defined before you run GA-QAS. Then, you need to create an `EEnvironment` object.
The important parameter is `fitness_func` which is the function we defined above.

```py
from qoop.evolution.environment import EEnvironmentMetadata
env_metadata = EEnvironmentMetadata(
        num_qubits, # As its name
        depth, # Ansatz depth you want
        num_circuit, # Number of ansatz per generation
        num_generation, # Number of generation/iteration for GA
        prob_mutate # Mutation probability, usually as small as 0.01 (1%)
)
from qoop.evolution.environment import EEnvironment
env = EEnvironment(
     metadata = env_metadata,
     fitness_func = fitnessW,
)
env.set_filename('GAQASPrepareW')
```

Default folder result will be set as `{num_qubit}+{fitness function}+{datetime}`, if you want shorter name, use `set_filename` method.

# Step 3. Run GA-QAS

You can call the method `evol()` to start running GA-QAS.

```py
env.evol()
```

<!-- ```py
env.evol(
   verbose = 1,
   auto_save = True
)
``` -->

<!-- There are only two parameters: the first is verbose in running process (1 means print process bar, 0 means no print anything), and the second is the saving option.
The saving result's folder is detailed on [GA-QAS: folder result](https://github.com/vutuanhai237/qoop/wiki/GA%E2%80%90QAS:-folder-result). -->

# Step 4. Plot result

Currently, we support plotting fitness values against a number of generations. In the future, we will develop more presentation plots.

```py
env.plot()
```

<figure>
    <img src='/images/ga-qas/ga-qas_example_plot_evolution.png' style="margin: auto" alt='' />
    <figcaption>Fig 4. Fitness values versus number of generations</figcaption>
</figure>

The result is saved in a folder; the default folder name is based on the fitness function name. We care about a file named `best_circuit.qpy`, which is our final solution. Then, we can load it by $\langle \mathrm{qo}|\mathrm{op} \rangle$ and put it into fitness again to test:

```py
from qoop.backend import utilities
best_circuit = utilities.load_circuit("./GAQASPrepareW/best_circuit.qpy")
fitness_value = fitnessW(best_circuit)
```

# Conclusion

In this post, we have demonstrated how to use GA-QAS. Simply define your own problem (fitness function) and the package will help you find the best ansatz automatically. For more information and documentation, explore the following links.

- $\langle \mathrm{qo}|\mathrm{op} \rangle$: core package for GA-QAS.
  - [Github](https://github.com/vutuanhai237/qoop)
  - [Wiki](https://github.com/vutuanhai237/qoop/wiki)
  - [Paper](https://doi.org/10.1016/j.softx.2024.101726)
- GA-QAS:
  - [Wiki](https://github.com/vutuanhai237/qoop/wiki/GA%E2%80%90QAS:-folder-result): Load folder result and continue to `evol()`
  - [Wiki](https://github.com/vutuanhai237/qoop/wiki/GA%E2%80%90QAS:-Full-pipeline): Full pipeline
  - [Paper](https://arxiv.org/abs/2407.01010)

Thanks for reading! Please do not hesitate to ask us any questions via e-mail at [vutuanhai237@gmail.com](mailto:vutuanhai237@gmail.com) or [LinkedIn](https://www.linkedin.com/in/vutuanhai237/).

# References

[1] https://pennylane.ai/qml/glossary/circuit_ansatz/

[2] Hai, V. T., Viet, N. T., & Ho, L. B. (2023). Variational preparation of entangled states on quantum computers. arXiv preprint arXiv:2306.17422.

[3] Khatri, S., LaRose, R., Poremba, A., Cincio, L., Sornborger, A. T., & Coles, P. J. (2019). Quantum-assisted quantum compiling. Quantum, 3, 140.

# Appendix: Full code

The Jupyter notebook which contains the above code can be found [here](https://github.com/vutuanhai237/GA-QAS/blob/694ff6a509df9acc7d3e8053ab89ccfd7d3127c0/example/w_preparation.ipynb)

```py
import qiskit
from qoop.core import state
from qoop.compilation.qsp import QuantumStatePreparation
from qoop.evolution.environment import EEnvironmentMetadata
from qoop.evolution.environment import EEnvironment

def fitnessW(qc: qiskit.QuantumCircuit):
    qsp = QuantumStatePreparation(
        u = qc,
        target_state = state.w(num_qubits = 3).inverse()
    ).fit(num_steps = 100)
    return 1 - qsp.compiler.metrics['loss_fubini_study'][-1] # Fitness value

env_metadata = EEnvironmentMetadata(
    num_qubits = 3, # As its name
    depth = 4, # Ansatz depth you want
    num_circuit = 4, # Number of ansatz per generation
    num_generation = 10, # Number of generation/iteration for GA
    prob_mutate = 0.01 # Mutation probability, usually as small as 0.01 (1%)
)

env = EEnvironment(
    metadata = env_metadata,
    fitness_func = fitnessW,
).evol()
```
