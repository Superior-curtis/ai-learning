# Claude Code Core Concepts at a Glance

> 📅 2026-07-27 · Core Concepts
> Understand Claude Code's core concepts and how it works at a glance.

---

## Claude Code Core Concepts at a Glance

Many people get bombarded by a pile of jargon when they first encounter Claude Code. If you haven't built a basic understanding of these terms first, it's very easy to get lost later.

The goal of this post isn't to dig into implementation details — it's to give you a "map of core concepts" first.

***

### 1. QueryEngine — the core engine

Claude Code's brain dispatcher. It decides how a task advances round by round: understand the request → choose a tool → execute → observe the result → adjust → continue.

```
User input → QueryEngine → choose tool → execute → return result
                                       ↑         │
                                       └─ loop ──┘
```

### 2. Tool — the execution interface

Tool is how Claude Code lets the model "actually get its hands dirty." The model doesn't just output text — through Tools it can read files, write files, run commands, and search the web.

Claude Code currently has **50+ tools**, spanning categories like file operations, Shell execution, network requests, and task management.

### 3. AppState — the session state hub

AppState is the runtime state hub of the terminal UI. What it records isn't the state of some small widget — it's what's happening in the entire session right now.

Understand it in one sentence: AppState decides "what state the current terminal session is in."

### 4. Context — project context

Context refers to the environment information Claude Code feeds the model in every round of a task. Typical contents include:

* The current directory structure
* The project conventions in CLAUDE.md
* Git history and changes
* The system prompt

Context explains why Claude Code seems to "understand your project."

### 5. Plan Mode — planning mode

Plan Mode doesn't change code directly. Instead, it has Claude Code plan and analyze first, you review, and only after the review passes does execution begin.

```
You: make X for me
      ↓
Claude: proposes a plan → you review it
      ↓
You: approve → Claude executes
```

### 6. MCP — external capability expansion

MCP (Model Context Protocol) lets Claude Code connect to external services: databases, APIs, file systems, and more. It doesn't rely only on built-in capabilities — it can connect to the outside world.

### 7. LSP — Language Server Protocol

LSP helps Claude Code get more structured, code-aware semantics, such as syntax checking, go-to-definition, and type queries. It lets Claude Code see code as more than plain text — it can leverage the language toolchain to understand code.

### 8. Skills — encapsulated experience

Skills can be understood as an encapsulation of task experience and working methods. They aren't tools themselves; they're more like a "SOP for what to do in this kind of situation."

### 9. Agent — multi-role collaboration

Agent means Claude Code doesn't necessarily have just one mainline assistant — it can also spawn sub-agents to run independent tasks, enabling multi-role collaboration.

### 10. Prompt / System Prompt — dynamic prompts

Claude Code's prompts aren't a fixed block of text — they're assembled dynamically. Every task combines the current context, project settings, tool list, and more to compose the most suitable prompt.

***

### A translation table to understand before reading the source code

| Term | How to understand it |
|------|----------|
| QueryEngine | The core task engine that decides how a task advances |
| Tool | The execution interface — how the model "gets its hands dirty" |
| Context | Project context that lets the AI understand your project |
| AppState | The session state hub that records what's happening right now |
| Plan Mode | A plan-and-approve mode: plan first, then execute |
| MCP | External capability expansion — connect databases and APIs |
| LSP | Language server for structured code understanding |
| Skills | Encapsulated experience — an SOP for specific situations |
| Agent | Multi-role collaboration — subtasks run independently |

Understanding these terms will make the upcoming posts much easier.
