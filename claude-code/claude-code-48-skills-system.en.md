# Claude Code's Skills System: More Than Just a Prompt

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Extensions
> From loading and discovery to execution — breaking down a Skill's full lifecycle and how it fundamentally differs from an ordinary prompt.

---

## Claude Code's Skills System: More Than Just a Prompt

Many people think of a Skill as "a prompt given to the model". But a Skill is not an ordinary file — it's a skill module with a **full lifecycle**: its own loading pipeline, discovery pipeline, and execution pipeline, formally executed by `SkillTool`.

***

### What's Inside a Skill File

The carrier format is `SKILL.md`, split into two layers:

***

### The Key Design: Discovery and Execution Are Separate

The model does **not** get the full body of every Skill in each turn. The real flow is:

1. **Load** (`loadSkillsDir.ts`) — scan `SKILL.md` files from multiple directories, parse the frontmatter, and convert them into internal `Command` objects
2. **Discover** (`utils/messages.ts`) — only expose the Skill's name, brief description, and applicable scenarios; the model knows which skills are available
3. **Execute** (`SkillTool.ts`) — the system expands the full content only after the model decides to invoke a Skill

LoadloadSkillsDir.ts → parseSkillFrontmatterFields → createSkillCommandDiscoverskill\_discovery / discoveredSkillNames / invoked\_skills — the three-part system promptExecuteSkillTool: lookup → validate → permission → expand → inline / fork

Two benefits: it saves context; and it lets the model enter a Skill's detailed flow only when it actually needs to. In essence, it's **exposing an overview of capabilities first, then expanding the details on demand**.

***

### Inline or Fork, What's the Difference

When `SkillTool` actually executes, it must choose between two modes:

* **inline** — expands within the same main loop, sharing the current context; cheaper, smaller blast radius
* **fork** — spawns a separate thread of execution, isolating context; suited for skills that are highly independent or may rewrite state

This is a tradeoff about "whether the Skill should see the full context of the parent conversation", not a performance micro-optimization.

***

### Why Skills Are a Key Step Toward a Platform

If you only have prompts, all capabilities are crammed into one system prompt, and the model has no way to tell apart "team conventions", "safety rules", and "specific scenario workflows". Skills solidify these into named, discoverable, executable modules — official built-in experience, team-shared experience, personal workflows, and project rules can all plug in through the same mechanism. **Skills aren't add-ons; they're first-class members of the command system.**

***

### One-Sentence Summary

> A Skill isn't writing another prompt — it's turning "procedural knowledge" into a **first-class module with a three-stage lifecycle: load, discover, execute** — and the separation of discovery from execution is the key to both saving context and staying precise.
