# LoRA in Practice: Making Fine-tuning Cheap

> 📅 2026-08-04 · Deep Dive
> From the LoRA idea to something you can run. What an adapter is, how to pick rank, how much data, which libraries, and how to fuse and quantize at the end.

---

`train-02-finetuning` left you with a slogan: LoRA is "the secret to cheap fine-tuning." Easy to grasp in concept — don't touch all the weights, just add a few low-rank parameters on a side branch. But "knowing the idea" and "actually producing an adapter" are two different things. This post closes that gap to the point where you can run it: what an adapter really is, how to pick rank, how much data, which libraries, and how to shrink the result at the end.

## The bet in one line

Full fine-tuning updates every weight (a 7B model means seven billion parameters). LoRA bets that **the change you need lives in a much smaller "low-rank" space.**

The trick: next to certain layers of the base model, insert two small matrices (A and B) whose product acts as a small adjustment to that layer. You train only those two pieces; everything else stays frozen. After training, those two matrices together are an **adapter**.

```text
Full fine-tuning: update 7 billion parameters
LoRA:             update only a few million adapter params (often < 1%)
```

Visually it looks like this — the original weights never move, only the small branch beside them:

```text
Frozen base weights W (untouched)
│
└── side branch: A (small) → B (small) → output = A×B adjustment
│
training updates only A and B; W stays fixed
```

## What an adapter actually is

**An adapter is those two trained matrices** — in practice a `.safetensors` file of a few MB to a few dozen MB. It does nothing by itself; before it meets the base model it isn't even a model. It must sit *on top of* the base model, adding a tiny offset to each layer, so the overall behavior becomes what you wanted.

That explains two practical facts:

* The same base model can hold several adapters, each for one job (one for tone, one for summarization); mount whichever you need.
* You don't duplicate the base model — one copy on the GPU, and switching adapters swaps only a few MB.

## Rank: one small number decides quality and size

LoRA's core parameter is **rank** (usually written `r`). It decides how wide the side branch is — and therefore how big the adapter is and how close the result gets to full fine-tuning.

| rank | Meaning | Cost |
|---|---|---|
| Low (r = 4–8) | Narrow branch, small adapter, cheap | Less expressive; complex tasks may under-learn |
| Mid (r = 16–32) | Sweet spot for typical tasks | Trade-off |
| High (r = 64+) | Closer to full fine-tuning | Bigger adapter, diminishing returns |

Rule of thumb: **start at r = 16**, go up only if it's not enough. Higher rank is not automatically better — once you're roughly sufficient, more rank just fattens the adapter and raises overfitting risk without visible gains.

> Rule of thumb: LoRA approximates full fine-tuning with a few million parameters, not replaces it. Rank is the "quality vs. size" knob; start around r ≈ 16, don't max it out on the first try.

### Two smaller knobs: alpha and target_modules

Besides rank, two parameters confuse beginners, though neither is subtle:

* **`lora_alpha`**: how much the branch output is scaled before adding back into the weights. Intuitively, it's the "volume" of the side branch. A common start is twice the rank (r=16 → alpha=32); beyond that it's a choice of relative strength, not bigger-is-better.
* **`target_modules`**: which layer types get a branch. More layers → more learning power, bigger adapter, more cost. A cheap starting point is just the Q and V projections (e.g. `["q_proj", "v_proj"]`); add more only if needed.

One line: rank decides "how wide," alpha decides "how loud," target decides "how far." Keep all three small and LoRA stays genuinely small and fast.

## How much data do you need?

People have fantasies about dataset size; LoRA pushes the bar lower. Rough magnitudes:

* **Style / tone**: a few hundred high-quality examples. You're after a "relatively small, relatively fixed" shift.
* **Fixed skills / structured output**: a few hundred to a few thousand. More isn't automatically better — what matters is quality and consistency.
* For contrast: full or large-scale fine-tuning wants thousands to tens of thousands; LoRA often needs a fraction of that.

