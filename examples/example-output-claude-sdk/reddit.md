## What's in the box

The `anthropic` package gives you typed access to the full Anthropic REST API from Python 3.9+. Sync and async clients, SSE streaming, tool use, message batches, token counting, file uploads — all with Pydantic models and autocomplete.

Recent releases have added a few things worth calling out:

- **Claude Opus 4.6** with fast-mode enabled
- **Structured Outputs** — constrain model output to a JSON schema via `output_config`
- **`@beta_tool` decorator** — define tools as plain Python functions, let the SDK handle the schema generation and tool-call loop
- **aiohttp backend** — drop-in alternative to httpx for async workloads
- **Adaptive thinking** — support for thinking/reasoning blocks in responses

## Quick start

```sh
pip install anthropic
export ANTHROPIC_API_KEY="my-anthropic-api-key"
```

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

## Streaming

The `stream()` helper accumulates content blocks and gives you a `.text_stream` iterator:

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

## Tool use with @beta_tool

Define a tool as a function. The decorator pulls the schema from the docstring and type hints. The `tool_runner` handles the back-and-forth loop automatically:

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

## Bedrock and Vertex

Both work with minimal config changes:

```python
from anthropic import AnthropicBedrock

client = AnthropicBedrock()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
    model="anthropic.claude-sonnet-4-5-20250929-v1:0",
)
```

Vertex follows the same pattern with `AnthropicVertex`.

## Install

```sh
pip install anthropic
# Optional backends:
pip install anthropic[aiohttp]
pip install anthropic[bedrock]
pip install anthropic[vertex]
```

Repo: check the project repository for full docs and examples.

---

For those of you already using the SDK in production — how are you handling the tool-use loop in practice? Are you using the new `tool_runner` or rolling your own loop? Curious what patterns people have settled on.
