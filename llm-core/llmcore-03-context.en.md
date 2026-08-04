# The Context Window: How Far the Model Can See

> 📅 2026-08-04 · Core Concepts
> Models have no memory — only a ceiling on how many tokens they can see at once, called the context window. It decides how big a conversation or file can fit.

---

A lot of people assume a chatbot "remembers" what you said. The truth is **the model has no memory** — it just re-reads everything so far within its current "window" on every turn. That "how much can it see at once" ceiling is the **context window**.

It's one of the most practical concepts you'll use: it decides how long a document you can paste, how long a conversation can run, and whether it "remembers" something you told it three hours ago.

## What the context window is

The context window = the ceiling on how many tokens the model can "see at once."

* 8K context → it can see up to ~8,000 tokens per turn (your pasted article + history).
* 200K context → it can hold a several-hundred-page book.

The key word is "**at once**": on every generation step the model takes **all** the tokens in the window as reference, then predicts the next word. Anything outside the window might as well never have existed.

```text
[system prompt][your article][conversation history]…[the word just generated] ← all must fit
│                                                                                  │
└────────────── at most = context window size (e.g. 200K tokens) ─────────────────┘
if the window fills early → the "oldest" content gets squeezed out
```

## The most misunderstood point: the model doesn't "remember"

What you call "memory" is actually a mix of two different mechanisms:

| You think | Reality |
|---|---|
| The model remembers the whole conversation | It re-reads the whole conversation every turn (inside the window) |
| The model remembers the preference you taught it yesterday | **No.** A fresh conversation starts from zero |
| The model has seen the PDF you pasted | Only within that one turn's window |

> Remember it in one sentence: the model "re-reads"; it doesn't "remember". Every message you send makes it flip through the window again. The moment the window changes (new conversation, over the limit), the old content is gone. Real "memory" comes from external storage, not the model.

## Context vs. long-term memory

So how is real long-term memory done, if the model forgets? **External tools** put the past back into the window:

* Store important facts in a database or file.
* On a new conversation, "paste back" the relevant snippets.

That's exactly what **RAG (retrieval-augmented generation)** does — it doesn't give the model a memory, it **moves the right old content back into the window whenever needed**. See `llm-03-rag`. So:

> Context window = the model's "working memory"; RAG / files = the model's "long-term memory". They get conflated all the time.

## Why the window has a ceiling

It's not that infinite windows are impossible — it's that **they're expensive**.

Attention makes every token "reference" every other token. As tokens grow, compute grows fast. Also, every paper you add to the window has to be read and billed; cost climbs with it.

```The&#x20;spirit&#x20;of&#x20;window&#x20;vs.&#x20;cost&#x20;(conceptual)
# Attention makes each token look at every other token → cost grows with length
# This is the other side of token billing: more input, more cost

# Each turn, you resend the ENTIRE conversation:
messages = [
  {"role": "user", "content": "first long article (8000 tokens)"},
  {"role": "assistant", "content": "first reply (500 tokens)"},
  {"role": "user", "content": "second long article (8000 tokens)"},
]
# This turn alone is already >16K tokens of input
```

So engineering is a trade-off:

* **Long-context models**: can read big files, but pricier and slower.
* **Short-context models**: cheap and fast, but can't hold long documents — you must "summarize then feed."

## Using a finite window well: practical habits

**1. Put the most important content first or last.** Models are often most sensitive to the opening (system prompt) and the ending (recent instructions); the middle gets diluted.

**2. Summarize, don't stack.** Mid-way through a long conversation, compress the earlier part into a paragraph and put that back — don't accumulate raw text to the brim.

**3. Feed only what's needed.** Instead of pasting a 100-page book, use RAG to pull out the relevant passages first.

**4. Chunk large files.** If it exceeds the window, split it into sections, ask in batches, then synthesize.

## "Putting knowledge in the window" vs. "changing the model"

This is one of the most important either-or choices in AI, and it's worth building intuition now:

| Approach | What changes | When |
|---|---|---|
| **Context**: put knowledge in the prompt | The input, every time | Always available, flexible, no retraining |
| **Fine-tuning**: bake knowledge into weights | The model itself | For fixed skills/styles; expensive |

For "should my model understand my company's product docs?", **nine times out of ten start with context / RAG — don't rush to fine-tune.** See `finetune-01-finetune-vs-rag`.

## So, how to think about "memory"

You now have the correct mental model:

* The model's "eyes" = the context window — how far it can see in one glance, with a ceiling.
* The model's "mouth" = one token per turn, endlessly continuing itself.
* The world outside the window, and the past between sessions — the model is simply "not present."

Next, let's look at that "next-word lottery": the odds can be tuned hotter or colder — `llmcore-04-sampling`.

#### Q: Why is the impression that 'the model remembers what I taught it yesterday' wrong?

* The model has a memory, just a small one

* The model re-reads the current conversation inside the window; a new conversation starts from zero with no cross-session memory

* Only the paid tier has memory

* The model stores preferences in the cloud, but they expire

> 💡 The model is stateless: it only sees (and re-reads) the current window. Cross-conversation 'memory' must come from external storage (like RAG/files) that moves content back into the window when needed.
