# Provider

- Provider can has an input or doen't has an input(empty array)
- Provider must have an output

```json
{
  "name": "",
  "description": "",
  "endpoint": "",
  "method": "",
  "apiKey": "",
  "icon": "",
  "input": [],
  "output": [],
  "playground": ""
}
```

- name: The provider name to show on web appllication
- description: Describe about your provider like what is purpose of this provider... or how to run your provider
- endpoint: The provider REST api endpoint
- method: GET/POST method for api endpoint
- apiKey: if your provider need an api key for requesting, please give us the demo api key or free api key
- icon: The icon of the provider to show on our web application
- input: An array of parameters. for example: [coinName-currency]

  - If your endpoint use this input as api parameters, it will like this "https://getPrice/[coinName-currency]"
  - If your endpoint use input in body of the api request, the body data will be like this `{"input":[coinName-currency]}`

- output: An array of parameters

# Adapter

- Adapter can be created by one or more adapters/providers
- Adapter can has input as an array or doesn't has input(empty array)
- Adapter must has output data as an array

```json
{
  "name": "",
  "description": "",
  "icon": "",
  "input": [],
  "output": [],
  "nodes": []
}
```

- name: name to display on our web application
- description: describe about your adapter
- input: An array data use for adapter (IR)
- output: An array data output
- nodes: set of node process for adapter

**node**

```json
{
  "id": "P1",
  "type": "provider",
  "input": ["IR"],
  "output": "OP1"
}
```

**set of nodes**

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
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/price.png",
  "input": [CoinName-Currency],
  "output": [price],
  "playground":"https://getprice.ai/bicoin-usdc"
}
```

- P2: Get market cap

```json
{
  "name": "Get market cap by given coin name",
  "description": "get market cap by given coin name. the input is coin name(coinName) and using as api params. example: https://market.ai/bicoin",
  "endpoint": "https://market.ai",
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/market.png",
  "input": [CoinName],
  "output": [marketCap],
  "playground": "https://market.ai/bicoin"
}
```

- P3: Get trending meme coins

```json
{
  "name": "Get trending meme coins",
  "description": "Get trending meme coins with given number of coins and market cap. The input contains 2 paramters: number of coin and market cap. Example: https://search.ai/5/1000000 . The output is the array with 5 coin names that have market cap equal or more than 1,000,000. it look like [doge, trump, elon, loki, xToken]",
  "endpoint": "https://search.ai",
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/search.png",
  "input": [CoinNumber,MarketCap],
  "output": [[coinName1,marketCap1],[coinName2,marketCap2]],
  "playground": "https://search.ai/5/1000000"
}
```

- P4: using chatgp-4o AI to make decision by given data

```json
{
  "name": "Make desicion by given data",
  "description": "This provider will analyst data from input and make decision depend on the requirement. The input contains data and AI context. example: https://aiInference.ai , body: {'input': [make decision should by or sell bitcoin]}",
  "endpoint": "https://aiInference.ai",
  "method": "GET",
  "apiKey": "",
  "icon": "https://icon.ai/inference.png",
  "input": [aiPrompt],
  "output": [decision],
  "playground": "https://aiInference.ai"
}
```

2. Adapters

- A1: analyst data and make trade decision

```json
{
  "name": "make decision for meme coin trading",
  "description": "Search trending meme coins and analyst data then make decision buy or sell. Input contain number of coin, market cap",
  "icon": "https://icon.ai/a1.png",
  "input": [numberOfCoin,marketCap],
  "output": [[coinName1,decision1],[coinName2,decision2]],
  "nodes": [
    {
        "id": "P3",
        "type": "provider",
        "input": ["IR[0]","IR[1]"],
        "output": [["coinName1","maketCap1"],["coinName2","maketCap2"]]
    },
    {
        "id": "P4",
        "type": "provider",
        "input": ["Base on this data please give me a decision should buy or sell coin. The output is array data like this [[coinName1,decision1],[coinName2,decision2]]
        Data: [coinName1,maketCap1],[coinName2,maketCap2]"],
        "output": "[[coinName1,decision1],[coinName2,decision2]]"
    }
  ]
}
```

**Process A1**

- Initial request: [3,1000000]
- nodes run
  - P3 with input [3,1000000]</br>
    => output: [["doge","50000000"],["loki","2000000"],["trump","1000000"]]
  - P4 with input [["doge","50000000"],["loki","2000000"],["trump","1000000"]]</br>
    => output is [["doge","sell"],["loki","sell"],["trump","buy"]]
