# ADCS - The Agentic Data Coordination Service

## Overview
# What is ADCS?
The concept of ADCS is derived from blockchain Data Oracles. While oracles can bring real-world data into the blockchain, ADCS goes beyond traditional oracles by providing both data and inference results from AI agents. Users can now leverage AI capabilities on the blockchain to power their applications.
For a practical example of blockchain oracles, see [Chainlink](https://chain.link/), one of the leading decentralized oracle networks.

## Why bring AI to the blockchain?

Integrating AI with blockchain technology creates a powerful synergy that addresses several critical needs in the modern digital ecosystem:

1. **Decentralized AI Decision Making**: By bringing AI to the blockchain, we can create transparent and verifiable AI decision-making processes. This is crucial for applications where trust and accountability are paramount, such as in financial services, healthcare, and supply chain management.

2. **Automated Smart Contracts**: AI can enhance smart contracts by adding intelligent decision-making capabilities, enabling more sophisticated and adaptive contractual agreements that can respond to complex real-world conditions.

3. **Democratization of AI Services**: Blockchain technology can help create a decentralized marketplace for AI services, making advanced AI capabilities accessible to a broader range of users and applications while ensuring fair compensation for AI service providers.

4. **Improved Security and Privacy**: The combination of blockchain's cryptographic security with AI's pattern recognition capabilities can create more robust security systems while maintaining privacy through techniques like federated learning and zero-knowledge proofs.

These capabilities open up new possibilities for decentralized applications (dApps) that can leverage both the trustless nature of blockchain and the intelligent processing power of AI systems.

## Architecture
The ADCS system follows an event-driven architecture where on-chain events trigger off-chain AI inference actions. When blockchain applications emit events, ADCS nodes detect these signals and coordinate the appropriate AI model executions and data processing in response. Below is a high-level diagram of the ADCS structure:

![View ADCS Structure Diagram](/images/ADCS_structure.JPG)

## Core Components

# ADCS Oracle Contracts

The ADCS Oracle contracts are responsible for requesting and fulfilling data from the ADCS platform. You can find the list of deployed contracts [here](https://github.com/Rivalz-ai/ADCS-core/tree/9f1ca495cce26b5859ad31f3836f03bf795ffb41/contracts/deployments)

- Coordinator Contract: This is the core contract used by consumers

  * Arbitrum: 0x07811b8B6151db734b8D1568918d3A62607879a7
  * Base: 0x91c5d6e9F50ec656e7094df9fC035924AAA428bb

# Providers

ADCS Providers are the actual LLM models and the entities that host them. A provider can be a single LLM model or a group of models that perform different tasks. This is the brain of the inference process where users can specify extra context and parameters via an Adaptor.
Provider attributes include:
- Provider Name
- LLM Model Name or list of LLM models
- Provider Description
- Provider API Endpoint
- Provider API Documentation
- Example execution result

# How to Become a Provider
METHOD 1: Self-hosted Service
- Fork our repository [here](https://github.com/Rivalz-ai/ADCS-core)
- Add information about your provider [here](https://github.com/Rivalz-ai/ADCS-core/blob/main/open/endpoints.json)
- Add your service endpoint, parameters, and expected output
- Create a Pull Request to the ADCS repository
We will review your PR and respond accordingly. After the PR is merged, you will become a provider.

METHOD 2: Use Our Hosted Service
- Fork our repository [here](https://github.com/Rivalz-ai/ADCS-core)
- Implement your custom service under the [inference](https://github.com/Rivalz-ai/ADCS-core/tree/main/inference/blueprints) folder
- Add information about your service in [descriptions.json](https://github.com/Rivalz-ai/ADCS-core/blob/main/inference/descriptions.json)
- Create a Pull Request to the ADCS repository
- We will review your PR and respond accordingly. After the PR is merged, you will become a provider.

# Adaptors

ADCS Adaptors act as bridges between the ADCS Oracle contracts and the Providers. They are responsible for providing extra context and formatting the request and response data between the two.

Adaptors can be created by leveraging other adaptors and providers as context. This composability allows developers to build more complex and specialized adaptors by combining the functionality of existing components, enabling greater flexibility and reusability within the ADCS ecosystem.

Adaptor attributes include:
- Adaptor Name
- Adaptor ID
- Category
- Network
- Providers Used
- Output Type
- Integration Code Example
- Playground for Testing
    - Parameters Required for Execution
    - Thinking Process
    - Correct Output Format

# Consumer Contracts

ADCS Consumer contracts request and process data from the ADCS platform. You can find example consumer contracts [here](https://github.com/Rivalz-ai/ADCS-core/tree/main/contracts/src/mock)

**We currently support 4 data types for fulfillment:**
1. **uint256**

```
abstract contract ADCSConsumerFulfillUint256 is ADCSConsumerBase {
    function fulfillDataRequest(uint256 requestId, uint256 response) internal virtual;

    function rawFulfillDataRequest(
        uint256 requestId,
        uint256 response
    ) external verifyRawFulfillment {
        fulfillDataRequest(requestId, response);
    }
}
```
2. **Bool**

```
abstract contract ADCSConsumerFulfillBool is ADCSConsumerBase {
    function fulfillDataRequest(uint256 requestId, bool response) internal virtual;

    function rawFulfillDataRequest(uint256 requestId, bool response) external verifyRawFulfillment {
        fulfillDataRequest(requestId, response);
    }
}
```

3. **Bytes32**

```
abstract contract ADCSConsumerFulfillBytes32 is ADCSConsumerBase {
    function fulfillDataRequest(uint256 requestId, bytes32 response) internal virtual;

    function rawFulfillDataRequest(
        uint256 requestId,
        bytes32 response
    ) external verifyRawFulfillment {
        fulfillDataRequest(requestId, response);
    }
}
```
4. **Bytes**

```
abstract contract ADCSConsumerFulfillBytes is ADCSConsumerBase {
    function fulfillDataRequest(uint256 requestId, bytes memory response) internal virtual;

    function rawFulfillDataRequest(
        uint256 requestId,
        bytes memory response
    ) external verifyRawFulfillment {
        fulfillDataRequest(requestId, response);
    }
}
```

5. **StringAndBool**

```
abstract contract ADCSConsumerFulfillStringAndBool is ADCSConsumerBase {
    function fulfillDataRequest(uint256 requestId, StringAndBool memory response) internal virtual;

    function rawFulfillDataRequest(
        uint256 requestId,
        StringAndBool memory response
    ) external verifyRawFulfillment {
        fulfillDataRequest(requestId, response);
    }
}
```
