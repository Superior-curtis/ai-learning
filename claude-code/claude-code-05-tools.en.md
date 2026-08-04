# The Tool System: Claude Code's Execution Power Comes from Tools, Not Just the Model

> 📅 2026-07-27 · Core Mechanics
> A deep dive into Claude Code's tool system — how it gives the model the ability to actually execute code.

---

## The Tool System: Claude Code's Execution Power Comes from Tools, Not Just the Model

When people discuss AI programming, it's easy to focus all the attention on the model. But looking at the Claude Code source code, what truly lets it "get work done" is the **tool system** that wraps the model.

***

Without tools, Claude Code at its strongest would still just analyze code. With tools, it can actually read files, modify files, run commands, connect to MCP, ask the user, and spawn sub-tasks.

Model
Analyze code → output suggestions⬇Tool system
Read files, edit files, run commands, search, MCP...⬇Execute
Actually rewrite files, deploy, query data...

***

### Tool.ts defines the unified contract

**Tool.ts** isn't an implementation of any specific tool — it's the system-wide tool abstraction layer. Its two most important values:

① Unified semanticsUniformly define the input, output, context, and permission semantics of tools② Unified contractGive every tool the same execution contract

***

### The key type: ToolInputJSONSchema

```text
This definition is critical. It shows that Claude Code's tools can't just be a loose chunk of text description — they need:

→ a clear input structure
→ enumerable parameters
→ a tool contract the system can understand
```

***

### Interactive exercise: compare Tool types

Which one conforms to the Tool spec?✅ Correct❌ Not spec-compliantWhy? Claude Code's tools need a structured JSON Schema so the model can understand the parameters precisely. A text description is meaningful to humans, but it isn't explicit enough for the system.

***

### ToolUseContext: the full context of a tool execution

A tool doesn't operate in isolation when it executes — it's placed into the complete runtime context.

#### What a tool gets at execution time

ToolUseContext📋
commands
Slash command registry🔧
tools
Available tool set🔌
mcpClients
MCP external services💾
AppState
Session state⛔
abortController
Abort control💬
messages
Conversation history

***

### tools.ts: the system's tool catalog

If **Tool.ts** defines the contract, then **tools.ts** defines "exactly which tools this system has."

**Two things worth noting:**

1. Claude Code's capability surface is very wide — it isn't just reading/writing files and Bash
2. The tool set isn't static — it's affected by environment variables and feature conditions

***

### The four tool groups

Core execution toolsSession control toolsExtension integration toolsCollaboration and task tools

***

### Tools aren't all exposed — they're dynamically filtered

The key is `filterToolsByDenyRules` — tools are filtered by permission rules before the model ever sees them.

#### The tool filtering layers

📦 All tools that exist in the codebase⬇🚩 feature-gate filtering⬇🖥️ platform / environment filtering⬇🔒 permission-rule filtering⬇🎯 The tools actually exposed in the current session

***

### Interactive exercise: match the filter conditions

Match conditions to resultsCondition A
ENABLE_LSP_TOOL is not setResult
LSPTool is not registeredCondition B
hasEmbeddedSearchTools() returns trueResult
GlobTool and GrepTool are skippedCondition C
The user denied BashToolResult
The model can't see BashTool

***

### Why is a unified tool contract so important?

Once the contract is stable, the way you add new capabilities becomes very predictable:

1
A new capability implements its own Tool2
Plugs into the unified context and permission system3
Registers into the tool catalog4
Is exposed to the model under the right conditions

***

### Claude Code's engineering philosophy

The tool system's design reveals these leanings:

🧩
Unified capability encapsulation
Abstract the contract first, then plug in specific capabilities🛡️
Permissions up front
Start constraining at the capability-exposure stage⚙️
Enabled per environment
Different environments get different tool sets🔌
Extension-first
MCP, LSP, and Team can all plug in

***

### One-sentence summary

> Use a unified **Tool** contract to wrap the file system, terminal, search, external integrations, and agent capabilities into an execution layer the model can call, control, and extend.

Once you understand this layer, you'll see why Claude Code isn't a simple "chat + plugins" combination, but a genuinely operable engineering agent.
