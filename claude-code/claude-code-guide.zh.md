# Claude Code 完全入門指南

> 📅 2026-07-27 · Claude Code / AI 工具 / 教學
> 從架構到實戰，深入理解 Claude Code 的核心運作方式

---

## Claude Code 是什麼

Claude Code 是 Anthropic 推出的終端機 AI 助手。它不是一個外掛或 IDE 擴充，而是一個直接在 terminal 裡運作的自主程式碼工具——你給它一個任務，它自己決定怎麼完成。

它與 Cursor、Copilot 最大的不同在於：Claude Code 擁有完整的 **Agent 循環（Agent Loop）**——從理解你的請求、規劃步驟、執行工具、觀察結果、到調整策略，整個過程由 AI 自主完成。

***

## 核心架構：Agent Loop

Claude Code 的工作流程可以歸納為一個不斷循環的過程。用下面的步驟導覽走一遍，你就懂了：

#### 接收輸入

你下達一個指令或提出一個問題。

#### 理解任務

模型分析當前上下文，決定需要做什麼。

#### 選擇工具

從 50+ 個工具中選出最合適的一個。

#### 執行工具

調用工具並獲得結果。

#### 觀察輸出

根據工具回傳的結果調整下一步。

#### 重複或完成

繼續循環直到任務完成，或回傳結果給你。

> 這個循環每秒都在發生，每次迭代都讓 AI 更接近你想要的結果。

***

## 50+ 工具系統

Claude Code 最強大的地方在於它的工具生態。每個工具都是一個能力單元，AI 會根據任務自動選擇和組合它們。

### 檔案操作類

* **FileReadTool** — 讀取檔案內容
* **FileWriteTool** — 寫入/建立檔案
* **FileEditTool** — 精確編輯檔案（取代特定行）
* **GlobTool** — 依模式搜尋檔案路徑
* **GrepTool** — 在檔案中搜尋內容

### 執行類

* **BashTool** — 在終端機執行任何指令。這是 Claude Code 最有威力的工具——可以編譯、安裝、測試、部署

### 網路類

* **WebFetchTool** — 抓取網頁內容並轉成 Markdown
* **WebSearchTool** — 搜尋網路（美國地區可用）

### 協作類

* **AskUserQuestionTool** — 需要釐清時主動問你問題
* **TodoWriteTool** — 建立任務清單追踪進度
* **SendMessageTool** — 子 Agent 之間的通訊

### 任務管理類

* **TaskCreateTool / TaskGetTool / TaskUpdateTool / TaskListTool / TaskStopTool / TaskOutputTool** — 完整的任務生命週期管理
* **AgentTool** — 啟動子 Agent 執行獨立任務

### 開發工具整合

* **LSPTool** — 整合語言伺服器，提供語法檢查、跳轉定義等功能
* **ListMcpResourcesTool / ReadMcpResourceTool** — MCP 伺服器資源存取

***

## Skills 系統

Skills 是 Claude Code 的可擴充功能套件。任何人可以編寫一個 Skill，讓 Claude Code 獲得特定領域的專業能力。

每個 Skill 本質上是一組 Markdown 格式的指令檔案，存放在專案的 `.claude/skills/` 目錄下。當你呼叫一個 Skill 時，它的內容會被注入到 AI 的提示詞中，讓 AI 知道如何處理特定類型的任務。

### Skills 可以做什麼

* 專案特定的程式碼規範檢查
* 部署流程自動化
* 程式碼審查標準
* 測試撰寫範本

***

## MCP 整合

MCP（Model Context Protocol）是 Anthropic 推出的開放協定，讓 AI 模型可以安全地存取外部工具和資料源。Claude Code 支援 MCP 伺服器，意味著它可以透過標準化介面連接資料庫、API、檔案系統等外部資源。

這讓 Claude Code 不只在你的專案資料夾內工作，還能與你整個開發基礎設施互動。

***

## 實際使用場景

### 1. 程式碼生成與重構

```
給一個 Express CRUD API 加上 TypeScript 型別定義
```

Claude Code 會先讀取現有檔案→分析型別需求→產生程式碼→寫入檔案。

### 2. Bug 除錯

```
為什麼這個 API 回傳 500？
```

它會先 grep 錯誤日誌→讀取相關檔案→分析可能原因→執行測試驗證→回報結果。

### 3. 專案遷移

```
把這個 Vue 2 專案遷移到 Vue 3
```

它會遍歷所有檔案→識別需要變更的部分→逐步修改→驗證編譯。

***

## 與其他工具比較

| 特性 | Claude Code | Copilot | Cursor |
|------|------------|---------|--------|
| 自主 Agent | ✅ 完整 Agent Loop | ❌ 僅補全 | ⚠️ 部分 |
| 工具數量 | 50+ | ~5 | ~20 |
| 終端機操作 | ✅ 原生 | ❌ | ❌ |
| Skills 系統 | ✅ 可擴充 | ❌ | ❌ |
| MCP 整合 | ✅ | ❌ | ❌ |
| 子 Agent | ✅ 多 Agent 編排 | ❌ | ❌ |

***

## 給新手的建議

1. **從簡單任務開始** —— 先讓 Claude Code 幫你重構一個函數或寫一個測試
2. **善用 CLAUDE.md** —— 在專案根目錄建立這個檔案，寫入專案規範、技術棧、程式碼風格，Claude Code 會自動遵守
3. **觀察它的決策過程** —— 注意它選擇了哪些工具、為什麼這樣做，這能幫助你更了解它的能力邊界
4. **使用 Plan Mode** —— 複雜任務先讓它規劃，你再審查計劃，然後執行
5. **給它明確的上下文** —— 告訴它你要改哪個檔案、什麼問題、期望的結果

***

## 檢查你的理解

#### Q: Claude Code 與 Cursor / Copilot 最大的差異是什麼？

* 它可以直接在終端機操作

* 它擁有完整的 Agent Loop，能自主完成任務

* 它的模型能力更強

* 它只能在 CLI 裡聊天

> 💡 完整的 Agent Loop——理解 → 規劃 → 執行工具 → 觀察 → 調整——讓 Claude Code 能自主完成任務，這才是它與補全型工具的關鍵差異。

***

Claude Code 不是一個寫程式工具——它是一個能理解、規劃、執行、除錯的自主開發夥伴。學會運用它的工具系統和 Agent 循環，你會發現很多過去需要花半天的任務，現在幾分鐘就能完成。
