#!/usr/bin/env -S ai --opus --skip --live

Analyze the code repository provided on stdin. Produce a structured product briefing. Output clean markdown only — no commentary, no preamble.

## Stdin Format

Your input arrives as delimited sections:

```
=== SECTION: repo ===
(Repository contents: README, package manifests, recent commits, changelog, code structure)
```

## Instructions

Read the repository contents and produce a structured briefing covering:

1. **What the project does** — One paragraph summary
2. **Key features** — Bulleted list, each with a one-line description
3. **How it works** — Technical overview: architecture, execution model, dependencies
4. **Installation & usage** — How to install, basic usage examples, key commands/flags
5. **What's new** — Recent changes from commits/changelog
6. **Code examples** — 3-4 real usage examples pulled from the repo. Use actual commands and flags — do not invent examples.

## Output Format

Output clean markdown with the sections above. Use actual code from the repo for examples. Be specific — include real flag names, real file paths, real command syntax.

Do not editorialize or write marketing copy. This is raw research material that feeds into content generation.

=== INPUT ===
