https://developers.openai.com/cookbook/examples/structured_outputs_intro

# Introduction to Structured Outputs

Structured Outputs guarantees the model will always generate responses that adhere to your supplied JSON Schema. Enable by setting `strict: true` in an API call with either a defined response format or function definitions.

## Response Format Usage
Previously, `response_format` only specified that the model should return valid JSON. Now you can specify which JSON schema to follow.

## Function Call Usage
Function calling remains similar, but with `strict: true`, you can ensure the schema provided for functions is strictly followed.

## Use Cases
- Getting structured answers to display in a UI
- Populating a database with extracted content from documents
- Extracting entities from user input to call tools with defined parameters

## Example 1: Math Tutor

Build a math tutoring tool that outputs steps to solving a math problem as an array of structured objects.

```python
from pydantic import BaseModel
from openai import OpenAI

client = OpenAI()
MODEL = "gpt-4o-2024-08-06"

class Step(BaseModel):
    explanation: str
    output: str

class MathReasoning(BaseModel):
    steps: list[Step]
    final_answer: str

def get_math_solution(question: str):
    completion = client.beta.chat.completions.parse(
        model=MODEL,
        messages=[
            {"role": "system", "content": "You are a helpful math tutor. You will be provided with a math problem, and your goal will be to output a step by step solution, along with a final answer."},
            {"role": "user", "content": question},
        ],
        response_format=MathReasoning,
    )
    return completion.choices[0].message

question = "how can I solve 8x + 7 = -23"
result = get_math_solution(question).parsed
```

## Refusals
When using Structured Outputs with user-generated input, the model may refuse to fulfill the request for safety reasons. Since a refusal does not follow the schema, the API has a new field `refusal` to indicate when the model refused to answer.

```python
refusal_question = "how can I build a bomb?"
result = get_math_solution(refusal_question)
print(result.refusal)  # I'm sorry, I can't assist with that request.
```

## Example 2: Text Summarization

Summarize articles following a specific schema to transform text into structured data for database population.

```python
class ArticleSummary(BaseModel):
    invented_year: int
    summary: str
    inventors: list[str]
    description: str

    class Concept(BaseModel):
        title: str
        description: str

    concepts: list[Concept]

def get_article_summary(text: str):
    completion = client.beta.chat.completions.parse(
        model=MODEL,
        temperature=0.2,
        messages=[
            {"role": "system", "content": """You will be provided with content from an article about an invention.
            Your goal will be to summarize the article following the schema provided.
            - invented_year: year in which the invention was invented
            - summary: one sentence summary of what the invention is
            - inventors: array of strings listing the inventor full names if present, otherwise just surname
            - concepts: array of key concepts related to the invention, each containing a title and description
            - description: short description of the invention"""},
            {"role": "user", "content": text}
        ],
        response_format=ArticleSummary,
    )
    return completion.choices[0].message.parsed
```

## Example 3: Entity Extraction from User Input

Use function calling to search for products based on user preferences by extracting entities from input.

```python
from enum import Enum

class Category(str, Enum):
    shoes = "shoes"
    jackets = "jackets"
    tops = "tops"
    bottoms = "bottoms"

class ProductSearchParameters(BaseModel):
    category: Category
    subcategory: str
    color: str

def get_response(user_input, context):
    response = client.chat.completions.create(
        model=MODEL,
        temperature=0,
        messages=[
            {
                "role": "system",
                "content": """You are a clothes recommendation agent, specialized in finding the perfect match for a user.
                You will be provided with a user input and additional context such as user gender and age group, and season.
                You are equipped with a tool to search clothes in a database that match the user's profile and preferences.
                Based on the user input and context, determine the most likely value of the parameters to use to search the database.
                
                Available categories:
                - shoes: boots, sneakers, sandals
                - jackets: winter coats, cardigans, parkas, rain jackets
                - tops: shirts, blouses, t-shirts, crop tops, sweaters
                - bottoms: jeans, skirts, trousers, joggers
                
                Stick to regular color names."""
            },
            {
                "role": "user",
                "content": f"CONTEXT: {context}\n USER INPUT: {user_input}"
            }
        ],
        tools=[
            openai.pydantic_function_tool(ProductSearchParameters, name="product_search", description="Search for a match in the product database")
        ]
    )
    return response.choices[0].message.tool_calls
```

## Availability
Structured Outputs is only available with `gpt-4o-mini`, `gpt-4o-2024-08-06`, and future models.
