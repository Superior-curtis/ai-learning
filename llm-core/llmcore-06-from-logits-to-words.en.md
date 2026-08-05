# From Scores to Words: The Decoding Pipeline

> 📅 2026-08-04 · Core Concepts
> Models don't speak — they just run the same tiny loop, over and over: score the context, draw one word, append it, and repeat. This post walks the full decoding pipeline and closes out the llm-core series.

---

Over the last five posts we took an LLM apart: tokens (`llmcore-02-tokens`), the context window (`llmcore-03-context`), sampling (`llmcore-04-sampling`), hallucination (`llmcore-05-hallucination`), all rooted in the one trick of predicting the next word (`llmcore-01-next-token`). Now let's put the parts back together and watch the whole machine run: from the moment you hit send to the moment text starts appearing — **what actually happens in between?**

The answer is **decoding**, and it is more boring — and more elegant — than you'd expect. A model doesn't "speak". It just runs the same tiny loop: **look at the current context, score every candidate, draw one token, append it, and do it again.**

## The whole loop at a glance

```text
the tokenizer chops the input into tokens
↓
[ context ]  ← system prompt + messages + history + tokens so far
↓
forward pass: one score per token in the vocabulary (logits)
↓
softmax → probability distribution
↓
sampling draws one token (llmcore-04-sampling)
↓
append it to the end of the context
↓
(loop) predict the next token again ……
```

Each lap outputs exactly one token. A 500-word reply is just this loop quietly running a few hundred times. Every step is dull in isolation; together they write an essay — the magic lives in the loop itself.

## Step one: tokens come in

Before the model does anything, the tokenizer chops the input into tokens (`llmcore-02-tokens`). Then the system prompt, your message, and the conversation so far all have to fit inside the context window (`llmcore-03-context`). Whatever fits, the model sees; whatever doesn't, doesn't exist.

Notice something easy to miss: the model reads a **snapshot** of those tokens. Each lap, it re-reads the entire context as it currently stands — not "remembers", just re-reads.

## Step two: one forward pass, out come logits

The model pushes those tokens through its dozens of layers — one **forward pass**. The output is a long list: **one score for every token in the vocabulary**. Those raw scores are called **logits**.

* Higher score = "the model thinks this token is a better continuation".
* Logits are raw scores, not probabilities — they can be positive, negative, any magnitude.

This is the only real "thinking" in the whole loop. A transformer is, at bottom, a giant function that maps a token sequence to a list of scores. The "knowledge" and "understanding" from the earlier posts? It all lives in the parameters of that function.

## Step three: sampling picks the winner

Logits aren't usable directly. First `softmax` turns them into probabilities (summing to 1), then we **draw one token** by probability — the lottery from `llmcore-04-sampling`.

* Temperature, top-p, and top-k all act at this step.
* Whichever token wins becomes this lap's output.

Ask the same question twice and get different answers? That's this draw. Want more predictable output? Lower the temperature and trim the low-probability tail — right here.

## Step four: append, then repeat

The winning token is **appended to the end of the context**, forming a slightly longer context. The loop goes back to step two: score the new context, draw again.

Because the model feeds its own output back in as input, it's called **auto-regressive** — the single most important word in the whole generation machinery. The loop doesn't run forever: it stops when the model draws the special end-of-text token (EOS), or when it hits the length limit.

## Why it reads like "writing"

Taken apart, every step is mechanical. Together, the effect is fluid — because **each step can see what earlier steps just wrote**. Once the model writes "Paris", the next lap already knows "Paris" appeared, so it can follow with "the Seine". Word by word, "remembering what it wrote" isn't some extra memory — it's what the loop does for free.

That also explains the other face of `llmcore-05-hallucination`: the loop only cares whether the next word flows, so it can happily produce paragraphs that are perfectly structured and completely false — it never checks the facts.

## The cost of emitting one token at a time

The loop emits exactly one token per lap, and that "one at a time" constraint shapes the whole experience in three ways:

