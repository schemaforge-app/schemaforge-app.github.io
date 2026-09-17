https://console.groq.com/docs/structured-outputs

# Structured Outputs

Guarantee model responses conform to JSON schema for type-safe data structures.

## Two Modes

### Strict Mode (strict: true)
- **Constrained decoding** guarantees 100% schema adherence
- Never produces invalid JSON
- Requirements: All fields `required`, `additionalProperties: false`
- Models: GPT-OSS 20B, GPT-OSS 120B
- Use for production apps requiring reliability

### Best-effort Mode (strict: false, default)
- Attempts schema compliance without hard constraints
- May produce schema-invalid JSON or 400 errors
- More flexible constraints (optional fields allowed)
- Broader model support: GPT-OSS models, Kimi K2, Llama 4 Maverick/Scout
- Use for development or unsupported models

**Benefits**: Type-safe responses, programmatic refusal detection, simplified prompting

**Note**: Streaming and tool use not currently supported.

## Product Review Extraction Example

```python
from groq import Groq
import json

groq = Groq()

response = groq.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {"role": "system", "content": "Extract product review information from the text."},
        {"role": "user", "content": "I bought the UltraSound Headphones last week and I'm really impressed! The noise cancellation is amazing and the battery lasts all day. Sound quality is crisp and clear. I'd give it 4.5 out of 5 stars."}
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "product_review",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "product_name": {"type": "string"},
                    "rating": {"type": "number"},
                    "sentiment": {"type": "string", "enum": ["positive", "negative", "neutral"]},
                    "key_features": {"type": "array", "items": {"type": "string"}}
                },
                "required": ["product_name", "rating", "sentiment", "key_features"],
                "additionalProperties": False
            }
        }
    }
)

result = json.loads(response.choices[0].message.content or "{}")
```

Output: `{"product_name": "UltraSound Headphones", "rating": 4.5, "sentiment": "positive", "key_features": ["amazing noise cancellation", "all-day battery life", "crisp and clear sound quality"]}`

## Schema Validation with Pydantic

```python
from groq import Groq
from pydantic import BaseModel
from typing import List, Optional, Literal
from enum import Enum

class SupportCategory(str, Enum):
    API = "api"
    BILLING = "billing"
    BUG = "bug"
    FEATURE_REQUEST = "feature_request"

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"

class CustomerInfo(BaseModel):
    name: str
    company: Optional[str] = None
    tier: Literal["free", "paid", "enterprise", "trial"]

class TechnicalDetail(BaseModel):
    component: str
    error_code: Optional[str] = None
    description: str

class SupportTicket(BaseModel):
    category: SupportCategory
    priority: Priority
    urgency_score: float
    customer_info: CustomerInfo
    technical_details: List[TechnicalDetail]
    keywords: List[str]
    requires_escalation: bool
    estimated_resolution_hours: float
    summary: str

client = Groq()
response = client.chat.completions.create(
    model="moonshotai/kimi-k2-instruct-0905",
    messages=[
        {"role": "system", "content": "Classify support tickets for efficient routing."},
        {"role": "user", "content": "Hello! I'd like a dark mode feature for the dashboard. Not urgent but would be nice!"}
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "support_ticket_classification",
            "schema": SupportTicket.model_json_schema()
        }
    }
)

result = SupportTicket.model_validate(json.loads(response.choices[0].message.content))
```

## Error Handling

### Strict Mode
No error handling needed - constrained decoding guarantees valid output.

### Best-effort Mode
Implement retry logic for HTTP 400 errors with message "Generated JSON does not match the expected schema."

```python
max_retries = 3
for attempt in range(max_retries):
    try:
        response = client.chat.completions.create(
            model="moonshotai/kimi-k2-instruct-0905",
            messages=[...],
            response_format={"type": "json_schema", "json_schema": {"name": "schema_name", "strict": False, "schema": {...}}}
        )
        data = json.loads(response.choices[0].message.content)
        validate_schema(data)
        break
    except ValidationError as e:
        if attempt == max_retries - 1:
            raise
```

## Best Practices

- **User input handling**: Specify fallback responses for invalid/incompatible inputs to prevent hallucinations
- **Output quality**: Structured outputs ensure schema compliance, not semantic accuracy. Refine instructions for persistent errors.
- **Schema requirements for strict mode**: Mark all fields required, add `additionalProperties: false`, use union types for optional fields (e.g., `{"type": ["string", "null"]}`)

## Migration to Strict Mode

1. Verify model supports `strict: true`
2. Mark all fields as required in schema
3. Add `additionalProperties: false` to all objects
4. Handle optional fields with union types: `{"type": ["string", "null"]}`
5. Update `response_format` to include `"strict": true`
