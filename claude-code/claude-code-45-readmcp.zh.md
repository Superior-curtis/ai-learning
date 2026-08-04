# ReadMcpResourceTool：讀取 MCP 資源

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 ReadMcpResourceTool 如何讀取遠端 MCP 資源（含二進位 blob）。

---

## ReadMcpResourceTool：讀取 MCP 資源

ReadMcpResourceTool 的職責非常明確：**給定 server + uri，把遠端 MCP 資源正文讀回來。**

### 一句話總結

> ReadMcpResourceTool 把**外部上下文讀入主循環**，是 MCP 整合真正落地的一環——還能處理二進位 blob 落盤。
