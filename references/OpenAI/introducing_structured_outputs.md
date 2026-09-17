https://openai.com/index/introducing-structured-outputs-in-the-api/

# Introducing Structured Outputs in the API

Structured Outputs in the API ensures model-generated outputs will exactly match JSON Schemas provided by developers. This solves the long-standing problem of LLMs not reliably matching developer-supplied schemas.

## Performance
On evaluations of complex JSON schema following, gpt-4o-2024-08-06 with Structured Outputs scores 100%. In comparison, gpt-4-0613 scores less than 40%.

## Two Forms of Structured Outputs

### 1. Function Calling
Enable by setting `strict: true` within your function definition. Works with all models that support tools, including `gpt-4-0613` and `gpt-3.5-turbo-0613` and later.

```python
{
  "model": "gpt-4o-2024-08-06",
  "messages": [...],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "query",
        "description": "Execute a query.",
        "strict": true,
        "parameters": {
          "type": "object",
          "properties": {
            "table_name": {"type": "string", "enum": ["orders"]},
            "columns": {
              "type": "array",
              "items": {
                "type": "string",
                "enum": ["id", "status", "expected_delivery_date", "delivered_at", "shipped_at", "ordered_at", "canceled_at"]
              }
            },
            "conditions": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "column": {"type": "string"},
                  "operator": {"type": "string", "enum": ["=", ">", "<", ">=", "<=", "!="]},
                  "value": {
                    "anyOf": [
                      {"type": "string"},
                      {"type": "number"},
                      {
                        "type": "object",
                        "properties": {"column_name": {"type": "string"}},
                        "required": ["column_name"],
                        "additionalProperties": false
                      }
                    ]
                  }
                },
                "required": ["column", "operator", "value"],
                "additionalProperties": false
              }
            },
            "order_by": {"type": "string", "enum": ["asc", "desc"]}
          },
          "required": ["table_name", "columns", "conditions", "order_by"],
          "additionalProperties": false
        }
      }
    }
  ]
}
```

### 2. Response Format
Supply a JSON Schema via `json_schema` in the `response_format` parameter. Works with `gpt-4o-2024-08-06` and `gpt-4o-mini-2024-07-18`.

```python
{
  "model": "gpt-4o-2024-08-06",
  "messages": [
    {"role": "system", "content": "You are a helpful math tutor."},
    {"role": "user", "content": "solve 8x + 31 = 2"}
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "math_response",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "steps": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "explanation": {"type": "string"},
                "output": {"type": "string"}
              },
              "required": ["explanation", "output"],
              "additionalProperties": false
            }
          },
          "final_answer": {"type": "string"}
        },
        "required": ["steps", "final_answer"],
        "additionalProperties": false
      }
    }
  }
}
```

## Safe Structured Outputs
The model can still refuse unsafe requests. A new `refusal` string value on API responses allows developers to programmatically detect if the model has generated a refusal instead of output matching the schema.

```python
{
  "id": "chatcmpl-9nYAG9LPNonX8DAyrkwYfemr3C8HC",
  "object": "chat.completion",
  "created": 1721596428,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "refusal": "I'm sorry, I cannot assist with that request."
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 81,
    "completion_tokens": 11,
    "total_tokens": 92
  },
  "system_fingerprint": "fp_3407719c7f"
}
```

## Native SDK Support
Python and Node SDKs have native support for Structured Outputs using Pydantic and Zod objects.

