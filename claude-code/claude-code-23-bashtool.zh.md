# BashTool：Shell 執行器深入解析

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 深入 BashTool 的實作細節，了解它如何讓 Claude Code 接入開發環境。

---

## BashTool：Shell 執行器

如果說 Read、Edit、Write 是 Claude Code 的精密手術刀，那 BashTool 就是它的重型工程機械。它讓 Claude Code 真正接入開發環境：跑測試、跑建置、看 Git 狀態、呼叫編譯器、包管理器、啟動開發服務。

***

### 命令語義分類

BashTool 不只是「執行字串」，它會主動理解命令類型：

🔍 搜尋改善 UI 呈現
📖 讀取自動折疊顯示
📋 列表行為更細理解

***

### 執行鏈路

模型決定執行 Shell
⬇
解析命令結構 (ast.js)
⬇
安全與權限檢查
⬇
判斷是否走 Sandbox
⬇
前台執行 或 註冊後台任務
⬇
stdout/stderr → 結果回注

***

### 前後台雙路徑

⚡ 前台同步
直接執行，等待結果適合快速命令🔙 後台 LocalShellTask
可透過 TaskOutputTool 讀取可透過 TaskStopTool 停止

***

### 一句話總結

> BashTool 不是裸執行器，而是被**權限、Sandbox、任務系統和產品邏輯**一起包住的受控執行層——它是 Claude Code 接入真實本地開發環境的橋樑。
