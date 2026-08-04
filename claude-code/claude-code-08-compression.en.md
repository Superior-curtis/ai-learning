# Context Compression Management: Claude Code's Six-Tier Compression

> 📅 2026-07-27 · Core Mechanics
> Understand Claude Code's six-tier compression mechanism and how it fits more content into a limited window.

---

## Context Compression Management: Claude Code's Six-Tier Compression

Many people assume, when they first meet Claude Code, that its "long context" ability comes mainly from the model itself. But looking at the source, what really sustains long tasks isn't just the context window — it's a complete **tiered compression mechanism**.

***

### Overview of the Six-Tier Compression Pipeline

1
Tool Result Budget
Trims oversized tool results2
Snip
Local slimming trim3
Microcompact
Fine-grained micro compression4
Context Collapse
Collapses the view, preserving structure5
Autocompact
Automatic summary compression6
Reactive Compact
Fallback after API errors

***

### Why Does Claude Code Have to Compress?

Claude Code's task isn't a one-off Q\&A — it's **continuously executing engineering tasks**:

📂 Read multiple files
🔍 Search the codebase
⚡ Run Bash commands
📝 Write files and apply patches

These actions keep appending new messages and tool results to the conversation history. **Without compression, the model would quickly fill up.**

***

### Layer 1: Tool Result Budget

The first to run is `applyToolResultBudget()`. Its goal is straightforward: before entering real compression, replace or trim tool results that are obviously too large.

What takes up space isn't usually user messages, but:

* Large chunks of terminal output from BashTool
* Long results returned by search tools
* Big file fragments read by file-read tools

***

### Layer 2: Snip

Snip and microcompact are **not mutually exclusive**. Snip runs earlier and is a lighter **local trim**: it makes local cuts to low-value parts without destroying the main conversation structure.

***

### Layer 3: Microcompact

Microcompact does fine-grained compression around **tool call records**. The difference from snip:

🗂 Tool call records✂️ Snip first trims obvious redundancy🔍 Microcompact then micro-compresses

***

### Layer 4: Context Collapse

This is the most easily overlooked layer — and actually **a very clever one**.

💡
The key idea isn't "deleting history" — it's "re-projecting the view"🎯
The underlying log isn't necessarily erased — but the view fed to the model is collapsed📝
There are commit records and snapshots: contextCollapseCommits, contextCollapseSnapshot

***

### Layer 5: Autocompact

This is where what people usually mean by "automatic summary compression" actually happens.

Key points:

✓
It doesn't just produce a summary — it also rebuilds the post-compact message sequence✓
The compaction result is written back into the current conversation execution chain

***

### Layer 6: Reactive Compact

If the proactive compression above still isn't enough, Claude Code also has a **fallback path**.

It handles two common failures:

* **prompt too long** (413 error)
* **media content too large** (images/PDFs/multi-image inputs)

This really shows Claude Code's engineering maturity — it doesn't assume "proactive compression will always succeed," but instead **builds failure recovery into the main loop design**.

***

### Tiered Compression Flow Diagram

Raw history
→ lots of tool results + attachments + multiple rounds of conversation⬇① Budget trimming
→ trims oversized tool output⬇② Snip
→ locally trims low-value content⬇③ Microcompact
→ micro-compresses by tool_use structure⬇④ Context Collapse
→ collapses the view, preserving structure as much as possibleStill within threshold → keep granular detail
Still over → continue ⬇⬇⑤ Autocompact
→ global summary compression + compact_boundary⬇⑥ Reactive
→ fallback recovery when the API returns 413/media

***

### Compact Boundary: Persisting Compression

Compression doesn't just happen in memory — it also affects **session persistence and recovery**.

***

### One-Sentence Summary

Claude Code's context compression isn't as simple as "doing one summary when you approach the limit" — it's a **tiered memory management system**: the earlier layers trim locally where possible, the middle collapses views to preserve structure, the later layers do global summaries, and finally there's recovery compression after errors.
