# ADCS Adaptor Definition Guide: Creating Adaptors Using Configuration Files

## Introduction

This guide demonstrates how to create adaptors in the ADCS (AI-Driven Consensus System) using a declarative approach with YAML or JSON definition files. Similar to The Graph Protocol's subgraph manifests, this approach allows you to define, maintain, and deploy adaptors using configuration files rather than programmatic creation.

## Prerequisites

* Understanding of ADCS adaptor concepts and components
* Access to the ADCS CLI tool (assumed for this guide)
* Knowledge of YAML or JSON formatting
* Access to your ADCS instance

## Adaptor Definition Structure

An adaptor definition file allows you to declare all adaptor attributes in a structured format. You can use either YAML (.yaml) or JSON (.json) format.

### Basic Structure

```yaml
# adaptor.yaml
version: "1.0"  # ADCS definition version
id: "adaptor-name-v1"  # Unique identifier
name: "Human Readable Name"
type: "single-input" | "multi-input" | "chained"  # Adaptor type

# Input sources (providers or adaptors)
input:
  sources: 
    - "provider-id-v1" | "adaptor-id-v1"
    # For multi-input: add additional sources

# Optional core LLM
coreLLM:
  model: "gpt-4" | "gpt-3.5" | null
  parameters:
    temperature: 0.7
    maxTokens: 512
    # Additional model parameters

# Static context
staticContext: |
  Multi-line static context that provides
  instructions or rules for this adaptor's processing.

# Configuration options
config:
  # Configuration parameters vary by adaptor type
  threshold: 0.7
  weights:
    "source-id-1": 0.6
    "source-id-2": 0.4
  # Additional configuration parameters

# Output format
outputFormat: "Bool" | "Uint256" | "StringAndBool" | "BytesOutput"
```

## Step-by-Step Guide

### 1. Creating a Single Input Adaptor Definition

```yaml
# sentiment-adaptor.yaml
version: "1.0"
id: "adaptor-sentiment-decision-v1"
name: "Sentiment Decision"
type: "single-input"

input:
  sources:
    - "provider-sentiment-v1"

coreLLM:
  model: "gpt-4"
  parameters:
    temperature: 0.3
    maxTokens: 256

staticContext: |
  Financial sentiment analysis should focus on words that indicate market trends.
  Positive indicators include: growth, profit, exceeded expectations, bullish.
  Negative indicators include: loss, decline, missed expectations, bearish.

config:
  threshold: 0.7
  outputMapping:
    positive: true
    negative: false
    neutral: false

outputFormat: "Bool"
```

### 2. Creating a Multi-Input Adaptor Definition

```yaml
# financial-decision.yaml
version: "1.0"
id: "adaptor-financial-decision-v1"
name: "Financial Decision"
type: "multi-input"

input:
  sources:
    - "provider-news-v1"
    - "provider-market-data-v1"
    - "adaptor-sentiment-decision-v1"

coreLLM:
  model: "gpt-4"
  parameters:
    temperature: 0.2
    maxTokens: 1024

staticContext: |
  Decision rules:
  1. Buy when sentiment is positive AND news contains no major warnings AND market trend is upward
  2. Sell when two or more inputs suggest negative outlook
  3. Hold otherwise

config:
  threshold: 0.6
  weights:
    "provider-news-v1": 0.3
    "provider-market-data-v1": 0.5
    "adaptor-sentiment-decision-v1": 0.2
  parameters:
    riskTolerance: "medium"
  aggregationMethod: "weightedVoting"
  conflictResolution: "highestConfidence"
  normalization:
    enabled: true
    method: "minMaxScaling"

outputFormat: "StringAndBool"
```

### 3. Creating a Chained Adaptor Definition

```yaml
# trading-pipeline.yaml
version: "1.0"
id: "adaptor-trading-pipeline-v1"
name: "Trading Pipeline"
type: "chained"

input:
  chain:
    - "adaptor-financial-decision-v1"
    - "adaptor-refinement-v1"

config:
  passthrough:
    metadata: true

outputFormat: "StringAndBool"
```

### 4. Investment Risk Assessment Example

This example shows how to create a risk assessment adaptor using existing adaptors:

```yaml
# investment-risk.yaml
version: "1.0"
id: "adaptor-investment-risk-v1"
name: "Investment Risk Assessment"
type: "multi-input"

input:
  sources:
    - "adaptor-financial-risk-v1"
    - "adaptor-news-risk-v1"
    - "adaptor-market-risk-v1"

coreLLM: null  # No LLM needed for this adaptor

staticContext: |
  Investment risk assessment guidelines:
  1. Financial risk factors take precedence, especially debt levels
  2. Recent negative news may temporarily increase risk assessment
  3. Overall market conditions provide context but are less decisive
  4. When inputs conflict, prefer financial data unless confidence is low

config:
  weights:
    "adaptor-financial-risk-v1": 0.5
    "adaptor-news-risk-v1": 0.3
    "adaptor-market-risk-v1": 0.2
  aggregationMethod: "weightedAverage"
  riskThresholds:
    low: 30
    medium: 60
    high: 100
  conflictResolution:
    method: "maxConfidence"
    significantDifference: 40
  normalization:
    enabled: true
    method: "minMaxScaling"

outputFormat: "StringAndUint256"
```

## Deployment Process

