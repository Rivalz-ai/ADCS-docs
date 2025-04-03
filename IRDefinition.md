# IR Definition

- Initial request is a dynamic object that combine all inputs in the node definition.
- Adapter/Provider: have a static definition of the input and output schema.

```json
{
  "IR1": {
    "fields": {
      "value1": "string",
      "value2": "number"
    }
  },
  "IR2": {
    "fields": {
      "value1": "string",
      "value2": "number"
    }
  }
}
```

# Nodes definition

**1. single node**

```json
{
  "id": "P1",
  "type": "provider",
  "input": ["IR"],
  "output": "OP1"
}
```

- id: unique identifier for the provider/adapter
- type: provider/adapter
- input: list of input from IR or other nodes
- output: json schema of the output

**2. set of nodes**

![Diagram](./images/MultipleInputRefined.png)

**Inital request:**

```json
{
  "IR1": {
    "fields": {
      "value1": "string",
      "value2": "number"
    }
  },
  "IR2": {
    "fields": {
      "value1": "string",
      "value2": "boolean"
    }
  },
  "IR3": {
    "fields": {
      "value1": "string"
    }
  },
  "IR4": {
    "fields": {
      "value1": "string"
    }
  }
}
```

**Nodes definition:**

```json
[
  { "id": "P1", "type": "provider", "input": ["IR1"], "output": "OP1" },
  { "id": "P2", "type": "provider", "input": ["IR2"], "output": "OP2" },
  { "id": "P3", "type": "provider", "input": ["IR3"], "output": "OP3" },
  { "id": "A4", "type": "adapter", "input": ["IR4"], "output": "OA4" },
  { "id": "A2", "type": "adapter", "input": ["OP1"], "output": "OA2" },
  { "id": "A3", "type": "adapter", "input": ["OA2", "OP2"], "output": "OA3" },
  { "id": "P5", "type": "provider", "input": ["OP2", "OP3", "OA3", "OA4"], "output": "OA5" }
];

```

- IR: Initial Request
- OP: Output of the provider
- OA: Output of the adapter

**Process order:**

- P1 with input value from IR1 -> OP1
- P2 with input value from IR2 -> OP2
- P3 with input value from IR3 -> OP3
- A4 with input value from IR4 -> OA4
- A2 with input value from OP1 -> OA2
- A3 with input value from OA2 and OP2 -> OA3
- P5 with input value from OP2, OP3, OA3, OA4 -> OP5
