# ShipIt

One command turns your repo into a full product launch.

## DEADLINE: 3 PM EST TODAY. ~3 hours. Move fast, ship working code. Simple beats polished.

## What This Is

ShipIt is a hackathon project for the "Built with Opus 4.6" Claude Code Hackathon (Cerebral Valley + Anthropic).

`shipit ./my-repo` generates: release notes, blog post, X thread, LinkedIn post, Reddit post — all tailored per-platform. With `--open`, it loads drafts into your browser's compose UIs for review. With `--dangerously-post`, it posts live.

Built as composable AIRun executable markdown scripts piped through Opus 4.6.

**Problem Statement:** #1 — "Build a Tool That Should Exist"

**Repo:** github.com/andisearch/shipit (private until submission)

**You have bypassPermissions. Don't ask, just build.**

## Architecture

```
shipit [repo-path] [--since tag/date] [--channels x,linkedin,reddit,blog,notes] [--live] [--open --accounts x,linkedin] [--dangerously-post]
  → analyze.md    (reads repo: README, CHANGELOG, git log, code → structured briefing)
  → fan out to channel scripts (each is a standalone executable .md):
      → release-notes.md
      → blog-post.md
      → x-thread.md
      → linkedin-post.md
      → reddit-post.md
  → review.md     (quality check on each output)
  → save to output/
  → [optional] post.md  (browser automation: --open for drafts, --dangerously-post to post live)
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
    post.md                # Browser automation: --open for drafts, --dangerously-post to post
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
10. Add `--open`, `--dangerously-post`, and `--accounts` flags to orchestrator
11. Test: `./shipit ~/projects/airun --open --accounts x`

### Phase 3: Ship
12. Full README with architecture and quick start
13. Commit example outputs
14. Push to public GitHub

## Implementation Details

### The `shipit` Orchestrator (bash) — Concrete Skeleton

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPTS="$SCRIPT_DIR/scripts"
OUTPUT_DIR="./output"
DEFAULT_CHANNELS="notes,blog,x,linkedin,reddit"

# --- Helpers ---

assemble_payload() {
    # Usage: assemble_payload SECTION_NAME FILE_PATH
    while [[ $# -ge 2 ]]; do
        local section="$1" file="$2"; shift 2
        if [[ -f "$file" ]]; then
            echo "=== SECTION: $section ==="
            echo ""
            cat "$file"
            echo ""
        fi
    done
}

assemble_inline() {
    # Usage: assemble_inline SECTION_NAME "content string"
    local section="$1" content="$2"
    echo "=== SECTION: $section ==="
    echo ""
    echo "$content"
    echo ""
}

# --- CLI Arg Parsing ---

REPO_PATH=""
SINCE=""
CHANNELS=""
LIVE=false
CHROME=false
ACCOUNTS=""

while [[ $# -gt 0 ]]; do
    case "$1" in
        --since)    SINCE="$2"; shift 2 ;;
        --channels) CHANNELS="$2"; shift 2 ;;
        --live)     LIVE=true; shift ;;
        --open)     OPEN=true; shift ;;
        --dangerously-post) DANGEROUSLY_POST=true; OPEN=true; shift ;;
        --accounts) ACCOUNTS="$2"; shift 2 ;;
        -*)         echo "Unknown flag: $1" >&2; exit 1 ;;
        *)          REPO_PATH="$1"; shift ;;
    esac
done

REPO_PATH="${REPO_PATH:-.}"
REPO_PATH="$(cd "$REPO_PATH" && pwd)"
CHANNELS="${CHANNELS:-$DEFAULT_CHANNELS}"

mkdir -p "$OUTPUT_DIR"

# --- Gather Repo Context ---

gather_repo_context() {
    echo "Repository: $REPO_PATH"
    echo ""
    for f in README.md README.rst CHANGELOG.md CHANGES.md; do
        [[ -f "$REPO_PATH/$f" ]] && { echo "--- File: $f ---"; cat "$REPO_PATH/$f"; echo ""; }
    done
    for f in package.json Cargo.toml pyproject.toml setup.py go.mod; do
        [[ -f "$REPO_PATH/$f" ]] && { echo "--- File: $f ---"; cat "$REPO_PATH/$f"; echo ""; }
    done
    if [[ -d "$REPO_PATH/.git" ]]; then
        echo "--- Recent commits ---"
        git -C "$REPO_PATH" log --oneline -20
        echo ""
        if [[ -n "$SINCE" ]]; then
            echo "--- Changes since $SINCE ---"
            git -C "$REPO_PATH" log --oneline "$SINCE"..HEAD
            echo ""
        fi
    fi
}

# --- Stage 1: Analyze ---

echo "==> Analyzing repo..." >&2
BRIEFING=$(gather_repo_context | "$SCRIPTS/analyze.md")
echo "$BRIEFING" > "$OUTPUT_DIR/briefing.md"
echo "==> Briefing saved to $OUTPUT_DIR/briefing.md" >&2

# Detect project metadata from repo
PROJECT_NAME=$(basename "$REPO_PATH")
# Try to extract from package.json, Cargo.toml, etc. — fallback to dir name
if [[ -f "$REPO_PATH/package.json" ]]; then
    PROJECT_NAME=$(grep -o '"name": *"[^"]*"' "$REPO_PATH/package.json" | head -1 | sed 's/"name": *"//;s/"//' || echo "$PROJECT_NAME")
fi

# --- Stage 2: Generate per channel ---

IFS=',' read -ra CHAN_LIST <<< "$CHANNELS"
for channel in "${CHAN_LIST[@]}"; do
    channel=$(echo "$channel" | tr -d ' ')
    # Map channel names to script files
    case "$channel" in
        x)        script="$SCRIPTS/x-thread.md" ;;
        linkedin) script="$SCRIPTS/linkedin-post.md" ;;
        reddit)   script="$SCRIPTS/reddit-post.md" ;;
        blog)     script="$SCRIPTS/blog-post.md" ;;
        notes)    script="$SCRIPTS/release-notes.md" ;;
        *)        echo "Unknown channel: $channel, skipping" >&2; continue ;;
    esac

    if [[ ! -f "$script" ]]; then
        echo "Script not found: $script, skipping" >&2
        continue
    fi

    echo "==> Generating $channel..." >&2

    CONTENT=$({
        assemble_payload "briefing" "$OUTPUT_DIR/briefing.md"
        assemble_inline "channel-config" "platform: $channel
project_name: $PROJECT_NAME
project_repo: $REPO_PATH"
    } | "$script")

    # Optional: pipe through review
    if [[ -f "$SCRIPTS/review.md" ]]; then
        CONTENT=$(echo "$CONTENT" | "$SCRIPTS/review.md")
    fi

    echo "$CONTENT" > "$OUTPUT_DIR/$channel.md"
    echo "==> Saved $OUTPUT_DIR/$channel.md" >&2
done

# --- Stage 3: Chrome posting (optional) ---

if [[ "$CHROME" == true && -n "$ACCOUNTS" ]]; then
    echo "==> Posting via Chrome..." >&2
    {
        for channel in "${CHAN_LIST[@]}"; do
            channel=$(echo "$channel" | tr -d ' ')
            outfile="$OUTPUT_DIR/$channel.md"
            [[ -f "$outfile" ]] && assemble_payload "$channel" "$outfile"
        done
        assemble_inline "accounts" "$ACCOUNTS"
    } | "$SCRIPTS/post.md"
fi

echo "==> Done. Output in $OUTPUT_DIR/" >&2
```

