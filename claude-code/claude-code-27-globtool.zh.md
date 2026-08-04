# GlobTool：查找檔案

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 GlobTool 如何將檔案路徑搜尋變成標準化、結構化的發現入口。

---

## GlobTool：查找檔案

GlobTool 做的事很直接：按檔名或萬用字元模式找檔案。但在 Claude Code 的主循環裡，它承擔的是把「我大概知道要找什麼檔案」變成「我已經定位到候選路徑」。

***

### 輸入與特性

📥 輸入 Schemapattern: string
path: string (optional)模式 + 搜尋範圍⚡ 特性🟢 isReadOnly = true
🔄 isConcurrencySafe = true
🔍 isSearch = true
📏 預設結果上限 100

***

### Glob vs Grep

📁
GlobTool
我知道檔案大概叫什麼🔍
GrepTool
我知道檔案裡有什麼文字

***

### 在搜尋鏈中的位置

💡 大概知道檔名或後綴
⬇
📁 GlobTool → 候選路徑
⬇
📖 FileReadTool 繼續精讀
⬇
✏️ FileEditTool / FileWriteTool

***

### 一句話總結

> GlobTool 把「按檔案路徑模式發現目標檔案」做成了 Claude Code 的標準、唯讀、結構化搜尋入口——是很多任務真正開始的第一步。
