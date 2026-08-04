# Inside the Core Loop: How QueryEngine Drives a Task

> 📅 2026-07-27 · Core Mechanics
> Dive into the core loop, QueryEngine, and see how Claude Code drives a complete task execution.

---

## Inside the Core Loop: How QueryEngine Drives a Task

If you had to pick a single file to represent the soul of Claude Code, it would be **QueryEngine.ts**.

It isn't responsible for some peripheral capability — it handles the **entire task lifecycle**:

Receives input
🎤
User submits a messageAssembles context
🔧
Prompt + tools + stateModel call
🧠
Drives model reasoningTool execution
⚡
Processes tool call resultsMaintains state
💾
State preserved across turnsDrives to finish
✅
Task completes or is interrupted

***

### QueryEngine's Workflow Diagram

1
User input → submitMessage()⬇2
Prepare config and context (tools, commands, mcpClients, cwd...)⬇3
Build the system prompt and message history → call the model⬇4
Model decision: should it call a tool?🔧 YES → run the tool → write result back to history → back to step 3
💬 NO → output the final result

***

### It Manages a "Conversation," Not a "Single Request"

The comment in the source already makes the positioning explicit:

> **One QueryEngine per conversation.**

QueryEngine isn't a one-shot request handler — it's an object that lives alongside the conversation long-term.

Just looking at the member variables makes it clear this is a "conversation object":

💬
Message history is preserved across turns🚫
Permission denials aren't asked about twice📊
Usage and cost are continuously tracked

***

### submitMessage(): The Real Task Entry Point

Every time the user submits a message, it eventually enters `submitMessage()`.

It doesn't just receive `prompt` — it also reads runtime resources like tools, commands, mcpClients, budget, thinking, and so on — essentially **opening a full task**.

***

### Interactive Exercise: QueryEngine Step-by-Step Scheduling

Tracing a typical QueryEngine task flow$ "Help me find the bug in this file"→ submitMessage() receives the input
→ reads config: tools, commands, cwd
→ assembles system prompt + message history
→ model calls FileReadTool
⬇ runs tool → result written back to history
→ model decides: the bug isn't in this file
→ calls GrepTool to search related code
⬇ runs tool → result written back to history
→ model finds the problem, outputs the answer
✓ task completeKey point: Without the "tool results flowing back for re-decision" loop, this kind of step-by-step debugging could never happen.

***

### QueryEngineConfig: The Dependency Ledger

`QueryEngineConfig` can almost be seen as the dependency ledger of Claude Code's main loop:

***

### The Model-Tool Closed Loop

🧠
The model makes decisions based on the system prompt and history🔧
The decision may include a tool call🛂
Tool calls pass through permission checks first📨
Tool results become messages written back to history⟳ Loop until the model outputs a final result

***

### It Also Handles Error Paths

QueryEngine isn't an idealized demo — it has to deal with real-world concerns like long sessions, interruptions, failures, compression, and recovery:

⛔
abortController
User interrupts the task📉
usage stats
Token and cost tracking🔐
permission denial
Tracks denied operations

***

### What State Survives After a Task Ends?

This is the biggest difference between a session-based agent and a one-shot script:

Kept
Message history → next turn can "continue the conversation"Kept
Known permission denials → won't be asked againKept
usage stats + file cache + Skill/Memory discovery state

***

### A More Accurate Mental Model

> QueryEngine isn't a "request handler" — it's the **task orchestrator inside Claude Code's session-level runtime**.

It connects upward to user input and the REPL, and downward to the model, tools, permissions, context, and state systems.

***

### One-Sentence Summary

> `main.tsx` decides "how this session starts," while **QueryEngine.ts** decides: how this task actually gets done, step by step.
