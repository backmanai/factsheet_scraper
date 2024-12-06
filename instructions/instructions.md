# Project factsheet scraper

## Project Overview

## Core functionalities

### 1. File Load

- Load the files from the directory example_factsheets
- The program when run should load PDFfiles from the directory
- each file is fully processed one at a time

### 2. Text Extraction

- Use Llamaparse to extract the text from the PDF files
- for each file, combine all the text chunks to one complete text
- strictily follow the Llamaparse documentation as code example
- this step should only be run once and the results need to be stored as markdown document
- the python script facilitating this should be written in a way that it can be easily set to only process one file as the service used to extract is expensive

### 3. Data Extraction

- data extraction part has the goal to extract a set of key financial figures and facts from the text.
- to start the following data must be extracted from the text:
  - the name of the fund
  - the issuer of the fund
  - the investment currency
  - the domicile of the fund
  - the performance figurey YTD, 1Y, 3Y, 5Y for both the fund and the benchmark
  - the name of the benchmark
  - the three largest investments held by the fund
  - the three largest currency holdings
  - the industry allocation
  - the management fees or TER of the fund
- the data must be returned as a JSON object
- the data must be retrieved from the llm using the structured output capabilities, see the chapter "OpenAI structured outputs example" below.

- in addition implement a side variant where the llm is called through llama-index, see the chapter "using the llm through llama-index with vector store" below.

### 4. Data Verification

- a step which will be used to verify the data extracted from the text
- the data verification step will be used to verify the data extracted from the text
- implement the skeleton based on the data extraction step
- the values to validate against will be provided later.
- store the results as a JSON files names after the funds name with a result_ prefix

## Doc

### Using Llamaparse example

Using Python, assume llama-index-core llama-parse llama-index-readers-file python-dotenv have already been installed

The LLAMA API Key has already been set:

```python
LLAMA_CLOUD_API_KEY=....
```

Now we have our libraries and our API key available, let’s create a parse.py file and parse a file. In this case, we're using this list of fun facts about Canada:

#### 1. bring in our LLAMA_CLOUD_API_KEY

```python
from dotenv import load_dotenv
load_dotenv()
```

#### 2. bring in dependencies

```python
from llama_parse import LlamaParse
from llama_index.core import SimpleDirectoryReader
```

#### 3. set up parser

```python
parser = LlamaParse(
    result_type="markdown"  # "markdown" and "text" are available
)
```

#### use SimpleDirectoryReader to parse our file

```python
file_extractor = {".pdf": parser}
documents = SimpleDirectoryReader(input_files=['data/canada.pdf'], file_extractor=file_extractor).load_data()
print(documents)
```

