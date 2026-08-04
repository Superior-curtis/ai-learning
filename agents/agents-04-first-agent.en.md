# Build Your First Agent, Step by Step

> 📅 2026-08-01 · Getting Started
> From picking a framework and defining tools to running the loop, adding memory, and handling errors — build your first AI agent one step at a time.

---

## Before you start

This tutorial walks you through building an agent that can check the weather, take notes, and survive errors. You need a Claude API key (or any model that supports tool use), Python 3.10+, and `pip install anthropic`.

Just want the full code? Jump to "The complete example"; otherwise follow the steps in order to understand each piece.

***

## Step 1: Pick your framework

| Approach | Best for | How much you write |
|----------|----------|--------------------|
| Hand-written loop | Understanding the fundamentals | Everything |
| Claude Agent SDK | Batteries-included tools and file access | Very little |
| LangGraph | Visual, graph-shaped flows | Medium |
| CrewAI | Multi-role division of labor | Medium |

> For beginners, write the loop by hand once. It is only a few dozen lines, and once you have done it yourself, you will understand what every framework is doing under the hood.

***

## Step 2: Define your tools

A tool = name + description + a JSON Schema for its inputs. The description matters — the model uses it to decide when to call the tool.

```python
TOOLS = [
    {
        "name": "get_weather",
        "description": "Get the current weather for a given city",
        "input_schema": {
            "type": "object",
            "properties": {"city": {"type": "string", "description": "City name"}},
            "required": ["city"],
        },
    },
    {
        "name": "save_note",
        "description": "Save a note to the notepad",
        "input_schema": {
            "type": "object",
            "properties": {"text": {"type": "string", "description": "Note content"}},
            "required": ["text"],
        },
    },
]

NOTES = []

def run_tool(name: str, args: dict) -> str:
    if name == "get_weather":
        return f"{args['city']}: sunny, 26°C"
    if name == "save_note":
        NOTES.append(args["text"])
        return f"Note saved ({len(NOTES)} notes so far)"
    return f"Unknown tool: {name}"
```

Each tool definition pairs with a handler function: it takes the arguments the model sent and returns a string result.

***

## Step 3: Run the agent loop

The core loop does four things: call the model → check `stop_reason` → execute tools → send the results back.

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
            return messages  # the model no longer wants a tool → done

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

`messages` is the accumulated conversation history, sent back in full on every call — this is the minimal form of an agent's "memory".

***

## Step 4: Add memory

To make the agent remember across conversations, the simplest approach is to persist the history and resume from it next time:

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
messages = agent_loop("Remind me to bring an umbrella tomorrow.", messages)
save_session(messages)
```

> Real long-term memory can go further: summarize stale conversations, write key facts into a note tool, or use a vector database. Start with persistence — it is enough to understand the principle.

***

## Step 5: Handle errors

The real world fails, and a robust agent handles two kinds of errors.

First, **tool execution failures** — mark them with `is_error` so the model knows and can change strategy:

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
                "content": f"Tool failed: {e}",
                "is_error": True,
            })
```

Second, **API call failures** — use the SDK's typed exceptions. `RateLimitError` (429) and `APIConnectionError` (network) can be retried after a backoff; for other `APIStatusError` cases, branch on the status code. The full approach is in `agent_loop` in the complete example below.

***

## The complete example

Stitching the pieces together gives you a complete, runnable agent:

```python
import time
import anthropic
client = anthropic.Anthropic()
MODEL = "claude-opus-5"
NOTES = []

TOOLS = [
    {"name": "get_weather", "description": "Get the current weather for a given city",
     "input_schema": {"type": "object",
                      "properties": {"city": {"type": "string"}},
                      "required": ["city"]}},
    {"name": "save_note", "description": "Save a note to the notepad",
     "input_schema": {"type": "object",
                      "properties": {"text": {"type": "string"}},
                      "required": ["text"]}},
]

def run_tool(name: str, args: dict) -> str:
    if name == "get_weather":
        return f"{args['city']}: sunny, 26°C"
    if name == "save_note":
        NOTES.append(args["text"])
        return f"Note saved ({len(NOTES)} notes so far)"
    return f"Unknown tool: {name}"

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
                    results.append({"type": "tool_result", "tool_use_id": block.id, "content": f"Tool failed: {e}", "is_error": True})
        messages.append({"role": "user", "content": results})

if __name__ == "__main__":
    messages = []
    messages = agent_loop("What's the weather in Taipei? Also note down 'bring an umbrella tomorrow'.", messages)
    for block in messages[-1]["content"]:
        if block.type == "text":
            print(block.text)
```

When you run it, you will see the model call `get_weather` first, then `save_note`, then return a complete paragraph — that is the task your first agent completed.

***

## Next steps

* Connect `run_tool` to a real API, like weather or a database.
* Move to LangGraph and turn your steps into a visual graph.
* Plug in MCP so your agent uses community-built tool servers.
* Check part 2 of this series and split your single agent into multi-agent collaboration.

***

## Key takeaways

* Beginners should hand-write the loop once to fully understand the mechanics.
* A tool = name + description + JSON Schema; the description decides when the model calls it.
* The loop does four things: call the model → read `stop_reason` → run tools → send results back.
* The minimal memory is the accumulated `messages` history; persist it for cross-conversation memory.
* Error handling comes in two kinds: tool failures via `is_error`, API failures via typed exceptions.
* When your agent checks the weather, takes notes, and survives errors, you are officially past the first step.
