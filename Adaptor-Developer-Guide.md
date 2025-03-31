# ADCS Developer Guide: Creating Adaptors

## Introduction

This guide provides detailed instructions for developers on how to create new adaptors by combining existing providers and adaptors within the ADCS (AI-Driven Consensus System) framework. Adaptors are essential components that transform, process, and format data between providers and blockchain applications.

## Key Concepts

### Components and Their Attributes

1. **Providers**
   - **Provider name**: The name of the provider
   - **Category**: The category of the provider:
     - **Inference**: Providers that provide inference services
     - **No inference**: Providers that provide data services
     - **Core LLM**: Providers that provide core LLM services
   - **Parameters**: Parameters for the provider. Optional.
   - **LLM**: The LLM to use for the provider. Optional.
   - **Description**: A description of the provider.
   - **Endpoint**: The endpoint to use for the provider.
   - **API documentation**: The API documentation for the provider.
   - **Input**: Raw data or query (type depends on provider implementation)
   - **Output Format**: The format of the output data.

#### Provider Examples by Category

1. **Inference Provider Example**
   ```
   {
     "id": "provider-meme-001",
     "name": "meme coin trend Provider",
     "category": "Inference",
     "description": "Provide a sentiment analysis of a meme coins and return which coin you should buy or sell",
     "endpoint": "https://api.sentiment-analysis.com/analyze",
     "parameters": {},
     "LLM": "gpt-4o-mini",
     "input": "popular X posts on a meme coin",
     "outputFormat": "StringAndBool"
   }
   ```

2. **No Inference Provider Example**
   ```
   {
     "id": "provider-market-001",
     "name": "Financial Market Data Provider",
     "category": "No inference",
     "description": "Provides real-time and historical market data for financial assets",
     "endpoint": "https://api.financial-data.com/market",
     "LLM": null,
     "parameters": {
       "Symbol": "Asset symbol or identifier (e.g., 'TSLA', 'BTC-USD')"
     },
     "outputFormat": "JSON"
   }
   ```

3. **Core LLM Provider Example**
   ```
   {
     "id": "provider-gpt4-001",
     "name": "GPT-4 Provider",
     "category": "Core LLM",
     "description": "Provides access to OpenAI's GPT-4 model for general-purpose text generation and reasoning",
     "endpoint": "https://api.openai.com/v1/chat/completions",
     "parameters": {},
     "LLM": "gpt-4",
     "input": "Prompt text with optional system instructions",
     "outputFormat": "String"
   }
   ```

2. **Adaptors**
   - **id**: The unique identifier for the adaptor.
   - **name**: The name of the adaptor.
   - **Input**: One or more provider/adaptor outputs as data sources.
   - **Core LLM**: Optional AI model used by the adaptor for processing or reasoning
   - **Static Context**: Predefined information or rules provided to the adaptor
   - **Output**: Structured data in a specific format for blockchain applications (e.g., `BoolOutput`, `Uint256Output`, `StringAndBoolOutput`, `BytesOutput`)


### Adaptor Types

1. **Adaptor as wrapper for Provider**: use provider as input and output
2. **Single Input Adaptor**: Takes output from one adaptor as input
3. **Multi-Input Adaptor**: Takes outputs from multiple providers/adaptors as input

## How to leverage your adaptors by using existing adaptors and providers

Adaptors and providers can be connected in various configurations to create processing workflows. These connections form a network of components that process and transform data in sequence or in parallel.

### UseCase 1: Creating a new adaptor by wrapping the XtrendProvider

#### Description

In this simplified use case, we create a basic adaptor that wraps the XtrendProvider. Since the provider already delivers data in our desired format, our adaptor will primarily serve as a gateway that controls access to the provider and provides a standardized interface within the ADCS system.

#### Step-by-Step Implementation

1. **Identify the existing provider**:
   The XtrendProvider has the following attributes:
   ```
   id: "provider-xtrend-001"
   name: "Crypto Trend Provider"
   category: "Inference"
   description: "Analyzes social media for cryptocurrency trends"
   endpoint: "https://api.xtrend.io/v1/crypto-trends"
   parameters: {
     timeframe: "24h"
   }
   outputFormat: "StringAndBool"  // Already in our desired format
   ```

2. **Create the wrapper adaptor**:

```
// Pseudocode for defining a simple CryptoTrendAdaptor

// Define the adaptor configuration
const adaptor = {
  id: "adaptor-crypto-trend-v1",
  name: "Cryptocurrency Trend Adaptor",
  input: {
    sources: ["provider-xtrend-001"]
  },
  coreLLM: null,  // No LLM needed for this simple pass-through
  staticContext: "",  // No transformation needed
  outputFormat: "StringAndBool"  // Same as the provider
};

// The adaptor simply passes through the provider's output
function processTrendData(providerOutput) {
  // Since the provider already gives us what we need,
  // we simply return its output directly
  return providerOutput;
}
```

#### Example Usage of the Wrapped Provider

```
// Pseudocode for using the adaptor

// The adaptor automatically uses whatever the provider returns
// No specific input needed - the provider already handles data collection

// Call the adaptor (handled by ADCS runtime)
const result = adaptorSystem.execute("adaptor-crypto-trend-v1");

// Example output (passed through directly from provider):
// {
//   string: "BTC: Strong positive sentiment with 5800 mentions in last 24h",
//   bool: true  // Provider's recommendation to buy
// }

// This output can be used directly by a smart contract:
if (result.bool) {
  // Execute buy order
} else {
  // Hold position
}
```
### UseCase 2: Creating a new adaptor by using the Coinmarketcap Provider

