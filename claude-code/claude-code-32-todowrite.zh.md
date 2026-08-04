# TodoWriteTool：待辦清單

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 TodoWriteTool 如何在會話內將計畫外顯化並推動驗證。

---

## TodoWriteTool：待辦清單

TodoWriteTool 讓模型在當前會話裡把工作拆成一串可見的 todo。它本質上是一個 **AppState 寫入工具**——輕量、會話內、快速追蹤。

***

### 關鍵設計

寫入 AppState → UI 顯示清單 → 模型按清單推進 → 完成時若無驗證步驟則推動驗證。

***

### 與 Task 系統的區別

📋 TodoWriteTool
輕量、會話內、快速追蹤📁 TaskCreateTool
結構化正式任務系統

***

### 一句話總結

> 用最低成本把模型目前計畫**顯式化**，並把「做完就清單消失、複雜任務別忘驗證」這種流程約束接進會話狀態。
