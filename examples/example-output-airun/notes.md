

# AIRun — v2.4.1

## Added
- **Script variables via YAML front-matter** — Declare `vars:` with defaults, use `{{placeholder}}` in prompts, override from CLI with `--varname "value"`. Bash 3.2 compatible (parallel indexed arrays).
- **Live heartbeat indicator** — Shows `[AI Runner] Working... Ns` during silent gaps so you know the model is still processing.
- **`--quiet` / `-q` flag** — Suppresses status messages for CI/CD pipelines and automation.
- **Ollama cloud models** — Added `glm-5` and `minimax-m2.5` for machines without enough VRAM to run models locally.

## Changed
- Nested `claude -p` calls now run in isolated processes, fixing child script execution within pipelines.
- Updated composable scripts documentation with pipe chaining patterns.

## Fixed
- **`set -e` safety** — `_parse_shebang_flags` and `load_defaults` no longer silently kill the script when missing files or bare shebangs trigger non-zero exit codes.
- Shebang flag parsing now respects correct precedence: CLI flags > shebang flags > saved defaults.
- `--live` streaming properly splits narration to stderr and clean content to stdout when redirecting to a file.
- YAML frontmatter is now stripped from live output splitting.

## Usage Examples

**Executable markdown with variables:**
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
./summarize-topic.md --live --length "100 words" --topic "the fall of rome"
```

**Live streaming with output redirect:**
```bash
ai --live --skip task.md                    # Stream to terminal
./live-report.md > report.md                # Narration to console, clean content to file
ai --quiet ./live-script.md > output.md     # Suppress status for CI/CD
```

**Resume across providers when rate-limited:**
```bash
ai --aws --resume
ai --ollama --model minimax-m2.5:cloud --resume
```
