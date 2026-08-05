# Choosing Hardware: Balancing Capacity, Bandwidth, and Budget

> 📅 2026-08-04 · Deep Dive
> What really matters for running models locally? Capacity decides what fits, bandwidth decides how fast, and quantization pushes the line right. A decision table from hobbyist to pro.

---

You already got a model running with `local-01-ollama`, and the next question almost everyone asks is: **which machine should I buy?** Most people's first instinct is to look at the GPU model number — but for running models, that is often the wrong question. Once you understand two numbers, the answer reveals itself.

***

## Ask the right question first: where are you stuck

"Can't run it" is actually two very different symptoms, and the remedies are completely different:

| Symptom | Real bottleneck | Key number |
|---------|-----------------|------------|
| The model fails to load, OOM crash | Not enough memory capacity | Capacity |
| It loads but takes forever per word | Memory bandwidth too low | Bandwidth |
| It loads and runs, but quality is poor | Model too small or quantized too hard | Capacity + bandwidth |

Capacity decides whether it "fits"; bandwidth decides how fast it runs. A machine needs to pass both to be comfortable, but with a limited budget, figure out which one is *your* bottleneck before spending.

***

## Capacity: the ruler that decides what fits

A model must sit entirely in memory to do inference. The rough formula:

```
Memory needed ≈ params × bits per weight / 8
```

A set of common numbers (KV cache not yet counted):

| Model | FP16 | Q8 | Q4 |
|-------|------|----|----|
| 7B / 8B | ~14 GB | ~7-8 GB | ~4-5 GB |
| 14B | ~28 GB | ~14 GB | ~8-9 GB |
| 32B | ~64 GB | ~32 GB | ~18-20 GB |
| 70B | ~140 GB | ~70 GB | ~40 GB |

**The capacity conclusion is simple: how much memory you have decides which tier of model you can touch.** But do not count only the weights — the OS and other programs eat 4-8GB, and the longer the context, the bigger the KV cache (`llmcore-03-context`). At 8K context, a 7B model's KV cache costs roughly 1GB more. Budget for all of it.

***

## Quantization: pushing the line to the right

Notice the table above: the same 7B model needs 14GB at `FP16` but only 4-5GB at `Q4` — a nearly fourfold difference. That means **quantization can literally rewrite the answer to "how big a model can my machine hold"**: an 8GB machine can't even fit a 7B at FP16, yet runs one comfortably at Q4.

The how-and-why of quantization lives in `llm-04-quantization`. Here, keep only the key point:

**Decide the memory budget first, then the quantization level — the two are chosen together.**

In practice the flow is: first figure out the highest quantization your machine can hold, then work backward to a usable model tier. Quantization is an invisible bonus to your budget: without spending an extra dollar, it pushes your machine's ceiling up one notch.

***

## Bandwidth: what decides "how fast"

After it fits, the second question is speed. LLM inference is a memory-bound workload: every token generated reads the entire model's weights once (that is the operating cost of the "predict the next token" trick from `llmcore-01-next-token`). So **speed is decided mostly by memory bandwidth, not compute power**.

Approximate bandwidth by platform (the full table is in `local-03-mac-vs-gpu`):

| Platform | Approx. bandwidth |
|----------|-------------------|
| M1 / M2 base | ~68-100 GB/s |
| M1 Pro / M2 Pro | ~200 GB/s |
| M1 Max / M3 Max | ~400 GB/s |
| RTX 4090 | ~1000 GB/s |
| RTX 5090 | ~1790 GB/s |

Rough math: a machine with 400 GB/s bandwidth running a 4GB model tops out near 100 token/s; in the real world, with overhead, 20-40 tok/s is already smooth for everyday interaction. **More bandwidth makes the same model more fluid; with too little bandwidth, you have to shrink the model or drop a quantization tier.**

***

## Mac unified memory vs NVIDIA VRAM

Both are called "memory", but the two roads have entirely different philosophies (full comparison in `local-03-mac-vs-gpu`):

* **Mac unified memory**: CPU and GPU share one pool; RAM is VRAM. A 32GB MacBook can in theory hold a 32GB model — big capacity, cheap to add, but bandwidth is comparatively limited.
* **NVIDIA VRAM**: dedicated memory on the graphics card, fixed, expensive, and not upgradeable, but bandwidth is extremely high and the software ecosystem is the most mature.

One sentence: **Mac gets hobbyists in the door with "capacity", NVIDIA raises the pro ceiling with "bandwidth".** For most personal use, a Mac with plenty of memory is often the best value; you only need NVIDIA once you truly need 70B+ or very high throughput.

***

## From hobbyist to pro: a decision table

Putting it all together, here is a table you can match yourself against:

| Where you are | Suggested setup | Models that run well | Budget tier |
|---------------|-----------------|----------------------|-------------|
| Pure beginner | Existing laptop, 16GB RAM | 7-8B @ Q4 | 0 |
| Casual user | 32GB Mac or 12GB VRAM | 7-14B @ Q4 | Medium |
| Enthusiast | 64GB Mac or 16-24GB VRAM | 32B @ Q4 | Medium-high |
| Semi-pro | 128GB Mac or 24GB VRAM | 70B @ Q4 | High |
| Pro deployment | Multi-GPU workstation / server | 70B+ or high throughput | Very high |

### Three shortcuts on a budget

* **Run first, upgrade later**: run a 7B @ Q4 on what you already own — zero cost, build the feel first.
* **Add memory, not a new machine**: upgrading a Mac's memory is usually cheaper than swapping GPUs.
* **Hunt the used market**: an older card with large VRAM is often better for big models than the newest small card.

### Three common misconceptions

* **"Pricier GPU equals faster"**: not necessarily. Running models is memory-bound; what matters is bandwidth, not compute. Some cards that are pricey for compute are merely average for LLMs.
* **"More memory is always better"**: true, but once it fits, extra is invisible — unless you plan to run bigger models.
* **"You must have a discrete GPU"**: no. Mac unified memory is genuinely capable for personal users.

### Do not trust spec sheets alone

Spec sheets are just a starting point. **The real standard is to run your actual data once** (`local-01-ollama` is enough to test); tok/s, quality, and memory usage will tell the truth at once. For cross-model comparisons, `models-06-evaluation` benchmarks help. Numbers lie; real runs do not.

***

## The standard buying routine

When choosing a machine, walk through this order and you will rarely step on a landmine:

#### Count capacity

Work out your usable memory, subtracting the OS and KV-cache overhead.

#### Pick the quantization

Find the highest level that fits; the next post details each tier.

#### Set the scale

Choose a parameter size for the job, then validate with a real run.

> Pick the memory first, then the quantization level, and only then the brand and model. For personal use, a Mac with plenty of memory is often the best-value starting point; NVIDIA only earns its keep when you need 70B+ or extreme speed.

***

## Summary and what is next

This post broke "which hardware" into three quantifiable numbers: capacity, quantization level, and bandwidth. The natural next question: **there are so many quantization tiers — what is the actual difference between Q4_K_M, Q5_K_M, Q6_K, and Q8_0?** The next post, `local-06-quantization-tiers`, works out how much each level saves and costs.

#### Q: For an 8GB machine, which statement best describes its local-AI situation?

* It can hold no model at all, so give up

* A 7B at FP16 does not fit, but the same 7B at Q4 runs fine

* It can hold a 70B model, just slowly

* Memory capacity does not matter; the GPU model number does

> 💡 Capacity decides what fits: 8GB cannot fit a 7B at FP16 (about 14GB), but the same model at Q4 takes only 4-5GB and runs fine. Quantization is exactly what pushes the fits-or-not line to the right.
