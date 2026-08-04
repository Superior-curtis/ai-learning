# SkillTool：執行 Skills

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 SkillTool 如何作為技能執行器統一調度本地與 MCP skills。

---

## SkillTool：執行 Skills

很多人把 skill 當作更長 prompt 的快捷方式，但 SkillTool 明顯比「文字別名」複雜得多。它負責：**找到 skill 對應的 command、解析參數、處理本地/MCP skill、必要時 fork 子 Agent 去跑**。

***

### 關鍵特點

🔍 查找
從 commands 或 MCP skills 中找到對應 command⚡ 執行
inline 或 fork 子 Agent 執行

***

### 執行鏈路

使用者輸入 /commit 或模型觸發 Skill("name")
⬇
SkillTool → 查 command / 解析參數
⬇
inline（主執行緒）或 fork（子 Agent）
⬇
結果回到主循環

***

### MCP Skills 整合

***

### 一句話總結

> SkillTool 代表 Claude Code 走向平台化的一步：它把「技能」從普通 prompt 文字**提升成了可被系統調度、可 fork、可統計、可擴展的正式能力單元**。
