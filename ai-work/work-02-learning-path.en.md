# An AI Learning Path for Developers: From Asking to Building

> 📅 2026-08-04 · Getting Started
> No framework memorization — a path that follows how the model thinks: first next-token, then prompting, RAG, agents, and MCP, and finally evaluating and shipping.

---

You already know how to code — that is your advantage. Frameworks will come and go, packages will rise and fade, but one thing stays constant: **how the model thinks.** And how it thinks can be stated in a single sentence: it predicts the next word (`llmcore-01`).

This article won't teach you to memorize any framework. It gives you a path you can actually follow, where each step builds on the last — from "asking AI" all the way to "shipping AI as a product." By the end you aren't just circling the user layer; you can build.

## Why learn the mental model before the frameworks

The most common beginner mistake is jumping straight into a package — LangChain, Pinecone, AutoGen — and finding yourself merely "gluing APIs" without understanding a thing. When the framework updates, your knowledge expires with it.

Build the mental model first and every framework becomes "some wrapper around that mental model":

* Prompting = narrowing the model's choices with words.
* RAG = putting knowledge into context so it doesn't guess from thin air.
* Agents = running the model inside a "act → see the result → act again" loop.
* MCP = a standard plug for giving that loop tools.

Understand the "guess the next word" machine, and all four follow by deduction rather than rote learning.

> The core of the whole path in one sentence: every AI application is just the predict-the-next-word machine wrapped with knowledge, tools, and loops. The mental model is the root; frameworks are only leaves.

## The learning path: six steps

Here is the road. Each step maps to a series in this book, and each prepares the ground for the next.

#### Step one: build the mental model

Read llmcore-01-next-token and really internalize what "predict the next word" means. Then read tokens (02), context (03), and sampling (04). These three tell you how it thinks, and every later skill is derived from them.\n\n→ Hands on: open a model and watch it continue sentence by sentence.

#### Step two: prompting

Read prompt-01-basics through prompt-04-robustness. The goal is not memorizing templates but learning the three ingredients — instruction, context, constraint — and how a system prompt stabilizes output. Write ten prompts; note which changes actually moved the result.

#### Step three: RAG — bring in knowledge

Read rag-01-what-is-rag through rag-04-evaluating. Learn how to answer things the model never trained on but your documents contain: chunking, indexing, hybrid search, reranking. This is the step that turns a know-it-all into an assistant that knows your stuff.

#### Step four: agents — make it act

Read agents-01-what-is-agent and agents-04-first-agent. Understand the loop: the model proposes an action, a tool executes it, the result feeds back, and it decides the next move. Skip heavy frameworks for now; write a minimal loop that can look things up.

#### Step five: MCP — connect your tools

Read mcp-01-what-is-mcp. MCP is the standard plug for tools and data, letting an agent reach your own APIs, databases, and file system. Take the agent from step four and wire it to a real tool over MCP.

#### Step six: evaluation and deployment

Read models-06-evaluation. Without evaluation you are just "feeling that it got better". Build a small test set and run it on every change. Then ship the result, monitor, and iterate.\n\n→ This step turns a demo into a product.

## Why this order

These six steps aren't arbitrary; each layer strips away an invisible assumption the next layer needs:

* Without the mental model, prompting is just reciting spells — when it fails, you don't know what to adjust.
* Without prompting, RAG tuning feels like luck — you can't tell whether the problem is retrieval or generation.
* Without a static RAG, jumping into agents means you can't tell "the model can't use tools" apart from "the retrieval returned garbage."
* Without agents, MCP is just another API format instead of the standard for plugging tools into a loop.

**Each layer builds the intuition for locating which layer a problem lives in.** That is something no framework tutorial gives you.

## Input and output of each layer

Collapse the six steps into four layers. Each one has a clean interface:

| Layer | You give it | It returns | Most common failure |
|---|---|---|---|
| **Prompt** | instruction, context, constraint | a stretch of output | answers too generic, wrong tone |
| **RAG** | documents + a question | a traceable, sourced answer | can't retrieve the right document |
| **Agent** | a task + available tools | a sequence of actions and results | the loop spins, goes in circles |
| **MCP** | tool definitions | standardized connections | tool overuse, runaway permissions |

