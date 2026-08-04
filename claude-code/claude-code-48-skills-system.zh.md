# Claude Code 的 Skills 系統：不只是 Prompt

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 擴展能力
> 從載入、發現到執行——拆解 Skill 完整生命週期，與它和普通 Prompt 的根本差異。

---

## Claude Code 的 Skills 系統：不只是 Prompt

很多人把 Skill 想成「一份給模型的提示詞」。但 Skill 不是普通文件，而是一套具備**完整生命週期**的技能模組：有自己的載入鏈路、發現鏈路、執行鏈路，並被 `SkillTool` 正式執行。

***

### 一個 Skill 檔案裡寫了什麼

承載形式是 `SKILL.md`，分兩層：

***

### 最關鍵設計：發現與執行分離

模型每一輪**不會**拿到全部 Skill 的完整正文。真正的流程是：

1. **載入** (`loadSkillsDir.ts`)——從多個目錄掃描 `SKILL.md`、解析 frontmatter、轉成內部 `Command` 物件
2. **發現** (`utils/messages.ts`)——只暴露 Skill 名稱、簡介、適用場景；模型知道有哪些技能可用
3. **執行** (`SkillTool.ts`)——模型決定呼叫某個 Skill 後系統才展開完整內容

載入loadSkillsDir.ts → parseSkillFrontmatterFields → createSkillCommand發現skill\_discovery / discoveredSkillNames / invoked\_skills 三段系統提示執行SkillTool：查表 → 校驗 → 權限 → 展開 → inline / fork

兩個好處：節省上下文；讓模型只在真正需要時才進入某個 Skill 的詳細流程。本質是**先暴露能力概覽，再按需展開細節**。

***

### inline 還是 fork，差在哪

`SkillTool` 真正執行時要在兩種模式裡二選一：

* **inline**——在同主循環內展開，共用當前上下文，省成本、波及面小
* **fork**——另起一條執行緒跑，隔離上下文，適合獨立性強、可能改寫狀態的技能

這是「要不要讓 Skill 看到母會話的完整脈絡」的取捨，不是效能微調。

***

### 為什麼 Skill 是平台化的關鍵一步

如果只有 prompt，能力全擠在一次系統提示裡，模型無從分辨「團隊約定」「安全規範」「特定場景流程」。Skill 把這些固化成可命名、可發現、可執行的模組——官方內建經驗、團隊共享經驗、個人工作流、專案規則都能以同一套機制介入。**Skill 不是附件，是命令系統裡的一等成員。**

***

### 一句話總結

> Skill 不是多寫一段 prompt，而是把「流程知識」做成**有載入、有發現、有執行三段式生命週期**的一等模組——發現與執行分離是它省上下文又保持精準的關鍵。
