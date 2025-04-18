## Adapter/Provider ID creation

**We will generate a 32 bytes hash, this is unique hash. And using prefix for adapter/provider**

- Provider: id of provider like this `P0x000123...`
- Adapter: id of adapter like this `A0x123000...`

**The url of adapter/provider**

Base url: `https://api.adcs.rivalz/v1`

- Provider: `[base_url]/provider/[providerId]`.

```cUrl
curl --location 'https://api.adcs.rivalz/v1/provider/P0x000123' \
        --header 'Content-Type: application/json' \
        --data '{ "content": "decision should buy or sell BTC at this time", "dataTypeId": 2 }'
```

- Adapter: `[base_url]/dapter/[adapterId]`.

```cUrl
curl --location 'https://api.adcs.rivalz/v1/adapter/A0x123000' \
        --header 'Content-Type: application/json' \
        --data '{ "content": "decision should buy or sell BTC at this time", "dataTypeId": 2 }'
```

## Api key management

- We will store api key of provider when they provide it from provider page. see [this](./ProviderV2.md/#how-to-become-a-provider) for more detail.
- After user submit api key, we will encrypt this key and store it to our database.
- Using: when running a provider, we will get it's api key(encrypted) and decrypt it to run provider.
