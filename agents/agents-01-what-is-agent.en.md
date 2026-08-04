# What Is an AI Agent? From Definition to the Agent Loop

> 📅 2026-08-01 · Core Concepts
> A one-sentence definition and one loop that explain AI agents, plus why they are fundamentally different from plain chatbots.

---

## What Is an AI Agent?

An AI agent is a system that **completes a goal autonomously**: instead of just answering your question, it works through a repeated cycle of "understand → plan → act → check results" until the task is done.

> One sentence to remember: A chatbot answers; an agent completes.

An agent is usually made of three parts:

* **Model** — the brain that judges and decides, e.g. Claude.
* **Tools** — the interfaces that let the model "get its hands dirty", e.g. checking the weather, doing math, reading files.
* **Loop** — the execution framework that ties perceive, think, and act together.

***

## One loop beats a thousand words

What makes an agent an agent is the **agentic loop**. The most common abstraction has four steps:

```
Perceive → Think → Act → Observe
    ↑                          │
    └──────── back to the loop ┘
```

### 1. Perceive

The agent first takes in the current state of its environment. In a chat scenario that is the user's input; in more complex scenarios it might be web content, files, or sensor data.

### 2. Think

The model decides "what to do next" based on the current state and the goal. It may draft a plan, judge which information is still missing, or decide which tool to call.

### 3. Act

The agent performs one concrete action: call a tool, send a request, or give the final answer.

### 4. Observe

The agent takes in the result of its action, records the new information into its context, and returns to the perceive step for another round.

***

## The skeleton of the agent loop

In pseudocode, the loop looks like this:

```text
while True:
    observation = perceive(environment)   # perceive current state
    plan       = think(observation)       # decide the next step
    action     = act(plan)                # execute the action
    result     = execute(action)          # get the result
    update_memory(observation, action, result)  # remember, then loop again
```

When does the loop end? When the agent decides the goal is met (it no longer needs to call a tool), it emits the final answer and exits.

***

## How an agent differs from a chatbot

| Aspect | Chatbot | AI Agent |
|--------|---------|----------|
| Goal | Answer questions | Complete tasks |
| State | Usually stateless | Has a goal and memory |
| Tools | Rarely uses them | Calls tools frequently |
| Behavior | One exchange | Multi-step loop |
| Failure handling | Ask again | Try a different strategy |

A chatbot is question-and-answer; an agent is one-task-one-loop. The moment a system **starts deciding on its own to call tools and adjusts its next step based on results**, it crosses the line from chat to agent.

***

## A real agent loop: a minimal version with the Claude API

Below is a tool-using agent built with the official Anthropic SDK — the smallest "hand-written loop" example:

```python
import anthropic

client = anthropic.Anthropic()

TOOLS = [
    {
        "name": "get_weather",
        "description": "Get the current weather for a given city",
        "input_schema": {
            "type": "object",
            "properties": {"city": {"type": "string"}},
            "required": ["city"],
        },
    }
]

def run_tool(name: str, args: dict) -> str:
    # In a real app, hook up a weather API here
    return f"{args['city']}: sunny, 26°C"

messages = [{"role": "user", "content": "What's the weather in Taipei?"}]

while True:
    response = client.messages.create(
        model="claude-opus-5",
        max_tokens=1024,
        tools=TOOLS,
        messages=messages,
    )
    messages.append({"role": "assistant", "content": response.content})

    # No tool call requested → task is done
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

Three things to notice:

* `stop_reason == "tool_use"` means the model wants to call a tool; `"end_turn"` means it is done.
* The API is stateless — you must send the full `messages` history back on every call.
* Tool results come back as `tool_result`, each paired with its matching `tool_use_id`.

***

## When should you build an agent?

Not every problem needs an agent. Four criteria to judge:

* **Complexity** — the task needs multiple steps that cannot be fully scripted in advance.
* **Value** — the outcome justifies the extra cost and latency.
* **Viability** — the model is actually capable of this kind of task.
* **Cost of error** — failures can be caught and recovered from.

Only when all four answers are "yes" is it worth building an agent.

***

## Key takeaways

* An agent = model + tools + loop; the core is completing goals autonomously.
* The agent loop has four steps: perceive → think → act → observe.
* A chatbot answers questions; an agent completes tasks.
* The key to a real loop is reading `stop_reason` and feeding tool results back to the model.
* Use the four criteria to decide whether you need an agent before you start building.
