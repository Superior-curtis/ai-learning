# 什麼是 AI Agent？從定義到代理迴圈

> 📅 2026-08-01 · 核心概念
> 用一句話與一個迴圈，把 AI Agent 講清楚，並說明它與一般聊天機器人的本質差異。

---

## 什麼是 AI Agent？

AI Agent（AI 代理）是一個能**自主完成目標**的系統：它不只是回答你的問題，而是透過「理解 → 計畫 → 行動 → 檢視結果」的反覆循環，一步步把任務做完。

> 一句話記住它：聊天機器人負責回答，Agent 負責完成。

一個 Agent 通常由三個部分組成：

* **模型（Model）**：負責判斷與決策的大腦，例如 Claude。
* **工具（Tools）**：讓模型能「動手」的介面，例如查天氣、算數學、讀檔案。
* **迴圈（Loop）**：把感知、思考、行動串起來的執行框架。

***

## 一個迴圈勝過千言萬語

Agent 之所以叫 Agent，關鍵在於它擁有一個**代理迴圈（agentic loop）**。最常見的抽象是四步：

```
感知 (Perceive) → 思考 (Think) → 行動 (Act) → 觀察 (Observe)
        ↑                                         │
        └──────────── 回到迴圈 ────────────────────┘
```

### 1. 感知（Perceive）

Agent 先取得目前的環境狀態。在對話場景中，這是使用者的輸入；在更複雜的場景中，這可能是網頁內容、檔案、感測器資料。

### 2. 思考（Think）

模型根據目前的狀態與目標，決定「接下來要做什麼」。它可能生成一段計畫、判斷還缺哪些資訊，或決定呼叫哪個工具。

### 3. 行動（Act）

Agent 執行一個具體動作：呼叫一個工具、發出一個請求、或直接給出最終答案。

### 4. 觀察（Observe）

Agent 接收行動的結果，把新的資訊記進上下文，然後回到感知步驟，繼續下一圈。

***

## 代理迴圈的骨架

用虛擬碼表達，迴圈長這樣：

```text
while True:
    observation = perceive(environment)   # 感知目前狀態
    plan       = think(observation)       # 決定下一步
    action     = act(plan)                # 執行動作
    result     = execute(action)          # 取得結果
    update_memory(observation, action, result)  # 記住，再轉一圈
```

迴圈什麼時候結束？當 Agent 判斷目標已達成（不再需要呼叫工具）時，就輸出最終答案並退出。

***

## Agent 與聊天機器人有什麼不同

| 面向 | 聊天機器人 | AI Agent |
|------|-----------|----------|
| 目標 | 回答問題 | 完成任務 |
| 狀態 | 通常無狀態 | 有目標與記憶 |
| 工具 | 幾乎不用 | 頻繁呼叫工具 |
| 行為 | 一次對話 | 多步驟循環 |
| 失敗處理 | 重問一次 | 換策略重試 |

聊天機器人是一問一答；Agent 是一「任務一循環」。當一個系統開始**自行決定呼叫工具、並根據結果調整下一步**時，它就跨越了從聊天到代理的界線。

***

## 實際的代理迴圈：用 Claude API 寫一個最小版

下面用 Anthropic 官方 SDK 實作一個能查天氣的工具型 Agent。這是最小的「手寫迴圈」範例：

```python
import anthropic

client = anthropic.Anthropic()

TOOLS = [
    {
        "name": "get_weather",
        "description": "查詢指定城市的目前天氣",
        "input_schema": {
            "type": "object",
            "properties": {"city": {"type": "string"}},
            "required": ["city"],
        },
    }
]

def run_tool(name: str, args: dict) -> str:
    # 真實應用裡，這裡可以接天氣 API
    return f"{args['city']}：晴，26°C"

messages = [{"role": "user", "content": "台北天氣如何？"}]

while True:
    response = client.messages.create(
        model="claude-opus-5",
        max_tokens=1024,
        tools=TOOLS,
        messages=messages,
    )
    messages.append({"role": "assistant", "content": response.content})

    # 沒有要呼叫工具 → 任務完成
    if response.stop_reason == "end_turn":
        break

    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            result = run_tool(block.name, block.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": result,
            })

    messages.append({"role": "user", "content": tool_results})

for block in messages[-1]["content"]:
    if block.type == "text":
        print(block.text)
```

注意三個重點：

* `stop_reason == "tool_use"` 表示模型想呼叫工具；`"end_turn"` 表示完成。
* 每次呼叫都要把完整的 `messages` 歷史送回模型，API 是無狀態的。
* 工具結果用 `tool_result` 回傳，並附上對應的 `tool_use_id`。

***

## 什麼時候該用 Agent？

不是所有問題都需要 Agent。判斷的四個標準：

* **複雜度**：任務需要多個步驟，且無法事先完全列成腳本。
* **價值**：成果值得更高的成本與延遲。
* **可行性**：模型本身有能力完成這類任務。
* **錯誤成本**：出錯時能被偵測與恢復。

如果四個答案都是「是」，才值得把系統做成 Agent。

***

## 重點回顧

* Agent = 模型 + 工具 + 迴圈，核心是自主完成目標。
* 代理迴圈的四步：感知 → 思考 → 行動 → 觀察。
* 聊天機器人回答問題，Agent 完成任務。
* 真實迴圈的關鍵是看懂 `stop_reason`，並把工具結果回傳給模型。
* 先用四個標準判斷「需不需要 Agent」，再動手實作。
