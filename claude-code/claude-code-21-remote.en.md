# Remote Sessions and Bridge Capabilities: A Local Terminal Doesn't Mean Local Execution

> 📅 2026-07-27 · Extensibility
> Learn about Claude Code's remote session and bridge capabilities for operating remote environments from a local terminal.

---

## Remote Sessions and Bridge Capabilities: A Local Terminal Doesn't Mean Local Execution

By most people's intuition, Claude Code is a local terminal tool. But the source code makes it clear that it already has quite a bit of remote capability built in: **remote session, direct connect, bridge/remote-control, SSH-related flows**. Its UI, execution location, and session location can be **separated**.

***

### Evidence at the Entry Layer

`main.tsx` directly has these imports:

Remote capability isn't something a plugin added later; it's **an execution mode formally considered at the entry layer**.

***

### Remote State in AppState

Looking at this set of fields, two things are basically certain:

1. Remote capability is no longer a one-off request but a **long-lived connection state**
2. The system has to handle the full lifecycle of connection, active state, reconnection, bridging, environment IDs, and more

***

### Remote Session vs Bridge

🌍 Remote SessionThe session runs remotely
The remote side holds the execution rights🔗 Bridge / Remote ControlThe local REPL exposes a channel to the remote
The local UI controls the remote agent

Both fall under the category of "**local UI separated from execution location**", but their semantics aren't quite the same.

***

### Why Remote?

📦
Run in a remote container🖥️
Run in a server environment🎮
Control the remote from a local UI🔀
Hand the task to the remote to continue

***

### One-Sentence Summary

> Claude Code is expanding from a "local terminal tool" into a **hybrid system of "local UI + multiple execution environments"** — this means it isn't just a personal development toy, but can enter more complex, real-world environments.
