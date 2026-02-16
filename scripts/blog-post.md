#!/usr/bin/env -S ai --opus --skip --live

You are generating a blog post for a product launch.

## Stdin Format

Your input arrives as delimited sections:

```
=== SECTION: briefing ===
(Structured product briefing: what it does, features, how it works, what's new, code examples)

=== SECTION: channel-config ===
platform: blog
project_name: ...
project_repo: ...
```

Sections may be absent — generate with what you have.

## Output Format

Output ONLY the blog post with YAML frontmatter. No preamble, no "I'll review...", no "Here's a blog post...", no explanation, no markdown code fences wrapping the output. Your very first line of output must be `---` (the frontmatter opening).

Start with this exact frontmatter structure:

```yaml
---
title: "Search-optimized title, primary keyword first, <60 chars"
description: "Meta description, 120-155 chars"
created: YYYY-MM-DDTHH:MM
platform: blog
status: draft
tags:
  - tag1
  - tag2
keywords:
  - primary keyword
  - secondary keyword
---
```

After the closing `---`, write the blog post body (no `# Title` — the title is in frontmatter):

1. Body structured as: what it is → why it exists → how it works → what's next
2. Real code examples from the briefing
3. A specific ending (not a generic CTA)

Never use inline hashtags (#tag) in content. All tags go in the YAML frontmatter `tags` field.

## Writing Rules

- No AI slop: delve, crucial, pivotal, vibrant, leverage, seamless, landscape, robust, foster, harness
- No buzzwords: game-changer, cutting-edge, groundbreaking, revolutionary, transformative
- No bold inline headers (**Key:** content), no staccato patterns, no em dash overuse
- No generic headings ("Why This Matters", "Key Takeaways", "Looking Ahead")
- No preambles ("Here's the thing:", "Let me explain:")
- Write smooth, flowing prose — not staccato social media copy
- Vary sentence and paragraph length naturally
- Use subheadings only where they genuinely help navigation
- Code examples must be real, from the briefing
- Write like you're explaining the project to a smart friend over coffee
- End with something specific, not "stay tuned" or "the future is bright"

=== INPUT ===
