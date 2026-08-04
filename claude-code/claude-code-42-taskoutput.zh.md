# TaskOutputTool：讀取任務輸出

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 TaskOutputTool 如何統一讀取各類後台任務的輸出。

---

## TaskOutputTool：讀取任務輸出

TaskOutputTool 用來讀取後台任務的輸出。它最重要的價值是**統一**：shell 任務輸出、本地 agent 輸出、遠端 agent 輸出——主執行緒不需要關心底層任務類型差異。

### 一句話總結

> TaskOutputTool 是後台任務鏈路裡的**「統一讀口」**——即使它未來可能被更通用的 Read 取代。
