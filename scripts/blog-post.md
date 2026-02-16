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

Output ONLY the blog post. No preamble, no explanation, no markdown code fences wrapping the output.

Write a full blog post with:

1. A compelling title as an H1
2. Body structured as: what it is → why it exists → how it works → what's next
3. Real code examples from the briefing
4. A specific ending (not a generic CTA)

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
