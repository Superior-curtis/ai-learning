# ListMcpResourcesTool：列出 MCP 資源

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 ListMcpResourcesTool 如何讓模型獲得 MCP 資源發現能力。

---

## ListMcpResourcesTool：列出 MCP 資源

ListMcpResourcesTool 的職責很像遠端世界裡的**目錄瀏覽**。如果模型已接上 MCP server，但不知道那邊暴露了哪些資源，第一步就應該用它先列出來。

### 一句話總結

> ListMcpResourcesTool 解決的是**資源發現問題**——沒有它，模型只能盲猜 URI。
