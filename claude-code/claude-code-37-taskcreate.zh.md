# TaskCreateTool：建立任務

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 TaskCreateTool 如何將工作項建立成帶生命週期的正式任務物件。

---

## TaskCreateTool：建立任務

TaskCreateTool 不是簡單往列表裡插入一行文字，而是把一個工作項建立成**正式任務物件**：有 id、subject、description、狀態，可被後續更新、阻塞、歸屬。

***

### 關鍵原始碼

***

### 與 TodoWriteTool 的區別

📋 TodoWriteTool
輕量清單，寫入 AppState📁 TaskCreateTool
正式任務物件，帶生命週期

***

### 一句話總結

> TaskCreateTool 代表的是 Claude Code 從「todo 文字」到\*\*「正式任務物件」\*\*的那一步——有 id、狀態、依賴、hooks 的執行時物件系統。
