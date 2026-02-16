# ShipIt

One command turns your repo into a full product launch.

## DEADLINE: 3 PM EST TODAY. ~3 hours. Move fast, ship working code. Simple beats polished.

## What This Is

ShipIt is a hackathon project for the "Built with Opus 4.6" Claude Code Hackathon (Cerebral Valley + Anthropic).

`shipit ./my-repo` generates: release notes, blog post, X thread, LinkedIn post, Reddit post — all tailored per-platform. With `--chrome`, it opens your browser and posts the content to your logged-in social accounts live.

Built as composable AIRun executable markdown scripts piped through Opus 4.6.

**Problem Statement:** #1 — "Build a Tool That Should Exist"

**Repo:** github.com/andisearch/shipit (private until submission)

**You have bypassPermissions. Don't ask, just build.**

## Architecture

```
shipit [repo-path] [--since tag/date] [--channels x,linkedin,reddit,blog,notes] [--live] [--chrome --accounts x,linkedin]
  → analyze.md    (reads repo: README, CHANGELOG, git log, code → structured briefing)
  → fan out to channel scripts (each is a standalone executable .md):
      → release-notes.md
      → blog-post.md
      → x-thread.md
      → linkedin-post.md
      → reddit-post.md
  → review.md     (quality check on each output)
  → save to output/
  → [optional] post.md  (browser automation to post content live via --chrome)
```

Each script is a standalone executable markdown file with shebang. The orchestrator (`shipit`) is a bash script that pipes data between them.

## Repo Structure

```
shipit/
  CLAUDE.md                # This file
  README.md                # Hero README with architecture, quick start
  shipit                   # Main orchestrator (bash, executable)
  scripts/
    analyze.md             # Stage 1: repo analysis → structured briefing
    release-notes.md       # Channel: release notes
    blog-post.md           # Channel: blog post
    x-thread.md            # Channel: X/Twitter thread
    linkedin-post.md       # Channel: LinkedIn post
    reddit-post.md         # Channel: Reddit post
    review.md              # Quality review stage
    post.md                # Browser automation posting via --chrome
  examples/
    output/                # Example generated outputs (committed)
  LICENSE                  # MIT
```

## Build Phases (time-boxed — stop and move on if a phase runs over)

### Phase 1: Core Pipeline (1 hour max)
1. Create repo structure: README stub, LICENSE (MIT), scripts/ dir, examples/output/
2. Write `scripts/analyze.md` — repo analysis (adapt from ~/projects/andi-promote/scripts/research/analyze-repo.md)
3. Write `scripts/release-notes.md` and `scripts/x-thread.md` — first two channels
4. Write `shipit` orchestrator (bash) — gather repo context, pipe to analyze, fan out to channels, save to output/
5. Test: `./shipit ~/projects/airun` — must produce output files

### Phase 2: Remaining Channels + Review (45 min)
6. Write `scripts/blog-post.md`, `scripts/linkedin-post.md`, `scripts/reddit-post.md`
7. Write `scripts/review.md` — writing quality check
8. Wire review into pipeline, test full run

### Phase 2.5: Chrome Posting (30 min)
9. Write `scripts/post.md` — browser automation (adapt from ~/projects/yc-application-advisor/test/scripts/check-upload-flow.md)
10. Add `--chrome` and `--accounts` flags to orchestrator
11. Test: `./shipit ~/projects/airun --chrome --accounts x`

### Phase 3: Ship (45 min)
12. Full README with architecture and quick start
13. Commit example outputs
14. Record 3-min demo video
15. Push to public GitHub, submit at https://cerebralvalley.ai/e/claude-code-hackathon/hackathon/submit

## Implementation Details

### The `shipit` Orchestrator (bash)

Simplified version of andi-promote's `stages/generate` + `campaign-utils.sh`. Key functions:

- Parse CLI args: repo path, --since, --channels, --live, --chrome, --accounts
- Gather repo context: README, CHANGELOG, package.json/Cargo.toml/etc, `git log --oneline -20`, `git diff` if --since
- Pipe context to `scripts/analyze.md` → save briefing
- For each channel: pipe briefing to channel script → pipe to review → save output
- If --chrome: pipe outputs to `scripts/post.md`

