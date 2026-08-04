# WebFetchTool: Fetching Web Pages

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how WebFetchTool safely fetches web pages and uses a small model to distill content.

---

## WebFetchTool: Fetching Web Pages

WebFetchTool is responsible for fetching a page at a known URL, converting it to text, and then distilling the result based on a prompt. Its division of labor with WebSearchTool is very clear: **WebSearch goes to the web to find results; WebFetch takes a confirmed URL and reads a specific page.**

***

### Core Design

📥 Input
url: stringprompt: string
The URL + what you want to ask🔐 Domain-level permissions
domain:hostnamedeny/ask/allow per domainpre-approved domains optimize the path

***

### Execution Pipeline

The model knows the target URL
⬇ WebFetchTool
URL validation + domain permission check
⬇
Fetch → convert HTML to Markdown
⬇
Small model (Haiku) distills according to the prompt
⬇
Distilled result returns to the main loop (not the whole page)

***

### Security Mechanisms

🔄 Redirect checksCross-domain redirects not auto-followed
⏳ 15-minute cacheLRU cache avoids re-fetching
📏 URL length limitMAX_URL_LENGTH = 2000

***

### Summary in One Sentence

> WebFetchTool isn't a browser; it's a **remote reader built for content extraction** — it fetches, converts, and distills with a small model, returning only the essence to the main loop and saving a great deal of context.