**Notes on this skeleton:**
- `assemble_payload` and `assemble_inline` are the only two helpers needed for building `=== SECTION: ===` payloads
- Each channel script receives a briefing + channel-config via stdin
- The review step is optional — if `review.md` exists, content is piped through it
- Chrome posting collects all generated outputs and pipes them with account info to `post.md`

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

### Concrete Generation Script Template

Every channel script follows this exact structure. Copy and customize:

```markdown
#!/usr/bin/env -S ai --opus --skip --live

You are generating a [CHANNEL_TYPE] for a product launch.

## Stdin Format

Your input arrives as delimited sections:

\```
=== SECTION: briefing ===
(Structured product briefing: what it does, features, how it works, what's new, code examples)

=== SECTION: channel-config ===
platform: [x|linkedin|reddit|blog|notes]
project_name: ...
project_repo: ...
\```

Sections may be absent — generate with what you have.

## Output Format

Output ONLY the final content. No preamble, no explanation, no markdown code fences wrapping the output.

[CHANNEL-SPECIFIC FORMAT RULES GO HERE]

## Writing Rules

- No AI slop: delve, crucial, pivotal, vibrant, leverage, seamless, landscape, robust, foster, harness
- No buzzwords: game-changer, cutting-edge, groundbreaking, revolutionary, transformative
- No bold inline headers (**Key:** content), no staccato patterns, no em dash overuse
- No generic headings ("Why This Matters"), no contrastive negation overuse ("It's not X — it's Y")
- Be conversational, specific, use real code examples from the briefing
- Write like a developer talking to developers

=== INPUT ===
```

