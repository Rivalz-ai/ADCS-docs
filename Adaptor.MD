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

#### Provider Implementation (Pseudocode)

```
# Provider Output Format - Raw inference data
ProviderOutput:
    result      # The main inference result (typically unstructured data)
    confidence  # Confidence score (0.0-1.0)
    metadata:   # Additional information
        model
        timestamp
        processingTime

class Provider:
    constructor(name, model, endpoint)
    
    function process(input) -> ProviderOutput:
        # Call AI model API with input
        response = callAPI(endpoint, input)
        
        # Return raw inference output without specific format constraints
        return ProviderOutput {
            result: response.prediction,  # Could be any data type (text, array, object)
            confidence: response.confidence,
            metadata: {
                model: model,
                timestamp: currentTime(),
                processingTime: response.timing
            }
        }
```

### 2. Adaptors

#### Mathematical Representation

##### Single Context Adaptors
An Adaptor can take only one provider as context, add custom logic, and produce a formatted result. We can express this mathematically as:

$$A_{single}: O_P \times C \rightarrow O_A$$

where:
- $O_P$ is the output domain of a single Provider $P$
- $C$ represents the custom logic and parameters added by the Adaptor
- $O_A$ is the output domain of the Adaptor

For a specific Provider $P_i$, the single context Adaptor operation can be expressed as:

$$O_A = A_{single}(P_i(I), C)$$

##### Multi-Context Adaptors
Adaptors can also take multiple inputs from different sources. We define a multi-context Adaptor ($A_{multi}$) as a function:

$$A_{multi}: (O_{P_1} \times O_{P_2} \times ... \times O_{P_n}) \times C \rightarrow O_A$$

Adaptors can also compose with other Adaptors, creating a chain of transformations:

$$A_2 \circ A_1: O_P \rightarrow O_{A_1} \rightarrow O_{A_2}$$

#### Adaptor Output Types

Adaptors convert unstructured Provider outputs into specific, structured formats needed by blockchain applications:

```
# Some example output formats supported by ADCS

# Boolean output format
BoolOutput:
    value: bool  # True/False decision

# Numerical output format
Uint256Output:
    value: uint256  # Numerical value for blockchain

# String and Boolean combined output format
StringAndBoolOutput:
    text: string  # Textual explanation
    decision: bool  # True/False decision 

# Bytes output format
BytesOutput:
    data: bytes  # Raw binary data
```

#### Adaptor Implementation (Pseudocode)

```
# Base Adaptor (abstract)
class Adaptor:
    constructor(name, config, outputType)  # outputType specifies the format
    
    # Common utility methods
    function formatResult(result, options)
    function generateMetadata()
    
    # Abstract method that must be implemented by subclasses
    function process(input) -> [Formatted Output] = abstract

# Single Context Adaptor
class SingleContextAdaptor extends Adaptor:
    constructor(name, provider, config, outputType)
    
    function process(input) -> [Formatted Output]:
        # Get provider output (unstructured)
        providerOutput = provider.process(input)
        
        # Apply custom logic with configuration
        decision = evaluateConfidence(providerOutput, config.threshold)
        
        # Format result based on specified output type
        if outputType == "StringAndBool":
            # Example: Format as [string, bool]
            explanation = generateExplanation(providerOutput)
            return StringAndBoolOutput {
                text: explanation,
                decision: decision
            }
        elif outputType == "Bool":
            # Example: Format as simple boolean
            return BoolOutput {
                value: decision
            }
        elif outputType == "Uint256":
            # Example: Format as numerical value
            score = calculateScore(providerOutput) * 10^18  # Convert to fixed-point
            return Uint256Output {
                value: score
            }
        # Other output types...

# Multi-Context Adaptor
class MultiContextAdaptor extends Adaptor:
    constructor(name, contexts, config, outputType)
    
    function process(input) -> [Formatted Output]:
        # Collect outputs from all context sources
        contextOutputs = []
        for each context in contexts:
            output = context.process(input)
            contextOutputs.add({ source: typeOf(context), output: output })
        
        # Aggregate results
        aggregatedResult = combineResults(contextOutputs)
        decision = votingDecision(contextOutputs)
        
        # Format based on specified output type
        if outputType == "StringAndBool":
            explanation = combineExplanations(contextOutputs)
            return StringAndBoolOutput {
                text: explanation,
                decision: decision
            }
        elif outputType == "Bool":
            return BoolOutput {
                value: decision
            }
        # Other format conversions...

# Chained Adaptor (function composition)
class ChainedAdaptor extends Adaptor:
    constructor(name, adaptorChain, config, outputType)
    
    function process(input) -> [Formatted Output]:
        # First adaptor can have any output type
        result = adaptorChain[0].process(input)
        
        # Pass through chain, with intermediate conversions if needed
        for i = 1 to adaptorChain.length - 1:
            # Convert previous result to format expected by next adaptor
            nextInput = convertFormat(result)
            result = adaptorChain[i].process(nextInput)
        
        # Ensure final result is in the specified output format
        return convertToOutputType(result, outputType)
```

### 3. System Composition

The complete ADCS system can be represented as a composition of Providers and Adaptors, working together to deliver AI-powered inference to blockchain applications. This reflects the mathematical expression:

$$ADCS: I \xrightarrow{P} O_P \xrightarrow{A} O_A \rightarrow Blockchain$$

#### Usage Example (Pseudocode)

```
# Create providers (generate unstructured inference data)
newsProvider = new Provider("NewsAnalysis", "gpt-4", "news-api-endpoint")
sentimentProvider = new Provider("SentimentAnalysis", "bert", "sentiment-api-endpoint")

# Create adaptors with specific output formats
sentimentAdaptor = new SingleContextAdaptor(
    "SentimentDecision", 
    sentimentProvider,
    { threshold: 0.7 },
    outputType: "Bool"  # Format as boolean
)

tradingAdaptor = new MultiContextAdaptor(
    "TradingStrategy",
    [newsProvider, sentimentAdaptor],
    { threshold: 0.6, parameters: { riskTolerance: "medium" } },
    outputType: "StringAndBool"  # Format as [string, bool]
)

refinementAdaptor = new SingleContextAdaptor(
    "Refinement",
    newsProvider,
    { formatOptions: { uppercase: true } },
    outputType: "StringAndBool"  # Format as [string, bool]
)

tradingPipeline = new ChainedAdaptor(
    "TradingPipeline",
    [tradingAdaptor, refinementAdaptor],
    { },
    outputType: "StringAndBool"  # Final output format
)

# Use the system - result will be formatted as StringAndBool [string, bool]
result = tradingPipeline.process("Tesla stock performance after earnings report")
# Example result: { text: "BUY BASED ON POSITIVE EARNINGS", decision: true }
```
###Examples of different Adaptors
1. Single Context Adaptor:
[singleContext](images/singleContext.png)

IR goes to provider 1, Adaptor 1 uses that results as the context, then used it custom logic to produce the output.


2. Multi-Context Adaptor:
[multiContext](images/multiContext.png)
Adaoptor 5 takes Output of Adaptor 3 as context
Takes output of  Provider 2 as context
Takes output of provider 3 as context
Takes output of provider 4 as context
Implement its own custom logic to produce the output 5, which is the final output

With this graph we need to trace back to the root of every context to execute each of them.
Then Combine the results to a final context for Adaptor 5 to use.
3. Chained Adaptor:
[chainedAdaptor](images/chained.JPG)
IR goes to Provider 1, creates output 1 
Ouptput 1 goes to Adaptor1, creates output2, 
Output 2 goes to Adaptor 3, creates output 3, which is the final output











