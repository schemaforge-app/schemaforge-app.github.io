https://api.llama.com/docs/json-structured-output

# JSON Structured Output

Generate model responses in a specific JSON format for programmatic consumption.

## Benefits
- Consistent format reduces complex parsing logic
- Minimizes processing errors from format variations
- Simplifies integration with APIs, databases, components
- Works well with tool calling

## Use Cases
- Extracting information (names, dates, locations, products)
- Classifying data into predefined categories
- Generating function arguments from natural language
- Producing JSON configuration files

## Combining with Other Features
- **Tool calling**: Define JSON format for tool arguments/results
- **Image understanding**: Extract structured data from images
- **Chat completion**: Use validated structured data as context

## Usage

Set `response_format` parameter with `type: json_schema` and provide JSON structure in `json_schema` field.

```python
import os
import requests

response = requests.post(
    url="https://api.llama.com/v1/chat/completions",
    headers={
        "Content-Type": "application/json",
        "Authorization": f"Bearer {os.environ.get('LLAMA_API_KEY')}"
    },
    json={
        "model": "Llama-4-Maverick-17B-128E-Instruct-FP8",
        "messages": [
            {"role": "system", "content": "Extract the address from the user input into the specified JSON format."},
            {"role": "user", "content": "Please format this address: 1 Hacker Wy Menlo Park CA 94025"}
        ],
        "max_completion_tokens": 1024,
        "temperature": 0.1,
        "response_format": {
            "type": "json_schema",
            "json_schema": {
                "name": "Address",
                "schema": {
                    "properties": {
                        "address": {
                            "type": "object",
                            "properties": {
                                "street": {"type": "string"},
                                "city": {"type": "string"},
                                "state": {"type": "string", "description": "2 letter abbreviation of the state"},
                                "zip": {"type": "string", "description": "5 digit zip code"}
                            },
                            "required": ["street", "city", "state", "zip"]
                        }
                    },
                    "required": ["address"],
                    "type": "object"
                }
            }
        }
    }
)

print(response.json()["completion_message"]["content"]["text"])
```

## Response Format

Structured data returned as JSON-formatted string in `completion_message.content.text` field.

Example response:
```json
{
  "completion_message": {
    "content": {
      "type": "text",
      "text": "{ \"address\": { \"street\": \"1 Hacker Way\", \"city\": \"Menlo Park\", \"state\": \"CA\", \"zip\": \"94025\" } }"
    },
    "role": "assistant",
    "stop_reason": "stop",
    "tool_calls": []
  },
  "metrics": [
    {"metric": "num_completion_tokens", "value": 37, "unit": "tokens"},
    {"metric": "num_prompt_tokens", "value": 38, "unit": "tokens"},
    {"metric": "num_total_tokens", "value": 75, "unit": "tokens"}
  ]
}
```
