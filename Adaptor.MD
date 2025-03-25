## Mathematical Representation of the System
Define the system using functions and composition, emphasizing how Providers and Adaptors work together.

### 1. Providers
A Provider generates inference data based on some input. We can define it as a function:

$$P: I \rightarrow O_P$$

where:
- $I$ is the input domain (e.g., raw data or a query)
- $O_P$ is the output domain of the Provider, representing the inference data it produces

For example, if you have two Providers, $P_1$ and $P_2$:
- $O_{P_1} = P_1(I)$
- $O_{P_2} = P_2(I)$

This mathematical representation shows that:
1. Each Provider is a function that transforms input data into inference results
2. Different Providers can produce different types of outputs from the same input
3. The input domain can be standardized across Providers while output domains may vary

### 2. Adaptors
An Adaptor transforms and processes data between different formats. We can define it as a function:

$$A: O_P \rightarrow O_A$$

where:
- $O_P$ is the output domain of a Provider
- $O_A$ is the output domain of the Adaptor after processing

Adaptors can also compose with other Adaptors, creating a chain of transformations:

$$A_2 \circ A_1: O_P \rightarrow O_{A_1} \rightarrow O_{A_2}$$

### 3. System Composition
The complete ADCS system can be represented as a composition of functions:

$$ADCS: I \xrightarrow{P} O_P \xrightarrow{A} O_A \rightarrow Blockchain$$

This composition demonstrates how data flows from raw input through Providers and Adaptors before being delivered to the blockchain.





