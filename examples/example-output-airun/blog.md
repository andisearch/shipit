

# AIRun: Make Your Markdown Files Executable AI Prompts

You probably have a folder somewhere full of prompts. Maybe they're in a notes app, maybe scattered across chat histories, maybe copy-pasted into a shared doc that nobody maintains. They work fine until you need to run the same prompt with different inputs, chain two prompts together, or switch from one AI provider to another because you just hit a rate limit.

AIRun fixes this by turning markdown files into executable programs. It's a CLI wrapper around Claude Code that adds Unix shebang support, cross-cloud provider switching, and proper pipe handling — so your prompts become first-class tools in your shell.

## Prompts as Programs

The core idea is simple. Take a markdown file, add a shebang line, make it executable:

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

Now you can run it like any other command:

```bash
./summarize-topic.md --topic "AI safety" --style formal
./summarize-topic.md --live --length "100 words" --topic "the fall of rome" --style "peter griffin"
```

Variables are declared in YAML front-matter with defaults. Override them from the command line without editing the file. The prompt template stays clean, reusable, and version-controllable in git.

Because these are real executables, they compose with standard Unix tools:

```bash
cat data.json | ./analyze.md > results.txt
git log --oneline -20 | ./summarize-changes.md
./generate-report.md | ./format-output.md > final.txt
```

Stdin gets piped in, stdout gets piped out. Stderr carries status messages. Everything works the way Unix programmers expect.

## Why Provider Switching Matters

Anyone who's used AI APIs in production knows the rate limit dance. You're in the middle of a session, the model cuts out, and you're stuck waiting or starting over. AIRun solves this with session-scoped provider switching:

```bash
# Hit rate limit on Pro subscription? Switch to AWS and continue the conversation
ai --aws --resume

# Or switch to local Ollama (free, no rate limits)
ai --ollama --resume

# Use cloud-hosted Ollama models if your machine lacks GPU
ai --ollama --model minimax-m2.5:cloud
```

The `--resume` flag picks up your previous conversation regardless of which provider you're now using. Under the hood, AIRun sets provider-specific environment variables (`ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, etc.) scoped to the session process. Your regular `claude` command is never affected.

Seven providers are supported: AWS Bedrock, Google Vertex, Azure, Anthropic API, Vercel AI Gateway, Ollama, and LM Studio. The Vercel gateway alone gives you access to 100+ models from OpenAI, xAI, Google, Meta, Mistral, and DeepSeek with `--vercel --model provider/model`.

Model tier flags keep things readable across providers:

```bash
ai --aws --opus      # or --high
ai --vertex --sonnet # or --mid
ai --ollama --haiku  # or --low
```

Set a default with `--set-default` so you don't have to type your preferred provider every time. CLI flags always override the default.

## Live Streaming and Automation

The `--live` flag streams output in real-time instead of waiting for the full response. When you're redirecting to a file, AIRun splits the output intelligently — narration and status messages go to stderr (your terminal), clean content goes to the file:

```bash
ai --live --skip task.md                    # Stream to terminal
./live-report.md > report.md                # Narration to console, clean content to file
ai --quiet ./live-script.md > output.md     # Suppress status for CI/CD
```

The `--quiet` flag suppresses all status messages, which is what you want in CI/CD pipelines where only the output matters. Permission shortcuts (`--skip` for dangerously-skip-permissions, `--bypass` for bypassPermissions) let automation scripts run without interactive prompts.

## Under the Hood

AIRun is pure Bash — compatible with macOS's built-in Bash 3.2, no associative arrays, no exotic dependencies beyond Claude Code itself and `jq` for live streaming. The architecture is modular:

- `scripts/ai` handles flag parsing, provider selection, and mode detection (interactive, file, or piped)
- `scripts/lib/` contains shared libraries for front-matter parsing, live streaming, and utilities
- `providers/` has one config file per provider
- `config/models.sh` maps tier flags to model IDs

Installation is a git clone and a setup script:

```bash
git clone https://github.com/andisearch/airun.git
cd airun && ./setup.sh
```

Provider credentials go in `~/.ai-runner/secrets.sh` — only configure what you actually use.

## What's in the Latest Release

v2.4.1 (February 14, 2026) fixed a subtle `set -e` issue where missing files or bare shebangs in `_parse_shebang_flags` and `load_defaults` would silently kill the script. v2.4.0 introduced the entire script variables system — YAML front-matter, `{{placeholder}}` substitution, CLI overrides — all implemented with parallel indexed arrays to stay Bash 3.2 compatible.

Recent releases also added the live heartbeat indicator (shows `Working... Ns` during silent processing gaps), nested `claude -p` support for child scripts calling other scripts, and updated Ollama cloud models.

## Getting Started

If you already have Claude Code installed, you're five minutes away from running your first executable prompt. Clone the repo, run setup, drop a shebang on a markdown file, and `chmod +x` it. The [GitHub repo](https://github.com/andisearch/airun) has example scripts covering everything from simple summaries to multi-step agent workflows with `--team`.

The thing that sold me on this approach: once a prompt is a file in your PATH, it stops being "a prompt" and starts being a tool. You tab-complete it. You pipe data into it. You put it in a Makefile. You commit it alongside the code it operates on. That's a different relationship with AI than pasting text into a chat window.
