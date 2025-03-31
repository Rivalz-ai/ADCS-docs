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
   - **Input**: One or more provider/adaptor outputs as data sources
   - **Core LLM**: Optional AI model used by the adaptor for processing or reasoning
   - **Static Context**: Predefined information or rules provided to the adaptor
   - **Output Format**: Structured data in a specific format for blockchain applications (e.g., `BoolOutput`, `Uint256Output`, `StringAndBoolOutput`, `BytesOutput`)

   ```
   # Adaptor attribute structure
   Adaptor:
       id            # Unique identifier for the adaptor instance
       name          # Descriptive name for the adaptor
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

For multi-input adaptors, any aggregation logic should be specified in the static context. By default, text outputs are combined using simple concatenation with appropriate separators. For more complex aggregation needs, developers should provide explicit instructions in the static context of the adaptor.

## Creating Adaptor Graphs

Adaptors and providers can be connected in various configurations to create processing graphs. When designing these graphs, follow these rules:

### Rules for Creating Adaptor Graphs

1. **Input/Output Compatibility**: Ensure that each adaptor receives inputs in the format it expects
2. **Type Safety**: Verify that the output format of a provider/adaptor matches the expected input format of the next adaptor in the chain
3. **Execution Order**: Consider the order of execution in the graph, especially for multi-input adaptors
4. **Error Handling**: Include strategies for handling failures at any point in the graph
5. **Circular Dependencies**: Avoid creating circular dependencies between adaptors

### Functions for Graph Construction

When creating a graph of adaptors and providers, you'll need to consider these key functions:

- **Compatibility Validation**: Functions to validate that the output format of a source component is compatible with the input requirements of a target component
- **Execution Order Resolution**: Algorithms (like topological sort) to determine the correct order of execution for components in the graph
- **Node Execution**: Functions to execute individual nodes in the graph, handling different processing for providers versus adaptors
- **Graph Execution**: Functions to execute the entire graph, gathering inputs, processing each node in the correct order, and returning results

## Step-by-Step Guide to Creating Adaptors

### 1. Creating a Single Input Adaptor

To create a single input adaptor, follow these steps:

1. **Define the Provider**: Create or reference an existing provider that will supply input data
2. **Configure the Adaptor**: Define configuration parameters like thresholds and output mappings
3. **Define Static Context**: Provide instructions or rules for processing the input data
4. **Create the Adaptor**: Instantiate the adaptor with the provider as input, optionally specifying a core LLM for additional processing

### 2. Creating a Multi-Input Adaptor

To create a multi-input adaptor, follow these steps:

1. **Define Input Sources**: Create or reference existing providers and adaptors that will supply input data
2. **Configure the Adaptor**: Define configuration parameters including weights for different input sources
3. **Define Aggregation Instructions**: Specify in the static context how inputs should be combined
4. **Create the Adaptor**: Instantiate the adaptor with multiple input sources, specifying how to process and aggregate these inputs

### 3. Creating a Chained Adaptor

To create a chained adaptor, follow these steps:

1. **Create Component Adaptors**: Define or reference the adaptors that will be chained together
2. **Define Processing Order**: Determine the sequence in which adaptors will process the data
3. **Configure the Chain**: Specify any special handling required between stages
4. **Create the Adaptor**: Instantiate the chained adaptor, ensuring each adaptor in the chain receives compatible inputs

## Best Practices for Complex Adaptor Graphs

1. **Modularize Your Design**: Break complex logic into smaller, specialized adaptors
2. **Reuse Adaptors**: Create adaptors that can be reused in multiple inputs
3. **Document Dependencies**: Clearly document the input/output relationships between components
4. **Test Each Component**: Test each provider and adaptor individually before connecting them
5. **Visualize Your Graph**: Use diagrams to visualize the flow of data through your graph
6. **Optimize Static Context**: Keep static context concise and relevant to the specific task
7. **Select Appropriate LLMs**: Choose the right core LLM based on the complexity of the task
8. **Define Aggregation in Static Context**: For multi-input adaptors, clearly specify aggregation rules in the static context
9. **Handle Edge Cases**: Include logic for handling missing, conflicting, or low-confidence inputs
10. **Use Consistent IDs**: Create unique, descriptive IDs for all providers and adaptors

## Debugging Adaptor Graphs

When debugging adaptor graphs, follow these steps:

1. **Trace Data Flow**: Trace how data flows through the graph
2. **Log Intermediate Results**: Log the output of each component
3. **Check Compatibility**: Verify that each component receives compatible inputs
4. **Test Edge Cases**: Test the graph with edge cases and unexpected inputs
5. **Validate Final Output**: Ensure the final output meets the expected format and quality
6. **Inspect LLM Interactions**: Review how core LLMs are processing the inputs
7. **Verify Static Context Usage**: Confirm static context is properly applied
8. **Check Aggregation Logic**: Ensure that multi-input aggregation is performed according to static context instructions
9. **Validate ID References**: Ensure all ID references are correct and components exist

## Conclusion

By following this guide, you should now be able to create complex adaptor graphs that combine various providers and adaptors to process and format data for blockchain applications. Remember to design your graphs with modularity, reusability, and clear data flow in mind. The inclusion of core LLMs and static context provides additional flexibility and intelligence to your adaptors, enabling more sophisticated processing and decision-making capabilities. 

For multi-input adaptors, always include clear aggregation instructions in the static context to ensure inputs are combined correctly according to your specific requirements. By default, simple text concatenation is used for combining outputs, but you can specify more complex aggregation logic in the static context when needed.

Always use unique, consistent IDs to ensure proper referencing between components in your adaptor graph. 