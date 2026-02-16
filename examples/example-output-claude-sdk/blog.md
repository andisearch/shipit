# The Anthropic Python SDK at 0.79: Structured Outputs, Tool Runners, and Fast Mode

The Anthropic Python SDK is how most Python developers talk to Claude. It wraps the full Messages API with typed parameters, Pydantic response models, and both sync and async clients — so you get autocomplete, type checking, and a clean interface without hand-rolling HTTP requests. Version 0.79.0 shipped on February 7th with fast mode for Claude Opus 4.6, capping a string of releases that added structured outputs, adaptive thinking, and a tool runner that handles multi-turn tool calls automatically.

## What you get

The SDK covers the entire Anthropic REST API surface. You create messages, stream responses, define tools for function calling, count tokens before sending requests, and run batch jobs — all through a single `Anthropic()` client (or `AsyncAnthropic()` if you prefer async). There are also dedicated clients for AWS Bedrock and Google Vertex AI if you're running Claude through those platforms.

Installation is one line:

```sh
pip install anthropic
```

And basic usage looks like this:

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

The client reads your API key from `ANTHROPIC_API_KEY` by default, handles retries with exponential backoff, and gives you a 10-minute timeout — all configurable.

## Streaming that accumulates

Raw SSE streaming is available via `stream=True`, but the higher-level `messages.stream()` helper is where things get comfortable. It accumulates content blocks as they arrive and emits SDK-specific events for text deltas, tool use blocks, and thinking blocks. You can iterate over the text stream directly:

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

When the stream finishes, `get_final_message()` hands you the fully assembled message object — same shape as what you'd get from a non-streaming call.

## Tool use without the boilerplate

Tool use (function calling) has first-class support. The `@beta_tool` decorator lets you define tools as plain Python functions with docstrings, and the tool runner handles the back-and-forth loop where Claude requests a tool call, you execute it, and send results back:

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

The runner iterates through API calls automatically. When Claude asks for `get_weather`, the runner calls your function, feeds the result back, and continues until Claude produces a final response. No manual loop management.

## Structured outputs and token counting

Version 0.77 introduced structured outputs through `output_config`, which constrains Claude's responses to match a JSON schema. This is useful when you need machine-readable output rather than free-form text.

Token counting lets you estimate costs before sending a request:

```python
from anthropic import Anthropic

client = Anthropic()
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

And message batches handle bulk workloads at reduced cost:

```python
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

batch = await client.messages.batches.create(
    requests=[
        {
            "custom_id": "my-first-request",
            "params": {
                "model": "claude-sonnet-4-5-20250929",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Hello, world"}],
            },
        },
        {
            "custom_id": "my-second-request",
            "params": {
                "model": "claude-sonnet-4-5-20250929",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Hi again, friend"}],
            },
        },
    ]
)
```

## The recent releases

The last few versions have moved quickly. Here's the progression:

**0.79.0** added fast mode for Claude Opus 4.6, letting you trade some output quality for speed via a parameter. It also fixed speed parameter passthrough in the sync beta `count_tokens` endpoint.

**0.78.0** brought Claude Opus 4.6 model support and adaptive thinking — reasoning traces that show Claude's chain of thought.

**0.77.0** landed structured outputs in the Messages API, along with a custom JSON encoder for extended type support.

**0.76.0** added raw JSON schema passthrough for `messages.stream()`, binary request streaming, and server-side tools support in the tool runner.

The SDK itself is generated via Stainless from the API spec, which means new API features tend to show up in the SDK within days of going live. Under the hood, it uses httpx for HTTP transport (with an optional aiohttp backend for async), anyio for async primitives, and Pydantic for response models.

## Getting started

```sh
pip install anthropic
export ANTHROPIC_API_KEY="my-anthropic-api-key"
```

Optional extras if you need them: `anthropic[aiohttp]` for the aiohttp async backend, `anthropic[bedrock]` for AWS Bedrock, `anthropic[vertex]` for Google Vertex AI.

The full SDK source and documentation live on [GitHub](https://github.com/anthropics/anthropic-sdk-python). The typed interface means your editor will show you what's available as you type — which, honestly, is the best documentation for day-to-day use.
