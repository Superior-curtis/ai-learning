# GlobTool: Finding Files

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how GlobTool turns file path search into a standardized, structured discovery entry point.

---

## GlobTool: Finding Files

What GlobTool does is straightforward: find files by name or wildcard pattern. But in Claude Code's main loop, it carries the job of turning "I roughly know which file I'm looking for" into "I've located candidate paths."

***

### Input and Features

📥 Input Schemapattern: string
path: string (optional)Pattern + search scope⚡ Features🟢 isReadOnly = true
🔄 isConcurrencySafe = true
🔍 isSearch = true
📏 Default result limit of 100

***

### Glob vs Grep

📁
GlobTool
I roughly know the file's name🔍
GrepTool
I know what text is inside the file

***

### Position in the Search Chain

💡 Roughly know the filename or suffix
⬇
📁 GlobTool → candidate paths
⬇
📖 FileReadTool reads deeper
⬇
✏️ FileEditTool / FileWriteTool

***

### Summary in One Sentence

> GlobTool turns "discovering target files by file path pattern" into Claude Code's standard, read-only, structured search entry point — the true first step where many tasks begin.
