---
title: "Anthropic Python SDK v0.79 — Fast Mode for Claude Opus 4.6"
created: 2026-02-16T10:00
platform: linkedin
status: draft
tags:
  - anthropic
  - python
  - sdk
  - claude
  - ai
  - developer-tools
  - open-source
---

The Anthropic Python SDK just shipped fast mode for Claude Opus 4.6, and the developer experience keeps getting better.

We've been iterating on the SDK steadily over the past few months. v0.79 adds a `speed` parameter so you can get faster output from Opus 4.6. Before that, v0.77 brought structured outputs to the Messages API: define a JSON schema, get typed responses back. And v0.76 added server-side tools to the tool runner.

The tool use workflow has evolved substantially. You decorate a Python function with `@beta_tool`, hand it to the tool runner, and the SDK handles the loop: calling the model, executing your function when requested, feeding results back. No manual orchestration.

A few other things worth knowing about: token counting before you send a request, message batches for bulk processing, and first-class support for Bedrock and Vertex if you're running Claude through AWS or GCP. The async story is solid too, with full parity between sync and async clients and aiohttp as an optional backend.

The SDK is generated via Stainless, which means the type coverage stays tight as the API surface grows. Every response comes back as a Pydantic model, and every parameter is typed.

If you're building with Claude in Python, the gap between "call an API" and "build something production-grade" keeps shrinking.

GitHub repo: https://github.com/anthropics/anthropic-sdk-python
PyPI: https://pypi.org/project/anthropic/
Docs: https://docs.anthropic.com/en/api/client-sdks/python
