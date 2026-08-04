# ExitPlanModeTool：退出 Plan Mode

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 ExitPlanModeTool 如何將計畫提交使用者審批。

---

## ExitPlanModeTool：退出 Plan Mode

ExitPlanModeTool 真正做的不是簡單退出，而是：**當計畫已經寫好後，把計畫提交給使用者審批。** 它代表 Plan Mode 的交付節點。

***

### 關鍵原始碼

***

### 與 AskUserQuestionTool 的邊界

❓ AskUserQuestionTool
不確定需求：補資訊✅ ExitPlanModeTool
計畫寫完求批准

***

### 一句話總結

> ExitPlanModeTool 把規劃階段的**收尾和審批流程**，變成了 Claude Code 裡一個明確的狀態轉換點——沒有它，Plan Mode 就會退化成一串自然語言消息。
