# MCP 架構：Server、Client、工具與資源

> 📅 2026-08-04 · 架構總覽
> Server 到底暴露了什麼？工具是「可呼叫的動作」，資源是「可讀取的上下文」。拆解兩者的差別、MCP 與 function calling 的分工，以及背後的 JSON-RPC 傳輸。

---

你可能不需要懂 MCP 的每個細節就能用得順手；但一旦要自己寫 Server，或 debug 別人寫的整合，一張清楚的施工圖能替你省下大量冤枉路。這一篇補上 `mcp-01-what-is-mcp` 沒展開的內部構造：**Server 到底暴露了什麼、Client 怎麼跟它說話、訊息長什麼樣。**

`mcp-01` 建立了三個角色（Host / Client / Server）和「一次請求的旅程」。現在把鏡頭拉近，看 Server 裡那幾塊積木。

## 先看全貌：Server 暴露什麼

MCP 定義了四種「能力」的形狀，Server 靠它們對外發聲：

* **Tools（工具）**：可執行的動作。
* **Resources（資源）**：可讀取的上下文。
* **Prompts（提示）**：可重複使用的提示模板。
* **Roots（根）**：Server 可以存取的位置清單。

前兩種最重要，這篇就專注在工具與資源。提示你在 `agents-03-mcp` 已經碰過，不重講；根是進階議題，用到再查。

## 工具：可呼叫的動作

工具是「會做事」的能力。查天氣、建 issue、送訊息、跑 SQL——這些都是工具。每個工具對 Client 承諾三樣東西：

1. **名稱**：`get_weather`、`create_issue`。
2. **描述**：白話說明這工具做什麼、什麼時候該用它。
3. **輸入 Schema**：一份 JSON Schema，描述該傳哪些參數、各自的型別。

工具的關鍵特徵是：**它會改變世界**——至少改變 Server 那一側的世界。呼叫 `create_issue`，GitHub 上就真的多一個 issue。這代表把工具交給模型，等於把「決定權」交給模型，安全上必須非常小心；這條線我們在 `security-07-agent-security` 再拉。

## 資源：可以被讀取的上下文

資源是「可以拿來讀」的資料。一份文件、一張圖片、資料庫的一段內容——都是資源。資源有兩個特色：

* **用 URI 定址**：`file://`、`db://`、`https://` 都是合法開頭，指明「這份材料在哪裡」。
* **可以是文字或二進位**：文件通常是純文字，圖片、PDF 則以二進位形式提供。

資源的關鍵特徵是：**它本身不做事**。它只是「被放進模型上下文」的素材。模型讀完資源，接著用工具去行動，或者直接用來回答。

一句話就能區分兩者：

> 工具 = 可以呼叫的動作（會做事）；資源 = 可以被讀取的上下文（提供材料）。「模型想要資料」→ 讀資源；「模型想要改變現狀」→ 呼叫工具。

## 工具 vs 資源：對照表

| | 工具 Tools | 資源 Resources |
|---|---|---|
| 本質 | 動作、function | 資料、content |
| 作用 | 改變 Server 那一側的世界 | 提供模型上下文 |
| 如何暴露 | 名稱 + 描述 + 輸入 Schema | URI 定址 |
| 呼叫方式 | `tools/call` | `resources/read` |
| 對世界的影響 | 有副作用 | 沒有（純讀取） |

把兩欄記住，九成的架構困惑就消失了。

## MCP 與 function calling 的差別

很多人會問：各家模型不是早就有「function calling / tool use」了嗎？為什麼還需要 MCP？

答案是「層級不同」。

* **Function calling 是模型側的能力**：模型學會「判斷該呼叫哪個函式、傳什麼參數」。它決定的是「怎麼把一次工具呼叫用對」。
* **MCP 是介面側的標準**：它決定「工具怎麼被描述、怎麼被傳遞」。它管的是「工具如何到達模型」。

用一張對照表固定這個理解：

| | Function calling | MCP |
|---|---|---|
| 層級 | 模型側：怎麼把一次呼叫用對 | 介面側：工具如何被描述與傳遞 |
| 回答的問題 | 該呼叫哪個函式、傳什麼參數 | 工具怎麼到達模型 |
| 由誰定義 | 各家模型供應商 | 開放協定與社群 |
| 彼此的關係 | 被呼叫的一方 | 讓工具「能被呼叫」的標準 |

