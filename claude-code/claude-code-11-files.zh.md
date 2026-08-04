# 檔案讀寫與編輯鏈路：Claude Code 的基本盤

> 📅 2026-07-27 · 核心機制
> 解析 Claude Code 的檔案讀寫與編輯鏈路，這是它操作程式碼的基礎能力。

---

## 檔案讀寫與編輯鏈路：Claude Code 的基本盤

再強的 AI 程式設計工具，如果不能穩定地讀取檔案、理解檔案、編輯檔案、寫回檔案，那它就只能停留在「建議型助手」的層面。Claude Code 之所以真正進入工程工作流，檔案鏈路是最底層的原因之一。

***

### 檔案工具是系統的一級公民

在 `tools.ts` 裡，檔案工具是基礎工具集的核心部分：

***

### 整體鏈路圖

1
模型判斷需要看檔案 → FileReadTool⬇2
檔案內容進入訊息歷史⬇3
模型判斷需要修改 → FileEditTool / FileWriteTool⬇4
權限檢查與 diff 預覽⬇5
寫回磁碟 → 結果摘要 → 繼續下一輪

***

### FileReadTool 的設計很能說明工程取向

這段程式碼至少傳達出 4 個資訊：

🖼️
支援多種格式
文字、圖片、PDF、Notebook✅
嚴格定義
工具行為 strict 模式🔒
唯讀標記
isReadOnly() = true📁
CWD 綁定
與當前工作目錄掛鉤

***

### 讀檔案 vs 寫檔案：不同風險等級

FileReadTool🟢 低風險：讀取
內容直接回傳，不修改任何東西FileEditTool🟡 中風險：修改已有檔案
需要權限檢查和 diff 預覽FileWriteTool🟠 高風險：新建或覆蓋檔案
整份檔案寫入，需要更嚴格的審批NotebookEditTool🔵 結構化修改
Jupyter Notebook 專用

***

### 為什麼同時保留 Edit 和 Write？

這對應兩種不同的語義：

✏️
Edit對已有內容做定向修改
→ 精準的行級修改，可產生 diff📝
Write新建或整體覆蓋檔案
→ 整份檔案寫入，無行級 diff

拆開才能讓權限模型清晰，UI diff 和預期穩定。

***

### 與 QueryEngine 的協作閉環

模型決定下一步⟷QueryEngine編排任務⟷📖 FileRead
✏️ FileEdit
📝 FileWrite檔案工具需要讀取 → 呼叫讀工具 → 內容進上下文 → 需要修改 → 呼叫編輯工具 → 結果摘要 → 繼續下一輪

***

### 互動練習：檔案工具風險配對

哪個工具適合哪個情境？情境工具查看圖片或 PDF
FileReadTool修改第 15 行的變數名稱
FileEditTool建立全新的 config 檔案
FileWriteTool修改 Jupyter 儲存格
NotebookEditTool

***

### 一句話總結

> Claude Code 把「檔案操作」從普通腳本呼叫，升級成**帶語義、帶權限、帶 UI、帶摘要的系統能力**——這也是它和玩具級 AI 程式碼助手的根本區別。
