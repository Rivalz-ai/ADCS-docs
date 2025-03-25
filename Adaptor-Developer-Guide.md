# ADCS Developer Guide: Creating Adaptors

## Introduction

This guide provides detailed instructions for developers on how to create new adaptors by combining existing providers and adaptors within the ADCS (AI-Driven Consensus System) framework. Adaptors are essential components that transform, process, and format data between providers and blockchain applications.

## Key Concepts

### Components and Their Attributes

1. **Providers**
   - **Input**: Raw data or query (type depends on provider implementation)
   - **Output**: Unstructured inference data in the `ProviderOutput` format:
     ```
     ProviderOutput:
         result      # The main inference result
         confidence  # Confidence score (0.0-1.0)
         metadata:   # Additional information
             model
             timestamp
             processingTime
     ```

2. **Adaptors**
   - **Input**: One or more provider/adaptor outputs as data sources
   - **Core LLM**: Optional AI model used by the adaptor for processing or reasoning
   - **Static Context**: Predefined information or rules provided to the adaptor
   - **Output Format**: Structured data in a specific format for blockchain applications (e.g., `BoolOutput`, `Uint256Output`, `StringAndBoolOutput`, `BytesOutput`)

   ```
   # Adaptor attribute structure
   Adaptor:
       name          # Unique identifier for the adaptor
       input         # Data sources from providers or other adaptors
       coreLLM       # AI model used for processing
       staticContext # Fixed information or rules
       config        # Processing parameters and settings
       outputFormat  # Type of structured output
   ```

### Adaptor Types

1. **Single Input Adaptor**: Takes output from one provider/adaptor as input
2. **Multi-Input Adaptor**: Takes outputs from multiple providers/adaptors as input
3. **Chained Adaptor**: Creates a sequential processing pipeline of adaptors

## Creating Adaptor Graphs

Adaptors and providers can be connected in various configurations to create processing graphs. When designing these graphs, follow these rules:

### Rules for Creating Adaptor Graphs

1. **Input/Output Compatibility**: Ensure that each adaptor receives inputs in the format it expects
2. **Type Safety**: Verify that the output format of a provider/adaptor matches the expected input format of the next adaptor in the chain
3. **Execution Order**: Consider the order of execution in the graph, especially for multi-input adaptors
4. **Error Handling**: Include strategies for handling failures at any point in the graph
5. **Circular Dependencies**: Avoid creating circular dependencies between adaptors

### Functions for Graph Construction

When creating a graph of adaptors and providers, you'll need to implement these key functions:

```
# Validate compatibility between components
function validateCompatibility(source, target):
    sourceOutput = getOutputType(source)
    targetInput = getInputType(target)
    return isCompatible(sourceOutput, targetInput)

# Resolve execution order for a graph
function resolveExecutionOrder(graph):
    # Use topological sort to determine execution order
    return topologicalSort(graph)

# Execute a node in the graph
function executeNode(node, inputs):
    if isProvider(node):
        return node.process(inputs.rawInput)
    else:
        return node.process(inputs.adaptorInputs)

# Execute the entire graph
function executeGraph(graph, input):
    order = resolveExecutionOrder(graph)
    results = new Map()
    
    # Execute nodes in topological order
    for each node in order:
        nodeInputs = getInputsForNode(node, results, input)
        result = executeNode(node, nodeInputs)
        results[node.id] = result
    
    return results[graph.output]
```

## Step-by-Step Guide to Creating Adaptors

### 1. Creating a Single Input Adaptor

```
# Example: Creating a sentiment analyzer adaptor
function createSentimentAdaptor():
    # Step 1: Create provider
    sentimentProvider = new Provider(
        name="SentimentAnalysis", 
        model="bert-sentiment", 
        endpoint="sentiment-api-endpoint"
    )
    
    # Step 2: Configure the adaptor
    config = {
        threshold: 0.7,
        outputMapping: {
            positive: true,
            negative: false,
            neutral: false
        }
    }
    
    # Define static context
    staticContext = "Financial sentiment analysis should focus on words that indicate market trends."
    
    # Step 3: Create adaptor with provider as input
    sentimentAdaptor = new SingleInputAdaptor(
        name="SentimentDecision",
        inputSource=sentimentProvider,
        coreLLM="gpt-4",  # Optional LLM for additional processing
        staticContext=staticContext,
        config=config,
        outputFormat="Bool"
    )
    
    return sentimentAdaptor
```

### 2. Creating a Multi-Input Adaptor

```
# Example: Creating a financial decision adaptor
function createFinancialDecisionAdaptor():
    # Step 1: Create input sources
    newsProvider = new Provider("NewsAnalysis", "gpt-4", "news-api-endpoint")
    marketDataProvider = new Provider("MarketData", "data-api", "market-api-endpoint")
    sentimentAdaptor = createSentimentAdaptor()
    
    # Step 2: Configure the adaptor
    config = {
        threshold: 0.6,
        weights: {
            news: 0.3,
            marketData: 0.5,
            sentiment: 0.2
        },
        parameters: {
            riskTolerance: "medium"
        }
    }
    
    # Define static context
    staticContext = """
    Decision rules:
    1. Buy when sentiment is positive AND news contains no major warnings AND market trend is upward
    2. Sell when two or more inputs suggest negative outlook
    3. Hold otherwise
    """
    
    # Step 3: Create multi-input adaptor
    financialAdaptor = new MultiInputAdaptor(
        name="FinancialDecision",
        inputSources=[newsProvider, marketDataProvider, sentimentAdaptor],
        coreLLM="gpt-4",
        staticContext=staticContext,
        config=config,
        outputFormat="StringAndBool"
    )
    
    return financialAdaptor
```

