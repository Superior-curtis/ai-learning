# Open vs. Closed Models

> 📅 2026-08-04 · Core Concepts
> Llama you download, GPT you call by API — this isn't just 'free vs paid', it's a fundamental choice about who controls what.

---

Open any AI forum and the first thing to get straight isn't "which model is strongest" — it's **"is this model open or closed?"** It decides what you can do with it, who controls it, and which licensing maze you've walked into.

## A spectrum, not a binary

"Open" isn't a switch; it's a spectrum:

```
fully closed              open-weight            truly open source (rare)
GPT / Claude             Llama / Mistral         open weights +
(API only)               / Qwen                  data & training process
```

| Type | Examples | What you get |
|---|---|---|
| **Closed (API)** | GPT, Claude, Gemini | call it via API only |
| **Open-weight** | Llama, Mistral, Qwen, DeepSeek | download weights, self-host |
| **Fully open** | some research models | weights + data + training details (rare) |

## The most-misunderstood point: open-weight ≠ open source

"Open-weight" means **you download the trained numbers (weights)**. But:

* **You often can't see the training data.**
* **Licenses usually carry usage limits** (e.g., Llama requires a separate agreement for commercial use above a monthly-active-user threshold).
* You get the "finished product", not the "recipe."

So don't read too much into "open source." The industry now prefers the more honest term **"open-weight".**

## Where closed models win

Why do so many people pay for closed APIs?

* **Quality usually leads**: flagship closed models sit atop the strongest leaderboards (especially reasoning, long documents).
* **Maintenance & safety are handled**: bug fixes, alignment, security updates are part of the service.
* **Zero hassle**: no GPUs, deployment, or upgrades — call and go.
* **Premium features**: some features are API-only (certain tools, multimodal services).

## Where open-weight wins

So why is the open-source community so passionate?

* **Privacy & compliance**: data never leaves your servers; regulated fields (medical, finance) are almost forced to self-host.
* **Predictable cost**: amortize one-time compute, free from per-token pricing and price hikes.
* **Control**: you can modify, fine-tune (`train-02`), audit — not at the mercy of vendor policy.
* **Offline**: runs without the internet (the home turf of `local-ai`).

#### 對照 / Comparison

## How to choose: a decision table

| Your situation | Better fit |
|---|---|
| Product needs fastest SOTA | Closed (API) |
| Handling records, contracts, internal secrets | Open-weight (self-host) |
| Cost-sensitive, high volume scares the bill | Open-weight (or do the math first) |
| Want on-demand calling, zero ops | Closed (API) |
| Research / teaching / offline dev | Open-weight |

> Rule of thumb: "open or closed" comes before "which model". For teams without sensitive data that want max capability, the closed API is the default; for privacy/cost/control-sensitive work, open-weight is home turf. You can also mix — sensitive parts self-hosted, convenient parts via API.

## This boundary isn't permanent

The market moves: open models (DeepSeek, Qwen) close in on closed flagships every year; closed companies also release weights (some models). **Don't treat "open wins" or "closed wins" as faith** — re-evaluate your spot on the spectrum each year.

## Next

Now that you know "open vs. closed", take a tour of the family stores: GPT, Claude, Llama, Gemini, Mistral, Qwen each have their strengths — `models-02-families-tour`.

#### Q: What's the biggest difference between 'open-weight' and 'open source'?

* Open-weight is weaker

* Open-weight gives you downloadable weights, but the training data is usually not public and licenses have limits

* Open-weight is free; open source costs money

* They’re exactly the same

> 💡 Open-weight means you can download weights and self-host, but 'weights' are just the finished product — training data is often private and licenses carry commercial gates. It's 'getting the model', not 'getting the recipe'.
