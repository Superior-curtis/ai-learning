# Claude Code Complete Beginner's Guide

> 📅 2026-07-27 · Claude Code / AI Tools / Tutorial
> From architecture to hands-on practice, deeply understand how Claude Code works at its core

---

## What Is Claude Code

Claude Code is a terminal AI assistant from Anthropic. It's not a plugin or an IDE extension — it's an autonomous coding tool that runs directly in the terminal: you give it a task, and it decides for itself how to complete it.

Its biggest difference from Cursor and Copilot is that Claude Code has a full **Agent Loop** — from understanding your request, planning steps, executing tools, observing results, to adjusting strategy, the entire process is completed autonomously by the AI.

***

## Core Architecture: The Agent Loop

Claude Code's workflow can be summarized as a continuously looping process. Walk through it with the step navigator below:

#### Receive input

You issue a command or ask a question.

#### Understand the task

The model analyzes the current context and decides what needs to be done.

#### Choose a tool

Picks the most suitable one from the 50+ tools.

#### Execute the tool

Invokes the tool and gets the result.

#### Observe the output

Adjusts the next step based on the results returned by the tool.

#### Repeat or finish

Keeps looping until the task is done, or returns the result to you.

> This loop happens every second; each iteration brings the AI closer to the result you want.

***

## The 50+ Tool System

Claude Code's greatest strength is its tool ecosystem. Each tool is a unit of capability, and the AI automatically selects and combines them based on the task.

### File Operations

* **FileReadTool** — read file content
* **FileWriteTool** — write/create files
* **FileEditTool** — precisely edit files (replace specific lines)
* **GlobTool** — search file paths by pattern
* **GrepTool** — search for content inside files

### Execution

* **BashTool** — run any command in the terminal. This is Claude Code's most powerful tool — it can compile, install, test, and deploy

### Network

* **WebFetchTool** — fetch web page content and convert it to Markdown
* **WebSearchTool** — search the web (available in the US)

### Collaboration

* **AskUserQuestionTool** — proactively ask you questions when clarification is needed
* **TodoWriteTool** — create a task list to track progress
* **SendMessageTool** — communication between sub-agents

### Task Management

* **TaskCreateTool / TaskGetTool / TaskUpdateTool / TaskListTool / TaskStopTool / TaskOutputTool** — full task lifecycle management
* **AgentTool** — launch a sub-agent to run an independent task

### Dev Tool Integration

* **LSPTool** — integrates a language server, providing syntax checking, jump-to-definition, and more
* **ListMcpResourcesTool / ReadMcpResourceTool** — access MCP server resources

***

## The Skills System

Skills are Claude Code's extensible capability packages. Anyone can write a Skill to give Claude Code specialized expertise in a particular domain.

Each Skill is essentially a set of Markdown-format instruction files, stored in the project's `.claude/skills/` directory. When you invoke a Skill, its content is injected into the AI's prompt, letting the AI know how to handle a specific type of task.

### What Skills Can Do

* Project-specific code convention checks
* Deployment process automation
* Code review standards
* Test-writing templates

***

## MCP Integration

MCP (Model Context Protocol) is an open protocol from Anthropic that lets AI models safely access external tools and data sources. Claude Code supports MCP servers, which means it can connect to databases, APIs, file systems, and other external resources through a standardized interface.

This lets Claude Code work not only inside your project folder, but also interact with your entire development infrastructure.

***

## Practical Use Cases

### 1. Code Generation and Refactoring

```
給一個 Express CRUD API 加上 TypeScript 型別定義
```

Claude Code will first read the existing files → analyze the type requirements → generate the code → write it to the files.

### 2. Bug Debugging

```
為什麼這個 API 回傳 500？
```

It will grep the error logs → read the relevant files → analyze possible causes → run tests to verify → report the results.

### 3. Project Migration

```
把這個 Vue 2 專案遷移到 Vue 3
```

It will traverse all the files → identify what needs to change → modify step by step → verify the build.

***

## Comparison with Other Tools

| Feature | Claude Code | Copilot | Cursor |
|------|------------|---------|--------|
| Autonomous agent | ✅ Full Agent Loop | ❌ Completion only | ⚠️ Partial |
| Number of tools | 50+ | ~5 | ~20 |
| Terminal operations | ✅ Native | ❌ | ❌ |
| Skills system | ✅ Extensible | ❌ | ❌ |
| MCP integration | ✅ | ❌ | ❌ |
| Sub-agents | ✅ Multi-agent orchestration | ❌ | ❌ |

***

## Advice for Beginners

1. **Start with simple tasks** — have Claude Code refactor a function or write a test for you first
2. **Make good use of CLAUDE.md** — create this file at the project root and write in your project conventions, tech stack, and code style; Claude Code will follow them automatically
3. **Watch its decision-making process** — notice which tools it picks and why; this helps you understand its capability boundaries
4. **Use Plan Mode** — for complex tasks, let it plan first, then you review the plan, then execute
5. **Give it clear context** — tell it which file to change, what the problem is, and what result you expect

***

## Check your understanding

#### Q: What is the biggest difference between Claude Code and Cursor / Copilot?

* It can operate directly in the terminal

* It has a full Agent Loop and can complete tasks autonomously

* Its model is more capable

* It can only chat in the CLI

> 💡 The full Agent Loop — understand → plan → execute tools → observe → adjust — lets Claude Code complete tasks autonomously. That is the key difference from completion-style tools.

***

Claude Code is not a code-writing tool — it's an autonomous development partner that can understand, plan, execute, and debug. Learn to use its tool system and Agent Loop, and you'll find that many tasks that used to take half a day now take just a few minutes.
