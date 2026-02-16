

# Andi AIRun — Product Briefing

## What the Project Does

AIRun is a CLI wrapper around Claude Code that makes markdown files executable as AI prompts (via Unix shebang) and adds cross-cloud provider switching. It lets users run the same Claude Code sessions across AWS Bedrock, Google Vertex, Azure, Anthropic API, Vercel AI Gateway, Ollama, and LM Studio — switching mid-conversation to dodge rate limits. It supports Unix pipes, stdin/stdout redirection, script variables, live streaming, and agent teams.

## Key Features

- **Executable markdown** — Run `.md` files as programs with `#!/usr/bin/env ai` shebang; prompts become reusable, versionable tools
- **Cross-cloud provider switching** — Switch between AWS Bedrock, Google Vertex, Azure, Anthropic API, Vercel AI Gateway, and Claude Pro subscription using CLI flags
- **Local model support** — Run free local models via Ollama (GGUF) or LM Studio (MLX on Apple Silicon); includes cloud-hosted Ollama models for low-VRAM machines
- **100+ alternate models** — Access OpenAI, xAI, Google, Meta, Mistral, DeepSeek models through Vercel AI Gateway with `--vercel --model provider/model`
- **Unix pipe support** — Pipe data into scripts, redirect output to files, chain scripts in pipelines with proper stdout/stderr separation
- **Script variables** — YAML front-matter with `vars:` block and `{{placeholder}}` substitution; CLI overrides without editing the script
- **Live streaming** — `--live` flag streams text output in real-time; smart output splitting sends narration to stderr and clean content to file when redirecting
- **Session continuity** — `--resume` picks up previous conversations across any provider/model combination
- **Agent teams** — `--team` enables Claude Code's multi-agent collaboration (experimental, interactive only)
- **Model tier selection** — `--opus`/`--high`, `--sonnet`/`--mid`, `--haiku`/`--low` for model tier control
- **Persistent defaults** — `--set-default` saves preferred provider+model; CLI flags always override
- **Permission shortcuts** — `--skip` (dangerously-skip-permissions) and `--bypass` (bypassPermissions) for automation scripts
- **Session-scoped isolation** — Provider environment variables are set per-session; plain `claude` is never affected

## How It Works

**Architecture:** A collection of Bash scripts organized as:
- `scripts/ai` — Main entry point, handles flag parsing, provider selection, mode detection (interactive / file / piped)
- `scripts/lib/` — Shared libraries: `core-utils.sh`, `system-utils.sh`, `live-stream.sh`, front-matter parsing
- `providers/` — Modular provider configs (`aws.sh`, `vertex.sh`, `ollama.sh`, `lmstudio.sh`, `vercel.sh`, `azure.sh`, `apikey.sh`)
- `config/models.sh` — Default model IDs per provider and tier
- `tools/` — Tool abstraction layer

**Execution model:** AIRun detects three modes:
1. **Interactive** — `ai --aws --opus` launches Claude Code with provider env vars set
2. **File** — `ai task.md` or `./task.md` reads the markdown, strips the shebang, extracts front-matter variables, substitutes placeholders, and passes the prompt to `claude -p`
3. **Piped** — `cat data.json | ai` or `curl ... | ai` reads stdin content, prepends/appends to prompt

Provider switching works by setting environment variables (`ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_MODEL`, `ANTHROPIC_SMALL_FAST_MODEL`, etc.) scoped to the session process. Original config is saved and restored on exit via trap.

**Dependencies:** Claude Code (required), Bash 3.2+ (macOS compatible, no associative arrays), `jq` (for `--live` streaming). Optional: Ollama, LM Studio, cloud provider credentials.

**Config locations:**
- `~/.ai-runner/secrets.sh` — API keys and credentials
- `~/.ai-runner/defaults.sh` — Saved default provider/model
- `/usr/local/share/ai-runner/` — Installed scripts and libraries
- `/usr/local/bin/` — Symlinked commands (`ai`, `airun`, `ai-status`, `ai-sessions`)

## Installation & Usage

**Install:**
```bash
git clone https://github.com/andisearch/airun.git
cd airun && ./setup.sh
```

**Configure providers** in `~/.ai-runner/secrets.sh` (only what you need):
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export AWS_PROFILE="your-profile-name"
export AWS_REGION="us-west-2"
export VERCEL_AI_GATEWAY_TOKEN="vck_..."
```

**Update:**
```bash
ai update
```

**Key flags:**

| Flag | Purpose |
|------|---------|
| `--aws`, `--vertex`, `--apikey`, `--azure`, `--vercel`, `--pro` | Provider selection |
| `--ollama` / `--ol`, `--lmstudio` / `--lm` | Local providers |
| `--opus` / `--high`, `--sonnet` / `--mid`, `--haiku` / `--low` | Model tier |
| `--model <id>` | Specific model ID |
| `--resume` | Continue previous conversation |
| `--live` | Stream output in real-time |
| `--quiet` / `-q` | Suppress status messages (CI/CD) |
| `--skip` | Shortcut for `--dangerously-skip-permissions` |
| `--bypass` | Shortcut for `--permission-mode bypassPermissions` |
| `--team` | Enable agent teams |
| `--set-default` | Save current flags as default |
| `--clear-default` | Remove saved default |
| `--stdin-position prepend\|append` | Control where piped input appears |

## What's New

**v2.4.1 (2026-02-14):** Fixed `set -e` safety — `_parse_shebang_flags` and `load_defaults` no longer silently kill the script when missing files or bare shebangs trigger non-zero exit codes.

**v2.4.0 (2026-02-14):** Script variables via YAML front-matter. Declare `vars:` with defaults, use `{{placeholder}}` in prompts, override from CLI with `--varname "value"`. Bash 3.2 compatible (parallel indexed arrays).

**v2.3.6 (2026-02-14):** Live heartbeat shows `[AI Runner] Working... Ns` during silent gaps. `--quiet` flag for CI/CD. Composable scripts documentation. Nested `claude -p` fix for child scripts. Updated Ollama cloud models (glm-5, minimax-m2.5).

**v2.3.3–2.3.5 (2026-02-12–13):** `--live` flag for real-time streaming with smart output splitting (narration to stderr, content to file). Fixed shebang flag parsing and flag precedence (CLI > shebang > defaults). YAML frontmatter support in live output splitting.

**Recent commits:** Variable override documentation, example script flag cleanup, set -e safety fixes, Ollama cloud model updates, process isolation for nested scripts, quiet mode.

## Code Examples

**1. Executable markdown with variables:**
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
./summarize-topic.md --topic "AI safety" --style formal
./summarize-topic.md --live --length "100 words" --topic "the fall of rome" --style "peter griffin"
```

**2. Provider switching to bypass rate limits:**
```bash
# Hit rate limit on Pro subscription, switch to AWS and resume
ai --aws --resume

# Or switch to local Ollama (free)
ai --ollama --resume

# Use Ollama cloud models (no GPU required)
ai --ollama --model minimax-m2.5:cloud
```

**3. Unix pipe automation:**
```bash
cat data.json | ./analyze.md > results.txt
git log --oneline -20 | ./summarize-changes.md
./generate-report.md | ./format-output.md > final.txt
```

**4. Live streaming with file redirect:**
```bash
ai --live --skip task.md                    # Stream to terminal
./live-report.md > report.md                # Narration to console, clean content to file
ai --quiet ./live-script.md > output.md     # Suppress status for CI/CD
ai --chrome --live --skip test-flow.md      # Browser automation with progress
```
