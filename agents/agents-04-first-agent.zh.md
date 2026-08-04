# 動手打造你的第一個 Agent：逐步教學

> 📅 2026-08-01 · 使用教學
> 從挑選框架、定義工具、跑起迴圈，到加上記憶與錯誤處理，一步步實作你的第一個 AI Agent。

---

## 在開始之前

這篇教學會帶你實作一個「會查天氣、會記筆記、不怕出錯」的 Agent。你只需要一個 Claude API key（或任何支援 tool use 的模型）、Python 3.10+，以及 `pip install anthropic`。

想直接看完整程式碼？跳到「完整範例」；想理解每一步，就沿著 Step 依序往下走。

***

## Step 1：挑選你的框架

| 方式 | 適合 | 你寫多少 |
|------|------|---------|
| 手寫迴圈 | 想徹底理解原理 | 全部 |
| Claude Agent SDK | 想要開箱即用的工具與檔案存取 | 很少 |
| LangGraph | 想要可視化的圖狀流程 | 中等 |
| CrewAI | 想要多角色分工 | 中等 |

> 新手建議先手寫一次迴圈。迴圈只有幾十行，自己寫過一遍，之後用任何框架都懂它在做什麼。

***

## Step 2：定義你的工具

工具 = 名稱 + 描述 + 輸入的 JSON Schema。描述很重要——模型靠它決定「什麼時候該呼叫這個工具」。

```python
TOOLS = [
    {
        "name": "get_weather",
        "description": "查詢指定城市的目前天氣",
        "input_schema": {
            "type": "object",
            "properties": {"city": {"type": "string", "description": "城市名稱"}},
            "required": ["city"],
        },
    },
    {
        "name": "save_note",
        "description": "把一段筆記存到記事本",
        "input_schema": {
            "type": "object",
            "properties": {"text": {"type": "string", "description": "筆記內容"}},
            "required": ["text"],
        },
    },
]

NOTES = []

def run_tool(name: str, args: dict) -> str:
    if name == "get_weather":
        return f"{args['city']}：晴，26°C"
    if name == "save_note":
        NOTES.append(args["text"])
        return f"已儲存筆記（目前共 {len(NOTES)} 則）"
    return f"未知工具：{name}"
```

每個工具定義搭配一個執行函式：函式接收模型傳來的參數，回傳字串結果。

***

## Step 3：跑起代理迴圈

核心迴圈只有四件事：呼叫模型 → 檢查 `stop_reason` → 執行工具 → 把結果回傳。

```python
import anthropic

client = anthropic.Anthropic()
MODEL = "claude-opus-5"

def agent_loop(user_input: str, messages: list[dict]) -> list[dict]:
    messages.append({"role": "user", "content": user_input})
    while True:
        response = client.messages.create(
            model=MODEL, max_tokens=2048, tools=TOOLS, messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})
        if response.stop_reason != "tool_use":
            return messages  # 模型不再呼叫工具 → 完成

        results = []
        for block in response.content:
            if block.type == "tool_use":
                text = run_tool(block.name, block.input)
                results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": text,
                })
        messages.append({"role": "user", "content": results})
```

`messages` 是累積的對話歷史，每次呼叫都完整送回——這就是 Agent「記憶」的最小形式。

***

## Step 4：加上記憶

想讓 Agent 跨對話記得事情，最簡單的做法是把歷史存起來，下次從歷史恢復：

```python
import json

def save_session(messages: list[dict], path: str = "session.json") -> None:
    with open(path, "w", encoding="utf-8") as f:
        json.dump(messages, f, ensure_ascii=False)

def load_session(path: str = "session.json") -> list[dict]:
    try:
        with open(path, encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        return []

messages = load_session()
messages = agent_loop("記得我明天要帶傘。", messages)
save_session(messages)
```

> 真正的長效記憶還可以有更多層次：摘要過期對話、把重要事實寫進筆記工具、或使用向量資料庫。先從「存檔」開始，就足以理解原理。

