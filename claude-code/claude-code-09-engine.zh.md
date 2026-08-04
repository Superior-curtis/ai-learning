# 核心循環解析：QueryEngine 如何驅動一次任務

> 📅 2026-07-27 · 核心機制
> 深入核心循環 QueryEngine，看 Claude Code 如何驅動一次完整的任務執行。

---

## 核心循環解析：QueryEngine 如何驅動一次任務

如果只能選一個檔案代表 Claude Code 的靈魂，那就是 **QueryEngine.ts**。

它負責的不是某個局部能力，而是**整個任務生命週期**：

接收輸入
🎤
使用者提交訊息組裝上下文
🔧
提示詞 + 工具 + 狀態模型呼叫
🧠
驅動模型推理工具執行
⚡
處理工具呼叫結果維護狀態
💾
跨輪次狀態保留推進結束
✅
任務完成或中斷

***

### QueryEngine 的工作流程圖

1
使用者輸入 → submitMessage()⬇2
準備配置與上下文（tools, commands, mcpClients, cwd...）⬇3
構造系統提示與訊息歷史 → 呼叫模型⬇4
模型決策：是否要呼叫工具？🔧 YES → 執行工具 → 結果寫回歷史 → 回到步驟 3
💬 NO → 輸出最終結果

***

### 它管理的是「會話」，不是「一次請求」

原始碼中的註解已經把定位寫得很明確：

> **One QueryEngine per conversation.**

QueryEngine 不是一次性 request handler，而是圍繞會話長期存在的物件。

只看成員變數就能看出它明顯是「會話物件」：

💬
訊息歷史跨輪次保留🚫
權限拒絕不會重複詢問📊
用量成本持續累積追蹤

***

### submitMessage()：真正的任務入口

使用者每次提交訊息，最終都會進入 `submitMessage()`。

它不只是接收 `prompt`，還會同時讀取 tools、commands、mcpClients、budget、thinking 等執行時期資源——本質上是在**開啟一次完整任務**。

***

### 互動練習：QueryEngine 逐步排程

典型的 QueryEngine 任務流程追蹤$ 「幫我找這個檔案裡的 bug」→ submitMessage() 接收輸入
→ 讀取 config: tools, commands, cwd
→ 組裝系統提示 + 歷史訊息
→ 模型呼叫 FileReadTool
⬇ 執行工具 → 結果寫回歷史
→ 模型判斷：bug 不在這個檔案
→ 呼叫 GrepTool 搜尋相關程式碼
⬇ 執行工具 → 結果寫回歷史
→ 模型找到問題點，輸出答案
✓ 任務完成關鍵： 沒有「工具結果回流再決策」的循環，這種逐步偵錯過程根本不可能發生。

***

### QueryEngineConfig：依賴總表

`QueryEngineConfig` 幾乎可被看成 Claude Code 主循環的依賴總表：

***

### 模型與工具的閉環

🧠
模型根據系統提示和歷史作出決策🔧
決策可能包含工具呼叫🛂
工具呼叫前先經過權限判斷📨
工具結果轉成訊息回寫歷史⟳ 循環直到模型輸出最終結果

***

### 它也處理錯誤路徑

QueryEngine 不是理想化的 demo——它必須應對長會話、中斷、失敗、壓縮、恢復等現實問題：

⛔
abortController
使用者中斷任務📉
usage 統計
Token 和成本追蹤🔐
permission denial
拒絕操作記錄追蹤

***

### 任務結束後哪些狀態留下來？

這是會話型 Agent 和一次性腳本最大的區別：

保留
訊息歷史 → 下一輪能「接著聊」保留
已知權限拒絕資訊 → 不會重複問保留
usage 統計 + 檔案快取 + Skill/Memory 發現狀態

***

### 更準確的心智模型

> QueryEngine 不是「請求處理器」，而是 Claude Code **會話級執行時期中的任務編排器**。

它向上連接使用者輸入和 REPL，向下連接模型、工具、權限、上下文與狀態系統。

***

### 一句話總結

> `main.tsx` 決定「這次會話怎麼啟動」，而 **QueryEngine.ts** 決定的是：這次任務接下來到底怎樣一步一步做完。
