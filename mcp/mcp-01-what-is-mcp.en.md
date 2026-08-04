# What Is MCP: The USB-C of AI

> 📅 2026-08-04 · Core Concepts
> Every new AI tool needs its own custom integration. MCP is the open standard built to fix that. Break down the problem it solves and how the Host, Client, and Server roles divide the work.

---

You have five devices on your desk, and every one needs a different cable: A takes a round plug, B a square one, C needs an adapter, D is wireless only. Every time you switch devices, you carry another cable — which is precisely what wiring AI tools feels like today.

For years, every AI tool has behaved like a gadget that only takes its own proprietary cable. Want a chatbot to query a database? Write a "query the database" integration. Want it to read files? Write a "read the files" integration. Want it to book a flight? Another one. Each service has its own calling convention, its own argument format, its own error handling. Huge chunks of a developer's time go not into "making the model smarter" but into "wiring up tools."

`agents-03-mcp` already laid out MCP's place in the wider agent ecosystem. This post is the starting point of the mcp series, with a simpler job: **explain plainly what MCP actually solves.** No hype, just the facts.

## The problem: every tool needs re-wiring

Let's define the problem precisely. Say you are an agent developer. The model already "predicts the next token" — the mental model `llmcore-01` leans on throughout — but to make it *do* things you must give it tools. And every tool sits in front of a real system: a database, a filesystem, Slack, GitHub.

The trouble stacks up layer by layer:

* **Every service is unique.** Slack's API and GitHub's API are nothing alike. Calling conventions, response shapes, auth flows — all vendor-specific.
* **Integration is ongoing maintenance.** The vendor ships a breaking change, and your integration breaks; you chase the fix, then they change again.
* **Agent frameworks disagree.** The same tool gets one wiring scheme in framework A, another in B, yet another in C.

So "add a new tool" becomes an expensive, re-do-the-same-work chore. Worse, the effort barely accumulates: the code you write today to wire up Slack is thrown away tomorrow when you switch frameworks and start over.

One line captures the pain: **the more tools you connect, the more repeated work you do, and the less reusable assets you keep.**

## The fix: make the wiring a standard

MCP (the Model Context Protocol) exists to do exactly this: **standardize how agents connect to tools.**

Notice what it is *not*: not a service, not a framework, not even a mandatory SDK. It is an agreed-upon set of rules that defines:

* what a tool looks like (name, description, argument schema)
* how a client and server shake hands and talk
* how call results return and how errors are expressed

The most common metaphor is USB. Before USB, keyboards, mice, and printers each had their own plug and wouldn't talk to each other. After USB, one cable works everywhere. MCP aims to do the same — except instead of peripherals, it is wiring up the tools an AI can use.

> USB standardizes not "how each device works internally" but "how each device gets connected." MCP standardizes not "how each tool is implemented" but "how each tool gets called by an agent." That is the single most important sentence for understanding MCP.

Laying "before vs. after" side by side makes it click:

| Before: one wiring per tool | After: one standard, compatible everywhere |
|---|---|
| Every API needs a bespoke integration | Write a server once; any client can connect |
| Argument formats differ everywhere | Always "name + description + JSON Schema" |
| Switching frameworks means rewriting | The standard is framework-agnostic |
| Integrations don't accumulate | Finished servers get shared and reused |

## Three roles: Host, Client, Server

MCP's architecture has just three roles. Learn their jobs first:

* **Host** — the application you are using: Claude Code, the Claude desktop app, or your own agent program. It handles the interaction and presides over the whole conversation. The model, the context, and the decision loop live here.
* **Client** — the "connector" inside the Host. Each Client corresponds to one Server and talks to it in the standard format, translating the Server's capabilities back for the Host.
* **Server** — whoever wraps a capability behind a standard interface. A "weather server" wraps the weather API; a "filesystem server" wraps the filesystem. It sits the farthest from the Host.

Grasp this relationship once: **one Host can hold many Clients, one Client connects to one Server, and one Server offers a batch of tools.** Want your agent to query a database, read git history, and post to Slack at once? Wire up one Server each and let them stand side by side.

## The journey of a single request

Put the roles into motion, and one tool call looks roughly like this:

#### Host asks around

The Host wants to know which tools this Server offers.

#### Client runs the errand

The Client sends tools/list to the Server in the standard format.

#### Server hands over the roster

The Server returns a standard tool list, a name, a description, an argument schema.

#### The model picks a tool

Reading the list, the model decides which tool fits and issues a tools/call.

#### Server does the real work

The Server actually queries the database, actually posts the message.

