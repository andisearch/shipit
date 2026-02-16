# The Anthropic Python SDK: A Typed, Production-Grade Interface to Claude

The Anthropic Python SDK is the official way to talk to Claude from Python. It covers the full surface of the Anthropic REST API — messages, streaming, tool use, batch processing, token counting, file uploads — with typed inputs and outputs, sync and async clients, and dedicated support for AWS Bedrock and Google Vertex AI deployments. Version 0.79.0 shipped on February 7, 2026.

## What you get

Install it with pip:

```sh
pip install anthropic
```

Set your API key as an environment variable:

```sh
export ANTHROPIC_API_KEY="my-anthropic-api-key"
```

And you're making API calls in four lines:

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

The SDK gives you `Anthropic` for synchronous code and `AsyncAnthropic` for async, with identical interfaces. Every request parameter is typed (via TypedDicts), every response is a Pydantic model with `.to_json()`, `.to_dict()`, and full autocomplete support. If you pass the wrong type, your editor catches it before your code runs.

## Streaming that doesn't fight you

Server-sent events are the transport layer, but you don't need to think about that. The `MessageStream` wrapper accumulates content blocks, emits SDK-level events, and gives you a `.text_stream` iterator that yields text as it arrives:

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

When the stream finishes, `.get_final_message()` hands you the complete message object, same as a non-streaming call would return. No manual reassembly.

## Tool use with Python functions

The `@beta_tool` decorator turns a regular Python function into a tool Claude can call. You write the function, add a docstring describing what it does, and the SDK handles schema generation, argument parsing, and result formatting:

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

The `tool_runner` handles the full loop: it sends the initial request, executes any tool calls Claude makes, sends the results back, and keeps going until Claude produces a final response. You iterate over it and get each message in the conversation.

## Architecture under the hood

The SDK is generated via Stainless and follows a resource-based client pattern. The top-level client holds resource namespaces — `client.messages`, `client.messages.batches`, `client.beta` — each mapping to REST endpoints. Sync calls go through httpx's synchronous transport; async calls use httpx async or, optionally, aiohttp if you install `anthropic[aiohttp]`:

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

Retries use configurable exponential backoff (two retries by default). Timeouts can be set per-request. For inspecting response headers or lazily reading bodies, `.with_raw_response` and `.with_streaming_response` expose the underlying HTTP details without changing the call pattern.

## Running Claude on Bedrock and Vertex AI

If you're deploying Claude through AWS Bedrock or Google Vertex AI, the SDK has dedicated clients that handle authentication natively:

```python
from anthropic import AnthropicBedrock

client = AnthropicBedrock()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
    model="anthropic.claude-sonnet-4-5-20250929-v1:0",
)
print(message)
```

`AnthropicBedrock` handles SigV4 signing and region inference automatically. `AnthropicVertex` does the same for Google Cloud credentials and regional endpoints. The API surface stays the same — swap the client, keep your code.

## What shipped recently

The 0.77 through 0.79 releases added several things worth knowing about. Structured Outputs landed in 0.77, letting you constrain model output to a JSON schema via the `output_config` parameter. Version 0.78 introduced Claude Opus 4.6 and adaptive thinking support, which gives you access to the model's reasoning process through thinking blocks in responses. And 0.79 enabled fast-mode for Opus 4.6 and fixed parameter passthrough in the sync beta `count_tokens` method.

Earlier, 0.76 brought raw JSON schema support in `messages.stream()`, binary request streaming, and server-side tools in the tool runner. Version 0.75 shipped Claude Opus 4.5 support alongside effort parameters, advanced tool use, autocompaction, and Computer Use v5.

## Getting started

The SDK supports Python 3.9+ and depends on httpx, pydantic (v1 or v2), anyio, and a few other packages. Optional extras pull in aiohttp, boto3 for Bedrock, or google-auth for Vertex AI:

```sh
pip install anthropic[aiohttp]
pip install anthropic[bedrock]
pip install anthropic[vertex]
```

The source lives on GitHub at [anthropics/anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python), and the [API reference](https://docs.anthropic.com/en/docs) covers every endpoint the SDK wraps. If you've been using an older version, the typed models and resource namespaces mean upgrading mostly involves letting your type checker tell you what changed.
