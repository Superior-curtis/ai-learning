# TaskUpdateTool：更新任務

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 TaskUpdateTool 如何作為任務系統的主寫入口推進工作流。

---

## TaskUpdateTool：更新任務

TaskUpdateTool 負責修改任務物件本身。如果說 TaskCreateTool 是建立節點，那 TaskUpdateTool 就是任務流真正推進的**主幹工具**。

### 一句話總結

> TaskUpdateTool 是 Claude Code 正式任務系統裡最像\*\*「狀態機推進器」\*\*的工具——不只是改狀態，而是整個任務物件的維護入口。
