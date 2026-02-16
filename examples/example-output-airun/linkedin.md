

I've been mass-deleting rate limit errors from my terminal for months. Every time Claude Code hits a ceiling mid-task, I'd scramble to swap API keys, reset environment variables, and hope my conversation context survived. It didn't.

So I built AIRun — a CLI wrapper that makes Claude Code work across every major cloud provider and lets you switch between them with a single flag.

`ai --aws --opus` starts a session on Bedrock. Hit a rate limit? `ai --pro --resume` picks up the same conversation on your Claude Pro subscription. Need something free while you wait? `ai --ollama --resume` drops you into a local model.

The part I'm most excited about is executable markdown. You write a prompt in a `.md` file, add a shebang line, and now it's a program. Variables in YAML front-matter, placeholder substitution, Unix pipes — the whole deal.

```
#!/usr/bin/env -S ai --haiku
---
vars:
  topic: "machine learning"
  style: casual
---
Write a summary of {{topic}} in a {{style}} tone.
```

Then run it like any script: `./summarize.md --topic "AI safety" --style formal`

You can pipe data through these scripts, chain them together, redirect output to files. Standard Unix patterns, but your prompts are the programs.

Seven providers work today: AWS Bedrock, Google Vertex, Azure, Anthropic API, Vercel AI Gateway (100+ models from OpenAI, xAI, Google, Meta), Ollama, and LM Studio. Provider switching is session-scoped, so your normal `claude` command is never touched.

The whole thing is Bash. No Node, no Python, no build step. Clone the repo, run setup, configure whichever providers you actually use.

Open source under MIT. Would genuinely love feedback from anyone else juggling multiple AI providers or building prompt tooling.

#ClaudeCode #AITooling #OpenSource

**First comment:**
GitHub repo: https://github.com/andisearch/airun
