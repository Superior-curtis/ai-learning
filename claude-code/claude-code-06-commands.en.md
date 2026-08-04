# Slash Commands: How Users Directly Control Claude Code

> 📅 2026-07-27 · Core Mechanics
> Breaking down Claude Code's slash command system and how users interact with tools directly.

---

## Slash Commands: How Users Directly Control Claude Code

There are two mechanisms in Claude Code that are easy to confuse:

🔧
Tool system
Called by the model
FileRead, Bash, WebSearch...⌘
Command system
Explicitly entered by the user
/config, /mcp, /review, /plugin...

Things like `/config`, `/mcp`, `/review`, `/plugin` aren't meant to be called at random by the model inside the main loop — they're **control entry points** the user triggers directly.

***

### Routing the user's input

User inputStarts with /?YES → Command handlerConfig / status / sub-feature interfaces
Directly control system behaviorNO → QueryEngineModel + tool loop
The AI decides how to handle it automatically

***

### commands.ts: the command aggregation hub

You can already tell from the import list that Claude Code's command capabilities are very rich:

These commands cover:

⚙️Configuration and environment🔐Login and sessions👁️Review and diff🔌MCP and plugins📊model/usage/status/cost📋plan/permissions/hooks

***

### Interactive exercise: commands vs natural language

Match: command or natural language?Switching models
→
/model commandRefactoring a function in a file
→
natural-language taskChecking usage stats
→
/usage commandExplaining an error message
→
natural-language taskReviewing code changes
→
/review commandManaging MCP servers
→
/mcp commandRule: When the goal is "get the system into a well-defined state," use a command. When the goal is "let the model reason and carry out a task," use natural language.

***

### Why does the user need a command system?

If everything is left to the model's automatic decisions, the system becomes powerful — but it also loses explicit control.

🎯
Explicit triggering
Some operations are better initiated by a human⚡
Deterministic flows
Management actions need deterministic flows🛂
Control-plane tasks
Configuring permissions shouldn't be left to AI fuzziness

***

### Command system vs tool system

Dimension
Command system
Tool systemUser
Entered directly by the user
Called indirectly by the modelTrigger
/command input
Called by the model's decisionPurpose
Control system state
Execute task stepsExamples
/review, /config, /mcp
FileRead, Bash, WebSearch

***

### The engineering thinking behind it

The command system isn't just an executor — it's part of the entire product's feature map.

1.The command layer is how capabilities are organized
commands.ts groups a sprawling set of capabilities by topic: configuration, state, MCP/plugins, and so on.2.It shares the underlying world with the tool system
It's affected by the same configuration and feature gates, and reads/writes the same AppState — only the user is different.3.Mature products keep a command surface
A good tool doesn't just do work automatically — it also lets you explicitly take over, switch, and manage the execution environment.

***

### One-sentence summary

> **Commands** are the entry point for "a human operating the system directly"; **tools** are the entry point for "the model operating the system on the human's behalf." Both share the same underlying world — only the user is different.
