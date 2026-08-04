# 寫一個自己的 MCP Server

> 📅 2026-08-04 · 使用教學
> 幾十行程式碼，就能把一個自訂工具變成標準的 MCP Server。走完 TypeScript 與 Python 兩個版本、跑起來、再接上任意 Client，附錯誤處理與安全筆記。

---

前面兩篇把 MCP 的「為什麼」和「長什麼樣」都講清楚了。這一篇換個姿勢：**動手寫一個真的會跑的 MCP Server。** 目標很小——一個 Server、暴露一個工具、跑起來、能被任何支援 MCP 的 Client 呼叫。完成之後，`mcp-01` 的比喻和 `mcp-02` 的架構就都變成你手上的肌肉記憶。

會用到的概念密度不高：MCP 的 SDK 替你把 JSON-RPC、握手、stdio 傳輸全都包好了，你只需要寫「工具本身」。這比你想像的更短。

## 事前準備

你需要一個 Node.js 環境（建議 18+），然後在專案資料夾安裝 MCP 官方 SDK：

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk
```

裝完之後，Server 的全部業務就是三件事：**建立 Server、註冊一個工具、接上傳輸。**

## 程式碼：一個能跑的 Server

建立 `server.ts`，貼上下面這段：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
name: "math-server",
version: "1.0.0",
});

server.registerTool(
"add",
{
  description: "把兩個整數相加，回傳總和",
  inputSchema: {
    type: "object",
    properties: {
      a: { type: "number" },
      b: { type: "number" },
    },
    required: ["a", "b"],
  },
},
async (args) => {
  const total = args.a + args.b;
  return {
    content: [{ type: "text", text: String(total) }],
  };
}
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

就這些。沒有手寫 TCP、沒有手動解析 JSON——SDK 全包了。

## 讀懂每一行

逐段拆解上面程式碼：

* **`new McpServer({ name, version })`**：宣告這顆 Server 的身分證。Client 握手時會看到它。
* **`registerTool("add", { description, inputSchema }, handler)`**：註冊一個工具。第一個參數是名稱，第二個是「長相」（描述 + JSON Schema），第三個是真正做事的函式。
* **`inputSchema`**：這就是 `mcp-02` 講的「工具如何暴露」。它是一份標準的 JSON Schema，告訴模型「這個工具吃 `a` 和 `b` 這兩個數字參數」。模型會讀它來決定該傳什麼。
* **`content: [{ type: "text", text: ... }]`**：這是 MCP 的回傳格式——結果以標準 content 回傳，Client 能統一處理。
* **`StdioServerTransport`**：用標準輸入輸出當傳輸，`mcp-02` 的架構圖裡那一條「JSON-RPC」就靠它落地。

注意我們刻意用 `String(total)` 而不是範本字串——不是必要，只是讓 CopyCode 範例保持乾淨。你在自己的程式裡當然可以用你最習慣的寫法。

## 跑起來

因為是 TypeScript，先用 `tsx` 直接跑：

```bash
npm install -D tsx
npx tsx server.ts
```

跑起來之後，程式會「坐在那等」——不輸出一行東西。這很正常。它正透過 stdin 聽有沒有訊息進來。你可以用一組手動的 JSON-RPC 訊息測它：

```bash&#x20;/&#x20;manual&#x20;test
printf '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"tester","version":"0.1.0"}}}\
' | npx tsx server.ts
```

看到回傳的 JSON，就代表 Server 已活著、能握手。接下來的事就交給 Client 去自動完成了。

## Python 版：一樣短

不想用 TypeScript？Python 配上官方 `fastmcp` 套件，短的甚至有過之。先安裝：

```bash
pip install fastmcp
```

建立 `server.py`：

```python
from fastmcp import FastMCP

mcp = FastMCP("math-server")

@mcp.tool()
def add(a: int, b: int) -> int:
  """把兩個整數相加，回傳總和。"""
  return a + b

if __name__ == "__main__":
  mcp.run()  # 預設走 stdio 傳輸
```

Python 版更簡潔：`@mcp.tool()` 裝飾器直接把 `add` 函式變成工具，函式的型別註解當成輸入 Schema、docstring 當成描述——模型靠它們理解該傳什麼參數。執行：

```bash
python server.py
```

幾個重點對照：**TypeScript 版是「顯式宣告 Schema」，Python 版是「從型別自動推導 Schema」。** 兩者的結果一致：Server 對外暴露的都是標準的「名稱 + 描述 + 輸入 Schema」，任何 Client 都認得。

## 接上任意 Client

Server 寫好，剩下的就是讓支援 MCP 的 Client 連接它。`mcp-01` 提到 Host 會管理 Client；以 Claude Code 為例，在設定裡宣告這顆 Server，它就會在你對話時自動 `tools/list` 去探索 `add`、並在你需要時呼叫它。因為介面是標準的，**同一顆 Server 可以換到任何 MCP 相容的 Client 上**，程式碼一行不用改——這就是標準化的回報。

## 錯誤處理：讓失敗可以被看見

工具永遠會失敗。參數不合、後端服務掛掉，都可能。MCP 的規則是：**用 `isError` 標記失敗，而不是把錯誤丟掉。**

```typescript
async (args) => {
if (typeof args.a !== "number" || typeof args.b !== "number") {
  return {
    content: [{ type: "text", text: "參數 a 和 b 都必須是數字" }],
    isError: true,
  };
}
const total = args.a + args.b;
return { content: [{ type: "text", text: String(total) }] };
}
```

為什麼要這樣做？`mcp-02` 講過：`isError: true` 讓「失敗」以正常結果回傳，模型才讀得到錯誤訊息、才能修正再試一次。把錯誤吞掉或直接丟例外，模型就失去自我修復的機會。

## 安全筆記

能跑、能接，別忘了你剛做了一件大事：**你讓「模型」可以透過你的工具影響真實系統。** 這等於把一部分控制權交給它。幾條底線：

* **預設最小權限**：工具只該碰它需要碰的東西，不要給整個系統。
* **驗證所有輸入**：別信任模型傳來的參數，跟對待使用者輸入一樣嚴格。
* **認證與授權分離**：認證留在 Server 這端管理，不要讓 Agent 持有所有金鑰。
* **記錄所有呼叫**：出問題時能回溯是誰、在何時、做了什麼。

> 把這個 MCP Server 當成「公開 API」來設計：預設最小權限、驗證每個輸入、記錄每次呼叫。開放工具給模型就是把控制權交出去——security-07-agent-security 會把整個 Agent 系統的攻擊面一次講透。

## 收尾

你現在握有一顆真的會跑的 MCP Server：暴露一個標準工具、能被任意 Client 呼叫、失敗可以被模型看見。把它接到一顆真實的資料庫或外部 API，`mcp-01` 以來所有抽象就都連成線了。

這也為下一層鋪路：MCP 正是 `agents-03-mcp` 裡 Agent 接工具的那條標準。工具到齊之後，你的 Agent 就能真正「做事」了。

#### Q: 在你寫的 MCP Server 裡，工具執行失敗時該怎麼回傳？

* 直接丟出例外，讓 Client 崩潰

* 把結果包進 content，並把 isError 設為 true

* 什麼都不做，靜默失敗

* 重新跑一次 initialize 再握手

> 💡 MCP 用 isError 標記失敗，讓錯誤以正常結果回傳給模型。這樣模型才讀得到錯誤訊息、能修正後重試——這是 Agent 自我修復能力的基礎。
