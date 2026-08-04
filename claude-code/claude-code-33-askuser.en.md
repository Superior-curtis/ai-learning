# AskUserQuestionTool: Asking the User

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tools
> Learn how AskUserQuestionTool turns asking questions into a formal interaction protocol.

---

## AskUserQuestionTool: Asking the User

When the model needs more information mid-task, AskUserQuestionTool turns "asking a question" into a formal, structured, interactive system capability rather than a casually typed sentence of natural language.

***

### Question Structure

***

### Execution Flow

Model hits ambiguity
⬇
AskUserQuestionTool → structured questions + options
⬇
Frontend renders → user selects → answers flow back
⬇
Model continues

***

### Boundary with ExitPlanMode

❓ AskUserQuestionTool
Still missing info, keeps asking✅ ExitPlanModeTool
Plan is done, request approval

***

### One-Sentence Summary

> Without this tool, the model facing ambiguity could only guess or drop an informal question into natural language. Claude Code's approach: make asking a **formal interaction protocol**, so results are structured, options are explainable, and previews can be attached for comparison.
