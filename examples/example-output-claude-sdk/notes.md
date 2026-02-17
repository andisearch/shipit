---
title: "Anthropic Python SDK — v0.79.0"
created: 2026-02-16T00:00
platform: notes
status: draft
tags:
  - release
  - anthropic
  - python-sdk
---

## Added

- Fast mode for Claude Opus 4.6: new `speed` parameter lets you opt into faster output from `claude-opus-4-6`
- Structured Outputs: JSON schema-based output formatting via `output_config` in the Messages API (v0.77.0)
- Adaptive thinking: support for thinking/reasoning blocks with context management (v0.78.0)
- Server-side tools in tool runner: tool runner now handles server-side tools alongside client-defined ones (v0.76.0)
- Raw JSON schema support in `messages.stream()` (v0.76.0)
- Binary request streaming (v0.76.0)
- Custom JSON encoder for extended type support in requests (v0.77.1)

## Changed

- Claude Opus 4.6 model now available as `claude-opus-4-6` (v0.78.0)

## Fixed

- `speed` parameter now passes through correctly in sync beta `count_tokens` (v0.79.0)
- Structured output beta header no longer sent when format is omitted (v0.77.1)

## Usage Examples

**Fast mode with Claude Opus 4.6:**

```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-opus-4-6",
    speed="fast",
)
print(message.content)
```

**Structured outputs:**

```python
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Describe the weather"}],
    model="claude-sonnet-4-5-20250929",
    output_config={"json_schema": your_schema},
)
```

**Token counting:**

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

**Tool use with `@beta_tool` decorator:**

```python
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