套回 `mcp-01` 的比喻：function calling 是模型「會講話」；MCP 是「電話線的插頭標準」。會講話不代表兩台電話能互通；插頭標準解決的正是「互通」這件事。兩者是互補關係，不是替代關係——MCP Server 裡宣告的工具，最後還是透過模型原生的 tool use 機制被呼叫的。

## 線上的訊息：JSON-RPC

Client 和 Server 用 JSON-RPC 2.0 對話。訊息就是「一封帶方法名和參數的 JSON」：

* Client → Server：`initialize`（握手）、`tools/list`、`tools/call`、`resources/read`
* Server → Client：對應的回應，或 `notifications`（單向通知，不需要回覆）

常用的方法就那幾個，整理成表：

| 方法 | 方向 | 作用 |
|---|---|---|
| `initialize` | Client → Server | 握手，交換版本與能力 |
| `tools/list` | Client → Server | 列出可用的工具 |
| `tools/call` | Client → Server | 呼叫一個工具 |
| `resources/read` | Client → Server | 讀取一個資源 |
| `notifications/*` | 雙向 | 單向通知，不需回覆 |

最常見的傳輸是 **stdio**：Server 就是一個「讀 stdin、寫 stdout」的程式，Client 把它當成子行程跑起來，用標準輸入輸出互通。架構長這樣：

```text
Host（Agent 程式：Claude Code / 你自己的程式）
│  一個 Host 可擁有多個 Client
▼
Client A ──JSON-RPC──▶ Server A  （資料庫：工具 + 資源）
Client B ──JSON-RPC──▶ Server B  （檔案系統：工具 + 資源）
```

一條 Client 對一條 Server；要加能力，就再加一顆 Server。

## 一次連線的生命週期

一顆 Client 連上 Server，大致會走過四個階段：

#### 握手 initialize

Client 與 Server 交換協定版本與各自的能力，確認彼此能溝通。

#### 探索 tools/list

Client 向 Server 要工具清單與資源清單，Server 交出名冊。

#### 使用 tools/call 與 resources/read

模型依需求呼叫工具或讀取資源——這是日常工作的主戰場。

#### 關閉與斷線

對話結束，Client 關閉連線；下次要再用，重新握手即可。

大部分時間都待在第三階段。握手與探索只是一次性的開場。

## 一段實際的訊息

一次 `tools/call` 在線上的長相大概如此：

```json
// Client → Server：呼叫工具
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{
  "name":"get_weather","arguments":{"city":"台北"}
}}

// Server → Client：回傳結果
{"jsonrpc":"2.0","id":3,"result":{
  "content":[{"type":"text","text":"台北：晴，26°C"}],
  "isError":false
}}
```

請注意 `isError` 這個欄位：工具執行失敗不算 JSON-RPC 層級的錯誤，而是用 `isError: true` 包在正常結果裡回傳。為什麼？因為這樣模型才「看得到失敗、讀得到錯誤訊息」，才能接著修正再試——這正是 Agent 迴圈能自我修復的基礎。

## 收尾

MCP 的架構用四塊積木就講完了：**工具是會做事的動作，資源是可以讀的材料，兩者透過 JSON-RPC 在 Host 與 Server 之間流動。** 把「動作」與「資料」分清楚，這張施工圖你就帶回家了。

下一步是動手：`mcp-03-build-your-own` 會帶你寫一個真正的 Server，讓上面這些從「圖」變成「你手上會跑的程式」。

#### Q: 模型需要一份最新合約的內容才能回答問題。它應該怎麼做？

* 用 tools/call 呼叫某個「讀合約」的工具

* 用 resources/read 讀取合約資源

* 什麼都不做，憑訓練記憶回答

* 重新呼叫 initialize 再握手一次

> 💡 讀取資料是「拿材料」而不是「改變現狀」，屬於資源的範疇，透過 resources/read 取得。工具適用於會改變世界的動作；initialize 只是握手，不提供內容。
