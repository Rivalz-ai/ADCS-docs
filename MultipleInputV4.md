# I. Provider

- Provider must has an input
- Provider must has an output

```json
{
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
      "input": {
        "coin_name": "string",
        "currency": "string"
      },
      "input_type": "QueryParams|BodyParams",
      "output": {
        "result": "string",
        "message": "string"
      },
      "type": "GET|POST",
      "playground": "playground url"
    },
    {
      "method_name": "method 2",
      "endpoint": "",
      "description": "description for method 2",
      "input": {
        "coin_name": "string",
        "currency": "string"
      },
      "input_type": "QueryParams|BodyParams",
      "output": {
        "result": "string",
        "message": "string"
      },
      "type": "GET|POST",
      "playground": "playground url"
    }
  ]
}
```

- `name`: The provider name to show on web appllication
- `description`: Describe about your provider like what is purpose of this provider... or how to run your provider
- `icon_url`: The icon of the provider to show on our web application
- `base_url`: The base url of the provider
- `apiKey`: If your provider need an api key for requesting, please give us the demo api key or free api key
- `endpoints`: List of endpoints
  - `method_name`: The method name of the provider
  - `description`: Describe about the endpoint, what data is provided by this endpoint
  - `type`: GET/POST method for api endpoint
  - `endpoint`: The endpoint of the provider
  - `input`: Json object. For example: `{"coinName":"Btc"}`
  - `input_type`: using query params or body data
  - `output`: Json object. For example: `{"price":"1000"}`
  - `playground`: curl to playground

# Adapter

- Adapter can be created by one or more adapters/providers
- Adapter must has input
- Adapter must has output

```json
{
  "name": "",
  "description": "",
  "icon": "",
  "input": {"json object"},
  "output": {"json object"},
  "nodes": ["nodes"]
}
```

- `name`: name to display on our web application
- `description`: describe about your adapter
- `input`: json object
- `output`: json object
- `nodes`: set of node process for adapter

# Node

```json
{
  "id": "P1",
  "type": "provider | adapter | LLM",
  "llm_id": "id of llm",
  "input": [{"IRValue1"},{"IRValue2"}],
  "output": "json object"
}
```

- `id`: id of provider or adapter that need to use
- `type`: provider, adapter or LLM
- `llm_id`: Optional. Id of llm that need to use
- `input`: array json objects with the item is the same object type. For example: ["getPriceValue1","getPriceValue2"] with getPriceValue is json object type
- `output`: json object

# Graph flow

```json
[
  { "id": "P1", "type": "provider", "input": ["IR"], "input_method": "method name", "output": "OP1" },
  { "id": "P2", "type": "provider", "input": ["IR"], "input_method": "method name", "output": "OP2" }
];

```

# Example

**1. Providers**

- P1: Get price
- `getPriceInput`: `{"coinName":"string"}`
- `getPriceOutput`: `{"price":"string"}`

```json
{
  "name": "Get price",
  "description": "get price from coingecko. the input is pair of coin name(coinName-Currency) and using as api params. example: https://getprice.ai/bicoin-usdc",
  "endpoint": "https://getprice",
  "type": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/price.png",
  "input": "getPriceInput",
  "input_type": "QueryParams",
  "output": "getPriceOutput",
  "playground": "https://getprice.ai/bitcoin-usdc"
}
```

- P2: Get market cap

- `getMarketCapInput`: `{"coinName":"string"}`
- `getMarketCapOutput`: `{"usd_market_cap": "number", "usd_24h_vol": "number", "usd_24h_change": "number", "last_updated_at": "timestamp"}`

```json
{
  "name": "Get market cap by given coin name",
  "description": "get market cap by given coin name. the input is coin name(coinName) and using as api params. example: https://market.ai/bicoin",
  "type": "https://market.ai",
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/market.png",
  "input": "getMarketCapInput",
  "input_type": "QueryParams",
  "output": "getMarketCapOutput",
  "playground": "https://market.ai/bicoin"
}
```

**2. Adapters**

**2.1 Adapter using 1 provider**

- A1: analyst data from getPrice

- `A1Input`: `{
  "getPrice":["getPriceInput1","getPriceInput1"]
}`
- `A1Output`: `["getPriceOutput1","getPriceOutput2"]`
- `llm-1`: chatgpt 4o

```json
{
  "name": "analyst data from getPrice",
  "description": "analyst data from getPrice",
  "icon": "https://icon.ai/a1.png",
  "input": "A1Input",
  "output": "A1Output",
  "nodes": [
    {
      "id": "P1",
      "type": "provider",
      "llm_id": "",
      "input": "A1Input.getPrice",
      "output": "getPriceOutput"
    }
  ]
}
```

**2.2 Adapter using multiple providers**

- A1: analyst data from getPrice and getMarketCap

- `A1Input`: `{
  "getPrice":["getPriceInput1","getPriceInput1"],
  "getMarketCap":["getMarketCapInput1","getMarketCapInput2"]
}`
- `A1Ouput`: `{
  "coinName":"string",
  "price":"string",
  "decision":"string"
}`
- `llm-1`: chatgpt 4o

```json
{
  "name": "make decision for meme coin trading",
  "description": "Search trending meme coins and analyst data then make decision buy or sell. Input contain number of coin, market cap",
  "icon": "https://icon.ai/a1.png",
  "input": "A1Input",
  "output": ["A1Output"],
  "nodes": [
    {
        "id": "P1",
        "type": "provider",
        "llm_id": "",
        "input": "A1Input.getPrice",
        "output": "getPriceOutput"
    },
    {
        "id": "P2",
        "type": "provider",
        "llm_id": "",
        "input": "A1Input.getMarketCap",
        "output": "getMarketCapOutput"
    },
    {
        "id": "A2",
        "type": "LLM",
        "llm_id": "llm-1",
        "input": {
            "prompt": "Base on this data please give me a decision should buy or sell coin. The output is json format like this {A1Output}
            Data: {getPriceOutput,getMarketCapOutput}"
        },
        "output": "A1Output"
    }
  ]
}
```

- When process A1
  - process all nodes use IR as input first -> process P1,P2 first
  - if input of node have more than 1 item, process node with loop statment
  - process A2 using `llm-1` (chatgtp4o). Output of p1,p2 as part of input
  - return A1output
