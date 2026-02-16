# Anthropic Python SDK — v0.79.0

## Added
- **Fast mode for Claude Opus 4.6** — New `speed` parameter enables faster output from Claude Opus 4.6
- **Adaptive thinking** — Support for extended thinking and reasoning traces in Claude Opus 4.6
- **Structured Outputs** — Constrain model responses to JSON schemas via `output_config` in the Messages API
- **Claude Opus 4.6 model support** — Full SDK support for the latest Claude model

## Changed
- Custom JSON encoder for extended type support
- Raw JSON schema passthrough for `messages.stream()`
- Binary request streaming support
- Server-side tools support in the tool runner

## Fixed
- Speed parameter passthrough in sync beta `count_tokens`
- Structured output beta header behavior
- Stream closure issues

## Usage Examples

**Fast mode with Claude Opus 4.6:**
```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-opus-4-6",
)
print(message.content)
```

**Streaming with text accumulation:**
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

asyncio.run(main())
```

**Tool use with `@beta_tool` decorator:**
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

**Structured outputs:**
```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Give me a JSON object with name and age"}],
    model="claude-sonnet-4-5-20250929",
    output_config={"json_schema": {"name": "person", "schema": {"type": "object", "properties": {"name": {"type": "string"}, "age": {"type": "integer"}}}}},
)
print(message.content)
```

**Token counting:**
```python
from anthropic import Anthropic

client = Anthropic()
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)
```