```python
from enum import Enum
from typing import Union
from pydantic import BaseModel
import openai
from openai import OpenAI

class Table(str, Enum):
    orders = "orders"
    customers = "customers"
    products = "products"

class Column(str, Enum):
    id = "id"
    status = "status"
    expected_delivery_date = "expected_delivery_date"
    delivered_at = "delivered_at"
    shipped_at = "shipped_at"
    ordered_at = "ordered_at"
    canceled_at = "canceled_at"

class Operator(str, Enum):
    eq = "="
    gt = ">"
    lt = "<"
    le = "<="
    ge = ">="
    ne = "!="

class OrderBy(str, Enum):
    asc = "asc"
    desc = "desc"

class DynamicValue(BaseModel):
    column_name: str

class Condition(BaseModel):
    column: str
    operator: Operator
    value: Union[str, int, DynamicValue]

class Query(BaseModel):
    table_name: Table
    columns: list[Column]
    conditions: list[Condition]
    order_by: OrderBy

client = OpenAI()

completion = client.beta.chat.completions.parse(
    model="gpt-4o-2024-08-06",
    messages=[
        {"role": "system", "content": "You are a helpful assistant. The current date is August 6, 2024. You help users query for the data they are looking for by calling the query function."},
        {"role": "user", "content": "look up all my orders in may of last year that were fulfilled but not delivered on time"},
    ],
    tools=[openai.pydantic_function_tool(Query)],
)

print(completion.choices[0].message.tool_calls[0].function.parsed_arguments)
```

### Response Format Example

```python
from pydantic import BaseModel
from openai import OpenAI

class Step(BaseModel):
    explanation: str
    output: str

class MathResponse(BaseModel):
    steps: list[Step]
    final_answer: str

client = OpenAI()

completion = client.beta.chat.completions.parse(
    model="gpt-4o-2024-08-06",
    messages=[
        {"role": "system", "content": "You are a helpful math tutor."},
        {"role": "user", "content": "solve 8x + 31 = 2"},
    ],
    response_format=MathResponse,
)

message = completion.choices[0].message
if message.parsed:
    print(message.parsed.steps)
    print(message.parsed.final_answer)
else:
    print(message.refusal)
```

## Additional Use Cases

### Dynamically Generating User Interfaces
Use Structured Outputs to create code- or UI-generating applications based on user input.

### Separating Final Answer from Reasoning
Give the model a separate field for chain of thought to improve final quality.

### Extracting Structured Data from Unstructured Data
Extract to-dos, due dates, and assignments from meeting notes.

## Under the Hood

### Constrained Decoding
We use constrained sampling to ensure models only select tokens that would be valid according to the supplied schema. This is done by converting JSON Schema into a context-free grammar (CFG), then dynamically determining which tokens are valid after each token is generated.

### Why Context-Free Grammars
CFGs can express a broader class of languages than finite state machines (FSMs). This matters for complex schemas involving nested or recursive data structures. FSMs cannot generally express recursive types, which means FSM-based approaches may struggle to match parentheses in deeply nested JSON.

## Limitations

- Structured Outputs allows only a subset of JSON Schema
- First API response with a new schema incurs additional latency (typically <10s, up to 1 minute for complex schemas)
- Model can fail to follow schema if it refuses an unsafe request or reaches max_tokens before finishing
- Structured Outputs doesn't prevent all mistakes (e.g., mathematical errors in values)
- Not compatible with parallel function calls
- JSON Schemas supplied with Structured Outputs aren't Zero Data Retention (ZDR) eligible

## Availability
Structured Outputs is generally available today in the API.

**Function calling:** Available on all models that support function calling, including `gpt-4o`, `gpt-4o-mini`, all models after `gpt-4-0613` and `gpt-3.5-turbo-0613`, and fine-tuned models. Available on Chat Completions API, Assistants API, and Batch API. Compatible with vision inputs.

**Response formats:** Available on `gpt-4o-mini` and `gpt-4o-2024-08-06` and fine tunes based on these models. Available on Chat Completions API, Assistants API, and Batch API. Compatible with vision inputs.

**Pricing:** gpt-4o-2024-08-06 saves 50% on inputs ($2.50/1M input tokens) and 33% on outputs ($10.00/1M output tokens) compared to gpt-4o-2024-05-13.

## Acknowledgements
Structured Outputs takes inspiration from open source libraries: outlines, jsonformer, instructor, guidance, and lark.
