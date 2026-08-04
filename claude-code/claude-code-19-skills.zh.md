# Claude Code 的 Skills 系統：不只是 Prompt

> 📅 2026-07-27 · 擴展能力
> 深入 Skills 系統，了解 Claude Code 如何超越單純的提示詞來擴展能力。

---

## Claude Code 的 Skills 系統：不只是 Prompt

第一次接觸 Claude Code 的人，通常會把 Skill 理解成一段寫好的 Prompt、一個 Slash Command 別名、或一份放在倉庫裡的說明文件。但從原始碼看，更準確的定義是：

> **一套可以被系統載入、被模型發現、再由 SkillTool 正式執行的技能模組。**

Skill 在 Claude Code 裡有自己的**載入鏈路、發現鏈路和執行鏈路**。

***

### Skill 生命週期

📄 SKILL.md
解析 frontmatter→Command 物件
加入命令系統→可用 Skills 列表
載入階段→模型發現
發現階段→SkillTool 執行
執行階段

***

### 第一步：載入 Skills

原始碼裡真正負責這件事的核心是 `skills/loadSkillsDir.ts`：

Skills 從多個位置載入（依優先級）：

1\. 平台或策略下發的 Skills2. 使用者自己的 Skills3. 專案的 .claude/skills4. 外部指定目錄 + 舊版 commands 目錄

***

### 第二步：發現 Skills（先暴露概覽，不展開全文）

Claude Code 不會一上來把所有 Skill 正文都塞給模型。它用 **discovery** 機制：

先告訴模型「有哪些 Skill 可用」等模型決定呼叫 Skill("name")，再展開完整內容這就是「發現」和「執行」分離

***

### 第三步：SkillTool 執行

模型呼叫 SkillTool → 根據名稱找到 Skill⬇校驗合法性 → 檢查權限規則 → 載入全文⬇inline
主流程執行fork
子 Agent 執行⬇ 結果回到主循環

***

### inline vs fork

這是 Skills 系統裡最值得記住的一個點：

📄 inlineSkill 內容直接展開到主流程
適合：輕量流程、補充規則🔀 fork開新子 Agent 執行，不污染主流程
適合：複雜流程、需隔離的任務

***

### Skills vs Prompt

Prompt
Skill持久性
本次會話有效
可保存可複用系統認知
不知道是什麼能力
可被載入/發現/執行執行方式
被動文字
SkillTool 正式執行

***

### 一句話總結

> Skills 不是幾篇 Markdown 提示詞，而是一套\*\*「先載入、再發現、最後由 SkillTool 執行」的技能執行時期\*\*——把經驗從一次性說明變成可複用的系統能力。
