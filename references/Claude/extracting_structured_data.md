https://platform.claude.com/cookbook/tool-use-extracting-structured-json

# Extracting Structured JSON using Claude and Tool Use

Use Claude's tool use feature to extract structured JSON data. Define custom tools with input schemas to guide Claude toward well-structured output. Alternative: [JSON mode](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/how_to_enable_json_mode.ipynb).

## Setup

```python
import json, requests
from anthropic import Anthropic
from bs4 import BeautifulSoup
client = Anthropic()
MODEL_NAME = "claude-haiku-4-5"
```

## Article Summarization

```python
tools = [{"name": "print_summary", "description": "Prints a summary of the article.", "input_schema": {"type": "object", "properties": {"author": {"type": "string"}, "topics": {"type": "array", "items": {"type": "string"}}, "summary": {"type": "string"}, "coherence": {"type": "integer"}, "persuasion": {"type": "number"}}, "required": ["author", "topics", "summary", "coherence", "persuasion"]}}]
response = client.messages.create(model=MODEL_NAME, max_tokens=4096, tools=tools, messages=[{"role": "user", "content": f"<article>{article}</article>\nUse the `print_summary` tool."}])
for content in response.content:
    if content.type == "tool_use" and content.name == "print_summary":
        json_summary = content.input
```

## Named Entity Recognition

```python
tools = [{"name": "print_entities", "description": "Prints extract named entities.", "input_schema": {"type": "object", "properties": {"entities": {"type": "array", "items": {"type": "object", "properties": {"name": {"type": "string"}, "type": {"type": "string"}, "context": {"type": "string"}}, "required": ["name", "type", "context"]}}}, "required": ["entities"]}}]
response = client.messages.create(model=MODEL_NAME, max_tokens=4096, tools=tools, messages=[{"role": "user", "content": f"<document>{text}</document>\nUse the print_entities tool."}])
```

## Sentiment Analysis

```python
tools = [{"name": "print_sentiment_scores", "description": "Prints the sentiment scores of a given text.", "input_schema": {"type": "object", "properties": {"positive_score": {"type": "number"}, "negative_score": {"type": "number"}, "neutral_score": {"type": "number"}}, "required": ["positive_score", "negative_score", "neutral_score"]}}]
```

## Text Classification

```python
tools = [{"name": "print_classification", "description": "Prints the classification results.", "input_schema": {"type": "object", "properties": {"categories": {"type": "array", "items": {"type": "object", "properties": {"name": {"type": "string"}, "score": {"type": "number"}}, "required": ["name", "score"]}}}, "required": ["categories"]}}]
```

## Unknown Keys

For dynamic schemas, use `additionalProperties: True`:

```python
tools = [{"name": "print_all_characteristics", "description": "Prints all characteristics which are provided.", "input_schema": {"type": "object", "additionalProperties": True}}]
response = client.messages.create(model=MODEL_NAME, max_tokens=4096, tools=tools, tool_choice={"type": "tool", "name": "print_all_characteristics"}, messages=[{"role": "user", "content": query}])
```

Define custom tools with specific input schemas to generate well-structured JSON output for NLP tasks.
