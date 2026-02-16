## 1

Most Python API clients make you choose: typed responses or streaming. Sync or async. Simple calls or tool use.

The Anthropic Python SDK does all of it — and just shipped Claude Opus 4.6 support with adaptive thinking and fast mode.

## 2

Basic usage is exactly what you'd expect:

```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-opus-4-6",
)
```

Typed params in, Pydantic models out.

## 3

Streaming that actually accumulates state for you:

```python
async with client.messages.stream(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Say hello"}],
    model="claude-sonnet-4-5-20250929",
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

SSE under the hood, clean iterator on top.

## 4

Tool use with a Python decorator — define a function, the SDK handles the rest:

```python
@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a city"""
    return json.dumps({"location": location, "temperature": "68°F"})

runner = client.beta.messages.tool_runner(
    tools=[get_weather],
    messages=[{"role": "user", "content": "Weather in SF?"}],
)
```

The tool runner loops automatically when Claude requests invocations.

## 5

Token counting before you send:

```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-5-20250929",
    messages=[{"role": "user", "content": "Hello, world"}],
)
print(count.input_tokens)  # 10
```

Know your costs before they happen.

## 6

Need to run thousands of requests? Message Batches let you fire off bulk jobs and stream results back:

```python
batch = await client.messages.batches.create(
    requests=[
        {"custom_id": "req-1", "params": {...}},
        {"custom_id": "req-2", "params": {...}},
    ]
)
```

No polling loops to write yourself.

## 7

What's new in v0.79.0:

- Claude Opus 4.6 with adaptive thinking
- Fast mode parameter for speed-optimized responses
- Structured outputs via output_config for JSON schema constraints
- Bedrock and Vertex AI clients with native auth

All typed. All with retry and timeout handling built in.

## 8

`pip install anthropic` — that's it.

Supports Python 3.9+, sync and async, with optional extras for aiohttp, Bedrock, and Vertex AI.

Two lines to your first API call.

## Reply (Link)

GitHub: https://github.com/anthropics/anthropic-sdk-python

Docs and API reference: https://docs.anthropic.com
