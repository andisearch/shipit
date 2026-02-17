---
title: "Anthropic Python SDK v0.79 — fast mode for Opus 4.6, structured outputs, and tool runner improvements"
created: 2026-02-16T10:00
platform: reddit
status: draft
tags:
  - python
  - anthropic
  - sdk
  - llm
  - api
---

**TL;DR:** The Anthropic Python SDK has had a bunch of updates recently: structured outputs via `output_config`, a `@beta_tool` decorator that turns plain Python functions into tool definitions, and a new `speed` parameter for fast mode on Claude Opus 4.6. Figured I'd share some of the highlights since the changelog moves fast.

## What the SDK actually does

It's a typed Python wrapper around the Anthropic REST API. You get sync and async clients, streaming, tool use, batching, and token counting, plus separate clients for AWS Bedrock and Google Vertex AI. Built on `httpx` and `pydantic`, it supports Python 3.9+.

```sh
pip install anthropic
```

## Recent changes worth knowing about

**Structured outputs** (v0.77): You can now pass `output_config` to get responses formatted to a JSON schema. No more prompt-engineering your way to valid JSON.

**Tool use with decorators** (v0.76+): The `@beta_tool` decorator pulls type hints and docstrings from your function to build the tool schema automatically. The `tool_runner` handles the back-and-forth loop of tool calls for you:

```python
import json
from anthropic import Anthropic, beta_tool

client = Anthropic()

@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a given city.

    Args:
        location: The city and state, e.g. San Francisco, CA
    """
    return json.dumps({"location": location, "temperature": "68°F", "condition": "Sunny"})

runner = client.beta.messages.tool_runner(
    max_tokens=1024,
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    messages=[{"role": "user", "content": "What is the weather in SF?"}],
)
for message in runner:
    print(message)
```

**Fast mode** (v0.79): Claude Opus 4.6 now supports a `speed` parameter. Haven't benchmarked it extensively yet but the latency difference is noticeable on longer generations.

## Streaming

The streaming API has a nice helper that accumulates text for you:

```python
import asyncio
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

async def main():
    async with client.messages.stream(
        max_tokens=1024,
        messages=[{"role": "user", "content": "Say hello there!"}],
        model="claude-sonnet-4-5-20250929",
    ) as stream:
        async for text in stream.text_stream:
            print(text, end="", flush=True)

asyncio.run(main())
```

## Other bits

- Token counting without creating a message: `client.messages.count_tokens()`
- Auto-pagination on list endpoints
- Configurable retries with exponential backoff (defaults to 2 retries)
- Bedrock and Vertex clients share the same interface, just different auth

Full source and docs: the SDK repo has examples for most of these patterns.

## Question

For those of you using tool use in production — how are you handling tool schemas? Writing them by hand, generating from types, or using something like the `@beta_tool` decorator? Curious whether people prefer explicit schema definitions or the decorator approach.