Similar to The Graph Protocol's deployment workflow, deploying an adaptor involves the following steps:

### 1. Initialize Adaptor

```bash
adcs init --name "Investment Risk Assessment" --template "multi-input"
```

This creates the directory structure:
```
investment-risk-assessment/
├── adaptor.yaml         # Main definition file
├── functions/           # Optional custom functions
│   └── aggregation.js   # Custom aggregation logic
└── README.md            # Documentation
```

### 2. Edit the Definition File

Customize the `adaptor.yaml` file with your adaptor configuration.

### 3. Add Custom Functions (Optional)

For complex aggregation logic or transformations, you can add custom JavaScript functions:

```javascript
// functions/aggregation.js
function aggregateRiskScores(inputs, config) {
  // Implementation of aggregation logic
  // This is called by the adaptor runtime
  // ...
  
  return {
    score: finalScore,
    assessment: `${riskCategory} RISK (${finalScore.toFixed(1)}/100): ${explanation}`
  };
}

module.exports = {
  aggregateRiskScores
};
```

Reference custom functions in your adaptor.yaml:

```yaml
config:
  # ...other config
  customAggregation:
    function: "aggregateRiskScores"
    path: "./functions/aggregation.js"
```

### 4. Validate the Definition

```bash
adcs validate ./investment-risk-assessment
```

### 5. Deploy the Adaptor

```bash
adcs deploy ./investment-risk-assessment
```

### 6. Test the Adaptor

```bash
adcs test adaptor-investment-risk-v1 --input "TSLA"
```

## JSON Definition Alternative

For those who prefer JSON over YAML, here's the investment risk example in JSON format:

```json
{
  "version": "1.0",
  "id": "adaptor-investment-risk-v1",
  "name": "Investment Risk Assessment",
  "type": "multi-input",
  "input": {
    "sources": [
      "adaptor-financial-risk-v1",
      "adaptor-news-risk-v1",
      "adaptor-market-risk-v1"
    ]
  },
  "coreLLM": null,
  "staticContext": "Investment risk assessment guidelines:\n1. Financial risk factors take precedence, especially debt levels\n2. Recent negative news may temporarily increase risk assessment\n3. Overall market conditions provide context but are less decisive\n4. When inputs conflict, prefer financial data unless confidence is low",
  "config": {
    "weights": {
      "adaptor-financial-risk-v1": 0.5,
      "adaptor-news-risk-v1": 0.3,
      "adaptor-market-risk-v1": 0.2
    },
    "aggregationMethod": "weightedAverage",
    "riskThresholds": {
      "low": 30,
      "medium": 60,
      "high": 100
    },
    "conflictResolution": {
      "method": "maxConfidence",
      "significantDifference": 40
    },
    "normalization": {
      "enabled": true,
      "method": "minMaxScaling"
    }
  },
  "outputFormat": "StringAndUint256"
}
```

## Managing Adaptors

### Version Control

Store adaptor definitions in a version control system (Git) to track changes and collaborate with team members:

```bash
git init
git add adaptor.yaml functions/
git commit -m "Initial adaptor definition"
```

### Updating Adaptors

To update an existing adaptor:

1. Edit the definition file
2. Increment the version in a version field or in the ID
3. Validate and deploy

```bash
# Update an adaptor
adcs update adaptor-investment-risk-v1 --file ./investment-risk-assessment/adaptor.yaml
```

### Listing Deployed Adaptors

```bash
adcs list adaptors
```

## Best Practices

1. **Use Semantic Versioning**: Include version numbers in adaptor IDs (e.g., `adaptor-name-v1`) and increment when making changes.

2. **Document Dependencies**: Clearly document input sources and requirements:
   ```yaml
   # At the top of your adaptor.yaml
   dependencies:
     - "adaptor-financial-risk-v1"
     - "adaptor-news-risk-v1"
     - "provider-market-data-v1"
   ```

3. **Validate Compatibility**: Ensure input sources provide compatible outputs:
   ```bash
   adcs check-compatibility adaptor-investment-risk-v1
   ```

4. **Create Adaptor Templates**: For common patterns, create reusable templates:
   ```bash
   adcs init --template risk-assessment
   ```

5. **Visualize Adaptor Graphs**: Generate visual representations of your adaptor relationships:
   ```bash
   adcs visualize adaptor-investment-risk-v1 --output graph.png
   ```

6. **Modular Configurations**: Split complex configurations into separate files:
   ```yaml
   # adaptor.yaml
   config:
     include: "./config/weights.yaml"
   ```

7. **Testing Adaptors**: Create test cases for adaptors:
   ```yaml
   # tests/test1.yaml
   input: "TSLA"
   expectedOutput:
     score: 42.7
     assessment: "MEDIUM RISK"
   ```

## Conclusion

By using definition files for adaptors, you can create a more maintainable, version-controlled, and collaborative workflow for your ADCS system. This approach makes it easier to document, deploy, and manage complex adaptor configurations while reducing the need for programmatic creation. The declarative nature of YAML/JSON definitions provides a clearer picture of adaptor capabilities and requirements, making it easier for developers to understand and work with adaptors in your system.

Similar to how The Graph Protocol uses subgraph manifests to define data sources and transformations, these adaptor definitions create a structured approach to building AI-driven consensus systems that can be easily shared, reused, and composed into more complex applications. 