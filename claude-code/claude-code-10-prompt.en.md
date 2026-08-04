# Claude Code's Prompt Engineering: A Six-Tier Prompt System

> 📅 2026-07-27 · Core Mechanics
> Break down Claude Code's six-tier prompt system and see how the model receives precise instructions.

---

## Claude Code's Prompt Engineering: A Six-Tier Prompt System

Don't think of Claude Code's prompt as "one big master prompt." What actually exists in the source isn't a single prompt — it's a **complete, layered prompt assembly system**.

Claude Code has at least these kinds of prompts coexisting:

📜
Session-level System Prompt🔧
Runtime-dynamic appended Sections🛠️
Tool-level Prompt📋
Specialized subsystem Prompt👥
Multi-agent appended Prompt🔄
Tool-result compression Prompt

***

### Overview Diagram of the Prompt System

constants/prompts.ts → the base System Prompt
Defines identity: interactive agent + software engineering task domain⬇Dynamic Section assemblySession Guidance
Memory
Env Info
Language
Output Style
MCP Instructions⬇User customization
&#x20;→&#x20;
\--system-prompt (replace) / --append-system-prompt (append)⬇Teammate Addendum
&#x20;\+&#x20;
Tool-level Prompt⬇QueryEngine main loop → finally sent to the model

***

### Layer 1: The Main Conversation's System Prompt

```text
function getSimpleIntroSection(outputStyleConfig) {
  return `
You are an interactive agent that helps users ...
with software engineering tasks.
Use the instructions below and the tools
available to you to assist the user.

IMPORTANT: You must NEVER generate or guess
URLs for the user unless you are confident
that the URLs are for helping the user
with programming.`
}
```

What this prompt does:

1. First establishes **identity**: an interactive agent
2. Then establishes **task domain**: software engineering
3. Then emphasizes **tools are available**
4. Finally gives a **safety boundary**: don't fabricate URLs

***

### Layer 2: The System Prompt Is Assembled Dynamically in Sections

What Claude Code does isn't shoving a fixed system prompt at the model — it's **assembling the system prompt that's most appropriate for the current turn, based on the current session state**.

***

### Layer 3: Users Can Replace or Append the System Prompt

R
\--system-promptFully replaces the default system prompt
I want to completely swap out the default behaviorA
\--append-system-promptAppends extra content after the default prompt
Keep the default, only add constraints at the end

***

### Layer 4: The Auto-Appended Prompt in Teammate Mode

This isn't a "knowledge prompt" — it's a **role-switching prompt**, letting the model understand it's currently in a team agent identity and that the way to communicate has changed.

***

### Layer 5: Tools Carry Their Own Prompts

Tool prompts don't tell the model "the tool exists" — they guide **when to use it, how to use it, and what not to do**:

📖 FileReadTool prompt• file\_path must be an absolute path
• reads 2000 lines by default
• can't read directories (use Bash ls)
• if the user gives a screenshot path, this tool must be used⚡ BashTool promptDon't use Bash when a dedicated tool exists
• read files → FileReadTool
• search → GrepTool
• match → GlobTool
Controls the model's tool routing strategy

***

### Layer 6: Specialized Subsystems Also Use Prompts

/init
Project onboarding prompt: summarizes structure, how it runs, and conventionsTool-result summarization
Compresses tool call results into shorter summariesPrompt Suggestion
Helps later turns reduce redundant contextMemory filtering and generation
Keeps working in the background flow

***

### Interactive Exercise: Which Layer Does This Prompt Come From?

Prompt source matchingPrompt content
Layer"You are an interactive agent responsible for software engineering tasks"
Layer 1: Main System Prompt"file\_path must be an absolute path"
Layer 5: Tool-level Prompt"You are running as a team agent; use SendMessage to communicate"
Layer 4: Teammate mode"Current language setting: Traditional Chinese"
Layer 2: Dynamic Section

***

### One-Sentence Summary

> Claude Code's prompt engineering isn't essentially "writing one super system prompt" — it's a **prompt runtime** that layers and modularizes prompts, and combines them with tools, roles, subsystems, and runtime state.
