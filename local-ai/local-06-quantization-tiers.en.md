# Quantization Tiers in Practice: Picking Between Q4 / Q5 / Q6 / Q8

> 📅 2026-08-04 · Getting Started
> What do Q4_K_M, Q5_K_M, Q6_K, and Q8_0 in filenames actually mean? How much memory each level saves, how much quality it costs, and which tier fits which model size.

---

Downloading local models, you will hit the same multiple-choice question nearly every time: what do the `Q4_K_M`, `Q5_K_M`, `Q6_K`, and `Q8_0` suffixes on filenames actually mean? Pick too high and the file will not fit; pick too low and quality visibly drops. This post works out how much each level saves and costs, once and for all.

***

## Read the code in the filename first

`llm-04-quantization` covered how quantization compresses weights from 16 bits down to fewer. The code on the filename is shorthand for "how hard it was squeezed":

| Code piece | Meaning |
|------------|---------|
| `Q4` / `Q5` / `Q6` / `Q8` | Average bits per weight |
| `_0` | Plainest integer quantization (e.g. `Q4_0`) |
| `_K` | k-quant: handles the "important" tensors more carefully with more bits |
| `_M` / `_S` / `_L` | Mixed strategy: M is medium, S is smaller, L is larger |
| `F16` / `FP16` | No quantization at all, raw 16-bit |

So `Q4_K_M` means: 4-bit target, k-quant algorithm, mixed strategy (key layers get a few more bits). The same model at `Q4_K_M` is usually a solid notch better than the old-style `Q4_0` — that is the practical gift of the `local-02-llamacpp` ecosystem.

***

## How much each level saves, and costs

Using a 7B model, lay the common tiers out in one table:

| Tier | Approx. bits/weight | 7B size | Quality | Best for |
|------|--------------------|---------|---------|----------|
| FP16 | 16 | ~14 GB | Baseline | Lots of memory, maximum quality |
| Q8_0 | ~8.5 | ~7.6 GB | Nearly lossless | Enough memory, quality first |
| Q6_K | ~6.6 | ~5.9 GB | Close to original | High quality with headroom |
| Q5_K_M | ~5.5 | ~5.0 GB | Very good | Balance of quality and size |
| Q4_K_M | ~4.8 | ~4.4 GB | Good | Best default when memory is tight |
| Q4_0 | ~4.1 | ~3.9 GB | Acceptable | Just testing the pipeline, speed first |
| Q3_K_M | ~3.9 | ~3.5 GB | Noticeable drop | Big models, no other option |
| Q2_K | ~2.6 | ~2.4 GB | Poor | Almost never recommended |

**The point is not to memorize numbers but the trend**: from Q8 down to Q4 you save roughly 40% of memory while quality drops only a little; below that, at Q3 and Q2, the memory savings shrink while quality falls faster and faster. **Q2 is the "saves little, costs a lot" last resort.**

***

## Pick the practical tier by model size

The same tier means entirely different things on different-sized models. **The bigger the model, the more it tolerates quantization** — a 70B at Q4 is nearly indistinguishable, while a 3B at Q4 can get visibly dumber. Practical tiers roughly look like:

| Model size | Recommended starting tier | Notes |
|------------|---------------------------|-------|
| 1-3B | Q8_0 or FP16 | Files are small anyway; no need to skimp |
| 7-8B | Q5_K_M | Drop to Q4_K_M only if memory is tight |
| 14B | Q4_K_M / Q5_K_M | Depends on memory |
| 32B | Q4_K_M | Often the only workable zone |
| 70B+ | Q4_K_M (or Q3) | Fit first, argue later |

The logic of this table is one sentence: **use a higher tier for small models, a lower tier for big ones.** Big models tolerate quantization; small models cannot afford even one notch of loss.

***

## How to find the sweet spot

The "sweet spot" has a simple definition: **the highest-quality tier your memory can actually hold.** The routine:

#### Count usable memory

Work out how much is left for weights after the OS and KV cache.

#### Find the ceiling tier

Use the table above to find the highest tier that fits; start there.

