# Anthropic Python SDK — Product Briefing

## What the project does

The Anthropic Python SDK (`anthropic`) provides typed access to the Anthropic REST API from Python 3.9+. It wraps the Messages API (and legacy Completions API) with synchronous and asynchronous clients, streaming support, tool use, structured outputs, message batches, token counting, and integrations for AWS Bedrock and Google Vertex AI. The library is built on `httpx` with Pydantic models for request/response typing.

## Key features

- **Sync and async clients** — `Anthropic` and `AsyncAnthropic` with identical interfaces; async also supports `aiohttp` backend
- **Streaming** — SSE-based streaming via `stream=True` or the `messages.stream()` helper with text accumulation and SDK-specific events
- **Tool use (function calling)** — Define tools as Python functions with `@beta_tool` decorator; automatic invocation via `tool_runner`
- **Structured outputs** — JSON schema-based output formatting via `output_config`
- **Message batches** — Batch API under `client.messages.batches` for bulk async processing
- **Token counting** — `client.messages.count_tokens()` endpoint without creating a message
- **AWS Bedrock integration** — `AnthropicBedrock` client with AWS auth (profile, region, keys)
- **Google Vertex AI integration** — `AnthropicVertex` / `AsyncAnthropicVertex` with Google credentials
- **Adaptive thinking** — Support for thinking/reasoning blocks with context management
- **Citations** — Citation support in responses with search result content blocks
- **File uploads** — `client.beta.files.upload()` with `bytes`, `PathLike`, or tuple input
- **Prompt caching** — Cache control for reducing repeated token costs
- **Auto-pagination** — Iterators that automatically fetch subsequent pages
- **Retries and timeouts** — Configurable exponential backoff (default 2 retries), 10-minute default timeout
- **Fast mode** — Speed parameter for Claude Opus 4.6

## How it works

**Architecture:** The SDK is a thin typed wrapper around Anthropic's REST API. It uses `httpx` for HTTP transport (with optional `aiohttp` for async), `pydantic` for request/response models, and `jiter` for fast JSON parsing. Resources are organized as nested client attributes (`client.messages`, `client.messages.batches`, `client.beta.messages`).

**Execution model:** Requests go through a base client that handles auth headers (`x-api-key`, `anthropic-version: 2023-06-01`), retries with exponential backoff, timeout enforcement, and response parsing into typed Pydantic models. Streaming uses SSE with an event iterator pattern. The Bedrock and Vertex clients override auth and endpoint resolution but share the same resource interface.

**Key dependencies:**
- `httpx` >=0.25.0 — HTTP client
- `pydantic` >=1.9.0 — Data validation/models (supports v1 and v2)
- `anyio` >=3.5.0 — Async compatibility
- `typing-extensions` >=4.10 — Type hints backports
- `jiter` >=0.4.0 — JSON parsing
- `docstring-parser` >=0.15 — Tool function docstring extraction
- Optional: `boto3`/`botocore` (Bedrock), `google-auth` (Vertex), `aiohttp` (async HTTP)

## Installation & usage

```sh
pip install anthropic
pip install anthropic[bedrock]   # AWS Bedrock support
pip install anthropic[vertex]    # Google Vertex support
pip install anthropic[aiohttp]   # aiohttp async backend
```

Set your API key:
```sh
export ANTHROPIC_API_KEY="my-anthropic-api-key"
```

Basic message creation:
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

## What's new

**v0.79.0 (2026-02-07):**
- Fast mode enabled for `claude-opus-4-6` — new `speed` parameter support
- Bug fix: `speed` parameter now passes through correctly in sync beta `count_tokens`

**v0.78.0 (2026-02-05):**
- Claude Opus 4.6 model release
- Adaptive thinking support
- New API features

**v0.77.0–0.77.1 (2026-01-29 to 2026-02-03):**
- Structured Outputs in the Messages API (via `output_config`)
- Custom JSON encoder for extended type support
- Bug fix for structured output beta header when format is omitted

**v0.76.0 (2026-01-13):**
- Server-side tools support in tool runner
- Raw JSON schema support in `messages.stream()`
- Binary request streaming

**v0.75.0 (2025-11-24):**
- Claude Opus 4.5 support
- Effort parameter
- Advanced tool use features
- Autocompaction
- Computer Use v5

## Code examples

**Streaming responses:**
```python
from anthropic import Anthropic

client = Anthropic()
stream = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-sonnet-4-5-20250929",
    stream=True,
)
for event in stream:
    print(event.type)
```

**Tool use with `@beta_tool` decorator:**
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

**Async streaming with helpers:**
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

**Token counting:**
```python
from anthropic import Anthropic

client = Anthropic()
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```