**Payload format** (same `=== SECTION:` delimiter pattern as andi-promote):
```
=== SECTION: briefing ===
(output from analyze.md)

=== SECTION: channel-config ===
platform: x
project_name: AIRun
project_github: https://github.com/andisearch/airun
```

### AIRun Script Patterns

**Shebang for generation scripts:**
```markdown
#!/usr/bin/env -S ai --opus --skip --live
```

**Shebang for review (cheaper model is fine):**
```markdown
#!/usr/bin/env -S ai --sonnet --skip --live
```

**Shebang for browser automation (needs --chrome for MCP tools):**
```markdown
#!/usr/bin/env -S ai --opus --skip --chrome --live
```

**Variable support (YAML frontmatter):**
```markdown
#!/usr/bin/env -S ai --opus --skip --live
---
vars:
  platform: x
  style: casual
---
Generate a {{platform}} post in {{style}} tone.
```

**stdin reading:** Scripts automatically receive piped content. End prompt with `=== INPUT ===` to signal where stdin content begins.

**stdout/stderr separation:** With `--live`, narration goes to stderr, final content goes to stdout. This enables clean piping: `./analyze.md < context | ./x-thread.md > output/x-thread.md`

### scripts/analyze.md

Reads repo contents from stdin, produces structured briefing covering:
1. What the project does (1 paragraph)
2. Key features (bulleted)
3. How it works (technical overview)
4. Installation & usage
5. What's new (recent changes/commits)
6. Code examples (real, from the repo)

Reference: `~/projects/andi-promote/scripts/research/analyze-repo.md`

### scripts/x-thread.md

Key elements:
- 6-10 tweets, each under 280 chars
- Tweet 1: Hook (conceptual, no links)
- Tweets 2-4: Demonstrate with examples
- Tweet 5-6: Broader implications
- Tweet 7+: CTA
- Reply tweet: Links (posted separately)
- No thread opener emoji — just start with the hook
- Links in reply ONLY (links in thread body kill reach)

Reference: `~/projects/andi-promote/scripts/generate/generate-thread.md`

### scripts/blog-post.md

Key elements:
- Full blog post with smooth flowing prose
- Structure: what → why → how → what's next
- Real code examples from the briefing
- No staccato social media copy style
- Subheadings only where they help navigation

Reference: `~/projects/andi-promote/scripts/generate/generate-blog.md`

### scripts/linkedin-post.md

Hook in first 2 lines (210 chars), NO links in body, professional but personal tone, links go in a `**First comment:**` section.

Reference: `~/projects/andi-promote/scripts/generate/generate-post.md` (LinkedIn section)

### scripts/reddit-post.md

Strong title, TL;DR section, body with headers, discussion question at end, no self-promotion tone.

Reference: `~/projects/andi-promote/scripts/generate/generate-post.md` (Reddit section)

### scripts/release-notes.md

Clean markdown release notes: version header, grouped changes (Added/Changed/Fixed), brief descriptions, code examples where relevant.

### scripts/review.md

Two checks:
1. Vocabulary tells (forbidden AI slop words: delve, crucial, vibrant, leverage, seamless, etc.)
2. Structural tells (bold inline headers, staccato sentences, em dash overuse, contrastive negation, etc.)

Output: PASS/FAIL verdict with specific issues.

Reference: `~/projects/andi-promote/scripts/review/check-writing.md`

### scripts/post.md (Chrome browser posting)

Uses `--chrome` flag for MCP browser tools. Flow:
1. Reads generated content + channel info from stdin
2. Calls `tabs_context_mcp` to check Chrome connectivity (fail fast if no browser)
3. Calls `update_plan` to declare target domains (e.g., `["x.com", "linkedin.com"]`) and approach
4. For each specified account:
   - Opens a new tab via `tabs_create_mcp`
   - Navigates to the platform
   - Checks if user is logged in (detect profile elements, compose buttons)
   - If logged in: composes and posts content using `find`, `form_input`, `computer` tools
   - If not logged in: skips with a warning
