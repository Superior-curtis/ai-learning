# Start Here: The Claude Code Learning Path

> 📅 2026-07-28 · Getting Started
> An interactive learning path to take you from zero to confident with Claude Code. Use the step navigator to find your next stop.

---

This is the entry point to the whole Claude Code series. If you don't know where to start, start here. Each article breaks down one topic with real examples and interactive elements that explain both the "why" and the "how".

> This is a series of articles that keeps growing. Walk the path today; come back weekly and new deep dives will appear in the categories on the right.

## The learning path

The step navigator below splits the series into five stages. Click the numbers to jump to the topic you're interested in.

#### Install and get going

Install Claude Code first and learn the sandwich rhythm: understand → plan → execute.\n\n→ Read: claude-code-01-quickstart

#### Build the right mental model

Grasp the core concept: it is not a chat box, but an agent that can read projects and run commands.\n\n→ Read: claude-code-03-core-concepts

#### Meet the core mechanics

Tools, commands, context, prompts — how these mechanics cooperate to drive a task.\n\n→ Read: claude-code-05-tools / 06-commands / 07-context

#### Go under the hood

Want the source-code view? Start with the QueryEngine core loop, then state and compression.\n\n→ Read: claude-code-09-engine / 13-state

#### Practice and extend

Write a good CLAUDE.md, use Plan Mode, connect MCP and Skills to build your own workflow.\n\n→ Read: claude-code-02-claudemd / 15-planmode / 17-mcp

## What happens inside a task

Once Claude Code takes your request, it runs a fixed loop internally. Here is the flow, simplified into five steps:

```text
user input → context assembly → model call → tool execution → observe feedback
↑____________________________↓
```

To dig deeper, visit the [simulator page](/simulator) where you can click each step to see the real prompts and payloads.

## Your first task

Try it. Open an empty directory and paste this:

```bash
cd ~/playground
mkdir -p first-project && cd first-project
claude
```

Then, inside Claude Code, type:

```
I'm in an empty directory and want to create a Next.js + TypeScript project from scratch.
First tell me the initialisation approach you'd suggest — don't run anything yet.
```

> Tip: every time, ask Claude Code to "propose a plan first, then execute only after confirmation". This is the most important habit for beginners.

## Check your understanding

#### Q: Which working rhythm suits Claude Code best?

* State the requirement and let it act immediately

* Understand → plan → execute, in three steps

* Dump a whole batch of requirements and let it sort them out

* Skip the terminal entirely

> 💡 The sandwich rhythm (understand → plan → execute) greatly reduces rework. Let it read the project, then ask for a plan, then execute.

## What's next

After walking the path, you can:

* Use the **search box** on the writing page to find a topic directly
* Jump to a section you care about from the **category sidebar**
* Play with the agent loop and tool catalog in the **simulator**
