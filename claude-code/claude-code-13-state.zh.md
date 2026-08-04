# 狀態管理解析：AppStateStore 裡到底保存了什麼？

> 📅 2026-07-27 · 深入研究
> 深入 AppStateStore，解析 Claude Code 的狀態管理機制與資料結構。

---

## 狀態管理解析：AppStateStore 裡到底保存了什麼？

只要看 `state/AppStateStore.ts`，你就會立刻意識到一件事：Claude Code 絕不是一個「執行一下就退出」的普通 CLI。它內部維護了大量 UI 與會話狀態資料——**它本質上是一個終端應用程式**。

***

### 這裡保存的狀態遠比你想像的多

AppState 裡能看到非常多類型的資訊：

⚙️ settings / model
當前設定和模型選擇🛂 工具權限上下文
canUseTool 狀態📋 任務列表
前後台任務管理🔌 MCP 客戶端
工具/命令/資源狀態🧩 插件狀態
啟用和錯誤資訊🔔 通知佇列
與 elicitation 佇列

📡 遠端連線狀態💡 prompt suggestion / thinking📝 Todo / attribution / file history🖥️ Companion / tmux / browser

***

### 為什麼終端程式需要這麼重的狀態？

因為 Claude Code 不只是顯示文字，它還要同時協調很多**非同步物件**：

🧠模型回應流🔧工具執行⏳背景任務🛂權限彈窗🔌MCP 連線🧩插件刷新🌉遠端橋接💬通知與提示

如果沒有統一狀態中心，這類系統會非常容易失控。

***

### AppStateStore 的真正作用

可以把它理解為 Claude Code **互動層的匯流排**：

🖥️
上層 UI
<- 讀取狀態⟷💾
AppStateStore
統一狀態中心⟷🔧
底層工具
-> 寫入狀態

它不是一個被動資料容器，而是終端應用程式穩定運轉的**基礎設施**。

***

### 為什麼 Claude Code 能做複雜交互

高級交互能力本質上都依賴統一狀態管理：

同時存在多個任務，前後台切換REPL 裡顯示不同面板遠端會話連線狀態即時更新MCP 資源、工具、命令動態變化

***

### 一句話總結

> AppStateStore 傳達出一個重要訊號：Claude Code 不是「命令列裡的模型呼叫器」，而是一個**狀態複雜、互動豐富、長期駐留的終端應用程式**。理解這點，很多看似「為什麼這裡這麼重」的設計就都說得通了。
