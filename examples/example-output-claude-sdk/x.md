---
title: "Anthropic Python SDK v0.79 — Fast Mode and Claude Opus 4.6"
created: 2026-02-16T12:00
platform: x
status: draft
tags:
  - anthropic
  - python
  - sdk
  - claude
  - ai
  - developer-tools
---

## 1

Most API wrappers just serialize JSON and call it a day.

The Anthropic Python SDK does something different — typed Pydantic models for every request and response, streaming with event iterators, and a tool runner that turns decorated Python functions into Claude-callable tools.

## 2

Five lines to talk to Claude:

```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-opus-4-6",
)
```

Sync, async, streaming — same interface either way.

## 3

Tool use without the boilerplate. Decorate a function, pass it in, and the SDK handles schema extraction, invocation, and response routing:

```python
@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a given city"""
    return json.dumps({"temperature": "68°F"})

runner = client.beta.messages.tool_runner(
    tools=[get_weather], ...
)
```

## 4

Streaming that doesn't make you manage chunked responses manually:

```python
async with client.messages.stream(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Say hello"}],
    model="claude-sonnet-4-5-20250929",
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

Text accumulation and SDK-specific events built in.

## 5

v0.79 adds fast mode for Claude Opus 4.6 — a new `speed` parameter that trades some depth for faster output.

Same model, same capabilities, just quicker when you need it.

## 6

The SDK also covers the full API surface now:

- Structured outputs via `output_config`
- Message batches for bulk async processing
- Token counting without creating a message
- Prompt caching to cut repeated token costs
- Adaptive thinking with context management

## 7

One SDK, three providers. Same code works across direct API, AWS Bedrock, and Google Vertex AI. Swap the client constructor and everything else stays the same.

```python
from anthropic import AnthropicBedrock
client = AnthropicBedrock()
```

## 8

If you're building with Claude in Python, this is the maintained, typed, production-grade way to do it. Python 3.9+, retries with backoff out of the box, and auto-pagination for list endpoints.

`pip install anthropic`

## Reply (Link)

GitHub: github.com/anthropics/anthropic-sdk-python

Docs and changelog in the repo. Latest release is v0.79.0.
