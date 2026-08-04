# Claude Code Source Architecture Overview

> 📅 2026-07-27 · Architecture
> A bird's-eye view of Claude Code's source architecture design and module boundaries.

---

## Claude Code Source Architecture Overview

If you only look at the source directory, it's easy to be intimidated by the sheer number of files. But looking at the backbone relationships, Claude Code's architecture isn't messy at all.

***

### Four system layers

***

### Layer 1: main.tsx — the system bootstrapper

main.tsx isn't like a typical CLI that just parses arguments and runs a single function. It does a lot of assembly work at startup:

**From the import list you can see the startup layer's responsibilities:**

```ts
import { getSystemContext, getUserContext } from './context.js'
import { launchRepl } from './replLauncher.js'
import { getTools } from './tools.js'
import { initializeLspServerManager } from './services/lsp/manager.js'
import { initBuiltinPlugins } from './plugins/bundled/index.js'
import { initBundledSkills } from './skills/bundled/index.js'
```

This import list is informative by itself. The startup layer assembles all of these at once:

| Imported module | Purpose |
|----------|------|
| getSystemContext | Prepares system context such as Git state |
| launchRepl | Launches the REPL interactive interface |
| getTools | Registers all available tools |
| initializeLspServerManager | Initializes the language server |
| initBuiltinPlugins | Loads built-in plugins |
| initBundledSkills | Initializes the Skills system |

**Key responsibilities of the startup layer:**

* Decides what form the current session takes (REPL / non-interactive / remote)
* Decides which tools and commands need to be enabled
* Decides which plugins and Skills need to be loaded

***

### Layer 2: QueryEngine — the heart

QueryEngine is the heart of Claude Code. It's responsible for turning a single user task into a continuously advancing execution process. Without this layer, Claude Code would degrade into "a large-model caller with a few tool descriptions."

```ts
export class QueryEngine {
  private config: QueryEngineConfig
  private mutableMessages: Message[]
  private abortController: AbortController
  private permissionDenials: SDKPermissionDenial[]
  private totalUsage: NonNullableUsage
  private readFileState: FileStateCache
}
```

QueryEngine manages more than just "sending requests to the model" — it also includes:

* **Abort control** — AbortController supports canceling in-flight tasks
* **Permission-denial handling** — records which operations the user rejected
* **Usage tracking** — tracks token consumption
* **File-state caching** — avoids re-reading the same file
* **Multi-turn message accumulation** — maintains conversation context

This is a typical session-level runtime, not a one-shot request handler.

***

### Layer 3: the tool system

```ts
export function getAllBaseTools(): Tools {
  return [
    AgentTool, TaskOutputTool, BashTool,
    ...(hasEmbeddedSearchTools() ? [] : [GlobTool, GrepTool]),
    FileReadTool, FileEditTool, FileWriteTool,
    WebFetchTool, TodoWriteTool, WebSearchTool,
    AskUserQuestionTool, SkillTool, EnterPlanModeTool,
    ...(isEnvTruthy(process.env.ENABLE_LSP_TOOL) ? [LSPTool] : []),
    ListMcpResourcesTool, ReadMcpResourceTool,
  ]
}
```

This code shows it directly: Claude Code's capabilities are a collection of tools **explicitly registered** in code, not some abstractly imagined features.

***

### Layer 4: context and state

#### What the model sees — Context

```ts
export const getSystemContext = memoize(
  async () => {
    const gitStatus =
      isEnvTruthy(process.env.CLAUDE_CODE_REMOTE)
      || !shouldIncludeGitInstructions()
        ? null : await getGitStatus()
    return {
      ...(gitStatus && { gitStatus }),
    }
  }
)
```

#### Current state — AppState

AppStateStore manages:

* REPL state
* Task queue
* Notification system
* Remote connections
* MCP servers
* Plugin state

The division of labor between these two layers is clear: one handles "what the model sees," and the other handles "what state the UI and session are currently in."

***

### Suggested reading order for the source code

Step 1: Start with the backbonemain.tsx — understand the startup flowQueryEngine.ts — understand the task looptools.ts — understand the tool systemStep 2: Then state and context
4\. state/AppStateStore.ts — session state management
5\. context.ts — context injection mechanismStep 3: Finally, the extensions
6\. services/mcp — MCP integration
7\. services/lsp — language server
8\. plugins/ — plugin system
9\. skills/ — Skills system

***

### One-sentence summary

> main.tsx assembles configuration, commands, tools, context, and extension capabilities, and QueryEngine drives the entire engineering task loop.

These four architecture layers stack together, so naturally the directory isn't small. Grab the backbone first, then look at the extensions, and you'll be far more efficient. Although the source code has over a thousand files, once you understand this backbone structure, you won't get lost.
