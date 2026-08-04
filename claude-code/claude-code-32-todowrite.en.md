# TodoWriteTool: Todo List

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tools
> Learn how TodoWriteTool makes plans visible within a session and drives verification.

---

## TodoWriteTool: Todo List

TodoWriteTool lets the model break work into a visible list of todos within the current session. At heart it is an **AppState write tool** — lightweight, session-scoped, fast to track.

***

### Key Design

Write to AppState → UI shows the list → the model works through the list → when done, if no verification step exists, it nudges verification.

***

### Difference from the Task System

📋 TodoWriteTool
Lightweight, session-scoped, fast to track📁 TaskCreateTool
Structured, formal task system

***

### One-Sentence Summary

> At minimal cost, it makes the model's current plan **explicit**, and wires workflow constraints — the list disappears when the work is done, and complex tasks should not forget verification — into the session state.
