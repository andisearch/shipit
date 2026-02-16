# Anthropic Python SDK — v0.79.0

## Added

- **Claude Opus 4.6 support** — New model `claude-opus-4-6` now available, including fast-mode output
- **Adaptive thinking** — Support for thinking/reasoning blocks in responses, with context management for clearing thinking history
- **Structured Outputs** — Constrain model output to JSON schemas via the `output_config` parameter in the Messages API
- **Raw JSON schema support** for `messages.stream()`
- **Binary request streaming** support
- **Server-side tools** support in the tool runner

## Changed

- Custom JSON encoder for extended type support
- Streams are now always closed properly

## Fixed

- `speed` parameter now correctly passed through in sync beta `count_tokens`
- Structured output beta header handling when `output_format` is omitted

## Usage Examples

**Streaming with the text_stream helper:**

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

**Tool use with @beta_tool decorator and auto-runner:**

```python
import json
import rich
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
    rich.print(message)
```

**Async with aiohttp backend:**

```python
import os
import asyncio
from anthropic import DefaultAioHttpClient, AsyncAnthropic

async def main() -> None:
    async with AsyncAnthropic(
        api_key=os.environ.get("ANTHROPIC_API_KEY"),
        http_client=DefaultAioHttpClient(),
    ) as client:
        message = await client.messages.create(
            max_tokens=1024,
            messages=[{"role": "user", "content": "Hello, Claude"}],
            model="claude-sonnet-4-5-20250929",
        )
        print(message.content)

asyncio.run(main())
```
