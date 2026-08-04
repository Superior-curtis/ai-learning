# LSPTool：語言服務接入

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 LSPTool 如何讓 Claude Code 獲得 IDE 級程式碼語義能力。

---

## LSPTool：語言服務接入

LSPTool 讓 Claude Code 不再只依賴文字搜尋，而是能呼叫語言伺服器做**語義級理解**：go to definition、find references、hover、document symbol、call hierarchy。

***

### 輸入 Schema

***

### 搜尋工具對比

GlobTool
按檔名定位GrepTool
按文字定位（字串匹配）LSPTool
按語義定位（IDE 級）FileReadTool
讀取內容

***

### 一句話總結

> LSPTool 最值得學的地方是：Claude Code 已經不滿足於文字搜尋，而是把 **IDE 裡的語言伺服器能力正式接進了 Agent 工具鏈**。
