# Choosing a Provider: A Practical Decision Framework

> 📅 2026-08-04 · Core Concepts
> One provider or several? Self-host or API? Lay out six axes — quality, latency, price, data handling, rate limits, lock-in — with a decision table, so choosing stops being a gut call and becomes a matter of priorities.

---

The earlier posts answered "where to buy" (`providers-01-ecosystem`) and "what it costs" (`providers-02-pricing`). Now the most practical question: **which provider do I actually pick? One or several? Should I self-host instead?**

A dose of reality first: there is no single right answer, and the answer will change as your stage changes. But there is a **framework you can reuse** — and its essence is one sentence: **figure out your priorities first, then filter through six axes.**

## The six axes: dimensions for choosing

You can score any provider on these six dimensions. Think hard about your own workload first, then apply:

| Axis | The question to ask | Key reference |
|---|---|---|
| **Quality** | Is it accurate enough for *my* tasks? | Your own task set, see `models-06-evaluation` |
| **Latency** | How fast is the first token? Enough throughput? | Small models + streaming are usually fast |
| **Price** | What does the bill look like at my volume? | The cost formula, see `providers-02-pricing` |
| **Data handling** | Where does my data go? Is it compliant? | Closed APIs route data through third parties |
| **Rate limits** | What are the per-second / per-minute ceilings? | Varies widely between providers |
| **Lock-in** | How much rewriting does switching cost? | A unified interface lowers it, see `providers-01-ecosystem` |

The six axes are rarely all-maxed at once — **choosing is inherently "trade off by your priorities," not "find the perfect provider."** Want lowest latency *and* lowest price *and* full compliance? That probably does not exist; you have to decide which two matter most.

## The two most-misread axes: quality and latency

**Quality**: `models-06-evaluation` made the point that "how strong" is not one number — it is "for whom, on which class of tasks." For provider selection, test quality on your own task set. Do not let "the latest leaderboard champion" decide for you. Run 15 of your real inputs, grade them on your rubric, and that is more honest than any ranking.

**Latency**: latency is not just "fast or slow"; it has three layers — **time to first token** (TTFT, how quickly it *feels* to start), **tokens per second** (throughput, how quickly it finishes), and **tail latency** (how slow the slowest 5% are). For chat products, TTFT drives perception; for batch aggregation, throughput drives cost; for SLA-sensitive services, tail latency decides whether you rage.

## A five-step decision process

Collapse the axes into a walkthrough you can actually run:

#### Pin your priorities

Write down the 2–3 axes you care about most (e.g., "latency" + "data compliance"). This sets the order of everything after.

#### Test quality on your own tasks

Run a dozen of your real inputs against each candidate and grade with your own rubric — trust your tasks, not leaderboards (see `models-06-evaluation`).

#### Estimate price and rate limits

Use the cost formula from `providers-02-pricing`; check whether the per-second or per-minute caps of each provider survive your volume.

#### Check data handling and lock-in

Can your data enter a third party? How costly is switching? Does it comply? A "no" here can veto outright.

#### Decide: one, several, or self-host

Land on the decision table below.

## One provider, or several?

* **One is enough**: your tasks are homogeneous and your scope is stable. Managing a single provider is simplest, and volume earns better pricing leverage.
* **Several**: you want "the best model for each kind of task," you want **redundancy** (when one is down, another takes over), or you want routing that sends easy tasks to cheap models. A proxy platform (`providers-01-ecosystem`) keeps the switching cost low.
* **Dynamic routing (advanced)**: use rules or cost to route requests automatically — small calls to small models, hard problems to flagships. That is already "orchestrating models as a service," in the same spirit as `agents-01-what-is-agent`.

Concretely: a startup building a support assistant might find that 80% of routine questions are fine on a cheap small model while the hard 20% need a flagship — that argues for "multiple providers + routing." Conversely, a single-purpose tool that only "translates contracts into plain language" should use one flagship and not invent extra complexity.

## When to self-host (instead of API)

Self-hosting really means going back to the `models-01-open-vs-closed` fork: you need both **"you can get the weights (an open model)"** and **"you are willing to run and maintain deployment yourself."** Both must hold for it to pay off. Signals that point to self-hosting:

* Data is extremely sensitive (healthcare, finance, internal secrets) and **must not leave your servers**.
* Volume is so high that "pay per token" exceeds "depreciation of the hardware."
* You need offline operation and full control over upgrade timing.

The technical map lives in the `local-ai` series: `local-01-ollama` is the friendliest entry point.

## Your decision table

A quick reference that spreads "situation → recommendation":

| Your situation | Recommendation |
|---|---|
| Want top quality, avoid ops | Closed API (one flagship) |
| Latency-first, high volume, sensitive | Fast cheap model + streaming; trim with `providers-02-pricing` |
| Data-sensitive / must be fully compliant | Self-host open weights (`models-01-open-vs-closed`) |
| Tasks span many models, cost-optimized | Proxy + multiple providers (`providers-01-ecosystem`) |
| Afraid of single dependency | One primary + one backup |
| Just starting, volume is small | Use one comfortable flagship API and get moving |

> One sentence to remember choosing: "Set priorities first, filter through the six axes, and only then talk about 'which is strongest.'" There is no best — only "the combination that fits your priorities" — and it shifts as your stage shifts, so revisit it once a year.

## How to cost out lock-in

Lock-in is the most underestimated axis: "we will worry about it later" feels fine until **switching three months in means rewriting half the product.** The real cost of lock-in has three parts:

* **Interface**: each vendor's SDK shapes the calling code a bit differently; hard-coding it means rewriting it.
* **Data and tooling**: caches, fine-tunes, and vendor-specific features (the LoRA-style artifacts from the `fine-tuning` series) do not travel when you move.
* **Team habits**: your prompts, eval sets, and debugging flows are all migration costs.

The fix is not "never switch" — it is **abstracting the interface first**, even a thin wrapper of uniform functions, or just using a proxy platform (`providers-01-ecosystem`). Only by making "switching" cheap do you earn the right to actually switch.

## Treat the choice as "moving," not "forever"

The provider market reshuffles every few months: prices change, models are replaced, and open models keep closing the gap with closed flagships (`models-01-open-vs-closed`). So choosing is not a marriage — it is a **quarterly or half-yearly review** you schedule. As long as you abstract the interface well (or use a proxy), the cost of re-evaluating stays low — which is exactly the biggest lever for keeping the lock-in axis cheap.

## Next

Once you have picked a provider, one layer deeper sits a bigger question: **what the cost structure of "running one inference" really looks like** — it determines whether you should build your own compute or just rely on APIs. That is the home turf of `econ-01-inference-cost`. Pocket the provider map; now we walk toward the industry's spreadsheets.

#### Q: Your app handles highly sensitive medical records and must be fully compliant. Which route fits best?

* Connect straight to the OpenAI flagship API

* Route through a proxy to several closed providers

* Self-host open-weight models so data never leaves your servers

* Pick the provider with the lowest unit price

> 💡 With highly sensitive data and full-compliance demands, closed APIs pass your data through third parties and are hard to fully control. Self-hosting open weights is the only option that keeps data on your own servers — the crux of the open vs. closed fork.
