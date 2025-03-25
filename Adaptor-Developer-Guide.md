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

### Input Aggregation Rules

When an adaptor takes multiple adaptors (or providers) as input, it needs to aggregate their outputs. Here are some basic rules for effective input aggregation:

1. **Type Compatibility**
   - Ensure all input formats can be processed together
   - Convert incompatible formats before aggregation

2. **Weighting Strategies**
   - **Equal Weighting**: Treat all inputs with equal importance
   - **Confidence-Based Weighting**: Weight inputs based on their confidence scores
   - **Custom Weighting**: Assign predefined weights to different input sources
   - **Dynamic Weighting**: Adjust weights based on input quality or relevance

3. **Aggregation Methods**
   - **Simple Methods**:
     - **Voting**: Use majority decision for boolean outputs
     - **Weighted Voting**: Account for input weights in voting process
     - **Averaging**: Calculate weighted average for numerical outputs
     - **Concatenation**: Combine text outputs with appropriate separators
     - **Thresholding**: Apply threshold to weighted sum of inputs
   
   - **Selection Methods**:
     - **Priority Selection**: Select outputs based on a predefined priority order
     - **Max Confidence**: Select the input with highest confidence score
     - **First Valid**: Use the first valid input that meets criteria
     - **Ensemble Selection**: Choose a subset of inputs based on quality metrics
   
   - **Combination Methods**:
     - **Logical Operations**: Apply AND, OR, XOR operations to boolean inputs
     - **Mathematical Functions**: Apply min, max, median, or custom functions
     - **Fuzzy Logic**: Use fuzzy logic operators for imprecise reasoning
     - **Statistical Methods**: Apply statistical aggregation (mean, variance, etc.)
   
   - **Advanced Methods**:
     - **LLM-Based Reasoning**: Use the Core LLM to reason over all inputs
     - **Decision Trees**: Apply decision tree logic to combine inputs
     - **Bayesian Fusion**: Combine inputs using Bayesian probability
     - **Neural Aggregation**: Use neural networks to learn optimal combination
     - **Rule-Based Systems**: Apply expert-defined rules for aggregation
     - **Multi-criteria Decision Making**: Formalize decision process with MCDM techniques
     - **Consensus Mechanisms**: Apply blockchain-style consensus to inputs
     - **Time-Series Aggregation**: Account for temporal aspects when aggregating
     - **Spatial Aggregation**: Consider spatial relationships between inputs

4. **Conflict Resolution**
   - Define clear rules for handling contradictory inputs
   - Implement tiebreaker mechanisms for voting scenarios
   - Consider confidence thresholds for including/excluding inputs

5. **Input Transformation**
   - Normalize inputs to a common scale before aggregation
   - Extract relevant features from complex input structures
   - Apply filters to remove outliers or low-quality inputs

