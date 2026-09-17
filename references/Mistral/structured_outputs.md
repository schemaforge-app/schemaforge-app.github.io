https://docs.mistral.ai/capabilities/structured_output

# Structured Outputs

Mistral provides two methods for structured outputs: Custom Structured Outputs (recommended) and JSON Mode.

## Custom Structured Outputs

Use JSON schema to enforce specific format with correct typing and keywords. More reliable than JSON mode.

### Python Example with Pydantic

```python
from pydantic import BaseModel
import os
from mistralai import Mistral

class Book(BaseModel):
    name: str
    authors: list[str]

api_key = os.environ["MISTRAL_API_KEY"]
client = Mistral(api_key=api_key)

chat_response = client.chat.parse(
    model="ministral-8b-latest",
    messages=[
        {"role": "system", "content": "Extract the books information."},
        {"role": "user", "content": "I recently read 'To Kill a Mockingbird' by Harper Lee."}
    ],
    response_format=Book,
    max_tokens=256,
    temperature=0
)
```

System prompt automatically prepended: "Your output should be an instance of a JSON object following this schema: {{ json_schema }}"

## JSON Mode

Ensures valid JSON output with flexible structure. Explicitly instruct model to output JSON and specify format in prompt.

### Python Example

```python
import os
from mistralai import Mistral

api_key = os.environ["MISTRAL_API_KEY"]
client = Mistral(api_key=api_key)

messages = [
    {"role": "user", "content": "What is the best French meal? Return the name and the ingredients in short JSON object."}
]

chat_response = client.chat.complete(
    model="mistral-large-latest",
    messages=messages,
    response_format={"type": "json_object"}
)
# Output example: {"meal": "Boeuf Bourguignon", "ingredients": ["beef", "red wine (Burgundy)", "onions", ...]}
```
