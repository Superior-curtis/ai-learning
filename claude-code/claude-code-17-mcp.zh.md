# MCP 與 LSP 整合：把外部世界接進 Claude Code

> 📅 2026-07-27 · 擴展能力
> 解析 MCP 與 LSP 整合機制，看 Claude Code 如何與外部工具生態連結。

---

## MCP 與 LSP 整合：把外部世界接進 Claude Code

如果說 Claude Code 的內建工具解決的是「我自己會什麼」，那 MCP 解決的就是\*\*「我還能接入誰」\*\*。

很多人第一次接觸 MCP，會把它理解成「插件協定」。但從原始碼來看，MCP 更像一層**外部能力接入匯流排**：

⚡
Claude Code
執行時期⟷🔧 MCP 工具
📁 MCP 資源
💬 MCP Prompts
🧩 MCP Skills⟷🌐
MCP Server
外部世界

***

### MCP 連線方式

🖥️
stdio
本地子行程 MCP
透過 stdin/stdout 通訊🌐
HTTP / SSE
遠端 HTTP MCP
直連遠端 MCP 服務🔗
WebSocket
遠端 WS MCP
持續連線、持續互動

***

### MCP 工具轉換流程

MCP Server → tools/list⬇fetchToolsForClient 翻譯⬇包裝成統一 Tool 物件⬇進入當前工具列表，模型像內建工具一樣呼叫

***

### MCP 不只是工具：資源與動態刷新

📁 資源接入🔄 動態刷新

MCP skills 被明確當成**遠端且不可信內容**處理——Claude Code 支援 MCP skills，同時明確降低它的信任級別。

***

### MCP vs LSP 對比

🔌
MCP
向外擴能力
接入外部系統、資源、工具📖
LSP
向內補語義
診斷、符號、引用、語義資訊

***

### 一句話總結

> MCP 把外部世界接進 Claude Code，LSP 把語言智慧接進 Claude Code——兩者讓 Claude Code 從「我會這些內建能力」變成「我可以在執行時期把更多能力接進來」。
