# AgentTool：子 Agent 調度器

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 深入了解 Claude Code 的 AgentTool，看它如何將任務拆分給子 Agent 執行。

---

## AgentTool：子 Agent 調度器

AgentTool 是 Claude Code 最有代表性的工具之一。它解決的不是「讀檔案」或「跑命令」這種單點能力，而是：**當主執行緒模型覺得一個任務太大、太雜、太適合並行時，如何把一部分工作拆給另一個 Agent 去做。**

這也是 Claude Code 和普通程式碼助手的核心分水嶺之一。

***

### 輸入 Schema

**AgentTool 本質上就是一個任務派發器。**

***

### 整體流程

🧑‍💼 主執行緒 → AgentTool⬇生成子 Agent System Prompt⬇組裝工具池 + 啟動 runAgent🖥️ LocalAgentTask🌍 RemoteAgentTask⬇ 結果回注主執行緒繼續決策或匯總

***

### 它的真實含義

AgentTool 至少做了 4 件事：

🔄
重新產生 System Prompt🔧
重新裁剪工具池📍
決定本地／遠端任務📋
掛到任務系統

***

### 常見誤解

✗
不是多輪聊天它是帶 schema 和生命週期的正式工具✗
子 Agent 和主執行緒不完全一樣prompt、工具池、運行模式都可能不同✗
不是進階版 prompt是把另一個 Agent 作為正式運行時物件掛進系統

***

### 一句話總結

> AgentTool 不是單個工具能力，而是 Claude Code 把「任務拆分 + 子 Agent 執行 + 任務生命週期管理」封裝成了一個正式工具——它是 Claude Code 從「會調工具」升級為「多任務 Agent 系統」的關鍵節點。