5. Reports what was posted where via `--live` text streaming

Safety: Only posts to platforms explicitly in `--accounts` AND with detected logged-in session.

Prerequisites for --chrome:
1. Chrome open and in the foreground (not minimized)
2. Claude in Chrome extension installed, active, showing "Connected"
3. `ai` CLI installed (AIRun)

Reference: `~/projects/yc-application-advisor/test/scripts/check-upload-flow.md` and `~/projects/yc-application-advisor/test/README.md`

## Key Reference Files

Read these before writing scripts — they contain the patterns to adapt:

| File | What to learn |
|------|--------------|
| `~/projects/andi-promote/stages/generate` | Bash orchestration: how to parse channel args, build SECTION payloads, pipe to generation scripts |
| `~/projects/andi-promote/stages/review` | Review pipeline: how to pipe post through quality check and merge findings |
| `~/projects/andi-promote/scripts/generate/generate-post.md` | Single post generation prompt: stdin format, output format, writing rules, platform-specific rules |
| `~/projects/andi-promote/scripts/generate/generate-thread.md` | Thread generation prompt: tweet structure, hook patterns, link placement |
| `~/projects/andi-promote/scripts/generate/generate-blog.md` | Blog generation prompt: SEO frontmatter, post types, writing rules |
| `~/projects/andi-promote/scripts/review/check-writing.md` | Writing review prompt: vocabulary/structural tells, PASS/FAIL verdict format |
| `~/projects/andi-promote/scripts/research/analyze-repo.md` | Repo analysis prompt: what to extract, output format |
| `~/projects/andi-promote/lib/campaign-utils.sh` | `assemble_payload` and `assemble_inline` functions for building SECTION-delimited stdin payloads |
| `~/projects/andi-promote/lib/stage-utils.sh` | `stream_section`, `stream_delim`, `frontmatter_field` — stream parsing helpers |
| `~/projects/airun/docs/SCRIPTING.md` | AIRun scripting guide: shebangs, variables, --live, --chrome, permission modes, composable chaining |
| `~/projects/yc-application-advisor/test/scripts/check-upload-flow.md` | Chrome browser automation pattern: tabs_context_mcp, update_plan, find, form_input, progress logging with --live |
| `~/projects/yc-application-advisor/test/README.md` | Chrome automation prerequisites and troubleshooting |

## Critical Design Principles

### Keep It Simple (vs. andi-promote)
- **No YAML config layers** (config.yaml, promote.yaml, overlay.yaml) — hardcode sensible defaults
- **No research stages** (subreddit checks, channel activity, fact gathering) — just analyze the repo
- **No campaign planning stage** — go straight from analysis to generation
- **No rewrite/optimize stages** — review is enough to show depth
- **No frontmatter templates or engagement protocols** — embed channel rules directly in each script
- **No session directories or work dirs** — output goes to `./output/`
- **No config-loader.sh or channel_field()** — simple bash arrays or case statements

### What Makes This Win the Hackathon
1. **Demo is king (30%)** — `--live` streaming + `--chrome` live posting = genuinely cool to watch
2. **Opus 4.6 plays multiple roles** — analyst, writer (5 voices), editor in separate pipeline stages
3. **Executable markdown is surprising** — scripts ARE prompts, prompts ARE programs
4. **Composable Unix pipes** — each script runs independently, pipes connect them
5. **Real utility** — every dev team needs this, it actually saves hours

## Writing Rules (for generated content)

All generation scripts should produce content that avoids AI slop:
- No: delve, crucial, pivotal, vibrant, leverage, seamless, landscape, robust, foster, harness
- No: game-changer, cutting-edge, groundbreaking, revolutionary, transformative
- No: bold inline headers (`**Key point:** content`), staccato patterns (4+ short declaratives), em dash overuse
- No: generic headings ("Why This Matters"), contrastive negation overuse ("It's not X — it's Y")
- Be conversational, specific, use real code examples
