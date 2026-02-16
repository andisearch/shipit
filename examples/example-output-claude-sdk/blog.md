---
title: "Anthropic Python SDK: Typed API Access for Claude Models"
description: "The Anthropic Python SDK provides sync and async clients, streaming, tool use, structured outputs, and integrations with Bedrock and Vertex AI."
created: 2026-02-16T12:00
platform: blog
status: draft
tags:
  - python
  - sdk
  - anthropic
  - claude
  - api
keywords:
  - anthropic python sdk
  - claude api python
  - anthropic sdk streaming
  - claude tool use python
---

The Anthropic Python SDK gives you typed, ergonomic access to the full Claude API from Python 3.9+. It covers everything from simple message creation to streaming, tool use, batch processing, and token counting, with both synchronous and asynchronous clients that share an identical interface.

## Getting started

Install the base package or add extras for your cloud provider:

```sh
pip install anthropic
pip install anthropic[bedrock]    # AWS Bedrock support
pip install anthropic[vertex]     # Google Vertex support
pip install anthropic[aiohttp]    # aiohttp async backend
```

Set your `ANTHROPIC_API_KEY` environment variable (or pass `api_key` directly), and you're ready to go:

```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-sonnet-4-5-20250929",
)
print(message.content)
```

Every API response comes back as a Pydantic model, so you get full type checking and autocompletion in your editor without any extra work.

## Streaming

For longer responses or real-time output, the SDK supports both low-level SSE iteration and a higher-level streaming helper that accumulates text for you:

```python
import asyncio
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

async def main() -> None:
    async with client.messages.stream(
        max_tokens=1024,
        messages=[{"role": "user", "content": "Say hello there!"}],
        model="claude-sonnet-4-5-20250929",
    ) as stream:
        async for text in stream.text_stream:
            print(text, end="", flush=True)
    message = await stream.get_final_message()
    print(message.to_json())

asyncio.run(main())
```

The `stream=True` parameter on `messages.create()` gives you raw SSE events if you prefer finer control. The `.stream()` helper handles accumulation and exposes SDK-specific events on top.

## Tool use

Claude can call functions you define, and the SDK makes wiring that up straightforward. Decorate a Python function with `@beta_tool`, pass it to the tool runner, and the SDK handles the call loop automatically — sending the model's tool requests to your functions and feeding results back:

```python
import json
import rich
from anthropic import Anthropic, beta_tool

client = Anthropic()

@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a given city in either celsius or fahrenheit

    Args:
        location: The city and state, e.g. San Francisco, CA
    Returns:
        A dictionary containing the location, temperature, and weather condition.
    """
    return json.dumps({"location": location, "temperature": "68°F", "condition": "Sunny"})

runner = client.beta.messages.tool_runner(
    max_tokens=1024,
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    messages=[{"role": "user", "content": "What is the weather in SF?"}],
)
for message in runner:
    rich.print(message)
```

The decorator extracts parameter types and descriptions from your function's type hints and docstring, so there's no schema boilerplate to maintain.

## Token counting

If you need to check input size before sending a request — for cost estimation or context window management — there's a dedicated endpoint:

```python
from anthropic import Anthropic

client = Anthropic()
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

## What shipped in recent releases

The v0.78–0.79 releases added support for Claude Opus 4.6, including a `speed` parameter for fast-mode output and adaptive thinking with context management. Structured outputs arrived in v0.77, letting you constrain responses to a JSON schema via the `output_config` parameter. The v0.76 release brought server-side tools support in the tool runner and raw JSON schema support in the streaming helper.

Other features available across the SDK: message batches for bulk async processing, automatic retries with exponential backoff on transient errors, file uploads, citation content blocks, and auto-paginating iterators for list endpoints.

## Cloud provider integrations

The SDK ships dedicated clients for AWS Bedrock (`AnthropicBedrock`) and Google Vertex AI (`AnthropicVertex` / `AsyncAnthropicVertex`). Each handles its provider's authentication flow — AWS profiles, regions, and session tokens for Bedrock; Google credentials for Vertex — while exposing the same message creation interface as the standard client.

## Under the hood

The SDK is generated via Stainless and organized around resource classes that map to API endpoints. HTTP goes through httpx (with optional aiohttp for async), models are Pydantic, and async coordination uses anyio. The architecture means you can swap between sync and async with minimal code changes, and the type system catches malformed requests before they hit the network.

Full documentation, additional examples, and the source code are available in the [anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) repository.
