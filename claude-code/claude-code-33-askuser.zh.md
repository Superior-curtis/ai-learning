# AskUserQuestionTool：向使用者提問

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 AskUserQuestionTool 如何將提問變成正式互動協議。

---

## AskUserQuestionTool：向使用者提問

當模型執行到一半需要補資訊時，AskUserQuestionTool 把「提問」做成了正式、結構化、可互動的系統能力，而不是隨手打一句自然語言。

***

### 問題結構

***

### 執行位置

模型遇到歧義
⬇
AskUserQuestionTool → 結構化問題+選項
⬇
前端渲染 → 使用者選擇 → 答案回流
⬇
模型繼續執行

***

### 與 ExitPlanMode 的邊界

❓ AskUserQuestionTool
還缺資訊，繼續問✅ ExitPlanModeTool
計畫已完成，請審批

***

### 一句話總結

> 如果沒有這個工具，模型在遇到歧義時只能自己瞎猜或在自然語言裡隨便問一句。Claude Code 的做法是：把提問做成**正式互動協議**，讓結果可結構化、選項可解釋、可附帶 preview 對比。
