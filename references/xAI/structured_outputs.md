https://docs.x.ai/developers/model-capabilities/text/structured-outputs

# Structured Outputs

Returns API responses in a specific JSON schema format with guaranteed schema compliance. Define schemas using Pydantic or Zod.

## Supported Models
All xAI language models.

## Supported Schema Types
- `string` (minLength/maxLength not supported)
- `number`, `integer`, `float`
- `object`
- `array` (minItems/maxItems/maxContains/minContains not supported)
- `boolean`
- `enum`
- `anyOf`
- `allOf` (not supported)

## Invoice Parsing Example

Extract structured data from raw invoice text using Pydantic schema:

```python
from datetime import date
from enum import Enum
from pydantic import BaseModel, Field

class Currency(str, Enum):
    USD = "USD"
    EUR = "EUR"
    GBP = "GBP"

class LineItem(BaseModel):
    description: str = Field(description="Description of the item or service")
    quantity: int = Field(description="Number of units", ge=1)
    unit_price: float = Field(description="Price per unit", ge=0)

class Address(BaseModel):
    street: str = Field(description="Street address")
    city: str = Field(description="City")
    postal_code: str = Field(description="Postal/ZIP code")
    country: str = Field(description="Country")

class Invoice(BaseModel):
    vendor_name: str = Field(description="Name of the vendor")
    vendor_address: Address = Field(description="Vendor's address")
    invoice_number: str = Field(description="Unique invoice identifier")
    invoice_date: date = Field(description="Date the invoice was issued")
    line_items: list[LineItem] = Field(description="List of purchased items/services")
    total_amount: float = Field(description="Total amount due", ge=0)
    currency: Currency = Field(description="Currency of the invoice")
```

System prompt: "Given a raw invoice, carefully analyze the text and extract the invoice data into JSON format."

Example invoice text:

```
Vendor: Acme Corp, 123 Main St, Springfield, IL 62704
Invoice Number: INV-2025-001
Date: 2025-02-10
Items:
- Widget A, 5 units, $10.00 each
- Widget B, 2 units, $15.00 each
Total: $80.00 USD
```

Implementation:

```python
import os
from xai_sdk import Client
from xai_sdk.chat import system, user

client = Client(api_key=os.getenv("XAI_API_KEY"))
chat = client.chat.create(model="grok-4-1-fast-reasoning")
chat.append(system("Given a raw invoice, carefully analyze the text and extract the invoice data into JSON format."))
chat.append(user("""
Vendor: Acme Corp, 123 Main St, Springfield, IL 62704
Invoice Number: INV-2025-001
Date: 2025-02-10
Items: - Widget A, 5 units, $10.00 each - Widget B, 2 units, $15.00 each
Total: $80.00 USD
"""))

# Returns tuple of (full response, parsed pydantic object)
response, invoice = chat.parse(Invoice)
assert isinstance(invoice, Invoice)

print(invoice.vendor_name)
print(invoice.invoice_number)
print(response.content)  # JSON schema representation
```

Output:

```json
{"vendor_name": "Acme Corp", "vendor_address": {"street": "123 Main St", "city": "Springfield", "postal_code": "62704", "country": "IL"}, "invoice_number": "INV-2025-001", "invoice_date": "2025-02-10", "line_items": [{"description": "Widget A", "quantity": 5, "unit_price": 10.0}, {"description": "Widget B", "quantity": 2, "unit_price": 15.0}], "total_amount": 80.0, "currency": "USD"}
```

## Structured Outputs with Tools

Available for Grok 4 models (grok-4-1-fast, grok-4-fast, grok-4-1-fast-non-reasoning, grok-4-fast-non-reasoning).

Combine structured outputs with:
- **Agentic tool calling**: Server-side tools (web search, X search, code execution)
- **Function calling**: User-supplied custom functions

### Agentic Tools Example

