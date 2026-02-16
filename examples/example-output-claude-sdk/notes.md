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

- **Fast mode for Claude Opus 4.6** — New `speed` parameter enables fast-mode output on Claude Opus 4.6 models
- **Adaptive thinking** — Support for thinking/reasoning content blocks with context management
- **Structured outputs** — JSON schema-based output formatting via `output_config` parameter in the Messages API
- **Server-side tools in tool runner** — `client.beta.messages.tool_runner()` now supports server-side tools
- **Raw JSON schema support** in `messages.stream()`
- **Binary request streaming**

## Changed

- New `BetaUsage` fields added to beta response models
- Custom JSON encoder for extended type support in structured outputs

## Fixed

- `speed` parameter now correctly passes through in sync beta `count_tokens`
- Structured output beta header fix

## Implementation Examples

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
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Describe the weather in SF"}],
    model="claude-sonnet-4-5-20250929",
    output_config={"json_schema": {"name": "weather", "schema": {"type": "object", "properties": {"temperature": {"type": "string"}, "condition": {"type": "string"}}}}},
)
```

**Tool use with automatic execution:**

```python
from anthropic import Anthropic, beta_tool

client = Anthropic()

@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a given city.

    Args:
        location: The city and state, e.g. San Francisco, CA
    """
    return '{"location": location, "temperature": "68°F", "condition": "Sunny"}'

runner = client.beta.messages.tool_runner(
    max_tokens=1024,
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    messages=[{"role": "user", "content": "What is the weather in SF?"}],
)
for message in runner:
    print(message)
```