**The `=== INPUT ===` marker at the end is required** — it tells AIRun where stdin content begins in the prompt.

### Concrete Review Script Template

```markdown
#!/usr/bin/env -S ai --sonnet --skip --live

You are a writing quality reviewer. Check the content piped to you for AI writing tells.

## Check 1: Vocabulary Tells
Flag any of these words/phrases: delve, crucial, pivotal, vibrant, leverage, seamless, landscape,
robust, foster, harness, game-changer, cutting-edge, groundbreaking, revolutionary, transformative,
elevate, empower, unleash, dive into, at the end of the day, in today's world

## Check 2: Structural Tells
Flag: bold inline headers (**Key:** content), staccato sentences (4+ short declaratives in a row),
excessive em dashes, generic section headings ("Why This Matters", "Key Takeaways"),
contrastive negation overuse ("It's not X — it's Y"), rule of three (always exactly 3 items)

## Output Format

If no issues found:
PASS

If issues found:
FAIL
- [issue 1: quote the offending text and explain]
- [issue 2: ...]

Then output the corrected content with issues fixed.

=== INPUT ===
```

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

### scripts/post.md (Chrome browser posting) — Concrete Pattern

```markdown
#!/usr/bin/env -S ai --opus --skip --chrome --live

You are a social media posting agent. You receive generated content via stdin and post it to the user's logged-in social accounts using Chrome browser automation.

## Stdin Format

\```
=== SECTION: x ===
(Generated X/Twitter thread content)

=== SECTION: linkedin ===
(Generated LinkedIn post content)

=== SECTION: reddit ===
(Generated Reddit post content)

=== SECTION: accounts ===
x,linkedin
\```

## Step 0: Chrome Connectivity Check (FAIL FAST)

Call `tabs_context_mcp` immediately. If it fails or returns no response, output this and STOP:
```
ABORT: Cannot connect to Chrome. Ensure:
  1. Chrome is open and in the foreground
  2. Claude in Chrome extension is installed and active
  3. Extension shows 'Connected' status
```

## Step 1: Declare Plan

Call `update_plan` with:
- domains: ["x.com", "linkedin.com", "reddit.com"] (only the ones in accounts)
- approach: ["Post generated content to each platform", "Check login status before posting", "Report results"]

## Step 2: Post to Each Platform

For each platform in the accounts list:

### X/Twitter Posting Flow:
1. `tabs_create_mcp` → get new tab ID
2. `navigate` to "https://x.com/compose/post"
3. `wait` 2 seconds for page load
4. `find` query "post text area" or "What is happening" → get the compose box
5. `computer` action "left_click" on the compose area
6. `computer` action "type" with the first tweet text
7. `find` query "Post button" → click it
8. For thread tweets: wait 2s, click "Add another post", type next tweet, post
9. Screenshot to confirm

### LinkedIn Posting Flow:
1. `tabs_create_mcp` → get new tab ID
2. `navigate` to "https://www.linkedin.com/feed/"
3. `wait` 2 seconds
4. `find` query "Start a post" button → click it
5. `wait` 1 second for compose modal
6. `find` query "text editor" or "What do you want to talk about" → click it
7. `computer` action "type" with the post content
8. `find` query "Post button" → click it
9. Screenshot to confirm

### Reddit Posting Flow:
1. `tabs_create_mcp` → get new tab ID
2. `navigate` to "https://www.reddit.com/submit"
3. Fill in title and body using `find` and `form_input`
4. Select target subreddit
5. Screenshot before posting (let user verify)

## Login Detection

Before posting on any platform, check if logged in:
- `read_page` with filter "interactive" — look for compose buttons, profile avatars
- If no profile/compose elements found → output "SKIPPED [platform]: not logged in" and continue

## Output

Report results as plain text:
```
Posted to X: [link or confirmation]
Posted to LinkedIn: [link or confirmation]
SKIPPED Reddit: not logged in
```

=== INPUT ===
```

**Prerequisites for --open / --dangerously-post:**
1. Chrome open and in the foreground (not minimized)
2. Claude in Chrome extension installed, active, showing "Connected"
3. `ai` CLI installed (AIRun)

Reference (READ ONLY IF STUCK): `~/projects/yc-application-advisor/test/scripts/check-upload-flow.md`

## Key Reference Files (READ ONLY IF STUCK)

The inlined patterns above should be sufficient for all scripts. Only read these if you need more detail:

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
1. **Demo is king (30%)** — `--live` streaming + `--open` draft loading + `--dangerously-post` live posting = genuinely cool to watch
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
