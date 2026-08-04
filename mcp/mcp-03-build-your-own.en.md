# Build Your Own MCP Server

> 📅 2026-08-04 · Getting Started
> A few dozen lines turn a custom tool into a standard MCP Server. Walk through TypeScript and Python versions, run it, connect any client, with notes on errors and security.

---

The last two posts covered why MCP exists and what it looks like. This one changes posture: **let's build an MCP Server that actually runs.** The goal is deliberately small — one Server, one exposed tool, running, callable from any MCP-compatible client. When you're done, the metaphor from `mcp-01` and the architecture from `mcp-02` become muscle memory instead of theory.

The concept load is low: MCP's SDK wraps the JSON-RPC handshake, the message formats, and the stdio transport for you. All you write is the tool itself. It's shorter than you expect.

## What you need

A Node.js environment (18+ recommended). Create a project folder and install the official MCP SDK:

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk
```

Once that's installed, the Server's entire job is three things: **create a Server, register one tool, attach a transport.**

## The server, end to end

Create `server.ts` and paste this:

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
  description: "Add two numbers and return the sum",
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

That's all of it. No hand-rolled TCP, no manual JSON parsing — the SDK handles everything.

## Reading it line by line

Break the code down:

* **`new McpServer({ name, version })`** — the Server's ID card. Clients see it during the handshake.
* **`registerTool("add", { description, inputSchema }, handler)`** — registers one tool. First argument is the name, second is the "shape" (description + JSON Schema), third is the function that does the real work.
* **`inputSchema`** — the "how tools are exposed" part from `mcp-02`. It's a standard JSON Schema telling the model "this tool takes two number parameters, `a` and `b`." The model reads it to decide what to pass.
* **`content: [{ type: "text", text: ... }]`** — MCP's return format. Results come back as standard content any client can handle uniformly.
* **`StdioServerTransport`** — stdio as the transport; the "JSON-RPC" line in `mcp-02`'s diagram lands here.

We use `String(total)` instead of a template literal — not a requirement, just keeping the snippet clean. In your own code, use whatever you prefer.

## Run it

Since it's TypeScript, run it directly with `tsx`:

```bash
npm install -D tsx
npx tsx server.ts
```

It starts and just *sits there*, printing nothing. That's normal. It's listening on stdin for messages. You can smoke-test it by piping in a manual JSON-RPC message:

```bash&#x20;/&#x20;manual&#x20;test
printf '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"tester","version":"0.1.0"}}}\
' | npx tsx server.ts
```

When you see JSON come back, the Server is alive and can handshake. Everything after that, a real client does for you automatically.

## The Python version: just as short

Not on the TypeScript train? The official `fastmcp` package is even tighter. Install it:

```bash
pip install fastmcp
```

Create `server.py`:

```python
from fastmcp import FastMCP

mcp = FastMCP("math-server")

@mcp.tool()
def add(a: int, b: int) -> int:
  """Add two numbers and return the sum."""
  return a + b

if __name__ == "__main__":
  mcp.run()  # stdio transport by default
```

The Python version is more compact: the `@mcp.tool()` decorator turns `add` into a tool, using type annotations as the input schema and the docstring as the description — the model reads both to figure out what to pass. Run it:

```bash
python server.py
```

The key contrast: **TypeScript declares the schema explicitly; Python derives it from types.** Either way, the Server exposes the same standard "name + description + input schema," and any client recognizes it.

## Connect any client

With the Server written, the rest is letting an MCP-compatible client connect. Hosts manage Clients, as `mcp-01` described. In Claude Code, for example, you declare this Server in config, and it automatically runs `tools/list` to discover `add`, then calls it whenever the conversation needs it. Because the interface is standard, **the same Server moves to any MCP-compatible client with zero code changes** — that's the payoff of standardizing.

## Errors: make failures visible

Tools will fail. Bad arguments, a downed backend — it happens. MCP's rule: **mark the failure with `isError` instead of dropping it.**

```typescript
async (args) => {
if (typeof args.a !== "number" || typeof args.b !== "number") {
  return {
    content: [{ type: "text", text: "a and b must both be numbers" }],
    isError: true,
  };
}
const total = args.a + args.b;
return { content: [{ type: "text", text: String(total) }] };
}
```

Why? As `mcp-02` noted, `isError: true` sends the failure back as a normal result, so the model can read the error message and correct itself. Swallow the error or throw an exception, and the model loses its chance to self-heal.

## Security notes

It runs, it connects — don't forget what you just did: **you let a model influence a real system through your tool.** That hands over part of the control. A few ground rules:

* **Least privilege by default** — the tool touches only what it needs, never the whole system.
* **Validate all inputs** — treat model-supplied arguments as untrusted, exactly like user input.
* **Keep auth on the server side** — the Server manages credentials; the agent shouldn't hold every key.
* **Log every call** — so you can trace who did what, when.

> Design this MCP Server like a public API: least privilege by default, validate every input, log every call. Opening a tool to a model is delegating control — security-07-agent-security lays out the full attack surface of agent systems.

## Wrapping up

You now hold a real, running MCP Server: one standard tool, callable by any client, with failures the model can see. Point it at a real database or an external API, and everything since `mcp-01` connects into a single line.

It also sets up the next layer: MCP is exactly the tool-wiring standard that `agents-03-mcp` describes. With tools in place, your agents can finally *do things*.

#### Q: In your MCP Server, how should a failed tool execution be returned?

* Throw an exception and crash the client

* Wrap the result in content and set isError to true

* Fail silently and return nothing

* Re-run initialize and handshake again

> 💡 MCP marks failures with isError, returning them as normal results so the model can read the error message and retry. That self-correction loop is the foundation of agent resilience.
