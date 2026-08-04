# Claude Code's Skills System: More Than Just a Prompt

> 📅 2026-07-27 · Extensibility
> A deep dive into the Skills system and how Claude Code extends its capabilities beyond plain prompts.

---

## Claude Code's Skills System: More Than Just a Prompt

People new to Claude Code usually understand a Skill as a pre-written Prompt, an alias for a Slash Command, or a documentation file sitting in the repository. But looking at the source code, a more accurate definition is:

> **A skill module that can be loaded by the system, discovered by the model, and formally executed by SkillTool.**

Skills have their own **loading pipeline, discovery pipeline, and execution pipeline** in Claude Code.

***

### Skill Lifecycle

📄 SKILL.md
parse frontmatter→Command object
added to the command system→Available Skills list
loading phase→Model discovery
discovery phase→SkillTool execution
execution phase

***

### Step 1: Loading Skills

The core that actually handles this in the source code is `skills/loadSkillsDir.ts`:

Skills are loaded from multiple locations (in priority order):

1\. Skills pushed down by the platform or policies2. The user's own Skills3. The project's .claude/skills4. External specified directories + legacy commands directories

***

### Step 2: Discovering Skills (Expose an Overview First, Not the Full Text)

Claude Code doesn't stuff every Skill's full body into the model right away. It uses a **discovery** mechanism:

First tells the model which Skills are availableWhen the model decides to call Skill("name"), expand the full contentThis is the separation of discovery and execution

***

### Step 3: SkillTool Execution

Model calls SkillTool → finds the Skill by name⬇Validate legality → check permission rules → load the full text⬇inline
executed in the main flowfork
executed by a sub-agent⬇ Results return to the main loop

***

### inline vs fork

This is the one point in the Skills system most worth remembering:

📄 inlineSkill content is expanded directly into the main flow
Best for: lightweight flows, supplemental rules🔀 forkSpawns a new sub-agent to execute, without polluting the main flow
Best for: complex flows, tasks that need isolation

***

### Skills vs Prompt

Prompt
SkillPersistence
Valid for the current session
Can be saved and reusedSystem awareness
The system doesn't know what capability it is
Can be loaded/discovered/executedExecution method
Passive text
Formally executed by SkillTool

***

### One-Sentence Summary

> Skills are not a few Markdown prompts; they are a **skill runtime of "load first, then discover, and finally execute via SkillTool"** — turning experience from one-off instructions into reusable system capabilities.
