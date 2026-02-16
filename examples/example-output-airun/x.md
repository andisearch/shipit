## 1

Your markdown files are sitting in git doing nothing. What if they could run themselves?

AIRun makes `.md` files executable with a shebang line. Write a prompt, `chmod +x`, run it like any script.

## 2

```
#!/usr/bin/env -S ai --haiku
---
vars:
  topic: "machine learning"
  style: casual
---
Write a summary of {{topic}} in a {{style}} tone.
```

Then from the terminal:

```
./summarize.md --topic "AI safety" --style formal
```

Prompts become reusable CLI tools with default arguments and overrides.

## 3

Hit a rate limit on your Claude Pro subscription? Switch providers and keep the conversation:

```bash
ai --aws --resume
ai --ollama --resume
ai --vertex --resume
```

Same session, different backend. Seven providers supported including local models through Ollama and LM Studio.

## 4

Because it's just bash and stdin/stdout, Unix pipes work:

```bash
cat data.json | ./analyze.md > results.txt
git log --oneline -20 | ./summarize-changes.md
./generate-report.md | ./format-output.md > final.txt
```

Chain AI scripts together the same way you'd chain grep, awk, and sed.

## 5

The `--live` flag streams output in real-time. When you redirect to a file, narration goes to stderr and clean content goes to the file:

```bash
./report.md > report.txt
```

You see progress in the terminal. The file gets clean output. No post-processing.

## 6

It wraps Claude Code, so you keep everything: MCP servers, CLAUDE.md project instructions, tool use, agent mode. AIRun just adds provider switching, script execution, and piping on top.

## 7

100+ models available through Vercel AI Gateway, including OpenAI, Google, Meta, Mistral, and DeepSeek:

```bash
ai --vercel --model openai/gpt-4o
ai --vercel --model google/gemini-2.0-flash
```

Or run local models free with Ollama and LM Studio. No API costs, nothing leaves your machine.

## 8

Open source, installs in one command:

```bash
git clone https://github.com/andisearch/airun.git
cd airun && ./setup.sh
```

Works on macOS and Linux. Only dependency is Claude Code and Bash 3.2+.

## Reply (Link)

GitHub: https://github.com/andisearch/airun

Built by the team at @andisearch
