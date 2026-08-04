# WebSearchTool：聯網搜尋

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 WebSearchTool 如何安全地聯網搜尋並附帶來源引用。

---

## WebSearchTool：聯網搜尋

WebSearchTool 負責讓 Claude Code 查詢網際網路的最新資訊。它和 WebFetchTool 的區別：**WebSearch 找資訊源，WebFetch 讀指定頁面**。兩者經常一前一後配合使用。

***

### 輸入與執行

📥 輸入query: string
allowed\_domains?: string\[]
blocked\_domains?: string\[]⚙️ 執行方式包裝型工具：
外層標準工具 → 內層二次模型呼叫
extraToolSchemas 注入 web\_search

***

### 搜尋 vs 讀取

🌐
WebSearchTool
找資訊源📄
WebFetchTool
讀指定頁面

***

### 來源要求

***

### 一句話總結

> WebSearchTool 是包裝型工具——外層標準工具呼叫，內層透過 `extraToolSchemas` 發起第二次模型請求執行搜尋，再把結果混合搜尋命中和文字說明一起回主循環，並**強制附帶來源引用**。
