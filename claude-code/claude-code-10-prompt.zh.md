# Claude Code 的提示詞工程：六層分級提示系統

> 📅 2026-07-27 · 核心機制
> 拆解 Claude Code 的六層分級提示系統，了解模型如何接收精確指令。

---

## Claude Code 的提示詞工程：六層分級提示系統

不要把 Claude Code 的 prompt 理解成「一大段總提示詞」。原始碼裡真正存在的，不是一條 prompt，而是**一整套分層裝配的提示詞系統**。

Claude Code 至少同時存在這幾類 prompt：

📜
會話級 System Prompt🔧
運行時動態追加 Section🛠️
工具級 Prompt📋
專項子系統 Prompt👥
多 Agent 附加 Prompt🔄
工具結果壓縮 Prompt

***

### 提示詞系統總覽圖

constants/prompts.ts → 基礎 System Prompt
定義身份：互動式 Agent + 軟體工程任務域⬇動態 Section 裝配Session Guidance
Memory
Env Info
Language
Output Style
MCP Instructions⬇使用者自訂
&#x20;→&#x20;
\--system-prompt（取代） / --append-system-prompt（追加）⬇Teammate Addendum
&#x20;+&#x20;
工具級 Prompt⬇QueryEngine 主循環 → 最終發送給模型

***

### 第一層：主會話的 System Prompt

```text
function getSimpleIntroSection(outputStyleConfig) {
  return `
You are an interactive agent that helps users ...
with software engineering tasks.
Use the instructions below and the tools
available to you to assist the user.

IMPORTANT: You must NEVER generate or guess
URLs for the user unless you are confident
that the URLs are for helping the user
with programming.`
}
```

這段 prompt 的作用：

1. 先確定**身份**：互動式 Agent
2. 再確定**任務域**：軟體工程
3. 再強調**工具可用**
4. 最後給出**安全邊界**：不憑空產生 URL

***

### 第二層：System Prompt 是動態分段拼裝的

Claude Code 在做的不是把一段固定 system prompt 塞給模型，而是**根據當前會話狀態，拼裝出當前這一輪最合適的 system prompt**。

***

### 第三層：使用者可以取代或追加 System Prompt

R
\--system-prompt完全取代預設的 system prompt
我要完全換掉預設行為A
\--append-system-prompt在預設 prompt後面追加額外內容
保留預設，只在結尾增加約束

***

### 第四層：Teammate 模式的自動追加 Prompt

這不是「知識型提示詞」，而是**角色切換提示詞**——讓模型理解自己當前處於團隊 Agent 身份，溝通方式不同了。

***

### 第五層：工具本身也帶 Prompt

工具 prompt 不是告訴模型「工具有存在」，而是指導**何時用、怎麼用、不要怎麼用**：

📖 FileReadTool prompt• file_path 必須是絕對路徑
• 預設讀取 2000 行
• 不能讀目錄（用 Bash ls）
• 使用者給截圖路徑必須用此工具⚡ BashTool prompt有專用工具時不要用 Bash
• 讀檔 → FileReadTool
• 搜尋 → GrepTool
• 匹配 → GlobTool
控制模型的工具路由策略

***

### 第六層：專項子系統也會用 Prompt

/init
專案 onboarding prompt：總結結構、運行方式、規範工具結果總結
把工具呼叫結果壓縮成更短總結Prompt Suggestion
幫助後續輪次減少冗餘上下文記憶篩選與生成
旁路流程中持續工作

***

### 互動練習：這個 prompt 來自哪一層？

Prompt 來源配對Prompt 內容
層級「你是一個互動式 Agent，負責軟體工程任務」
第一層：主 System Prompt「file_path 必須是絕對路徑」
第五層：工具級 Prompt「你正在以團隊 Agent 身分運行，使用 SendMessage 溝通」
第四層：Teammate 模式「當前語言設定：繁體中文」
第二層：動態 Section

***

### 一句話總結

> Claude Code 的提示詞工程，本質上不是「寫一個超強 system prompt」——它是一個**提示詞執行時期**，把 prompt 分層、模組化，並與工具、角色、子系統、執行時期狀態結合起來。