```python
from pydantic import BaseModel, Field

class ProofInfo(BaseModel):
    name: str = Field(description="Name of the proof or paper")
    authors: str = Field(description="Authors of the proof")
    year: str = Field(description="Year published")
    summary: str = Field(description="Brief summary of the approach")

from xai_sdk import Client
from xai_sdk.chat import user
from xai_sdk.tools import web_search

client = Client(api_key=os.getenv("XAI_API_KEY"))
chat = client.chat.create(model="grok-4-1-fast", tools=[web_search()])
chat.append(user("Find the latest machine-checked proof of the four color theorem."))
response, proof = chat.parse(ProofInfo)
print(f"Name: {proof.name}\nAuthors: {proof.authors}")
```

### Client-side Function Tools Example

```python
from pydantic import BaseModel, Field

class CollatzResult(BaseModel):
    starting_number: int = Field(description="The input number")
    steps: int = Field(description="Number of steps to reach 1")

import json
from xai_sdk.chat import tool, tool_result, user

def collatz_steps(n: int) -> int:
    """Returns number of steps for n to reach 1 in Collatz sequence."""
    steps = 0
    while n != 1:
        n = n // 2 if n % 2 == 0 else 3 * n + 1
        steps += 1
    return steps

collatz_tool = tool(
    name="collatz_steps",
    description="Compute steps for a number to reach 1 in Collatz sequence",
    parameters={
        "type": "object",
        "properties": {"n": {"type": "integer", "description": "Starting number"}},
        "required": ["n"],
    },
)

chat = client.chat.create(model="grok-4-1-fast-non-reasoning", tools=[collatz_tool])
chat.append(user("Use collatz_steps tool to find steps for 20250709 to reach 1."))

# Handle tool calls until final response
while True:
    response = chat.sample()
    if not response.tool_calls:
        break
    chat.append(response)
    for tc in response.tool_calls:
        args = json.loads(tc.function.arguments)
        result = collatz_steps(args["n"])
        chat.append(tool_result(str(result)))

response, result = chat.parse(CollatzResult)
print(f"Starting: {result.starting_number}\nSteps: {result.steps}")
```

## Alternative: Using response_format with sample() or stream()

Instead of `parse()`, pass Pydantic model to `response_format` when creating chat:

| Approach | Method | Returns | Parsing |
|----------|--------|---------|---------|
| Using parse() | chat.parse(Model) | (Response, Model) | Automatic |
| Using response_format | chat.sample() or chat.stream() | Response with JSON string | Manual via response.content |

Use `parse()` for simplest experience. Use `response_format` + `sample()`/`stream()` for:
- More control over parsing
- Raw JSON string handling
- Streaming with structured outputs

Example:

```python
import os
from xai_sdk import Client
from xai_sdk.chat import system, user

client = Client(api_key=os.getenv("XAI_API_KEY"))
chat = client.chat.create(model="grok-4-1-fast-reasoning", response_format=Invoice)

chat.append(system("Given a raw invoice, carefully analyze the text and extract the invoice data into JSON format."))
chat.append(user("""
Vendor: Acme Corp, 123 Main St, Springfield, IL 62704
Invoice Number: INV-2025-001
Date: 2025-02-10
Items: - Widget A, 5 units, $10.00 each - Widget B, 2 units, $15.00 each
Total: $80.00 USD
"""))

response = chat.sample()  # Returns Response object
print(response.content)  # Valid JSON string

# Manually parse JSON into Pydantic model
invoice = Invoice.model_validate_json(response.content)
print(invoice.vendor_name)
```

### Streaming with response_format

```python
from pydantic import BaseModel, Field

class Summary(BaseModel):
    title: str = Field(description="A brief title")
    key_points: list[str] = Field(description="Main points from the text")
    sentiment: str = Field(description="Overall sentiment: positive, negative, or neutral")

chat = client.chat.create(model="grok-4-1-fast-reasoning", response_format=Summary)
chat.append(system("Analyze the following text and provide a structured summary."))
chat.append(user("The new product launch exceeded expectations with record sales..."))

# Stream response - chunks contain partial JSON
for response, chunk in chat.stream():
    print(chunk.content, end="", flush=True)

summary = Summary.model_validate_json(response.content)
print(f"Title: {summary.title}")
```