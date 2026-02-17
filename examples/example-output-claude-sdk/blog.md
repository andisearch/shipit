---
title: "Anthropic Python SDK v0.79: Fast Mode, Structured Outputs"
description: "The Anthropic Python SDK adds fast mode for Claude Opus 4.6, structured outputs, adaptive thinking, and tool use decorators in recent releases."
created: 2026-02-16T10:00
platform: blog
status: draft
tags:
  - python
  - sdk
  - anthropic
  - claude
  - api
keywords:
  - Anthropic Python SDK
  - Claude API Python
  - Claude Opus 4.6
  - structured outputs API
---

The Anthropic Python SDK has picked up a string of updates over the past few weeks. If you're building with Claude in Python, here's what landed and how to use it.

## What the SDK does

The `anthropic` package is a typed Python wrapper around the Anthropic REST API, covering sync and async clients, streaming, tool use, message batches, token counting, and integrations with AWS Bedrock and Google Vertex AI. It uses `httpx` for transport and Pydantic for request/response models.

```sh
pip install anthropic
```

Set your API key and you're ready:

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

## What's new in v0.78–0.79

**Claude Opus 4.6 and fast mode.** Version 0.78 added support for Claude Opus 4.6, and v0.79 introduced a `speed` parameter that enables fast mode on that model. A bug fix in v0.79 also ensures the speed parameter passes through correctly in the sync beta `count_tokens` path.

**Structured outputs.** Starting in v0.77, you can constrain Claude's responses to match a JSON schema via `output_config` in the Messages API. This is useful when you need machine-readable output without hoping the model follows your formatting instructions.

**Adaptive thinking.** The SDK now supports thinking and reasoning blocks with context management, letting you work with Claude's intermediate reasoning steps.

## Tool use with decorators

The `@beta_tool` decorator turns a plain Python function into a tool Claude can call. The SDK extracts the function's docstring and type hints to build the tool schema automatically:

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

The `tool_runner` handles the back-and-forth of tool invocation for you: Claude decides to call `get_weather`, the runner executes it, and feeds the result back.

## Streaming

Two ways to stream. The low-level approach gives you raw SSE events:

```python
stream = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-sonnet-4-5-20250929",
    stream=True,
)
for event in stream:
    print(event.type)
```

The higher-level `messages.stream()` helper accumulates text and exposes a `text_stream` iterator, which is cleaner for most use cases:

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
        print()
    message = await stream.get_final_message()
    print(message.to_json())

asyncio.run(main())
```

## Token counting

You can check how many tokens a request would use before sending it:

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

This hits a dedicated endpoint without creating a message, so you're not billed for a completion.

## Installation options

The base package covers the standard API. Extras pull in dependencies for specific integrations:

```sh
pip install anthropic[bedrock]   # AWS Bedrock
pip install anthropic[vertex]    # Google Vertex AI
pip install anthropic[aiohttp]   # aiohttp async backend
```

The SDK requires Python 3.9+ and handles retries with exponential backoff (two retries by default) and a 10-minute default timeout. Both are configurable on the client.

## Links and resources

The full changelog and API reference live in the [anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) repository. If you're upgrading from an older version, the structured outputs feature in v0.77+ and the fast mode parameter in v0.79 are the two changes most likely to affect your code. Both are additive, so existing code won't break.
