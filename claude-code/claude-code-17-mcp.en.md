# MCP and LSP Integration: Wiring the Outside World Into Claude Code

> 📅 2026-07-27 · Extensibility
> Break down the MCP and LSP integration mechanisms and see how Claude Code connects to the external tool ecosystem.

---

## MCP and LSP Integration: Wiring the Outside World Into Claude Code

If Claude Code's built-in tools answer "what can I do myself," then MCP answers **"who else can I connect to."**

Many people first encountering MCP understand it as a "plugin protocol." But from the source code, MCP looks more like a **bus for external capability integration**:

⚡
Claude Code
Runtime⟷🔧 MCP tools
📁 MCP resources
💬 MCP Prompts
🧩 MCP Skills⟷🌐
MCP Server
The outside world

***

### MCP Connection Methods

🖥️
stdio
Local subprocess MCP
Communicates via stdin/stdout🌐
HTTP / SSE
Remote HTTP MCP
Directly connects to remote MCP services🔗
WebSocket
Remote WebSocket MCP
Sustained connection, sustained interaction

***

### The MCP Tool Conversion Flow

MCP Server → tools/list⬇fetchToolsForClient translation⬇Wrapped into a unified Tool object⬇Enters the current tool list; the model calls it just like a built-in tool

***

### MCP Is More Than Tools: Resources and Dynamic Refresh

📁 Resource integration🔄 Dynamic refresh

MCP skills are explicitly treated as **remote and untrusted content** — Claude Code supports MCP skills while explicitly lowering their trust level.

***

### MCP vs. LSP Comparison

🔌
MCP
Expands capability outward
Connects external systems, resources, and tools📖
LSP
Fills in semantics inward
Diagnostics, symbols, references, semantic info

***

### One-Sentence Summary

> MCP wires the outside world into Claude Code, and LSP wires language intelligence into Claude Code — together they turn Claude Code from "I have these built-in capabilities" into "I can connect even more capabilities in at runtime."