#### The result flows back

It returns in the standard format and lands in context, then the model keeps predicting the next token to answer a human.

The turning point is step 3 onward: **the model receives a standardized tool description and never needs to know how Slack or GitHub's API actually looks.** You just wrap the Server; the standard does the connecting.

## Walk through it with weather

Abstract concepts only stick once they land on a concrete case: you want your agent to check the weather.

1. **Server** — you (or the community) write a "weather server" that wraps a weather API behind a standard tool called `get_weather`, declaring via JSON Schema that it takes a city name and returns temperature and conditions.
2. **Client** — inside your agent program you add a Client that connects to this Server over stdio or HTTP.
3. **Host** — you ask the agent "will it rain in Taipei tomorrow?" The model in the Host sees `get_weather`'s description, decides to use it, and calls it with `city: Taipei`.
4. **Result** — the Server queries the weather API and returns the result in the standard format; the model turns the numbers into a plain-language answer for you.

From start to finish you wrote zero lines of weather-API integration code. The connection is standard; the lookup is the Server's job. That is the effect MCP wants for every tool.

## Misconceptions, cleared up

MCP's rise has brought a few myths along with it. Here are the three most common:

* **Myth one: MCP is a concrete product.** It isn't. It is a protocol, a set of rules. You can skip every official SDK and implement a Server against the spec yourself — perfectly legal.
* **Myth two: with MCP you skip the coding.** The opposite — you still write the logic behind the tools. MCP removes the *wiring* work, not the *implementation* work.
* **Myth three: MCP is only for big companies.** On the contrary, it is an open standard anyone can join, from individual developers to hobbyists. `mcp-03-build-your-own` is the proof.

## Who drives MCP

MCP was open-sourced by Anthropic in late 2024, but the operative word is "open": the spec is public, the SDK is open source, and any company or framework can implement it.

* **Agent apps** — Claude-family products support it natively, and more agent frameworks are adopting MCP as the default way to wire tools.
* **Community servers** — databases, GitHub, Slack, browsers: ready-made MCP Servers are accumulating fast, install one and it just works.
* **The wider developer community** — a standard survives not because "one big vendor pushes it" but because everyone can plug in. That is the fundamental difference between MCP and a private SDK.

## Where this series goes next

This is part one of the mcp series — the foundation. The next two posts:

* `mcp-02-architecture`: dive into what sits between Host and Server — tools, resources, the transport layer, and how MCP differs from the various vendors' "function calling."
* `mcp-03-build-your-own`: build your own MCP Server hands-on, a few dozen lines that actually run.

If you want more grounding around agents generally, `agents-03-mcp` reads well as a companion; that post takes the "agent ecosystem" bird's-eye view, while this one explains the protocol itself.

## The honest, no-hype version

MCP is talked up a lot these days, so let's set expectations straight:

* **It is not the only standard.** Other protocols compete; MCP is simply the one the industry has most widely adopted. The "USB-C" metaphor captures the power of standardizing, not a claim of total victory.
* **It does not buy you security.** Opening tools to a model hands the controls to it. Tools are a fresh attack surface — a theme `security-07-agent-security` will develop in full.
* **It does not make tools magical.** Someone still has to write and maintain the Server. MCP removes the wiring chore, not the job of building the system.

But the direction is right: once wiring becomes a standard, every finished integration stops being a one-off asset and becomes shared infrastructure any MCP-compatible agent can reuse. That is what lets the whole ecosystem keep compounding.

## Quick FAQ

#### Do I have to use Anthropic products with MCP?

No. MCP is an open protocol; any Host that supports it can use it. Anthropic is just the originator.

#### How many Servers can one Host connect to?

There is no hard limit. In practice it depends on memory, the length of the tool list, and how many options the model can handle at once.

#### Can only agents call MCP Servers?

Mostly. But because the interface is standard, a human can talk to a Server directly too, for example sending JSON-RPC messages by hand while debugging.

## The one-liner to remember

MCP does not fix "the model is not smart enough"; it fixes "the tools won't plug in." Turn tool wiring into a standard, and every integration flips from "done once" to "accumulated." Next time someone says "MCP is the USB-C of the agent world," you know what they really mean: **standardize the connection so integrations can be shared and compounded.**

#### Q: What does MCP standardize?

* How every AI tool is implemented internally

* How agents connect to and call tools

* How models predict the next token

* How tools store their data

> 💡 MCP governs the connection interface, not internal implementation: how tools are described, called, and how results return. Like USB, it unifies the plug, not the device itself.
