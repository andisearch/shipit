## 1

Most Python API clients make you choose: typed responses or streaming. Good DX or async support.

The Anthropic Python SDK doesn't make you choose. Here's what ships in v0.79.0.

## 2

Basic usage is what you'd expect — typed all the way through:

```python
from anthropic import Anthropic

client = Anthropic()
message = client.messages.create(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    model="claude-sonnet-4-5-20250929",
)
```

Pydantic models with autocomplete, .to_json(), discriminated unions. No guessing at response shapes.

## 3

Streaming that actually feels good to use:

```python
async with client.messages.stream(
    max_tokens=1024,
    messages=[{"role": "user", "content": "Say hello there!"}],
    model="claude-sonnet-4-5-20250929",
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

SSE parsing, text accumulation, and event iteration handled for you.

## 4

Tool use with a Python decorator — define a function, hand it to the runner, done:

```python
@beta_tool
def get_weather(location: str) -> str:
    """Lookup the weather for a city"""
    return json.dumps({"location": location, "temperature": "68°F"})

runner = client.beta.messages.tool_runner(
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    messages=[...],
)
```

The tool runner loops tool calls automatically. No manual parsing.

## 5

Same interface works sync and async. Same interface works with httpx or aiohttp. Same interface works against the Anthropic API, AWS Bedrock, or Google Vertex AI.

One SDK, pick your transport and deployment target.

## 6

Structured Outputs landed in v0.77 — constrain model output to a JSON schema via `output_config`. Token counting is built in with `client.messages.count_tokens()` for pre-flight estimation.

Small things that save you from building your own wrappers.

## 7

Message Batches API for when you need to fire off hundreds of requests and collect results later. Auto-paginating iterators so you don't hand-roll pagination. Configurable retries with exponential backoff.

Production details that matter when you're past the prototype stage.

## 8

The SDK supports Python 3.9+ and works with both Pydantic v1 and v2.

Install and go:

```
pip install anthropic
export ANTHROPIC_API_KEY="my-key"
```

Optional extras for your setup:
`pip install anthropic[bedrock]`
`pip install anthropic[vertex]`
`pip install anthropic[aiohttp]`

## Reply (Link)

GitHub: github.com/anthropics/anthropic-sdk-python

Docs: docs.anthropic.com
