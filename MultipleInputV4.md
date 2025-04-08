# I. Provider

Providers are designed to be reusable components that serve as the foundational data sources that Adaptors can then build upon, transform, and combine. They encapsulate the details of connecting to external data sources and standardize the way data is fetched and formatted within the ADCS system.

Provider is defined by a structured JSON configuration as follows:

```json
{
  "id": "id of the provider",
  "name": "provider name",
  "description": "description for provider",
  "icon_url": "icon url",
  "apiKey": "if needed",
  "base_url": "base url",
  "methods": [
    {
      "method_name": "method 1",
      "endpoint": "",
      "description": "description for method 1",
      "input_schema": {Json Object},
      "input_type": "QueryParams|BodyParams",
      "output_schema": {Json Object},
      "type": "GET|POST",
      "playground": "playground url"
    },
    {
      "method_name": "method 2",
      "endpoint": "",
      "description": "description for method 2",
      "input_schema": {Json Object},
      "input_type": "QueryParams|BodyParams",
      "output_schema": {Json Object},
      "type": "GET|POST",
      "playground": "playground url"
    }
  ]
}
```
- `id`: id of the provider
- `name`: The provider name to show on web application
- `description`: Describe about your provider like what is purpose of this provider... or how to run your provider
- `icon_url`: The icon of the provider to show on our web application
- `base_url`: The base url of the provider
- `apiKey`: If your provider need an api key for requesting, please give us the demo api key or free api key
- `endpoints`: List of endpoints
  - `method_name`: The method name of the provider
  - `description`: Describe about the endpoint, what data is provided by this endpoint
  - `type`: GET/POST method for api endpoint
  - `endpoint`: The endpoint of the provider
  - `input_schema`: Json object. For example: `{"coinName":"Btc"}`
  - `input_type`: using query params or body data
  - `output_schema`: Json object. For example: `{"price":"1000"}`
  - `playground`: curl to playground

# II. Initial Request.

The initial request starts the data processing pipeline by providing the source data that flows through the various providers and adaptors. In ADCS onchain usescase, it is the event that emitted after the user call a `Request()` function on their consumer contract.
An initial should have these properties:

```json
{
  "adapterID": "id of the adapter",
  "params": {"json object"},
}
```
`adapterID` is the id of the adapter that will be used to process the initial request.
`params` is the parameters that will be used as inputs for every entity in the adapter.

# III. Node
A Node is a processing unit within the ADCS system that represents either a provider, an adapter. A node should have these properties:

- `id`: id of the node
- `node_type`: nodeType of the node
- `input`: input of the node
- `input_method`: input method of the node
- `output`: output of the node

# Example:

```json
{
  "id": "P1",
  "node_type": "provider",
  "input": ["IR.coinSymbol"],
  "input_method": "getPrice",
  "output": "priceData"
}
```

# IV.  Adapter

Adaptors serve as the intermediary processing layers that allow complex data transformations from multiple input sources and return an executable output format. Adaptor should have these properties:

- `id`: id of the adapter
- `name`: name to display on our web application
- `description`: describe about your adapter
- `icon`: icon to display on our web application
- `input_schema`: json object
- `output_schema`: json object
- `nodes`: 1 or a set of nodes

# 1. Single input adapter: is an adapter that take 1 adaptor OR provider as input.

```json
{
  "id": "id of the adapter",
  "name": "",
  "description": "",
  "icon": "",
  "input_schema": {"json object"},
  "output_schema": {"json object"},
  "nodes": [nodeID, node_type, input, input_method, output]
}
```

- `id`: id of the adapter 
- `name`: name to display on our web application
- `description`: describe about your adapter
- `icon`: icon to display on our web application
- `input_schema`: json object
- `output_schema`: json object
- `nodes`: Adapter|provider ID, input, input_method, output

# Example of a single input adapter

```json
{
  "id": "A1",
  "name": "Sentiment Analysis Adaptor",
  "description": "Analyzes sentiment from crypto-related news data and determines if it's positive or negative",
  "icon": "https://icon.ai/sentiment.png",
  "input_schema": {"newsText": "string"},
  "output_schema": {"score": "number"},
    "nodes": [
     {
        "id": "P1",
        "node_type": "provider",
        "input": ["IR.coinSymbol"],
        "input_method": "getPrice",
        "output": "OP1"
      }
      ]
}
```
The Output OP1 is the final output of the adapter A1.

# 2. Graph flow

A graphFlow is an adapter that contains multiple nodes. It defines the execution pathway of data through the ADCS system, represented as an array of nodes

```json
{
  "id": "",
  "name": "",
  "description": "",
  "icon": "",
  "input_schema": {"json object"},
  "output_schema": {"json object"},
  "nodes": [
    {P1},
    {P2},
    {A1},
    ...
    ]
}
```
# Example

```json
{
  "id": "GF1",
  "name": "Simple Price Analyzer",
  "description": "Gets price data and determines if it's a good time to buy",
  "icon": "https://icon.ai/simple-analyzer.png",
  "input_schema": {"coinSymbol": "string"},
  "output_schema": {"shouldBuy": "boolean", "reason": "string"},
  "nodes": [
    { 
      "id": "P1", 
      "input": ["IR.coinSymbol"], 
      "input_method": "getPrice",
      "output": "priceData" 
    },
    { 
      "id": "A1", 
      "input": ["priceData"], 
      "input_method": "AnalyzePriceData",
      "output": "output_schema" 
    }
  ]
}
```
This Graph Flow:
- Takes a coin symbol as input (like "BTC")
- Uses provider P1 to fetch price data
- Passes that price data to adapter A1
- A1 uses AI to analyze the data and determine if it's a good time to buy
- Outputs a boolean decision and reasoning
Just two nodes connected in sequence for a basic price analysis workflow.

## Disclaimer

The ADCS framework represents experimental technology. Implementation details, specifications, and best practices may change as the system evolves.