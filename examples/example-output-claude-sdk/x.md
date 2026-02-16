---
title: "Anthropic Python SDK v0.79 — Fast Mode, Structured Outputs, Tool Runner"
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

Most API wrappers give you untyped dicts and leave you to figure out the rest.

The best ones disappear — you just write Python and the API call happens to work.

That's what the Anthropic Python SDK actually does well.

## 2

Streaming responses in four lines:

```python
stream = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
    model="claude-sonnet-4-5-20250929",
    stream=True,
)
for event in stream:
    print(event.type)
```

No websocket setup, no callback wiring. Just iterate.

## 3

The tool runner is where it gets interesting. Decorate a function, pass it in, and the SDK handles the entire tool-use loop automatically:

```python
@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a city"""
    return json.dumps({"location": location, "temperature": "68°F"})

runner = client.beta.messages.tool_runner(
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    messages=[{"role": "user", "content": "Weather in SF?"}],
)
```

## 4

New in v0.77: structured outputs via `output_config`.

Instead of parsing free text or hoping the model follows your format instructions, you get JSON schema-validated responses directly from the API.

## 5

v0.79 adds fast mode for Claude Opus 4.6 — a `speed` parameter that gets you faster output from the same model.

Same API surface, one extra parameter. No model swap, no quality tradeoff flags.

## 6

Token counting before you send:

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

Know your costs upfront instead of after the fact.

## 7

The SDK also handles Bedrock and Vertex natively. Swap `Anthropic()` for `AnthropicBedrock()` or `AnthropicVertex()` and the auth, routing, and endpoint differences are handled for you.

One codebase, three deployment targets.

## 8

Async support mirrors the sync API exactly — `AsyncAnthropic` gives you the same methods with `await`. Works with both httpx and aiohttp backends.

Pick your async HTTP library, the SDK adapts.

## 9

If you're building with Claude in Python, this is the starting point. Typed responses, automatic retries with backoff, Pydantic models throughout.

`pip install anthropic` and go.

## Reply (Link)

GitHub: github.com/anthropics/anthropic-sdk-python

Docs: docs.anthropic.com/en/api/client-sdks/python
