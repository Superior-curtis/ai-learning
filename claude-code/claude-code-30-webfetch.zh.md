# WebFetchTool：抓取網頁

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 WebFetchTool 如何安全地抓取網頁並用小模型提煉內容。

---

## WebFetchTool：抓取網頁

WebFetchTool 負責抓取已知 URL 的網頁，轉成文字，再根據 prompt 提煉結果。它和 WebSearchTool 的分工非常明確：**WebSearch 去網上找結果，WebFetch 拿著確定的 URL 去讀具體頁面。**

***

### 核心設計

📥 輸入
url: stringprompt: string
URL + 想問什麼🔐 域名級權限
domain:hostname按域名 deny/ask/allow預批准域名優化路徑

***

### 執行鏈路

模型知道目標 URL
⬇ WebFetchTool
URL 校驗 + 域名權限檢查
⬇
抓取 → HTML 轉 Markdown
⬇
小模型（Haiku）根據 prompt 提煉
⬇
提煉結果回主循環（非整頁塞入）

***

### 安全機制

🔄 重定向檢查跨域不自動跟隨
⏳ 15 分鐘快取LRU 快取避免重複抓取
📏 URL 長度限制MAX\_URL\_LENGTH = 2000

***

### 一句話總結

> WebFetchTool 不是瀏覽器，而是**面向內容提取的遠端閱讀器**——抓取、轉換、用小模型提煉，只把精華回給主循環，大幅節省上下文。
