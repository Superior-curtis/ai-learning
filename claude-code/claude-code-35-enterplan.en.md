# EnterPlanModeTool: Entering Plan Mode

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tools
> Learn how EnterPlanModeTool switches the session into a planning execution state.

---

## EnterPlanModeTool: Entering Plan Mode

EnterPlanModeTool's job is not to "let the model think a bit longer" — it switches the current session into a new execution state: **plan**. The system simultaneously changes the permission mode, session goal, and expectations for user interaction.

***

### Key Source Code

***

### Execution Pipeline

Model decides the task is too large/complex
⬇
EnterPlanModeTool
⬇
Update permission mode → plan
⬇
Enter planning state

***

### One-Sentence Summary

> EnterPlanModeTool upgrades "plan before code" from a prompting habit into a **formal state-machine transition within Claude Code's runtime**.
