# The Real Cost of One Inference Call

> 📅 2026-08-04 · Core Concepts
> Hit send and the answer appears — but every generation burns GPU time and electricity. Split inference into prefill vs output tokens, and see where a call really costs money — and what a heavy user burns in a month.

---

You hit send and the model answers almost instantly. Easy to assume the reply is free. It isn't — **every generation burns a very expensive machine.** GPUs spin, electricity flows, chips depreciate, and it all ends up on the API bill as "dollars per million tokens." That bill is the denominator of every app-layer business in `industry-01-value-chain`.

This article opens that bill: where the money goes in one model call, why output costs more than input, and what a heavy user actually spends in a month. Understand it and you'll see why some AI products charge and others quietly burn cash.

## A call has two kinds of tokens

To do the math, split one call into two phases. A model doesn't "think everything" and then dump it out. As `llmcore-01` put it, the model first reads your entire prompt, then squeezes out the answer one token at a time.

| Aspect | Prefill (reading) | Output (generating) |
|---|---|---|
| What it does | Reads your whole prompt at once | Writes the answer token by token |
| How it computes | Parallel, all at once | Sequential, one token at a time |
| Typical speed | Hundreds to thousands of tokens/sec | Tens of tokens/sec |
| Price | Cheap | Expensive (often several times more) |

Reading is like scanning a full page in one glance; generating is like speaking the answer word by word. The second is much slower — and much pricier. This isn't a billing preference; it is a direct consequence of the physics of the hardware.

## Why output costs more

The reason fits in one idea: **it can't parallelize.** Each next token depends on the previous one, so generation is strictly sequential. Worse, computing one token means reading the entire model's weights from memory — the whole "brain" passes through once. On hardware that is a memory-bandwidth constraint; no amount of wishing speeds it up.

Three direct consequences:

* **Time is money**: GPUs are billed per hour. Generating 1,000 tokens takes tens of seconds — the GPU is genuinely busy for those tens of seconds.
* **Batching can't smooth it**: reads can be packed side by side and amortized across many requests; generation is sequential, so there is little left to amortize.
* **It occupies the machine**: while generating, the GPU is held by this one request and can't serve anyone else.

That is why nearly every API price sheet charges more for output than for input — often several times more. Knowing who pays for which token is the first step to reading any AI product's pricing; a lot of the "why is this so expensive" complaints trace straight back to the output side.

## The hidden line: hardware amortization

Even beyond the GPU, money hides in **amortization**. A flagship GPU costs tens of thousands of dollars; you don't throw it away after a year. Its cost is spread over every second it serves — divided by its three-to-five-year life and by how much it is actually used, then folded into the per-million-token price.

This leads to the most overlooked sentence in inference economics: **an idle GPU still burns money.** Power is billed, depreciation accrues, the data center must be maintained. A provider's real cost is "GPU-hours × utilization," not "how many tokens came out." Grasp that and you understand why providers push caching and throughput so hard — both are ways of raising the same denominator and spreading fixed cost over more tokens.

## Unit economics, worked by hand

Let's turn that into numbers. The figures below are **illustrative, not a real price list** — but the skeleton is exactly what you'll see on a bill:

```python
# Illustrative unit economics — not a real price list
input_tokens  = 2000    # prompt length
output_tokens = 1000    # answer length

price_in  = 2.50  / 1_000_000   # prefill cost per token
price_out = 15.00 / 1_000_000   # output cost per token (much higher)

call_cost = input_tokens * price_in + output_tokens * price_out
print("one call costs  $%.4f" % call_cost)

# timing: reads parallelize, generation does not
prefill_s = input_tokens / 500    # assume 500 tok/s reading
output_s  = output_tokens / 60    # assume 60 tok/s generating
print("read ~%.1fs, generate ~%.1fs" % (prefill_s, output_s))

# heavy user: 50 calls a day
monthly = call_cost * 50 * 30
print("heavy user monthly  $%.1f" % monthly)
```

Work it through: one call is roughly 0.005 + 0.015 = **$0.02**. A user who makes 50 calls a day costs about **$30 a month** in pure inference. Output dominates — 1,000 output tokens cost three times as much as 2,000 input tokens.

> Remember this line: prefill is cheap, output is expensive. See which direction the money flows and you know where to save — almost every cost trick is really about producing fewer output tokens.

## From one call to a monthly bill

