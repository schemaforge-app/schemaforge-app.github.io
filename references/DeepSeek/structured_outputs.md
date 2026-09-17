https://api-docs.deepseek.com/guides/json_mode/

# JSON Output

DeepSeek provides JSON Output to ensure the model outputs valid JSON strings.

## Requirements
- Set `response_format` to `{'type': 'json_object'}`
- Include "json" in the system or user prompt with an example of desired format
- Set `max_tokens` reasonably to prevent truncation
- Note: API may occasionally return empty content

## Python Example

```python
import json
from openai import OpenAI

client = OpenAI(
    api_key="<your api key>",
    base_url="https://api.deepseek.com",
)

system_prompt = """
Parse the "question" and "answer" and output in JSON format.
EXAMPLE: {"question": "Which is the highest mountain?", "answer": "Mount Everest"}
"""

messages = [
    {"role": "system", "content": system_prompt},
    {"role": "user", "content": "Which is the longest river in the world? The Nile River."}
]

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=messages,
    response_format={'type': 'json_object'}
)

print(json.loads(response.choices[0].message.content))
# Output: {"question": "Which is the longest river in the world?", "answer": "The Nile River"}
```