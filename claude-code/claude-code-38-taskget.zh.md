# TaskGetTool：讀取任務

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 TaskGetTool 如何按 ID 查詢帶依賴關係的正式任務物件。

---

## TaskGetTool：讀取任務

TaskGetTool 的職責很純粹：**按 ID 讀取單個任務**。重要性在於讓主線程可以在複雜任務流裡隨時檢查某個任務的當前真實狀態。

### 一句話總結

> TaskGetTool 讓任務系統具備了**正式物件查詢能力**，而不是只能看列表摘要。
