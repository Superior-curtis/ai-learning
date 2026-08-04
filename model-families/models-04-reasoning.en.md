# Reasoning Models: The Pause Before the Answer

> 📅 2026-08-04 · Core Concepts
> o1/o3, thinking modes, effort levels — why do some models 'think for a while' before answering? Where it wins, what it costs, and when you simply shouldn't turn it on.

---

That **olympiad math problem** your desktop can't run — ask a normal model and it may stubbornly blunder. But ask the new generation of "reasoning models" and it will pause for a moment — that pause isn't a hang, it's the model **working through several steps in its own head before it starts talking**.

Last stop was multimodal (`models-03-multimodal`) — models that can "see." This post is about "thinking deeply": why a wave of new models openly brag, "I'll think first, then answer," and what that "thinking" is actually worth.

## The origin: still predicting the next token

Don't forget our single rule (`llmcore-01-next-token`): a model only predicts the next token. Reasoning models are no exception. So what is "reasoning"?

Its secret lives in an elegant design: **it treats "the thinking process" as tokens to emit in the conversation too.** A normal model goes "question → answer" in one go; a reasoning model goes "question → first emit a long stretch of internal chain-of-thought → then emit the final answer."

| Step | Normal model | Reasoning model |
|---|---|---|
| Receives the question | Starts answering directly | First writes an internal draft |
| Process | One-shot | Step-by-step self-reasoning |
| Output | Answer (no thinking trace) | Final answer (thinking kept behind the scenes) |

That internal thinking is called **chain of thought**: breaking a hard problem into many small steps, each "said to itself." It's the engine of a reasoning model — and its most valuable part.

## How that "thinking" gets trained

The "think first, then answer" behavior isn't innate — it's **deliberately taught during training via alignment techniques (the `train-03-rlhf` lineage)**: reward models that break reasoning into steps, walk back their own wrong paths, and then give a final answer. The "think aloud" demonstrations in the training data are its textbook.

So think of "thinking" this way: **the model has been trained to be willing to spend more tokens to buy accuracy.** It would rather emit a thousand tokens to reason a problem through than fifty tokens to blunder fast. **Accuracy and cost start trading with each other inside the model for the first time.**

## Why it wins on hard problems

Reasoning power isn't mysticism — it has solid reasons. Multi-step reasoning tasks share a trait: **one wrong step and everything is wrong.** A normal model's "one shot" style is like betting everything on a single step; a reasoning model can reach the middle, notice "this path is a dead end," and loop back to rethink.

| Problem type | Why it's strong |
|---|---|
| **Math** | Needs multi-step arithmetic, verified at each step |
| **Code** | First decomposes the problem, then considers edge cases |
| **Hard logic / patterns** | Needs to explore multiple hypotheses and rule out wrong ones |
| **Planning / scheduling** | Needs to enumerate options and compare trade-offs |

OpenAI's o1/o3, and the "thinking / effort" toggles every lab now ships, all control "how long it dares to think." **Higher means thinking longer, more accurately — but also slower and pricier.**

## The cost: no free lunch

Thinking comes with a bill. That "silent pause" isn't free:

* **Slower**: it writes hundreds or thousands of extra tokens before answering; latency balloons.
* **Pricier**: those thinking tokens are billed too; cost can multiply.
* **More power-hungry**: for data centers, reasoning mode is an energy hog.

So "turn reasoning to max on every request" is massively wasteful. You have to judge whether this question is worth its contemplation. A smarter approach is tiering: hand easy questions to the cheap, fast normal model, and escalate to reasoning mode only when a hard one is detected — an extension of the "pick the model by the task" lesson from `models-01-open-vs-closed`.

> Remember the one-line judgment: reasoning mode is for questions you genuinely can't figure out and where a wrong answer has a cost. Everyday trivia, simple lookups, repetitive tasks — leaving it on just burns time and money, without making answers noticeably better.

## When you don't need it

* **Everyday Q\&A**: what's for breakfast, how to translate this sentence — not worth a month of contemplation.
* **Simple knowledge lookups**: the model already knows, or RAG can answer (see `llm-03-rag`).
* **Low-latency products**: chat needs instant replies; who wants to wait 30 seconds every time.
* **Bulk, repetitive, mindless** generation tasks: thinking is waste.

One rule: **"Decide whether the question is easy or hard first; if it's easy, don't call the reasoning model."**

