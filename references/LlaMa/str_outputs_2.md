Get started
Essentials
Features
Guides
API reference
Resources
JSON structured output
While chat completion is optimized for human-readable model responses, sometimes the audience for a model response is another process rather than an end user.
Use JSON structured output to get model responses in a specific JSON format that you define, then use the structured response directly as part of your application’s business logic.
JSON structured output offers several benefits over plain text responses:
•Consistent format: Guarantees the output follows your defined structure, reducing the need for complex parsing logic.
•Reduced processing errors: Minimizes errors caused by unexpected variations in the model's response format.
•Simpler integration: Enables you to directly use the model's output with APIs, databases, or other components that expect structured data.
•Better tool interaction: Works well with tool calling by providing structured data that external tools can easily use.
Use cases

Structured output is especially helpful for tasks such as:
•Extracting information: Pulling specific details such as names, dates, locations, or product information from unstructured text.
•Classifying data: Categorizing user input or text into predefined categories.
•Generating function arguments: Creating structured arguments for other functions or APIs based on natural language prompts.
•Generating configurations: Producing JSON-based configuration files from user requirements.
Combining with other Llama API features

You can effectively combine JSON structured output with other Llama API features:
•Tool calling: Use schemas to define a consistent JSON format for arguments passed to your tools or for results returned from your tools that the LLM needs to process.
•Image understanding: Extract structured data from images, such as detected objects or recognized text, and output it in JSON format.
•Chat completion: Use validated, structured data from one turn as reliable context for the next messages in a conversation.
How to use JSON structured output

To request a response with JSON structured output, specify a JSON schema in your request, by using the response_format parameter in your API request. Set its type field to json_schema and provide your desired JSON structure in the json_schema field. The model will then return output matching the schema you provided in the request.
Request example
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354
import osimport requests      response = requests.post(    url="https://api.llama.com/v1/chat/completions",     headers={        "Content-Type": "application/json",        "Authorization": f"Bearer {os.environ.get('LLAMA_API_KEY')}"    },    json={        "model": "Llama-4-Maverick-17B-128E-Instruct-FP8",        "messages": [            {                "role": "system",                "content": "Extract the address from the user input into the specified JSON format."            },            {                "role": "user",                "content": "Please format this address: 1 Hacker Wy Menlo Park CA 94025"            }        ],        "max_completion_tokens": 1024,        "temperature": 0.1,        "response_format": {            "type": "json_schema",            "json_schema": {                "name": "Address",                "schema": {                    "properties": {                        "address": {                        "type": "object",                        "properties": {                            "street": {"type": "string"},                            "city": {"type": "string"},                            "state": {                            "type": "string",                             "description": "2 letter abbreviation of the state"                            },                            "zip": {                            "type": "string",                             "description": "5 digit zip code"                            }                        },                        "required": ["street", "city", "state", "zip"]                        }                    },                    "required": ["address"],                    "type": "object"                }            }        },    })print(response.json()["completion_message"]["content"]["text"])

Response example
The API returns the structured data as a JSON-formatted string within the completion_message.content.text field.
JSON
{  "completion_message": {    "content": {      "type": "text",      "text": "{ \"address\": { \"street\": \"1 Hacker Way\", \"city\": \"Menlo Park\", \"state\": \"CA\", \"zip\": \"94025\" } }"    },    "role": "assistant",    "stop_reason": "stop",    "tool_calls": []  },  "metrics": [    {      "metric": "num_completion_tokens",      "value": 37,      "unit": "tokens"    },    {      "metric": "num_prompt_tokens",      "value": 38,      "unit": "tokens"    },    {      "metric": "num_total_tokens",      "value": 75,      "unit": "tokens"    }  ]}

Next steps

Explore the full capabilities of Llama API with these resources:
•Extended Guide: See the Chat and conversation guide for examples of multi-turn conversations, memory management, and streaming.
•API Reference: Read the chat completion API reference for specific parameters and endpoint details.
Was this page helpful?



Get started
Essentials
Features
Guides
API reference
Resources
JSON structured output
While chat completion is optimized for human-readable model responses, sometimes the audience for a model response is another process rather than an end user.
Use JSON structured output to get model responses in a specific JSON format that you define, then use the structured response directly as part of your application’s business logic.
JSON structured output offers several benefits over plain text responses:
•Consistent format: Guarantees the output follows your defined structure, reducing the need for complex parsing logic.
•Reduced processing errors: Minimizes errors caused by unexpected variations in the model's response format.
•Simpler integration: Enables you to directly use the model's output with APIs, databases, or other components that expect structured data.
•Better tool interaction: Works well with tool calling by providing structured data that external tools can easily use.
Use cases

Structured output is especially helpful for tasks such as:
•Extracting information: Pulling specific details such as names, dates, locations, or product information from unstructured text.
•Classifying data: Categorizing user input or text into predefined categories.
•Generating function arguments: Creating structured arguments for other functions or APIs based on natural language prompts.
•Generating configurations: Producing JSON-based configuration files from user requirements.
Combining with other Llama API features

You can effectively combine JSON structured output with other Llama API features:
•Tool calling: Use schemas to define a consistent JSON format for arguments passed to your tools or for results returned from your tools that the LLM needs to process.
•Image understanding: Extract structured data from images, such as detected objects or recognized text, and output it in JSON format.
•Chat completion: Use validated, structured data from one turn as reliable context for the next messages in a conversation.
How to use JSON structured output

To request a response with JSON structured output, specify a JSON schema in your request, by using the response_format parameter in your API request. Set its type field to json_schema and provide your desired JSON structure in the json_schema field. The model will then return output matching the schema you provided in the request.
Request example
Python (Llama API client)
123456789101112131415161718192021222324252627282930313233
from llama_api import LlamaAPIfrom pydantic import BaseModelclass Address(BaseModel):    street: str    city: str    state: str    zip: str        client = LlamaAPI()response = client.chat.completions.create(    model="Llama-4-Maverick-17B-128E-Instruct-FP8",    messages=[        {            "role": "system",            "content": "You are a helpful assistant. Summarize the address in a JSON object.",        },        {            "role": "user",            "content": "123 Main St, Anytown, USA",        },    ],    temperature=0.1,    response_format={        "type": "json_schema",        "json_schema": {            "name": Address.__name__,            "schema": Address.model_json_schema(),        },    },)print(response.completion_message.content.text)

Response example
The API returns the structured data as a JSON-formatted string within the completion_message.content.text field.
JSON
{  "completion_message": {    "content": {      "type": "text",      "text": "{ \"address\": { \"street\": \"1 Hacker Way\", \"city\": \"Menlo Park\", \"state\": \"CA\", \"zip\": \"94025\" } }"    },    "role": "assistant",    "stop_reason": "stop",    "tool_calls": []  },  "metrics": [    {      "metric": "num_completion_tokens",      "value": 37,      "unit": "tokens"    },    {      "metric": "num_prompt_tokens",      "value": 38,      "unit": "tokens"    },    {      "metric": "num_total_tokens",      "value": 75,      "unit": "tokens"    }  ]}

Next steps

Explore the full capabilities of Llama API with these resources:
•Extended Guide: See the Chat and conversation guide for examples of multi-turn conversations, memory management, and streaming.
•API Reference: Read the chat completion API reference for specific parameters and endpoint details.
Was this page helpful?
