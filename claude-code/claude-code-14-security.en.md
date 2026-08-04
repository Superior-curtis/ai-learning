# Permissions and Security: Why Claude Code Doesn't Run Wild

> 📅 2026-07-27 · Deep Dive
> Understand Claude Code's permission and security mechanisms, and see how it prevents unauthorized operations.

---

## Permissions and Security: Why Claude Code Doesn't Run Wild

Once an AI system can read and write files, execute commands, and access external resources, security is no longer a bolt-on feature — it is **part of the core feature set**.

Claude Code has clearly invested a lot in this — its security is not a patch applied after the fact, but **architecture-level security**.

***

### Permission Decision Flowchart

🧠 The model wants to call a tool⬇Is the tool visible to the current session?⬇Match allow / deny / ask rules✅ allow
Let it execute❓ ask
Pop up a permission request🚫 deny
Log the denial and feed it back

***

### Permissions Are Not a Single-Point Check, But Multi-Layer Control

Looking at the source structure, Claude Code's permission system has at least six layers:

🛂
Permission Mode
default / acceptEdits / plan✅
allow / deny / ask rules
Flexible rules set by the user🔍
Tool-level filtering
filterToolsByDenyRules🚫
Automatic denial
Avoid pop-ups in specific scenarios📋
Permission request component
The permission dialog in the UI layer⚠️
Dangerous rule stripping
Automatically excludes high-risk operations

***

### Some Tools Are Not Even Exposed to the Model First

This is a key point. The filtering logic in `tools.ts` shows that certain tools are stripped out before the model ever sees them.

📦 All tools⬇Visibility filtering
（environment / feature gates / deny rules）⬇🎯 Tools the model can see⬇ Invoke a callRuntime permission check
（execute / ask / deny）

The security strategy is not just runtime interception; it also includes: **capability exposure control**, **tool visibility control**, and **environment-level availability control**. This is far more robust than merely confirming at tool execution time.

***

### Why Does This Matter Even More for Multi-Agent Scenarios?

Once the system supports sub-agents, background tasks, and remote connections, permission issues immediately become complex:

Who can pop up a permission dialog?Who can only auto-deny?Which actions can run in the background?Which rules must take effect in advance?

***

### The Design Goal Is Not Full Automation, But "Controllable Automation"

> Claude Code's goal is not to cut the user out of the loop entirely. More precisely, what it pursues is: tasks can advance automatically, but critical actions must remain constrainable, and permission policies can flexibly adjust across different modes and contexts.

***

### One-Sentence Summary

> Claude Code doesn't run wild — not because the model is naturally cautious, but because the system does extensive design work in **tool exposure, call approval, mode control, and rule matching** — which is also why it can truly bring agent execution into engineering workflows.
