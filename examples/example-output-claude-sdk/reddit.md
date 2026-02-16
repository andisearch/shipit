---
title: "Anthropic Python SDK v0.79: fast mode for Opus 4.6, structured outputs, and tool runner improvements"
created: 2026-02-16T12:00
platform: reddit
status: draft
tags:
  - python
  - anthropic
  - sdk
  - claude
  - api
---

**TL;DR:** The Anthropic Python SDK has had a bunch of updates recently — structured outputs via `output_config`, a `@beta_tool` decorator that handles tool execution loops automatically, fast mode for Opus 4.6, and adaptive thinking. Figured I'd share what's new since most of these shipped without much fanfare.

## What the SDK actually does

`anthropic` is the official typed Python client for the Anthropic REST API. Sync and async clients, streaming, tool use, batches, token counting, plus Bedrock and Vertex integrations. Built on httpx and Pydantic.

```sh
pip install anthropic
```

## What's new

### Structured outputs

You can now pass `output_config` to get JSON schema-enforced responses directly from the API, instead of hoping the model follows your formatting instructions. Landed in v0.77.

### Tool runner with `@beta_tool`

The decorator approach cuts down on boilerplate compared to manually defining tool schemas. The runner handles the call-respond loop for you:

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

It pulls the schema from your function signature and docstring, calls the model, executes the tool when requested, feeds the result back, and loops until the model is done. Server-side tools are also supported now (v0.76).

### Fast mode

v0.79 added a `speed` parameter for Opus 4.6. Same model, faster output when you don't need maximum deliberation.

### Streaming

The `.stream()` helper accumulates text and gives you SDK-level events, which is nicer than parsing raw SSE yourself:

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

### Token counting

Pre-flight token counts without burning an API call:

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

## Additional features

- Automatic retries with exponential backoff on 429s and 5xx
- Auto-paginating iterators for list endpoints
- File uploads via `client.beta.files.upload()`
- Citation content blocks
- Bedrock and Vertex clients handle their own auth flows separately

Full changelog and docs are on the [GitHub repo](https://github.com/anthropics/anthropic-sdk-python).

## Question

For those using the SDK in production — are you doing anything interesting with the tool runner, or still wiring up tool calls manually? Curious whether people are hitting limitations with the `@beta_tool` approach or if it covers most use cases.
