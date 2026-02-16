# ShipIt

> One command turns your repo into a full product launch.

<!-- TODO: Add demo GIF here -->

```bash
./shipit ~/projects/my-app --live
```

ShipIt reads your repo (README, CHANGELOG, git log, package manifests), builds a structured briefing, then generates platform-specific content for release notes, a blog post, an X/Twitter thread, a LinkedIn post, and a Reddit post. With `--chrome`, it opens your browser and posts the content to your logged-in social accounts live.

Built for the [Cerebral Valley + Anthropic "Built with Opus 4.6" hackathon](https://cerebralvalley.ai/e/claude-code-hackathon). Problem statement: "Build a Tool That Should Exist."

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed (`claude` command available)
- [AIRun](https://github.com/andisearch/airun) installed (`ai` command available)
- For `--chrome`: Chrome with [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension

## Quick Start

```bash
# Install Claude Code (if you don't have it)
npm install -g @anthropic-ai/claude-code

# Install AIRun (executable markdown runner)
git clone https://github.com/andisearch/airun.git
cd airun && ./setup.sh

# Clone ShipIt
git clone https://github.com/andisearch/shipit.git
cd shipit

# Generate launch content for any repo
./shipit ~/projects/my-app --live
```

## Usage

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

<!-- TODO: Add example output snippets for 1-2 channels -->

## What It Generates

| Channel | What You Get |
|---------|-------------|
| Release Notes | Grouped changes (Added / Changed / Fixed), version header |
| Blog Post | Long-form technical post with code examples |
| X Thread | 6-10 tweets: hook, demos, CTA. Links in reply only |
| LinkedIn Post | Professional narrative, links in first comment |
| Reddit Post | TL;DR + technical body + discussion question |

## How It Works

ShipIt is a bash orchestrator that pipes data between standalone AI scripts. Each script is an executable markdown file that runs through [AIRun](https://github.com/andisearch/airun) and Claude Opus 4.6.

```
shipit [repo-path] [flags]
  |
  |-- analyze.md        reads repo context -> structured briefing
  |
  |-- release-notes.md  -+
  |-- blog-post.md       | each receives the briefing
  |-- x-thread.md        + + channel config via stdin,
  |-- linkedin-post.md   | generates platform-specific content
  |-- reddit-post.md    -+
  |
  |-- review.md         checks each output for AI writing tells
  |
  +-- post.md           (optional) posts via Chrome browser automation
```

The scripts communicate through `=== SECTION: ===` delimited stdin payloads -- the same pattern Unix pipes use, but for structured AI context.

## Architecture

Opus 4.6 plays a different role at each pipeline stage:

1. **Analyst** -- `analyze.md` reads raw repo files and produces a structured briefing
2. **Writer** (5 voices) -- Each channel script generates content matched to its platform's format, audience, and algorithm
3. **Editor** -- `review.md` checks for AI writing tells (vocabulary flags, structural patterns) and corrects them
4. **Publisher** -- `post.md` uses Chrome browser automation to post content to logged-in social accounts

All scripts enforce writing quality rules: no "delve," no "game-changer," no bold inline headers, no staccato sentence patterns. The review stage catches anything the generation stage missed.

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
- The prompt IS the program -- no wrapper code, no templates, no config files

## Chrome Posting

```bash
./shipit ~/projects/my-app --live --chrome --accounts x,linkedin
```

With `--chrome`, Opus opens your browser, navigates to each platform, and posts the generated content to your logged-in accounts. No API keys, no OAuth tokens -- it uses your existing browser sessions.

Prerequisites:
- Chrome open with [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) extension installed
- Logged into your social accounts

<!-- TODO: Add demo GIF of Chrome posting flow -->

## Adding a Channel

Each channel is a markdown file. Add a platform by copying an existing script and customizing the prompt:

```bash
cp scripts/x-thread.md scripts/mastodon-post.md
# Edit the voice and format rules for Mastodon
# Add 'mastodon' to the case statement in the shipit orchestrator
```

## CLI Reference

| Flag | Description |
|------|-------------|
| `--channels x,linkedin,...` | Which channels to generate (default: all) |
| `--since tag-or-date` | Include changes since a git ref |
| `--chrome` | Enable browser posting |
| `--accounts x,linkedin` | Which platforms to post to (requires `--chrome`) |
| `--live` | Stream output in real-time |

Available channels: `notes`, `blog`, `x`, `linkedin`, `reddit`

## Example Output

See `examples/output/` for generated content from running ShipIt against the [AIRun](https://github.com/andisearch/airun) repo.

## Built With

- [Claude Opus 4.6](https://anthropic.com) -- powers every pipeline stage
- [AIRun](https://github.com/andisearch/airun) -- executable markdown runtime
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) -- browser automation for `--chrome` posting
- Bash -- orchestrator, because Unix pipes are the right abstraction

## License

MIT
