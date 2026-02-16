# Andi AIRun — v2.4.1

## Added
- **Script variables** — Declare variables in YAML front-matter with `vars:` block, use `{{placeholder}}` substitution in prompts, override from CLI with `--varname "value"`. Bash 3.2 compatible.
- **`--quiet` / `-q` flag** — Suppresses status messages for CI/CD pipelines. Clean stdout only.
- **`--live` heartbeat** — Shows elapsed time during silent gaps so you know the process is still running.
- **Composable scripts documentation**

## Changed
- `--live` now handles YAML frontmatter output correctly
- Ollama cloud model updates: added `glm-5:cloud` and `minimax-m2.5:cloud`
- Nested `claude -p` fix for child scripts called from parent scripts

## Fixed
- `set -e` safety: `_parse_shebang_flags` now returns explicit `0` on no-op paths, preventing unexpected script exits
- `load_defaults` no longer kills scripts when the defaults file (`~/.ai-runner/defaults.sh`) is missing
- 4 new test assertions added (188 total)

## Usage Examples

**Script variables with front-matter:**
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

**Override variables from CLI:**
```bash
./summarize-topic.md --topic "AI safety" --style formal
./summarize-topic.md --live --length "100 words" --topic "the fall of rome" --style "peter griffin"
```

**Quiet mode for CI/CD:**
```bash
ai --quiet ./live-script.md > output.md
```

**Live streaming with file redirect:**
```bash
./live-report.md > report.md          # Narration to console, clean content to file
```