The real bottleneck is almost always **data quality**, not quantity — exactly what `finetune-03-data-and-eval` will dissect. For now: **little and clean beats lots and dirty.**

## Which libraries

Two tools carry LoRA from "idea" to "one-line switch":

* **HF PEFT**: the de facto standard. `get_peft_model` wraps LoRA on in one call; the ecosystem is the most complete.
* **Unsloth**: built for speed and memory savings — claims to slash training time and VRAM substantially, which is especially friendly to consumer GPUs.

Below is the spirit of attaching LoRA: wrap it, train only the branch.

```peft&#x20;lora
from peft import LoraConfig, get_peft_model

lora = LoraConfig(
  r=16,           # rank: higher = closer to full params, bigger adapter
  lora_alpha=32,  # scaling, 2x the rank is a common start
  target_modules=["q_proj", "v_proj"],  # pick certain layers
)
model = get_peft_model(base_model, lora)
# only adapter params are trained; the rest stay frozen
```

If you'd rather move fast than touch details, Unsloth usually takes just a few changed imports and handles bits and optimizations for you.

## Fusing and quantizing: the final shrinking tricks

After training you have two options:

**1. Fuse / merge.** Some people don't want a "base + adapter" pair and prefer the effect welded permanently back into the weights, producing one normal model file. `PeftModel.merge_and_unload()` does exactly that. The price: the output is as big as the base model you picked (GB-scale).

**2. Quantize.** To shrink the base model itself and save memory, that's the `llm-04-quantization` playbook: represent weights with fewer bits. LoRA and quantization stack — a quantized base plus LoRA training is a common path for "I want to fine-tune on one consumer GPU."

> Remember the relationship: the adapter is the change; the base is the foundation. Fusing welds the change back in (one big file); not fusing means carrying "a small change + a big base" as a pair. Both are legitimate — neither is more "correct."

## How small is the output, really

The headline number: **adapters usually land between a few MB and a few dozen MB.** Contrast that with base models that start at gigabytes:

| Artifact | Size magnitude |
|---|---|
| Base model (7B, FP16) | ~14 GB |
| LoRA adapter | ~5–80 MB |
| Quantized base model | ~4–8 GB depending on bits |

So the "portable specialty" of fine-tuning is that little adapter — it fits in a git repo, lives on the cloud, and mounts onto another machine in seconds. The full thing you'd actually drag around is still base + adapter, or the fused GB-scale file.

## Three beginner traps

**1. Crank the rank.** "Bigger rank must be stronger" — straight to r = 128. Result: bigger adapter, slower VRAM, no better output, and more overfitting risk. Start at 16.

**2. Dirty data, unknowingly.** Train on repetitive, inconsistently formatted examples and the model *learns the mess* — only to reply sloppily in production. Data quality is the ceiling for LoRA, and the next post dissects it.

**3. Using LoRA for "knowledge updates."** Weekly-changing product data isn't LoRA's job — that's RAG's (see `finetune-01-finetune-vs-rag`). LoRA learns fixed patterns, not "whatever changes every time."

## Wrap-up

LoRA lets you learn a fixed skill with a little data, a little VRAM, and one small file — without touching that multi-gigabyte heart. You now know how to make it cheap. Next, the harder question: **how to prepare the training data, and how to evaluate that the tuned model is actually better** (`finetune-03-data-and-eval`).

#### Q: Your trained LoRA adapter works well but is larger than you'd like. Which parameter should you change first to shrink it?

* Lower the rank (e.g. 16 to 8)

* Switch to a bigger base model

* Add more training steps

* Crank lora_alpha way up

> 💡 Rank directly decides branch width and adapter size. Lowering it narrows the branch and shrinks the adapter, at the cost of some expressiveness. A bigger base model or more steps is heavier and more expensive, and alpha is not 'bigger is better'.
