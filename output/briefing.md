# Anthropic Python SDK — Product Briefing

## What the project does

The Anthropic Python SDK (`anthropic`) provides typed, convenient access to the Anthropic REST API from Python 3.9+ applications. It covers the full Messages API surface — including streaming, tool use, message batches, token counting, and structured outputs — with both synchronous and asynchronous clients powered by httpx. The SDK also ships dedicated clients for AWS Bedrock and Google Vertex AI deployments.

## Key features

- **Sync and async clients** — `Anthropic` and `AsyncAnthropic` with identical APIs; async supports both httpx and aiohttp backends
- **Streaming responses** — SSE-based streaming via `stream=True` or the higher-level `client.messages.stream()` helper with text accumulation and SDK-specific events
- **Tool use (function calling)** — First-class support for defining tools, including a `@beta_tool` decorator for pure Python functions and an auto-running tool runner (`client.beta.messages.tool_runner()`)
- **Structured outputs** — `output_config` support for constraining model responses to JSON schemas
- **Message Batches API** — Create, poll, and retrieve results from batch message requests under `client.messages.batches`
- **Token counting** — `client.messages.count_tokens()` endpoint for pre-flight token estimation
- **Adaptive thinking** — Support for extended thinking / reasoning traces in Claude Opus 4.6
- **Fast mode** — Speed parameter support for Claude Opus 4.6
- **AWS Bedrock integration** — `AnthropicBedrock` client with automatic AWS auth, region inference, and profile support
- **Google Vertex AI integration** — `AnthropicVertex` / `AsyncAnthropicVertex` with Google credential management and global region endpoint support
- **File uploads** — `client.beta.files.upload()` accepting bytes, PathLike, or (filename, contents, media_type) tuples
- **Typed request/response models** — Full TypedDict params and Pydantic response models with `to_json()` / `to_dict()` helpers
- **Auto-pagination** — Iterator-based pagination for list endpoints
- **Retries and timeouts** — Configurable exponential backoff (default 2 retries), customizable timeouts (default 10 minutes), TCP keepalive
- **Raw response access** — `.with_raw_response` and `.with_streaming_response` for header inspection and lazy body reading

## How it works

**Architecture:** The SDK is generated via Stainless and wraps the Anthropic REST API. It uses httpx as the default HTTP transport, with an optional aiohttp backend for async. Request parameters are typed as `TypedDict`s; responses are Pydantic models. The client handles auth headers (`x-api-key`), versioning (`anthropic-version: 2023-06-01`), retries, and streaming internally.

**Execution model:** Sync calls block; async calls use `asyncio`. Streaming uses Server-Sent Events. The `messages.stream()` helper accumulates content blocks and emits SDK-level events (text deltas, tool use blocks, thinking blocks). The tool runner loops API calls automatically when Claude requests tool invocations.

**Dependencies:** httpx (>=0.25.0), pydantic (>=1.9.0), typing-extensions (>=4.10), anyio (>=3.5.0), distro, sniffio, jiter (>=0.4.0), docstring-parser (>=0.15). Optional extras: `aiohttp` (aiohttp + httpx_aiohttp), `vertex` (google-auth), `bedrock` (boto3 + botocore).

**Current version:** 0.79.0 (released 2026-02-07)

## Installation & usage

```sh
pip install anthropic
# Optional backends:
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

**v0.79.0 (2026-02-07)**
- Enabled fast-mode parameter for Claude Opus 4.6
- Fixed speed parameter passthrough in sync beta `count_tokens`

**v0.78.0 (2026-02-05)**
- Released Claude Opus 4.6 model support
- Added adaptive thinking support
- Additional API features

**v0.77.0–0.77.1 (2026-01-29 to 2026-02-03)**
- Structured Outputs support in Messages API (via `output_config`)
- Custom JSON encoder for extended type support
- Fixed structured output beta header behavior

**v0.76.0 (2026-01-13)**
- Raw JSON schema passthrough for `messages.stream()`
- Binary request streaming support
- Server-side tools support in tool runner
- Stream closure fixes

**v0.75.0 (2025-11-24)**
- Claude Opus 4.5 support
- Effort parameter
- Advanced tool use features
- Autocompaction
- Computer Use v5

## Code examples

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
    message = await stream.get_final_message()
    print(message.to_json())

asyncio.run(main())
```

**Tool use with `@beta_tool` decorator and auto-runner:**
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

**Message Batches:**
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

# After batch completes:
result_stream = await client.messages.batches.results(batch.id)
async for entry in result_stream:
    if entry.result.type == "succeeded":
        print(entry.result.message.content)
```
