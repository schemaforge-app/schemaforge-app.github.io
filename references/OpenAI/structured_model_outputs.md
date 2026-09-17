https://developers.openai.com/api/docs/guides/structured-outputs/

# Structured Model Outputs

Structured Outputs ensures the model generates responses that adhere to your supplied JSON Schema, eliminating issues with missing keys or invalid enum values.

## Benefits
- **Reliable type-safety:** No need to validate or retry incorrectly formatted responses
- **Explicit refusals:** Safety-based model refusals are programmatically detectable
- **Simpler prompting:** No need for strongly worded prompts to achieve consistent formatting

## Supported Models
Available in GPT-4o and later. Older models like `gpt-4-turbo` may use JSON mode instead.

## Function Calling vs Response Format
**Use function calling when:** Connecting the model to tools, functions, data in your system
**Use structured `text.format` when:** Structuring the model's output when responding to the user

## Structured Outputs vs JSON Mode

| Feature | Structured Outputs | JSON Mode |
|---------|-------------------|-----------|
| Outputs valid JSON | Yes | Yes |
| Adheres to schema | Yes | No |
| Compatible models | `gpt-4o-mini`, `gpt-4o-2024-08-06`, and later | `gpt-3.5-turbo`, `gpt-4-*` and `gpt-4o-*` models |
| Enabling | `text: { format: { type: "json_schema", "strict": true, "schema": ... } }` | `text: { format: { type: "json_object" } }` |

## Refusals
When using Structured Outputs with user-generated input, models may refuse unsafe requests. The API response includes a `refusal` field to indicate this.

```python
class Step(BaseModel):
    explanation: str
    output: str

class MathReasoning(BaseModel):
    steps: list[Step]
    final_answer: str

completion = client.chat.completions.parse(
    model="gpt-4o-2024-08-06",
    messages=[
        {"role": "system", "content": "You are a helpful math tutor."},
        {"role": "user", "content": "how can I solve 8x + 7 = -23"},
    ],
    response_format=MathReasoning,
)

math_reasoning = completion.choices[0].message
if math_reasoning.refusal:
    print(math_reasoning.refusal)
else:
    print(math_reasoning.parsed)
```

## Best Practices

### Handling User-Generated Input
Include instructions on how to handle situations where the input cannot result in a valid response. The model will always try to adhere to the provided schema, which can result in hallucinations if the input is unrelated.

### Handling Mistakes
Structured Outputs can still contain mistakes. Adjust instructions, provide examples, or split tasks into simpler subtasks.

### Avoid JSON Schema Divergence
Use native Pydantic/Zod SDK support to prevent JSON Schema and types from diverging.

## JSON Mode
JSON mode ensures valid JSON output but does not guarantee schema adherence. Enable with `text.format: { "type": "json_object" }`.

Important notes:
- Always instruct the model to produce JSON in your prompt
- Use validation libraries to ensure output matches your desired schema
- The API will throw an error if "JSON" doesn't appear in the context

## Resources
- [Introductory cookbook](https://developers.openai.com/cookbook/examples/structured_outputs_intro)
- [Multi-agent systems guide](https://developers.openai.com/cookbook/examples/structured_outputs_multi_agent)