# The Provider Ecosystem: OpenAI, Anthropic, Google... and Beyond

> 📅 2026-08-04 · Core Concepts
> Want to call a model API? The world is far bigger than OpenAI. Map the three routes — closed labs, proxy platforms, and self-hosting — then master the conventions of requests, responses, streaming, and versioning.

---

You wrote a program and you want to "call AI." The first picture that pops into your head is usually OpenAI — but the real world is far wider than one company. "Provider" is not just "a company that sells models"; it is a whole ecosystem: closed labs, proxy platforms that forward your calls, and the option of running everything yourself. The `providers-` series, starting here, maps that whole territory.

First, the foundation: no matter who the provider is, the underlying engine does the same thing — **predicting the next token** (`llmcore-01-next-token`). The difference between providers is not the principle; it is **in what form you get this capability, what it costs, where your data goes, and who manages the model**. This post covers the "what form"; the next covers "what it costs," and the third covers "how to choose."

## Three routes: find your lane first

"Providers" roughly split into three lanes. Pin down which one you are in before going further, so you do not get lost:

| Route | Examples | What you control | What constrains you |
|---|---|---|---|
| **Closed lab API** | OpenAI, Anthropic, Google | Almost nothing but "calling" | Per-token billing, data passes through a third party |
| **Proxy platform** | Various gateways / routers | Choosing models, routing | An extra hop, possibly an extra margin |
| **Self-host / open-weight** | Llama, Qwen, Mistral | Weights, deployment, everything | You run the servers and maintenance |

These three lanes line up exactly with the "open vs. closed" split: that was the **first fork** (`models-01-open-vs-closed` — "can I download this model?"), and this "buy API / use a proxy / self-host" question is the **second fork** ("how do I plug this capability into my product?"). The two often overlap but are not the same thing — closed models can only be called by API, while open-weight models can be self-hosted or, in some cases, reached through a proxy. Keep both forks in your head at once, and every later decision has a place to land.

## A quick tour of the four major providers

If you simply want to "buy an API," you usually run into these names. The model families and their personalities live in `models-02-families-tour`; here is the "as an API" angle:

| Provider | Flagship models | One-line positioning |
|---|---|---|
| **OpenAI** | GPT family | Most mature ecosystem; the "default option" out of the box |
| **Anthropic** | Claude family | Strong long-form and writing; heavy focus on alignment |
| **Google** | Gemini family | Native multimodality; deeply wired into Google's ecosystem |
| **Mistral** | Mistral / open models | European origin; famous for efficient small models |

Do not read this table as a ranking — it is a "each has its strengths" map. To compare "which is actually better for you," use the own-task-set method from `models-06-evaluation`, not marketing; and `models-02-families-tour` has already toured which models suit what. The point here is: **they all hand you capability through an "API," and that shape looks surprisingly uniform.**

## The calling interface looks almost the same

Good news: **these providers' HTTP interfaces are strikingly similar.** Each gives you "one endpoint plus an API key"; the request is basically "a model name plus a list of messages," and the response returns "the generated text" and "how many tokens were used":

```The&#x20;shared&#x20;shape&#x20;of&#x20;request&#x20;and&#x20;response&#x20;(conceptual)
# send a request that understands "model + messages"
model      = "claude-..."
messages   = [ (role=user, content="Explain tokens in one sentence") ]
max_tokens = 100

# the response returns "text + usage"
reply.text          # the generated text
reply.input_tokens  # how many input tokens this call used
reply.output_tokens # how many output tokens were generated
```

Field names differ slightly (OpenAI reports `prompt_tokens` / `completion_tokens`; Anthropic reports `input_tokens` / `output_tokens`), but the concept is the same — and "what a token is, and why billing is counted in tokens" is the core of `llmcore-02-tokens`. Same idea, different names; learn it once and it generalizes.

### SDK or raw HTTP

Each provider ships official SDKs (Python, JS, and more), and you can also hit the endpoints as **raw HTTP**. A pragmatic take:

* **New projects: use the SDK.** It handles keys, retries, error types, and streaming details, saving you a pile of foot-guns.
* **Deeper work: look at HTTP.** For unified multi-provider interfaces, low-level debugging, or forwarding through a proxy layer, raw HTTP is often clearer.

