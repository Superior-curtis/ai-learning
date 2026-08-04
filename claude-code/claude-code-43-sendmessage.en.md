# SendMessageTool: Agent Communication

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how SendMessageTool acts as the communication bus in multi-agent mode.

---

## SendMessageTool: Agent Communication

In a multi-agent system, plain text is not automatically visible to other teammates. If an agent wants to notify another agent, broadcast a message, or reply to an approval request, it must go through **SendMessageTool**. This is the foundational infrastructure that makes a multi-agent system genuinely work.

### One-Sentence Summary

> SendMessageTool turns **message passing** in multi-agent collaboration into a formal protocol, rather than leaving it to chance with the model's raw text output.
