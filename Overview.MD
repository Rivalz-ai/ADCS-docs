# ADCS - The Agentic Data Coordination Servcie

## Overview
# What is ADCS?
The Idea of ADCS derives from blockchain Data Oracles. With oracles, we can get data from the real world into the blockchain. But ADCS is different from traditional oracles where we provide both data and inference results from AI agents. User can now bring AI to the blockchain and fuel their products with AI.
For a practical example of blockchain oracles, see [Chainlink](https://chain.link/), one of the leading decentralized oracle networks.

## Why need to bring AI to the blockchain?

Integrating AI with blockchain technology creates a powerful synergy that addresses several critical needs in the modern digital ecosystem:

1. **Decentralized AI Decision Making**: By bringing AI to the blockchain, we can create transparent and verifiable AI decision-making processes. This is crucial for applications where trust and accountability are paramount, such as in financial services, healthcare, and supply chain management.

2. **Automated Smart Contracts**: AI can enhance smart contracts by adding intelligent decision-making capabilities, enabling more sophisticated and adaptive contractual agreements that can respond to complex real-world conditions.

3. **Democratization of AI Services**: Blockchain technology can help create a decentralized marketplace for AI services, making advanced AI capabilities accessible to a broader range of users and applications while ensuring fair compensation for AI service providers.

4. **Improved Security and Privacy**: The combination of blockchain's cryptographic security with AI's pattern recognition capabilities can create more robust security systems while maintaining privacy through techniques like federated learning and zero-knowledge proofs.


These capabilities open up new possibilities for decentralized applications (dApps) that can leverage both the trustless nature of blockchain and the intelligent processing power of AI systems.

## Architecture
The ADCS system follows an event-driven architecture where on-chain events trigger off-chain AI inference actions. When blockchain applications emit events, ADCS nodes detect these signals and coordinate the appropriate AI model executions and data processing in response. Below is a high-level diagram of the ADCS structure:

[View ADCS Structure Diagram](https://raw.githubusercontent.com/ADCS-AI/ADCS-docs/main/images/ADCS_structure.png)

## Core Components

# Adcs Oracle contracts

The ADCS Oracle contracts are responsible for requesting data from the ADCS platform and fulfilling the request. You can find the list of deployed contracts [here]
- Coordinator Contract:

# Providers

The ADCS Providers are the actualy LLM and the entity that host the LLM models. It can be a single LLM model or a group of models that perform different tasks. This is the brain of the inference process where user can spcecify extra context, parameters via an Adaptor.
Providers contains these attributes:
- Provider Name
- LLM Model Name OR list of LLM models
- Provider Description
- Provider API Endpoint
- Provider API Documentation
- Example execution result

# How to become a Provider
METHOD 1: self host service.
- Fork our boilerplate repo [here]
- Copy the Definition Json file
- Add your service endpoint, parameters, and expected output.
- Create the Pull Request to the ADCS repo.
We will review your PR and get back to you. After the PR is merged, you will be a provider.

METHOD 2: Use our hosted service.
- Fork our boilerplate repo [here]
- implement your custom service under the `service` folder.
- Raise a PR to the ADCS repo.
- We will review your PR and get back to you. After the PR is merged, you will be a provider.

# Adaptors

The ADCS Adaptors acts as the bridge between the ADCS Oracle contracts and the Providers. It is responsible for providing extra contexts, formatting the request and response data between the two.
Adaptors contains these attributes:
- Adaptor Name
- Adaptor ID
- Category
- Network
- Provider used
- Output Type
- Integration Code example
- Playground for testing
    - Parameters required for execution
    - Thinking process
    - Correct output format

# Consumer contracts

The ADCS Consumers are the smart contracts that request the data from the ADCS platform and fulfill the request.


