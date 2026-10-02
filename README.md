# SchemaForge

**SchemaForge** is an interactive tool for designing, validating, and exporting structured-output schemas for Large Language Models.

**Website:** https://schemaforge-app.github.io

SchemaForge provides a visual interface for building hierarchical JSON schemas and translating them into provider-specific structured-output configurations. It supports the major LLM providers covered by the application, including OpenAI, Anthropic Claude, Google Gemini, DeepSeek, Groq, Mistral, Meta Llama, and xAI.

## What SchemaForge offers

- **Visual schema building** for objects, arrays, nested fields, types, required fields, enums, and validation constraints.
- **Provider-aware validation** that identifies schema features or constraints that are unsupported by a selected provider or model.
- **Provider-specific exports** for structured-output APIs and SDKs.
- **JSON Schema and Pydantic generation** from the same visual specification.
- **Ready-to-use API examples** showing how to use the generated schema with different LLM providers.
- **Structured-output reference material** summarizing provider capabilities, supported approaches, and practical limitations.

The aim is to make structured outputs easier to design correctly while making differences in implementation and schema support across LLM providers explicit.

## Conceptual foundation

SchemaForge is built around the framework developed in Jesús Villota's paper **[Structured Data with LLMs Done Right](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5636430)**.

The paper develops the practical approach that underlies the application: how to obtain reliable structured data from Large Language Models, how schemas and validation should be used, and why provider-specific capabilities and limitations need to be handled explicitly rather than treated as interchangeable.
