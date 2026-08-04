# 上下文壓縮管理：Claude Code 的六層分級壓縮

> 📅 2026-07-27 · 核心機制
> 了解 Claude Code 的六層分級壓縮機制，如何在有限視窗中塞入更多內容。

---

## 上下文壓縮管理：Claude Code 的六層分級壓縮

很多人第一次接觸 Claude Code，都會以為它的「長上下文」能力主要來自模型本身。但從原始碼看，真正撐住長任務的，不只是上下文視窗，而是一整套**分級壓縮機制**。

***

### 六層壓縮管線總覽

1
Tool Result Budget
裁剪過大的工具結果2
Snip
局部瘦身裁剪3
Microcompact
細粒度微壓縮4
Context Collapse
折疊視圖保留結構5
Autocompact
自動摘要壓縮6
Reactive Compact
API 報錯後的兜底

***

### 為什麼 Claude Code 必須做壓縮？

Claude Code 的任務不是一次問答，而是**持續執行工程任務**：

📂 讀取多個檔案
🔍 搜尋程式碼庫
⚡ 執行 Bash 命令
📝 寫檔和打補丁

這些行為會不斷把新訊息和工具結果追加進對話歷史。**如果沒有壓縮，模型很快就會被塞滿。**

***

### 第 1 層：Tool Result Budget

最先運作的是 `applyToolResultBudget()`，目標很直接：在進入真正的壓縮之前，先把明顯過大的工具結果做替換或裁剪。

佔空間的往往不是使用者訊息，而是：

* BashTool 打出的大段終端輸出
* 搜尋工具回的長結果
* 檔案讀取工具讀到的大檔案片段

***

### 第 2 層：Snip

Snip 和 microcompact **不是互斥關係**。Snip 更靠前，屬於更輕量的**局部瘦身**：在不破壞主要會話結構的前提下，先對低價值部分做局部裁剪。

***

### 第 3 層：Microcompact

Microcompact 圍繞**工具呼叫記錄**做細粒度壓縮。和 snip 的區別：

🗂 工具呼叫記錄✂️ Snip 先裁掉明顯冗餘🔍 Microcompact 微壓縮

***

### 第 4 層：Context Collapse

這是最容易被忽視、但實際上**非常聰明的一層**。

💡
關鍵思想不是「刪除歷史」，而是「重新投影視圖」🎯
底層日誌未必被抹掉，但餵給模型的視圖被折疊了📝
有提交記錄和快照：contextCollapseCommits, contextCollapseSnapshot

***

### 第 5 層：Autocompact

真正大家通常理解的「自動摘要壓縮」在這裡。

關鍵點：

✓
不只是產生摘要，還會重建 post-compact 訊息序列✓
壓縮結果會回寫進當前對話執行鏈

***

### 第 6 層：Reactive Compact

如果前面的主動壓縮還是不夠，Claude Code 還有一條**兜底鏈路**。

處理兩種常見失敗：

* **prompt too long**（413 錯誤）
* **媒體內容過大**（圖片/PDF/多圖輸入）

這很能體現 Claude Code 的工程成熟度——它不是假設「主動壓縮一定成功」，而是**把失敗恢復也納入主循環設計**。

***

### 分級壓縮流程圖

原始歷史
→ 大量工具結果 + 附件 + 多輪對話⬇① 預算裁剪
→ 裁掉過大的工具輸出⬇② Snip
→ 局部裁剪低價值內容⬇③ Microcompact
→ 按 tool_use 結構微壓縮⬇④ Context Collapse
→ 折疊視圖，盡量保留結構仍在閾值內 → 保留細粒度
仍超標 → 繼續 ⬇⬇⑤ Autocompact
→ 全局摘要壓縮 + compact_boundary⬇⑥ Reactive
→ API 報 413/media 時兜底恢復

***

### Compact Boundary：壓縮落盤

壓縮不僅發生在記憶體裡，還會影響**會話持久化和恢復**。

***

### 一句話總結

Claude Code 的上下文壓縮，不是「接近上限時做一次摘要」這麼簡單——它是一套**分層記憶管理系統**：前面幾層盡量局部瘦身，中間折疊視圖保留結構，後面才做全局摘要，最後還有報錯後的恢復壓縮。
