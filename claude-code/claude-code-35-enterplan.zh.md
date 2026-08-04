# EnterPlanModeTool：進入 Plan Mode

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 EnterPlanModeTool 如何將會話切換到規劃執行狀態。

---

## EnterPlanModeTool：進入 Plan Mode

EnterPlanModeTool 的作用不是「讓模型多想一會兒」，而是把當前會話切到一種新的執行狀態：**plan**。系統會同時改變權限模式、會話目標、使用者互動預期。

***

### 關鍵原始碼

***

### 執行鏈路

模型判斷任務過大/過複雜
⬇
EnterPlanModeTool
⬇
更新 permission mode → plan
⬇
進入規劃態

***

### 一句話總結

> EnterPlanModeTool 把「先規劃再編碼」從一種提示習慣，**升級成了 Claude Code 執行時期裡的正式狀態機切換**。
