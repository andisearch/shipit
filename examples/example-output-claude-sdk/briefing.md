# Anthropic Python SDK — Product Briefing

## What the project does

The Anthropic Python SDK (`anthropic`) provides typed access to the Anthropic REST API from Python 3.9+. It wraps the Messages API (and legacy Completions API) with synchronous and asynchronous clients, streaming support, tool use, message batches, token counting, structured outputs, and integrations for AWS Bedrock and Google Vertex AI. The SDK is built on httpx, uses Pydantic models for request/response typing, and supports both httpx and aiohttp as async HTTP backends.

## Key features

- **Sync and async clients** — `Anthropic` and `AsyncAnthropic` with identical APIs; async also supports aiohttp backend via `DefaultAioHttpClient()`
- **Streaming** — Server-sent events streaming via `stream=True` or the higher-level `client.messages.stream()` helper with text accumulation and SDK-specific events
- **Tool use (function calling)** — Define tools as Python functions with `@beta_tool` decorator; automatic tool execution loop via `client.beta.messages.tool_runner()`
- **Structured outputs** — JSON schema-based output formatting via `output_config` parameter
- **Token counting** — `client.messages.count_tokens()` endpoint to get input token counts without creating a message
- **Message batches** — Batch API under `client.messages.batches` for bulk asynchronous processing
- **AWS Bedrock integration** — `AnthropicBedrock` client with AWS auth (profile, region, keys, session token)
- **Google Vertex AI integration** — `AnthropicVertex` / `AsyncAnthropicVertex` clients with Google credentials auth
- **Automatic retries** — Exponential backoff on connection errors, 408, 409, 429, and 5xx; configurable via `max_retries`
- **Adaptive thinking** — Support for thinking/reasoning content blocks with context management
- **Fast mode** — `speed` parameter support for Claude Opus 4.6
- **File uploads** — `client.beta.files.upload()` accepting bytes, PathLike, or (filename, contents, media_type) tuples
- **Citations** — Citation content blocks with source references in responses
- **Pagination** — Auto-paginating iterators for list endpoints

## How it works

**Architecture:** The SDK is generated via Stainless and structured around resource classes (`client.messages`, `client.messages.batches`, `client.beta.messages`). Each resource maps to API endpoints and returns Pydantic models. The client handles auth headers (`x-api-key`), retry logic, timeout management, and SSE stream parsing.

**Execution model:** Synchronous calls block on httpx; async calls use httpx with asyncio or optionally aiohttp. Streaming uses SSE event iteration. The tool runner loops API calls automatically when the model requests tool use.

**Dependencies:** httpx (HTTP), pydantic (models), anyio (async), typing-extensions (type support), distro (platform info), sniffio (async library detection), jiter (JSON parsing), docstring-parser (tool docstring extraction). Optional: aiohttp, google-auth, boto3/botocore.

**Package structure:** Source under `src/anthropic/`. Types in `src/anthropic/types/`. Resources in `src/anthropic/resources/`. Bedrock/Vertex clients are separate classes with their own auth flows.

## Installation & usage

```sh
pip install anthropic
pip install anthropic[bedrock]    # AWS Bedrock support
pip install anthropic[vertex]     # Google Vertex support
pip install anthropic[aiohttp]    # aiohttp async backend
```

Set `ANTHROPIC_API_KEY` environment variable or pass `api_key` to the client constructor.

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

**v0.79.0 (2026-02-07)**
- Fast mode support for Claude Opus 4.6 — `speed` parameter enabling fast-mode output
- Bug fix: `speed` parameter now correctly passes through in sync beta `count_tokens`

**v0.78.0 (2026-02-05)**
- Release of Claude Opus 4.6 model support
- Adaptive thinking feature
- New `BetaUsage` fields added

**v0.77.0–0.77.1 (2026-01-29 to 2026-02-03)**
- Structured Outputs in the Messages API (via `output_config`)
- Custom JSON encoder for extended type support
- Structured output beta header fix

**v0.76.0 (2026-01-13)**
- Server-side tools support in tool runner
- Raw JSON schema support in `messages.stream()`
- Binary request streaming

**v0.75.0 (2025-11-24)**
- Claude Opus 4.5, effort parameter, advanced tool use, autocompaction, Computer Use v5

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

**Async with streaming helper:**
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
    message = await stream.get_final_message()
    print(message.to_json())

asyncio.run(main())
```

**Tool use with @beta_tool decorator:**
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
