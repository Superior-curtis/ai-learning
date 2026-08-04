# TaskOutputTool: Reading Task Output

> 📅 2026-07-27 · Tooling
> Learn how TaskOutputTool reads the output of various background tasks through a unified interface.

---

## TaskOutputTool: Reading Task Output

TaskOutputTool is used to read the output of background tasks. Its most important value is **uniformity**: shell task output, local agent output, remote agent output — the main thread never needs to care about differences in the underlying task type.

### One-Sentence Summary

> TaskOutputTool is the **"unified read interface"** in the background task chain — even though it may someday be replaced by a more general-purpose Read.
