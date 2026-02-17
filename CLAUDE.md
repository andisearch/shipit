# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

ShipIt generates platform-specific launch content from any git repo. One command produces release notes, a blog post, an X thread, a LinkedIn post, and a Reddit post. With `--open`, it loads drafts into Chrome compose UIs for review.

Built as composable [AIRun](https://github.com/andisearch/airun) executable markdown scripts piped through Claude Opus 4.6.

## Running

```bash
# Generate all channels
./shipit ~/projects/my-app

# Works with GitHub URLs (shallow-clones to temp dir)
./shipit https://github.com/user/repo

# Specific channels only
./shipit ~/projects/my-app --channels x,linkedin

# Focus the analysis on a specific area
./shipit ~/projects/my-app --focus "the new plugin system"

# Changes since a tag
./shipit ~/projects/my-app --since v1.2.0

# Open drafts in Chrome compose UIs
./shipit ~/projects/my-app --open --accounts x,linkedin

# Actually post (implies --open)
./shipit ~/projects/my-app --dangerously-post --accounts x,linkedin
```

Output goes to `./output/`. No build step, no dependencies beyond `ai` (AIRun) and `claude` (Claude Code).

## Architecture

```
shipit [repo-path] [flags]
  → analyze.md        repo context → structured briefing (stdout)
  → channel scripts   briefing + config → platform content (each via stdin/stdout)
      → release-notes.md, blog-post.md, x-thread.md, linkedin-post.md, reddit-post.md
  → review.md         checks each output for AI writing tells, corrects if needed
  → post.md           (optional) Chrome browser automation for --open / --dangerously-post
```

The orchestrator (`shipit`) is a bash script. Each stage is a standalone executable markdown file with an AIRun shebang. They communicate via `=== SECTION: name ===` delimited payloads on stdin.

### Pipeline Data Flow

1. `shipit` gathers repo context (README, CHANGELOG, package manifests, git log)
2. Pipes it to `analyze.md` which produces a structured briefing saved to `output/briefing.md`
3. For each channel: pipes the briefing + channel-config to the channel script
4. Pipes each channel's output through `review.md` (PASS keeps original, FAIL extracts corrected content)
5. Saves final output to `output/<channel>.md`
6. If `--open`, pipes all outputs + account list to `post.md` for Chrome automation

### AIRun Script Conventions

**Shebangs:**
- Generation scripts: `#!/usr/bin/env -S ai --opus --skip --live`
- Review script: `#!/usr/bin/env -S ai --opus --skip --live` (was sonnet, now opus)
- Browser automation: `#!/usr/bin/env -S ai --opus --skip --chrome --live`

**`--live`** sends narration to stderr, final content to stdout. This enables clean piping.

**`=== INPUT ===`** marker at the end of every script signals where stdin content begins.

**YAML frontmatter** in generated output (x-thread, blog, etc.) includes metadata like title, platform, status, tags. The review script preserves frontmatter when correcting content.

### Review Pipeline Details

The orchestrator's review logic (`shipit:165-199`):
- Review output starts with `---` then `PASS` or `FAIL` on the next line
- On PASS: uses original generated content unchanged
- On FAIL: extracts corrected content after the issue list, with a sanity check that corrected content is at least half the original length
- Skip review with `NO_REVIEW=true`

### Chrome Posting (`post.md`)

Requires Chrome with [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension connected. Two modes:
- **draft** (`--open`): fills compose UIs, leaves tabs open for manual review
- **post** (`--dangerously-post`): fills and clicks Post/Submit

Reddit posts are never auto-submitted regardless of mode.

## Adding a New Channel

1. Copy an existing channel script (e.g., `scripts/x-thread.md`)
2. Customize the prompt: output format, platform rules, writing guidelines
3. Add the channel to the `case` statement in `shipit` (~line 140)
4. Add it to `DEFAULT_CHANNELS` if it should run by default

All channel scripts follow the same structure: stdin format docs, output format rules, writing rules, then `=== INPUT ===`.

## Writing Rules (enforced in all generation scripts)

**Forbidden vocabulary:** delve, crucial, pivotal, vibrant, leverage, seamless, landscape, robust, foster, harness, game-changer, cutting-edge, groundbreaking, revolutionary, transformative, elevate, empower, unleash

**Forbidden structures:** bold inline headers (`**Key:** content`), staccato patterns (4+ short declaratives in a row), em dash overuse (>2 per paragraph), generic headings ("Why This Matters"), contrastive negation overuse ("It's not X — it's Y"), rule of three (every list having exactly 3 items)

**Tone:** Conversational, specific, real code examples from the briefing. Write like a developer talking to developers.
