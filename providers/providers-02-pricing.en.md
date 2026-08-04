# Token Pricing and Comparison: Reading Your Bill

> 📅 2026-08-04 · Core Concepts
> How do model APIs charge you? Grasp input vs. output token prices, lay out a full turn's cost, then trim the bill with caching and batching — the point is not 'which unit price is cheaper' but 'how many tokens the same task actually uses.'

---

The first shock after hooking up an API is usually not "is the model strong?" but **the bill**. One careless integration, and a few thousand requests can send your cloud spending out of control — often without you being able to explain "why is this this expensive." This post takes apart exactly **how tokens become dollars**, so you go from "shocked by the bill" to "able to predict it."

Here is the skeleton of the conclusion up front: providers almost always **price input and output tokens separately**, and the skill you need is reducing a full turn into those two numbers. Everything else — language, context length, and whether you try to economize — is a variable on top of that.

## Two prices: input is cheaper, output is more

Memorize this one split first. Providers almost always bill separately, and **output is usually 2–4× the price of input**:

| Direction | Meaning | Relative unit price |
|---|---|---|
| **Input tokens** | your prompt and context, sent out | cheaper |
| **Output tokens** | the reply the model generates | pricier (2–4×) |

Why is output pricier? Because **generation samples "the next token" one step at a time** (see `llmcore-01-next-token`), and **every step runs a full forward pass**. Input is computed once and then sits there, waiting to be attended to; output is manufactured token by token along the way. One is "compute once, then read"; the other is "compute a full pass per character produced" — so their costs differ by nature.

## How a turn's cost is computed

Remember the lesson from `llmcore-02-tokens`: **the unit of billing is the token.** So a full turn's bill is simply two lines added together:

```
total cost =
  input token count  × input unit price
  + output token count × output unit price
```

Everything hinges on those two counts. A key reminder (one of `llmcore-02-tokens`'s core takeaways): **the same sentence in Chinese often takes 2–3× the tokens as English** — so its cost scales proportionally too. Language, how big a document you stuff into context, and how verbose the model is, are all variables. To save money, "send fewer tokens" often beats "pick the cheaper brand."

## A worked example: the bill for one turn

A formula alone is too abstract. Compute a real one. Suppose you use a model priced at $3 / million input tokens and $15 / million output tokens, and weigh a "long Q\&A" turn:

```Computing&#x20;the&#x20;cost&#x20;of&#x20;one&#x20;turn
input_price_per_m = 3      # USD per 1M input tokens
output_price_per_m = 15     # USD per 1M output tokens

# one real turn: a long prompt + context in, a long reply out
prompt_tokens     = 24_000   # context-heavy; more still if Chinese
completion_tokens = 1_600    # the model's generated reply

input_cost  = prompt_tokens      / 1_000_000 * input_price_per_m
output_cost = completion_tokens  / 1_000_000 * output_price_per_m
turn_cost   = input_cost + output_cost

print(round(turn_cost, 4))   # roughly 0.096 USD
```

Multiply that 0.096 USD by "a few thousand calls a day," and you have that application's monthly baseline. **Learn to compute it first, and you will not be caught off guard.**

## Confirm token counts from the API itself

You do not have to guess it all. Every response reports its usage (remember to log it, as `providers-01-ecosystem` advised), and most SDKs offer counting helpers. Build these habits:

* During development, print `input_tokens` / `output_tokens` to see how much of your prompt is actually consumed.
* When supported, call a pure counting endpoint to measure the same prompt before it hits the model.
* Feed those numbers back into the formula above, and your cost estimate moves from "vibes" to "a ledger."

## The right way to "compare prices": it is not about the unit price

The cheapest unit price is not necessarily the cheapest bill — because the bill drinks "tokens," not "unit price." One illustrative table (numbers are examples, not real quotes):

| Model | Per 1M input | Per 1M output | Actual tokens for the task | Estimated cost |
|---|---|---|---|---|
| Cheap small model | $0.5 | $2 | Verbose, often retried | Not necessarily low |
| Flagship model | $3 | $15 | Terse, first-try correct | Often lower |

The reason "pricier can be cheaper" is that the bill = **unit price × token count**, and the token count lives in the *task*, not in the price sheet. Three gaps that actually matter:

* **Stronger capability**: it may answer correctly on the first try, while a weaker model makes you retry two or three times — **retried tokens often cost more than the higher price**.
* **Tighter output**: a good model writes less fluff, so output tokens (the more expensive kind) stay low.
* **Context is the base**: in a long-context system (RAG, see `rag-01-what-is-rag`), **input tokens are the big line item**; each extra page of context is extra money per call.

## The hidden items people miss on the bill

Even with the formula down, a few costs slip under the radar:

* **Cache misses**: caching looks cheap, but each cold start of a cache entry still bills at full price — and hot-but-constantly-changing prompts save almost nothing.
* **Tool calls / agent loops**: tool calls inside the model's reply count as output tokens, and every turn an agent takes adds to input — see `agents-01-what-is-agent`. More turns, more money.
* **Multimodal input**: images and audio are often priced per image / per second rather than by plain tokens, and can vastly exceed text-only assumptions (see `models-03-multimodal`).
* **TTL expiry**: some providers expire cache hits and free daily generations after a time window, so they snap back to full price when the window rolls.

Fold these four into your cost model and your estimate will track the real bill.

## Trimming the bill: caching and batching

You do not have to "pick the cheapest" to save. Turn on these two switches first; they often pay off immediately:

**1. Prompt caching**: mark long, rarely-changing context (system prompts, long documents, conversation history) as cacheable. On a cache hit, that segment of input tokens is deeply discounted (an order of magnitude at some providers). Long, mostly-stable histories save the most.

**2. Batching**: some providers offer a "not real-time but cheaper" batch endpoint — ship it in the evening, get results a few hours later, at a lower unit price. Ideal for offline aggregation, nightly jobs, and high-volume generation that does not need to be instant.

Compare "with / without caching":

#### 對照 / Comparison

> One sentence to remember the saving lever: "Cache first, batch second, and let optimization follow volume — do not over-engineer on day one." Before your volume exists, spending energy on cache design is premature.

## A practical caution: estimate the magnitude before optimizing

Caching and batching are tempting, but do not build a tangle of cache logic before your volume justifies it (see `llmcore-03-context` on context cost). A pragmatic order: **estimate the monthly baseline with the formula first; only if the bill is visibly high, then add caching and batching.** Optimization follows volume; it is not a day-one move.

## Next

"Affordable" is not the same as "the right pick." Once you can read the bill, the closing post spreads six axes — quality, latency, price, data handling, rate limits, lock-in — into a decision table and shows you how to trade them off by priority: `providers-03-choosing`.

#### Q: Why are 'output tokens' usually pricier than 'input tokens'?

* Because output consumes more electricity

* Because generation samples one token at a time, and every step reruns a full forward pass, so it costs more

* Because providers just want more money, with no technical reason

* Because output tokens are longer than input tokens

> 💡 Generation is not computed all at once — it samples tokens one by one (next-token prediction from llmcore-01), and each sample runs a full computation, so the production end is naturally pricier.
