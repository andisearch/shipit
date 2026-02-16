# Anthropic Python SDK — Product Briefing

## What the project does

The Anthropic Python SDK (`anthropic`) provides typed, production-grade access to the Anthropic REST API from Python 3.9+. It wraps the Messages API, streaming, tool use, message batches, token counting, and file uploads with both synchronous and asynchronous clients powered by httpx. The SDK also ships dedicated clients for AWS Bedrock and Google Vertex AI deployments of Claude models.

## Key features

- **Sync and async clients** — `Anthropic` and `AsyncAnthropic` with identical interfaces; async supports both httpx and aiohttp backends
- **Streaming responses** — SSE-based streaming via `stream=True` or the higher-level `client.messages.stream()` helper with text accumulation and SDK-specific events
- **Tool use (function calling)** — First-class support for defining tools, including a `@beta_tool` decorator for pure Python functions and an auto-running `tool_runner` that loops tool calls automatically
- **Structured Outputs** — `output_config` parameter for constraining model output to JSON schemas
- **Message Batches API** — Create, poll, and retrieve results from batch message requests under `client.messages.batches`
- **Token counting** — `client.messages.count_tokens()` endpoint for pre-flight token estimation
- **Streaming helpers** — `MessageStream` with `.text_stream`, `.get_final_message()`, `.get_final_text()` and event iteration
- **Typed request/response models** — Pydantic models with `to_json()`, `to_dict()`, autocomplete, and discriminated union support
- **AWS Bedrock client** — `AnthropicBedrock` with automatic AWS region inference and SigV4 auth
- **Google Vertex AI client** — `AnthropicVertex` / `AsyncAnthropicVertex` with credential refresh and global region endpoint support
- **Adaptive thinking** — Support for thinking/reasoning blocks in responses, including clearing thinking in context management
- **File uploads** — `client.beta.files.upload()` accepting bytes, PathLike, or `(filename, contents, media_type)` tuples
- **Retry and timeout handling** — Configurable exponential backoff (default 2 retries), per-request timeout overrides, TCP keepalive
- **Raw response access** — `.with_raw_response` and `.with_streaming_response` for header inspection and lazy body reading
- **Pagination** — Auto-paginating iterators for list endpoints like `client.messages.batches.list()`

## How it works

**Architecture:** The SDK is generated via Stainless and follows a resource-based client pattern. The top-level `Anthropic`/`AsyncAnthropic` classes hold resource namespaces (`client.messages`, `client.messages.batches`, `client.beta`). Each resource maps to REST endpoints with typed params (TypedDicts) and Pydantic response models.

**Execution model:** Sync calls use httpx synchronous transport; async calls use httpx async transport or optionally aiohttp via `DefaultAioHttpClient()`. Streaming uses SSE parsing with event iteration. The `MessageStream` wrapper accumulates content blocks and emits SDK-level events.

**Dependencies (from pyproject.toml):**
- `httpx >=0.25.0, <1`
- `pydantic >=1.9.0, <3` (supports both v1 and v2)
- `typing-extensions >=4.10, <5`
- `anyio >=3.5.0, <5`
- `distro >=1.7.0, <2`
- `jiter >=0.4.0, <1`
- `docstring-parser >=0.15, <1`
- `sniffio`

**Optional extras:** `anthropic[aiohttp]`, `anthropic[bedrock]`, `anthropic[vertex]`

**Current version:** 0.79.0 (as of 2026-02-07)

## Installation & usage

```sh
pip install anthropic
# With optional backends:
pip install anthropic[aiohttp]
pip install anthropic[bedrock]
pip install anthropic[vertex]
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
- Enabled fast-mode in `claude-opus-4-6`
- Fixed: pass `speed` parameter through in sync beta `count_tokens`

**v0.78.0 (2026-02-05):**
- Released Claude Opus 4.6, adaptive thinking, and related features

**v0.77.0–0.77.1 (2026-01-29 to 2026-02-03):**
- Structured Outputs support in Messages API via `output_config`
- Custom JSON encoder for extended type support
- Fix: structured output beta header handling when `output_format` is omitted

**v0.76.0 (2026-01-13):**
- Raw JSON schema support for `messages.stream()`
- Binary request streaming support
- Server-side tools support in tool runner
- Streams always closed properly

**v0.75.0 (2025-11-24):**
- Claude Opus 4.5 support, effort parameter, advanced tool use, autocompaction, Computer Use v5

## Code examples

**Streaming with text_stream helper:**
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

**AWS Bedrock usage:**
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