#### Description

In this use case, we create a more sophisticated adaptor that uses the Coinmarketcap Provider to analyze cryptocurrency market data and identify the top 5 tokens worth buying based on a set of predefined criteria. Unlike the simple wrapper in UseCase 1, this adaptor performs significant analysis and transformation of the provider's data to generate actionable investment recommendations.

#### Step-by-Step Implementation

1. **Identify the existing provider**:
   The Coinmarketcap Provider has the following attributes:
   ```
   id: "provider-coinmarketcap-001"
   name: "Coinmarketcap Data Provider"
   category: "No inference"
   description: "Provides real-time market data for cryptocurrencies from Coinmarketcap"
   endpoint: "https://api.coinmarketcap.com/v1/cryptocurrency/listings/latest"
   parameters: {
     limit: "100",  // Number of tokens to retrieve
     sort: "market_cap",  // Sort by market capitalization
     sort_dir: "desc"  // Sort in descending order
   }
   LLM: null
   outputFormat: "JSON"  // Structured market data
   ```

2. **Define the selection and analysis criteria**:
   Our adaptor will filter and analyze tokens based on:
   - Market capitalization (top 100 tokens by default)
   - Recent price performance (24h, 7d changes)
   - Trading volume vs. market cap ratio
   - Relative volume increase/decrease
   - Market sentiment indicators

3. **Create the investment analysis adaptor**:

```
// Pseudocode for defining a CryptoInvestmentAdaptor

// Define the adaptor configuration
const adaptor = {
  id: "adaptor-crypto-investment-v1",
  name: "Crypto Investment Recommendation Adaptor",
  input: {
    sources: ["provider-coinmarketcap-001"]
  },
  coreLLM: "gpt-4",  // Using an LLM to analyze patterns and generate insights
  staticContext: `
    Analyze the top 100 cryptocurrencies by market cap and identify the 5 tokens most worth buying based on:
    1. Price momentum: Look for positive but not overheated price action (10-30% gains in last 7 days)
    2. Volume profile: Trading volume should be increasing but sustainable (volume/market cap ratio between 0.1-0.5)
    3. Market position: Prefer tokens in the top 50 by market cap for liquidity reasons
    4. Avoid tokens that have increased more than 40% in the last 24 hours (potential pump and dump)
    5. Consider relative value compared to similar tokens in the same category
    return an array of 5 strings
    `
  outputFormat: "String[5]"  // return an array of 5 strings
  }
```

#### Example Usage of the Investment Adaptor

```
// Pseudocode for using the adaptor

// Call the adaptor (handled by ADCS runtime)
const result = adaptorSystem.execute("adaptor-crypto-investment-v1");

// Example output: ["BTC", "ETH", "SOL", "XRP", "ADA"]

This use case demonstrates creating an adaptor that performs significant analysis and transformation on data from a provider. By applying filtering, scoring algorithms, and LLM-based analysis, the adaptor converts raw market data into actionable investment recommendations with explanations.

## Best Practices for Complex Adaptor 
1. **Modularize Your Design**: Break complex logic into smaller, specialized adaptors
2. **Reuse Adaptors**: Create adaptors that can be reused in multiple inputs
3. **Document Dependencies**: Clearly document the input/output relationships between components
4. **Test Each Component**: Test each provider and adaptor individually before connecting them
5. **Visualize Your Network**: Use diagrams to visualize the flow of data through your adaptor connections
6. **Optimize Static Context**: Keep static context concise and relevant to the specific task
7. **Select Appropriate LLMs**: Choose the right core LLM based on the complexity of the task
8. **Define Aggregation in Static Context**: For multi-input adaptors, clearly specify aggregation rules in the static context
9. **Handle Edge Cases**: Include logic for handling missing, conflicting, or low-confidence inputs
10. **Use Consistent IDs**: Create unique, descriptive IDs for all providers and adaptors

## Debugging Adaptor Connections

When debugging connected adaptors, follow these steps:

1. **Trace Data Flow**: Trace how data flows through the connected components
2. **Log Intermediate Results**: Log the output of each component
3. **Check Compatibility**: Verify that each component receives compatible inputs
4. **Test Edge Cases**: Test the workflow with edge cases and unexpected inputs
5. **Validate Final Output**: Ensure the final output meets the expected format and quality
6. **Inspect LLM Interactions**: Review how core LLMs are processing the inputs
7. **Verify Static Context Usage**: Confirm static context is properly applied
8. **Check Aggregation Logic**: Ensure that multi-input aggregation is performed according to static context instructions
9. **Validate ID References**: Ensure all ID references are correct and components exist

## Conclusion

By following this guide, you should now be able to create complex adaptor networks that combine various providers and adaptors to process and format data for blockchain applications. Remember to design your connections with modularity, reusability, and clear data flow in mind. The inclusion of core LLMs and static context provides additional flexibility and intelligence to your adaptors, enabling more sophisticated processing and decision-making capabilities. 

For multi-input adaptors, always include clear aggregation instructions in the static context to ensure inputs are combined correctly according to your specific requirements. By default, simple text concatenation is used for combining outputs, but you can specify more complex aggregation logic in the static context when needed.

Always use unique, consistent IDs to ensure proper referencing between components in your adaptor network. 