A single $0.02 call sounds like nothing. Scale it up and it bites:

| Scenario | Cost per call | Volume | Monthly inference cost |
|---|---|---|---|
| Light user | $0.002 | 5 calls/day | ~$0.30 |
| Typical user | $0.02 | 50 calls/day | ~$30 |
| Heavy / agentic | $0.05 | 200 calls/day | ~$300 |

This table explains why `industry-01-value-chain` called the app layer "the most divergent" in margins: the same feature, used as a once-in-a-while lookup versus an always-on assistant, moves your cost by a factor of a hundred. **Generation-heavy products (chat, agents) burn money far faster than reading-heavy ones (summaries, retrieval)** — which forces completely different pricing and acquisition strategies.

#### 對照 / Comparison

## Price backwards from your unit economics

$0.02 is a cost, not revenue. To turn it into a business you need to work the price backwards: at a 60% gross-margin target and a cost of $0.02 per call, that call must be priced at about $0.05 just to break even — before marketing and operations. The numbers are illustrative; the point is the method:

| Cost per call | Target margin | Must charge | Monthly ARPU (50 calls/mo) |
|---|---|---|---|
| $0.02 | 40% | ~$0.033 | ~$1.70 |
| $0.02 | 60% | ~$0.05 | ~$2.50 |
| $0.02 | 80% | ~$0.10 | ~$5.00 |

Too many AI products leave cost until the very end and wind up with "the more users, the bigger the loss." **Healthy unit economics are derived backward: compute cost first, then price — not price first and pray the cost is low enough.**

## How to use this bill

Now that the structure is clear, three levers appear:

1. **Generate less**: asking for a "short answer" beats any discount — it trims the most expensive tokens.
2. **Cache repeated inputs**: sending the same prompt twice shouldn't pay the read twice; caching rules differ by vendor, and `providers-02-pricing` covers the concrete pricing options.
3. **Route by difficulty**: simple tasks go to cheap small models, hard ones to the flagship (see `models-05-small-models`).

The deeper shift is mindset: **inference cost is not luck; it is a variable you design.** Treat tokens as a real resource and your product economics will look healthier than most teams.

## When "cost" actually becomes a life-and-death line

Whether cost matters comes down to two numbers: **cost per call × call volume.** A few scenarios where it quietly decides survival:

* **High volume, low price**: autocomplete, bulk translation, transcripts — each call is cheap, but you might sell millions a day at a low price. One order of magnitude in cost eats a whole quarter of margin.
* **Agent loops**: a "task" is not one call. `agents-02-orchestration` showed agents often split a task into dozens of steps, each a call — amplifying cost tenfold or more.
* **Huge outputs**: long reports, long replies — the longer the output, the faster it burns.

Conversely, some scenarios barely care: you sell the quality of the result, not the quantity of tokens, and the ticket is high enough. **The test is simple: when inference cost approaches or exceeds the revenue each user brings, it's a life-and-death line. Otherwise, spend the effort on product value instead.**

## The efficiency flywheel: why cost drifts down over time

The per-token price is not a constant — a force in the industry keeps pushing it down:

* **Better chips**: each new generation of hardware lowers the compute cost per token (see `econ-02-gpu-economics`).
* **Smaller models**: distillation and quantization let "good enough" capability run on much smaller models; `models-05-small-models` and `llm-04-quantization` are both tools on this line.
* **Engineering optimization**: caching, batching, inference servers — providers and the open-source community are really doing the same thing: raising the output of every GPU.

But watch the other side: **every drop in cost gets absorbed by heavier use.** Users demand longer outputs, agents decompose into more steps, and products wedge AI into more corners. So rather than chasing today's price, watch the cost structure — those who understand it can redesign their product at every efficiency leap instead of being chased by the bill.

## Next

That was the bill for one call. The natural follow-up: **why is GPU compute so expensive, and why is it the industry-wide bottleneck?** That is exactly what `econ-02-gpu-economics` answers.

#### Q: For the same one token, why does output (generation) usually cost more than input (prefill)?

* Because generated text is higher quality

* Because generation is strictly sequential, keeping the GPU busy the whole time, and each step reads the entire model from memory

* Because input tokens are usually shorter

* Because providers simply want more money

> 💡 Each output token depends on the previous one, so generation cannot parallelize; every step must read all weights from memory (a memory-bandwidth bottleneck), and time equals money on billed hardware.
