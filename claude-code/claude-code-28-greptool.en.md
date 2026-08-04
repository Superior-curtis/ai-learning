# GrepTool: Searching Content

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how GrepTool wraps ripgrep into a formal search capability with structured, fed-back results.

---

## GrepTool: Searching Content

GrepTool is responsible for searching file contents for text or regex patterns. In real usage, it is often the first step in Claude Code's troubleshooting, code understanding, and locating implementations.

***

### Input Schema

***

### Three Output Modes

📄 content
See the context around matches📁 files\_with\_matches
First know which files are relevant🔢 count
Scope of impact or match volume

***

### Common Search Pipeline

🔎 Need to locate keywords/functions/config
⬇
🔍 GrepTool → regex search
⬇
📖 FileReadTool reads deeply at matched locations
⬇
✏️ Edit / Write / Bash follow-ups

***

### Summary in One Sentence

> GrepTool is the most commonly used **content-locating tool** in the main loop. It wraps ripgrep into a formal search capability that can be paginated, trimmed, and fed back structurally — many source-code analysis tasks truly begin with it.