A useful framing from the prompting world: chain-of-thought (`prompt-03-chain-of-thought`) is the same idea you can turn on by hand in a normal model — "think step by step" — but a reasoning model has it built in and keeps it private. The boundary between the two is thinner than most people think.

## What one reasoning pass looks like

Take a "multiply two numbers, then take the remainder" problem and watch the difference between a normal and a reasoning model:

#### Read

First confirm what the question asks; clarify what is known and unknown.

#### Break down

Split the computation: multiply first, then take the remainder, each step independent.

#### Verify

Go back and check the intermediate values; make sure no condition was missed.

#### Answer

Only after confirming, output the final answer.

A normal model often skips the middle two steps — jumping straight to the answer; a reasoning model walks all four. **On easy problems, four steps are wasteful; on hard problems, they're a lifeline.**

## Controlling "how long to think" in code

Each SDK names the parameter differently, but the mental model is consistent — **one parameter tunes the length of thinking.** Pseudocode looks like this:

```python&#x20;—&#x20;adjusting&#x20;reasoning&#x20;effort
# pseudocode: each vendor's parameter name differs
# low effort: fast but careless; high effort: slow but careful
reply = client.chat(
model="reasoning-model",
messages=[{"role": "user", "content": "How to solve this"}],
reasoning_effort="medium",   # low / medium / high
)
print(reply)
```

Practical rule: **start at medium, raise only if it isn't enough.** The same mindset as tuning temperature (`llmcore-04-sampling`) — the default is usually the best value. Tuning effort is the single highest-leverage cost control a reasoning model gives you.

## Three practical tips for using reasoning models

A few small habits prevent a lot of pain:

1. **Give it a "how long" budget**: many models support an effort parameter; don't max it every time — medium often has the best price-performance.
2. **Don't interrupt it**: interrupting a reasoning model mid-thought forces it to hand over a half-baked answer. Give it room to finish.
3. **Watch its path**: some models publish a summary of their reasoning. That's a diagnostic signal — if it keeps circling one wrong assumption, the problem may be the question itself.

## Where reasoning mode actually hurts

Reasoning isn't always a plus; in some scenes it backfires — "overthinking" isn't "thinking well":

| Scene | Why reasoning mode is worse here |
|---|---|
| **Creative writing / ideation** | Too much thinking stiffens prose and kills flow |
| **Open-ended topics** | No "standard answer"; deep thought doesn't guarantee a hit |
| **Tone and mood** | Relentless calculation dries up the feel of language |
| **High-volume generation** | Cost and latency both explode, quality unchanged |

So the complete judgment is: **reasoning mode is only for problems with a clear right-or-wrong answer, multi-step structure, and a real cost to being wrong.** Sadly, that's exactly the slice most often misused — people throw creative problems at reasoning models and end up with something both slow and stiff.

## Treat them like two employees

A healthier metaphor: think of the normal model and the reasoning model as **two employees with different personalities**:

* **The normal model**: quick hands, efficient at everyday chores.
* **The reasoning model**: a slow, deliberate thinker — only worth waking for major problems.

A good team knows who to call when. **Run the one-step filter "is this worth waking the slow thinker?" before calling anyone** — that small habit can slash your API bill. It echoes the thread running through `models-01-open-vs-closed` and `models-02-families-tour`: **understand the task first, then pick the tool.**

## Before the last stop of this series

Reasoning models answer "how models think," which raises a deeper question: how strong are these models, really? Measuring "how strong" requires an honest yardstick — that's the stage for `models-06-evaluation`. But before that, there's a fleet of small-but-mighty models worth meeting: **distillation and small models**, at `models-05-small-models`.

Next step: go meet those models that are "small yet capable" — distilled small models, `models-05-small-models`.

#### Q: You're building a customer-service chatbot, and users expect instant replies. How should you configure reasoning mode?

* Use max reasoning mode on every request

* Turn it off or keep it low, because customer service is mostly everyday Q\&A and simple lookups

* All users should wait 30 seconds for thinking

* Reasoning mode only affects accuracy, not speed

> 💡 Customer-service conversations are mostly everyday trivia and simple lookups, and users want instant replies. Reasoning mode is slow and expensive; it only pays off when the question is genuinely hard. Turning it off or low for daily scenarios is cheapest and gives the best experience.