The table's real value is debugging: when something fails, ask which layer. Generic answers → check the prompt layer. Confident but wrong → check retrieval. Erratic actions → check the agent loop. **Layering the blame correctly wins half the battle.**

## How to budget your time

You don't need to spread time evenly across the path. A practical split looks like:

* **Mental model + prompting (steps 1-2)**: about 20% — well spent, because it saves detours everywhere after.
* **RAG (step 3)**: about 30%. The most visible value per hour — it turns the model from a know-it-all into someone who knows your files.
* **Agents + MCP (steps 4-5)**: about 30%. The most likely place to stall, because you start handling uncertainty.
* **Evaluation + deployment (step 6)**: about 20%. Without this, everything before is just a demo.

Treat the split as a reference, not a law. The point: **don't spend 80% of your time on the showy 20%.**

## Evaluation: knowing you're improving

The middle of the path is where people get lost: "am I actually getting better?" Don't rely on feel. Build a minimal evaluation set.

```text
your test QA set ──→ run it on every change ──→ score up? keep it; down? roll back
↑______________________←____________________↓
```

Keep it simple. Pick ten to twenty questions you genuinely care about (they are *your* bar, not someone else's), and note the key points of an ideal answer. Run that set on every change — new prompt, new retriever, new model — and watch which direction the score moves. That is more reliable than any "it feels smarter now," and it's the spirit of `models-06-evaluation`.

You don't need perfection in evaluation to start. Letting the model grade itself (LLM-as-judge) is plenty for the first rounds; once scores stabilize, gradually add human-reviewed samples. **Evaluation isn't about being scientific — it's about being able to tell whether a change helped.**

## Three traps to avoid

* **Framework-first.** The urge to pick up each new package. APIs change; the mental model doesn't. Root yourself first; frameworks come along for the ride.
* **Tuning prompts without an eval.** Prompt-tuning is fast, but without an evaluation you're just overfitting today's example.
* **Building the "do-everything agent" in one shot.** Skipping steps 1-3 and expecting an autonomous all-rounder burns hours in "I don't know where the problem is." Smaller pieces, verified step by step, beat one brave leap.

The common thread: all three try to skip ahead before the layering intuition exists. The value of the path is that it makes you build that intuition first — then talk about speed.

## From example to product

By step six you have every ingredient to turn a demo into a product. Pick a real problem — small, but actual:

1. **Choose** something you do every day — reading reports, replying to email, organizing data — repetitive, rule-bound, time-consuming.
2. **Build it**: feed relevant documents via RAG, fix the output shape with prompting, and let the agent "look things up before answering."
3. **Evaluate** with that twenty-question set until it stabilizes.
4. **Deploy** into your everyday workflow, run it for two weeks, and record the time it saves and the mistakes it makes.

One round teaches you more than tools: it gives you the full method — **building AI = mental model + knowledge + tools + loops + evaluation.**

Don't pick too big a project. "An AI assistant for the whole company" is not a good project; "automating the fifteen minutes I spend reading reports and summarizing them each day" is. **Small enough to finish in two weeks, big enough to be worth doing — that's a good first step.** Then see where it can grow.

## Next

The theory and the path are in place. The most valuable first implementation is your first agent — landing step four, hands-on, following `agents-04-first-agent` from zero.

If someone on your team isn't a developer, hand them the other side of this series — using AI without writing code — in `work-03-non-developers`.

#### Q: Why does the learning path put the mental model before the frameworks?

* Because every framework costs money to learn

* Because prompting, RAG, agents, and MCP are all wrappers around the next-token machine, and the core lets you locate which layer a problem lives in

* Because the mental model is easier to memorize

* Because frameworks slow down your program

> 💡 Prompts, RAG, agents, and MCP are the same next-token machine wearing different add-ons. Once you have the core, you can tell whether a problem comes from the prompt, retrieval, tools, or the loop — a layering intuition that framework tutorials never give you.