#### Try one up and one down

Go one tier up for quality, one tier down for speed, and feel the gap.

#### Decide on real data

Run your actual workload; the tier that passes on both quality and speed is the answer.

One practical reminder: **the quality gap of "one notch" is far smaller than you imagine.** Most people cannot feel the everyday difference between Q4_K_M and Q5_K_M. Instead of agonizing over that single notch, spend the saved memory on a bigger model or a longer context — usually far more noticeable.

***

## Do not forget the KV cache: context is memory too

All the numbers above count only "weights." But as your conversation grows, the model also maintains a KV cache (the metadata of what it has already processed — see `llmcore-03-context`), and this memory is a **variable that grows with context** — weight quantization does not reduce it.

For a 7B model, the rough pattern is:

| Context length | KV cache, approx. |
|----------------|-------------------|
| 4K | ~0.5 GB |
| 8K | ~1 GB |
| 32K | ~4 GB |

So "my machine just fits a 7B at Q4" does not mean you can run a 32K context — try it and you get an OOM. **When choosing a tier, budget the KV cache in too**; this also explains why some people prefer Q4 to save weight memory, just to leave room for a longer context.

***

## Feel the difference with the comparison slider

The same question, the same model, two very different quantized states:

#### 對照 / Comparison

The left is good enough; the right is steadier. **The point is not which side is better — it is which side your machine and your task can afford.**

***

## Compare it yourself, once

The most reliable way is to run both. Ollama takes a quantization tag directly; llama.cpp runs two GGUF files side by side:

```bash&#x20;—&#x20;the&#x20;same&#x20;model,&#x20;two&#x20;tiers,&#x20;run&#x20;once&#x20;and&#x20;you&#x20;will&#x20;know
# Ollama: specify the quantized version with a tag
ollama run qwen2.5:7b-instruct-q4_K_M
ollama run qwen2.5:7b-instruct-q8_0

# llama.cpp: two GGUF files side by side
llama-cli -m models/qwen2.5-7b-instruct-q4_K_M.gguf -p "write a short poem" -n 64
llama-cli -m models/qwen2.5-7b-instruct-q8_0.gguf -p "write a short poem" -n 64
```

Prepare one **hard, checkable question** (something like a logic puzzle), then compare the two tiers on answers and speed — far more reliable than any spec sheet.

***

## Common misconceptions

* **"Lower number is worse, so always take the highest"**: wrong. If it does not fit, it is zero; pick the highest that fits and only that has meaning.
* **"Q4_0 and Q4_K_M are the same"**: they are not. The K variant's k-quant groups weights more finely; at the same 4 bits, quality is clearly better — pick `_K` over `_0` when you can.
* **"Once quantized, there is no going back"**: no. A GGUF can always be re-quantized, though re-quantizing only makes it worse, so grab the original FP16 or the official quantized file in the first place.

> The sweet spot is the highest tier that fits. Count your usable memory, then work backward to the tier; small models take a higher tier, big models a lower one — the bigger the model, the more it tolerates quantization.

***

## Summary and what is next

This post broke the trade-off between `Q4` and `Q8` and `FP16` into plain terms: lower tiers get smaller, but quality starts sliding noticeably below `Q4`; in practice, "the highest tier that fits" is a rule that rarely steers you wrong.

By now you can pick your hardware (`local-05-hardware`) and your quantization tier yourself. The next question turns a bit philosophical: **if everything lives on your own machine, is privacy truly airtight?** The next post, `local-07-offline-privacy`, spells out offline and privacy in full.

#### Q: A machine can hold only 5GB of weights. Running a 7B model, what is the most sensible choice?

* FP16, the best quality

* Q4_0, the fastest

* Q4_K_M, the balance of quality and size

* Q2_K, the smallest file

> 💡 The sweet spot is the highest tier that fits. 5GB fits neither FP16 (about 14GB) nor Q5_K_M (about 5.0GB is too tight), while Q4_K_M (about 4.4GB) is the best quality/size balance; Q4_0 is smaller but worse, and Q2_K costs far too much quality.
