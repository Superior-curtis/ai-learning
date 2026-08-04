# SkillTool: Running Skills

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tools
> Learn how SkillTool acts as a skill executor that dispatches both local and MCP skills.

---

## SkillTool: Running Skills

Many people treat a skill as a shortcut for a longer prompt, but SkillTool is clearly much more than a "text alias." It is responsible for: **finding the command behind a skill, parsing arguments, handling local/MCP skills, and forking a subagent to run it when necessary**.

***

### Key Features

🔍 Lookup
Find the matching command from commands or MCP skills⚡ Execution
Run inline or fork a subagent to execute

***

### Execution Pipeline

User types /commit or the model triggers Skill("name")
⬇
SkillTool → look up command / parse arguments
⬇
inline (main thread) or fork (subagent)
⬇
Result returns to the main loop

***

### MCP Skills Integration

***

### One-Sentence Summary

> SkillTool marks Claude Code's step toward becoming a platform: it elevates "skills" from ordinary prompt text into **formal capability units that the system can dispatch, fork, count, and extend**.
