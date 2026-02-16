## What's in the box

The `anthropic` package gives you typed sync and async clients for the full Messages API — streaming, tool use, batches, token counting, structured outputs. It runs on httpx with Pydantic models for responses and TypedDict params. Bedrock and Vertex AI each get their own client classes with native auth handling.

```sh
pip install anthropic
```

## What changed recently

**v0.79.0** added the speed parameter for Opus 4.6's fast mode and fixed its passthrough in sync beta `count_tokens`.

**v0.78.0** brought Claude Opus 4.6 model support and adaptive thinking (reasoning traces).

**v0.77.x** introduced structured outputs — you can now pass `output_config` to constrain responses to a JSON schema, which is useful if you're tired of wrestling with freeform model output.

**v0.76.0** added raw JSON schema passthrough for `messages.stream()`, server-side tools in the tool runner, and binary request streaming.

## Code examples

Basic usage:

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

The tool runner is the feature I find most interesting. You decorate a function, hand it to the runner, and the SDK handles the call loop — Claude asks to use the tool, the SDK executes your function, sends the result back, and repeats until Claude has a final answer:

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
    Returns:
        A dictionary with location, temperature, and condition.
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

Async streaming with text accumulation:

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

Token counting before you send:

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

## Other details worth knowing

- Default 2 retries with exponential backoff, 10-minute timeout
- `.with_raw_response` if you need headers
- Auto-pagination on list endpoints
- Optional `aiohttp` backend if you prefer it over httpx for async
- Python 3.9+

Full docs and source: the SDK is generated via Stainless and lives on PyPI as `anthropic`.

## Question

For those of you using tool use in production — are you finding the auto-running tool runner pattern useful, or do you prefer to manage the tool call loop yourself for more control over retry logic and error handling? Curious how people are approaching this.
