# Tools & Actions: How Agents Act on the World

> 📅 2026-08-04 · Core Mechanics
> Tools are an agent's hands: from tool schemas and action types to error handling and graceful failure, a full look at how an agent actually gets things done.

---

## Recap in one take: tools are the "act" step of the loop

In the first article of this series we split an agent into "model + tools + loop"; in the last one you already wrote a working agent with `get_weather` and `save_note`. But notice: the only line that actually changes the world in the whole loop is `run_tool(name, args)`.

This article unpacks that one line from top to bottom — **what a tool looks like, how actions are classified, and how to fail gracefully when things go wrong**. Understand this and you are not writing "a program that calls an API" but an agent that genuinely acts on the world — and knows how to clean up when an action breaks.

***

## A tool is a contract between human and machine

Every tool is essentially a contract that states when it can be used, what arguments it takes, and what it returns. The contract has three fields:

| Field | Role | Why it matters |
|-------|------|----------------|
| `name` | Unique identifier the model uses to invoke it | Be precise: `get_weather` is obvious, `process_thing` is not |
| `description` | Natural-language purpose and when to use it | The model relies on it to decide *whether to call this tool* |
| `input_schema` | JSON Schema defining the arguments | Tells the model what to fill in and what is required |

> The description is the easiest field to overlook and the most critical. No matter how powerful a tool is, if the model does not know when to use it, it might as well not exist.

The schema uses JSON Schema — it describes whether an argument is a string or a number, whether it is required, and what values it may take. Given the schema, the model knows what shape of `arguments` to generate.

```json
{
"name": "get_weather",
"description": "Gets the current weather for a city, returning temperature and conditions.",
"input_schema": {
  "type": "object",
  "properties": {
    "city": { "type": "string", "description": "City name, e.g. Taipei" },
    "unit": { "type": "string", "enum": ["celsius", "fahrenheit"], "default": "celsius" }
  },
  "required": ["city"]
}
}
```

***

## Four basic kinds of action

Tools come in many flavors, but actions mostly fall into four buckets. Knowing which bucket your tool belongs in helps you design its description and its error handling.

### 1. Call: invoke an existing API

Wrap an existing service into a tool — check an order, send email, create an issue. This is the most common kind.

### 2. Search / Browse: query and explore

Look things up in documents, web pages, or databases. The catch is the result can be **multiple entries or incomplete**, so the model must judge which one is most relevant.

### 3. Execute: run code and write files

Run commands and edit files. These tools carry the highest privilege and the most room for accidents — separating "read-only" from "can write" is a basic safety floor.

### 4. Hybrid

For example, "search then return a summary" or "read a file and tally it" — doing two things in one call to cut round trips.

> A quick way to classify: Call asks "is it there?", Search asks "where is it?", Execute asks "change it to what?". Different questions, different failure handling.

***

## The full lifecycle of an action

A tool call, from "the model wants to act" to "the result comes back", is a short journey:

```text
model decides → generates arguments → system runs tool → gets result
│
←── result into context ── success or failure alike ──┘
```

* The model never truly "runs" code — it only **generates a structured call intent** (a tool use block).
* The actual execution happens on the system side, in code like your `run_tool` function.
* After it runs, wrap the result in a `tool_result` and hand it back so the model can continue its loop.

Keeping "the intent to call" separate from "the actual execution" is the key to agent safety: **the model proposes, the system approves**. The standardized interface from `agents-03-mcp` exists precisely to give every stop on this journey a consistent format.

***

## Failure is an everyday part of using tools

The real world has no hundred-percent-reliable tools. Networks drop, APIs return 500, files do not exist. The question is not whether something fails but **whether the model knows it failed**.

The most common approach is to return `is_error: true` so the model clearly sees that this action went wrong:

```text
tool_result (is_error: true)
"Orders API returned 404: order ORD-12345 not found"
```

The error message itself is also a craft:

| Failure type | Good message | Bad message |
|-------------|--------------|-------------|
| Not found | "Order not found — the ID may be wrong" | "error" |
| Bad argument | "city cannot be empty; provide a city name" | "Bad Request" |
| Transient fault | "Service busy — please retry shortly" | "fail" |

A good error message tells the model **what to do next** — retry, change an argument, or admit it cannot do this. That is what "graceful failure" means:

> An agent's reliability depends less on how perfect its tools are and more on whether it honestly tells the model when they fail. Pretending to succeed is where every agent disaster begins.

***

## Turning failure into recovery: three lines of defense

### First line: retry at the system layer

Rate limits (429) or transient connection errors should be auto-retried by the system — for instance, sleeping two seconds and trying again, without bothering the model.

### Second line: hand the failure to the model

If the system retries a few times and still fails, return `is_error: true` and let the model decide whether to change an argument, switch tools, or apologize and wrap up.

### Third line: provide a safe fallback

Some tools can fall back to a conservative result on failure. For example, if a search engine is down, return "search temporarily unavailable" instead of an empty string — an empty string is easily misread by the model as "no results".

***

## Before you build a tool, answer three questions

Answer these for every tool you write and you will save yourself hours of debugging:

* **Can the model always produce this tool's inputs?** If not, simplify the schema or merge tools.
* **What should the model do when this tool fails?** Hint at the next step inside the error message.
* **Does this tool deserve to exist?** If three lines of code replace it, do not add the extra layer.

Fewer, focused, well-described tools make an agent's actions more reliable. Design tools like an API, write error messages like a conversation — and your agent becomes not "a toy that moves" but "a teammate that finishes work."

***

## Next stop

Tools give an agent hands, but after finishing something it also needs to **remember what it did**. The next article, `agents-06-memory`, covers an agent's memory system: working vs long-term memory, the context window vs external storage, and when to hand memory over to RAG. Nail your tools first, then make them memorable, and your agent is truly "alive."

***

## Key takeaways

* A tool = name + description + input_schema, a human-machine contract.
* The description decides *when* the model uses the tool — the most critical field.
* Four action types: Call, Search/Browse, Execute, and hybrid — each fails differently.
* The model only *proposes* calls; the system *executes* them — separation is key to safety.
* On failure, return `is_error: true` with a message about what to do next.
* Three defenses: system retry → hand to the model → safe fallback.

#### Q: What is the main role of a tool's description field?

* It defines the format of the returned data

* It helps the model decide when to invoke the tool

* It sets the tool invocation permissions

* It determines how fast the tool runs

> 💡 The description states the tool purpose and when it applies in natural language, guiding the model on when to act; return format is set by the implementation, permissions by the system, and speed has nothing to do with it.
