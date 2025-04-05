# I. Provider

- Provider can has an input
- Provider must have an output

```json
{
  "name": "",
  "description": "",
  "endpoint": "",
  "type": "GET|POST",
  "apiKey": "",
  "icon": "",
  "input": {"json object"},
  "inputType": "QueryParams|Body json",
  "output": {"json object"},
  "playground": ""
}
```

- `name`: The provider name to show on web appllication
- `description`: Describe about your provider like what is purpose of this provider... or how to run your provider
- `endpoint`: The provider REST api endpoint
- `type`: GET/POST method for api endpoint
- `apiKey`: if your provider need an api key for requesting, please give us the demo api key or free api key
- `icon`: The icon of the provider to show on our web application
- `input`: Json object. For example: `{"coinName":"Btc"}`
- `inputType`: using query params or body data
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

- name: name to display on our web application
- description: describe about your adapter
- input: json object
- output: json object
- nodes: set of node process for adapter

**node**

```json
{
  "id": "P1",
  "type": "provider | adapter",
  "input": [{"IRValue1"},{"IRValue2"}],
  "output": "json object"
}
```

- `id`: id of provider or adapter that need to use
- `type`: provider or adapter
- `input`: array json objects with the item is the same object type. For example: ["getPriceValue1","getPriceValue2"] with getPriceValue is json object type
- `output`: json object

**Set of nodes**

```json
[
  { "id": "P1", "type": "provider", "input": ["IR"], "output": "OP1" },
  { "id": "P2", "type": "provider", "input": ["IR"], "output": "OP2" }
];

```

# Example

1. Providers

- P1: Get price

```json
{
  "name": "Get price",
  "description": "get price from coingecko. the input is pair of coin name(coinName-Currency) and using as api params. example: https://getprice.ai/bicoin-usdc",
  "endpoint": "https://getprice",
  "type": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/price.png",
  "input": {
    "coinName": "string"
  },
  "inputType": "query params",
  "output": { "price": "" },
  "playground": "https://getprice.ai/bitcoin-usdc"
}
```

==> So, we have </br>

- `getPriceInput`

```json
{
  "coinName": "string"
}
```

- `getPriceOutput`

```json
{
  "price": "number"
}
```

- P2: Get market cap

```json
{
  "name": "Get market cap by given coin name",
  "description": "get market cap by given coin name. the input is coin name(coinName) and using as api params. example: https://market.ai/bicoin",
  "type": "https://market.ai",
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/market.png",
  "input": { "coinName": "string" },
  "output": {
    "usd_market_cap": "number",
    "usd_24h_vol": "number",
    "usd_24h_change": "number",
    "last_updated_at": "timestamp"
  },
  "playground": "https://market.ai/bicoin"
}
```

==> So we have

- `getMarketCapInput`

```json
{ "coinName": "string" }
```

- `getMarketCapOutput`

```json
{
  "usd_market_cap": "number",
  "usd_24h_vol": "number",
  "usd_24h_change": "number",
  "last_updated_at": "timestamp"
}
```

- P3: using chatgp-4o AI to make decision by given data

```json
{
  "name": "Make desicion by given data",
  "description": "This provider will analyst data from input and make decision depend on the requirement. The input contains data and AI context. example: https://aiInference.ai , body: {'input': [make decision should by or sell bitcoin]}",
  "endpoint": "https://aiInference.ai",
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/inference.png",
  "input": { "prompt": "string" },
  "output": { "decision": "json object" },
  "playground": "https://aiInference.ai"
}
```

2. Adapters

- A1: get token price and market cap

- We have
  - `A1Input`: `{
  "getPrice":["getPriceInput1","getPriceInput1"],
  "getMarketCap":["getMarketCapInput1","getMarketCapInput2"]
}`
  - `A1Ouput`: `{
  "coinName":"string",
  "price":"string",
  "decision":"string"
}`

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
        "input": "A1Input.getPrice",
        "output": "getPriceOutput"
    },
    {
        "id": "P2",
        "type": "provider",
        "input": "A1Input.getCoinMarketCap",
        "output": "getCoinMarketCapOutput"
    },
    {
        "id": "P3",
        "type": "provider",
        "input": {"prompt":"Base on this data please give me a decision should buy or sell coin. The output is json format like this {A1Output}
        Data: {getPriceOutput,getCoinMarketCapOutput}".},
        "output": "A1Output"
    }
  ]
}
```

- When process A1
  - process all nodes use IR as input first - process P1,P2 first
  - if input of node have more than 1 item, process node with loop statment
  - process P3 using output of p1,p2 as part of input
  - return A1output
