# MCP 與 Agent 生態系：工具標準化的關鍵

> 📅 2026-08-01 · 深入研究
> Model Context Protocol 為什麼重要？拆解伺服器、用戶端、工具與資源，看它如何統一 Agent 的工具介面。

---

## 為什麼需要 MCP

每個 Agent 都要接工具：查資料庫、存取檔案、呼叫 API。過去，每接一個新服務，開發者就要為「這個服務的工具」寫一套專屬整合——prompt 格式、參數校驗、錯誤處理全都要自己來。

Model Context Protocol（MCP）是 Anthropic 於 2024 年底開源的開放協定，目的是把「Agent 如何連到工具」標準化。它解決的是同一個問題的「USB 埠版本」：

> 在 USB 出現之前，每台設備都有自己的連接埠；在 MCP 出現之前，每個 AI 工具都有自己的整合方式。MCP 就是 Agent 世界的 USB。

***

## 四個核心概念

### 用戶端與伺服器（Client / Server）

MCP 使用**用戶端－伺服器**架構：

* **MCP Server**：把某個能力（例如天氣 API、檔案系統）包成標準介面並對外提供。
* **MCP Client**：Agent 應用（例如 Claude Code、你自己的程式）連接伺服器並呼叫其能力。

一個 Agent 可以同時連接多個伺服器，一個伺服器也可以服務多個 Agent。

### 工具（Tools）

工具是讓模型「執行動作」的能力，例如 `get_weather`、`create_issue`。每個工具都有名稱、描述，以及 JSON Schema 格式的 `inputSchema`，讓模型知道該傳什麼參數。Agent 呼叫工具的方式是標準化的。

### 資源（Resources）

資源是讓模型「讀取」的資料，例如一份文件、一張圖片或資料庫內容。透過 URI（如 `file://`、`db://`）定址，模型可以把它們納入上下文。

### 提示（Prompts）

提示是伺服器提供給使用者的**可重複使用 prompt 模板**，協助使用者更快啟動某類任務。這三樣東西——工具、資源、提示——就是 MCP 暴露「能力」的三種形式。

***

## 線上的長相：JSON-RPC

MCP 建立在 JSON-RPC 2.0 之上。用戶端與伺服器先握手（initialize），再列出可用的工具：

```json
// 用戶端 → 伺服器：初始化
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{
  "protocolVersion":"2025-06-18",
  "capabilities":{},
  "clientInfo":{"name":"my-agent","version":"0.1.0"}
}}

// 用戶端 → 伺服器：列出工具
{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}

// 伺服器 → 用戶端：回應
{"jsonrpc":"2.0","id":2,"result":{"tools":[
  {"name":"get_weather","description":"查詢城市天氣",
   "inputSchema":{"type":"object","properties":{"city":{"type":"string"}}}}
]}}

// 用戶端 → 伺服器：呼叫工具
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{
  "name":"get_weather","arguments":{"city":"台北"}
}}}
```

對 Agent 開發者來說，不需要知道天氣 API 的內部細節——只要 `tools/list` 拿到 Schema、`tools/call` 拿到結果，就完成整合。

***

## 一個最小的 MCP 伺服器

用官方 TypeScript SDK，幾十行就能寫出一個伺服器：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "weather-server",
  version: "1.0.0",
});

server.registerTool(
  "get_weather",
  {
    title: "Get weather",
    description: "查詢指定城市的目前天氣",
    inputSchema: {
      type: "object",
      properties: { city: { type: "string" } },
      required: ["city"],
    },
  },
  async (args) => ({
    content: [{ type: "text", text: `${args.city}：晴，26°C` }],
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

用 stdio 作為傳輸時，你的程式可以啟動它，並透過標準輸入輸出與它交談。

***

## 為什麼 MCP 標準化了 Agent 的工具存取

* **一次整合，處處可用**：伺服器寫好後，任何支援 MCP 的用戶端（Claude Code、各種 Agent 框架）都能直接連接。
* **介面統一**：工具一律是「名稱 + 描述 + JSON Schema」，模型端不需要學習各家格式。
* **權限與授權分離**：伺服器可以獨立管理自己的認證，Agent 不需要持有所有服務的密鑰。
* **生態系統壯大**：因為標準是開放的，社群伺服器快速累積——資料庫、GitHub、Slack、瀏覽器，應有盡有。

***

## MCP 與 Anthropic 生態系

MCP 是 Anthropic 發起的標準，因此在 Claude 生態系中無所不在：

* **Claude Code**：支援透過 MCP 連接外部伺服器，讓終端代理存取資料庫與第三方服務。
* **Claude API**：提供 MCP connector，讓 Messages API 直接呼叫遠端 MCP 伺服器上的工具。
* **Claude Agent SDK / Managed Agents**：在 Agent 設定中宣告 `mcp_servers`，認證交給 vault 管理，憑證不必進到沙箱。

換句話說：你在本系列第一篇學到的「模型 + 工具 + 迴圈」，工具的接入標準就是 MCP。

***

## 安全性考量

MCP 開放也代表風險擴大：

* **工具是新的攻擊面**：讓模型呼叫工具，等於把系統操作權交給模型——工具必須驗證輸入、限制權限。
* **提示注入**：外部內容可能誘騙模型呼叫不該呼叫的工具。伺服器應做好權限控管。
* **不要相信工具回傳的未經驗證資料**：工具結果會進入模型上下文，也可能被回寫到其他地方。

> 把 MCP 伺服器當成公開 API 來設計：預設最小權限、驗證所有輸入、記錄所有呼叫。

***

## 重點回顧

* MCP 是「Agent 世界的 USB」，統一了工具接入標準。
* 四個核心概念：用戶端／伺服器、工具、資源、提示。
* 協定建立在 JSON-RPC 2.0 之上，核心方法是 `tools/list` 與 `tools/call`。
* 一次寫好伺服器，任何 MCP 用戶端都能用。
* MCP 深入 Anthropic 生態：Claude Code、Claude API、Managed Agents 都支援。
* 開放的工具面 = 更大的攻擊面，安全要當第一優先。