This will print an object that contains the full text of the parsed document. Let’s go a step further, and query this document using an LLM! For this, you will need an OpenAI API key (LlamaIndex supports dozens of LLMs, we're just picking a popular one). Get an OpenAI API key and add it to your .env file:

### Using OpenAI

There are two ways we need to explor. Using the vector store and using the llm through llama-index.
Or providing the entire document to the LLM as prompt context.
In both cases we need to receive structured outputs from the LLM.

#### using the llm through llama-index with vector store

The OpenAI API Key has already been set in the .env file:

```python
OPEN_AI_API_KEY=...
```

The dependencies have already been installed:
llama-index-llms-openai llama-index-embeddings-openai

Now, add these lines to your parse.py:

##### import dependencies

```python
from llama_index.core import VectorStoreIndex
```

##### create an index from the parsed markdown

```python
index = VectorStoreIndex.from_documents(documents)
```

##### create a query engine for the index

```python
query_engine = index.as_query_engine()
```

##### query the engine

```python
query = "What can you do in the Bay of Fundy?"
response = query_engine.query(query)
print(response)
```

Which will give us this output:
You can raft-surf the world’s highest tides at the Bay of Fundy.

Done

#### OpenAI structured outputs example

The OpenAI API Key has already been set in the .env file, just load it with the dotenv library:

```python
OPEN_AI_API_KEY=...
```

[OpenAI Structured Outputs Guide](https://platform.openai.com/docs/guides/structured-outputs)

##### Structured Outputs

Ensure responses follow JSON Schema for Structured Outputs.

##### Introduction

JSON is one of the most widely used formats in the world for applications to exchange data.

Structured Outputs is a feature that ensures the model will always generate responses that adhere to your supplied [JSON Schema](https://json-schema.org/overview/what-is-jsonschema), so you don't need to worry about the model omitting a required key, or hallucinating an invalid enum value.

Some benefits of Structed Outputs include:

1. **Reliable type-safety:** No need to validate or retry incorrectly formatted responses
2. **Explicit refusals:** Safety-based model refusals are now programmatically detectable
3. **Simpler prompting:** No need for strongly worded prompts to achieve consistent formatting

In addition to supporting JSON Schema in the REST API, the OpenAI SDKs for [Python](https://github.com/openai/openai-python/blob/main/helpers.md#structured-outputs-parsing-helpers) and [JavaScript](https://github.com/openai/openai-node/blob/master/helpers.md#structured-outputs-parsing-helpers) also make it easy to define object schemas using [Pydantic](https://docs.pydantic.dev/latest/) and [Zod](https://zod.dev/) respectively. Below, you can see how to extract information from unstructured text that conforms to a schema defined in code.

Getting a structured response

```python
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 from pydantic import BaseModel from openai import OpenAI client = OpenAI() class CalendarEvent(BaseModel): name: str date: str participants: list[str] completion = client.beta.chat.completions.parse( model="gpt-4o-2024-08-06", messages=[ {"role": "system", "content": "Extract the event information."}, {"role": "user", "content": "Alice and Bob are going to a science fair on Friday."}, ], response_format=CalendarEvent, ) event = completion.choices[0].message.parsed
```

##### Supported models

Structured Outputs are available in our [latest large language models](https://platform.openai.com/docs/models), starting with GPT-4o:

- `gpt-4o-mini-2024-07-18` and later
- `gpt-4o-2024-08-06` and later

Older models like `gpt-4-turbo` and earlier may use [JSON mode](https://platform.openai.com/docs/guides/structured-outputs#json-mode) instead.

##### When to use Structured Outputs via function calling vs via response\_format

Structured Outputs is available in two forms in the OpenAI API:

1. When using [function calling](https://platform.openai.com/docs/guides/function-calling)
2. When using a `json_schema` response format

Function calling is useful when you are building an application that bridges the models and functionality of your application.

For example, you can give the model access to functions that query a database in order to build an AI assistant that can help users with their orders, or functions that can interact with the UI.

Conversely, Structured Outputs via `response_format` are more suitable when you want to indicate a structured schema for use when the model responds to the user, rather than when the model calls a tool.

For example, if you are building a math tutoring application, you might want the assistant to respond to your user using a specific JSON Schema so that you can generate a UI that displays different parts of the model's output in distinct ways.

Put simply:

- If you are connecting the model to tools, functions, data, etc. in your system, then you should use function calling
- If you want to structure the model's output when it responds to the user, then you should use a structured `response_format`

The remainder of this guide will focus on non-function calling use cases in the Chat Completions API. To learn more about how to use Structured Outputs with function calling, check out the [Function Calling](https://platform.openai.com/docs/guides/function-calling#function-calling-with-structured-outputs) guide.

##### Structured Outputs vs JSON mode

Structured Outputs is the evolution of [JSON mode](https://platform.openai.com/docs/guides/structured-outputs#json-mode). While both ensure valid JSON is produced, only Structured Outputs ensure schema adherance. Both Structured Outputs and JSON mode are supported in the Chat Completions API, Assistants API, Fine-tuning API and Batch API.

We recommend always using Structured Outputs instead of JSON mode when possible.

However, Structured Outputs with `response_format: {type: "json_schema", ...}` is only supported with the `gpt-4o-mini`, `gpt-4o-mini-2024-07-18`, and `gpt-4o-2024-08-06` model snapshots and later.

|  | Structured Outputs | JSON Mode |
| --- | --- | --- |
| **Outputs valid JSON** | Yes | Yes |
| **Adheres to schema** | Yes (see [supported schemas](https://platform.openai.com/docs/guides/structured-outputs#supported-schemas)) | No |
| **Compatible models** | `gpt-4o-mini`, `gpt-4o-2024-08-06`, and later | `gpt-3.5-turbo`, `gpt-4-*` and `gpt-4o-*` models |
| **Enabling** | `response_format: { type: "json_schema", json_schema: {"strict": true, "schema": ...} }` | `response_format: { type: "json_object" }` |

##### Examples

###### Chain of thought

You can ask the model to output an answer in a structured, step-by-step way, to guide the user through the solution.

Structured Outputs for chain-of-thought math tutoring

```python
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 from pydantic import BaseModel from openai import OpenAI client = OpenAI() class Step(BaseModel): explanation: str output: str class MathReasoning(BaseModel): steps: list[Step] final_answer: str completion = client.beta.chat.completions.parse( model="gpt-4o-2024-08-06", messages=[ {"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."}, {"role": "user", "content": "how can I solve 8x + 7 = -23"} ], response_format=MathReasoning, ) math_reasoning = completion.choices[0].message.parsed
```

###### Example response

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 { "steps": [ { "explanation": "Start with the equation 8x + 7 = -23.", "output": "8x + 7 = -23" }, { "explanation": "Subtract 7 from both sides to isolate the term with the variable.", "output": "8x = -23 - 7" }, { "explanation": "Simplify the right side of the equation.", "output": "8x = -30" }, { "explanation": "Divide both sides by 8 to solve for x.", "output": "x = -30 / 8" }, { "explanation": "Simplify the fraction.", "output": "x = -15 / 4" } ], "final_answer": "x = -15 / 4" }
```

###### How to use Structured Outputs with response\_format

You can use Structured Outputs with the new SDK helper to parse the model's output into your desired format, or you can specify the JSON schema directly.

**Note:** the first request you make with any schema will have additional latency as our API processes the schema, but subsequent requests with the same schema will not have additional latency.

Step 1: Define your object

Step 2: Supply your object in the API call

Step 3: Handle edge cases

Step 4: Use the generated structured data in a type-safe way

###### Refusals with Structured Outputs

When using Structured Outputs with user-generated input, OpenAI models may occasionally refuse to fulfill the request for safety reasons. Since a refusal does not necessarily follow the schema you have supplied in `response_format`, the API response will include a new field called `refusal` to indicate that the model refused to fulfill the request.

When the `refusal` property appears in your output object, you might present the refusal in your UI, or include conditional logic in code that consumes the response to handle the case of a refused request.

```python
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 class Step(BaseModel): explanation: str output: str class MathReasoning(BaseModel): steps: list[Step] final_answer: str completion = client.beta.chat.completions.parse( model="gpt-4o-2024-08-06", messages=[ {"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."}, {"role": "user", "content": "how can I solve 8x + 7 = -23"} ], response_format=MathReasoning, ) math_reasoning = completion.choices[0].message # If the model refuses to respond, you will get a refusal message if (math_reasoning.refusal): print(math_reasoning.refusal) else: print(math_reasoning.parsed)
```

The API response from a refusal will look something like this:

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 { "id": "chatcmpl-9nYAG9LPNonX8DAyrkwYfemr3C8HC", "object": "chat.completion", "created": 1721596428, "model": "gpt-4o-2024-08-06", "choices": [ { "index": 0, "message": { "role": "assistant", "refusal": "I'm sorry, I cannot assist with that request." }, "logprobs": null, "finish_reason": "stop" } ], "usage": { "prompt_tokens": 81, "completion_tokens": 11, "total_tokens": 92, "completion_tokens_details": { "reasoning_tokens": 0, "accepted_prediction_tokens": 0, "rejected_prediction_tokens": 0 } }, "system_fingerprint": "fp_3407719c7f" }
```

##### Tips and best practices

###### Handling user-generated input

If your application is using user-generated input, make sure your prompt includes instructions on how to handle situations where the input cannot result in a valid response.

The model will always try to adhere to the provided schema, which can result in hallucinations if the input is completely unrelated to the schema.

You could include language in your prompt to specify that you want to return empty parameters, or a specific sentence, if the model detects that the input is incompatible with the task.

###### Handling mistakes

Structured Outputs can still contain mistakes. If you see mistakes, try adjusting your instructions, providing examples in the system instructions, or splitting tasks into simpler subtasks. Refer to the [prompt engineering guide](https://platform.openai.com/docs/guides/prompt-engineering) for more guidance on how to tweak your inputs.

###### Avoid JSON schema divergence

To prevent your JSON Schema and corresponding types in your programming language from diverging, we strongly recommend using the native Pydantic/zod sdk support.

If you prefer to specify the JSON schema directly, you could add CI rules that flag when either the JSON schema or underlying data objects are edited, or add a CI step that auto-generates the JSON Schema from type definitions (or vice-versa).

###### Supported schemas

Structured Outputs supports a subset of the [JSON Schema](https://json-schema.org/docs) language.

###### Supported types

The following types are supported for Structured Outputs:

- String
- Number
- Boolean
- Integer
- Object
- Array
- Enum
- anyOf

#### Root objects must not be `anyOf`

Note that the root level object of a schema must be an object, and not use `anyOf`. A pattern that appears in Zod (as one example) is using a discriminated union, which produces an `anyOf` at the top level. So code such as the following won't work:

```javascript
1 2 3 4 5 6 7 8 9 10 11 12 13 import { z } from 'zod'; import { zodResponseFormat } from 'openai/helpers/zod'; const BaseResponseSchema = z.object({ /* ... */ }); const UnsuccessfulResponseSchema = z.object({ /* ... */ }); const finalSchema = z.discriminatedUnion('status', [ BaseResponseSchema, UnsuccessfulResponseSchema, ]); // Invalid JSON Schema for Structured Outputs const json = zodResponseFormat(finalSchema, 'final_schema');
```

#### All fields must be `required`

To use Structured Outputs, all fields or function parameters must be specified as `required`.

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 { "name": "get_weather", "description": "Fetches the weather in the given location", "strict": true, "parameters": { "type": "object", "properties": { "location": { "type": "string", "description": "The location to get the weather for" }, "unit": { "type": "string", "description": "The unit to return the temperature in", "enum": ["F", "C"] } }, "additionalProperties": false, "required": ["location", "unit"] } }
```

Although all fields must be required (and the model will return a value for each parameter), it is possible to emulate an optional parameter by using a union type with `null`.

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 { "name": "get_weather", "description": "Fetches the weather in the given location", "strict": true, "parameters": { "type": "object", "properties": { "location": { "type": "string", "description": "The location to get the weather for" }, "unit": { "type": ["string", "null"], "description": "The unit to return the temperature in", "enum": ["F", "C"] } }, "additionalProperties": false, "required": [ "location", "unit" ] } }
```

#### Objects have limitations on nesting depth and size

A schema may have up to 100 object properties total, with up to 5 levels of nesting.

#### Limitations on total string size

In a schema, total string length of all property names, definition names, enum values, and const values cannot exceed 15,000 characters.

#### Limitations on enum size

A schema may have up to 500 enum values across all enum properties.

For a single enum property with string values, the total string length of all enum values cannot exceed 7,500 characters when there are more than 250 enum values.

#### `additionalProperties: false` must always be set in objects

`additionalProperties` controls whether it is allowable for an object to contain additional keys / values that were not defined in the JSON Schema.

Structured Outputs only supports generating specified keys / values, so we require developers to set `additionalProperties: false` to opt into Structured Outputs.

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 { "name": "get_weather", "description": "Fetches the weather in the given location", "strict": true, "schema": { "type": "object", "properties": { "location": { "type": "string", "description": "The location to get the weather for" }, "unit": { "type": "string", "description": "The unit to return the temperature in", "enum": ["F", "C"] } }, "additionalProperties": false, "required": [ "location", "unit" ] } }
```

#### Key ordering

When using Structured Outputs, outputs will be produced in the same order as the ordering of keys in the schema.

#### Some type-specific keywords are not yet supported

Notable keywords not supported include:

- **For strings:** `minLength`, `maxLength`, `pattern`, `format`
- **For numbers:** `minimum`, `maximum`, `multipleOf`
- **For objects:** `patternProperties`, `unevaluatedProperties`, `propertyNames`, `minProperties`, `maxProperties`
- **For arrays:** `unevaluatedItems`, `contains`, `minContains`, `maxContains`, `minItems`, `maxItems`, `uniqueItems`

If you turn on Structured Outputs by supplying `strict: true` and call the API with an unsupported JSON Schema, you will receive an error.

#### For `anyOf`, the nested schemas must each be a valid JSON Schema per this subset

Here's an example supported anyOf schema:

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 { "type": "object", "properties": { "item": { "anyOf": [ { "type": "object", "description": "The user object to insert into the database", "properties": { "name": { "type": "string", "description": "The name of the user" }, "age": { "type": "number", "description": "The age of the user" } }, "additionalProperties": false, "required": [ "name", "age" ] }, { "type": "object", "description": "The address object to insert into the database", "properties": { "number": { "type": "string", "description": "The number of the address. Eg. for 123 main st, this would be 123" }, "street": { "type": "string", "description": "The street name. Eg. for 123 main st, this would be main st" }, "city": { "type": "string", "description": "The city of the address" } }, "additionalProperties": false, "required": [ "number", "street", "city" ] } ] } }, "additionalProperties": false, "required": [ "item" ] }
```

#### Definitions are supported

You can use definitions to define subschemas which are referenced throughout your schema. The following is a simple example.

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 { "type": "object", "properties": { "steps": { "type": "array", "items": { "$ref": "#/$defs/step" } }, "final_answer": { "type": "string" } }, "$defs": { "step": { "type": "object", "properties": { "explanation": { "type": "string" }, "output": { "type": "string" } }, "required": [ "explanation", "output" ], "additionalProperties": false } }, "required": [ "steps", "final_answer" ], "additionalProperties": false }
```

#### Recursive schemas are supported

Sample recursive schema using `#` to indicate root recursion.

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 { "name": "ui", "description": "Dynamically generated UI", "strict": true, "schema": { "type": "object", "properties": { "type": { "type": "string", "description": "The type of the UI component", "enum": ["div", "button", "header", "section", "field", "form"] }, "label": { "type": "string", "description": "The label of the UI component, used for buttons or form fields" }, "children": { "type": "array", "description": "Nested UI components", "items": { "$ref": "#" } }, "attributes": { "type": "array", "description": "Arbitrary attributes for the UI component, suitable for any element", "items": { "type": "object", "properties": { "name": { "type": "string", "description": "The name of the attribute, for example onClick or className" }, "value": { "type": "string", "description": "The value of the attribute" } }, "additionalProperties": false, "required": ["name", "value"] } } }, "required": ["type", "label", "children", "attributes"], "additionalProperties": false } }
```

Sample recursive schema using explicit recursion:

```json
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 { "type": "object", "properties": { "linked_list": { "$ref": "#/$defs/linked_list_node" } }, "$defs": { "linked_list_node": { "type": "object", "properties": { "value": { "type": "number" }, "next": { "anyOf": [ { "$ref": "#/$defs/linked_list_node" }, { "type": "null" } ] } }, "additionalProperties": false, "required": [ "next", "value" ] } }, "additionalProperties": false, "required": [ "linked_list" ] }
```

## JSON mode

JSON mode is a more basic version of the Structured Outputs feature. While JSON mode ensures that model output is valid JSON, Structured Outputs reliably matches the model's output to the schema you specify. We recommend you use Structured Outputs if it is supported for your use case.

When JSON mode is turned on, the model's output is ensured to be valid JSON, except for in some edge cases that you should detect and handle appropriately.

To turn on JSON mode with the Chat Completions or Assistants API you can set the `response_format` to `{ "type": "json_object" }`. If you are using function calling, JSON mode is always turned on.

Important notes:

- When using JSON mode, you must always instruct the model to produce JSON via some message in the conversation, for example via your system message. If you don't include an explicit instruction to generate JSON, the model may generate an unending stream of whitespace and the request may run continually until it reaches the token limit. To help ensure you don't forget, the API will throw an error if the string "JSON" does not appear somewhere in the context.
- JSON mode will not guarantee the output matches any specific schema, only that it is valid and parses without errors. You should use Structured Outputs to ensure it matches your schema, or if that is not possible, you should use a validation library and potentially retries to ensure that the output matches your desired schema.
- Your application must detect and handle the edge cases that can result in the model output not being a complete JSON object (see below)

## Important notes

tbd