Tooling is a means, not an end — an SDK is just the same interface wrapped more comfortably.

### House rules before you call: keys and error handling

Every API hands you an **API key**, like an ID card. A few practical habits:

* **Never put the key in frontend code**: it will be scraped and stolen. Keep it in the backend or in environment variables.
* **Handle errors**: rate limits return 429, bad keys return 401, an over-long context returns 400 (context limits live in `llmcore-03-context`). Retries with backoff are the baseline of reliability.
* **Store the usage numbers**: each response's token usage is **your cost ledger**. Record it, and the next post's math will not be guesswork.

## Streaming: no need to wait for the whole answer

Predicting the next token means the answer emerges one token at a time. If you wait for the model to "finish thinking through the whole passage" before returning, the user stares at a spinner for seconds. So mainstream APIs support **streaming**: the server sends text as it is generated, and you see characters appear one by one:

* **Streaming**: the first character shows up immediately; better perceived speed; ideal for chat and typewriter-style UIs.
* **Non-streaming**: wait for the full reply; ideal for backend aggregation where the user does not need real-time output.

Switching between the two is usually just a flag in the request, and the surrounding code is nearly identical. For latency-sensitive products (an axis we will weigh in `providers-03-choosing`), streaming should be the default.

## "Model versioning" is a real thing, not a detail

Providers use **the model name itself as version control** — `claude-3-5`, `gpt-4o`, `gemini-2.0`. Two habits matter:

1. **A name may point to a shifting implementation**: some labels (a `-latest` marker, or just convention) silently swap the actual model behind the scenes. If you care about stable behavior, pin the **dated** versions.
2. **Upgrading a model is changing your product**: a new version may alter output format or behavior. Before shipping, re-run your own task set (the `models-06-evaluation` method) against the new version.

> One sentence to remember versioning: "The model name IS your API contract." A name can silently change when you think it is fixed. Pin your versions and keep your acceptance tests, and a quiet "minor upgrade" will not break your product.

## Proxy platforms: one card to rule them all

Because the interfaces are so similar, someone was bound to build **proxy / gateway platforms**: they forward your calls to many providers behind one unified interface, so a **single key and a single program** can switch between models:

* Upside: **portable and comparable** — switching providers does not mean rewriting your calling code.
* Cost: an extra network hop, possibly an extra fee, and one more third party to trust.
* Great for: teams that want to "try two providers," or use routing (cheap models for easy tasks, flagships for hard ones).

Here is the whole ecosystem as one picture, showing how "your program" can connect:

```text
your program
↓ unified interface (proxy)
├──→ OpenAI (GPT)
├──→ Anthropic (Claude)
└──→ Google (Gemini)
↓ or connect to one provider directly
single API
↓ or skip providers entirely
self-hosted model (local-ai)
```

## Self-hosting: the other road

If what you want is total control — data never leaves your machines, a predictable bill, upgrades on your own schedule — then the `local-ai` series (`local-01-ollama`, `local-02-llamacpp`) and open-weight models are your home turf. You do not go through any "provider's API"; you run the weights on your own hardware. This is exactly where `models-01-open-vs-closed` lands in practice: there are several positions from which to buy capability, and which one you pick depends on how much you care about privacy, cost, and control.

## How to read this map going forward

A "provider" is not someone to worship or pledge loyalty to — it is a **source of tools**. The key habit is treating each one as a replaceable part: learn the interface once, pin versions, hide keys, and log usage. **Master those four habits and you stay free in this ecosystem** — switching providers becomes just a matter of swapping a key.

## Next

Once you know "where to buy," the natural question is "**what does it cost**" — because token pricing drives the economics of your whole application. Reading your bill, splitting it into input and output, and trimming it with caching and batching is arguably the highest-ROI post in this series: `providers-02-pricing`.

#### Q: You want to 'switch easily between two providers for A/B testing.' Which setup fits best?

* Integrate two raw SDKs and hard-code two separate programs

* Go through a proxy platform with a unified interface — one key to switch between providers

* Self-host an open-weight model

* Tightly couple your calling code to both providers

> 💡 A proxy unifies multiple providers behind one interface, so switching does not require rewriting your calling code — ideal for comparison and migration. Self-hosting is a different kind of control, not a way to 'switch easily.'