### 3. Creating a Chained Adaptor

```
# Example: Creating a complete trading pipeline
function createTradingPipeline():
    # Step 1: Create first adaptor in chain
    financialAdaptor = createFinancialDecisionAdaptor()
    
    # Step 2: Create second adaptor in chain
    staticContext = "Format all decisions in uppercase and include timestamp in ISO format."
    
    refinementAdaptor = new SingleInputAdaptor(
        name="Refinement",
        inputSource=null,  # Takes input from previous adaptor in chain
        coreLLM=null,      # No additional LLM needed
        staticContext=staticContext,
        config={
            formatOptions: {
                uppercase: true,
                addTimestamp: true
            }
        },
        outputFormat="StringAndBool"
    )
    
    # Step 3: Create chained adaptor
    tradingPipeline = new ChainedAdaptor(
        name="TradingPipeline",
        adaptorChain=[financialAdaptor, refinementAdaptor],
        config={},
        outputFormat="StringAndBool"
    )
    
    return tradingPipeline
```

## Best Practices for Complex Adaptor Graphs

1. **Modularize Your Design**: Break complex logic into smaller, specialized adaptors
2. **Reuse Adaptors**: Create adaptors that can be reused in multiple inputs
3. **Document Dependencies**: Clearly document the input/output relationships between components
4. **Test Each Component**: Test each provider and adaptor individually before connecting them
5. **Visualize Your Graph**: Use diagrams to visualize the flow of data through your graph
6. **Optimize Static Context**: Keep static context concise and relevant to the specific task
7. **Select Appropriate LLMs**: Choose the right core LLM based on the complexity of the task

## Example: Building a Complete Trading System

Here's a complete example showing how to build a trading system that analyzes news, market data, and sentiment to make trading decisions:

```
# Step 1: Create providers
newsProvider = new Provider("NewsAnalysis", "gpt-4", "news-api-endpoint")
marketDataProvider = new Provider("MarketData", "data-api", "market-api-endpoint")
sentimentProvider = new Provider("SentimentAnalysis", "bert", "sentiment-api-endpoint")

# Step 2: Create basic adaptors
sentimentAdaptor = new SingleInputAdaptor(
    name="SentimentDecision",
    inputSource=sentimentProvider,
    coreLLM="gpt-3.5",
    staticContext="Focus on financial sentiment in news headlines.",
    config={ threshold: 0.7 },
    outputFormat="Bool"
)

newsAdaptor = new SingleInputAdaptor(
    name="NewsAnalysis",
    inputSource=newsProvider,
    coreLLM="gpt-4",
    staticContext="Extract key events: earnings, mergers, acquisitions, leadership changes.",
    config={ keywords: ["earnings", "merger", "acquisition"] },
    outputFormat="StringAndBool"
)

# Step 3: Create a multi-input adaptor combining results
tradingDecisionAdaptor = new MultiInputAdaptor(
    name="TradingStrategy",
    inputSources=[newsAdaptor, sentimentAdaptor, marketDataProvider],
    coreLLM="gpt-4",
    staticContext="Apply standard trading principles with medium risk tolerance.",
    config={
        threshold: 0.6,
        weights: { news: 0.4, sentiment: 0.3, marketData: 0.3 },
        parameters: { riskTolerance: "medium" }
    },
    outputFormat="StringAndBool"
)

# Step 4: Create refinement adaptor for final formatting
refinementAdaptor = new SingleInputAdaptor(
    name="Refinement",
    inputSource=null,  # No provider needed
    coreLLM=null,      # No additional LLM needed
    staticContext="Format as actionable trading advice with timestamp.",
    config={ formatOptions: { uppercase: true, addTimestamp: true } },
    outputFormat="StringAndBool"
)

# Step 5: Chain everything together
tradingPipeline = new ChainedAdaptor(
    name="TradingPipeline",
    adaptorChain=[tradingDecisionAdaptor, refinementAdaptor],
    config={},
    outputFormat="StringAndBool"
)

# Step 6: Use the system
input = "Tesla stock performance after earnings report"
result = tradingPipeline.process(input)
# Example result: { text: "BUY BASED ON POSITIVE EARNINGS [2023-05-01T12:34:56]", decision: true }

# Execution flow:
# 1. Input goes to newsProvider, sentimentProvider, and marketDataProvider
# 2. newsProvider output → newsAdaptor (applies core LLM and static context)
# 3. sentimentProvider output → sentimentAdaptor (applies core LLM and static context)
# 4. newsAdaptor, sentimentAdaptor, marketDataProvider outputs → tradingDecisionAdaptor
# 5. tradingDecisionAdaptor output → refinementAdaptor
# 6. refinementAdaptor produces final result
```

## Debugging Adaptor Graphs

When debugging adaptor graphs, follow these steps:

1. **Trace Data Flow**: Trace how data flows through the graph
2. **Log Intermediate Results**: Log the output of each component
3. **Check Compatibility**: Verify that each component receives compatible inputs
4. **Test Edge Cases**: Test the graph with edge cases and unexpected inputs
5. **Validate Final Output**: Ensure the final output meets the expected format and quality
6. **Inspect LLM Interactions**: Review how core LLMs are processing the inputs
7. **Verify Static Context Usage**: Confirm static context is properly applied

## Conclusion

By following this guide, you should now be able to create complex adaptor graphs that combine various providers and adaptors to process and format data for blockchain applications. Remember to design your graphs with modularity, reusability, and clear data flow in mind. The inclusion of core LLMs and static context provides additional flexibility and intelligence to your adaptors, enabling more sophisticated processing and decision-making capabilities. 