```
# Example input aggregation function
function aggregateInputs(inputs, config):
    # Initialize aggregation result
    aggregatedResult = initializeResult(config.outputFormat)
    
    # Apply weights to inputs
    weightedInputs = []
    for input in inputs:
        weight = determineWeight(input, config.weights)
        weightedInputs.append({ source: input.source, data: input.data, weight: weight })
    
    # Select aggregation method based on output type
    if config.outputFormat == "Bool":
        # Weighted voting for boolean outputs
        aggregatedResult.value = weightedVoting(weightedInputs)
    
    elif config.outputFormat == "Uint256":
        # Weighted average for numerical outputs
        aggregatedResult.value = weightedAverage(weightedInputs)
    
    elif config.outputFormat == "StringAndBool":
        # Complex aggregation for mixed outputs
        aggregatedResult.decision = weightedVoting(weightedInputs, 'decision')
        aggregatedResult.text = combineTextWithLLM(weightedInputs, config.coreLLM)
    
    # Include metadata about the aggregation
    aggregatedResult.metadata = {
        sources: inputs.map(i => i.source),
        method: config.aggregationMethod,
        confidence: calculateAggregatedConfidence(weightedInputs)
    }
    
    return aggregatedResult
```

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
        id="provider-sentiment-v1",       # Unique provider ID
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
        id="adaptor-sentiment-decision-v1",  # Unique adaptor ID
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
    newsProvider = new Provider(
        id="provider-news-v1",
        name="NewsAnalysis", 
        model="gpt-4", 
        endpoint="news-api-endpoint"
    )
    
    marketDataProvider = new Provider(
        id="provider-market-data-v1",
        name="MarketData", 
        model="data-api", 
        endpoint="market-api-endpoint"
    )
    
    sentimentAdaptor = createSentimentAdaptor()  # Already has ID "adaptor-sentiment-decision-v1"
    
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
        },
        # Specify aggregation method
        aggregationMethod: "weightedVoting",
        # Define conflict resolution strategy
        conflictResolution: "highestConfidence",
        # Input normalization settings
        normalization: {
            enabled: true,
            method: "minMaxScaling"
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
        id="adaptor-financial-decision-v1",  # Unique adaptor ID
        name="FinancialDecision",
        inputSources=[
            newsProvider.id,              # Reference by ID
            marketDataProvider.id,        # Reference by ID
            sentimentAdaptor.id           # Reference by ID
        ],
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
    financialAdaptor = createFinancialDecisionAdaptor()  # Has ID "adaptor-financial-decision-v1"
    
    # Step 2: Create second adaptor in chain
    staticContext = "Format all decisions in uppercase and include timestamp in ISO format."
    
    refinementAdaptor = new SingleInputAdaptor(
        id="adaptor-refinement-v1",      # Unique adaptor ID
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
        id="adaptor-trading-pipeline-v1",  # Unique adaptor ID
        name="TradingPipeline",
        adaptorChain=[
            financialAdaptor.id,          # Reference by ID
            refinementAdaptor.id          # Reference by ID
        ],
        config={},
        outputFormat="StringAndBool"
    )
    
    return tradingPipeline
```

## Example: Combining Existing Adaptors with Simple Aggregation

Let's create a practical example of building a "Risk Assessment Adaptor" that combines multiple existing adaptors and uses simple aggregation rules to determine an overall risk score for a potential investment.

### Step 1: Assume we already have these existing adaptors

```
# These adaptors have already been created elsewhere in our application

# Evaluates news sentiment for a company
newsRiskAdaptor = SingleInputAdaptor(
    id="adaptor-news-risk-v1",           # Unique adaptor ID
    name="NewsRiskAnalysis",
    inputSource="provider-news-v1",      # Reference provider by ID
    coreLLM="gpt-4",
    staticContext="Analyze news for risk factors related to the company.",
    config={ riskFactorKeywords: ["lawsuit", "investigation", "fine", "recall"] },
    outputFormat="Uint256"  # 0-100 risk score
)

# Evaluates financial indicators
financialRiskAdaptor = SingleInputAdaptor(
    id="adaptor-financial-risk-v1",      # Unique adaptor ID
    name="FinancialRiskAnalysis",
    inputSource="provider-financial-data-v1",  # Reference provider by ID
    coreLLM=null,  # Uses rule-based processing instead of an LLM
    staticContext="Calculate financial risk based on debt ratio, liquidity, and volatility.",
    config={ thresholds: { debtRatio: 0.5, quickRatio: 1.0 } },
    outputFormat="Uint256"  # 0-100 risk score
)

# Evaluates market conditions
marketRiskAdaptor = SingleInputAdaptor(
    id="adaptor-market-risk-v1",         # Unique adaptor ID
    name="MarketRiskAnalysis",
    inputSource="provider-market-data-v1",  # Reference provider by ID
    coreLLM=null,
    staticContext="Assess overall market conditions affecting investments.",
    config={ indicators: ["VIX", "interest_rates", "sector_performance"] },
    outputFormat="Uint256"  # 0-100 risk score
)
```

### Step 2: Create our composite risk assessment adaptor

```
# Now we'll create a new adaptor that combines all three risk assessments
function createInvestmentRiskAdaptor():
    # Define the aggregation configuration
    config = {
        # Weighted average configuration
        weights: {
            financial: 0.5,    # Financial data has highest importance
            news: 0.3,         # News sentiment is second
            market: 0.2        # Market conditions is third
        },
        
        # Aggregation method
        aggregationMethod: "weightedAverage",
        
        # Risk thresholds for the final output
        riskThresholds: {
            low: 30,           # Risk scores below 30 are low risk
            medium: 60,        # Risk scores 30-60 are medium risk
            high: 100          # Risk scores above 60 are high risk
        },
        
        # Conflict resolution if the inputs disagree significantly 
        conflictResolution: {
            method: "maxConfidence",
            significantDifference: 40  # Point spread that triggers conflict resolution
        },
        
        # Input normalization ensures all scales are comparable
        normalization: {
            enabled: true,
            method: "minMaxScaling"
        }
    }
    
    # Define static context
    staticContext = """
    Investment risk assessment guidelines:
    1. Financial risk factors take precedence, especially debt levels
    2. Recent negative news may temporarily increase risk assessment
    3. Overall market conditions provide context but are less decisive
    4. When inputs conflict, prefer financial data unless confidence is low
    """
    
    # Create the composite multi-input adaptor
    riskAdaptor = new MultiInputAdaptor(
        id="adaptor-investment-risk-v1",    # Unique adaptor ID
        name="InvestmentRiskAssessment",
        inputSources=[
            "adaptor-financial-risk-v1",    # Reference by ID
            "adaptor-news-risk-v1",         # Reference by ID
            "adaptor-market-risk-v1"        # Reference by ID
        ],
        coreLLM=null,  # Using simple weighted average, no LLM needed
        staticContext=staticContext,
        config=config,
        outputFormat="StringAndUint256"  # Returns risk score and textual assessment
    )
    
    return riskAdaptor
```

### Step 3: Implement our custom aggregation logic

```
# This demonstrates the internal aggregation algorithm
function aggregateRiskScores(inputs, config):
    # Collect and normalize all input scores
    scores = []
    for input in inputs:
        score = {
            source: input.source,
            value: input.value,            # The risk score (0-100)
            confidence: input.confidence,  # How confident the adaptor is (0.0-1.0)
            weight: config.weights[input.source]  # Predefined weight for this source
        }
        scores.push(score)
    
    # Check if inputs have significant disagreement
    minScore = min(scores.map(s => s.value))
    maxScore = max(scores.map(s => s.value))
    
    if (maxScore - minScore > config.conflictResolution.significantDifference):
        # Inputs disagree significantly, use conflict resolution
        if config.conflictResolution.method == "maxConfidence":
            # Sort by confidence and return the highest confidence score
            scores.sort((a, b) => b.confidence - a.confidence)
            finalScore = scores[0].value
            explanation = `Using ${scores[0].source} assessment due to highest confidence (${scores[0].confidence}).`
        else:
            # Default to weighted average
            finalScore = weightedAverage(scores)
            explanation = "Using weighted average due to significant disagreement."
    else:
        # Normal case: calculate weighted average
        finalScore = sum(scores.map(s => s.value * s.weight * s.confidence)) / 
                     sum(scores.map(s => s.weight * s.confidence))
        explanation = "Calculated weighted average of all risk assessments."
    
    # Determine risk category
    let riskCategory
    if (finalScore < config.riskThresholds.low):
        riskCategory = "LOW"
    elif (finalScore < config.riskThresholds.medium):
        riskCategory = "MEDIUM"
    else:
        riskCategory = "HIGH"
    
    # Return both numerical score and text assessment
    return {
        score: finalScore,
        assessment: `${riskCategory} RISK (${finalScore.toFixed(1)}/100): ${explanation}`
    }
```

### Step 4: Use the risk assessment adaptor

```
# Create the risk assessment adaptor
investmentRiskAdaptor = createInvestmentRiskAdaptor()  # Has ID "adaptor-investment-risk-v1"

# Process a company for risk assessment
result = investmentRiskAdaptor.process("TSLA")

# Example output:
# {
#   score: 42.7,
#   assessment: "MEDIUM RISK (42.7/100): Calculated weighted average of all risk assessments."
# }
```

### Step 5: Extend with additional outputs

```
# We could further chain this with a recommendation adaptor
recommendationAdaptor = new SingleInputAdaptor(
    id="adaptor-investment-recommendation-v1",  # Unique adaptor ID
    name="InvestmentRecommendation",
    inputSource="adaptor-investment-risk-v1",   # Reference by ID
    coreLLM="gpt-3.5",
    staticContext="Generate investment recommendations based on risk assessment.",
    config={
        recommendationTypes: {
            "LOW": "Consider for long-term portfolio.",
            "MEDIUM": "Consider with hedging strategy.",
            "HIGH": "Avoid or limit exposure."
        }
    },
    outputFormat="StringAndBool"  # Recommendation text and buy/don't buy decision
)

# Chain them together
investmentAdvisorPipeline = new ChainedAdaptor(
    id="adaptor-investment-advisor-v1",      # Unique adaptor ID
    name="InvestmentAdvisor",
    adaptorChain=[
        "adaptor-investment-risk-v1",        # Reference by ID
        "adaptor-investment-recommendation-v1"  # Reference by ID
    ],
    config={},
    outputFormat="StringAndBool"
)

# Process a company for investment advice
advice = investmentAdvisorPipeline.process("TSLA")

# Example output:
# {
#   text: "MEDIUM RISK (42.7/100): Consider with hedging strategy. Recent positive financial indicators offset by market volatility.",
#   decision: true  # Still recommends buying with hedging
# }
```

This example demonstrates:
1. **Reusing Existing Adaptors**: We use three specialized risk assessment adaptors as inputs
2. **Simple Aggregation Rules**: We implement weighted averaging with conflict resolution
3. **Practical Application**: The resulting adaptor provides meaningful investment risk scores
4. **Extensibility**: The adaptor can be further chained with other adaptors for more complex workflows

## Best Practices for Complex Adaptor Graphs

1. **Modularize Your Design**: Break complex logic into smaller, specialized adaptors
2. **Reuse Adaptors**: Create adaptors that can be reused in multiple inputs
3. **Document Dependencies**: Clearly document the input/output relationships between components
4. **Test Each Component**: Test each provider and adaptor individually before connecting them
5. **Visualize Your Graph**: Use diagrams to visualize the flow of data through your graph
6. **Optimize Static Context**: Keep static context concise and relevant to the specific task
7. **Select Appropriate LLMs**: Choose the right core LLM based on the complexity of the task
8. **Define Clear Aggregation Rules**: For multi-input adaptors, be explicit about how inputs are combined
9. **Handle Edge Cases**: Include logic for handling missing, conflicting, or low-confidence inputs
10. **Use Consistent IDs**: Create unique, descriptive IDs for all providers and adaptors

## Example: Building a Complete Trading System

Here's a complete example showing how to build a trading system that analyzes news, market data, and sentiment to make trading decisions:

```
# Step 1: Create providers
newsProvider = new Provider(
    id="provider-news-v1",
    name="NewsAnalysis", 
    model="gpt-4", 
    endpoint="news-api-endpoint"
)

marketDataProvider = new Provider(
    id="provider-market-data-v1",
    name="MarketData", 
    model="data-api", 
    endpoint="market-api-endpoint"
)

sentimentProvider = new Provider(
    id="provider-sentiment-v1",
    name="SentimentAnalysis", 
    model="bert", 
    endpoint="sentiment-api-endpoint"
)

# Step 2: Create basic adaptors
sentimentAdaptor = new SingleInputAdaptor(
    id="adaptor-sentiment-decision-v1",
    name="SentimentDecision",
    inputSource="provider-sentiment-v1",  # Reference by ID
    coreLLM="gpt-3.5",
    staticContext="Focus on financial sentiment in news headlines.",
    config={ threshold: 0.7 },
    outputFormat="Bool"
)

newsAdaptor = new SingleInputAdaptor(
    id="adaptor-news-analysis-v1",
    name="NewsAnalysis",
    inputSource="provider-news-v1",  # Reference by ID
    coreLLM="gpt-4",
    staticContext="Extract key events: earnings, mergers, acquisitions, leadership changes.",
    config={ keywords: ["earnings", "merger", "acquisition"] },
    outputFormat="StringAndBool"
)

# Step 3: Create a multi-input adaptor combining results
tradingDecisionAdaptor = new MultiInputAdaptor(
    id="adaptor-trading-strategy-v1",
    name="TradingStrategy",
    inputSources=[
        "adaptor-news-analysis-v1",     # Reference by ID
        "adaptor-sentiment-decision-v1", # Reference by ID
        "provider-market-data-v1"        # Reference by ID
    ],
    coreLLM="gpt-4",
    staticContext="Apply standard trading principles with medium risk tolerance.",
    config={
        threshold: 0.6,
        weights: { news: 0.4, sentiment: 0.3, marketData: 0.3 },
        parameters: { riskTolerance: "medium" },
        aggregationMethod: "llmReasoning",
        conflictResolution: "majorityWithConfidence",
        inputPreprocessing: {
            normalizeScores: true,
            filterLowConfidence: true,
            minimumConfidence: 0.4
        }
    },
    outputFormat="StringAndBool"
)

# Step 4: Create refinement adaptor for final formatting
refinementAdaptor = new SingleInputAdaptor(
    id="adaptor-refinement-v1",
    name="Refinement",
    inputSource=null,  # No provider needed, will take input from chain
    coreLLM=null,      # No additional LLM needed
    staticContext="Format as actionable trading advice with timestamp.",
    config={ formatOptions: { uppercase: true, addTimestamp: true } },
    outputFormat="StringAndBool"
)

# Step 5: Chain everything together
tradingPipeline = new ChainedAdaptor(
    id="adaptor-trading-pipeline-v1",
    name="TradingPipeline",
    adaptorChain=[
        "adaptor-trading-strategy-v1",  # Reference by ID
        "adaptor-refinement-v1"         # Reference by ID
    ],
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
# 4. newsAdaptor, sentimentAdaptor, marketDataProvider outputs → tradingDecisionAdaptor (aggregates inputs)
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
8. **Analyze Input Aggregation**: For multi-input adaptors, verify that inputs are being combined correctly
9. **Validate ID References**: Ensure all ID references are correct and components exist

## Conclusion

By following this guide, you should now be able to create complex adaptor graphs that combine various providers and adaptors to process and format data for blockchain applications. Remember to design your graphs with modularity, reusability, and clear data flow in mind. The inclusion of core LLMs and static context provides additional flexibility and intelligence to your adaptors, enabling more sophisticated processing and decision-making capabilities. When working with multiple inputs, carefully consider your aggregation strategy to ensure that the combined result accurately reflects the information from all sources. Always use unique, consistent IDs to ensure proper referencing between components in your adaptor graph. 