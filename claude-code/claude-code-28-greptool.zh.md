# GrepTool：搜尋內容

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 GrepTool 如何將 ripgrep 封裝成可結構化回注的正式搜尋能力。

---

## GrepTool：搜尋內容

GrepTool 負責在檔案內容裡搜尋文字或正則模式。在真實使用中，它常常是 Claude Code 排查問題、理解程式碼和定位實作的第一步。

***

### 輸入 Schema

***

### 三種輸出模式

📄 content
看匹配周邊上下文📁 files_with_matches
先知道哪些檔案相關🔢 count
影響範圍或匹配規模

***

### 常見搜尋鏈路

🔎 需要定位關鍵詞/函數/配置
⬇
🔍 GrepTool → 正則搜尋
⬇
📖 FileReadTool 深讀命中位置
⬇
✏️ Edit / Write / Bash 後續

***

### 一句話總結

> GrepTool 是主循環裡最常用的**內容定位工具**，它把 ripgrep 封裝成了可分頁、可裁剪、可結構化回注的正式搜尋能力——很多原始碼分析任務，真正開始於它。
