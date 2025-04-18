# GraphFlow

A graphFlow is an attribute of an adaptor. It defines the execution pathway of data through the ADCS system, represented as an array of nodes
You need to initialize the Nodes first before you can use them in the graphFlow.

# Example

```json
{
  "nodes": {"P1": "P2843923827", "A1": "A74523698", "A2": "A74523698"},
  "graph_flow": [
    {"id": "P1", "input": "IR.key1", "input_method": "methodName", "output": "OP1"},
    {"id": "A1", "input": "OP1", "input_method": "", "output": "OA1"},
    {"id": "A2", "input": "OA1", "input_method": "", "output": "OA2"},
    ...
    ]
}
```

In this example, the graph flow executes as follows:

- `P1` is a provider node that fetches data from the provider using the `methodName` method with the parameter `IR.key1`
- `A1` is an adapter node that uses the data from P1 as input, generate a new output `OA1`
- `A2` is an adapter node that uses the data from A1 as input, generate a new output `OA2`
- `OA2` is the final output of the GraphFlow
