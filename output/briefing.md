

# ShipIt — Product Briefing

## What the Project Does

ShipIt is a bash-based CLI tool that reads a code repository's context (README, CHANGELOG, git log, package manifests) and generates platform-specific launch content across five channels: release notes, blog post, X/Twitter thread, LinkedIn post, and Reddit post. It orchestrates a pipeline of executable markdown scripts, each powered by Claude Opus 4.6 via AIRun, and optionally opens drafts in Chrome for review or direct posting.

## Key Features

- **One-command launch content** — Run `./shipit ~/projects/my-app` to generate content for all five channels from a single repo
- **GitHub URL support** — Accepts GitHub URLs directly; shallow-clones to a temp directory and cleans up after
- **Channel selection** — Generate for specific platforms with `--channels x,linkedin`
- **Focus mode** — Direct the analyst to emphasize a specific feature or angle with `--focus "topic"`
- **Git-aware diffing** — Analyze changes since a tag or date with `--since v1.2.0`
- **AI writing quality review** — A dedicated review stage checks generated content for AI writing tells (vocabulary flags, structural patterns) and corrects them
- **Chrome integration** — `--open` fills compose UIs in separate Chrome tabs for review; `--dangerously-post` posts directly
- **Executable markdown architecture** — Each pipeline stage is a `.md` file with a shebang, composable with Unix pipes
- **Streaming output** — `--live` flag streams generation in real-time

## How It Works

ShipIt is a bash orchestrator (`shipit`) that pipes data between standalone AI scripts. The pipeline has four stages:

1. **Analyze** (`analyze.md`) — Reads repo context (README, manifests, git log) and produces a structured briefing
2. **Generate** (5 channel scripts) — Each receives the briefing via stdin and generates platform-specific content: `release-notes.md`, `blog-post.md`, `x-thread.md`, `linkedin-post.md`, `reddit-post.md`
3. **Review** (`review.md`) — Checks each output for AI writing tells and corrects them
4. **Post** (`post.md`) — Opens drafts in Chrome compose UIs or posts live

Scripts communicate through `=== SECTION: ===` delimited stdin payloads. Each script is an executable markdown file that runs through AIRun with Claude Opus 4.6. Output lands in `./output/` as individual markdown files.

**Dependencies:**
- Claude Code (`claude` CLI)
- AIRun (`ai` CLI) — executable markdown runtime
- Chrome + Claude in Chrome extension (for `--open` / `--dangerously-post`)

## Installation & Usage

```bash
# Prerequisites
curl -fsSL https://claude.ai/install.sh | bash   # Claude Code
git clone https://github.com/andisearch/airun.git && cd airun && ./setup.sh  # AIRun

# Clone ShipIt
git clone https://github.com/andisearch/shipit.git
cd shipit

# Basic usage — generate all channels
./shipit ~/projects/my-app --live

# GitHub URL
./shipit https://github.com/andisearch/airun

# Specific channels only
./shipit ~/projects/my-app --channels x,linkedin

# Focus on a topic
./shipit ~/projects/my-app --focus "the new plugin system"

# Changes since a tag
./shipit ~/projects/my-app --since v1.2.0 --focus "performance improvements"

# Open in Chrome for review
./shipit ~/projects/my-app --live --open --accounts x,linkedin

# Post directly
./shipit ~/projects/my-app --live --dangerously-post --accounts x,linkedin
```

**Output structure:**
```
output/
  briefing.md     # Structured repo analysis
  notes.md        # Release notes
  blog.md         # Blog post
  x.md            # X/Twitter thread
  linkedin.md     # LinkedIn post
  reddit.md       # Reddit post
```

## What's New

Based on recent commits:

- **Initial pipeline release** — Full orchestrator with 7 AI scripts (analyze, 5 channels, review, post) and example outputs
- **Project documentation** — CLAUDE.md with architecture overview and development conventions
- **Expanded README** — Detailed setup instructions, usage examples, architecture explanation, and CLI reference

## Code Examples

**Generate all launch content for a local repo:**
```bash
./shipit ~/projects/my-app --live
```

**Generate only X and LinkedIn posts, focused on a specific feature:**
```bash
./shipit ~/projects/my-app --channels x,linkedin --focus "the new plugin system"
```

**Analyze changes since a release tag with more commit history:**
```bash
./shipit ~/projects/my-app --since v1.2.0 --commits 50 --focus "performance improvements"
```

**Open drafts in Chrome for review before posting:**
```bash
./shipit ~/projects/my-app --live --open --accounts x,linkedin
```
