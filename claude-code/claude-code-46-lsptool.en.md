# LSPTool: Language Service Integration

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how LSPTool gives Claude Code IDE-grade code semantics.

---

## LSPTool: Language Service Integration

LSPTool lets Claude Code stop relying only on text search, and instead call a language server for **semantic-level understanding**: go to definition, find references, hover, document symbol, call hierarchy.

***

### Input Schema

***

### Search Tool Comparison

GlobTool
Locate by filenameGrepTool
Locate by text (string matching)LSPTool
Locate by semantics (IDE-grade)FileReadTool
Read content

***

### One-Sentence Summary

> The most valuable lesson from LSPTool: Claude Code is no longer satisfied with text search — it has formally brought **the language server capabilities from the IDE into the agent toolchain**.
