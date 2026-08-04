# MCP Architecture: Server, Client, Tools, and Resources

> 📅 2026-08-04 · Architecture
> What exactly does an MCP Server expose? Tools are callable actions; resources are readable context. Break down the difference, how MCP and function calling divide labor, and the JSON-RPC transport underneath.

---

You may not need to understand every detail of MCP to use it comfortably — but the moment you write your own server, or debug someone else's integration, a clear blueprint saves you a lot of dead ends. This post fills in the internals `mcp-01-what-is-mcp` left out: **what a Server actually exposes, how a Client talks to it, and what the messages look like.**

`mcp-01` set up the three roles (Host / Client / Server) and the journey of a single request. Now zoom in on the building blocks inside the Server.

## The big picture: what a Server exposes

MCP defines four shapes of "capability" that a Server speaks through:

* **Tools** — callable actions.
* **Resources** — readable context.
* **Prompts** — reusable prompt templates.
* **Roots** — a list of locations the Server may access.

The first two matter most, and this post sticks to them. You already met Prompts in `agents-03-mcp`, so no rehash; Roots are an advanced topic to look up when you need them.

## Tools: callable actions

Tools are the capabilities that *do things*. Look up weather, create an issue, post a message, run a SQL query — those are tools. Every tool promises a Client three things:

1. **A name**: `get_weather`, `create_issue`.
2. **A description**: plain language for what the tool does and when to use it.
3. **An input schema**: a JSON Schema describing which arguments to pass and their types.

The defining trait of a tool: **it changes the world** — at least the world on the Server's side. Call `create_issue`, and a real issue appears on GitHub. Handing tools to a model therefore hands over decision-making power, which deserves real care; we pull that thread in `security-07-agent-security`.

## Resources: readable context

Resources are data you can *read*. A document, an image, a slice of a database — all resources. Two features stand out:

* **They are addressed by URI**: `file://`, `db://`, `https://` are all legal prefixes that say where the material lives.
* **They can be text or binary**: documents are usually plain text; images and PDFs come through as binary.

The defining trait of a resource: **it does nothing by itself.** It is just material that can be brought into the model's context. The model reads the resource, then either acts via tools or answers directly.

One sentence separates the two:

> A tool is a callable action (it does things); a resource is readable context (it supplies material). "The model wants data" → read a resource. "The model wants to change something" → call a tool.

## Tools vs. resources: side by side

| | Tools | Resources |
|---|---|---|
| Essence | action, function | data, content |
| Effect | changes the world on the Server's side | feeds the model context |
| How exposed | name + description + input schema | addressed by URI |
| How called | `tools/call` | `resources/read` |
| Side effects | yes | none (read-only) |

Hold those two columns in mind and most architecture confusion dissolves.

## MCP vs. function calling

A common question: don't the vendors already have "function calling / tool use"? Why do we need MCP?

The answer is that they operate on different layers.

* **Function calling is a model-side capability.** The model learns to decide which function to call and what arguments to pass. It decides how to get a single tool call right.
* **MCP is an interface-side standard.** It decides how tools are described and transported. It governs how tools reach the model.

Lock the distinction in with a side-by-side:

| | Function calling | MCP |
|---|---|---|
| Layer | model side: getting one call right | interface side: how tools are described and carried |
| Answers | which function, what arguments | how tools reach the model |
| Defined by | each model vendor | an open protocol and its community |
| Relationship | the thing being called | the standard that makes tools callable |

Back to the `mcp-01` metaphor: function calling is the model being *able to speak*; MCP is the *plug standard for the phone line*. Being able to speak doesn't guarantee two phones can interoperate — and the plug standard is exactly what solves interoperability. They complement, not replace, each other: the tools declared by an MCP Server are ultimately invoked through the model's native tool-use mechanism.

## On the wire: JSON-RPC

Client and Server talk JSON-RPC 2.0. A message is simply "a JSON with a method name and parameters":

* Client → Server: `initialize` (the handshake), `tools/list`, `tools/call`, `resources/read`
* Server → Client: matching responses, or `notifications` (one-way, no reply expected)

Only a handful of methods come up in practice, so here they are:

| Method | Direction | Purpose |
|---|---|---|
| `initialize` | Client → Server | handshake: exchange versions and capabilities |
| `tools/list` | Client → Server | list the available tools |
| `tools/call` | Client → Server | invoke one tool |
| `resources/read` | Client → Server | read one resource |
| `notifications/*` | both ways | one-way notices, no reply |

The most common transport is **stdio**: the Server is just a program that reads stdin and writes stdout, and the Client runs it as a child process, communicating over standard input and output. The architecture looks like this:

```text
Host (the agent app: Claude Code / your program)
│  one Host can own many Clients
▼
Client A ──JSON-RPC──▶ Server A  (database: tools + resources)
Client B ──JSON-RPC──▶ Server B  (filesystem: tools + resources)
```

One Client per Server; want more capability, add another Server.

## The life cycle of a connection

A Client connecting to a Server roughly goes through four phases:

#### Handshake with initialize

Client and Server exchange protocol versions and capabilities to confirm they can talk.

#### Explore with tools/list

The Client asks for the tool list and resource list; the Server hands over the roster.

#### Work with tools/call and resources/read

The model calls tools or reads resources as needed — the day-to-day battlefield.

#### Close and disconnect

The conversation ends and the Client closes the connection; next time, just handshake again.

Most of the time is spent in phase three. The handshake and exploration are one-time openers.

## A concrete message

A `tools/call` exchange looks like this on the wire:

```json
// client → server: call a tool
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{
  "name":"get_weather","arguments":{"city":"Taipei"}
}}

// server → client: return the result
{"jsonrpc":"2.0","id":3,"result":{
  "content":[{"type":"text","text":"Taipei: sunny, 26°C"}],
  "isError":false
}}
```

Notice the `isError` field: a failed tool execution is not a JSON-RPC-level error. It comes back wrapped in a normal result with `isError: true`. Why? So the model can *see the failure and read the message*, then correct and retry — which is exactly the self-healing loop agents rely on.

## Wrapping up

MCP's architecture fits in four building blocks: **tools are actions that do things, resources are material that can be read, and both flow between Host and Server over JSON-RPC.** Once you keep "action" and "data" apart, this blueprint is yours.

Next comes the hands-on part: `mcp-03-build-your-own` walks you through writing a real Server, turning this diagram into a program that actually runs.

#### Q: A model needs the contents of the latest contract to answer a question. What should it do?

* Call a "read contract" tool via tools/call

* Read the contract resource via resources/read

* Answer from training memory without looking

* Re-run initialize and handshake again

> 💡 Reading data is fetching material, not changing the world, so it belongs to resources and goes through resources/read. Tools are for actions with side effects, and initialize is only the handshake, not a way to fetch content.
