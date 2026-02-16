# ShipIt

One command turns your repo into a full product launch.

```bash
./shipit ~/projects/my-app
```

ShipIt reads your repo (README, CHANGELOG, git log, package manifests), builds a structured briefing, then generates platform-specific content for release notes, a blog post, an X/Twitter thread, a LinkedIn post, and a Reddit post. With `--chrome`, it opens your browser and posts the content to your logged-in social accounts live.

Built for the [Claude Code Hackathon](https://cerebralvalley.ai/e/claude-code-hackathon) (Cerebral Valley + Anthropic, Feb 2026). Problem statement: "Build a Tool That Should Exist."

## Demo

```bash
# Generate all channels for a repo
./shipit ~/projects/my-app

# Only specific channels
./shipit ~/projects/my-app --channels x,linkedin

# Include changes since a tag
./shipit ~/projects/my-app --since v1.2.0

# Generate and post to X and LinkedIn via Chrome
./shipit ~/projects/my-app --chrome --accounts x,linkedin
```

Output goes to `./output/`:

```
output/
  briefing.md     # Structured repo analysis
  notes.md        # Release notes
  blog.md         # Blog post
  x.md            # X/Twitter thread (6-10 tweets)
  linkedin.md     # LinkedIn post with first-comment link
  reddit.md       # Reddit post with TL;DR
```

## How It Works

ShipIt is a bash orchestrator that pipes data between standalone AI scripts. Each script is an executable markdown file that runs through [AIRun](https://github.com/andisearch/airun) and Claude Opus 4.6.

```
shipit [repo-path] [flags]
  │
  ├── analyze.md        reads repo context → structured briefing
  │
  ├── release-notes.md  ─┐
  ├── blog-post.md       │ each receives the briefing
  ├── x-thread.md        ├ + channel config via stdin,
  ├── linkedin-post.md   │ generates platform-specific content
  ├── reddit-post.md    ─┘
  │
  ├── review.md         checks each output for AI writing tells
  │
  └── post.md           (optional) posts via Chrome browser automation
```

The scripts communicate through `=== SECTION: ===` delimited stdin payloads — the same pattern Unix pipes use, but for structured AI context.

### Why Executable Markdown?

Each `.md` script has a shebang that makes it a real program:

```markdown
#!/usr/bin/env -S ai --opus --skip --live
You are generating an X/Twitter thread for a product launch.
...
=== INPUT ===
```

This means:
- Scripts run directly: `./scripts/x-thread.md`
- Scripts compose with pipes: `cat briefing.md | ./scripts/x-thread.md > output/x.md`
- The prompt IS the program — no wrapper code, no templates, no config files

## Architecture

Opus 4.6 plays a different role at each pipeline stage:

1. **Analyst** — `analyze.md` reads raw repo files and produces a structured briefing
2. **Writer** (5 voices) — Each channel script generates content tailored to its platform's format, audience, and algorithm
3. **Editor** — `review.md` checks for AI writing tells (vocabulary flags, structural patterns) and corrects them
4. **Publisher** — `post.md` uses Chrome browser automation to post content to logged-in social accounts

All scripts enforce the same writing quality rules: no "delve," no "game-changer," no bold inline headers, no staccato sentence patterns, no generic headings. The review stage catches anything the generation stage missed.

## Flags

| Flag | Description |
|------|-------------|
| `--channels x,linkedin,...` | Which channels to generate (default: all) |
| `--since tag-or-date` | Include changes since a git ref |
| `--chrome` | Enable browser posting |
| `--accounts x,linkedin` | Which platforms to post to (requires `--chrome`) |
| `--live` | Stream output in real-time |

Available channels: `notes`, `blog`, `x`, `linkedin`, `reddit`

## Prerequisites

- [AIRun](https://github.com/andisearch/airun) installed (`ai` command available)
- Claude Code with Opus 4.6 access
- For `--chrome`: Chrome browser with [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension

## Setup

```bash
git clone https://github.com/andisearch/shipit.git
cd shipit
chmod +x shipit scripts/*.md
./shipit ~/path/to/your/repo
```

## Example Output

See `examples/output/` for generated content from running ShipIt against the [AIRun](https://github.com/andisearch/airun) repo.

## Built With

- [Claude Opus 4.6](https://anthropic.com) — powers every pipeline stage
- [AIRun](https://github.com/andisearch/airun) — executable markdown runtime
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) — browser automation for `--chrome` posting
- Bash — orchestrator, because Unix pipes are the right abstraction

## License

MIT
