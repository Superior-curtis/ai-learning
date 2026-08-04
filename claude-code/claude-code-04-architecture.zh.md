# Claude Code 原始碼架構總覽

> 📅 2026-07-27 · 架構總覽
> 俯瞰 Claude Code 的原始碼架構設計與模組劃分。

---

## Claude Code 原始碼架構總覽

如果只看原始碼目錄，很容易被大量檔案嚇到。但從主幹關係來看，Claude Code 的架構並不亂。

***

### 四大系統層級

***

### 第一層：main.tsx — 系統引導器

main.tsx 不像一般 CLI 那樣只解析參數後執行一個函數。它會在啟動階段做很多裝配工作：

**從導入清單看啟動層的職責：**

```ts
import { getSystemContext, getUserContext } from './context.js'
import { launchRepl } from './replLauncher.js'
import { getTools } from './tools.js'
import { initializeLspServerManager } from './services/lsp/manager.js'
import { initBuiltinPlugins } from './plugins/bundled/index.js'
import { initBundledSkills } from './skills/bundled/index.js'
```

這段導入清單本身很有信息量。啟動層在同時裝配：

| 導入模組 | 用途 |
|----------|------|
| getSystemContext | 準備 Git 狀態等系統上下文 |
| launchRepl | 啟動 REPL 交互介面 |
| getTools | 註冊所有可用工具 |
| initializeLspServerManager | 初始化語言伺服器 |
| initBuiltinPlugins | 載入內建插件 |
| initBundledSkills | 初始化 Skills 系統 |

**啟動層的關鍵職責：**

* 決定當前 session 是什麼形態（REPL / 非交互 / 遠端）
* 決定哪些工具和命令需要啟用
* 決定哪些插件和 Skills 需要載入

***

### 第二層：QueryEngine — 心臟

QueryEngine 是 Claude Code 的心臟。它負責把一次使用者任務轉化為連續推進的執行過程。如果沒有這一層，Claude Code 就會退化成「帶一些工具描述的大模型調用器」。

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

QueryEngine 管理的不只是「發請求給模型」，還包括：

* **中止控制** — AbortController 支援取消進行中的任務
* **權限拒絕處理** — 記錄使用者拒絕了哪些操作
* **用量統計** — 追蹤 Token 消耗
* **檔案狀態快取** — 避免重複讀取相同檔案
* **多輪訊息累積** — 維持對話上下文

這就是典型的會話級執行時期，而不是一次性請求處理器。

***

### 第三層：工具系統

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

這段程式碼直接說明：Claude Code 的能力是**明確註冊出來**的工具集合，而不是抽象想像的功能。

***

### 第四層：上下文與狀態

#### 給模型看的 — Context

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

#### 當前狀態 — AppState

AppStateStore 管理：

* REPL 狀態
* 任務佇列
* 通知系統
* 遠端連線
* MCP 伺服器
* 插件狀態

這兩層的分工很清楚：一個負責「給模型看什麼」，一個負責「界面和會話當前處於什麼狀態」。

***

### 閱讀原始碼的建議順序

步驟 1：先看主幹main.tsx — 理解啟動流程QueryEngine.ts — 理解任務循環tools.ts — 理解工具系統步驟 2：再看狀態與上下文
4\. state/AppStateStore.ts — 會話狀態管理
5\. context.ts — 上下文注入機制步驟 3：最後看擴展
6\. services/mcp — MCP 整合
7\. services/lsp — 語言伺服器
8\. plugins/ — 插件系統
9\. skills/ — Skills 系統

***

### 一句話總結

> 用 main.tsx 把配置、命令、工具、上下文和擴展能力裝配起來，再由 QueryEngine 驅動整個工程任務循環。

這四層架構疊在一起，目錄自然不會小。先抓住主幹，再看延展，效率會高很多。原始碼雖然有上千個檔案，但理解了這個主幹結構，你就不會迷路。
