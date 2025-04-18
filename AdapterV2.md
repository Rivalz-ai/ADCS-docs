# Adapter

Adaptors serve as the intermediary processing layers that allow complex data transformations from multiple input sources and return an executable output format.

Adaptor should have these attributes:

```json
{
  "id": "String",
  "name": "String",
  "description": "String",
  "icon": "String",
  "core_llm": "String",
  "static_context": "String",
  "input_schema": { "key": "value" },
  "output_schema": { "key": "value" },
  "nodes": { "key": "value" },
  "graph_flow": [
    {
      "id": "string",
      "input": ["IR"],
      "input_method": "string",
      "output": "string"
    }
  ]
}
```

- `id`: unique id of the adaptor
- `name`: name to display on our web application
- `description`: describe about your adaptor
- `icon`: icon to display on our web application
  - Recommend to use svg format for the icon
  - size: 100x100px
- `input_schema`: expected input of the adaptor
- `output_schema`: expected output format of the adaptor
- `core_llm`: LLM model to use for the adaptor
  - Optional. In case you want to use an inference provider, you can leave it empty.
- `static_context`: extra prompt to the LLM
  - Optional. In case you dont want to use any extra context, you can leave it empty.
- `nodes`: Initialize the nodes first before you can use them in the graphFlow. For more information about node, please see [this](/Node.md)
- `graph_flow`: 1 or a set of nodes. For more information about graphFlow, please see [this](/GraphFlow.md)

# Create new adapter

## 1. Install adcs cli.

- Install adcs cli lastest version from npm package

```Shell
npm i -g rivalz-adcs-cli
```

- Verify cli is working

```Shell
rivalz-adcs --version
```

It should return the version number like this `1.0.0`.

Congratuation! Adcs cli is ready to use.

## 2. Register a key to use CLI

To use adcs cli, you need to register a key [here](https://adcs.rivalz.ai/dashboard) (This page not have yet, need to design the page)

- Login with your wallet
- Go to dashboard page
- At 'api key' section, click to 'generate new key' button to create new key

## 3. Using cli to create new adapter

Before using cli you need to provide cli's api key.
To convinent, you can export enviroment variable for your machine

```shell
export ADCS_KEY='0x123'
```

Change `0x123` with your real key(get from dashboard page)

### 3.1 Init a adapter

Using `init [adapter name]` subcommand to init new adapter

```Shell
rivalz-adcs init --name hello_adapter --path ./adapters
```

- `--name`: name of adapter
  - The name of the adapter you want to create
  - Must be unique and use snake_case format
  - Example: hello_adapter, sentiment_analysis
- `--path`: directory to store adapter json file

  - The directory path where you want to store the adapter JSON file
  - Can be relative or absolute path
  - Example: ./adapters, /home/user/adapters

After init new adapter, you can see new json file in your `path`.</br>
Configue adapter with structure [above](./AdapterV2.md/#adapter)

### 3.2 Verify adapter

Using `verify adapter` to verify the structure of adapter

```Shell
rivalz-adcs verify adapter  --path [path_to_file]
```

- `--path`: path to json adapter configuration file. Example: `./adapters/hello_adapter.json`

If you can see message 'successfully', congratuaration! your adapter is valid. If not, please review your json file or follow error message to fix it and then verify it again.

### 3.3 Run a test

Using `test adapter` to run a test for adapter

```Shell
rivalz-adcs test adapter --path --key
```

- `--path`: path to adapter json file
- `--key`: api key get from dashboard page. The default value will get from environment variable `ADCS_KEY`.

The result will be like this

```shell
{
    "message":"success",
    "data":"out put of adapter here"
}
```

### 3.4 Deploy a dapter

Using `deploy adapter` to deploy your adapter. After deploying successfull, you can go to our website [here](https://testnet-adcs.rivalz.ai) to see your adapter.

```shell
rivalz-adcs deploy adapter --path --key
```

- `--path`: path to adapter json file. Example: ./adapters/hello_adapter.json
- `--key`: api key get from dashboard page. The default value will get from environment variable `ADCS_KEY`.
- If you can see message 'successfully', it mean your adapter deploy success. If not please follow the error message to fix it and deploy again.

### 3.5 Other command

- You can use `--help` command to see all commands
- Some common commands
  - `--version`: show version of cli
  - `list adapter`: get list of your adapter
  - `update adapter --id [adapter_id]`: update existing adapter
  - `delete adapter --id [adapter_id]`: delete existing adapter
