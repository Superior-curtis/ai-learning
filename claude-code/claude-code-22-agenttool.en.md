# AgentTool: The Sub-Agent Dispatcher

> 📅 2026-07-27 · Tooling
> A deep dive into Claude Code's AgentTool and how it splits tasks out to sub-agents for execution.

---

## AgentTool: The Sub-Agent Dispatcher

AgentTool is one of the most representative tools in Claude Code. What it solves isn't a single-point capability like reading a file or running a command, but rather: **when the main-thread model decides a task is too large, too messy, or too well-suited to parallelism, how to split part of the work off to another Agent.**

This is also one of the core dividing lines between Claude Code and ordinary code assistants.

***

### Input Schema

**AgentTool is, in essence, a task dispatcher.**

***

### Overall Flow

🧑‍💼 Main thread → AgentTool⬇Generate the sub-agent System Prompt⬇Assemble the tool pool + launch runAgent🖥️ LocalAgentTask🌍 RemoteAgentTask⬇ Results feed backMain thread continues to decide or aggregate

***

### What It Really Means

AgentTool does at least 4 things:

🔄
Regenerates the System Prompt🔧
Re-trims the tool pool📍
Decides local vs. remote tasks📋
Attaches to the task system

***

### Common Misconceptions

✗
It's not multi-turn chatIt's a formal tool with a schema and a lifecycle✗
A sub-agent isn't exactly the same as the main threadprompt, tool pool, and execution mode can all differ✗
It's not an advanced version of a promptIt mounts another Agent into the system as a first-class runtime object

***

### One-Sentence Summary

> AgentTool is not a single tool capability; it's Claude Code wrapping "task splitting + sub-agent execution + task lifecycle management" into a formal tool — it's the key node where Claude Code upgrades from "being able to call tools" to a "multi-task agent system."
