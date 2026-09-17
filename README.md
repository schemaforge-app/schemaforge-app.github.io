# SchemaForge

SchemaForge is an interactive, browser-based tool for designing, validating, and exporting structured output schemas across major Large Language Model (LLM) providers (including Anthropic Claude, OpenAI, Google Gemini, DeepSeek, Groq, Mistral, Meta Llama, and xAI).

## Features

- **Visual Schema Builder**: Build hierarchical JSON schemas with nested objects, arrays, types, and validation constraints.
- **Multi-Provider Export**: Generate provider-compliant structured output configurations, Pydantic models, JSON Schema definitions, and API client snippets.
- **Provider Reference Library**: Includes structured outputs documentation and best practices across supported AI providers.
- **Zero Build Step**: Fully self-contained static HTML/CSS/JS application running entirely client-side.

## Getting Started

Because SchemaForge is a pure static site, you can run it locally with no build tooling required:

### Option 1: Direct File Open
Open `index.html` directly in any modern web browser.

### Option 2: Local HTTP Server (Recommended)
Serve the directory using Python's built-in HTTP server or any local static server:

```bash
python3 -m http.server 8000
```

Then navigate to `http://localhost:8000` in your web browser.

## Deployment & Extraction

This standalone repository was extracted from Jesus Villota's personal website ([jesusvillota.github.io](https://jesusvillota.github.io)) to live at its own dedicated GitHub Pages root URL: [https://schema-forge.github.io](https://schema-forge.github.io) once published under the `schema-forge` GitHub organization.
