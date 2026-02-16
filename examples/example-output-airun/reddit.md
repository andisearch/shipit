

**Title:** AIRun — Make markdown files executable as AI prompts, switch cloud providers mid-conversation

**TL;DR:** I built a CLI wrapper around Claude Code that lets you run `.md` files as programs (via Unix shebang), switch between AWS Bedrock, Google Vertex, Azure, Anthropic API, Vercel AI Gateway, Ollama, and LM Studio mid-session, and pipe data through AI scripts like any other Unix tool.

## What it does

AIRun makes markdown files into executable programs. You add a shebang line, write your prompt, and run it directly from the terminal:

```markdown
#!/usr/bin/env -S ai --haiku
---
vars:
  topic: "machine learning"
  style: casual
  length: short
---
Write a {{length}} summary of {{topic}} in a {{style}} tone.
```

```bash
chmod +x summarize.md
./summarize.md --topic "AI safety" --style formal
```

Variables are declared in YAML front-matter with defaults, then overridden from the CLI without editing the file. The prompt template uses `{{placeholder}}` substitution — nothing fancy, just enough to make scripts reusable.

## Provider switching

The main reason I built this: rate limits. When you hit a wall on one provider, you switch to another and keep going.

```bash
# Hit rate limit on Pro subscription, switch to AWS
ai --aws --resume

# Or drop to a free local model
ai --ollama --resume

# Use 100+ models through Vercel AI Gateway
ai --vercel --model openai/gpt-4o
```

Provider env vars are scoped to the session process. Your normal `claude` config is never touched — original settings are saved and restored on exit via trap.

Supported providers: AWS Bedrock, Google Vertex, Azure, Anthropic API, Vercel AI Gateway, Ollama (local GGUF or cloud-hosted), LM Studio (MLX on Apple Silicon).

## Unix pipes work the way you'd expect

```bash
cat data.json | ./analyze.md > results.txt
git log --oneline -20 | ./summarize-changes.md
./generate-report.md | ./format-output.md > final.txt
```

Stdout and stderr are separated properly, so you can redirect output to a file while still seeing status messages in your terminal. The `--live` flag streams text in real-time and splits narration to stderr when you're redirecting:

```bash
./live-report.md > report.md    # Progress in terminal, clean content in file
ai --quiet ./script.md > out.md # Suppress status messages for CI/CD
```

## How it works

It's all Bash — no compiled dependencies beyond Claude Code itself (and `jq` for live streaming). The architecture is:

- `scripts/ai` — Entry point. Parses flags, detects mode (interactive / file / piped), selects provider.
- `scripts/lib/` — Shared libraries for front-matter parsing, streaming, utilities.
- `providers/` — Modular configs for each cloud provider. Adding a new one is just a new `.sh` file.
- `config/models.sh` — Default model IDs per provider and tier (`--opus`/`--sonnet`/`--haiku`).

Three execution modes: interactive (launches Claude Code with provider env vars), file (reads markdown, strips shebang, substitutes variables, passes to `claude -p`), and piped (reads stdin, prepends or appends to prompt).

Bash 3.2 compatible — works on stock macOS without homebrew Bash.

## Install

```bash
git clone https://github.com/andisearch/airun.git
cd airun && ./setup.sh
```

Configure providers in `~/.ai-runner/secrets.sh` (only set what you use):

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export AWS_PROFILE="your-profile-name"
export AWS_REGION="us-west-2"
```

## What I'm unsure about

I went back and forth on whether variable substitution should use `{{mustache}}` syntax or `$shell_style` syntax. Mustache felt safer since there's no risk of accidental shell expansion, but it's one more syntax to remember. For those of you who've built similar prompt tooling — what syntax felt most natural to your users?