* **Speed**: every output token means another full forward pass. Longer output, longer wait — which is exactly why `llmcore-02-tokens` says "fewer output tokens, faster replies".
* **Cost**: every output token is billed. Long replies are expensive replies — the most direct fact on your API bill.
* **No going back**: in the usual decoding pipeline, a token is locked in the moment it is drawn; the model never rewrites it. It can only "talk its way back" with later tokens — which is both the source of coherent prose and a breeding ground for logical holes and hallucination.

## Three practical generation strategies

The same loop, paired with different sampling knobs (`llmcore-04-sampling`), gives three very different personalities:

| Strategy | How | Best for |
|---|---|---|
| **Greedy** | always take the top score, no draw | summarization, translation, anything needing certainty |
| **Low-temperature sampling** | draw, but almost only high scorers | general chat, code generation |
| **High-temperature sampling** | low scorers get a real chance | writing, brainstorming, a relaxed tone |

The knobs don't change the loop — they only change the drawing rule at step three. The same model can be an engineer or a poet; it just depends on which parameters you hand it.

## The one sentence to remember

> A model doesn't think something up and then say it. It looks at the current context, scores every token, draws one, and appends it — everything that looks like understanding or knowledge is just statistics running inside that loop.

## A conceptual snippet to tie it together

Real decoding happens in matrix multiplications on a GPU, but the skeleton fits in a few lines. Notice the middle step, the "draw" — it turns a deterministic set of scores into a one-time choice:

```python
# Conceptual demo: one lap of the decode loop (the real model runs on a GPU)
import math, random

# Step 1 — the context so far (a sequence of tokens)
context = ["The", "cat", "sat"]

# Step 2 — one forward pass: a score (logit) for every candidate token
logits = {"on": 3.2, "in": 2.1, "down": 1.0, "under": 0.7}

def softmax(scores):
  exp = {t: math.exp(s) for t, s in scores.items()}
  total = sum(exp.values())
  return {t: e / total for t, e in exp.items()}

# Step 3 — softmax turns scores into probabilities
probs = softmax(logits)

# Step 4 — sampling: draw one token by probability (knobs in llmcore-04-sampling)
next_token = random.choices(list(probs), weights=probs.values(), k=1)[0]

# Step 5 — append it, update the context, then loop back to Step 2
context.append(next_token)
# Stop when an end-of-text token is drawn or the length limit is hit
```

## The whole pipeline in four steps

If you take away just four words from this post:

#### Tokens in

The tokenizer chops the input and it all lands in the context window (see llmcore-02-tokens, llmcore-03-context).

#### Score

One forward pass gives every token in the vocabulary a logit.

#### Draw

Softmax turns scores into probabilities; sampling picks one winner (see llmcore-04-sampling).

#### Append and repeat

The token joins the context; unless an end token shows up, go back to step two.

## A table that ties the series together

| Part | Post | Role in the loop |
|---|---|---|
| token | `llmcore-02-tokens` | the unit the loop consumes — and bills for |
| context | `llmcore-03-context` | the snapshot each forward pass reads |
| logits | this post | the raw score for every candidate |
| sampling | `llmcore-04-sampling` | decides which token wins; stability vs creativity |
| hallucination | `llmcore-05-hallucination` | when the loop flows fluently but wrong |

## And that completes the series

From "predict the next word" (01) to "from scores to words" (here), the llm-core series is whole: you know how the model sees the world (tokens, context), how it picks words (logits, sampling), and where it goes wrong (hallucination). Next, swing the camera to how those scores get *learned* in the first place — starting with how a fluent model gets reshaped into a specialist: `train-02-finetuning`.

#### Q: What does “auto-regressive” mean in the decoding loop?

* The model revises one sentence until it is satisfied

* The model appends its own output to the context and predicts the next token from it

* The model searches the web for similar sentences

* The model replays the same training batch

> 💡 Auto-regressive means feeding output back in as input: the drawn token is appended to the context, which becomes the input for the next forward pass — looping until an end token appears or the limit is hit.