***

## Step 5：處理錯誤

真實世界會出錯，一個健壯的 Agent 要處理兩類錯誤。

第一類，**工具執行失敗**——用 `is_error` 標記回傳，讓模型知道並換個策略：

```python
for block in response.content:
    if block.type == "tool_use":
        try:
            text = run_tool(block.name, block.input)
            results.append({"type": "tool_result", "tool_use_id": block.id, "content": text})
        except Exception as e:
            results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": f"工具失敗：{e}",
                "is_error": True,
            })
```

第二類，**API 呼叫失敗**——用 SDK 的型別例外處理。`RateLimitError`（429）與 `APIConnectionError`（網路問題）可以稍後重試；其他 `APIStatusError` 則依狀態碼決定。完整做法請看下方「完整範例」中的 `agent_loop`。

***

## 完整範例

把上面的片段組起來，就是一個完整可執行的 Agent：

```python
import time
import anthropic
client = anthropic.Anthropic()
MODEL = "claude-opus-5"
NOTES = []

TOOLS = [
    {"name": "get_weather", "description": "查詢指定城市的目前天氣",
     "input_schema": {"type": "object",
                      "properties": {"city": {"type": "string"}},
                      "required": ["city"]}},
    {"name": "save_note", "description": "把一段筆記存到記事本",
     "input_schema": {"type": "object",
                      "properties": {"text": {"type": "string"}},
                      "required": ["text"]}},
]

def run_tool(name: str, args: dict) -> str:
    if name == "get_weather":
        return f"{args['city']}：晴，26°C"
    if name == "save_note":
        NOTES.append(args["text"])
        return f"已儲存筆記（目前共 {len(NOTES)} 則）"
    return f"未知工具：{name}"

def agent_loop(user_input: str, messages: list[dict]) -> list[dict]:
    messages.append({"role": "user", "content": user_input})
    while True:
        try:
            response = client.messages.create(
                model=MODEL, max_tokens=2048, tools=TOOLS, messages=messages,
            )
        except anthropic.RateLimitError:
            time.sleep(5)
            continue
        except anthropic.APIConnectionError:
            time.sleep(3)
            continue
        messages.append({"role": "assistant", "content": response.content})
        if response.stop_reason != "tool_use":
            return messages
        results = []
        for block in response.content:
            if block.type == "tool_use":
                try:
                    text = run_tool(block.name, block.input)
                    results.append({"type": "tool_result", "tool_use_id": block.id, "content": text})
                except Exception as e:
                    results.append({"type": "tool_result", "tool_use_id": block.id, "content": f"工具失敗：{e}", "is_error": True})
        messages.append({"role": "user", "content": results})

if __name__ == "__main__":
    messages = []
    messages = agent_loop("台北天氣如何？順便幫我記下『明天帶傘』。", messages)
    for block in messages[-1]["content"]:
        if block.type == "text":
            print(block.text)
```

執行後，你會看到模型先呼叫 `get_weather`、再呼叫 `save_note`，最後回傳一段完整的文字——那就是你第一個 Agent 完成的任務。

***

## 下一步

* 把 `run_tool` 接到真實的 API，例如氣象或資料庫。
* 改用 LangGraph，把多個步驟變成可視化的圖。
* 接上 MCP，讓你的 Agent 使用社群現成的工具伺服器。
* 參考本系列第二篇，把單一 Agent 拆成多代理協作。

***

## 重點回顧

* 新手先手寫一次迴圈，徹底理解原理。
* 工具 = 名稱 + 描述 + JSON Schema，描述決定模型何時呼叫。
* 迴圈的四件事：呼叫模型 → 看 `stop_reason` → 執行工具 → 回傳結果。
* 記憶的最小形式是「累積的 messages 歷史」，跨對話就存檔。
* 錯誤處理分兩類：工具失敗用 `is_error`，API 失敗用型別例外。
* 完成一個會查天氣、會記筆記、不怕出錯的 Agent，你就算正式入門了。
