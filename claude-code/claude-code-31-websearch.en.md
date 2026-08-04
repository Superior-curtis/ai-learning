# WebSearchTool: Web Search

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tools
> Learn how WebSearchTool searches the web safely and always includes source citations.

---

## WebSearchTool: Web Search

WebSearchTool lets Claude Code query the internet for up-to-date information. The difference from WebFetchTool: **WebSearch finds sources, WebFetch reads a specific page**. The two are often used back to back.

***

### Input & Execution

📥 Inputquery: string
allowed_domains?: string[]
blocked_domains?: string[]⚙️ How it runsWrapper tool:
Outer standard tool call → inner second model call
extraToolSchemas injects web_search

***

### Search vs. Fetch

🌐
WebSearchTool
Finds sources📄
WebFetchTool
Reads a specific page

***

### Source Requirement

***

### One-Sentence Summary

> WebSearchTool is a wrapper tool — an outer standard tool call that internally issues a second model request via `extraToolSchemas` to run the search, then feeds the results (search hits mixed with text explanation) back to the main loop, and **mandates source citations**.
